---
title: "Mastering SSL"
date: 2023-10-01
categories:
  - SSL
tags:
  - SSL
  - Certificates
  - OpenSSL
---

## Inhaltsverzeichnis

1. [Creating Certificate Signing Requests](#creating-certificate-signing-requests)
   - [About Certificate Signing Requests](#about-certificate-signing-requests)
   - [Create a Private Key and a CSR](#create-a-private-key-and-a-csr)
   - [Create a CSR from an Existing Private Key](#create-a-csr-from-an-existing-private-key)
2. [Building SSL Certificates](#building-ssl-certificates)
   - [Create a Self-Signed Certificate](#create-a-self-signed-certificate)
   - [Create a Self-Signed Certificate from an Existing Private Key](#create-a-self-signed-certificate-from-an-existing-private-key)
   - [Create a Self-Signed Certificate from an Existing Private Key and CSR](#create-a-self-signed-certificate-from-an-existing-private-key-and-csr)
3. [Inspect Certificates](#inspect-certificates)
   - [Show Signing Request](#show-signing-request)
   - [Show Certificate](#show-certificate)
   - [Verify a Certificate was Signed by a CA](#verify-a-certificate-was-signed-by-a-ca)
4. [Private Keys](#private-keys)
   - [Create a Private Key](#create-a-private-key)
   - [Verify a Private Key](#verify-a-private-key)
   - [Verify a Private Key Matches a Certificate and CSR](#verify-a-private-key-matches-a-certificate-and-csr)
   - [Encrypt a Private Key](#encrypt-a-private-key)
   - [Decrypt a Private Key](#decrypt-a-private-key)
5. [Convert Certificate Formats](#convert-certificate-formats)
   - [Convert PEM to DER](#convert-pem-to-der)
   - [Convert DER to PEM](#convert-der-to-pem)
   - [Convert PEM to PKCS7](#convert-pem-to-pkcs7)
   - [Convert PEM to PKCS12](#convert-pem-to-pkcs12)
   - [Convert PKCS7 to PEM](#convert-pkcs7-to-pem)
   - [Convert PKCS12 to PEM](#convert-pkcs12-to-pem)
6. [Automating SSL Certificate Renewal](#automating-ssl-certificate-renewal)
7. [Troubleshooting SSL Issues](#troubleshooting-ssl-issues)
8. [Securing SSL Configurations](#securing-ssl-configurations)
9. [Common OpenSSL Commands](#common-openssl-commands)
10. [Advanced OpenSSL Commands](#advanced-openssl-commands)

---

## Creating Certificate Signing Requests

### About Certificate Signing Requests

Obtaining an SSL certificate from a certificate authority (CA) requires creating a certificate signing request (CSR). Provide accurate information, especially the common name (CN), which must match the fully qualified domain name (FQDN) of the server. **Note:** If the CN does not match the server's FQDN, browsers may display a security warning.

### Create a Private Key and a CSR

Use this method if you want to use HTTPS (HTTP over TLS). Create a new folder, navigate into it, and build a 2048-bit private key (`domain.key`) and a CSR (`domain.csr`):

```bash
openssl req \
       -newkey rsa:2048 \
       -nodes \
       -keyout domain.key \
       -out domain.csr
```

**Note:** The `-nodes` flag ensures the private key is not encrypted. If you want to encrypt the private key, remove this flag.

### Create a CSR from an Existing Private Key

Use this method if you already have a private key.

```bash
openssl req \
       -key domain.key \
       -new \
       -out domain.csr
```

**Correction:** The original command incorrectly used `openssl x509`. The correct command for generating a CSR from an existing private key is shown above.

## Building SSL Certificates

### Create a Self-Signed Certificate

This is for use without a CA. Self-signed certificates are not trusted by browsers but can be used for internal testing or development.

```bash
openssl req \
       -newkey rsa:2048 \
       -nodes \
       -keyout domain.key \
       -x509 \
       -days 365 \
       -out domain.crt
```

**Note:** Replace `-days 365` with the desired validity period in days.

### Create a Self-Signed Certificate from an Existing Private Key

Use this method if you already have a private key.

```bash
openssl req \
       -key domain.key \
       -new \
       -x509 \
       -days 365 \
       -out domain.crt
```

### Create a Self-Signed Certificate from an Existing Private Key and CSR

This is for use without a CA.

```bash
openssl x509 \
       -signkey domain.key \
       -in domain.csr \
       -req \
       -days 365 \
       -out domain.crt
```

**Note:** Self-signed certificates do not provide the same level of trust as certificates issued by a CA.

## Inspect Certificates

### Show Signing Request

```bash
openssl req \
       -text \
       -noout \
       -verify \
       -in domain.csr
```

### Show Certificate

```bash
openssl x509 \
       -text \
       -noout \
       -in domain.crt
```

### Verify a Certificate was Signed by a CA

```bash
openssl verify \
       -verbose \
       -CAFile ca.crt domain.crt
```

**Correction:** Ensure that `ca.crt` contains the correct CA certificate chain. If the chain is incomplete, verification will fail.

## Private Keys

### Create a Private Key

Generate a 2048-bit private key:

```bash
openssl genrsa \
       -des3 \
       -out domain.key 2048
```

### Verify a Private Key

Check private key validity.

```bash
openssl rsa \
       -check \
       -in domain.key
```

### Verify a Private Key Matches a Certificate and CSR

Always double-check to ensure the private key, certificate, and CSR are consistent.

```bash
openssl rsa \
       -noout \
       -modulus \
       -in domain.key \
       | openssl md5
openssl x509 \
       -noout \
       -modulus \
       -in domain.crt \
       | openssl md5
openssl req \
       -noout \
       -modulus \
       -in domain.csr \
       | openssl md5
```

**Correction:** Split the commands for clarity. Each command checks the modulus of the respective file. Ensure all outputs match.

### Encrypt a Private Key

Set a password for encrypting the private key:

```bash
openssl rsa \
       -des3 \
       -in unencrypted.key \
       -out encrypted.key
```

### Decrypt a Private Key

Provide credentials for the encrypted key when prompted.

```bash
openssl rsa \
       -in encrypted.key \
       -out decrypted.key
```

## Convert Certificate Formats

### Convert PEM to DER

```bash
openssl x509 \
       -in domain.crt \
       -outform der \
       -out domain.der
```

### Convert DER to PEM

```bash
openssl x509 \
       -inform der \
       -in domain.der \
       -out domain.crt
```

### Convert PEM to PKCS7

```bash
openssl crl2pkcs7 -nocrl \
       -certfile domain.crt \
       -certfile ca-chain.crt \
       -out domain.p7b
```

### Convert PEM to PKCS12

```bash
openssl pkcs12 \
       -inkey domain.key \
       -in domain.crt \
       -export \
       -out domain.pfx
```

**Note:** The `-export` flag ensures the output is in PKCS12 format. You may be prompted to set a password for the `.pfx` file.

### Convert PKCS7 to PEM

```bash
openssl pkcs7 \
       -in domain.p7b \
       -print_certs \
       -out domain.crt
```

### Convert PKCS12 to PEM

```bash
openssl pkcs12 \
       -in domain.pfx \
       -nodes \
       -out domain.combined.crt
```

**Correction:** The `-nodes` flag ensures the private key in the output is not encrypted. If encryption is required, remove this flag.

## Automating SSL Certificate Renewal

### Using Certbot for Let's Encrypt

Certbot is a popular tool for automating SSL certificate issuance and renewal with Let's Encrypt. Install Certbot and run the following command to obtain and install a certificate:

```bash
sudo certbot --apache
```

For Nginx, use:

```bash
sudo certbot --nginx
```

To automate renewal, add the following cron job:

```bash
0 0 * * * /usr/bin/certbot renew --quiet
```

**Note:** Adjust the path to Certbot if it's installed elsewhere.

### Testing Renewal

Test the renewal process to ensure it works as expected:

```bash
sudo certbot renew --dry-run
```

---

## Troubleshooting SSL Issues

### Check Certificate Expiry

Verify when a certificate will expire:

```bash
openssl x509 \
       -enddate \
       -noout \
       -in domain.crt
```

### Debug SSL Connections

Use `openssl s_client` to debug SSL/TLS connections:

```bash
openssl s_client \
       -connect example.com:443
```

**Note:** Replace `example.com` with your domain. Look for errors or warnings in the output.

### Verify Certificate Chain

Ensure the certificate chain is complete:

```bash
openssl verify \
       -CAfile ca-chain.crt \
       domain.crt
```

### Check for Mixed Content

Use browser developer tools to identify mixed content issues (HTTP resources on an HTTPS page). Update all resources to use HTTPS.

### Common Error: "SSL Handshake Failed"

- Ensure the server supports the required TLS version.
- Verify the private key matches the certificate.
- Check for firewall or proxy interference.

---

## Securing SSL Configurations

### Enforce Strong Cipher Suites

To ensure secure connections, configure your web server to use strong cipher suites. Example for Nginx:

```nginx
ssl_protocols TLSv1.2 TLSv1.3;
ssl_ciphers HIGH:!aNULL:!MD5;
ssl_prefer_server_ciphers on;
```

For Apache, use:

```apache
SSLProtocol all -SSLv3 -TLSv1 -TLSv1.1
SSLCipherSuite HIGH:!aNULL:!MD5
SSLHonorCipherOrder on
```

### Enable HTTP/2

HTTP/2 improves performance by multiplexing multiple requests over a single connection. Enable it in your web server configuration:

For Nginx:

```nginx
listen 443 ssl http2;
```

For Apache:

```apache
Protocols h2 http/1.1
```

### Enable OCSP Stapling

OCSP stapling improves SSL performance by reducing the need for clients to query the certificate authority. Example for Nginx:

```nginx
ssl_stapling on;
ssl_stapling_verify on;
resolver 8.8.8.8 8.8.4.4 valid=300s;
resolver_timeout 5s;
```

For Apache:

```apache
SSLUseStapling on
SSLStaplingResponderTimeout 5
SSLStaplingReturnResponderErrors off
```

**Note:** Ensure your server has access to the CA's OCSP responder.

### Redirect HTTP to HTTPS

Force all traffic to use HTTPS by redirecting HTTP requests. Example for Nginx:

```nginx
server {
    listen 80;
    server_name example.com;
    return 301 https://$host$request_uri;
}
```

For Apache:

```apache
<VirtualHost *:80>
    ServerName example.com
    Redirect permanent / https://example.com/
</VirtualHost>
```

---

## Common OpenSSL Commands

This section provides a quick reference for frequently used OpenSSL commands. Each command is formatted for easy copy-paste.

---

### Generate a New Private Key

Create a 2048-bit RSA private key:

```bash
openssl genrsa -out private.key 2048
```

---

### Generate a New CSR

Create a CSR using an existing private key:

```bash
openssl req -new -key private.key -out request.csr
```

---

### Generate a Self-Signed Certificate

Create a self-signed certificate valid for 1 year (365 days):

```bash
openssl req -x509 -new -nodes -key private.key -sha256 -days 365 -out certificate.crt
```

---

### Convert Certificate Formats

#### PEM to DER

```bash
openssl x509 -in certificate.pem -outform der -out certificate.der
```

#### DER to PEM

```bash
openssl x509 -inform der -in certificate.der -out certificate.pem
```

#### PEM to PKCS12

```bash
openssl pkcs12 -export -out certificate.pfx -inkey private.key -in certificate.pem
```

#### PKCS12 to PEM

```bash
openssl pkcs12 -in certificate.pfx -nodes -out certificate.pem
```

---

### Verify Files

#### Verify a Private Key

```bash
openssl rsa -check -in private.key
```

#### Verify a CSR

```bash
openssl req -text -noout -verify -in request.csr
```

#### Verify a Certificate

```bash
openssl x509 -text -noout -in certificate.crt
```

#### Verify a Certificate Chain

```bash
openssl verify -CAfile ca-chain.crt certificate.crt
```

---

### Check Expiry Date of a Certificate

```bash
openssl x509 -enddate -noout -in certificate.crt
```

---

### Debug SSL/TLS Connections

Test an SSL/TLS connection to a server:

```bash
openssl s_client -connect example.com:443
```

**Note:** Replace `example.com` with your domain.

---

### Extract Information from Certificates

#### Extract Public Key from Certificate

```bash
openssl x509 -pubkey -noout -in certificate.crt > public.key
```

#### Extract Modulus from Certificate

```bash
openssl x509 -noout -modulus -in certificate.crt | openssl md5
```

#### Extract Modulus from Private Key

```bash
openssl rsa -noout -modulus -in private.key | openssl md5
```

#### Extract Modulus from CSR

```bash
openssl req -noout -modulus -in request.csr | openssl md5
```

**Note:** Ensure the modulus matches across the private key, CSR, and certificate.

---

### Encrypt and Decrypt Private Keys

#### Encrypt a Private Key

```bash
openssl rsa -des3 -in private.key -out encrypted.key
```

#### Decrypt a Private Key

```bash
openssl rsa -in encrypted.key -out private.key
```

---

### Create a Certificate Chain File

Combine a certificate and intermediate CA certificates into a single file:

```bash
cat certificate.crt intermediate.crt > fullchain.crt
```

---

### Generate Diffie-Hellman Parameters

Generate a DH parameter file for stronger security:

```bash
openssl dhparam -out dhparam.pem 2048
```

---

### Generate Random Data

Generate 32 bytes of random data (e.g., for a password or key):

```bash
openssl rand -hex 32
```

---

### Encrypt and Decrypt Files

#### Encrypt a File

```bash
openssl enc -aes-256-cbc -salt -in plaintext.txt -out encrypted.txt
```

#### Decrypt a File

```bash
openssl enc -aes-256-cbc -d -in encrypted.txt -out plaintext.txt
```

---

### Test OpenSSL Version

Check the installed OpenSSL version:

```bash
openssl version
```

---

## Advanced OpenSSL Commands

This section provides additional OpenSSL commands for advanced use cases. Each command is structured for easy copy-paste.

---

### Generate a CSR with Subject Alternative Names (SAN)

Create a CSR with SANs by using a configuration file:

1. Create a configuration file `san.cnf`:

```ini
[ req ]
default_bits       = 2048
distinguished_name = req_distinguished_name
req_extensions     = req_ext
prompt             = no

[ req_distinguished_name ]
C  = DE
ST = Baden-Württemberg
L  = Offenburg
O  = Example Company
CN = example.com

[ req_ext ]
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = example.com
DNS.2 = www.example.com
```

2. Generate the CSR:

```bash
openssl req -new -key private.key -out request.csr -config san.cnf
```

---

### Check Certificate Revocation Status

Check if a certificate has been revoked using OCSP:

```bash
openssl ocsp -issuer ca.crt -cert certificate.crt -url http://ocsp.example.com
```

---

### Create a Certificate Authority (CA)

#### Generate a CA Private Key

```bash
openssl genrsa -out ca.key 4096
```

#### Create a Self-Signed CA Certificate

```bash
openssl req -x509 -new -nodes -key ca.key -sha256 -days 3650 -out ca.crt
```

---

### Sign a Certificate with a CA

Sign a CSR with your CA to issue a certificate:

```bash
openssl x509 -req -in request.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out certificate.crt -days 365 -sha256
```

---

### Revoke a Certificate

Revoke a certificate using a CRL (Certificate Revocation List):

1. Create a CRL:

```bash
openssl ca -keyfile ca.key -cert ca.crt -gencrl -out ca.crl
```

2. Revoke the certificate:

```bash
openssl ca -keyfile ca.key -cert ca.crt -revoke certificate.crt
```

---

### View CRL Contents

```bash
openssl crl -in ca.crl -text -noout
```

---

### Combine Certificate and Private Key into a Single PEM File

```bash
cat certificate.crt private.key > combined.pem
```

---

### Split a PEM File into Certificate and Private Key

Extract the certificate:

```bash
awk '/BEGIN CERTIFICATE/,/END CERTIFICATE/' combined.pem > certificate.crt
```

Extract the private key:

```bash
awk '/BEGIN PRIVATE KEY/,/END PRIVATE KEY/' combined.pem > private.key
```

---

### Generate a Random Password

```bash
openssl rand -base64 16
```

---

### Test SSL/TLS Protocol Support

Check which protocols a server supports:

```bash
openssl s_client -connect example.com:443 -tls1_2
```

Replace `-tls1_2` with `-tls1_3`, `-tls1`, or `-ssl3` to test other protocols.

---

### Generate a SHA256 Hash of a File

```bash
openssl dgst -sha256 file.txt
```

---

### Encrypt and Decrypt Messages

#### Encrypt a Message

```bash
echo "Secret Message" | openssl rsautl -encrypt -inkey public.key -pubin -out encrypted.bin
```

#### Decrypt a Message

```bash
openssl rsautl -decrypt -inkey private.key -in encrypted.bin
```
