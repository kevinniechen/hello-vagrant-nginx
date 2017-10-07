# Vagrant/Nginx Hello World
This is a hello-world project for learning vagrant and nginx.

## Vagrant

Vagrant is a VM manager abstracts the use of VM providers (e.g Virtualbox) to easily manage VMs. The goal of Vagrant is to allow develops to create portable development environments that are consistent not only between developers, but also between development and production environments.

We can create a Vagrantfile: `vagrant init ubuntu/trusty64`

By running `vagrant up`, we now use virtualbox to bring up the requested ubuntu64 VM. Note you can also manually download images from the vagrant catalog by running `vagrant box add hashicorp/precise64`. [Full vagrant catalog here](https://app.vagrantup.com/boxes/search). Vagrant will fetch the base image from the catalog if it is not already on your computer. The VM is now running.

By running `vagrant ssh`, we are now in our VM.

Note that `/vagrant` is a shared folder between the VM and your host computer. It holds the vagrantfile and any other files in that directory.

To exit, use `Ctrl+d`. To save current state and stop (lots of disk space to store RAM), run `vagrant suspend`. To shut down VM and power off (medium disk space for installations), run `vagrant halt`. To remove all resources (no disk space), run `vagrant destroy`. Remove images using `vagrant box remove`.

In your Vagrantfile, adding `config.vm.network "forwarded_port", guest: 80, host: 8080` will cause port 8080 to be listened to on the host, and will forward requests to port 80 (official web app port) on the VM.

Adding ```config.vm.network "private_network", ip: "192.168.33.10"``` will create a private network and allow you to connect to the VM through the given IP. You can then ```ssh vagrant@192.168.33.10``` and log in with the password as `vagrant`.

You can use provisioners to install software onto the box in a repeatable way. Provisioning occurs upon the first `vagrant up` as well as `vagrant provision` (keeps the box running), and `vagrant reload -provision`. 

You can perform inline provisioning like the following examples:
```
config.vm.provision "shell", inline: "echo hello"
```

```
config.vm.provision "shell", inline <<-SHELL
    apt-get update
    apt-get install -y apache2
SHELL
```

Or you can tell the vagrant provisioners the path to an independent installation script:
```
config.vm.provision :shell, path: "bootstrap.sh"
```
```
#!/usr/bin/env bash

apt-get update
apt-get install -y apache2
if ! [ -L /var/www ]; then
  rm -rf /var/www
  ln -fs /vagrant /var/www
fi
```

If you have a web server installed like apache and you have a forwarded port, you can use `vagrant share` to get a URL to your web service (e.g localhost:8080) to anyone in the world via ngrok.

You can also use `vagrant share --ssh` to get a new keypair with which anyone in the world can SSH into your machine e.g `vagrant connect --ssh yankee_riviera:clinic_iris`

## Nginx

We start by creating a provisioning script in `.provision/bootstrap.sh`. In it, we will have the following three components:

1. Installing and starting nginx:
    ```
    sudo apt-get -y install nginx
    sudo service nginx start
    ```
    You will be able to visit the nginx homepage at `127.0.0.1:8080`.

2. Copy nginx configuration from shared folder to the box's nginx site-available settings. Ensure permissions are valid. Then create a symbolic link from the site-enabled settings to the existing site-available settings:
    ```
    # set up nginx configuration
    sudo cp /vagrant/.provision/nginx/nginx.conf /etc/nginx/sites-available/site.conf
    sudo chmod 644 /etc/nginx/sites-available/site.conf
    sudo ln -s /etc/nginx/sites-available/site.conf /etc/nginx/sites-enabled/site.conf
    sudo service nginx restart
    ```

    (Files are generally created in site-available and sym-linked to site-enabled when ready).

3. In `.provision/bootstrap.sh`, we finally create a symbolic link from `/var/www`, where we usually put our code, to `/vagrant`, our shared vagrant folder:
    ```
    # clean /var/www
    sudo rm -Rf /var/www

    # symlink /var/www => /vagrant
    ln -s /vagrant /var/www
    ```

Now, we can finally create our nginx server config in `.provision/nginx/nginx.conf`:

```
server {
  listen 80;

  root   /var/www;

  server_name filler;

  access_log /var/log/nginx/access.log;
  error_log /var/log/nginx/error.log;

  location / {
  }
}
```

Run `vagrant reload --provision` to install the nginx service.

Note that nginx comes with a default sites-available (welcome page) that takes over the default host dispatch. Need to manually run `unlink /etc/nginx/sites-enabled/default`.