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

### Prepare the MV once created

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

### Port forwarding

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
### Private network

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

To sum up:

* Download the source code.
* Bring the cloud code to the root directory of the web server.
* Unzip the content directly to the root directory, in our case `/var/www/html`.
* Enter the website `http://localhost` with the browser and follow the instructions.

[Web application installation manual] (installacio-appliqués-web.md)

## Links to the different clouds
* ***OwnCloud***: http://www.owncloud.org
* ***NextCloud***: http://www.nextcloud.com

