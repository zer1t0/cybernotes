# Windows credentials

## LM hashes

### Check if Windows stores LM hashes

We can check the `NoLmHash` value under the
`HKLM\SYSTEM\CurrentControlSet\Control\Lsa` to see if Windows generates the
password LM hash and stores it in the SAM database:
```
C:\> reg query "HKLM\SYSTEM\CurrentControlSet\Control\Lsa" /v NoLmHash

HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Lsa
    NoLmHash    REG_DWORD    0x1
```

In this case the `NoLmHash` value is set to 1, so no LM hashes are stored. We
can also confirm this by dumping the SAM database and checking that lm hashes
contain the value `aad3b435b51404eeaad3b435b51404ee`, which is the lm hash for
the empty password.

Resources:
- [How to prevent Windows from storing a LAN manager hash of your password in Active Directory and local SAM databases](https://learn.microsoft.com/en-us/troubleshoot/windows-server/windows-security/prevent-windows-store-lm-hash-password)
