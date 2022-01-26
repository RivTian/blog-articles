## 单文件上传

1. 导入 fileupload 和 io 库

   ```xml
   <dependency>
       <groupId>commons-fileupload</groupId>
       <artifactId>commons-fileupload</artifactId>
       <version>1.3.1</version>
   </dependency>
   
   <dependency>
       <groupId>commons-io</groupId>
       <artifactId>commons-io</artifactId>
       <version>2.3</version>
   </dependency>
   ```

2. .在`spring-mvc.xml`中配置文件上传解析器

   ```xml
   <!--配置文件上传解析器-->
   <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
       <!--上传文件总大小-->
       <property name="maxUploadSize" value="5242800"/>
       <!--上传单个文件的大小-->
       <property name="maxUploadSizePerFile" value="5242800"/>
       <!--上传文件的编码类型-->
       <property name="defaultEncoding" value="UTF-8"/>
   </bean>
   ```

3. 实现文件上传代码

   ```java
   // 实现文件上传代码
   @RequestMapping("/quick22")
   @ResponseBody
   public void save21(String username, MultipartFile uploadFile) throws IOException {
       System.out.println("username:" + username);
       System.out.println("uploadFile:" + uploadFile.getOriginalFilename());
       File path = new File("D:\\File");
       if (!path.exists()) {
           path.mkdirs();
       }
       File file = new File(path, uploadFile.getOriginalFilename());
       uploadFile.transferTo(file);
   }
   ```

4. 多文件上传

   ```java
   // 多文件上传
   @RequestMapping("/quick23")
   @ResponseBody
   public void save23(String username, MultipartFile uploadFile,MultipartFile uploadFile2) throws IOException {
       System.out.println("username:" + username);
       System.out.println("uploadFile:" + uploadFile.getOriginalFilename());
       System.out.println("uploadFile2:" + uploadFile2.getOriginalFilename());
   }
   ```

5. 上传文件列表

   ```java
   // 上传文件列表
   @RequestMapping("/quick24")
   @ResponseBody
   public void save24(String username, MultipartFile[] uploadFile) throws IOException {4System.out.println("username:" + username);
       for (MultipartFile multipartFile : uploadFile) {
           System.out.println("uploadFile:" + multipartFile.getOriginalFilename());
           multipartFile.transferTo(new File("D:\\File", multipartFile.getOriginalFilename()));
       }
   }
   ```

   