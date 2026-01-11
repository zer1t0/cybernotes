# Wireshark

## Filters

### Show DNS Queries

Show all queries:
```
dns.flags.response == 0
```

Only A queries:
```
dns.flags.response == 0 && dns.qry.type == 1
```


Resources:
- [How Do You Identify DNS Queries And Responses In Wireshark?](https://www.cyberly.org/en/how-do-you-identify-dns-queries-and-responses-in-wireshark/index.html)


### Show TLS Client Hello

```
tls.handshake.type == 1
```

Resources:
- [Wireshark Tutorial: Changing Your Column Display](https://unit42.paloaltonetworks.com/unit42-customizing-wireshark-changing-column-display/)
