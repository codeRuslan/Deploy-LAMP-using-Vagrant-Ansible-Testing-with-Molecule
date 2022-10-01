# -*- mode: ruby -*-
# vi: set ft=ruby :

cluster_cidr_prefix = "192.168.23"
ubuntu_version      = "ubuntu/focal64"

Vagrant.configure("2") do |config|
  config.vm.provision "file", source: "ansible/files/vagrant.key.pub", destination: "/home/vagrant/.ssh/vagrant.key.pub"
  config.vm.provision "file", source: "ansible/files/vagrant.key", destination: "/home/vagrant/.ssh/vagrant.key"
  config.vm.define "mngm" do |mngm|
    mngm.vm.box = "#{ubuntu_version}"
    mngm.vm.provider "virtualbox"
    mngm.vm.hostname = "mngm"
    mngm.vbguest.auto_update = true
    mngm.vm.provider "virtualbox" do |vb|
      vb.check_guest_additions = true
      vb.name   = "mngm"
      vb.cpus   = 2
      vb.memory = 4096
    end
    mngm.vm.network "private_network", ip: "#{cluster_cidr_prefix}.235",
      nic_type: "82543GC"
    config.vm.synced_folder "ansible/", "/home/vagrant/ansible"

    mngm.vm.provision "ansible_local" do |ansible|
      ansible.playbook = "./ansible/playbook.yml"
      ansible.install        = true
    end
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
      python3 -m pip install --user "molecule"
      python3 -m pip install --user "molecule[docker]"
      pip install molecule docker
      pip3 install molecule-docker
    SHELL
  end
end
