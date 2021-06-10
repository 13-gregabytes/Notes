# SSL Certificates
## High level explanation
>Taken from https://serverfault.com/questions/9708/what-is-a-pem-file-and-how-does-it-differ-from-other-openssl-generated-key-file#answer-9717

SSL has been around for long enough you'd think that there would be agreed upon container formats. And you're right, there are. Too many standards as it happens. So this is what I know, and I'm sure others will chime in.

- **.csr** - This is a Certificate Signing Request. Some applications can generate these for submission to certificate-authorities. The actual format is PKCS10 which is defined in RFC 2986. It includes some/all of the key details of the requested certificate such as subject, organization, state, whatnot, as well as the public key of the certificate to get signed. These get signed by the CA and a certificate is returned. The returned certificate is the public certificate(which includes the public key but not the private key), which itself can be in a couple of formats.
- **.pem** - Defined in RFCs 1421 through 1424, this is a container format that may include just the public certificate (such as with Apache installs, and CA certificate files /etc/ssl/certs), or may include an entire certificate chain including public key, private key, and root certificates. Confusingly, it may also encode a CSR (e.g. as used here) as the PKCS10 format can be translated into PEM. The name is from Privacy Enhanced Mail (PEM), a failed method for secure email but the container format it used lives on, and is a base64 translation of the x509 ASN.1 keys.
- **.key** - This is a PEM formatted file containing just the private-key of a specific certificate and is merely a conventional name and not a standardized one. In Apache installs, this frequently resides in /etc/ssl/private. The rights on these files are very important, and some programs will refuse to load these certificates if they are set wrong.
- **.pkcs12 .pfx .p12** - Originally defined by RSA in the Public-Key Cryptography Standards(abbreviated PKCS), the "12" variant was originally enhanced by Microsoft, and later submitted as RFC 7292. This is a passworded container format that contains both public and private certificate pairs. Unlike .pem files, this container is fully encrypted. Openssl can turn this into a .pem file with both public and private keys: openssl pkcs12 -in file-to-convert.p12 -out converted-file.pem -nodes

A few other formats that show up from time to time:

- **.der** - A way to encode ASN.1 syntax in binary, a .pem file is just a Base64 encoded .der file. OpenSSL can convert these to .pem (openssl x509 -inform der -in to-convert.der -out converted.pem). Windows sees these as Certificate files. By default, Windows will export certificates as .DER formatted files with a different extension. Like...
- **.cert .cer .crt** - A .pem (or rarely .der) formatted file with a different extension, one that is recognized by Windows Explorer as a certificate, which .pem is not.
- **.p7b .keystore** - Defined in RFC 2315 as PKCS number 7, this is a format used by Windows for certificate interchange. Java understands these natively, and often uses .keystore as an extension instead. Unlike .pem style certificates, this format has a defined way to include certification-path certificates.
- **.crl** - A certificate revocation list. Certificate Authorities produce these as a way to de-authorize certificates before expiration. You can sometimes download them from CA websites.

In summary, there are four different ways to present certificates and their components:

- **PEM** - Governed by RFCs, its used preferentially by open-source software. It can have a variety of extensions (.pem, .key, .cer, .cert, more)
- **PKCS7** - An open standard used by Java and supported by Windows. Does not contain private key material.
- **PKCS12** - A Microsoft private standard that was later defined in an RFC that provides enhanced security versus the plain-text PEM format. This can contain private key material. Its used preferentially by Windows systems, and can be freely converted to PEM format through use of openssl.
- **DER** - The parent format of PEM. It's useful to think of it as a binary version of the base64-encoded PEM file. Not routinely used very much outside of Windows.

## Creating self signed certificates for dev
### Configure openssl
#### Find the file openssl.cnf
```
find / -name "openssl.cnf"
```
#### Copy the openssl.cnf to openssl.cnf.ca and openssl.cnf.cert
We copy the files so we can modify and use them to create our certificates without disturbing the actual openssl config.
```
cp [/etc/pki/tls/]openssl.cnf openssl.cnf.ca
cp [/etc/pki/tls/]openssl.cnf openssl.cnf.cert
```
#### Edit openssl.cnf.cert
Find `[ CA_default ]` section

