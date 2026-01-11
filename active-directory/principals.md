# AD Principals (Users, Computers and Groups)

## Users

## List AD users

We can list users (without computer accounts) with the following query:
- Base: `DC=<domain>,DC=<tld>`
- Scope: Subtree
- Filter: `(&(objectClass=user)(objectCategory=person))`

```
ldapsearch -LLL -x -H ldaps://dc01.hack.lab -D user@hack.lab -w 'P4ssw0rd' \
-E pr=1000/noprompt \
-b 'DC=hack,DC=lab' \
'(&(objectClass=user)(objectCategory=person))'
```

An alternative LDAP query is:
- Base: `DC=<domain>,DC=<tld>`
- Scope: Subtree
- Filter: `(sAMAccountType=805306368)`

Some useful LDAP filters for the accounts:
- Disabled accounts: `(userAccountControl:1.2.840.113556.1.4.803:=2)`
- Accounts with SPN (**Warning**: Can raise alerts): `(servicePrincipalName=*)`
- Password never expires: `(userAccountControl:1.2.840.113556.1.4.803:=65536)`
- Members of group (nested): `(memberOf:1.2.840.113556.1.4.1941:=<group-cn>)`

### List GMSA

We can list the GMSAs with the following query:
- Base: `DC=<domain>,DC=<tld>`
- Scope: Subtree
- Filter: `(objectClass=msDS-GroupManagedServiceAccount)`

