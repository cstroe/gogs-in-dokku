# gogs-in-dokku

A walkthrough for running Gogs in Dokku.

# Requirements

* VirtualBox 5.0+
* Vagrant (optional)
* Ubuntu Xenial (16.04)

## Start a Dokku VM

To get a Dokku instance quickly up and running, I've created a [Vagrantfile](https://github.com/cstroe/gogs-in-dokku/blob/master/Vagrantfile) and [provision.sh](https://github.com/cstroe/gogs-in-dokku/blob/master/provision.sh) that creates a VM with Dokku installed.  

**NOTE**: You can use the same commands from [provision.sh](https://github.com/cstroe/gogs-in-dokku/blob/master/provision.sh) to install Dokku on your Ubuntu 16.04 server if you don't want to use Vagrant.

To create the VM simply type:

    vagrant up

The provisioning will take about 10 minutes.  Once it's done, you can ssh into the vagrant VM to inspect the networking configuration of the VM:

    vagrant ssh
    ifconfig -a | grep inet

### Notes

* The `Vagrantfile` bridges to `eth0`, expecting that it will be your external network interface.  This allows the VM to get an external IP.  You can disable this to use the NAT interface only.

## Configure Dokku

Dokku requires some configuration after installing and this is done via a one-time install web page.  Simply browse to the Dokku server via its IP or hostname.  For example, if the external IP is `192.168.1.102`, you should browse to `http://192.168.1.102` in your browser to finish the Dokku configuration.

Fill out the following information:
* The public SSH key that you want to use for git access to Dokku.  If in doubt, use the contents of `~/.ssh/id_rsa.pub` to allow yourself access from your machine.
* If you have control over your local DNS (such as with [OpenWrt](https://openwrt.org/)), you can set the hostname of the Dokku box, and enable virtualhost naming for apps.  Otherwise, just use the IP.
* Click `Finish Setup` when you're done.

## Postgres Dokku Plugin

Because Gogs requires a database, we can use the Postgres plugin for Dokku to manage our database container.  From inside the Vagrant VM (via `vagrant ssh`), type the following:

    sudo dokku plugin:install https://github.com/dokku/dokku-postgres.git


## Deploy Gogs

We will run Gogs from the dockerhub.com image.  Inside the Vagrant VM, start by creating the application and our database:

    dokku apps:create gogs
    dokku postgres:create gogs-db
    dokku postgres:link gogs-db gogs

We need to save the Gogs files in a volume so that we don't lose our Git repositories between application restarts:

    sudo mkdir /var/lib/dokku/data/storage/gogs_data
    sudo chown root:root /var/lib/dokku/data/storage/gogs_data
    dokku storage:mount gogs /var/lib/dokku/data/storage/gogs_data:/data

We also need to use a custom proxy port for Gogs, since our system SSH service is already listening on port `22`:

    dokku proxy:ports-add gogs http:2222:22
    dokku proxy:ports-remove gogs http:22:22

Now we can pull and deploy the Docker image from dockerhub:

    docker pull gogs/gogs:latest
    docker tag gogs/gogs:latest dokku/gogs:latest
    dokku tags:deploy gogs latest

## Configure Gogs

***IMPORTANT***: Currently, there is a Dokku bug that does not properly expose the HTTP port of Gogs.

To configure Gogs, you need to browse to Gogs' HTTP port and finish the Gogs setup.  All the database configuration can be see by issuing:

    dokku postgres:info gogs-db

Note the `Dsn:` entry, as it contains the username, password, hostname, port, and database name of the Postgres server.

