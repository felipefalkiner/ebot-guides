

## **SETUP SERVER IN A LINUX MACHINE**

Most of the content of this guide can be found here: [https://forum.esport-tools.net/d/2-how-to-install-ebot-on-debian-ubuntu-step-by-step](https://forum.esport-tools.net/d/2-how-to-install-ebot-on-debian-ubuntu-step-by-step)

The reason I'm reposting here is because actually the eBot Forum has a lot of issues navigating through it (CORS, Mixed Content, etc) and don't know for how long this will be online. Also if you access the link direct, you can view the Forum post, but will not be able to see all the posts due to the problems I've mentioned before.

**Thanks vince52 for the original guide!**

### Server Requirements

 - Linux Machine with PHP5 support
 - PHP5 with MySQL, PDO and OpenSSL (also with enable-maintainer-zts and enable-sockets)
 - Apache using PHP5
 - MySQL Server
 - Node.JS version 0.12

**If you like Ubuntu you will need to use [Ubuntu 14.04](https://releases.ubuntu.com/14.04/)**

*I've also seen people installing it on CentOS 7.

And if you like Debian people on eBot's Forum recomendo Debian 7 or 8.*

On Ubuntu 16.04 and beyond PHP was updated to PHP7 and I couldn't install PHP5, even if I' ve compiled myself I still had problems in making Apache2 using PHP5 from what I've understand I would need `libapache2-mod-php5` but I couldn't get it from anywhere even from `ppa:ondrej/php`.

Also don't try to install it via WSL on Windows, since you can't install Ubuntu 14.04 via WSL (at least I couldn't find an image for it).

### Installation (Ubuntu and Debian)

**1 - Become Root**

Login to you Linux Machine and become root, usually you will use `sudo su` for this.

**2 - Update and Required Packages**

```sh
apt-get update
apt-get upgrade
apt-get install apache2 gcc make libxml2-dev autoconf ca-certificates unzip nodejs curl libcurl4-openssl-dev pkg-config libssl-dev screen
```

**3 - Install a custom PHP version**

First let's remove any other PHP version installed (just in case you have it, if you are unsure, run both commands anyway).
```sh
apt-get autoremove php php-dev php-cli 
apt-get autoremove php5 php5-dev php5-cli 
```
Now let's build the custom version:
```sh
mkdir /home/install
cd /home/install
wget http://be2.php.net/get/php-5.6.36.tar.bz2/from/this/mirror -O php-5.6.36.tar.bz2
tar -xjvf php-5.6.36.tar.bz2
cd php-5.6.36
./configure --prefix /usr/local --with-mysql --enable-maintainer-zts --enable-sockets --with-openssl --with-pdo-mysql 
make
make install
cd /home/install
wget http://pecl.php.net/get/pthreads-2.0.10.tgz
tar -xvzf pthreads-2.0.10.tgz
cd pthreads-2.0.10
/usr/local/bin/phpize
./configure
make
make install
echo 'date.timezone = 	America/Sao_Paulo' >> /usr/local/lib/php.ini
echo 'extension=pthreads.so' >> /usr/local/lib/php.ini
```

Then link php5 to Apache2:
```sh
apt-get install libapache2-mod-php5 
```

**4 - Install & Configure MySQL Server**

**4.1 - Installation**
```sh
apt-get install mysql-server php5-mysql
```
On the setup you will be prompted to type an password for root, remember this password (if you are not sure you can use a password generator or use `ebotv3`, careful if you are exposing your eBot to the internet).

This next section is optional if you know how to use MySQL via console, but if not, do this to easy manage your database.

Install phpMyAdmin
```sh
apt-get install phpmyadmin
```
Choose Yes, type a password for phpMyAdmin application (you will need this in case you choose to uninstall it later), then type the root password you've choose on the previous step, confirm the root password and then select Apache2.

Now let's add phpMyAdmin to Apache2, after the installation (if you don't know how to use nano, search a tutorial for this, nano is a text editor for Linux Terminals):
```sh
nano /etc/apache2/apache2.conf
```
Add the phpmyadmin config at the end of the file.
```sh
Include /etc/phpmyadmin/apache.conf
```
Restart apache:
```sh
service apache2 restart
```

**4.2 - Create the Database**

You can do this by Console or phpMyAdmin.
Fastest way = Console  
Graphic way = phpMyAdmin

**4.2.1 - Console**
Type
```sh
mysql -u root -p
```
Then type the root password defined previous.
After this you should be in MySQL console, now you can run this query:
```sql
create database ebotv3;
create user 'ebotv3'@'localhost' IDENTIFIED by 'ebotv3';
```
Please notice that after `IDENTIFIED by ` is the password for the database user, `ebotv3` is the default but you can change if you want it.

Now you need to give permissions to this user with the following query:

```sql
grant all privileges on ebotv3.* to 'ebotv3'@'localhost' with grant option;
exit
```
Now go to Step 5.

**4.2.2 - phpMyAdmin**
`SOON TM`

**5 - Install eBot-CSGO**

Now we are going to start installing eBot.
eBot is made of 2 applications, eBot-CSGO and eBot-CSGO-Web, the first one is responsible for connecting to the CSGO Server and sending commands, it's also a bridge between eBot-CSGO-Web and the CSGO Servers.

```sh
mkdir /home/ebot
cd /home/ebot
wget https://github.com/deStrO/eBot-CSGO/archive/master.zip
unzip master.zip
mv eBot-CSGO-master ebot-csgo
cd ebot-csgo
curl --silent --location https://deb.nodesource.com/setup_0.12 | bash -
apt-get install nodejs -y --force-yes
npm install socket.io@0.9.12 archiver@0.4.10 formidable
curl -sS https://getcomposer.org/installer | php
php composer.phar install
```

If you get any warning from Composer about installing as root, simply type Y and confirm.

Please notice that, this will install Node.js 0.12, I strongly recommend that you check you Node.js version using the following command:
```sh
node -v
```
You should expect the output as: `v0.12.18`

Now let's create a copy of the config file:

```sh
cp config/config.ini.smp config/config.ini
```

Now you need to edit the file config.ini located in `/config/config.ini`
For this you can use nano again (or any other text editor):
```sh
nano /home/ebot/ebot-csgo/config/config.ini
```

Thins you may need to edit:

 - MYSQL_PASS - only if you choose a password different from ebotv3 in step 4.2.1 or 4.2.2
 - BOT_IP - put your server IP (if it's a Lan server your local IP, ex. 192.168.0.1)
 - EXTERNAL_LOG_IP - use this in case your server isn't binded with the external IP (behind a NAT)

Now let's add the new maps to eBot, for this on the `config.ini` file go to the `[MAPS]` section, just create a new line following the pattern, here's how this section should look like:
```ini
[MAPS]
MAP[] = "de_dust2_se"
MAP[] = "de_nuke_se"
MAP[] = "de_inferno_se"
MAP[] = "de_mirage_ce"
MAP[] = "de_train_se"
MAP[] = "de_cache"
MAP[] = "de_season"
MAP[] = "de_dust2"
MAP[] = "de_nuke"
MAP[] = "de_inferno"
MAP[] = "de_train"
MAP[] = "de_mirage"
MAP[] = "de_cbble"
MAP[] = "de_overpass"
MAP[] = "de_vertigo"
MAP[] = "de_ancient"
```
We added Vertigo and Ancient by doing this, you can check the Active Duty Map Pool here: https://liquipedia.net/counterstrike/Portal:Maps

**6 - Install eBot-Web**

Now let's install the web interface so we can manage our eBot:
```sh
cd /home/ebot
rm -R master*
wget https://github.com/deStrO/eBot-CSGO-Web/archive/master.zip
unzip master.zip
mv eBot-CSGO-Web-master ebot-web
cd ebot-web
cp config/app_user.yml.default config/app_user.yml
```
Now you need to edit two config files `config/app_user.yml` and `config/databases.yml`.

```sh
 nano config/app_user.yml 
```
What to edit:

 - ebot_ip - put your IP
 - mode - if your ebot is exposed to the internet set net if not set lan

```sh
 nano config/databases.yml 
```
What to edit:

 - username - set ebotv3
 - password - set ebotv3
 - Only change the host if the database is installed in another machine!
 
Example:
```yml
# You can find more information about this file on the symfony website:
# http://www.symfony-project.org/reference/1_4/en/07-Databases

all:
  doctrine:
    class: sfDoctrineDatabase
    param:
      dsn:      mysql:host=127.0.0.1;dbname=ebotv3
      username: ebotv3
      password: ebotv3

```

Now let's finish the web installation:
```sh
mkdir cache

chown -R www-data *
chmod -R 777 cache

php symfony cc
php symfony doctrine:build --all --no-confirmation
php symfony guard:create-user --is-super-admin admin@ebot admin admin
```

**7 - Config Apache**
If you don't have a domain, use the `7.1 - Without sub-domain`, it will be easier

**7.1 - Without sub-domain**

Create the file /etc/apache2/conf.d/ebotv3
```sh
 nano /etc/apache2/sites-available/ebotv3.conf 
```
The file must have this inside:
```conf
Alias / /home/ebot/ebot-web/web/

<Directory /home/ebot/ebot-web/web/>
	AllowOverride All
	<IfVersion < 2.4>
		Order allow,deny
		allow from all
	</IfVersion>

	<IfVersion >= 2.4>
		Require all granted
	</IfVersion>
</Directory>
```
Now run the following commands:
```sh
a2enmod rewrite
a2ensite ebotv3.conf
service apache2 reload 
```

Now we need to edit the `.htaccess` file of the eBot Web:
```sh
nano /home/ebot/ebot-web/web/.hthtaccess
```
On line 8 you will find `#RewriteBase /` remove the `#` to uncomment the line and save the file.

Now go to 7.3

**7.2 - For a sub-domain**
(I haven't tested this method, just copied and pasted from the article at the beginning of the documment)

Create the file /etc/apache2/conf.d/ebotv3
```sh
 nano /etc/apache2/sites-available/ebotv3.conf 
```

and put this inside:
```conf
<VirtualHost *:80>
	#Edit your email
	ServerAdmin contact@mydomain.com

#Edit your sub-domain
ServerAlias ebot.mydomain.com

DocumentRoot /home/ebot/ebot-web/web

<Directory /home/ebot/ebot-web/web/>
	Options Indexes FollowSymLinks MultiViews
	AllowOverride All
	<IfVersion < 2.4>
		Order allow,deny
		allow from all
	</IfVersion>

	<IfVersion >= 2.4>
		Require all granted
	</IfVersion>
</Directory>
</VirtualHost>
```

Then:

```
a2enmod rewrite
a2ensite ebotv3.conf
service apache2 reload 
```

**7.3 - Discover your eBot-Panel**

You can now access the eBot Web via the browser (if you are unable to, something is wrong).

Do to this simple go to http://your-ip/ or http://ebot.your-domain.com/

If you managed to see the Panel, access the admin panel now by typing /admin.php at the end of the url http://your-ip/admin.php or http://ebot.your-domain.com/admin.php

The credentials are:
Username: admin
Password: admin

You can change this or create a new user when you login.

**7.4 - Delete /web/installation**
```sh
cd /home/ebot/ebot-web/web
rm -R installation
```

**8 - Start/Stop eBot Daemon**

Usually you just need to `php /home/ebot/ebot-csgo/bootstrap.php ` to start eBot, but I couldn't get done this way, so use the following script to install eBot as a Service on Debian/Ubuntu.

**8.1 - Installing eBot Daemon**
```sh
cd /home/install
wget https://raw.githubusercontent.com/vince52/eBot-initscript/master/ebotv3; mv ebotv3 /etc/init.d/ebot && chmod +x /etc/init.d/ebot
```
Then do this:
```sh
service ebot start
```
The output should be something like: `Starting eBot-V3: ebotv3 started (PID: 27640).`

Now go to the Admin Panel in the eBot Web and on the left menu you SHOULD see this:

![Image of the eBot Panel with everything working](https://i.imgur.com/VI4Z7Sm.png)

If the your output is not like this, something is wrong (maybe bot IPs on the config files).

**8.2 - Useful commands:**

**Start ebot-csgo:**
```
service ebot start
```
or
```
/etc/init.d/ebot start
```

**Stop ebot-csgo:**
```
service ebot stop
```
or
```
/etc/init.d/ebot stop
```

**Restart ebot-csgo:**
```
service ebot restart
```
or
```
/etc/init.d/ebot restart
```

**Status ebot-csgo:**
```
service ebot status
```
or
```
/etc/init.d/ebot status
```
