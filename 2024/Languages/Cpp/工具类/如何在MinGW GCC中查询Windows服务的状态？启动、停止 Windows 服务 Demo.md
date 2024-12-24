#Cpp #Win 


## 使用 MinGW GCC 完成查询 Windows 服务的状态：

```cpp
#include <windows.h>
#include <stdio.h>

// 查询服务状态的函数
void QueryServiceStatus(const char* serviceName)
{
    SC_HANDLE scm, service;
    SERVICE_STATUS status;

    // 打开服务控制管理器
    scm = OpenSCManager(NULL, NULL, SC_MANAGER_CONNECT);
    if (scm == NULL)
    {
        printf("OpenSCManager failed, error=%d\n", GetLastError());
        return;
    }

    // 打开指定的服务
    service = OpenService(scm, serviceName, SERVICE_QUERY_STATUS);
    if (service == NULL)
    {
        printf("OpenService failed, error=%d\n", GetLastError());
        CloseServiceHandle(scm);
        return;
    }

    // 查询服务状态
    if (!QueryServiceStatus(service, &status))
    {
        printf("QueryServiceStatus failed, error=%d\n", GetLastError());
    }
    else
    {
        // 输出服务状态信息
        printf("Service %s status:\n", serviceName);
        printf("  State: ");
        switch (status.dwCurrentState)
        {
        case SERVICE_STOPPED:
            printf("Stopped\n");
            break;
        case SERVICE_START_PENDING:
            printf("Start Pending\n");
            break;
        case SERVICE_STOP_PENDING:
            printf("Stop Pending\n");
            break;
        case SERVICE_RUNNING:
            printf("Running\n");
            break;
        // 其他状态的处理
        }
    }

    // 关闭服务句柄和服务控制管理器句柄
    CloseServiceHandle(service);
    CloseServiceHandle(scm);
}

int main()
{
    // 调用查询服务状态的函数
    QueryServiceStatus("MyService");

    return 0;
}
```

在这个示例中，定义了一个名为`QueryServiceStatus`的函数，该函数接受服务名称作为参数，并使用Windows API中的函数来查询该服务的状态。在`main`函数中，调用了`QueryServiceStatus`函数来查询名为"MyService"的服务的状态。

## 如何在MinGW GCC中停止和启动Windows服务？

### 停止Windows服务的示例代码：

```cpp
#include <windows.h>
#include <stdio.h>

// 停止服务的函数
void StopService(const char* serviceName)
{
    SC_HANDLE scm, service;
    SERVICE_STATUS status;

    // 打开服务控制管理器
    scm = OpenSCManager(NULL, NULL, SC_MANAGER_CONNECT);
    if (scm == NULL)
    {
        printf("OpenSCManager failed, error=%d\n", GetLastError());
        return;
    }

    // 打开指定的服务
    service = OpenService(scm, serviceName, SERVICE_STOP | SERVICE_QUERY_STATUS);
    if (service == NULL)
    {
        printf("OpenService failed, error=%d\n", GetLastError());
        CloseServiceHandle(scm);
        return;
    }

    // 查询服务状态
    if (!QueryServiceStatus(service, &status))
    {
        printf("QueryServiceStatus failed, error=%d\n", GetLastError());
    }
    else
    {
        // 如果服务正在运行，则停止服务
        if (status.dwCurrentState == SERVICE_RUNNING)
        {
            if (!ControlService(service, SERVICE_CONTROL_STOP, &status))
            {
                printf("ControlService failed, error=%d\n", GetLastError());
            }
            else
            {
                printf("Service %s stopped successfully\n", serviceName);
            }
        }
        else
        {
            printf("Service %s is not running\n", serviceName);
        }
    }

    // 关闭服务句柄和服务控制管理器句柄
    CloseServiceHandle(service);
    CloseServiceHandle(scm);
}

int main()
{
    // 调用停止服务的函数
    StopService("MyService");

    return 0;
}
```

### 启动Windows服务的示例代码：

```cpp
#include <windows.h>
#include <stdio.h>

// 启动服务的函数
void StartService(const char* serviceName)
{
    SC_HANDLE scm, service;
    SERVICE_STATUS status;

    // 打开服务控制管理器
    scm = OpenSCManager(NULL, NULL, SC_MANAGER_CONNECT);
    if (scm == NULL)
    {
        printf("OpenSCManager failed, error=%d\n", GetLastError());
        return;
    }

    // 打开指定的服务
    service = OpenService(scm, serviceName, SERVICE_START | SERVICE_QUERY_STATUS);
    if (service == NULL)
    {
        printf("OpenService failed, error=%d\n", GetLastError());
        CloseServiceHandle(scm);
        return;
    }

    // 查询服务状态
    if (!QueryServiceStatus(service, &status))
    {
        printf("QueryServiceStatus failed, error=%d\n", GetLastError());
    }
    else
    {
        // 如果服务未在运行，则启动服务
        if (status.dwCurrentState != SERVICE_RUNNING)
        {
            if (!StartService(service, 0, NULL))
            {
                printf("StartService failed, error=%d\n", GetLastError());
            }
            else
            {
                printf("Service %s started successfully\n", serviceName);
            }
        }
        else
        {
            printf("Service %s is already running\n", serviceName);
        }
    }

    // 关闭服务句柄和服务控制管理器句柄
    CloseServiceHandle(service);
    CloseServiceHandle(scm);
}

int main()
{
    // 调用启动服务的函数
    StartService("MyService");

    return 0;
}
```

定义了一个停止服务的函数`StopService`和一个启动服务的函数`StartService`。这些函数接受服务名称作为参数，并使用Windows API中的函数来停止或启动相应的服务。