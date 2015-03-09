# Install Drupal7 on LEMP stack (Debian + Nginx + MySQL + PHP-FPM)


## Overview

My goal is to create a local developement environnment for D7 with LEMP stack (Debian + Nginx + MySQL + PHP-FPM).

This article will present two following parts :

* How to install the LEMP stack
* How to install Drupal7 on LEMP stack

## My environnement

I'm using Mac OS X Mavericks and virtualbox with debian wheezy installed.
For debian virtual machine, add one host-only adapter and set eth1 static ip : 

```
auto eth1
iface eth1 inet static
  address 192.168.56.101
  netmask 255.255.255.0
```

Add IP `192.168.56.101` to mac's hosts file `/etc/hosts`

```
192.168.56.101 kuangshi-yan.dev.net
```

So that the url `http://kuangshi-yan.dev.net` will point to IP `192.168.56.101`. 


### Install the LEMP Stack

The **LEMP stack**, which means Linux + Nginx + MySQL + PHP-FPM environnement.
Here I will show you how to install LEMP stack on Debian Wheezy. 

#### Step 1 - Update and upgrade apt-get

```
$ apt-get update
$ apt-get upgrade
```

#### Step 2 - Install MySQL

```
$ apt-get install mysql-server php5-mysql
```
During the installation, MySQL will ask you to set the root password. If you miss the chance to set the password while installing, it's very easy to set the password later from within the MySQL shell.

#### Step 3 - Install and configure PHP

```
$ apt-get install php5-fpm
```

Now we need to make one small change to php configuration. Open up `php.int` :

```
$ vim /etc/php5/fpm/php.ini
```

