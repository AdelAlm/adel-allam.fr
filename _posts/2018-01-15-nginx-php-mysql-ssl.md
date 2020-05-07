---
layout: post
title: nginx + php + mysql + ssl/tls
categories: linux
---
<!--more-->

Mise en place d’un serveur web de type Nginx avec configuration de base avec PHP et MySql. Nginx est un serveur web d’origine russe crée en 2002 pour les besoins d’un site russe à très fort trafic. C’est un concurrent à Apache qui date de 1995. La dernière partie consistera à la mise en place d'un certificat SSL/TSL pour avoir du `https`.

---
## nginx
---

#### installation

```bash
# Debian
$ sudo apt update
$ sudo apt install nginx
$ sudo apt install mysql-client mysql-server
$ sudo apt install php7.0-fpm php7.0-mysql

# CentOS
$ sudo yum makecache
$ sudo yum install nginx
$ sudo yum install mariadb mariadb-server 
$ sudo yum install php-fpm php-mysql
```
> Pensez à désactiver les règles d’iptables sur CentOS ou dérivée !

#### gérer le service

```bash
# via nginx
$ sudo nginx # start server
$ sudo nginx –s stop # stop server
$ sudo nginx –s quit # stop server but reply to latests client
$ sudo nginx –t # check config files syntax
$ sudo nginx –s reload # reload config

# via systemd
$ sudo systemctl status nginx
$ sudo systemctl start nginx
$ sudo systemctl stop nginx
$ sudo systemctl enable nginx # start the service on boot
$ sudo systemctl reload nginx
```

#### configuration

Le fichier de configuration globale est `/etc/nginx/nginx.conf`, ce fichier inclus les différents fichiers de configuration de serveurs.

###### Debian

Fonctionnement avec 2 dossiers, `/etc/nginx/sites-available/*` qui contient tous les sites disponibles sur les serveurs, ainsi que `/etc/nginx/sites-enabled/*` qui contient les sites qui sont activés et accessibles par les visiteurs. L’activation d’un site consiste en la création d’un lien symbolique de `sites-available/` vers `sites-enables/` du fichier de configuration du serveur. C'est un fonctionnement proche de celui d'Apache.

###### CentOS

On utilise `/etc/nginx/conf.d/*` pour mettre la configuration de nos serveurs, les fichiers doivent être du type `*.conf` (se terminent par `.conf`) pour être pris en compte.

On peut bien sûr modifier l'architecture de base, et utiliser celle que l'on souhaite.

###### Configuration basique

```bash
server {
    listen 80;
    server_name example.fr www.example.fr;
    
    root /var/www/example.fr;
    index index.html;
}
```

---
## php
---

#### installation

```bash
# Debian
$ sudo apt install php7.0-fpm php7.0-mysql

# CentOS
$ sudo yum install php-fpm php-mysql
```

Nginx n’a pas de module intégré pour lire directement du PHP, mais possède un module (`fastcgi`) qui permet de se connecter à un worker qui va exécuter le code. Celui-ci va prendre les fichiers `*.php` en entrée, les traiter et renvoyer la version HTML compréhensible par Nginx. C’est PHP-FPM qui va être le worker.

Lors de son exécution PHP-FPM va mettre en place un socket d’écoute qui sera utilisé par le module `fastcgi` de Nginx. 

> On peut aussi configurer PHP-FPM pour qu’il écoute sur un port de la machine.

#### Configuration

###### Debian

Configuration de PHP-FPM: `/etc/php/7.0/fpm/pool.d/www.conf`.

On peut mettre `listen = 127.0.0.1:9000` pour écouter sur un port, ou utiliser un socket avec `listen = /run/php/php7.0-fpm.sock`.

Attention : il peut être parfois nécessaires de changer les droits du socket pour permettre à Nginx d’y avoir accès.

###### CentOS

Configuration de PHP-FPM: `/etc/php-fpm.d/www.conf`.

On peut mettre `listen = 127.0.0.1:9000` pour écouter sur un port, ou utiliser un socket avec `listen = /run/php-fpm/php5-fpm.sock`.


```bash
# démarrer le service PHP-FPM
$ sudo systemctl start php-fpm
# ou
$ sudo systemctl start php7.0-fpm
```


```bash
# code à ajouter à la configuration de nginx
location ~ \.php$ {
    fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;
    # fastcgi_pass 127.0.0.1:9000;
    fastcgi_index index.php;
    include fastcgi_params;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
}
```

---
## mysql
---

```bash
# Debian
$ sudo apt install mysql-client mysql-server 

# CentOS
$ sudo yum install mariadb mariadb-server 
```


```bash
# lancer mysql en tant que root en fournissant un mdp
$ mysql -uroot -p

# création d'une BDD
mysql> create database db_wp;

mysql> create user 'db_wp_user'@'localhost' identified by 'my_password';

mysql> grant all privileges on db_wp.* to 'db_wp_user'@'localhost';

mysql> flush provileges;

mysql> exit
```

Vous devez ensuite connecter votre nouvel utilisateur à la BDD dans votre code PHP. Pour accéder à votre BDD en ligne, vous pouvez utiliser phpMyAdmin, ou Adminer (très léger, simple fichier `.php`).

---
## certificat ssl/tls
---

#### install

```bash
$ git clone https://github.com/Neilpang/acme.sh.git
$ cd acme.sh/
$ ./acme.sh --install
```

#### create cert

```bash
$ acme.sh --issue -d adel-allam.fr \
                  -d www.adel-allam.fr \
                  -d monit.adel-allam.fr \
                  -w /var/www/html/mon-site.fr

# The certs will be placed in ~/.acme.sh/example.com/
```

#### install cert

```bash
# DO NOT use the certs files in ~/.acme.sh/ folder, they are for internal use only,
# the folder structure may change in the future.

$ acme.sh --install-cert -d ade-allam.fr
                         --key-file /home/adel/certs/adel-allam.fr/key.pem \
                         --fullchain-file /home/adel/certs/adel-allam.fr/cert.pem \
                         --reloadcmd "sudo systemctl reload nginx"

# renew with cron
0 0 * * * "/home/adel/.acme.sh"/acme.sh --cron --home "/home/adel/.acme.sh" > /dev/null
```

---
## example
---
```nginx
# redirection en HTTPS
server {
    listen 80;
    server_name mon-site.fr www.mon-site.fr;

    return 301 https://mon-site.fr$request_uri;
}

server {
    listen 443 ssl;
    server_name mon-site.fr;

    root /var/www/mon-site.fr/;
    index index.php index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    # optionnel (pour let's encrypt)
    location ~ /.well-know {
        allow all;
    }
    
    location ~ /\. {
        deny all;
        access_log off;
        log_not_found off;
    }

    # optionnel (pour PHP)
    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php-fpm/php5-fpm.sock;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }

    error_page 404 /404.html;

    error_log /var/logs/mon-site.fr/error.log;
    access_log /var/logs/mon-site.fr/access.log;

    # optionnel (pour SSL)
    ssl_certificate /var/certs/mon-site.fr/cert.pem;
    ssl_certificate_key /var/certs/mon-site.fr/key.pem;
}
```
