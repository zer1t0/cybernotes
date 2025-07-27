# Windows Registry

The **windows registry is like an alternative file that stores the
configuration of the windows components**. It is composed by
keys, values and data.

## Hives

The [hives](https://en.wikipedia.org/wiki/Windows_Registry#Hives) are the files where the registry is stored. We can find
many of the in the `%SystemRoot%\System32\config\` (aka
`C:\Windows\System32\config\`).


## SAM
The SAM hive is stored in `%SystemRoot%\System32\config\SAM` and maps
to `HKLM\SAM`. It contains the information about the local users.

```
PS C:\> .\PsExec.exe -s powershell.exe
PS C:\Windows\system32> reg query HKLM\SAM\SAM

HKEY_LOCAL_MACHINE\SAM\SAM
    C    REG_BINARY    0900010000..stripped..20020000
    ServerDomainUpdates    REG_BINARY    FEFF0F

HKEY_LOCAL_MACHINE\SAM\SAM\Domains
HKEY_LOCAL_MACHINE\SAM\SAM\LastSkuUpgrade
HKEY_LOCAL_MACHINE\SAM\SAM\RXACT
```

## Security
The Security hive is stored in `%SystemRoot%\System32\config\SECURITY`
and maps to `HKLM\SECURITY`. It stores several security information like the
[LSA Secrets](https://passcape.com/index.php?section=docsys&cmd=details&id=23) that contains the machine domain credentials or the
services passwords.
  
  
```
PS C:\> .\PsExec.exe -s powershell.exe
PS C:\Windows\system32> reg query HKLM\SECURITY

HKEY_LOCAL_MACHINE\SECURITY\Cache
HKEY_LOCAL_MACHINE\SECURITY\Policy
HKEY_LOCAL_MACHINE\SECURITY\RXACT
HKEY_LOCAL_MACHINE\SECURITY\SAM
```
  
## Software

The Software hive is stored in `%SystemRoot%\System32\config\SOFTWARE`
and maps to `HKLM\SOFTWARE`. It contains the configuration of the 
programs at system level.

```
PS C:\> reg query HKLM\SOFTWARE

HKEY_LOCAL_MACHINE\SOFTWARE\Classes
HKEY_LOCAL_MACHINE\SOFTWARE\Clients
HKEY_LOCAL_MACHINE\SOFTWARE\CVSM
HKEY_LOCAL_MACHINE\SOFTWARE\DefaultUserEnvironment
HKEY_LOCAL_MACHINE\SOFTWARE\dotnet
HKEY_LOCAL_MACHINE\SOFTWARE\Google
HKEY_LOCAL_MACHINE\SOFTWARE\Intel
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft
HKEY_LOCAL_MACHINE\SOFTWARE\ODBC
HKEY_LOCAL_MACHINE\SOFTWARE\OEM
HKEY_LOCAL_MACHINE\SOFTWARE\Partner
HKEY_LOCAL_MACHINE\SOFTWARE\Policies
HKEY_LOCAL_MACHINE\SOFTWARE\Python
HKEY_LOCAL_MACHINE\SOFTWARE\RegisteredApplications
HKEY_LOCAL_MACHINE\SOFTWARE\Thingamahoochie
HKEY_LOCAL_MACHINE\SOFTWARE\Windows
HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node
```

## System
The System hive is stored in `%SystemRoot%\System32\config\SYSTEM` and
maps to `HKLM\SYSTEM`. It stores system configuration, including the
Boot Key used to decrypt the SAM and SECURITY hives.

```
PS C:\> reg query HKLM\SYSTEM

HKEY_LOCAL_MACHINE\SYSTEM\ActivationBroker
HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001
HKEY_LOCAL_MACHINE\SYSTEM\DriverDatabase
HKEY_LOCAL_MACHINE\SYSTEM\HardwareConfig
HKEY_LOCAL_MACHINE\SYSTEM\Input
HKEY_LOCAL_MACHINE\SYSTEM\Keyboard Layout
HKEY_LOCAL_MACHINE\SYSTEM\Maps
HKEY_LOCAL_MACHINE\SYSTEM\MountedDevices
HKEY_LOCAL_MACHINE\SYSTEM\ResourceManager
HKEY_LOCAL_MACHINE\SYSTEM\ResourcePolicyStore
HKEY_LOCAL_MACHINE\SYSTEM\RNG
HKEY_LOCAL_MACHINE\SYSTEM\Select
HKEY_LOCAL_MACHINE\SYSTEM\Setup
HKEY_LOCAL_MACHINE\SYSTEM\Software
HKEY_LOCAL_MACHINE\SYSTEM\State
HKEY_LOCAL_MACHINE\SYSTEM\WaaS
HKEY_LOCAL_MACHINE\SYSTEM\WPA
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet
```

## Default
It is stored in `%SystemRoot%\System32\config\DEFAULT` and maps to `HKLM\.DEFAULT`.

```
PS C:\> reg query HKU\.DEFAULT

HKEY_USERS\.DEFAULT\Console
HKEY_USERS\.DEFAULT\Control Panel
HKEY_USERS\.DEFAULT\Environment
HKEY_USERS\.DEFAULT\EUDC
HKEY_USERS\.DEFAULT\Keyboard Layout
HKEY_USERS\.DEFAULT\PersistentPropertyBag
HKEY_USERS\.DEFAULT\Printers
HKEY_USERS\.DEFAULT\Software
HKEY_USERS\.DEFAULT\System
```

## User

There is one User hive per user and it is stored in
`%USERPROFILE%\Ntuser.dat` (aka `C:\Users\<user>\Ntuser.dat`) and
maps to `HKU\<User-SID>`.
  
## User classes
There is one User classes hive per user and it is stored in
`%USERPROFILE%\AppData\Local\Microsoft\Windows\Usrclass.dat` and maps
to `HKU\<User-SID>\Software\Classes`.

## Root keys

The [Root keys](https://en.wikipedia.org/wiki/Windows_Registry#Root_keys) of the registry are like the registry root
folders. Under these we can find all the subkeys that are stored in
the registry.

### HKLM

The *HKLM* or *HKEY_LOCAL_MACHINE* root key contains configurations for
the machine.

```
PS C:\> reg query HKLM

HKEY_LOCAL_MACHINE\BCD00000000
HKEY_LOCAL_MACHINE\HARDWARE
HKEY_LOCAL_MACHINE\SAM
HKEY_LOCAL_MACHINE\SECURITY
HKEY_LOCAL_MACHINE\SOFTWARE
HKEY_LOCAL_MACHINE\SYSTEM
```

### HKU
The *HKU* or *HKEY_USERS* root key contains the configurations for 
each user. Between its subkeys it contains each user SID.
  
```
PS C:\> reg query HKU

HKEY_USERS\.DEFAULT
HKEY_USERS\S-1-5-19
HKEY_USERS\S-1-5-20
HKEY_USERS\S-1-5-21-1004336348-1177238915-682003330-1001
HKEY_USERS\S-1-5-21-1004336348-1177238915-682003330-1001_Classes
HKEY_USERS\S-1-5-21-1004336348-1177238915-682003330-1002
HKEY_USERS\S-1-5-21-1004336348-1177238915-682003330-1002_Classes
HKEY_USERS\S-1-5-18
```
  
### HKCU
  
The *HKCU* or *HKEY_CURRENT_USER* root key is a pointer to current
user configuration in *HKU* `HCKU -> HKU\<user-SID>\`. This allows to
manage the configurations of the current user.

```
PS C:\> reg query HKCU

HKEY_CURRENT_USER\AppEvents
HKEY_CURRENT_USER\Console
HKEY_CURRENT_USER\Control Panel
HKEY_CURRENT_USER\Environment
HKEY_CURRENT_USER\EUDC
HKEY_CURRENT_USER\Keyboard Layout
HKEY_CURRENT_USER\Network
HKEY_CURRENT_USER\PersistentPropertyBag
HKEY_CURRENT_USER\Printers
HKEY_CURRENT_USER\SOFTWARE
HKEY_CURRENT_USER\System
HKEY_CURRENT_USER\Remote
HKEY_CURRENT_USER\Volatile Environment
```
  
### HKCR

The *HKCR* or *HKEY_CLASSES_ROOT* root key is intended to have the
definitive configuration for a user of file associations and COM
objects. It contains a mixture between `HKLM\SOFTWARE\Classes` and
`HKCU\SOFTWARE\Classes`. It takes the values from HKCU, the current
user configuration, and those not found are taken from the HKLM, the
system configuration. So accessing HKCR is like "give me the
definitive configuration for a value between user and system", if a
user configuration exists it will take precedence, but if not, then
the system configuration will apply.

### HKCC
I have no idea what is the purpose of *HKCC* or *HKEY_CURRENT_CONFIG*
root key.



## Resources

- [Windows Registry](https://en.wikipedia.org/wiki/Windows_Registry)
