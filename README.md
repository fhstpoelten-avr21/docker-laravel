# docker-laravel
A simple multi-container docker-compose for laravel applications

This configuration is using the following images from [DockerHub](https://hub.docker.com/):
* __PHP__: php:8.0-fpm (including xDebug)
* __Webserver__: nginx:latest
* __Database__: mariadb:10.5.8
* __Cache__: redis
* __Email__: mailhog


## Useful commands
```
# create and start containers without logging
docker-compose up -d
 
# create and start containers with log
docker-compose up
 
# stop and remove containers, networks, images, and volumes
docker-compose down
 
# step into the php container
docker-compose exec app bash
 
# step into the mysql container
docker-compose exec mysql bash
 
# list containers
docker-compose ps
 
# remove unused data
docker system prune
```

<br>

## DB

_jdbc:mariadb://localhost:8001/my_app_db_
```
HOST=db
PORT=3306
DATABASE=my_app_db
USERNAME=root
PASSWORD=root
```

### Laravel
Add this DB configuration in your __.env__ file:
```
DB_CONNECTION=mysql #defined in /app/config/database.php
DB_HOST=db
DB_PORT=3306
DB_DATABASE=my_app_db
DB_USERNAME=root
DB_PASSWORD=root
```

<br>

## SSL
Create a self-signed certificate for using https on localhost.

### Create Certificate Authority
To sign your own certificate, you need your own certification authority.

1. Create private key and set password (needed later)
   ```
   openssl genrsa -des3 -out grandgarageDev.key 2048
   ```

2. Create root certificate and set common_name=connector.dev.grandgarage.eu (let other fields empty)
   ```
   openssl req -x509 -new -nodes -key grandgarageDev.key -sha256 -days 825 -out grandgarageDev.pem
   ```

### Register in Browser
**Chrome:** Settings/Security/Certificates verwalten

**Firefox:** Settings/Security/ZertifikateCertificates/Import


### Create Self Signed Key
1. Create key
    ```
   openssl genrsa -out connector.key 2048
    ```

2. Create Certificate Signing Requests (CSR)
    ```
   openssl req -new -key connector.key -out connector.csr -subj '/CN=grandgarageDev/O=grandgarageDev/OU=IT' -extensions SAN -reqexts SAN -config <( \
   printf "[SAN]\nsubjectAltName=DNS:connector.dev.grandgarage.eu
   \n[dn]\nCN=grandgarageDev\n[req]\ndistinguished_name = dn\n[EXT]\nauthorityKeyIdentifier=keyid,issuer\nbasicConstraints=CA:FALSE\nsubjectAltName=DNS:connector.dev.grandgarage.eu
   \nkeyUsage=digitalSignature,nonRepudiation,keyEncipherment,dataEncipherment\nextendedKeyUsage=serverAuth")
    ```

3. Create file connector.ext and insert the following content
    ```
   authorityKeyIdentifier=keyid,issuer
   basicConstraints=CA:FALSE
   keyUsage=digitalSignature,nonRepudiation,keyEncipherment,dataEncipherment
   subjectAltName = @alt_names
   [alt_names]
   DNS.1 = connector.dev.grandgarage.eu
    ```

4. Create key by using the self created certificate authority
    ```
   openssl x509 -req -in connector.csr -out connector.crt -CA grandgarageDev.pem -CAkey grandgarageDev.key -CAcreateserial \
   -sha256 -days 3650 \
   -extfile connector.ext
    ```


### Debugging
For debugging xdebug has been already installed in _app.dockerfile_. To use it in PHPstorm:

1. __Phpstorm Port:__ Set Port 9003 in _Phpstorm/Preferences/PHP/Debug_

2. __Phpstorm Path config:__ _Phpstorm/Run/EditConfiguration/PHP Web Page/xdebug_.
   * Add configuration
   * Domain = https://connector.dev.grandgarage.eu
   * Port = 443
   * Use path mapping: /path/to/application => /var/www/





Browser Plugin
Install browser extension and activate:

Chrome: https://chrome.google.com/webstore/detail/xdebug-helper/eadndfjplgieldjbigjakmdgkmoaaaoc?hl=de
Firefox: https://addons.mozilla.org/de/firefox/addon/xdebug-helper-for-firefox/
