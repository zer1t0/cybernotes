# AD to Azure

## Kerberos AZUREADSSOACC

The `AZUREADSSOACC$` account NT hash is used to create Kerberos tickets that are
used to provide SSO from AD to Azure.

We can create a new Kerberos ticket for Desktop SSO with [New-ADDIntKerberosTicket](https://aadinternals.com/aadinternals/#new-aadintkerberosticket)
```
New-AADIntKerberosTicket -SidString <Account AD SID> -Hash <SSOACC$ NT Hash>
```

Resources:
- 23/03/2025 [Abusing AZUREADSSOACC for Pivoting From on Premises Active
  Directory to Azure](https://d4rthmaulcop.com/blog/66c9ec2c-0c31-11f0-8000-82a14e64ae37/) by d4rthmaulcop
- [Active Directory and Desktop SSO (Seamless SSO)](https://aadinternals.com/post/on-prem_admin/#active-directory-and-desktop-sso-seamless-sso) by Dr Nestori Syynimaa
- 09/02/2023 [Azure AD Kerberos Tickets: Pivoting to the Cloud](https://trustedsec.com/blog/azure-ad-kerberos-tickets-pivoting-to-the-cloud) by Edwin David


## Resources

- [AADInternals](https://aadinternals.com/)
- [AADInternals (tool)](https://github.com/Gerenios/AADInternals)
