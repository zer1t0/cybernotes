# OpenSSL


## Certificates

To get the documentation about handling certificates with OpenSSL
check [openssl-x509(1SSL)](https://docs.openssl.org/3.0/man1/openssl-x509/).

### Read a certificate

```
openssl x509 -in certificate.cer -noout -text
```

- [How do I view the details of a digital certificate .cer file?](https://serverfault.com/a/215617)


### Generate self signed certificate

```
openssl req -x509 -newkey rsa:4096 -keyout cert.key -out cert.pem -sha256 -days 3650 -nodes -subj "/CN=test.local"
```

- [How can I generate a self-signed SSL certificate using OpenSSL?](https://stackoverflow.com/a/10176685)

### Extract certificate and key from pfx, p12

Extract the certificate
```
openssl pkcs12 -in <file.pfx> -clcerts -nokeys -out <file.crt>
```

- [Extracting the certificate and keys from a .pfx file](https://www.ibm.com/docs/en/arl/9.7?topic=certification-extracting-certificate-keys-from-pfx-file)
