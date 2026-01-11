# AD Recon

## Tools for AD recon

Most of the AD recon is done over LDAP protocol, so here are some useful tools
to interact with LDAP:

- [Bloodhound CE](https://github.com/SpecterOps/BloodHound): (GUI)
- [Sharphound](https://github.com/SpecterOps/SharpHound): (CLI) The csharp bloodhound ingestor
- [BloodHound.py](https://github.com/dirkjanm/BloodHound.py): (CLI) The python bloodhound ingestor
- [AdExplorer](https://learn.microsoft.com/en-us/sysinternals/downloads/adexplorer): (GUI) Sysinternals interactive LDAP tool
- [ldapsearch](https://docs.ldap.com/ldap-sdk/docs/tool-usages/ldapsearch.html): (CLI) For querying LDAP
- [ldeep](https://github.com/franc-pentest/ldeep): (CLI) For querying LDAP and also a cache
- [godap](https://github.com/Macmod/godap): (TUI) Interactive LDAP in terminal
- [Powerview](https://github.com/BC-SECURITY/Empire/blob/main/empire/server/data/module_source/situational_awareness/network/powerview.ps1): (CLI) Enumerate domain from Powershell
- [ActiveDirectory Powershell Module](https://learn.microsoft.com/en-us/powershell/module/activedirectory/?view=windowsserver2025-ps): (CLI) Enumerate domain from Powershell
- [msldap](https://github.com/skelsec/msldap): (CLI) Tool to work with AD LDAP.

## Identify the domain

Probably the first thing to know is the domain we are going actually recon. In
many situations this is obvious, but sometimes is not and it is good to know how
to discover the domain to which a compromised machine belongs.

### Identify the domain in Windows

Read the environment variables:
- `USERDNSDOMAIN`: Current user domain in DNS format.
- `USERDOMAIN`: Current user domain in NetBios format.

Read the `Domain` value from
`HKLM\System\CurrentControlSet\Services\Tcpip\Parameters` key in the registry:
```
C:\>reg query "HKLM\System\CurrentControlSet\Services\Tcpip\Parameters" /v Domain

HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\Tcpip\Parameters
    Domain    REG_SZ    hack.lab
```

### Identify the domain in GNU/Linux

In GNU/Linux domain added machines have several different services that is
required to configure in order to work properly with an AD domain. In the
configuration files of these services we can find which is the configured
domain. Here is a list of some of these files:

- `/etc/resolv.conf`: The DNS configuration
- `/etc/krb5.conf`: The kerberos configuration
- `/etc/sssd/sssd.conf`: The [sssd](https://linux.die.net/man/8/sssd) configuration
- `/etc/samba/smb.conf`: The samba configuration
- `/etc/security/pam_mount.conf.xml`: File with information about mounted
  shares

For example, the DNS may have a hint of the domain in the `/etc/resolv.conf`
file under a `search` attribute:
```
$ cat /etc/resolv.conf
search hack.lab.
nameserver 192.168.0.1
nameserver 192.168.0.2
```

## Identify DNS servers

We perfoming recon on AD, it is important to identify the DNS servers we need to
use resolve the hostnames of the domain.

### Identify DNS servers in Windows

In a Windows machine, we can retrieve the DNS servers from the registry (each
network interface can have different DNS servers):
- Key:
  `HKLM\System\CurrentControlSet\Services\Tcpip\Parameters\Interfaces\<interface-guid>`
- Value: `NameServer`

Here is an example with reg:
```
C:\>reg query "HKLM\System\CurrentControlSet\Services\Tcpip\Parameters\Interfaces" /s /v NameServer

HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\Tcpip\Parameters\Interfaces\{baf593e7-4b26-497e-bb02-27f308a0aeeb}
    NameServer    REG_SZ

HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\Tcpip\Parameters\Interfaces\{b6e0d476-96a6-4caf-a7ad-131e418e7ca1}
    NameServer    REG_SZ    192.168.0.1,192.168.0.2

End of search: 2 match(es) found.
```

We can also retrieve the DNS with tools like `ipconfig` or in the network
settings:
```
C:\>ipconfig /all
...
   DNS Servers . . . . . . . . . . . : 192.168.0.1
                                       192.168.0.2
...
```

### Identify DNS servers in GNU/Linux

We can identify the DNS servers commonly in the `/etc/resolv.conf` file:
```
$ cat /etc/resolv.conf
search hack.lab.
nameserver 192.168.0.1
nameserver 192.168.0.2
```

Alternatively, when managed by systemd, the DNS servers can be found in
`/run/systemd/resolve/resolv.conf`:
```
$ cat /run/systemd/resolve/resolv.conf | grep nameserver
nameserver 192.168.0.1
nameserver 192.168.0.2
```

## Identify Domain Controllers

One of the first things when we are in an AD environment is to identify the
Domain Controllers, at least one, to know to which computers send our queries
for recon.

To identify the DCs with we can perform a [DNS *SRV* query](https://www.cloudflare.com/learning/dns/dns-records/dns-srv-record/) to some of these
resources:
- `_ldap._tcp.dc._msdcs.<domain>`: Identifies the LDAP servers, aka DCs.
- `_kerberos._tcp.<domain>`: Identifies the Kerberos servers, aka
  DCs.

From Windows we can use `nslookup` to perform the DNS request:
```
nslookup -q=srv _ldap._tcp.dc._msdcs.contoso.local
```

Windows example:
```
C:\>nslookup -q=srv _ldap._tcp.dc._msdcs.hack.lab
Server:  ip6-localhost
Address:  ::1

_ldap._tcp.dc._msdcs.hack.lab   SRV service location:
          priority       = 0
          weight         = 100
          port           = 389
          svr hostname   = dc01.hack.lab
dc01.hack.lab   internet address = 192.168.0.1
```

From GNU/Linux:
```
# Basic command
dig srv _ldap._tcp.dc._msdcs.contoso.local

# With socks proxy and specifying the DNS
proxychains dig +tcp srv _ldap._tcp.dc._msdcs.contoso.local @192.168.0.1
```

GNU/Linux example:
```
$ dig +noall +answer +additional srv _ldap._tcp.dc._msdcs.hack.lab
_ldap._tcp.dc._msdcs.hack.lab. 600 IN	SRV	0 100 389 dc01.hack.lab.
dc01.hack.lab.		3600	IN	A	192.168.0.1
```

## ldapsearch

The LDAP protocol is the most common way used to ask information about a domain
to a DC. `ldapsearch` is the most common tool to use from a GNU/Linux machine.

ldapsearch template:
```
ldapsearch -LLL -E pr=1000/noprompt -H ldap://<dc-address> \
-D <user>@<domain> -w <password> -Q \
-b 'DC=<domain-component>,DC=<tld>' \
-s sub \
<filter> [attribute]..
```

Here is the explanation for the used parameters:
- `-LLL`: Removes the unnecesary comments in the output.
- `-E pr=1000/noprompt`: Performs paged queries that retrieve 1000 items each
  one until all the items are returned, without prompting for continue. (This is
  useful in case of large environments with many items like users, since by
  default ldapsearch only retrieves the first 1000 items)

- `-H ldap://<dc-address>`: Indicates to connect via LDAP to the given hostname
  or ip, which in our case should be a Domain Controller address. To use LDAPS
  with ldapsearch you need to specify `-H ldaps://<dc-address>` and you may need
  to [disable the certificate checking](https://convincingbits.wordpress.com/2016/01/27/ldapsearch-using-tls-and-self-signed-server-certificates/), which can be done with
  `sudo sh -c "echo 'TLS_REQCERT never' >> /etc/ldap/ldap.conf"`.

- `-D <user>@<domain>`: The user to use for authentication, with the email
  format that includes the domain.

- `-w <password>`: The password for the user. Use `-W` in case you want to be
  prompted for the password interactively.

- `-Q`: Indicates to use SASL authentication, that doesn't send the password in
  plaintext. In case of error, you can use `-x` flag instead to use
  simple authentication (be aware that this sends the credentials directly so it
  can expose the credentials if LDAPS is not used).

- `-b 'DC=<domain-component>,DC=<tld>'`: The base in the LDAP tree to search, it
  is usually the domain name splitted (in case of `hack.lab` the base is
  `DC=hack,DC=lab`) but can be also some container like an OU or some CNs (like
  `CN=Users,DC=hack,DC=lab` which restricts the scope of the search to that
  subtree).

- `-s sub`: The scope of the search, that can be `sub` to search in whole LDAP
  subtree under the specified base, this is the default in ldapsearch and
  usually in any tool. The other options are `base` to only retrieve the object
  specified in the base, and `one` to retrieve only the objects one level below
  in the subtree.

- `<filter>`: Is the LDAP filter used to only retrieve the objects (in scope
  under the base) that match the given conditions.

- `[attribute]`: We can optionally specified the specific attributes that are
  going to be returned in our search. If none is specified, all are retrieved


```
ldapsearch -LLL -x -H ldaps://dc01.hack.lab -D user@hack.lab -w 'P4ssw0rd' \
-b 'DC=hack,DC=lab' \
-s sub \
"(objectclass=user)" samaccountname
```

Retrieve computer FQDNs:
```
ldapsearch -LLL -x -H ldaps://dc01.hack.lab -D user@hack.lab -w 'P4ssw0rd' \
 -b 'DC=hack,DC=lab' \
 "(objectclass=computer)" \
 dnshostname | grep -i dnshostname | cut -f 2 -d ' '
```

### ntSecurityDescriptor in ldap search

In case you want to [retrieve the ntSecurityDescriptor](https://nitter.net/tifkin_/status/1372628611677753344) attribute from a LDAP
query, you have to specify the following option
`-E '!1.2.840.113556.1.4.801=::MAMCAQc='`. This adds a the
[LDAP_SERVER_SD_FLAGS_OID](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-adts/3888c2b7-35b9-45b7-afeb-b772aa932dd0?redirectedfrom=MSDN) control to the LDAP
search request that specifies we want to retrieve the NT Security Descriptor
except the SACL (value 0x7), since this last one it is only available for
admins.


## Bloodhound

### Bloodhound: Connect directly to neo4j

In Bloodhound CE, we can connect to the neo4j from `http://127.0.0.1:7474/` and
default credentials `neo4j:bloodhoundcommunityedition`.

### Bloodhound: Cypher queries

Resources:
- [Exegol bloodhound queries](https://github.com/ThePorgs/Exegol-images/blob/3d6d7a41e46acb6898da996c4198971be02e4d77/sources/bloodhound/customqueries.json)

Users/Computers belonging a group:
```cypher
MATCH a = (g:Group)<-[:MemberOf*1..]-(u)
WHERE tolower(g.samaccountname) = "domain admins"
RETURN a
LIMIT 1000
```

Users whose password never expires:
```cypher
match (u:User) where u.enabled and u.pwdneverexpires return u.samaccountname
```

### Bloodhound Resources
- [BloodHound Community Edition Quickstart](https://bloodhound.specterops.io/get-started/quickstart/community-edition-quickstart)
- 24/07/2024 [BloodHound CE and neo4j queries — statistics, ADCS and more](https://arth0s.medium.com/bloodhound-ce-and-neo4j-queries-statistics-adcs-and-more-3a060be0a98c) by arth0s
- [SharpHound](https://github.com/SpecterOps/SharpHound) by rvazarkar, JonasBK and [others](https://github.com/SpecterOps/SharpHound/graphs/contributors)
- [BloodHound.py](https://github.com/dirkjanm/BloodHound.py) by dirkjanm

## Resources

- 14/05/2022 [LDAPSearch Reference](https://malicious.link/posts/2022/ldapsearch-reference/) by Rob Fuller
- [LDAP Wiki](https://ldapwiki.com/wiki/)
- [LDAP Search Filter Cheatsheet](https://gist.github.com/jonlabelle/0f8ec20c2474084325a89bc5362008a7) by jonlabelle
- 23/06/2025 [Active Directory Penetration Testing Using Impacket](https://www.hackingarticles.in/active-directory-penetration-testing-using-impacket/) by Raj
- [How to join Debian to Active Directory](https://hackliza.gal/en/posts/linux-en-ad/)
- 17/05/2024 [How To Get BitLocker Recovery Passwords from Active Directory Using
  PowerShell](https://medium.com/@dbilanoski/how-to-get-bitlocker-recovery-passwords-from-active-directory-using-powershell-with-30a93e8dd8f2) by Danilo Bilanoski
-
