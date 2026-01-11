# Shares

Shares are (root) directories that can be accessed between machines through SMB protocol.

## Common shares

> **Note**: Shares ending in $ are hidden shares, so you may need to use a flag
> in some tools to show them.

There are certain shares that can view in common situations:

- **C$** (and other letters ending in $): This represents a disk partition, being
  usually C$ the main Windows partition. Only administrators can access.
- **ADMIN$**: Usually the `C:\Windows\` folder. Only administrators can access.
- **IPC$**: An special share that is not a folder, but used by SMB to process RPC
  calls.

Moreover, we have some shares present in domain controllers:

- **SYSVOL**: A share used to store files accessible by the domain machines,
  like the GPO templates. By default, any user can read from SYSVOL share.

- **NETLOGON**: It points to `SYSVOL\<domain>\scripts\` folder. It is used to
  store scripts that are going to be executed by domain machines.

Additionally, the [SCCM](./sccm.md) machines also have interesting [shares](https://github.com/subat0mik/Misconfiguration-Manager/blob/main/attack-techniques/RECON/RECON-2/recon-2_description.md):

- **SCCMContentLib$**: This share contains files that are distributed by SCCM
  (in a [weird format](https://rzec.se/blog/looting-microsoft-configuration-manager/)).

## List machine shares

In Windows, we can use the `net view` command to list the shares of a remote
machine:
```
PS C:\> net view /all dc01.hack.lab
Shared resources at dc01.hack.lab

Share name  Type  Used as  Comment

-------------------------------------------------------------------------------
ADMIN$      Disk           Remote Admin
C$          Disk           Default share
IPC$        IPC            Remote IPC
NETLOGON    Disk           Logon server share
SYSVOL      Disk           Logon server share
The command completed successfully.
```

> **Note**: In case we want to use alternative credentials on the network when
> running commands on Windows we can use `runas /netonly`. Ex:
> `runas /netonly /user:hack.lab\user powershell`.


We can use the [netexec](https://github.com/Pennyw0rth/NetExec) `nxc smb --shares` command:
```
$ nxc smb --shares -u hack.lab\\user -p 'P4ssw0rd' -- dc01.hack.lab
SMB         192.168.1.125 445    DC01             [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:hack.lab) (signing:True) (SMBv1:False)
SMB         192.168.1.125 445    DC01             [+] hack.lab\user:P4ssw0rd
SMB         192.168.1.125 445    DC01             [*] Enumerated shares
SMB         192.168.1.125 445    DC01             Share           Permissions     Remark
SMB         192.168.1.125 445    DC01             -----           -----------     ------
SMB         192.168.1.125 445    DC01             ADMIN$                          Remote Admin
SMB         192.168.1.125 445    DC01             C$                              Default share
SMB         192.168.1.125 445    DC01             IPC$            READ            Remote IPC
SMB         192.168.1.125 445    DC01             NETLOGON        READ            Logon server share
SMB         192.168.1.125 445    DC01             SYSVOL          READ            Logon server share
```

Another option is to use the [impacket](https://github.com/fortra/impacket) `smbclient` command:
```
$ echo shares | impacket-smbclient hack.lab/user:'P4ssw0rd'@dc01.hack.lab
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies

Type help for list of commands
# ADMIN$
C$
IPC$
NETLOGON
SYSVOL
```

## Search secrets in shares

To automatically search secrets in shares we can use these tools:

- [Snaffler](https://github.com/SnaffCon/Snaffler)
- [PowerHuntShares](https://github.com/NetSPI/PowerHuntShares)
- [CMLoot](https://github.com/1njected/CMLoot): For searching in SCCM shares

## Resources

- [NetExec wiki: Enumerate Shares and Access](https://www.netexec.wiki/smb-protocol/enumeration/enumerate-shares-and-access)
- [Misconfiguration-Manager: RECON-2 (SMB Enumeration)](https://github.com/subat0mik/Misconfiguration-Manager/blob/main/attack-techniques/RECON/RECON-2/recon-2_description.md)
- 21/01/2023 [Looting Microsoft Configuration Manager](https://rzec.se/blog/looting-microsoft-configuration-manager/) by Tomas Rzepka
- [NetExec (tool)](https://github.com/Pennyw0rth/NetExec) by NeffIsBack, Marshall-Hallenbeck, mpgn, zblurx, termanix
  and [others](https://github.com/Pennyw0rth/NetExec/graphs/contributors)
- [impacket (tool)](https://github.com/fortra/impacket) by asolino and [others](https://github.com/fortra/impacket/graphs/contributors)
- [Snaffler (tool)](https://github.com/SnaffCon/Snaffler) by @l0ss, @Sh3r4 and hkelley
- [PowerHuntShares (tool)](https://github.com/NetSPI/PowerHuntShares) by nullbind
- [CMLoot (tool)](https://github.com/1njected/CMLoot) by 1njected
