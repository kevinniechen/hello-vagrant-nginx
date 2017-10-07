# Vagrant/Nginx Hello World
This is a hello-world project for learning vagrant and nginx.

Vagrant is a VM manager abstracts the use of VM providers (e.g Virtualbox) to easily manage VMs. The goal of Vagrant is to allow develops to create portable development environments that are consistent not only between developers, but also between development and production environments.

We can create a Vagrantfile: `vagrant init ubuntu/trusty64`

By running `vagrant up`, we now use virtualbox to bring up the requested ubuntu64 VM. Vagrant will fetch the image from the catalog if it is not already on your computer. The VM is now running.

By running `vagrant ssh`, we are now in our VM.

Note that `/vagrant` is a shared folder between the VM and your host computer. It holds the vagrantfile and any other files in that directory.

To exit, use `Ctrl+d`. To suspend, run `vagrant suspend`. To kill, run `vagrant halt`.