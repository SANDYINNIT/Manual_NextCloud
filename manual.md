1. [Introducció a Vagrant (Ingles)](https://github.com/SANDYINNIT/Manual_NextCloud/blob/main/manual.md#introducci%C3%B3-a-vagrant-ingles)
2. [Prepare the MV once created (NOT REALLY IMPORTANT)](https://github.com/SANDYINNIT/Manual_NextCloud/blob/main/manual.md#prepare-the-mv-once-created-not-really-imporant)
3. [Installation of apache 2, mysql and some libraries in the container](https://github.com/SANDYINNIT/Manual_NextCloud/blob/main/manual.md#installation-of-apache2-mysql-and-some-libraries-in-the-container)
4. [Installation and configuration of clouds](https://github.com/SANDYINNIT/Manual_NextCloud/blob/main/manual.md#installation-and-configuration-of-clouds)
5. [MySQL configration](https://github.com/SANDYINNIT/Manual_NextCloud/blob/main/manual.md#mysql-configuration)
6. [Creation of DATABASE](https://github.com/SANDYINNIT/Manual_NextCloud/blob/main/manual.md#creation-of-the-database)

# Introducció a Vagrant (Ingles)

`Vagrant` is a free tool for creating and working with development environments. These development environments are supported on some virtualization tool such as `VirtualBox`, `libvirt` or `Docker`, so in practice `Vagrant` will allow us to define our infrastructure in a `Vagrantfile` file.

From the `Vagrantfile` the vagrant tool will take care of:

* Download the images
* Build the MVs with the specified configuration
* Execute the necessary tasks to install software inside it, create users, ...
* The management (creation, ignition, storage, detention and destruction) of MVs
* Share the project directory with the MVs
* Manage access with ssh

## A directory for the project

For each project `vagrant` uses a directory, as an example we can create the `example` directory to work with an `Ubuntu 22.04 Jammy Jellyfish` MV.

```console
[alumne@elpuig ~]$ mkdir example
[alumne@elpuig ~]$ cd example/
```

The project configuration is written to the `Vagrantfile` file which we can create directly with the `vagrant init <box>` command pointing to one of the MVs found in `Vagrant Cloud`.

**Do not do it if you already have `Vagrantfile` in the folder, just go to the next 2 step**
```console
[alumne@elpuig example]$ vagrant init ubuntu/jammy64
A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.
[alumne@elpuig example]$ ll
total 4
-rw-rw-r--. 1 alumne alumne 3020 15 jul. 14:23 Vagrantfile
[alumne@elpuig example]$
```

Now we can bring up the whole infrastructure (by adding the `--provider=virtualbox` parameter because on some machines the default provider is `libvirt`) with `vagrant up`:

```console
[alumne@elpuig example]$ vagrant up --provider=virtualbox
==> default: Clearing any previously set network interfaces...
==> default: Preparing network interfaces based on configuration...
    default: Adapter 1: nat
==> default: Forwarding ports...
    default: 22 (guest) => 2222 (host) (adapter 1)
==> default: Running 'pre-boot' VM customizations...
==> default: Booting VM...
==> default: Waiting for machine to boot. This may take a few minutes...
    default: SSH address: 127.0.0.1:2222
    default: SSH username: vagrant
    default: SSH auth method: private key
    default: 
    default: Vagrant insecure key detected. Vagrant will automatically replace
    default: this with a newly generated keypair for better security.
    default: 
    default: Inserting generated public key within guest...
    default: Removing insecure key from the guest if it's present...
    default: Key inserted! Disconnecting and reconnecting using new SSH key...
==> default: Machine booted and ready!
==> default: Checking for guest additions in VM...
    default: The guest additions on this VM do not match the installed version of
    default: VirtualBox! In most cases this is fine, but in rare cases it can
    default: prevent things such as shared folders from working properly. If you see
    default: shared folder errors, please make sure the guest additions within the
    default: virtual machine match the version of VirtualBox you have installed on
    default: your host and reload your VM.
    default: 
    default: Guest Additions Version: 6.0.0 r127566
    default: VirtualBox Version: 6.1
==> default: Mounting shared folders...
    default: /vagrant => /home/alumne/example

[alumne@elpuig example]$
```

To enter into the virtual box, you must do the following:

```console
[alumne@elpuig example]$ vagrant ssh
Welcome to Ubuntu 20.04.2 LTS (GNU/Linux 5.4.0-77-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Thu Jul 15 12:38:47 UTC 2021

  System load:  0.0               Processes:               110
  Usage of /:   3.2% of 38.71GB   Users logged in:         0
  Memory usage: 19%               IPv4 address for enp0s3: 10.0.2.15
  Swap usage:   0%


1 update can be applied immediately.
To see these additional updates run: apt list --upgradable


vagrant@ubuntu-jammy:~$
```
```console
vagrant@ubuntu-jammy:~$ ll /vagrant
total 8
drwxrwxr-x  1 vagrant vagrant   38 Jul 15 12:32 ./
drwxr-xr-x 20 root    root    4096 Jul 15 12:33 ../
drwxrwxr-x  1 vagrant vagrant   32 Jul 15 12:32 .vagrant/
-rw-rw-r--  1 vagrant vagrant 3020 Jul 15 12:23 Vagrantfile
vagrant@ubuntu-jammy:~$
```

The `vagrant status` command will show us information about the state of our MVs and we can stop them with `vagrant halt` or suspend them with `vagrant suspend`. In any case, they can be turned on again with `vagrant up`.

When the MVs are no longer needed they can be deleted with `vagrant destroy`.

### Prepare the MV once created (NOT REALLY IMPORANT)

Once the MV is created from the official image, it will need to be prepared in some way to fulfill the function expected by the user.

For this task `vagrant` uses different provisioners including the `shell` and `Ansible` interpreter. With them you can specify any automatic task that needs to be done to prepare the MVs.

For example, if we wanted to install an `apache` web server in our MV we could find a commented example in our `Vagrantfile`.

If we uncomment the following lines:

```console
config.vm.provision "shell", inline: <<-SHELL
  apt-get update
  apt-get install -y apache2
SHELL
```

The `apache` web server will be installed automatically when creating the environment, although without adding a redirect for a port or a new interface it will still not be possible to access the web server from our machine.

**PORT FORWARDING:**

The easiest way to access a service, such as our Apache server, running on the MV is to forward traffic from a port on our physical machine to the MV.

As usual, we will find an example prepared in our Vagrantfile that precisely redirects the traffic from port `8080 TCP` of our physical machine to port `80 TCP` of the MV.

To make the apache server of the MV visible at http://localhost:8080 of the physical machine we will only have to uncomment the following line:

```console
# Create a forwarded port mapping which allows access to a specific port
# within the machine from a port on the host machine. In the example below,
# accessing "localhost:8080" will access port 80 on the guest machine.
# NOTE: This will enable public access to the opened port
config.vm.network "forwarded_port", guest: 80, host: 8080
```
**PRIVATE NETWORK:**

It is also possible to add a new network interface to the MV on a private network other than the network the host computer is connected to.

By uncommenting the following line in `Vagrantfile`:

```console
# Create a private network, which allows host-only access to the machine
# using a specific IP.
config.vm.network "private_network", ip: "192.168.33.10"
```

A new network interface will be added to the MV with `IP 192.168.33.10` and a `vboxnet0` network interface (if the first one) with `IP 192.168.33.1` will be automatically added to the host machine for allow communication.

```console
---
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s8:
      addresses:
      - 192.168.33.10/24
```

It will also be possible to assign addresses automatically using:

``` console
config.vm.network "private_network", type: "dhcp"
```

# Installation and configuration of clouds

To install a cloud we must download its code and follow the generic application installation manual.
We had downloaded it from https://nextcloud.com/install/#instructions-server

To sum up:

* Download the source code.
* Bring the cloud code to the root directory of the web server.
* Unzip the content directly to the root directory, in our case `/var/www/html`.
* Enter the website `http://localhost` with the browser and follow the instructions.

[Web application installation manual] (installacio-appliqués-web.md)

## Links to the different clouds
* ***OwnCloud***: http://www.owncloud.org
* ***NextCloud***: http://www.nextcloud.com

# Installation and configuration of web applications

To install a web application we need to download its source code and bring it to the root directory of our application server, in our case, `apache2`. When we install `apache2` a folder is created in `/var/www/html` which, by default, the web server uses as the root directory.

So if we move our application to the directory `/var/www/html` we will have access to our application via the address `http://localhost`.

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

go to either NextCloud or OwnCloud to download one of the servers. https://owncloud.com/download-server/

**_____________________________**
# Steps by steps to download OwnCloud

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

In this part download the server for owncloud and then move it to /var/www/html/, to do this:
**somehow firstly go all the way back through ``cd``**

``` console
cd vagrant
ls - to check the name
sudo mv owncloud.zip /var/www/html/
cd /var/www/html/
```

Step 7:

``` console
sudo apt install zip
ls - to check the name
sudo unzip owncloud.zip
sudo rm -rf index.html
sudo rm -rf owncloud.zip
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

``` console
sudo apt-get remove php-common

sudo add-apt-repository ppa:ondrej/php -y

sudo apt install php7.4 php7.4-intl php7.4-mysql php7.4-mbstring \
       php7.4-imagick php7.4-igbinary php7.4-gmp php7.4-bcmath \
       php7.4-curl php7.4-gd php7.4-zip php7.4-imap php7.4-ldap \
       php7.4-bz2 php7.4-ssh2 php7.4-common php7.4-json \
       php7.4-xml php7.4-dev php7.4-apcu php7.4-redis \
       libsmbclient-dev php-pear php-phpseclib

sudo update-alternatives --config php
```
