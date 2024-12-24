#3rd_Party 

每个开发者编程中都会记录log信息，多数人都会使用log第三方库，log库使用起来很方便，但我们也需要了解log系统的原理，这里以glog为例进行分析。

### 开始

这里不会介绍glog中是如何控制INFO、ERROR等级别的输出的，其实就是一个宏控制，主要介绍google glog中一次LOG(INFO)过程中究竟发生了什么，以及为什么glog是线程安全的。

glog中的LOG(INFO)其实是一个宏定义，如下：

```cpp
#define LOG(severity) COMPACT_GOOGLE_LOG_ ## severity.stream()

那么LOG(INFO)相当于
COMPACT_GOOGLE_LOG_INFO.stream()
```

而COMPACT_GOOGLE_LOG_INFO也是一个宏，如下：

```cpp
#define COMPACT_GOOGLE_LOG_INFO google::LogMessage(__FILE__, __LINE__)
那么LOG(INFO)相当于
COMPACT_GOOGLE_LOG_INFO.stream()也就相当于
google::LogMessage(__FILE__, __LINE__).stream()
```

这里就构造了一个google::LogMessage的临时对象，语句执行完就会自动析构，google::LogMessage的大体结构如下：

```cpp
// 这里是摘下来的极简版，实际比这复杂很多
class LogMessage {
    LogMessageData::LogStream stream_; // 自定义的输出流，继承的std::ostream
    LogMessage(const char* file, int line);
    ~LogMessage(); // 在析构函数中有大量逻辑，后面讲解
    std::ostream& stream(); // 返回stream_
    void SendToLog(); // 主要函数
    void Init(const char* file, int line, LogSeverity severity,
            void (LogMessage::*send_method)());
    void Flush();
    LogMessageData* allocated_; // LogMessageData，主要的结构体
    LogMessageData* data_;
};
```

LogMessage的构造函数如下：

```cpp
LogMessage::LogMessage(const char* file, int line)
    : allocated_(NULL) {
  Init(file, line, GLOG_INFO, &LogMessage::SendToLog);
}
```

Init函数精简版如下：

```cpp
void LogMessage::Init(const char* file,
                      int line,
                      LogSeverity severity,
                      void (LogMessage::*send_method)()) {
    allocated_ = NULL;
    // 主要看这里，glog比较高效的地方就在于它没有频繁的申请内存，而是使用线程thread_local特性，为每个线程创建一块私有内存，同一线程频繁的使用这块内存构造LogMessageData的对象，因为这个LogMessageData对象是临时对象，每次都会立刻析构，这样下一个对象可以重复在这块内存地址来构造对象
    if (thread_data_available) {
        thread_data_available = false;
    // 这里会将LogMessageData对象构造在内存对齐的地址上，HAVE_ALIGNED_STORAGE这个宏在C++11后会有效，C++11前就会走到这个#else分支，需要自己进行手动内存对齐，为什么HAVE_ALIGNED_STORAGE宏下面的代码可以直接构造对象，下面会介绍
    #ifdef HAVE_ALIGNED_STORAGE
        data_ = new (&thread_msg_data) LogMessageData;
    #else
        const uintptr_t kAlign = sizeof(void*) - 1;

        char* align_ptr =
            reinterpret_cast<char*>(reinterpret_cast<uintptr_t>(thread_msg_data + kAlign) & ~kAlign);
        data_ = new (align_ptr) LogMessageData;
        assert(reinterpret_cast<uintptr_t>(align_ptr) % sizeof(void*) == 0);
    #endif
    } else {
        allocated_ = new LogMessageData();
        data_ = allocated_;
    }
    // 下面这一堆代码就是在每一行log具体信息前加上一些信息前缀，时间戳线程号文件名行数等等
    stream().fill('0');
    data_->preserved_errno_ = errno;
    data_->severity_ = severity;
    data_->line_ = line;
    data_->send_method_ = send_method;
    data_->sink_ = NULL;
    data_->outvec_ = NULL;
    WallTime now = WallTime_Now();
    data_->timestamp_ = static_cast<time_t>(now);
    localtime_r(&data_->timestamp_, &data_->tm_time_);
    int usecs = static_cast<int>((now - data_->timestamp_) * 1000000);

    data_->num_chars_to_log_ = 0;
    data_->num_chars_to_syslog_ = 0;
    data_->basename_ = const_basename(file);
    data_->fullname_ = file;
    data_->has_been_flushed_ = false;

    // If specified, prepend a prefix to each line.  For example:
    //    I1018 160715 f5d4fbb0 logging.cc:1153]
    //    (log level, GMT month, date, time, thread_id, file basename, line)
    // We exclude the thread_id for the default thread.
    if (FLAGS_log_prefix && (line != kNoLogPrefix)) {
    stream() << LogSeverityNames[severity][0]
                << setw(2) << 1+data_->tm_time_.tm_mon
                << setw(2) << data_->tm_time_.tm_mday
                << ' '
                << setw(2) << data_->tm_time_.tm_hour  << ':'
                << setw(2) << data_->tm_time_.tm_min   << ':'
                << setw(2) << data_->tm_time_.tm_sec   << "."
                << setw(6) << usecs
                << ' '
                << setfill(' ') << setw(5)
                << static_cast<unsigned int>(GetTID()) << setfill('0')
                << ' '
                << data_->basename_ << ':' << data_->line_ << "] ";
    }
    data_->num_prefix_chars_ = data_->stream_.pcount();
}
```

