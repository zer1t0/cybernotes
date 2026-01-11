# Windows persistence

## COM Hijacking

Refer to [DLL Hijacking](./dll-hijacking.md).

## DLL Hijacking

Refer to [DLL Hijacking](./dll-hijacking.md).

## Office Trusted Locations

[Office Trusted Locations](https://learn.microsoft.com/en-us/microsoft-365-apps/security/trusted-locations) are folders that Office applications assume are
safe to load content from. Therefore we can try to place Office files in these
folders that will be loaded by Office without warnings.

We can check the Trusted Locations of an Office program by going to:

File > Options > Trust Center > Trust Center Settings... > Trusted Locations

Additionally, we can get the Trusted Locations of an Office program by querying
the registry, specifically the subkeys of key
`HKCU\Software\Microsoft\Office\<version>\<program>\Security\Trusted Locations`
where `<version>` is the version of the Office Suite (16.0, 15.0) and
`<program>` is the Office program (Access, Excel, PowerPoint, Word).

Resources:

- 2019 [Persistence: “the continued or prolonged existence of something”: Part 1 –
  Microsoft Office](https://www.mdsec.co.uk/2019/05/persistence-the-continued-or-prolonged-existence-of-something-part-1-microsoft-office/) by Dominic Chell
- 21/04/2017 [Add-In Opportunities for Office Persistence](https://web.archive.org/web/20190526112859/https://labs.mwrinfosecurity.com/blog/add-in-opportunities-for-office-persistence/) by William Knowles

### Excel Trusted Locations

Usually to gain persistence at user level, we can put files like .xll in
`%USERPROFILE%\AppData\Roaming\Microsoft\Excel\XLSTART` path.

However, the Trusted Locations can be modified. We can list the current user
Excel Trusted Locations by checking the
`HKCU\Software\Microsoft\Office\16.0\Excel\Security\Trusted Locations` key.

```
C:\>reg query "HKCU\Software\Microsoft\Office\16.0\Excel\Security\Trusted Locations" /s /f Path

HKEY_CURRENT_USER\Software\Microsoft\Office\16.0\Excel\Security\Trusted Locations\Location0
    Path    REG_SZ    C:\Program Files\Microsoft Office\root\Office16\XLSTART\

HKEY_CURRENT_USER\Software\Microsoft\Office\16.0\Excel\Security\Trusted Locations\Location1
    Path    REG_EXPAND_SZ    %APPDATA%\Microsoft\Excel\XLSTART

HKEY_CURRENT_USER\Software\Microsoft\Office\16.0\Excel\Security\Trusted Locations\Location2
    Path    REG_EXPAND_SZ    %APPDATA%\Microsoft\Templates

HKEY_CURRENT_USER\Software\Microsoft\Office\16.0\Excel\Security\Trusted Locations\Location3
    Path    REG_SZ    C:\Program Files\Microsoft Office\root\Templates\

HKEY_CURRENT_USER\Software\Microsoft\Office\16.0\Excel\Security\Trusted Locations\Location4
    Path    REG_SZ    C:\Program Files\Microsoft Office\root\Office16\STARTUP\

HKEY_CURRENT_USER\Software\Microsoft\Office\16.0\Excel\Security\Trusted Locations\Location5
    Path    REG_SZ    C:\Program Files\Microsoft Office\root\Office16\Library\

End of search: 6 match(es) found.
```

### Word Trusted Locations

Usually to gain persistence at user level, we can put files like .wll in
`%USERPROFILE%\AppData\Roaming\Microsoft\Word\Startup` path.

However, the Trusted Locations can be modified. We can list the current user Word
Trusted Locations by checking the
`HKCU\Software\Microsoft\Office\16.0\Word\Security\Trusted Locations` key.

```
C:\>reg query "HKCU\Software\Microsoft\Office\16.0\Word\Security\Trusted Locations" /s /f Path

HKEY_CURRENT_USER\Software\Microsoft\Office\16.0\Word\Security\Trusted Locations\Location0
    Path    REG_EXPAND_SZ    %APPDATA%\Microsoft\Templates

HKEY_CURRENT_USER\Software\Microsoft\Office\16.0\Word\Security\Trusted Locations\Location1
    Path    REG_SZ    C:\Program Files\Microsoft Office\root\Templates\

HKEY_CURRENT_USER\Software\Microsoft\Office\16.0\Word\Security\Trusted Locations\Location2
    Path    REG_EXPAND_SZ    %APPDATA%\Microsoft\Word\Startup

End of search: 3 match(es) found.
```



## Registry Run Keys

- HKLM\Software\Microsoft\Windows\CurrentVersion\Run
- HKLM\Software\Microsoft\Windows\CurrentVersion\RunOnce
- HKCU\Software\Microsoft\Windows\CurrentVersion\Run
- HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce

Here is an example of contents of one of the Run keys:
```
C:\> reg query "HKCU\Software\Microsoft\Windows\CurrentVersion\Run"

HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run
    OneDrive    REG_SZ    "C:\Users\user\AppData\Local\Microsoft\OneDrive\OneDrive.exe" /background
```

Resources:
- [Run and RunOnce Registry Keys](https://learn.microsoft.com/en-us/windows/win32/setupapi/run-and-runonce-registry-keys)
- [Boot or Logon Autostart Execution: Registry Run Keys / Startup Folder](https://attack.mitre.org/techniques/T1547/001/)

## Services

Resources:
- [Create or Modify System Process: Windows Service](https://attack.mitre.org/techniques/T1543/003/)

## Startup Folder

Resources:
- [Boot or Logon Autostart Execution: Registry Run Keys / Startup Folder](https://attack.mitre.org/techniques/T1547/001/)
