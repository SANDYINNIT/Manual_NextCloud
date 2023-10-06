## Installation of apache2, mysql and some libraries in the container

1. Updating the machine.
```console
apt update
apt upgrade
```

2. Installation of the `apache2` web server.
```console
apt install -y apache2
```

3. Installation of the database server `mysql-server`.
```console
apt install -y mysql-server
```

4. Installation of some `php` libraries, the main language used by the applications.
```console
apt install -y php libapache2-mod-php
apt install -y php-fpm php-common php-mbstring php-xmlrpc php-soap php-gd php-xml php-intl php-mysql php-cli php-ldap php-zip php-curl
```

5. Restart the apache2 server
```console
systemctl restart apache2
```

## MySQL configuration
### Access the MySQL console
From a terminal where we are `root` we must execute the following command:
```console
root@elpuig:~$ mysql
```

### Creation of the database:
Once inside the MySQL console, run the commands to create the database. In this case we are creating a database with the name `bbdd`.

```console
CREATE DATABASE bbdd;
```

### Creation of a user
Note that the IP from which the database will be accessed will need to be identified, in this case `localhost`.

```console
CREATE USER 'usuario'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
```

### We give privileges to the user:
```console
GRANT ALL ON bbdd.* to 'user'@'localhost';
```

### Exit the database
```console
exit
```

### Let's test the connection to the database
From a terminal with a user without privileges we must be able to connect by entering our password.

```console
alumne@elpuig:~$ mysql -u usuario -p
```

**Extra: allow connection from a remote machine:**

For security, MySQL does not allow connections from other than localhost by default. If we want to change this behavior we must create another user who will access from a remote machine and will be identified by the user name and his IP. So there can be different users called `usuario` who connect from different machines.

**We change the default access to our machine:**

We allow access from any computer to our database.

``` console
cat /etc/mysql/mysql.conf.d/mysqld.cnf | grep bind-address
bind-address = 127.0.0.1
```

We need to change bind-address to `0.0.0.0`
``` console
bind-address = 0.0.0.0
```

### Restart the server
``` console
systemctl restart mysql
```

**Creation of a user to access from a remote machine:**

To access from a remote machine, we should create a new user identified by the username and the IP of the machine from which it will access.

``` console
CREATE USER 'usuario'@'192.168.22.100' IDENTIFIED WITH mysql_native_password BY 'password';
```

We need to give privileges to the user who will access from the remote machine.
To access from outside, we should also give privileges to the user on the other machine:

``` console
GRANT ALL ON bbdd.* to 'user'@'192.168.22.100';
```

## Applying permissions to our web applications
After the web application files are extracted to the `/var/www/html` directory, we apply the following permissions to the `/var/www/html` directory

``` console
cd /var/www/html
chmod -R 775 .
chown -R root:www-data .
```

And then afterwards you'll need the IP of your vagrant MV, by putting it into google as a link you'll be able to go to your next cloud server for example.

# Downloading Server

go to either NextCloud or NextCloud to download one of the servers. https://owncloud.com/download-server/

**_____________________________**
# Steps by steps to download NextCloud

Step 1:

``` console
vagrant up vagrant up --provider=virtualbox
```
``` console
vagrant ssh
```

Step 2:

``` console
sudo apt update
sudo apt upgrade
```

Step 3:

``` console
sudo apt install -y mysql-server
sudo apt install -y php libapache2-mod-php
```

Step 4:

``` console
sudo apt install -y php-fpm php-common php-mbstring php-xmlrpc php-soap php-gd php-xml php-intl php-mysql php-cli php-ldap php-zip php-curl

sudo systemctl restart apache2
```

Step 5:

**After doing this command scroll all the way up to do the DATABASE stuff.**

``` console
sudo mysql
```
afterwards
``` console
mysql -u usuario -p
```

Step 6:

In this part download the server for nextcloud and then move it to /var/www/html/, to do this:
**somehow firstly go all the way back through ``cd``**

``` console
cd vagrant
ls - to check the name
sudo mv NextCloud.zip /var/www/html/
cd /var/www/html/
```

Step 7:

``` console
sudo apt install zip
ls - to check the name
sudo unzip NextCloud.zip
sudo rm -rf index.html
sudo rm -rf NextCloud.zip
```

Step 8:

``` console
sudo chmod -R 775 .
sudo chown -R root:www-data .
exit ```

Step 9:

The drill is to do ``vagrant halt`` and that (dont forget to change the VagrantFile ip and stuff) and then ``vagrant up (etc)`` and do ``vagrant ssh``

``` console
sudo apt install -y php libapache2-mod-php

sudo apt install -y php-fpm php-common php-mbstring php-xmlrpc php-soap php-gd php-xml php-intl php-mysql php-cli php-ldap php-zip php-curl

sudo systemctl restart apache2

ip -c a
```
- **use the ip to get into the website**