#### 为什么HAVE_ALIGNED_STORAGE宏下面的代码可以直接构造对象呢？

看这里

```cpp
#ifdef HAVE_ALIGNED_STORAGE
static GLOG_THREAD_LOCAL_STORAGE
    std::aligned_storage<sizeof(LogMessage::LogMessageData),
                         alignof(LogMessage::LogMessageData)>::type thread_msg_data;
#else
static GLOG_THREAD_LOCAL_STORAGE
    char thread_msg_data[sizeof(void*) + sizeof(LogMessage::LogMessageData)];
#endif  // HAVE_ALIGNED_STORAGE
```

是因为C++11后可以直接使用std::aligned_storage来申请对齐的内存地址。

#### LOG(INFO)<<”xxx”<<”yyy”中，glog是如何收集到xxx和yyy数据的呢？

我们上面看到LogMessage的构造函数中会构造核心的LogMessageData对象，看如下LogMessageData的结构体和构造函数中的注释

```cpp
struct LogMessage::LogMessageData  {
    LogMessageData();

    int preserved_errno_;      // preserved errno
    // Buffer space; contains complete message text.
    char message_text_[LogMessage::kMaxLogMessageLen+1]; // 这里存放log的具体信息
    LogStream stream_; // std::ostream
    char severity_;      // What level is this LogMessage logged at?
    int line_;                 // line number where logging call is.
    void (LogMessage::*send_method_)();  // Call this in destructor to send
    union {  // At most one of these is used: union to keep the size low.
        LogSink* sink_;             // NULL or sink to send message to
        std::vector<std::string>* outvec_; // NULL or vector to push message onto
        std::string* message_;             // NULL or string to write message into
    };
    time_t timestamp_;            // Time of creation of LogMessage
    struct ::tm tm_time_;         // Time of creation of LogMessage
    size_t num_prefix_chars_;     // # of chars of prefix in this message
    size_t num_chars_to_log_;     // # of chars of msg to send to log
    size_t num_chars_to_syslog_;  // # of chars of msg to send to syslog
    const char* basename_;        // basename of file that called LOG
    const char* fullname_;        // fullname of file that called LOG
    bool has_been_flushed_;       // false => data has not been flushed
    bool first_fatal_;            // true => this was first fatal msg

    private:
    LogMessageData(const LogMessageData&);
    void operator=(const LogMessageData&);
};

LogMessage::LogMessageData::LogMessageData()
  : stream_(message_text_, LogMessage::kMaxLogMessageLen, 0) {
}
// 这里就不列出LogStream的构造函数啦，LogStream内部会使用到streambuf，而这个streambuf的地址就是上面LogMessageData构造函数中传入的message_text_地址，这样stream_接收到的所有的数据都会存到这个message_text_中.
```

#### 将stream接收到的数据都存到了message*text*中，那数据是如何写到文件中去的并且是线程安全的呢？

这些逻辑都是在LogMessage的析构函数中，看精简代码：

