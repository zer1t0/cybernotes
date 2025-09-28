# Email

## DKIM

DKIM (DomainKeys Identified Mail) is an email authentication mechanism
based in public key cryptography. The idea is that the sender creates
a signature of the email with its private key and the recipient can
verify this signature with the public key.

In order to DKIM to work we have two parts: The DKIM email header and
the DKIM record.

The DKIM header contains two signatures, one for the body, and another
for some headers. Here is an example of a DKIM header:
```
DKIM-Signature: v=1;
 a=rsa-sha256;
 s=selector1;
 d=example.com;
 h=To:From:Subject:Date:Message-ID:MIME-Version:Content-Type:Content-Transfer-Encoding;
 bh=Z2huY2F0bHFibWtyc2h3b2xwdWxucXFyeml0eHhncA==;
 b=aWh4a3R5dGVjeHJ6aWZsY3N2a3RsbWF0cmpkY3hnZHlmb2ZrbGJr
```

The sections of the DKIM header are the following:
- a: Cryptographic algorithm
- s: Selector, used to retrieve the DKIM public key
- d: Sender domain
- h: Headers whose content was included in the header signature
- bh: The header signature
- b: The body signature


Once the recipient receives the email with the DKIM header, it can
retrieve the public key of the domain from a DKIM record, which is a
TXT DNS record, stored in the subdomain
`<selector>._domainkey.<domain>`. In this case the subdomain would be
`selector1._domainkey.example.com`. Here is the example of DKIM record:
```
$ dig TXT selector1._domainkey.example.com | grep DKIM
selector1._domainkey.example.com. 3600 IN TXT "v=DKIM1; k=rsa; p=MIIBI
R4Ccc4X4vVgomKVfxh9Glxh0gAGP2HsCKFh27nha8Es4TydPdXnvmWpI1bPoRSKUU8+mHL
76/noYE2Qdm12BWrPI/QVfK4aMrOS6LaW2pfGMmbLFpeFY8pMLJKjvEDdhovtOYmOkVGsu
KQS55NwGIdM7F2gYLhrEkPsqYbLkAqyNvJPG/Q+OFc5OdvMklv08Knod14BHfWD//GNuSp
KU8ahwBLRfWnaNzgwpFiQ8YGWRWt2jGN2H6yFVo/VsRzTrx62Gy4kEjjx4K0vxoO7eaygU
CdIRPIgFXKxw5N8+RJHSuBPokcxUY7FCO0PjHF1OvemIokfHNzzaQDj3xY0itx5VQgv6oy
XOHr4NHO2vBr5QHWW4fqdk7VkMjLb6wR4E+FO" "11IY4BBWqZQXFP/ITj6dv56fDhaIF9
Rhsfja14MfOSEtoQJdrDSE1rS5G7XtIZdJ/MGvGZmXm3nxy7BgdPfGH3/dDPO0pQiY4QWQ
rZGFIKvL+9BneqIJP/YWBSlPZGNogz4EgrIhblAfttQ++V9qJYfI9+0;"
```

The reason behind using a selector to retrieve the DKIM public key is
that a domain can delegate its email management to several providers,
each one having a different key to sign the email with DKIM.

Once the recipient access to the DKIM public key, it can verify if the
signature is correct. In case of a failed verification, it should
follow the DMARC policies in order to handle the email, that can be
descarded or marked as spam.

Resources:
- [What is a DNS DKIM record?](https://www.cloudflare.com/learning/dns/dns-records/dns-dkim-record/)
- [Wikipedia: DomainKeys Identified Mail](https://en.wikipedia.org/wiki/DomainKeys_Identified_Mail)

## SPF

SPF (Sender Policy Framework) is a mechanism for authenticating email
senders. It includes a DNS record known as the SPF record that
indicates which hosts are allowed to send emails from the given
domain.

The SPF record is a TXT record that starts by `v=spf1`. Here is an example:
```
$ dig TXT wikipedia.org | grep spf1
wikipedia.org. 2400 IN TXT "v=spf1 include:_cidrs.wikimedia.org ~all"
```

In this example we can see two directives,
`include:_cidrs.wikimedia.org` that indicates that the hosts defined
by `_cidrs.wikimedia.org`, in their own SPF record, must be allowed,
and `~all` that indicates that the rest of emails received from
different hosts should be treated as spam.

Then, if we continue the SPF chain, we can find the following SPF
record:
```
$ dig TXT _cidrs.wikimedia.org | grep spf1
_cidrs.wikimedia.org. 3600 IN TXT "v=spf1 ip4:208.80.152.0/22 ip6:2620:0:860::/56 ip6:2620:0:861::/56 ~all"
```

This SPF record indicates that emails coming from IPv4 range
`208.80.152.0/22` and IPv6 ranges `2620:0:860::/56` and
`2620:0:861::/56` are accepted, whereas the rest should be treated as
spam for domain `_cidrs.wikimedia.org`.

SPF records must be carefully handled as a misconfiguration can lead
to email impersonation. For example, including the directive `+all`
indicates that anyone can send an email from the given domain.

Resources:
- [Wikipedia: Sender Policy Framework](https://en.wikipedia.org/wiki/Sender_Policy_Framework)
- [What is a DNS SPF record?](https://www.cloudflare.com/learning/dns/dns-records/dns-spf-record/)

## DMARC

DMARC (Domain-based Message Authentication Reporting and Conformance)
is policy mechanism that instructs what to do if SPF or DKIM
authentication fails.


Resources:
- [Wikipedia: DMARC](https://en.wikipedia.org/wiki/DMARC)
- [What is a DNS DMARC record?](https://www.cloudflare.com/learning/dns/dns-records/dns-dmarc-record/)
