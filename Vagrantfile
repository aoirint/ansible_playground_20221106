# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/focal64"

  # Virtual NIC to allow ssh access from the host machine
  config.vm.network "private_network", ip: "192.168.56.10"

  # Mount public key to connect with ssh
  config.vm.provision "file", source: "./secrets/key.pub", destination: "/home/vagrant/.ssh/key.pub"

  # Add public key to authorized_keys to allow ssh access
  config.vm.provision "shell", inline: <<-SHELL
    cat /home/vagrant/.ssh/key.pub >> /home/vagrant/.ssh/authorized_keys
  SHELL

  # Add passwordless sudo permission to the login user
  config.vm.provision "shell", inline: <<-SHELL
    echo "vagrant ALL=NOPASSWD: ALL" >> /etc/sudoers.d/vagrant
  SHELL
end