```cpp
LogMessage::~LogMessage() {
  Flush();// 主要功能在Flush函数中
  if (data_ == static_cast<void*>(&thread_msg_data)) {
    data_->~LogMessageData();
    thread_data_available = true;
  }
  else {
    delete allocated_;
  }
}
```

Flush函数精简：

```cpp
void LogMessage::Flush() {
    {
    MutexLock l(&log_mutex);// 注意，这里使用了全局log锁，所以写操作等等都是线程安全的，即没有出现日志错乱现象
    (this->*(data_->send_method_))(); // 这里的send_method_可以从上面Init函数中看见其实就是使用SendToLog()
    ++num_messages_[static_cast<int>(data_->severity_)];
    }
}
```

SendToLog函数精简：

```cpp
void LogMessage::SendToLog() {
    // LogDestination类中有好多静态函数，LogToAllLogfiles和MaybeLogToStderr是两个常用的函数，一个写到文件中，一个写到输出流中, data->severity_这里指的是log级别，即INFO、WARNING或者ERROR等
    LogDestination::LogToAllLogfiles(data_->severity_, data_->timestamp_,
                                    data_->message_text_,
                                    data_->num_chars_to_log_);

    LogDestination::MaybeLogToStderr(data_->severity_, data_->message_text_,
                                     data_->num_chars_to_log_);
}
```

LogDestination::LogToAllLogfiles函数精简：

```cpp
inline void LogDestination::LogToAllLogfiles(LogSeverity severity,
                                             time_t timestamp,
                                             const char* message,
                                             size_t len) {

  if ( FLAGS_logtostderr ) {           // global flag: never log to file
    ColoredWriteToStderr(severity, message, len);
  } else {
    // 看这里，glog会把高级别的log也会写在低级别的log文件中，即ERROR的log也会出现在INFO的log文件中
    for (int i = severity; i >= 0; --i)
      LogDestination::MaybeLogToLogfile(i, timestamp, message, len);
  }
}
```

LogDestination::MaybeLogToLogfile函数精简：

```cpp
inline void LogDestination::MaybeLogToLogfile(LogSeverity severity,
                                              time_t timestamp,
                          const char* message,
                          size_t len) {
  const bool should_flush = severity > FLAGS_logbuflevel;
  // 这里每个log level都会有一个LogDestination的对象指针
  LogDestination* destination = log_destination(severity);
  // Write就是写文件操作
  destination->logger_->Write(should_flush, timestamp, message, len);
}
```

#### glog中日志写到文件中是什么时候Flush到磁盘中的呢？

看下destination->logger_->Write函数精简实现:

