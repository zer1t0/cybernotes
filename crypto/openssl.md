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

### Check if private key and certificate matches

- 2016 [OpenSSL: Check If Private Key Matches SSL Certificate & CSR](https://www.shellhacks.com/openssl-check-private-key-matches-ssl-certificate-csr/)


### Generate Let'sEncrypt certificate

```
sudo apt update && sudo apt install certbot
```

The following command will give you a value to set in a TXT dns
record, and then will store the certificate and key into
`/etc/letsencrypt/archive/`:
```
certbot certonly --manual -d <example.com>  --agree-tos \
    --no-bootstrap --manual-public-ip-logging-ok --preferred-challenges dns-01 \
    -m  <mail@mail.com>  \
    --server https://acme-v02.api.letsencrypt.org/directory
```

- [How To Generate Let’s Encrypt Wildcard SSL Certificate](https://computingforgeeks.com/generating-letsencrypt-wildcard-ssl-certificate/)

## PKCS 12

### Extract certificate and key from PKCS 12 (.pfx, .p12)

Extract the certificate
```
openssl pkcs12 -in <file.pfx> -clcerts -nokeys -out <file.crt>
```

- [Extracting the certificate and keys from a .pfx file](https://www.ibm.com/docs/en/arl/9.7?topic=certification-extracting-certificate-keys-from-pfx-file)

### Store certificate and key into PKCS 12 (.pfx, .p12)

```
openssl pkcs12 -export -in <cert.pem> -inkey <key.pem> -out <file.pfx>
```

### Transform PKCS 12 into Java keystore

```
keytool -importkeystore -deststorepass <password> -destkeystore my-keystore.jks -srckeystore <file.pfx> -srcstoretype PKCS12
```

- [Import private key and certificate into java keystore](https://coderwall.com/p/3t4xka/import-private-key-and-certificate-into-java-keystore)
