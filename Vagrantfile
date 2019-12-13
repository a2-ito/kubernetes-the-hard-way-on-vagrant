# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.

$script = <<-SCRIPT

mkdir -p /home/vagrant/.ssh
chmod 700 /home/vagrant/.ssh
cat /vagrant/keys/public >> /home/vagrant/.ssh/authorized_keys

SCRIPT
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  #config.vm.box = "centos/7"
  #config.vm.box = "oraclelinux/7"
  #config.vm.box = "ubuntu/trusty64"
  config.vm.box = "bento/ubuntu-18.04"

#  config.ssh.private_key_path = ["keys/private_key", "~/.vagrant.d/insecure_private_key"]
#  config.ssh.insert_key = false
#  config.vm.provision "file", source: "keys/public", destination: "~/.ssh/authorized_keys"

  config.vm.define "master1" do |master1|
    master1.vm.hostname = "master1"
    master1.vm.network "forwarded_port", guest: 22, host: 2201, host_ip: "0.0.0.0"
    master1.vm.network "forwarded_port", guest: 6443, host: 64431, host_ip: "0.0.0.0"
    master1.vm.network "private_network", ip: "192.168.33.101"
    #master1.ssh.private_key_path = ["keys/private_key", "~/.vagrant.d/insecure_private_key"]
    #master1.vm.provision "file", source: "keys/public", destination: "~/.ssh/authorized_keys"
    master1.vm.provision "shell", inline: $script
  end
  config.vm.define "master2" do |master2|
    master2.vm.hostname = "master2"
    master2.vm.network "forwarded_port", guest: 22, host: 2202, host_ip: "0.0.0.0"
    master2.vm.network "forwarded_port", guest: 6443, host: 64432, host_ip: "0.0.0.0"
    master2.vm.network "private_network", ip: "192.168.33.102"
    #master2.vm.provision "file", source: "keys/public", destination: "~/.ssh/authorized_keys"
    master2.vm.provision "shell", inline: $script
  end
  config.vm.define "master3" do |master3|
    master3.vm.hostname = "master3"
    master3.vm.network "forwarded_port", guest: 22, host: 2203, host_ip: "0.0.0.0"
    master3.vm.network "forwarded_port", guest: 6443, host: 64433, host_ip: "0.0.0.0"
    master3.vm.network "private_network", ip: "192.168.33.103"
    #master3.vm.provision "file", source: "keys/public", destination: "~/.ssh/authorized_keys"
    master3.vm.provision "shell", inline: $script
  end
  config.vm.define "worker1" do |worker1|
    worker1.vm.hostname = "worker1"
    worker1.vm.network "forwarded_port", guest: 22, host: 2204, host_ip: "0.0.0.0"
    # Service for Grafana
    worker1.vm.network "forwarded_port", guest: 30080, host: 30081, host_ip: "0.0.0.0"
    # Service for Prometheus
    worker1.vm.network "forwarded_port", guest: 30090, host: 30091, host_ip: "0.0.0.0"
    # Service for Traefik Dashboard
    worker1.vm.network "forwarded_port", guest: 30000, host: 30001, host_ip: "0.0.0.0"
    # Service for Alertmanager
    worker1.vm.network "forwarded_port", guest: 30093, host: 30193, host_ip: "0.0.0.0"
    worker1.vm.network "forwarded_port", guest: 80, host: 10081, host_ip: "0.0.0.0"

    # Service for Docker Registry
    worker1.vm.network "forwarded_port", guest: 30500, host: 30501, host_ip: "0.0.0.0"

    worker1.vm.network "private_network", ip: "192.168.33.111"
    worker1.vm.provision "shell", inline: $script
    worker1.vm.provider "virtualbox" do |vb|
      vb.memory = 2048
    end
  end
  config.vm.define "worker2" do |worker2|
    worker2.vm.hostname = "worker2"
    worker2.vm.network "forwarded_port", guest: 22, host: 2205, host_ip: "0.0.0.0"
    # Service for Grafana
    worker2.vm.network "forwarded_port", guest: 30080, host: 30082, host_ip: "0.0.0.0"
    # Service for Prometheus
    worker2.vm.network "forwarded_port", guest: 30090, host: 30092, host_ip: "0.0.0.0"
    # Service for Traefik Dashboard
    worker2.vm.network "forwarded_port", guest: 30000, host: 30002, host_ip: "0.0.0.0"
    # Service for Alertmanager
    worker2.vm.network "forwarded_port", guest: 30093, host: 30293, host_ip: "0.0.0.0"
 
    # Service for Docker Registry
    worker2.vm.network "forwarded_port", guest: 30500, host: 30501, host_ip: "0.0.0.0"

    worker2.vm.network "forwarded_port", guest: 80, host: 10082, host_ip: "0.0.0.0"
    worker2.vm.network "private_network", ip: "192.168.33.112"
    worker2.vm.provision "shell", inline: $script
    worker2.vm.provider "virtualbox" do |vb|
      vb.memory = 2048
    end
  end
  config.vm.define "lb" do |lb|
    lb.vm.hostname = "lb"
    lb.vm.network "forwarded_port", guest: 22, host: 2206, host_ip: "0.0.0.0"
    lb.vm.network "forwarded_port", guest: 6443, host: 6443, host_ip: "0.0.0.0"
    lb.vm.network "forwarded_port", guest: 443, host: 443, host_ip: "0.0.0.0"
    lb.vm.network "private_network", ip: "192.168.33.100"
    lb.vm.provision "shell", inline: $script
  end

  #config.vm.network "forwarded_port", guest: 22, host: 2201, host_ip: "0.0.0.0"
  #config.vm.network "forwarded_port", guest: 22, host: 2202, host_ip: "0.0.0.0"
  #config.vm.network "forwarded_port", guest: 22, host: 2203, host_ip: "0.0.0.0"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"
  #config.vm.network "forwarded_port", guest: 8080, host: 8080, host_ip: "0.0.0.0"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  #config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
    # Customize the amount of memory on the VM:
  #  vb.memory = "1024"
  #end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   apt-get update
  #   apt-get install -y apache2
  # SHELL
end
