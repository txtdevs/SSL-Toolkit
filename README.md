# Welcome to TxtDevs - Certification Tools!

**TO DO LIST:**
- Generate certificate private key
- Generate certificate signing requests (CSR)
- Generate self-signed certificate
- Configure your webserver with your new certificate

**Our features:**
- Supports multiple common names and subject alternative names
- Supports elliptic curve cryptography
- OpenSSL commands and configurations

## Working Directory
```console
$ sudo mkdir -p /opt/ssltoolkit
$ mkdir ssl_file
```

## Extract our toolkit to Wokring Directory
```console
$ git clone git@github.com:txtdevs/SSL-Toolkit.git /opt/ssltoolkit
```
## Generate & Sign your Certification

### Generate Private Key
For ECC Private Key:

```console
openssl ecparam -name prime256v1 -param_enc named_curve -genkey -noout -out ssl_file/server.key
```

For RSA Private Key

```console
openssl genpkey -outform PEM -algorithm RSA -pkeyopt rsa_keygen_bits:4096 -out ssl_file/server.key
```
### Generate CSR file

```console
openssl req -new -nodes -key ssl_file/server.key \
-config /opt/ssltoolkit/cert_csrconfig.txt -nameopt utf8 -utf8 \
-out ssl_file/server.csr
```

### Generate Certificate file & Sign with our self-sign Intermediate CA

```console
openssl x509 -req -in ssl_file/server.csr -days 364 \
-CA /opt/ssltoolkit/txtdevs-intermediateCA/intermediateCA-cert.crt \
-CAkey /opt/ssltoolkit/txtdevs-intermediateCA/intermediateCA-priv.key \
-extfile /opt/ssltoolkit/cert_certconfig.txt -extensions req_ext \
-CAserial /tmp/tmp-15259lzrTYKtHyyeP -CAcreateserial -nameopt utf8 -sha256 \
-out ssl_file/server.crt
```
### Create Full Chain Certificate
```console
$ cat ssl_file/server.crt /opt/ssltoolkit/txtdevs-intermediateCA/intermediateCA-cert.crt > ssl_file/server_fullchain.crt
```

### Output Certificate
```
ssl_file
	├── server.crt
	├── server.csr
	└── server.key
```
## Apache
```
SSLEngine             on
SSLCertificateFile    /path/to/ssl_file/folder/server.crt

SSLCertificateKeyFile /path/to/ssl_file/folder/server.key
```
Additional SSL Configuration

```bash
# enable HTTP/2, if available
Protocols h2 http/1.1

# intermediate configuration
SSLProtocol             all -SSLv3 -TLSv1 -TLSv1.1
SSLCipherSuite          "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384"
SSLHonorCipherOrder     off
SSLSessionTickets       off
```

## Nginx
```
listen	443 ssl http2;
ssl_certificate     /path/to/ssl_file/folder/server.crt;
ssl_certificate_key /path/to/ssl_file/folder/server.key;
```
Additional SSL Configuration

```bash
# intermediate configuration
ssl_protocols TLSv1.2 TLSv1.3;
ssl_ciphers "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384";
ssl_prefer_server_ciphers off;
ssl_session_tickets off;
ssl_session_timeout 1d;
ssl_session_cache shared:MozSSL:10m;  # about 40000 sessions
```
