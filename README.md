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


Playbooks
---------
Playbook is an entry level to provisioning, right after Inventory file. It's a place where you can specify a role for given host or group from inventory file.
Create a YAML file and name it as you want. In our example it's just `playbook.yml`.
 
Example playbook:
```yaml
- hosts: ansible_tutorial
  become: yes
  become_user: root
  roles:
   - nginx
```

First line contains host or group from Inventory file. It's recommended to use groups, instead of hosts.
Two next lines uses `become` that leads to privileges escalation. In Inventory file we specified ssh user as vagrant. 
But it does not have enough permission (sudo) to install things. We will connect as vagrant user and then switch to root user.

Last part is list of roles that we will apply to our server. 

You can specify multiple groups with hosts in single playbook file, or you can keep each hosts/group in separate file.

Roles and tasks
---------------
Roles are essential part of Ansible. When you install new tool like php or nginx you should always create a role for that.
Roles are reusable and they can be used in different playbooks. They hold number of tasks, handlers and templates. Think of them like unix packages.

First, create directory structure. Roles must be keep in `roles` directory. Inside that directory create a directory with a name of your role, like `nginx`. Inside `nginx` create another directory called `tasks`. Inside that create file and name it `main.yml`.
So it should go like that: 
```
Your directory project -> roles -> NAME_OF_THE_ROLE -> tasks -> main.yml
```

main.yml will be automatically included by Ansible. Inside this file place following list of tasks:
```yaml
- name: install epel
  yum:
    name: epel-release
    state: present
  tags: [nginx]

- name: install nginx
  yum:
    name: nginx
    state: present
  tags: [nginx]

- name: enable nginx
  service:
    name: nginx
    state: started
  tags: [nginx, status]
```

It will contain 3 simple tasks:
1. install epel repository
2. install nginx
3. run nginx

Each task should have a descriptive name. Under that you need to specify an Ansible module you want to use.
[Here](http://docs.ansible.com/ansible/modules_by_category.html) you can find list of all modules available in Ansible. It's worth checking, you can find tons of examples there.
Each module takes a list of parameters. In order to install nginx we use yum. It takes a name of a package to install and state. Present means that it will be installed.
Last thing is a list of tags. They are optional but it's nice to have them.

Execute following command to run a playbook:
```
ansible-playbook -i hosts playbook.yml
```

It should execute 4 tasks (additional task is named setup and it's default Ansible task for gathering information about the machine).
At the end there is section called `recap`
If playbook is executed correctly, you should not get any failed or unreachable errors. You should have only ok and changed tasks.
You should write the playbooks/roles/tasks in a way  where second execution of playbook will result only in ok tasks.
 
Tags
----
You can run only tasks with specific tags:
```
ansible-playbook -i hosts playbook.yml --tags="status"
```
or
```
ansible-playbook -i hosts playbook.yml --tags="status,nginx"
```