(If you haven't yet installed `vim`, use `nano` or `vi`)

Find the line, `cgi.fix_pathinfo=1`, and change the 1 to 0, like :

```
cgi.fix_pathinfo=0
```

Save and exit. 

We need to make another change in the php5-fpm configuration. Open up `www.conf` :

```
$ vim /etc/php5/fpm/pool.d/www.conf
```

Make sure to listen to a unix socket, instead of a TCP socket :

```
;listen = 127.0.0.1:9000
listen = /var/run/php5-fpm.sock
```

Save and exit. Restart php5-fpm : 

```
$ service php5-fpm restart
```

#### Step 4 - Install and configure Nginx
Once PHP and MySQL are all set up, we can move on to installing Nginx.
Run command : 

```
$ apt-get install nginx
```

Make a copy of default virtual host file : 

```
$ cp /etc/nginx/sites-available/default /etc/nginx/sites-available/default.bak
```

Open up the default virtual host file :

```
$ vim /etc/nginx/sites-available/defaut 
```

make these changes : 

* add index.php to the index line
* change server_name from localhost to your server domain name or IP address (e.g. kuangshi-yan.dev.net for me)
* uncomment the `location ~ \.php$ {` section.

So here is my default virtual host file : 

```
server {
        root /usr/share/nginx/www;
        index index.php index.html index.htm; # <-- add index.php

        server_name kuangshi-yan.dev.net; # <-- change server name to your server domain name

        location / {
                try_files $uri $uri/ /index.html;
        }

        location /doc/ {
                alias /usr/share/doc/;
                autoindex on;
                allow 127.0.0.1;
                allow ::1;
                deny all;
        }

        location ~ \.php$ {		# <-- uncomment this section if it's commented
                try_files $uri =404;
                fastcgi_split_path_info ^(.+\.php)(/.+)$;
                fastcgi_pass unix:/var/run/php5-fpm.sock; # <-- use unix socket
                fastcgi_index index.php;
                include fastcgi_params;
        }
}

```
Save and exit. Restart Nginx : 

```
$ service nginx restart
```

Now go to `http://kuangshi-yan.dev.net/` or (`http://your-server-domain or IP/`), you can see the `Welcome to nginx!` page displayed.

#### Step 5 - Install phpmyadmin

The next step is to install phpmyadmin. Run command : 

```
$ apt-get install phpmyadmin
```
During the installation, you will be prompted for some information. It will ask you which web server you would like the software to automatically configure. Since Nginx, the web server we are using, is not one of the available options, you can hit `TAB` to bypass this prompt. Next you will be prompted to ask if you would like `dbconfig-common` to configure a database for phpmyadmin to use. Select "Yes" to continue.
Once the phpmyadmin is installed successfully, go to Nginx's web root and link phpmyadmin : 

```
$ cd /usr/share/nginx/www
$ ln -s /user/share/phpmyadmin .
```
Now you should be able to go to `http://kuangshi-yan.dev.net/phpmyadmin` or (`http://your-server-domain or IP/phpmyadmin`).


### Install Drupal7 on LEMP Stack

Untill now, you've the LEMP stack installed on debian. We're going to install Drupal7 on LEMP stack.

#### Step 1 - Download Drupal 

Create the folder :

```
$ mkdir /var/www/html
```

Download drupal using `wget` or `drush`: 

```
$ wget http://ftp.drupal.org/files/projects/drupal-x.x.tar.gz
```

or 

```
$ drush dl drupal
```

Extract downloaded file : 

```
$ tar -xvzf drupal-x.x.tar.gz
```

Remove the compressed drupal file :

```
$ rm drupal-x.x.tar.gz
```

Rename the drupal directory : 

```
$ mv drupal-x.x/ dev.kuangshi-yan.net/
```

Move the drupal directory to `/var/www/html` folder : 

```
$ mv dev.kuangshi-yan.net/ /var/www/html/
```

Now navigate to the `/var/www/html/dev.kuangshi-yan.net/sites/default/` directory, make a copy of the `default.settings.php` named `settings.php` and make the directory and file writable to all : 

```
$ cd /var/www/html/dev.kuangshi-yan.net/sites/default/
$ cp default.settings.php settings.php
$ chmod 666 settings.php
$ chmod 777 /var/www/html/dev.kuangshi-yan.net/sites/default
```

#### Step 2 - Create database
Create a database for drupal site : 

```
$ mysql -u root -p

mysql> CREATE DATABASE drupal_db;
mysql> GRANT ALL PRIVILEGES ON drupal_db.* TO 'username'@'localhost' IDENTIFIED BY 'password';
```

#### Step 3 - Create an Nginx virtual host configuration for Drupal site

```
$ apt-get install nginx-doc

$ cd /etc/nginx/sites-availabe/
$ cp /usr/share/doc/nginx-doc/examples/drupal.gz .
$ gunzip drupal.gz
$ mv drupal kuangshi-yan_dev
$ cd /etc/nginx/sites-enabled/
$ ln -s ../sites-available/kuangshi-yan_dev .
```

In the `/etc/nginx/sites-enabled/kuangshi-yan_dev` file, modify `server_name` and `root`, and also `access_log` and `error_log` :

```
server {
	server_name kuangshi-yan.dev.net;
	root /var/www/html/dev.kuangshi-yan.net
	
	access_log /var/log/nginx/kuangshi-yan_dev.access.log;
    error_log /var/log/nginx/kuangshi-yan_dev.error.log info;
	
	[...]
}
```

Modify the name of the unix socket at `fastcgi_pass` line : 

```
location ~ \.php$ {
	fastcgi_split_path_info ^(.+\.php)(/.+)$;
	include fastcgi_params;
	fastcgi_intercept_errors on;
	fastcgi_index index.php;
	fastcgi_pass unix:/var/run/php5-fpm.sock;
}
```

Add the index line at the root location : 
 
```
location / {
	index index.php;
	try_files $uri $uri/ @rewrite;
}
```

Allow only localhost has access to txt and log files : 

```
location ~* \.(txt|log)$ {
	allow 127.0.0.1;
    deny all;
}
```

Add configuration for phpmyadmin : 

```
location /phpmyadmin {
	root /usr/share/;
	index index.php index.html index.htm;
	location ~ ^/phpmyadmin/(.+\.php)$ {
		try_files $uri =404;
		root /usr/share/;
		fastcgi_pass unix:/var/run/php5-fpm.sock;
		fastcgi_index index.php;
		fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
		include /etc/nginx/fastcgi_params;
	}
	location ~* ^/phpmyadmin/(.+\.(jpg|jpeg|gif|css|png|js|ico|html|xml|txt))$ {
		root /usr/share/;
	}
}

location /phpMyAdmin {
	rewrite ^/* /phpmyadmin last;
}
```

Disable the default virtual host configuration in `/etc/nginx/sites-enabled/`

```
$ rm /etc/nginx/sites-enabled/default
```


Now reload Nginx : 

```
$ service nginx reload
```

Go to `http://kuangshi-yan.dev.net` or (`http://your-server-domain or IP`)you will see the drupal installation page. 
Go for phpmyadmin page via `http://kuangshi-yan.dev.net/phpmyadmin` or (`http://your-server-domain or IP/phpmyadmin`).

## References : 

* [http://michal.karzynski.pl/blog/2013/06/07/running-drupal-nginx-memcached/](http://michal.karzynski.pl/blog/2013/06/07/running-drupal-nginx-memcached/)
* [http://dashohoxha.blogspot.fr/2012/10/using-nginx-as-web-server-for-drupal.html](http://dashohoxha.blogspot.fr/2012/10/using-nginx-as-web-server-for-drupal.html)
* [http://www.lonelycoder.be/nginx-php-fpm-mysql-phpmyadmin-on-ubuntu-12-04/](http://www.lonelycoder.be/nginx-php-fpm-mysql-phpmyadmin-on-ubuntu-12-04/)
* [https://www.drupal.org/documentation/install/developers](https://www.drupal.org/documentation/install/developers)
