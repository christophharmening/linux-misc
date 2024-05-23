# Create a local CA and Certs for your local Webservices

## Default Config

If you want to get the right default answers you can set this in
/etc/ssl/openssl.cnf in section [ req_distinguished_name ]

```
[ req_distinguished_name ]
countryName                     = Country Name (2 letter code)
countryName_default             = DE
countryName_min                 = 2
countryName_max                 = 2

stateOrProvinceName             = State or Province Name (full name)
stateOrProvinceName_default     = Niedersachsen

localityName                    = Locality Name (eg, city)
localityName_default            = myCity

0.organizationName              = Organization Name (eg, company)
0.organizationName_default      = MyCompany

organizationalUnitName          = Organizational Unit Name (eg, section)
#organizationalUnitName_default =

commonName                      = Common Name (e.g. server FQDN or YOUR name)
commonName_max                  = 64

emailAddress                    = Your Email Address
emailAddress_default            = myMail@mail.com
emailAddress_max                = 64
```

## Create CA

Create folderstrukture

```
mkdir -p /root/ca/{csr,crt,key,ext}
```

Go into folder and create ca keyfile

```
cd /root/ca
```

```
openssl genrsa -des3 -out key/myCA.key 2028
```

Check the permissions on file. It should only readable by root!

```
ls -l /root/ca/key/myCA.key
```
> -rw------- 1 root root 1751 Mai 22 15:17 /root/ca/key/myCA.key


Now create the ca certificate

```
openssl req -x509 -new -nodes -key key/myCA.key -sha256 -days 3650 -out myCA.pem
```

## Creating CA signed certificate for your website

Create Keyfile
```
openssl genrsa -out key/webserver.key 2048
```

Create certifiacte signing request
```
openssl req -new -key key/webserver.key -out csr/webserver.csr
```
Important here! Use the right fqdn from your webserver!

Here, we’ll create an X509 V3 certificate extension file, which is used to create the Subject Alternative Name (SAN) for the certificate.
```
nano ext/webserver.ext
```
```
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names

[alt_names]
DNS.1 = webservername
```

Now you can create the CA signed certificate
```
openssl x509 -req -in csr/webserver.csr -CA myCA.pem -CAkey key/myCA.key -CAcreateserial -out crt/webserver.crt -days 730 -sha256 -extfile ext/webserver.ext
```

## Create ssl config in apache2

Copy your Certs in right folder
```
cp /root/ca/crt/webserver.crt /etc/ssl/certs/
```
```
cp /root/ca/key/webserver.key /etc/ssl/private/
```

Enable ssl in apache2
```
a2enmod ssl
```
```
systemctl restart apache2
```

Create your virtual-host file. Here is used the default 000-ssl.conf
```
nano /etc/apache2/site-avaiable/000-ssl.conf
```
```
<VirtualHost *:443>
    ServerName webservername

    DocumentRoot /var/www/html

    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/webserver.crt
    SSLCertificateKeyFile /etc/ssl/private/webserver.key

    DirectoryIndex index.php
    AddDefaultCharset UTF-8

    <Directory /var/www/html
        AllowOverride None
        <IfVersion >= 2.3>
            Require all granted
        </IfVersion>
        <IfVersion < 2.3>
            Order Deny,Allow
            Allow from all
        </IfVersion>
    </Directory>

    LogLevel warn
</VirtualHost>
```

Reload apache
```
systemctl reload apache2
```

Now copy the myCA.pem file to your Client and add these Certificate to your Browser.