- 29/05/2020 [Attacking Active Directory Group Managed Service Accounts
  (GMSAs)](https://adsecurity.org/?p=4367) by Sean Metcalf
- [gMSA Active Directory Attacks](https://www.semperis.com/blog/golden-gmsa-attack/) by Yuval Gordon


## Computers

### List AD computers

We can list computer accounts with the following query:
- Base: `DC=<domain>,DC=<tld>`
- Scope: Subtree
- Filter: `(objectclass=computer)`

```
ldapsearch -LLL -x -H ldaps://dc01.hack.lab -D user@hack.lab -w 'P4ssw0rd' \
-E pr=1000/noprompt \
-b 'DC=hack,DC=lab' \
'(objectclass=computer)'
```

Some useful LDAP filters:
- Exchange Servers: `(objectCategory=msExchExchangeServer)`
- Using LAPS: `(ms-Mcs-AdmPwdExpirationtime=*)`

Computer attributes:
- **dn**: The Distinguished Name that identifies any object in LDAP.
- **sAMAccountName**: The computer account name (ends with $).
- **name**: The computer name.
- **dNSHostName**: The computer FQDN.
- **operatingSystem**: The OS name.
- **operatingSystemVersion**: The OS version.
- **whenCreated**: Indicates when the account was created. In
  [FILETIME format](https://www.epochconverter.com/ldap).
- **lastLogon**: Last time the computer account authenticates. In
  [FILETIME format](https://www.epochconverter.com/ldap).
- **pwdLastSet**: Last time the password was changed. In [FILETIME format](https://www.epochconverter.com/ldap).
- **servicePrincipalName**: The name of a service exposed by the computer.
- **ms-Mcs-AdmPwd**: Used by LAPS 1.0 to store the local admin password. It
  [requires high privileges](https://www.thehacker.recipes/ad/movement/dacl/readlapspassword) over the computer account to read it.
- **msLAPS-Password**: Used by LAPS 2.0 to store the local admin password in
  plaintext.
- **msLAPS-EncryptedPassword**: Used by LAPS 2.0 to store the local admin
  password encrypted. [It can be decrypted by the Windows API](https://blog.xpnsec.com/LAPSv2-Internals/).
- **ms-FVE-RecoveryPassword**: Includes the bitlocker recovery password.


## Groups

### List AD groups

We can list groups with the following LDAP query:
- Base: `DC=<domain>,DC=<tld>`
- Scope: Subtree
- Filter: `(objectCategory=group)`

```
ldapsearch -LLL -x -H ldaps://dc01.hack.lab -D user@hack.lab -w 'P4ssw0rd' \
-E pr=1000/noprompt \
-b 'DC=hack,DC=lab' \
'(objectCategory=group)'
```


### Add user to group

In order to add an user to an Active Directory group, we need to use LDAP to
modify the `members` attribute of the group and add the distinguished name of
the target user. Here are some examples with different tools:

With [Add-ADGroupMember](https://learn.microsoft.com/en-us/powershell/module/activedirectory/add-adgroupmember?view=windowsserver2025-ps) cmdlet from Active Directory module:
```
PS C:\> Add-ADGroupMember -Server hack.lab -Identity "Protected Users" -Members duser
```

With [powerview](https://github.com/BC-SECURITY/Empire/blob/main/empire/server/data/module_source/situational_awareness/network/powerview.ps1) `Add-DomainGroupMember` cmdlet:
```
PS C:\> Add-DomainGroupMember -Domain hack.lab -Identity "Protected Users" -Members duser
```

With [msldap](https://github.com/skelsec/msldap):
```
$ msldap ldaps+kerberos-password://hack.lab\\Administrator:'Admin1234!'@dc01.hack.lab/?dc=dc01.hack.lab \
login \
"addusertogroup 'CN=duser,CN=Users,DC=hack,DC=lab' 'CN=Domain Admins,CN=Users,DC=hack,DC=lab'"
User added to group!
```

### Protected Users

The [Protected Users](https://learn.microsoft.com/en-us/windows-server/security/credentials-protection-and-management/protected-users-security-group) group is intended to be use to increment the security
of privileged domain accounts.


We can retrieve the "Protected Users" with an ldap query:
```
$ ldapsearch -LLL -x -H ldaps://dc01.hack.lab -D user@hack.lab -w 'P4ssw0rd' \
-b 'DC=hack,DC=lab' \
'(&(objectCategory=group)(samaccountname=Protected Users))' \
member | grep member:
member: CN=LYDIA_MUNOZ,OU=ServiceAccounts,OU=ITS,OU=Stage,DC=hack,DC=lab
member: CN=DONNIE_BRADSHAW,OU=T2-Accounts,OU=Tier 2,OU=Admin,DC=hack,DC=lab
member: CN=STANLEY_TODD,OU=ServiceAccounts,OU=AWS,OU=Tier 1,DC=hack,DC=lab
```

From Windows, we can Rrtrieve the "Protected Users" with [Get-ADGroupMember](https://learn.microsoft.com/en-us/powershell/module/activedirectory/get-adgroupmember?view=windowsserver2025-ps):
```
PS C:\> Get-ADGroupMember -Server hack.lab -Identity "Protected Users" | Select-Object -ExpandProperty samaccountname
STANLEY_TODD
DONNIE_BRADSHAW
LYDIA_MUNOZ
```

For the members of this group:
- NTLM auth is disabled
- Kerberos keys DES and RC4 (NT hash) cannot be used to request tickets (except
  for Administrator account)
- Kerberos delegation cannot be applied

Here is an example of NTLM auth when user is in "Protected Users" group:
```
$ impacket-smbclient hack.lab/user@dc01.hack.lab -hashes :AC1DBEF8523BAFECE1428E067C1B114F
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[-] SMB SessionError: code: 0xc000006e - STATUS_ACCOUNT_RESTRICTION - Indicates a referenced user name and authentication information are valid, but some user account restriction has prevented successful authentication (such as time-of-day restrictions).
```

Here is an example of requesting Kerberos ticket by using the NT hash when user
is in "Protected Users" group:
```
$ impacket-getTGT hack.lab/duser -hashes :164331DA3283A26D69E5C05051572567
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

Kerberos SessionError: KDC_ERR_ETYPE_NOSUPP(KDC has no support for encryption type)
```



