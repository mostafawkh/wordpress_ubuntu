
# LAMP stack and configure wordpress on Ubuntu 20.04  



## Step1:Update packages!

```
sudo apt update

sudo apt upgrade
```





## Step2: Install Apache Web Server
for installation use following command:
```
 sudo apt install -y apache2 apache2-utils
 ```
 
 while it installed , Server should start automatically.
 check the status with the follwing command:
 ```
 systemctl status apache
 ```

 the output should be somthing like:
 ```
 apache2.service - The Apache HTTP Server
     Loaded: loaded (/lib/systemd/system/apache2.service; enabled; vendor preset: enabled)
     Active: active (running) since Sat 2020-04-11 11:31:31 CST; 2s ago
       Docs: https://httpd.apache.org/docs/2.4/
    Process: 53003 ExecStart=/usr/sbin/apachectl start (code=exited, status=0/SUCCESS)
   Main PID: 53011 (apache2)...
 ```

 if the server status is not active you can use below command in order to activate the server:
 ```
 sudo systemctl start apache2
 ```

 Now the server is running and you should see the PHP entry page in your localhost!
 
 Then we need to set www-data (Apache user) as the owner of document root (otherwise known as web root). By default it’s owned by the root user.

 ```
 sudo chown www-data:www-data /var/www/html/ -R
 ```

## Step 3: Install MariaDB Database Server

install with following command:
```
sudo apt install mariadb-server mariadb-client

```
After it’s installed, MariaDB server should be automatically started. Use systemctl to check its status.
```
systemctl status mariadb
```

if it’s not running, start it with this command:
```
sudo systemctl start mariadb
```

while the installation completed, you can login into your mariaDB local server:
```
sudo mariadb -u root
```

## Step 4: Install PHP7.4
 commands for php and some requirements installation:
 ```
sudo apt install php7.4 libapache2-mod-php7.4 php7.4-mysql php-common php7.4-cli php7.4-common php7.4-json php7.4-opcache php7.4-readline
 ```

Enable the Apache php7.4 module then restart Apache Web server.

```
sudo a2enmod php7.4

sudo systemctl restart apache2
```

### Run PHP-FPM with Apache
```
sudo a2dismod php7.4

sudo apt install php7.4-fpm

sudo a2enmod proxy_fcgi setenvif

sudo a2enconf php7.4-fpm

sudo systemctl restart apache2
```

## Step 5: Install phpMyAdmin

### step 5-1:Download and Install phpMyAdmin on Ubuntu 20.04
```
sudo apt install phpmyadmin
```

Then you need to configure phpmyadmin...

Now run the following command to check if the /etc/apache2/conf-enabled/phpmyadmin.conf file exists.
```
file /etc/apache2/conf-enabled/phpmyadmin.conf
```
If there’s no error in the installation process, you should see the following command output.
```
/etc/apache2/conf-enabled/phpmyadmin.conf: symbolic link to ../conf-available/phpmyadmin.conf
```

If this file doesn’t exist on your server, it’s likely that you didn’t select Apache web server in the phpMyAdmin setup wizard. You can fix it with the following commands.
```
sudo ln -s /etc/phpmyadmin/apache.conf /etc/apache2/conf-available/phpmyadmin.conf

sudo a2enconf phpmyadmin

sudo systemctl reload apache2
```


## Step 6:Install and configure WordPress

### Step 6-1:Install requirements
```
sudo apt install apache2 ghostscript libapache2-mod-php 
```

### Step 6-2:Create the installation directory and download the file from https://wordpress.org:
```
sudo mkdir -p /srv/www
sudo chown www-data: /srv/www
curl https://wordpress.org/latest.tar.gz | sudo -u www-data tar zx -C /srv/www
```


### Step 6-3:Configure Apache for WordPress
Create Apache site for WordPress. Create /etc/apache2/sites-available/wordpress.conf with following lines:
```
<VirtualHost *:80>
    DocumentRoot /srv/www/wordpress
    <Directory /srv/www/wordpress>
        Options FollowSymLinks
        AllowOverride Limit Options FileInfo
        DirectoryIndex index.php
        Require all granted
    </Directory>
    <Directory /srv/www/wordpress/wp-content>
        Options FollowSymLinks
        Require all granted
    </Directory>
</VirtualHost>
```

Enable the site with:
```
sudo a2ensite wordpress
```

Enable URL rewriting with:
```
sudo a2enmod rewrite

```
Disable the default “It Works” site with:
```
sudo a2dissite 000-default
```

Finally, reload apache2 to apply all these changes:

```
sudo service apache2 reload
```
To configure WordPress, we need to create MySQL database.
```
$ sudo mysql -u root

CREATE DATABASE wordpress;

CREATE USER wordpress@localhost IDENTIFIED BY '<your-password>';

GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP,ALTER ON wordpress.*TO wordpress@localhost;

FLUSH PRIVILEGES;
```

Now, let’s configure WordPress to use this database. First, copy the sample configuration file to wp-config.php:
```
sudo -u www-data cp /srv/www/wordpress/wp-config-sample.php /srv/www/wordpress/wp-config.php
```

Next, set the database credentials in the configuration file (do not replace database_name_here or username_here in the commands below. Do replace <your-password> with your database password.):
```
sudo -u www-data sed -i 's/database_name_here/wordpress/' /srv/www/wordpress/wp-config.php
sudo -u www-data sed -i 's/username_here/wordpress/' /srv/www/wordpress/wp-config.php
sudo -u www-data sed -i 's/password_here/<your-password>/' /srv/www/wordpress/wp-config.php
```

Finally, in a terminal session open the configuration file in nano:
```
sudo -u www-data nano /srv/www/wordpress/wp-config.php
```
Find the following:
```
define( 'AUTH_KEY',         'put your unique phrase here' );
define( 'SECURE_AUTH_KEY',  'put your unique phrase here' );
define( 'LOGGED_IN_KEY',    'put your unique phrase here' );
define( 'NONCE_KEY',        'put your unique phrase here' );
define( 'AUTH_SALT',        'put your unique phrase here' );
define( 'SECURE_AUTH_SALT', 'put your unique phrase here' );
define( 'LOGGED_IN_SALT',   'put your unique phrase here' );
define( 'NONCE_SALT',       'put your unique phrase here' );
```
Delete those lines (ctrl+k will delete a line each time you press the sequence). Then replace with the content of https://api.wordpress.org/secret-key/1.1/salt/. (This address is a randomiser that returns completely random keys each time it is opened.) This step is important to ensure that your site is not vulnerable to “known secrets” attacks.


Thats it .
## Enjoy !




