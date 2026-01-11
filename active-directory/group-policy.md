# Group Policy

[Group Policy](https://adsecurity.org/?p=2716) is the Active Directory mechanism to deploy rules and policies into
domain machines. From the Group Policy we can adjust settings like:

- Password max age and minimum requirements for local accounts
- User privileges in local machine
- System proxy
- Firewall settings

Policies can be applied through GPOs (Group Policy Object) that describe the
policies to be applied in the machines of the domain. For each GPO we can find
three relevant elements:

- **Group Policy Template**: A folder with files located in the SYSVOL share under
  `\\<domain>\SYSVOL\<domain>\Policies\`. The Group Policy Template contains the
  files that define the policy along with any other file required to deploy the
  policy (like scripts for example).

- **GPO (Group Policy Object)** or **Group Policy Container**: An LDAP object
  that is stored under `CN=Policies,CN=System,DC=<domain>,DC=<tld>`
  container. The GPO indicates the name of the policy as well as the path in
  which the policy template can be found.

- **Group Policy Link**: Each GP must be linked to a Domain, Organizational Unit
  (OU) or Site in order to be applied. For this purpose the `gPLink` attribute
  is used in the Domain, OU or Site LDAP objects. The object to which the GPO is
  linked defines its scope. For example we can apply only a GPO to the domain
  controllers by linking it to the `OU=Domain Controllers,DC=<domain>,DC=<tld>`
  OU.


## GPO

We can enumerate the GPOs (Group Policy Objects), or Group Policy Containers,
with the following LDAP query:
- Base: `DC=<domain>,DC=<tld>`
- Scope: Subtree
- Filter: `(ObjectClass=GroupPolicyContainer)`

This is the [LDAP query used by Group3r to retrieve GPOs](https://github.com/Group3r/Group3r/blob/3c4911ed65e09f0935ca4e684b72bf650233602f/LibSnaffle/ActiveDirectory/ActiveDirectory.cs#L358).

Another option is:
- Base: `CN=Policies,CN=System,DC=<domain>,DC=<tld>`
- Scope: One level

```
$ ldapsearch -LLL -x -H ldaps://dc01.hack.lab -D user@hack.lab -w 'P4ssw0rd' \
-b 'CN=Policies,CN=System,DC=hack,DC=lab' \
-s one \
name displayName gPCFileSysPath flags versionNumber gPCMachineExtensionNames \
gPCUserExtensionNames gPCWQLFilter
dn: CN={31B2F340-016D-11D2-945F-00C04FB984F9},CN=Policies,CN=System,DC=hack,DC
 =lab
displayName: Default Domain Policy
name: {31B2F340-016D-11D2-945F-00C04FB984F9}
flags: 0
versionNumber: 3
gPCFileSysPath: \\hack.lab\sysvol\hack.lab\Policies\{31B2F340-016D-11D2-945F-0
 0C04FB984F9}
gPCMachineExtensionNames: [{35378EAC-683F-11D2-A89A-00C04FBBCFA2}{53D6AB1B-248
 8-11D1-A28C-00C04FB94F17}][{827D319E-6EAC-11D2-A4EA-00C04F79F83A}{803E14A0-B4
 FB-11D0-A0D0-00A0C90F574B}][{B1BE8D72-6EAC-11D2-A4EA-00C04F79F83A}{53D6AB1B-2
 488-11D1-A28C-00C04FB94F17}]

dn: CN={6AC1786C-016F-11D2-945F-00C04fB984F9},CN=Policies,CN=System,DC=hack,DC
 =lab
displayName: Default Domain Controllers Policy
name: {6AC1786C-016F-11D2-945F-00C04fB984F9}
flags: 0
versionNumber: 1
gPCFileSysPath: \\hack.lab\sysvol\hack.lab\Policies\{6AC1786C-016F-11D2-945F-0
 0C04fB984F9}
gPCMachineExtensionNames: [{827D319E-6EAC-11D2-A4EA-00C04F79F83A}{803E14A0-B4F
 B-11D0-A0D0-00A0C90F574B}]

dn: CN={94D7D5B1-04AC-4ABE-AEF3-7E3564CF9F92},CN=Policies,CN=System,DC=hack,DC
 =lab
displayName: System Proxy
name: {94D7D5B1-04AC-4ABE-AEF3-7E3564CF9F92}
flags: 0
versionNumber: 393216
gPCFileSysPath: \\hack.lab\SysVol\hack.lab\Policies\{94D7D5B1-04AC-4ABE-AEF3-7
 E3564CF9F92}
gPCUserExtensionNames: [{00000000-0000-0000-0000-000000000000}{5C935941-A954-4
 F7C-B507-885941ECE5C4}][{E47248BA-94CC-49C4-BBB5-9EB7F05183D0}{5C935941-A954-
 4F7C-B507-885941ECE5C4}]
gPCWQLFilter: [hack.lab;{9D1C7EAF-62D1-42AF-A1B5-9A7D26EB129F};0]
```

Here are several [GPO attributes](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-gpod/d360d288-d7d5-49a9-83be-603805da1379):
- **displayName**: The GPO name
- **name**: The GPO GUID
- **flags**: Indicate if GPO is enabled or disabled:
  + 0 = Enabled
  + 1 = User configuration disabled
  + 2 = Machine configuration disabled
  + 3 = Totally disabled
- **versionNumber**: The GPO version. When the GPO changes it must be updated.
- **gPCFileSysPath**: Indicates the location of the policy files. In case we can
  overwrite it, we could deploy a rogue policy.
- **gPCMachineExtensionNames**:
- **gPCUserExtensionNames**:
- **gPCWQLFilter**: The WMI filter associated with the GPO.

## Group Policy Links

In order to know to which machines apply a given GPO, we need to know to which
object is linked, since this will give us the Scope.

We can retrieve the domains and OUs with linked GPOs with the following LDAP
query:
- Base: `DC=<domain>,DC=<tld>`
- Scope: Subtree
- Filter: `(gplink=*)`

```
$ ldapsearch -LLL -x -H ldaps://dc01.hack.lab -D user@hack.lab -w 'P4ssw0rd' \
-b 'DC=hack,DC=lab' \
"(gplink=*)" \
dn gplink

dn: DC=hack,DC=lab
gPLink: [LDAP://CN={31B2F340-016D-11D2-945F-00C04FB984F9},CN=Policies,CN=Syste
 m,DC=hack,DC=lab;0]

dn: OU=Domain Controllers,DC=hack,DC=lab
gPLink: [LDAP://CN={6AC1786C-016F-11D2-945F-00C04fB984F9},CN=Policies,CN=Syste
 m,DC=hack,DC=lab;0]

dn: OU=Testing,DC=hack,DC=lab
gPLink: [LDAP://cn={94D7D5B1-04AC-4ABE-AEF3-7E3564CF9F92},cn=policies,cn=syste
 m,DC=hack,DC=lab;0]
```

Additionally, to retrieve the sites with GPO we need to change the LDAP query
base, since sites are stored in a different LDAP partition:
- Base: `CN=Configuration,DC=<domain>,DC=<tld>`
- Scope: Subtree
- Filter: `(gplink=*)`

```
$ ldapsearch -LLL -x -H ldaps://dc01.hack.lab -D user@hack.lab -w 'P4ssw0rd' \
-b 'CN=Configuration,DC=hack,DC=lab' \
"(gplink=*)" \
dn gplink

dn: CN=SQLServersEurope,CN=Sites,CN=Configuration,DC=hack,DC=lab
gPLink: [LDAP://cn={94D7D5B1-04AC-4ABE-AEF3-7E3564CF9F92},cn=policies,cn=syste
 m,DC=hack,DC=lab;0]
```

The `gPLink` attribute has two pieces of information, separated by a `;`:
- gplink path: The distinguish name of the GPO in the LDAP directory
- gplink status: A number that indicates if the link is enabled and/or
  enforced. These are the gplink status possible values:
  + 0 = Enabled, Not enforced
  + 1 = Disabled, Not enforced
  + 2 = Enabled, Enforced
  + 3 = Disabled, Enforced


## GPO scope

There are several factors that influence when a GPO is applied:
- GPO 

- **Group Policy Link**: Indicates the Domain, OU or Site to which the GPO is
  applied.
  
- **GPO preference**: The GPOs are applied in the following order:
  1. Local
  2. Site
  3. Domain
  4. Organizational Unit
  If a GPO is applied afterwards, it will overwrite previous GPOs, so by default
  OU GPOs have the higher preference. However it is possible to establish a
  Domain GPO as *No Override*, making it the one with higher preference.

- **GPO enforcement** and **OU inheritance**: If an OU blocks inheritance, a GPO
  won't apply to its children, unless the GPO is enforced, in which case it will
  apply to all the OU tree. This is nicely explained in
  [A Red Teamer’s Guide to GPOs and OUs](https://wald0.com/?p=179) in the "GPO Enforcement Logic"
  section.
  
- **GPO ACL**: A GPO will only be applied to a group or user if they have the
  "Read" and "Apply Group Policy" permissions for the GPO. By default these
  permissions are granted to "Authenticated Users" group, which means all
  users. More info in [Filtering the Scope of a GPO](https://learn.microsoft.com/en-us/previous-versions/windows/desktop/Policy/filtering-the-scope-of-a-gpo?redirectedfrom=MSDN)

- **WMI Filtering**:


## WMI Filters

LDAP search to retrieve WMI Filters:
- Base: `CN=SOM,CN=WmiPolicy,CN=System,DC=<domain>,DC=<tld>`
- Scope: One Level

And alternative is:
- Base: `DC=<domain>,DC=<tld>`
- Scope: Subtree
- Filter: `(objectClass=msWMI-Som)`


```
$ ldapsearch -LLL -x -H ldaps://dc01.hack.lab -D user@hack.lab -w 'P4ssw0rd' \
-b 'CN=SOM,CN=WmiPolicy,CN=System,DC=hack,DC=lab' \
-s one msWMI-ID msWMI-Name msWMI-Parm2

dn: CN={9D1C7EAF-62D1-42AF-A1B5-9A7D26EB129F},CN=SOM,CN=WMIPolicy,CN=System,DC
 =hack,DC=lab
msWMI-ID: {9D1C7EAF-62D1-42AF-A1B5-9A7D26EB129F}
msWMI-Name: OnlyWindows7
msWMI-Parm2: 1;3;10;60;WQL;root\CIMv2;select * from Win32_OperatingSystem wher
 e Version like "6.%";
```

## Create GPO


 

To create a GPO, we can use `New-GPO` from [Group Policy Management in Active Directory](https://woshub.com/group-policy-active-directory/):

```
Add-WindowsCapability -Online -Name Rsat.GroupPolicy.Management.Tools~~~~0.0.1.0
```

```
New-GPO -Name "Test GPO" -Comment "GPO for testing purposes"
New-GPLink -Name "Test GPO" -Target "dc=hack,dc=lab" -LinkEnabled Yes
```

```
C:\> .\SharpGPOAbuse.exe --DomainController "dc01.hack.lab" `
--AddComputerTask --GPOName "Test GPO" --Author "hack\Administrator" `
--TaskName "Test GPO" --Command "powershell.exe" `
--Arguments '"Set-Content -Path C:\hello.txt -Value Hello"' `
--FilterEnabled --TargetDnsName ws01.hack.lab
[+] Domain = hack.lab
[+] Domain Controller = dc01.hack.lab
[+] Distinguished Name = CN=Policies,CN=System,DC=hack,DC=lab
[+] GUID of "Test GPO" is: {01206BCF-A053-43D7-A473-DE3A81C7543B}
[+] Creating file \\hack.lab\SysVol\hack.lab\Policies\{01206BCF-A053-43D7-A473-DE3A81C7543B}\Machine\Preferences\ScheduledTasks\ScheduledTasks.xml
[+] versionNumber attribute changed successfully
[+] The version number in GPT.ini was increased successfully.
[+] The GPO was modified to include a new immediate task. Wait for the GPO refresh cycle.
[+] Done!
```

Once we are done, we can delete the GPO:
```
Remove-GPO -Name "Test GPO"
```

## Resources

- [Group3r (tool)](https://github.com/Group3r/Group3r) by l0ss
- [pyGPOAbuse](https://github.com/Hackndo/pyGPOAbuse) by Hackndo and crosscutsaw
- [SharpGPOAbuse](https://github.com/FSecureLABS/SharpGPOAbuse?tab=readme-ov-file#configuring-a-computer-or-user-immediate-task) by pkb1s and panagioto

- 2/04/2018 [A Red Teamer’s Guide to GPOs and OUs](https://wald0.com/?p=179) by Andy Robbins
- 14/03/2016 [Sneaky Active Directory Persistence #17: Group Policy](https://adsecurity.org/?p=2716) by Sean
  Metcalf
- [Bloodhound: WriteGPLink](https://bloodhound.specterops.io/resources/edges/write-gp-link)
- [Filtering the Scope of a GPO](https://learn.microsoft.com/en-us/previous-versions/windows/desktop/Policy/filtering-the-scope-of-a-gpo?redirectedfrom=MSDN)
- 6/11/2019 [OU having a laugh?](https://labs.withsecure.com/publications/ou-having-a-laugh)
- 07/06/2022 [OUs and GPOs and WMI Filters, Oh My!](https://rastamouse.me/ous-and-gpos-and-wmi-filters-oh-my/) by Rasta Mouse
- 19/04/2024 [OUned.py: exploiting hidden Organizational Units ACL attack
  vectors in Active Directory](https://www.synacktiv.com/en/publications/ounedpy-exploiting-hidden-organizational-units-acl-attack-vectors-in-active-directory) by Quentin Roland
- [Create WMI Filters for the GPO](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/jj717288(v=ws.11))
