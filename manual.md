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

