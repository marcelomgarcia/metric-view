# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.synced_folder ".", "/vagrant", type: "rsync",
    rsync__exclude: ".git/"

  config.vm.box = "ubuntu/focal64"

  config.ssh.insert_key = false
  config.vm.provision "file", source: "keys/config",  destination: "~/.ssh/config"
  config.vm.provision "file", source: "keys/vagrant", destination: "~/.ssh/id_rsa"
  config.vm.provision "file", source: "keys/vagrant.pub", destination: "~/.ssh/id_rsa.pub"
  config.vm.provision "shell", inline: "chmod 600 /home/vagrant/.ssh/id_rsa"
  config.vm.provision "shell", inline: "chmod 600 /home/vagrant/.ssh/id_rsa.pub"
  config.vm.provision "shell", inline: "chmod 600 /home/vagrant/.ssh/config"

  # Define the first node.
  config.vm.define "flik" do |flik|
    flik.vm.hostname = "flik"
    flik.vm.provider :virtualbox do |vb|
        vb.memory = 2048
        vb.cpus = 2
    end
    flik.vm.network "private_network", ip: "192.168.56.11"
  end

  # Define the second node.
  config.vm.define "hopper" do |hopper|
    hopper.vm.hostname = "hopper"
    hopper.vm.provider :virtualbox do |vb|
        vb.memory = 2048
        vb.cpus = 2
    end
    hopper.vm.network "private_network", ip: "192.168.56.12"
  end

# Define the master node.
  config.vm.define "atta", primary: true do |atta|
    atta.vm.hostname = "atta"
    atta.vm.network "private_network", ip: "192.168.56.10"
    atta.vm.network "forwarded_port", guest: 5601, host: 5601
    atta.vm.network "forwarded_port", guest: 9200, host: 9200
    atta.vm.provider :virtualbox do |vb|
        vb.memory = 4096
        vb.cpus = 4
    end
  end

end