Uncomment `copy_extensions = copy`

Find `[ req ]` section

Uncomment `req_extensions = v3_req # The extensions to add to a certificate request`

Add `[ alternate_names ]` section

Add `DNS.1 = *.domainname.net`

Find `[ v3_req ]` section

Add `subjectAltName=@alternate_names`

Find `[ v3_ca ]` section

Add `keyUsage = digitalSignature, keyEncipherment`

Add `subjectAltName=@alternate_names`

Add `[ SAN ]` section

Add `subjectAltName=@alternate_names`

>There are no changes to openssl.cnf.ca

#### Create /etc/ssl/openssl.cnf
```
[ alternate_names ]
DNS.1 = *.domainname.net
[ SAN ]
subjectAltName=@alternate_names
```
### Build a Certificate Authority (CA) root certificate
#### CA Private key
```
openssl genrsa -out ca.key 4096
```
#### Self-sign the certificate (make sure path to openssl.cnf.ca is correct)
>Valid for 5 years (1826 days)
```
openssl req -x509 -new -sha256 -days 1826 -key ca.key -subj '/C=UK/ST=Herts/L=StAlbans/OU=Dev/O=CHP/CN=StAlbansDev' -config /etc/pki/tls/openssl.cnf.ca -out ca.crt
```
#### Check the certificate does **NOT** contain X509v3 Subject Alternative Name
```
openssl x509 -text -noout -in ca.crt
```
### Build a certificate for your server
#### Private key
```
openssl genrsa -out privatekey.key 4096
```
#### Certificate signing request (make sure path to openssl.cnf.cert is correct)
```
openssl req -new -sha256 -key privatekey.key -subj '/C=UK/ST=Herts/L=StAlbans/OU=Dev/O=CHP/CN=*.domainname.net' -config /etc/pki/tls/openssl.cnf.cert -reqexts SAN -extensions SAN -out certsignreq.csr
```
#### Check the certificate **DOES** contain X509v3 Subject Alternative Name
```
openssl req -text -noout -in certsignreq.csr
```
### Sign the CSR with the CA root key
>Valid for 2 years (730 days)
```
openssl x509 -sha256 -req -days 730 -in certsignreq.csr -CA ca.crt -CAkey ca.key -CAcreateserial -extfile /etc/ssl/openssl.cnf -extensions SAN -out certificate.crt
```
Alternatively
```
openssl x509 -sha256 -req -days 730 -in certsignreq.csr -CA ca.crt -CAkey ca.key -set_serial 01 -extfile /etc/ssl/openssl.cnf -extensions SAN -out certificate.crt
```
#### Check the certificate does contain X509v3 Subject Alternative Name
```
openssl x509 -text -noout -in certificate.crt
```
### Convert the certificates into a p12 file for import into Chrome
```
openssl pkcs12 -export -inkey privatekey.key -in certificate.crt -certfile ca.crt -out bundle.p12
```
Alternatively
```
openssl pkcs12 -export -inkey privatekey.key -in certificate.crt -chain -CAfile ca.crt -caname root -out bundle.p12
```
### Build a keystore for your server
```
keytool -importkeystore -srckeystore bundle.p12 -srcstoretype PKCS12 -srcstorepass xxxxxxxx -destkeystore privatekey.keystore -deststorepass xxxxxxxx -destkeypass xxxxxxxx
```
### Merging your Certificate Authority (CA) root certificate into Java’s cacerts file
#### Find Java’s cacerts file
```
find / -type f -name "cacerts"
```
#### Merge created ca with java's ca file
```
keytool -keystore /<java_home>/lib/security/cacerts -storepass changeit -importcert -alias stalbansdev -file ca.crt
```
#### Verify the import was successful
```
keytool -list -keystore /<java_home>/lib/security/cacerts -storepass changeit > output.txt
grep -i "stalbans" output.txt
```
