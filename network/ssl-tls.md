# SSL/TLS

## Encrypted Client Hello

When using ECH (Encrypted Client Hello), the Client Hello message contains a
generic SNI that indicates the general CDN, for example, Cloudflare supports ECH
and the required generic SNI is `cloudflare-ech.com`. Additionally, the ECH
contains an inner SNI that is encrypted and contains the real endpoint domain,
which can be decrypted by the provider, like Cloudflare, and thus serving the
proper certificate.


Resources:
- [Encrypted Client Hello - the last puzzle piece to privacy](https://blog.cloudflare.com/announcing-encrypted-client-hello/#how-does-ech-work)
