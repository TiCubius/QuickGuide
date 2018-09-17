# Laravel Setup Guide : Apache Subdirectory

## EXAMPLE OF Laravel Project

Assuming your **DocumentRoot** is `/var/www/html`, you need to put your Laravel Project in the **parent** directory, and create a Symbolic Link **from** the Laravel's `public` folder **to** a name of your choice, **in the `/var/www/html` directory**.

```bash
cd /var/www
git clone https://github.com/LSW-G1/GSB-laravel
cd /var/www/html
ln -s ../GSB-laravel/public gsbfrais-laravel
```

## EDIT THE APACHE2 CONFIG FILE

```bash
nano /etc/apache2/apache2.conf
```

Notice the **`AllowOverride All`** in the **`<Directory /var/www/>`**

```config
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

#<Directory /srv/>
        # Options Indexes FollowSymLinks
        # AllowOverride None
        # Require all granted
#</Directory>
```

## EDIT YOUR WEBSITE APACHE2 CONFIG

```bash
nano /etc/apache2/site-enabled/website.conf
```

Notice the **`AllowOverride All`** in **`<Directory />`**

```conf
<VirtualHost *:80>
        ServerAdmin admin@localhost.fr
        DocumentRoot /var/www/html/

        <Directory />
                Options FollowSymLinks
                AllowOverride All
        </Directory>

        LogLevel warn
        ErrorLog /var/log/apache2/error.log
        CustomLog /var/log/apache2/access.log combined
</VirtualHost>
```

## ENABLE THE REWRITE MODULE

```bash
a2enmod rewrite
```

## RESTART APACHE2 SERVICE

```bash
systemctl restart apache2
```

## ALL GOOD

You can now check if everything works as expected. Assuming the URL is **example.com** and the name you chose in the Symbolic Link is **laravel**, you should now be able to access the url : `http://example.com/laravel`
