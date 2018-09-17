# LARAVEL - Mise en production d'un projet avec Apache2

Ce guide vous permettra de mettre en production un projet Laravel sur un serveur Web Apache2.

- [LARAVEL - Mise en production d'un projet avec Apache2](#laravel---mise-en-production-dun-projet-avec-apache2)
- [Pré-requis](#pr%C3%A9-requis)
- [ETAPE 01 - Installation du Projet Laravel](#etape-01---installation-du-projet-laravel)
- [ETAPE 02 - Configuration d'Apache 2](#etape-02---configuration-dapache-2)
- [ETAPE 03 - Tester la configurarion](#etape-03---tester-la-configurarion)

# Pré-requis

Afin de suivre ce guide, vous devez:
- Installer le paquet `apache2`
- Installer les paquets php nécessaires au fonctionnement de Laravel
- Posséder un projet Laravel (il peut s'agir de l'installation de base)
- Posséder un nom de domaine (dans notre exemple, il s'agira de `exemple.fr`)

# ETAPE 01 - Installation du Projet Laravel

Notre projet Laravel se situera dans le dossier `/var/www/html/laravel`

```bash
cd /var/www/html
composer create-project laravel/laravel laravel
```

# ETAPE 02 - Configuration d'Apache 2

La configuration des hôtes virtuels (Virtual Hosts) sur Apache2 se situe dans le dossier `/etc/apache2/sites-enabled`

Créez un fichier dans ce dossier :
```bash
nano /etc/apache2/sites-enabled/exemple.fr.conf
```

Ce fichier contiendra la configuration suivante : 
```apache
<VirtualHost *:80>
    ServerName "exemple.fr"
    DocumentRoot /var/www/html/laravel/public

    <Directory />
        Options FollowSymLinks
        AllowOverride All
    </Directory>

    LogLevel warn
    ErrorLog /var/log/apache2/error.log
    CustomLog /var/log/apache2/access.log combined
</VirtualHost>
```

**EXPLICATIONS**
- `VirtualHost *:80` indique le port sur lequel la configuration est active
- `ServerName "exemple.fr"` indique le nom de domaine sur lequel la configuration est active
- `DocumentRoot /var/www/html/laravel/public` indique le dossier dans lequel se situe notre projet. On souhaite que le visiteur atteigne immédiatement le dossier public.
- `<Directory />` indique la configuration spécifique lorsque le visiteur arrive sur le nom de domaine
- `Options FollowSymLinks` indique à Apache2 de suivre les liens symboliques (ex: dossier storage)
- `AllowOverride All` indique à Apache2 d'utiliser les fichiers `.htaccess`


Il faut également modifier la configuration du fichier `/etc/apache2/apache2.conf`
```bash
nano /etc/apache2/apache2.conf`
```

Ce fichier contient la configuration générale d'Apache2.\
Déplacez-vous vers le bas de fichier et retrouvez cette section:

```apache
<Directory />
    Options FollowSymLinks
    AllowOverride None
    Require all denied
</Directory>

<Directory /usr/share>
    AllowOverride None
    Require all granted
</Directory>

<Directory /var/www/>
    Options Indexes FollowSymLinks
    AllowOverride None
    Require all granted
</Directory>
```

Il faut modifier la configuration afin de permettre l'utilisation des `.htaccess` sur le `<Directory />` et `<Directory /var/www/>`: 

```apache
<Directory />
    Options FollowSymLinks
    AllowOverride All
    Require all denied
</Directory>

<Directory /usr/share>
    AllowOverride None
    Require all granted
</Directory>

<Directory /var/www/>
    Options Indexes FollowSymLinks
    AllowOverride All
    Require all granted
</Directory>
```
**Remarquez le changement de la valeur d'AllowOverride !**

Il faut également activer un module Apache :
```bash
a2enmod rewrite
```

Maintenant que la configuration de Apache2 est complète, nous pouvons la recharger :
```bash
service apache2 restart
```

# ETAPE 03 - Tester la configurarion

Votre projet devrait maintenant être fonctionnel. \
Afin de vérifier votre configuration, essayer d'accéder à une page ne correspondant à aucune route, située dans un "sous-dossier" : `exemple.fr/test/page/not/found`

Si le message d'erreur est celle d'Apache2, la configuration n'est pas correcte, s'il s'agit de celle de Laravel, votre configuration est complète.

![Page d'erreur Laravel](images/nginx.png)