```cpp
void LogFileObject::Write(bool force_flush,
                          time_t timestamp,
                          const char* message,
                          int message_len) {
  MutexLock l(&lock_);

  // We don't log if the base_name_ is "" (which means "don't write")
  if (base_filename_selected_ && base_filename_.empty()) {
    return;
  }

  // If there's no destination file, make one before outputting
  // 没有创建文件那就创建一个文件，文件名中包含时间进程号等信息
  if (file_ == NULL) {
    // Try to rollover the log file every 32 log messages.  The only time
    // this could matter would be when we have trouble creating the log
    // file.  If that happens, we'll lose lots of log messages, of course!
    if (++rollover_attempt_ != kRolloverAttemptFrequency) return;
    rollover_attempt_ = 0;

    struct ::tm tm_time;
    localtime_r(&timestamp, &tm_time);

    // The logfile's filename will have the date/time & pid in it
    ostringstream time_pid_stream;
    time_pid_stream.fill('0');
    time_pid_stream << 1900+tm_time.tm_year
                    << setw(2) << 1+tm_time.tm_mon
                    << setw(2) << tm_time.tm_mday
                    << '-'
                    << setw(2) << tm_time.tm_hour
                    << setw(2) << tm_time.tm_min
                    << setw(2) << tm_time.tm_sec
                    << '.'
                    << GetMainThreadPid();
    const string& time_pid_string = time_pid_stream.str();

    if (!CreateLogfile(time_pid_string)) {
      perror("Could not create log file");
      fprintf(stderr, "COULD NOT CREATE LOGFILE '%s'!\n",
              time_pid_string.c_str());
      return;
    }
    // 在文件的最开头写入log创建的相应信息
    // Write a header message into the log file
    ostringstream file_header_stream;
    file_header_stream.fill('0');
    file_header_stream << "Log file created at: "
                       << 1900+tm_time.tm_year << '/'
                       << setw(2) << 1+tm_time.tm_mon << '/'
                       << setw(2) << tm_time.tm_mday
                       << ' '
                       << setw(2) << tm_time.tm_hour << ':'
                       << setw(2) << tm_time.tm_min << ':'
                       << setw(2) << tm_time.tm_sec << '\n'
                       << "Running on machine: "
                       << LogDestination::hostname() << '\n'
                       << "Log line format: [IWEF]mmdd hh:mm:ss.uuuuuu "
                       << "threadid file:line] msg" << '\n';
    const string& file_header_string = file_header_stream.str();

    const int header_len = file_header_string.size();
    fwrite(file_header_string.data(), 1, header_len, file_);
    file_length_ += header_len;
    bytes_since_flush_ += header_len;
  }

  // Write to LOG file
  if ( !stop_writing ) {
    // fwrite() doesn't return an error when the disk is full, for
    // messages that are less than 4096 bytes. When the disk is full,
    // it returns the message length for messages that are less than
    // 4096 bytes. fwrite() returns 4096 for message lengths that are
    // greater than 4096, thereby indicating an error.
    errno = 0;
    // 开始把数据写到log文件中
    fwrite(message, 1, message_len, file_);
    if ( FLAGS_stop_logging_if_full_disk &&
         errno == ENOSPC ) {  // disk full, stop writing to disk
      stop_writing = true;  // until the disk is
      return;
    } else {
      file_length_ += message_len;
      bytes_since_flush_ += message_len;
    }
  } else {
    if ( CycleClock_Now() >= next_flush_time_ )
      stop_writing = false;  // check to see if disk has free space.
    return;  // no need to flush
  }

  // See important msgs *now*.  Also, flush logs at least every 10^6 chars,
  // or every "FLAGS_logbufsecs" seconds.
  // 控制什么时候把文件内容Flush到磁盘中
  if ( force_flush ||
       (bytes_since_flush_ >= 1000000) ||
       (CycleClock_Now() >= next_flush_time_) ) {
    FlushUnlocked();
  }
}

// 这里是FlushUnlocked函数
void LogFileObject::FlushUnlocked(){
  if (file_ != NULL) {
    fflush(file_);
    bytes_since_flush_ = 0;
  }
  // Figure out when we are due for another flush.
  const int64 next = (FLAGS_logbufsecs
                      * static_cast<int64>(1000000));  // in usec
  next_flush_time_ = CycleClock_Now() + UsecToCycles(next);
}
```

从这里可以看到glog是根据内容长度和时间来控制是否Flush到磁盘中，有1000000字节没有被Flush就执行一次Flush操作，或者FLAGS_logbufsecs秒没有Flush也会执行一次Flush，这个FLAGS_logbufsecs默认是30，可以设置。

### 总结

glog通过重写std::ostream、std::stream_buf以及thread_local等技术实现了精简高效的日志输出功能，每行log语句都会创建一个LogMessage对象，通过LogMessage对象内部的stream收集需要输出的消息存到对象内部的栈内存message_text中，一行语句结束，LogMessage对象析构，析构时会把message_text中的数据写到文件和控制台里，写文件操作会每隔1000000字节或者每隔FLAGS_logbufsecs秒Flush到磁盘中。

### 如何设计一个高效的log模块？

从上述源码分析可知，glog是每次log都会执行写操作，并且写操作是等锁的，写文件本身就比较耗时，再加上等锁的时间，会阻塞当前写log的业务工作线程，所以glog在多线程中会导致应用程序性能不是特别好，所以如果能够减少阻塞工作线程的时间就可以设计出一个高效的log模块，将日志的写文件操作放在单独的线程中，参考陈硕muduo代码，log架构其实有很大改良空间。

#### muduo async log日志逻辑

