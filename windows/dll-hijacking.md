# DLL Hijacking

## COM Hijacking

When a program wants to use a COM object, it must locate its server by
the Class ID or CLSID. This translates, at least locally, into
searching the CLSID into the registry.

We can find classes in the registry in the following places:

- `HKLM\SOFTWARE\Classes\CLSID\`: The global COM Classes.
- `HKCU\SOFTWARE\Classes\CLSID\`: Classes specific for the current
  User. This is linked from `HKU\<user-SID>\SOFTWARE\Classes\CLSID\`.
- `HKCR\CLSID`: A mixture between the previous two, it tries to map
  the classes from user first, and then those from the system, so when
  searching it, if HKCU and HKLM have the same CLSID defined, HKCU
  (current user) will take preference.


The key point here is that user classes have preference over system
classes, and current user classes can be altered by the current
user. Imagine the following situation, the program will search for the
CLSID `018D5C66-4533-4307-9B53-224DE2ED1FE6` by checking the following
registry keys:

- `HKCU\SOFTWARE\Classes\CLSID\{018D5C66-4533-4307-9B53-224DE2ED1FE6}`
- `HKCR\CLSID\{018D5C66-4533-4307-9B53-224DE2ED1FE6}`

If the first one exists, the program we will load the COM server
pointed by the that one. In case it doesn't exists, it will check the
second one.

The important thing is that `HKCU` is checked first and the user can
write into it. So we can look for programs with [procmon](https://learn.microsoft.com/en-us/sysinternals/downloads/procmon) that are
unabled to get a COM class from `HKCU` and use this knowledge to
hijack the execution.

The another important thing to note is that COM servers can be DLLs
that are load in the target process. Those are one that we are
interested in. In this case we will have the process searching for the
`InProcServer32` subkey. Here is an example:

```
PS C:\> reg query "HKLM\Software\Classes\CLSID\{f0ae1542-f497-484b-a175-a20db09144ba}\InProcServer32"

HKEY_LOCAL_MACHINE\Software\Classes\CLSID\{f0ae1542-f497-484b-a175-a20db09144ba}\InProcServer32
    (Default)    REG_EXPAND_SZ    %SystemRoot%\system32\windows.storage.dll
    ThreadingModel    REG_SZ    Apartment
```

In case the process wants to use the COM class
`f0ae1542-f497-484b-a175-a20db09144ba` it must load the
`windows.storage.dll` DLL.

So if we find a program that is trying load the previous class, we
will probably found that are unabled to load it from `HKCU`, since it
doesn't exists there:

```
PS C:\> reg query "HKCU\Software\Classes\CLSID\{f0ae1542-f497-484b-a175-a20db09144ba}\InProcServer32"
ERROR: The system was unable to find the specified registry key or value.
```

The what we can do is to register our DLL to be load by the program:

```
PS C:\> reg add "HKCU\Software\Classes\CLSID\{f0ae1542-f497-484b-a175-a20db09144ba}\InprocServer32" /d "C:\Users\user\FakeCLSID.dll"
The operation completed successfully.
```

Additionally, in order for our DLL to work properly, we may want to
provide with the correct functions. That can be done through DLL proxy
by using [Koppeling](https://github.com/monoxgas/Koppeling) that includes in our DLL the exports of other
DLL and redirects to the original one. In our case this could be:

```
PS C:\> cd Koppeling\Bin
PS C:\> .\NetClone.exe --target C:\Users\user\BaseFakeCLSID.dll --reference C:\Windows\system32\windows.storage.dll --output "C:\Users\user\FakeCLSID.dll"
```

We add the exports of `windows.storage.dll` to `BaseFakeCLSID.dll`,
resulting into `FakeCLSID.dll`, that contains our payload, and
includes the required functionality to redirect the calls to the
original DLL, avoiding disruptions.

Finally, once done, to remove the CLSID, we can do:
```
PS C:\> reg delete "HKCU\Software\Classes\CLSID\{f0ae1542-f497-484b-a175-a20db09144ba}" /f
The operation completed successfully.
```

- 2018/06/28 [Abusing the COM Registry Structure: CLSID, LocalServer32, &
  InprocServer32](https://bohops.com/2018/06/28/abusing-com-registry-structure-clsid-localserver32-inprocserver32/) by Bohops

- 2018/08/18 [Abusing the COM Registry Structure (Part 2): Hijacking & Loading
  Techniques](https://bohops.com/2018/08/18/abusing-the-com-registry-structure-part-2-loading-techniques-for-evasion-and-persistence/) by Bohops

## DLL Proxying

- 2020/02/19 [Adaptive DLL Hijacking](https://silentbreaksecurity.com/adaptive-dll-hijacking/) by Nick Landers
- [Koppeling](https://github.com/monoxgas/Koppeling) by monoxgas (aka Nick Landers)
