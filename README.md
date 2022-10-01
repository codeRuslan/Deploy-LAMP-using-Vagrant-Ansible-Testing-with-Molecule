LAMP deploy using Vagrant, Ansible and Molecule for testing
======

![image](https://www.cloudsigma.com/wp-content/uploads/lamp-stack-blog-post.jpg)

This is an example project to showcase my skills in usage Ansible + Molecule (as testing framework with ansible-lint, yaml-lint) and Vagrant.
---
Technological stack used:

- Vagrant
- Ansible 
- Molecule + linters

---
## Overview

This is an example project of deployment LAMP stack (Linux, NGINX, MySQL, PHP) to showcase  abilities to use configuration management tool such as Vagrant (Packer also good - if cloud-solution provider is used)- configure development environment , Ansible - to configure virtual machine, Molecule(with linters) - testing and development of Ansible Roles. **Ansible playbooks will include both ways to work with CentOS and Ubuntu machine**


### Vagrant
(Disclaimer: For presentation purposes I have kept my SSH keys that I had use in a folder called keys, where Vagrant took keys to insert into provisioned VM)
First of all, Ubuntu machine will be locally deployed with a help of Vagrant using Vagrantfile. There I have deployes SSH keys and setted up initial SSH connection between my machine and provisioned local machine in Virtualbox. Also, there I had connected right away Ansible playbook to deploy my playbook with roles inserted in it. Worth to mention, that also that machine is provisioned with installed Docker and both lints that I will need to use. It all, could be found in this line:
```
mngm.vm.provision "shell", inline: <<-SHELL
      sed -i 's/ChallengeResponseAuthentication no/ChallengeResponseAuthentication yes/#g' /etc/ssh/sshd_config
      service ssh restart
      curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
      sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
      sudo apt update && sudo apt --assume-yes install docker-ce docker-compose python3-pip git
      sudo usermod -aG docker vagrant
      chmod 600 /home/vagrant/.ssh/vagrant.key
      pip3 install "ansible-lint"
      pip install --user yamllint
    SHELL
```

### Ansible 
Let's check Ansible, I have developed 3 roles, where there is main **playbook.yml** located right in ansible folder, with following code including all roles and restarting for main service in LAMP stack.
```
---
- name: LAMP init
  hosts: mngm
  become: true
  gather_facts: true

  roles:
    - nginx
    - php
    - mysql

  tasks:
    - name: Restart php-fpm restarted
      ansible.builtin.service:
        name: php7.4-fpm
        state: restarted
    - name: Service nginx restarted
      ansible.builtin.service:
        name: nginx
        state: restarted
    - name: Service MySQL restarted
      ansible.builtin.service:
        name: mysql
        state: restarted
```

Following that, I have 3 roles for provisioning each in of 3 services. **Each roles has two scenarios both for CentOS and Ubuntu machine**. There is following file structure for all roles
```
ansible
      roles
          mysql──────│defaults
          nginx      │molecule
          php        │tasks
                     │.ansible-lint
                     │.yamllint             
```
- defaults - will have all variables used in playbook for this role
- tasks - will have 3 tasks (Where main.yml - will be main playbook (where right os_family will be choosen), and two other ubuntu.yml and centos.yml will contain right playbook for each of linux distribution)

### Molecule
**Molecule is inserted there in order to test all of roles for idempotence and conduct some final tests**. All my additional test included in verify.yml, where I built tests to conducted whether main package has installed good for each of the roles and where the service is running fine. As example here is code from verify.yml for NGINX role.
```
---
# This is an example playbook to execute Ansible tests.

- name: Verify
  hosts: all
  gather_facts: false
  tasks:
  - name: Example assertion
    ansible.builtin.assert:
      that: true
  - name: Nginx - installed
    ansible.builtin.package:
      name: nginx
      state: present
  - name: Nginx - start and enabled
    ansible.builtin.service:
      name: nginx
      state: started
      enabled: true
```

Also, worth to mention that both linters for ansible and yaml where configured manually configured, even though Molecule inserts this linters by default I had to add a few arguments into warn_list and skip_list in ansible linter.