muduo的异步日志是将写日志的操作放在单独的日志线程中，这里分为多个应用线程和专用的日志线程，同时有多块缓存，大概可以分为两大块缓存池，有收集日志的缓存池和专用于写日志的缓存池，收集日志的缓存池(buffer_vector)中有两块buffer，称为current_buffer和next_buffer，多个应用线程的日志都会写到current_buffer(buffer_mutex)中，当current_buffer满的时候，将current_buffer的指针存到buffer_vector中，同时current_buffer的指针指向next_buffer，这样应用线程可以继续写日志到current_buffer中，current_buffer的指针存到buffer_vector后，会通知到日志线程，这里加上锁来控制current_buffer(buffer_mutex)，写日志的缓存池叫write_buffer_vector，里面也有两块缓存newBuffer1和newBuffer2，这时再将current_buffer的指针存入buffer_vector中，这时buffer_vector中有两块缓存的指针，之后将buffer_vector和write_buffer_vector交换，buffer_vector就是空，同时current_buffer指针指向newBuffer1,next_buffer指针指向newBuffer2，释放锁(buffer_mutex)，这时log线程可以进行写操作，write_buffer_vector的大小为2，将里面的两块内存都写到文件中，同时newBuffer1和newBuffer2指针分别指向这两块内存，这样下次再执行交换操作时候write_buffer_vector和newBuffer1和newBuffer2都是空，一直循环执行这类操作，log一般都是写文件时候时间比较长，将数据memcpy到buffer中耗时较少，这样可以大幅减少等锁的时间，提升log的性能。
代码如下：

```cpp
void AsyncLogging::append(const char* logline, int len)
{
  muduo::MutexLockGuard lock(mutex_);
  if (currentBuffer_->avail() > len)
  {
    currentBuffer_->append(logline, len);
  }
  else
  {
    buffers_.push_back(std::move(currentBuffer_));

    if (nextBuffer_)
    {
      currentBuffer_ = std::move(nextBuffer_);
    }
    else
    {
      currentBuffer_.reset(new Buffer); // Rarely happens
    }
    currentBuffer_->append(logline, len);
    cond_.notify();
  }
}

void AsyncLogging::threadFunc()
{
  assert(running_ == true);
  latch_.countDown();
  LogFile output(basename_, rollSize_, false);
  BufferPtr newBuffer1(new Buffer);
  BufferPtr newBuffer2(new Buffer);
  newBuffer1->bzero();
  newBuffer2->bzero();
  BufferVector buffersToWrite;
  buffersToWrite.reserve(16);
  while (running_)
  {
    assert(newBuffer1 && newBuffer1->length() == 0);
    assert(newBuffer2 && newBuffer2->length() == 0);
    assert(buffersToWrite.empty());

    {
      muduo::MutexLockGuard lock(mutex_);
      if (buffers_.empty())  // unusual usage!
      {
        cond_.waitForSeconds(flushInterval_);
      }
      buffers_.push_back(std::move(currentBuffer_));
      currentBuffer_ = std::move(newBuffer1);
      buffersToWrite.swap(buffers_);
      if (!nextBuffer_)
      {
        nextBuffer_ = std::move(newBuffer2);
      }
    }

    assert(!buffersToWrite.empty());

    if (buffersToWrite.size() > 2)
    {
      char buf[256];
      snprintf(buf, sizeof buf, "Dropped log messages at %s, %zd larger buffers\n",
               Timestamp::now().toFormattedString().c_str(),
               buffersToWrite.size()-2);
      fputs(buf, stderr);
      output.append(buf, static_cast<int>(strlen(buf)));
      buffersToWrite.erase(buffersToWrite.begin()+2, buffersToWrite.end());
    }

    for (const auto& buffer : buffersToWrite)
    {
      // FIXME: use unbuffered stdio FILE ? or use ::writev ?
      output.append(buffer->data(), buffer->length());
    }

    if (buffersToWrite.size() > 2)
    {
      // drop non-bzero-ed buffers, avoid trashing
      buffersToWrite.resize(2);
    }

    if (!newBuffer1)
    {
      assert(!buffersToWrite.empty());
      newBuffer1 = std::move(buffersToWrite.back());
      buffersToWrite.pop_back();
      newBuffer1->reset();
    }

    if (!newBuffer2)
    {
      assert(!buffersToWrite.empty());
      newBuffer2 = std::move(buffersToWrite.back());
      buffersToWrite.pop_back();
      newBuffer2->reset();
    }

    buffersToWrite.clear();
    output.flush();
  }
  output.flush();
}
```

### 扩展

C++当前有两个比较好用的log第三方库easylogging++和spdlog，具体使用读者可以网上找相关资料，个人推荐spdlog，后续会对spdlog进行原理分析。