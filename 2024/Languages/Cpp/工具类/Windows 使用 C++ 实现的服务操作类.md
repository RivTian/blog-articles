
#Cpp 

```cpp
#pragma once
#include <windows.h>
#include <string>
// #include <iostream>

class ServiceManage
{
public:
	ServiceManage() = default;
	~ServiceManage() = default;

	/**
	 * \brief Start service based on service name
	 * \param strServiceName
	 * \return
	 */
	inline bool Start_Server(const std::string& strServiceName)
	{
		bool bResult = false;
		if (strServiceName.empty())
		{
			return bResult;
		}
		SC_HANDLE sc_Manager = ::OpenSCManagerA(NULL, NULL, GENERIC_EXECUTE);
		if (sc_Manager)
		{
			SC_HANDLE sc_service = ::OpenServiceA(sc_Manager, strServiceName.c_str(), SERVICE_ALL_ACCESS);
			if (sc_service)
			{
				SERVICE_STATUS_PROCESS service_status;
				ZeroMemory(&service_status, sizeof(SERVICE_STATUS_PROCESS));
				DWORD dwpcbBytesNeeded = sizeof(SERVICE_STATUS_PROCESS);
				if (::QueryServiceStatusEx(sc_service, SC_STATUS_PROCESS_INFO,
					(LPBYTE)&service_status,
					dwpcbBytesNeeded,
					&dwpcbBytesNeeded))
				{
					if (service_status.dwCurrentState == SERVICE_STOPPED)
					{
						if (!::StartService(sc_service, NULL, NULL))
						{
							::CloseServiceHandle(sc_service);
							::CloseServiceHandle(sc_Manager);
							return bResult;
						}
						while (::QueryServiceStatusEx(sc_service, SC_STATUS_PROCESS_INFO,
							(LPBYTE)&service_status,
							dwpcbBytesNeeded,
							&dwpcbBytesNeeded))
						{
							Sleep(service_status.dwWaitHint);
							if (service_status.dwCurrentState == SERVICE_RUNNING)
							{
								bResult = true;
								break;
							}
						}
					}
				}
				::CloseServiceHandle(sc_service);
			}
			::CloseServiceHandle(sc_Manager);
		}
		return bResult;
	}

	/**
	 * \brief Stop service based on service name
	 * \param strServiceName
	 * \return
	 */
	inline bool Stop_Server(const std::string& strServiceName)
	{
		bool bResult = false;
		if (strServiceName.empty())
		{
			return bResult;
		}
		SC_HANDLE sc_Manager = ::OpenSCManagerA(NULL, NULL, GENERIC_EXECUTE);
		if (sc_Manager)
		{
			SC_HANDLE sc_service = ::OpenServiceA(sc_Manager, strServiceName.c_str(), SERVICE_ALL_ACCESS);
			if (sc_service)
			{
				SERVICE_STATUS_PROCESS service_status;
				ZeroMemory(&service_status, sizeof(SERVICE_STATUS_PROCESS));
				DWORD dwpcbBytesNeeded = sizeof(SERVICE_STATUS_PROCESS);
				if (::QueryServiceStatusEx(sc_service, SC_STATUS_PROCESS_INFO,
					(LPBYTE)&service_status,
					dwpcbBytesNeeded,
					&dwpcbBytesNeeded))
				{
					SERVICE_CONTROL_STATUS_REASON_PARAMSA service_control_status;
					DWORD dwerror = NULL;
					ZeroMemory(&service_control_status, sizeof(SERVICE_CONTROL_STATUS_REASON_PARAMSA));
					if (service_status.dwCurrentState == SERVICE_RUNNING)
					{
						service_control_status.dwReason = SERVICE_STOP_REASON_FLAG_PLANNED | SERVICE_STOP_REASON_MAJOR_APPLICATION | SERVICE_STOP_REASON_MINOR_NONE;;
						if (!::ControlServiceExA(sc_service, SERVICE_CONTROL_STOP, SERVICE_CONTROL_STATUS_REASON_INFO, &service_control_status))
						{
							dwerror = ::GetLastError();
							::CloseServiceHandle(sc_service);
							::CloseServiceHandle(sc_Manager);
							return bResult;
						}
						while (::QueryServiceStatusEx(sc_service, SC_STATUS_PROCESS_INFO,
							(LPBYTE)&service_status,
							dwpcbBytesNeeded,
							&dwpcbBytesNeeded))
						{
							Sleep(service_status.dwWaitHint);
							if (service_status.dwCurrentState == SERVICE_STOPPED)
							{

								bResult = true;
								break;
							}
						}
					}
				}
				::CloseServiceHandle(sc_service);
			}
			::CloseServiceHandle(sc_Manager);
		}
		return bResult;
	}

	inline bool Exist_Service(const std::string& strServiceName)
	{
		bool bResult = false;
		const int MAX_SERVICE_SIZE = 1024 * 64;
		// const int MAX_QUERY_SIZE   = 1024 * 8;
		if (strServiceName.empty())
			return bResult;
        SC_HANDLE SCMan = OpenSCManager(NULL, NULL, SC_MANAGER_ALL_ACCESS);
        if(SCMan == NULL) {
            // std::cout << "OpenSCManager failed." << std::endl;
            return bResult;
        }
        LPENUM_SERVICE_STATUS service_status;
        DWORD cbBytesNeeded = NULL;
        DWORD ServicesReturned = NULL;
        DWORD ResumeHandle = NULL;

        service_status = (LPENUM_SERVICE_STATUS)LocalAlloc(LPTR, MAX_SERVICE_SIZE);


        BOOL ESS = EnumServicesStatus(SCMan,
            SERVICE_WIN32,
            SERVICE_STATE_ALL,
            (LPENUM_SERVICE_STATUS)service_status,
            MAX_SERVICE_SIZE,
            &cbBytesNeeded,
            &ServicesReturned,
            &ResumeHandle);
        if(ESS == NULL) {
            // std::cout << "EnumServicesStatus Failed." << std::endl;
            return bResult;;
        }

        for(int i = 0; i < static_cast<int>(ServicesReturned); i++) {
            if (0 != strcmp(service_status[i].lpDisplayName, strServiceName.c_str())) continue;
            // std::cout << service_status[i].lpDisplayName << " ";
        	bResult = true;
        }
        CloseServiceHandle(SCMan);
		return bResult;
	}

	/**
	 * \brief Query service status based on service name
	 * \param strServiceName
	 * \return
	 */
	inline DWORD Query_Server_Status(const std::string& strServiceName)
	{
		DWORD nResult = 0;
		if (strServiceName.empty())
		{
			return nResult;
		}
		SC_HANDLE sc_Manager = ::OpenSCManagerA(NULL, NULL, GENERIC_EXECUTE);
		if (sc_Manager)
		{
			SC_HANDLE sc_service = ::OpenServiceA(sc_Manager, strServiceName.c_str(), SERVICE_ALL_ACCESS);
			if (sc_service)
			{
				SERVICE_STATUS_PROCESS service_status;
				ZeroMemory(&service_status, sizeof(SERVICE_STATUS_PROCESS));
				DWORD dwpcbBytesNeeded = sizeof(SERVICE_STATUS_PROCESS);
				if (::QueryServiceStatusEx(sc_service, SC_STATUS_PROCESS_INFO,
					(LPBYTE)&service_status,
					dwpcbBytesNeeded,
					&dwpcbBytesNeeded))
				{
					nResult = service_status.dwCurrentState;
				}
				::CloseServiceHandle(sc_service);
			}
			::CloseServiceHandle(sc_Manager);
		}
		return nResult;
	}
private:

};
```