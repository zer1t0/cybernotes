# DCOM

DCOM (Distributed COM) allows to call COM objects in remote systems. DCOM uses
MSRPC as its transport protocol (so it can be accessed via TCP or SMB).


- [Object exporter](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-dcom/241d7e93-b4ef-45bf-9c84-2c741a8c23d8) contains the COM objects, identified by an OXID (Object
  Exporter ID).
  
- [DCOM Activation](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-dcom/c767a336-608a-4005-a39d-0d5bc68d34b7): Is the action of creating an object.

## OXID Resolver

The OXID resolver allows clients to know how to connect to a COM server based on
the OXID (Object Exporter ID). A client can call the OXID resolver
[ResolveOxid2](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-dcom/65292e10-ef0c-43ee-bce7-788e271cc794) or [ResolveOxid](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-dcom/b6c19c08-54fc-4b5e-bbcd-6d50a2330e9e) methods to retrieve the COM server
address, including IP and port, and the security bindings, that indicates how to
authenticate against that COM server (NTLM, Kerberos, etc).

Additionally, the OXID resolver also allows to keep connection with servers by
providing some ping methods ([OXID Resolver methods](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-dcom/8ed0ae33-56a1-44b7-979f-5972f0e9416c)). This is done to avoid
the garbage collector from delete objects, since if the client don't comunicate
with the target object for a long time, the garbage collector will remove its
reference to the object.

The OXID resolver RPC interface UUID is 99FCFEC4-5260-101B-BBCB-00AA0021347A.

We can check that OXID resolver is available with [rpcmap (impacket)](https://github.com/fortra/impacket/blob/master/examples/rpcmap.py) (no
auth required):
```
$ impacket-rpcmap 'ncacn_ip_tcp:192.168.0.1' | grep 99FCFEC4 -B 2
Protocol: [MS-DCOM]: Distributed Component Object Model (DCOM) Remote
Provider: rpcss.dll
UUID: 99FCFEC4-5260-101B-BBCB-00AA0021347A v0.0
```

## Remote SCM Activator

The remote activator is on charge of requesting the creation of new COM objects
in the server, through the [RemoteCreateInstance](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-dcom/64af4c57-5466-4fdf-9761-753ea926a494) method. (Other
[IRemoteSCMActivator Methods](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-dcom/fd0682f8-8f5a-4082-830f-861c34db6251))

The Remote Activator RPC interface UUID is 000001A0-0000-0000-C000-000000000046.

We can check that Remote SCM Activator is available with [rpcmap (impacket)](https://github.com/fortra/impacket/blob/master/examples/rpcmap.py)
(no auth required):
```
$ impacket-rpcmap 'ncacn_ip_tcp:192.168.122.125' | grep 000001A0-0000-0000-C000-000000000046 -B 2
Protocol: [MS-DCOM]: Distributed Component Object Model (DCOM) Remote
Provider: rpcss.dll
UUID: 000001A0-0000-0000-C000-000000000046 v0.0
```

## DCOM permissions

For DCOM permissions we can find two levels:
- System-wide: Apply to all the applications.
- Process-wide or Application-wide: Each application defines its own.

Both permissions levels must allow the specific action over the DCOM object. 


We can use the `DCOMCNFG` application to view and manage the DCOM applications
permissions.

System-wide permissions can be found under [HKLM\SOFTWARE\Microsoft\Ole](https://learn.microsoft.com/en-us/windows/win32/com/hkey-local-machine-software-microsoft-ole).

Here we can found the key 

- [MachineLaunchRestriction](https://learn.microsoft.com/en-us/windows/win32/com/machinelaunchrestriction): Defines the permissions for launch and
  activation system wide.
- [LegacyAuthenticationLevel](https://learn.microsoft.com/en-us/windows/win32/com/legacyauthenticationlevel): 

Application-level permissions can be found under `HKCR\AppID\<app-id>`:
- [AccessPermission](https://learn.microsoft.com/en-us/windows/win32/com/accesspermission)
- [LaunchPermission](https://learn.microsoft.com/en-us/windows/win32/com/launchpermission): Indicates the launch and activation permissions for the
  application.

## Resources

- [InsideCOM+](https://www.thrysoee.dk/InsideCOM+/) by Guy Eddon and Henry Eddon
- 02/04/2020 [The OXID Resolver (Part 1) – Remote enumeration of network interfaces without
  any authentication](https://web.archive.org/web/20211018235221/https://airbus-cyber-security.com/the-oxid-resolver-part-1-remote-enumeration-of-network-interfaces-without-any-authentication/) by Nicolas Delhaye
-  [rpcmap (impacket) (tool)](https://github.com/fortra/impacket/blob/master/examples/rpcmap.py)
