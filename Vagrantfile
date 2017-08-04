# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.provider "virtualbox" do |v|
    v.name = "gogs-in-dokku"
    v.memory = 2048
    v.cpus = 2
  end

  config.vm.box = "ubuntu/xenial64"
  # Setting MAC address to get static DHCP lease
  config.vm.network :public_network, bridge: "eth0", :mac => "0800278f9cfa"
  config.vm.provision "shell", path: "provision.sh"
end
