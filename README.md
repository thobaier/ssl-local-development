# SSL for local development
![alt text](https://www.udocx.com/sites/default/files/uploads/Wiki/Miscellaneous/ssl-logo.png)
> A workflow for generating SSL certificates for local development with CA root certificates  

![alt text](https://qph.fs.quoracdn.net/main-qimg-09e8c163a687f3006bd18e5ce4b11217)

## WORKFLOW
In this workflow we will generate a example certificate for the following dev-domains:  
`https://www.symfony/`  
`https:/symfony/`  
`https://mysql.symfony/`

Like a standard symfony application (with and without www in front) and a phpMyAdmin instance for quick reviews in the database layer (https://mysql.symfony/)

## 1) Clone this repository in a new folder ("ssl" in this example)
```bash
git clone https://github.com/thobaier/ssl-local-development ssl
```

## 2) Generate the root certificate.
Here we generate our own CA! This root certificate is used to generate the SSL certificates. We will import these root certificate as a certificate authority in our browser, after that the browser will trust all of our certificates (generated with this CA) in the future.  
  
#### Change directory to the repository folder
```bash
cd ssl
```

#### Generate your CA root certificate
```bash
openssl genrsa -des3 -out root/root.key 2048
```
**remember the passphrase you entered here - we will need it in the next steps!**

Now we got a new keyfile in the `root` directory of our repo called `root.key`

#### Generate the final root CA pem file
```bash
openssl req -x509 -new -nodes -key root/root.key -sha256 -days 1825 -out root/root.pem
```
Now we got a new keyfile in the `root` directory of our repo called `root.pem`. This pem file we will later import in our browsers.

## 3) Generate certificates for our applications by using the new CA root.

#### Generate the project keyfile
***Again***: in this example i will use `http://symfony/` as hostname/domain. How the files are called dont matter, but i prefer to call them like the project itself:
```bash
openssl genrsa -out symfony.key 2048
```
#### Generate the project csr file
```bash
openssl req -new -key symfony.key -out symfony.csr
```
The console will prompt you for some infos. Please make sure that you insert your domain name in this section:
```
Common Name (e.g. server FQDN or YOUR name) []: symfony
```

#### Edit your host names
In the file `config.template.ext` insert your domains as dns alt names:
```
...
[alt_names]
DNS.1 = www.symfony
DNS.2 = symfony
DNS.3 = mysql.symfony
```
#### Generate the final certificate file
No we will generate our certificate with our given root CA, our symfony.csr and the config template:
```bash
openssl x509 -req -in symfony.csr -CA root/root.pem -CAkey root/root.key -CAcreateserial \
-out symfony.crt -days 1825 -sha256 -extfile config.template.ext
```

# **That's it**
No you should have a `symfony.crt` and a `symfony.key` file in the repository base directory.
Those you can use in a nginx config:
```
...
ssl on;
ssl_certificate /etc/nginx/ssl/symfony.crt;
ssl_certificate_key /etc/nginx/ssl/symfony.key;
server_tokens off;
...
```
or in a apache vhost:
```
...
SSLEngine On
SSLCertificateFile /etc/apache2/symfony.crt
SSLCertificateKeyFile /etc/apache2/symfony.key
SSLCompression Off
SSLHonorCipherOrder on
...
```
#### Make SSL working correctly
For getting a nice green lock in your browser (without confirmation of untrusted certificate) just import the root certificate in your browsers:
(The root certificate is located under the root folder named `root.pem`)

**Chromium/Chrome:**
```
Einstellungen --> Erweitert --> Zertifikate verwalten --> Zertifizierungsstellen --> Import der dmc.pem Datei
```
**Firefox:**
```
Bearbeiten --> Einstellungen --> Zertifikate (einfach in suche eingeben) --> Zertifikate anzeigen --> Zertifizierungsstellen --> Import der dmc.pem Datei
```

I'am german, sorry for that :P

#### Trust system-wide
Find the SSL certificates location for your OS, in Linux mostly: `/etc/ssl/certs` and copy your **root** certificate (pem file) in there. After that find the `ca-certificates.crt` file and add your root certificate to the file with:
```bash
tee -a ca-certificates.crt < certs/your_root_cert_name.pem
```