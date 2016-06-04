Ansible tutorial
================

> **Note:**
> This is **tl;dr** version for full tutorial here:  [Ansible tutorial](http://blog.astaz3l.com/2016/06/04/ansible-tutorial-part-1/)
> Read it if you want to get more information. Tutorial on GitHub has only essential things from full tutorial:)



What is Ansible?
----------------

Simple yet powerful tool for configuration management and orchestration of your infrastructure. Great alternative for Chef and Puppet!
Ansible is using SSH for communication with the server. Everything is based on YAML files and Jinja2 templates. 



Use cases
---------
With Ansible you can perform following tasks on your server(s):
 - Install software
 - Configure all services
 - Test your servers
 - Deploy application
 - Many more...
 
 

How to install Ansible?
-----------------------

Make sure that you are installing Ansible **2.0** or higher

|                                            Linux                                           |                                             OSX                                             |                                                      Windows                                                     |
|:------------------------------------------------------------------------------------------:|:-------------------------------------------------------------------------------------------:|:----------------------------------------------------------------------------------------------------------------:|
| apt or yum                                                                                 | homebrew or pip                                                                             | apt or babun                                                                                                     |
| [Tutorial](http://docs.ansible.com/ansible/intro_installation.html#latest-release-via-yum) | [Tutorial](http://docs.ansible.com/ansible/intro_installation.html#latest-releases-via-pip) | [Tutorial](https://chrisgilbert1.wordpress.com/2015/06/17/install-a-babun-cygwin-shell-and-ansible-for-windows/) |



Test environment using Vagrant
------------------------------

Create new directory and place following content in `Vagrantfile`:
```
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  config.vm.box = "geerlingguy/centos7"
  config.vm.network "forwarded_port", guest: 80, host: 8080
  config.vm.network "private_network", ip: "192.168.60.70"
  config.vm.provider "virtualbox" do |vb|
     vb.memory = "2048"
  end
end
```

Start Vagrant Box and you are ready to start with Ansible.



Inventory file
--------------

In inventory file you will keep connection parameters to your servers. 
Create `hosts` file in the directory with Vagrantfile with following contents:

```
[ansible_tutorial]
192.168.60.70 ansible_user=vagrant ansible_private_key_file="./.vagrant/machines/default/virtualbox/private_key"
```

First line is a group. You can have many groups like [web] or [database]. You will use them later on in playbooks. 
Each group can contain multiple servers. 

Next line contains parameters for connections. You need to start with IP address of your server or domain. After that specify user and private key path. Please DON'T use password authentication. SSH Keys are the way to go.



Test connections to your server
-------------------------------

In command line execute following command (ansible [GROUP_NAME|IP_ADDRESS or DOMAIN|ALL] -m ping -i PATH_TO_INVENTORY_FILE):
```
ansible ansible_tutorial -m ping -i hosts
```

You should get SUCCESS status. For debugging failed connections add `-vvvv` parameter to command above. 

