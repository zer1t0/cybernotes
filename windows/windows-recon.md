# Windows Recon


## Powershell history

We can find the Powershell history for an user in:

```
%APPDATA%\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
```

Which is usually at:
```
C:\Users\<user>\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
```

## Services

The most common way to list services is to use the `sc` program:
```
PS C:\> sc.exe queryex
..stripped..
SERVICE_NAME: WpnUserService_3acc86
DISPLAY_NAME: Windows Push Notifications User Service_3acc86
        TYPE               : f0   ERROR
        STATE              : 4  RUNNING
                                (STOPPABLE, NOT_PAUSABLE, ACCEPTS_PRESHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x0
        PID                : 2520
        FLAGS              :
```

> **Note**: I suggest invoke sc with `sc.exe` to avoid collisions with
> the alias sc of the Set-Content cmdlet in Powershell

Query one service executable path:
```
PS C:\Users\user> sc.exe qc UnistoreSvc
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: UnistoreSvc
        TYPE               : 60  USER_SHARE_PROCESS TEMPLATE
        START_TYPE         : 3   DEMAND_START
        ERROR_CONTROL      : 0   IGNORE
        BINARY_PATH_NAME   : C:\Windows\System32\svchost.exe -k UnistackSvcGroup
        LOAD_ORDER_GROUP   :
        TAG                : 0
        DISPLAY_NAME       : User Data Storage
        DEPENDENCIES       :
        SERVICE_START_NAME : LocalSystem
```

Services information is stored under the 
[HKLM\SYSTEM\CurrentControlSet\Services Registry Tree](https://learn.microsoft.com/en-us/windows-hardware/drivers/install/hklm-system-currentcontrolset-services-registry-tree), so we can
list them with the `reg` command too:
```
PS C:\> reg query HKLM\SYSTEM\CurrentControlSet\Services
..stripped..
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\1394ohci
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\3ware
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\AarSvc
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\AarSvc_3acc86
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\ACPI
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\AcpiDev
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\acpiex
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\acpipagr
..stripped..
```


Another example to get all the services executable paths:
```
PS C:\> reg query HKLM\SYSTEM\CurrentControlSet\Services /s /f ImagePath
..stripped..
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\XboxGipSvc
    ImagePath    REG_EXPAND_SZ    %SystemRoot%\system32\svchost.exe -k netsvcs -p

HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\XboxNetApiSvc
    ImagePath    REG_EXPAND_SZ    %SystemRoot%\system32\svchost.exe -k netsvcs -p

HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\xinputhid
    ImagePath    REG_EXPAND_SZ    \SystemRoot\System32\drivers\xinputhid.sys

End of search: 649 match(es) found.
```


You may notice that many services are run by [svchost.exe](https://en.wikipedia.org/wiki/Svchost.exe), those
services are implemented as dlls whose path is indicated under the
`Parameters` key. For example:

```
PS C:\> reg query HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\XboxNetApiSvc\Parameters

HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\XboxNetApiSvc\Parameters
    ServiceDll    REG_EXPAND_SZ    %SystemRoot%\system32\XboxNetApiSvc.dll
    ServiceDllUnloadOnStop    REG_DWORD    0x1
```
