first create rootCA certificate

>openssl genpkey -algorithm RSA -out root.key
>openssl req -new -key root.key -out root.csr
>openssl x509 -req -in root.csr -signkey root.key -out root.crt -days 3650
>openssl x509 -in root.crt -text -noout

second create subCA certificate

>openssl genpkey -algorithm RSA -out sub.key
>openssl req -new -key sub.key -out sub.csr
>openssl x509 -req -in sub.csr -CA root.crt -CAkey root.key -CAcreateserial -out sub.crt -days 1825
>openssl x509 -in sub.crt -text -noout

Third create personal certificate

>openssl genpkey -algorithm RSA -out server.key
>openssl req -new -key server.key -out server.csr
common name:cdac.ac.in
>openssl x509 -req -in server.csr -CA sub.crt -CAkey sub.key -CAcreateserial -out server.crt -days 365
>openssl x509 -in server.crt -text -noout

now certificates are created lest apply its to the cdac.ac.in site

>sudo apt update
>sudo apt install apache2
>sudo systemctl status apache2
>sudo ufw allow 'Apache'
>sudo a2enmod ssl
go to this file location and make one file server.conf
>/etc/apache2/sites-available$ sudo vim server.conf

<VirtualHost *:443>
        ServerName cdac.ac.in
        DocumentRoot /var/www/server

        SSLEngine on

        SSLCertificateFile      /home/ubuntu/server.crt
        SSLCertificateKeyFile   /home/ubuntu/server.key
        SSLCertificateChainFile /home/ubuntu/sub.crt
        SSLCACertificateFile    /home/ubuntu/root.crt

</VirtualHost>

go to this file location now
>/etc/apache2$ sudo vim apache2.conf

add this line at the end of the file 
ServerName      cdac.ac.in

go to this file now
>sudo vim /etc/hosts

add this line at the end of the file
127.0.0.1       cdac.ac.in

>sudo mkdir -p /var/www/server
>sudo chown -R www-data:www-data /var/www/server
>sudo a2ensite server.conf
>sudo systemctl restart apache2
>cd /var/www/html
>sudo vim index.html
<h1> this is certified site</h1>
>cd ~
>sudo cp root.crt /usr/local/share/ca-certificates/rootca.crt
>sudo update-ca-certificates
>openssl s_client -connect cdac.ac.in:443 -showcerts

you can see certificates which we created

---------------chaining----------------
>vim sub.ext
basicConstraints=critical,CA:TRUE,pathlen:0
keyUsage=critical,keyCertSign,cRLSign
subjectKeyidentifier=hash
authorityKeyidentifier=keyid,issuer

>openssl genpkey -algorithm RSA -out sub.key
>openssl req -new -key sub.key -out sub.csr
>openssl x509 -req -in sub.csr -CA root.crt -CAkey root.key -CAcreateserial -out sub.crt -days 1825 -ext sub.ext
>openssl x509 -in sub.crt -text -noout
>vim server.ext
basicConstrains=CA:FALSE
keyUsage = digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth

subjectAltName = @alt_names

[alt_names]
DNS.1 = cdac.in.ac

>openssl genpkey -algorithm RSA -out server.key
>openssl req -new -key server.key -out server.csr
>openssl x509 -req -in server.csr -CA sub.crt -CAkey sub.key -CAcreateserial -out server.crt -days 1825 -ext server.ext
>openssl x509 -in server.crt -text -noout
>cat server.crt sub.crt > fullchain.crt
>cd /etc/apache2/sites-available/
>sudo vim server.conf
<VirtualHost *:443>
        ServerName cdac.in.ac
        DocumentRoot /var/www/server

        SSLEngine on

        SSLCertificateFile      /home/ubuntu/fullchain.crt
        SSLCertificateKeyFile   /home/ubuntu/server.key

</VirtualHost>

>sudo systemctl restart apache2
Now check on browser again there will see all certificates
