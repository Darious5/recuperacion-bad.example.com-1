# coding: utf-8
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "debian/bullseye64"
  
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "2048"
    vb.linked_clone = true
  end

  config.vm.define "bad" do |bad|
    bad.vm.hostname = "bad.example.com"
    bad.vm.network :private_network, ip: "192.168.57.2"
    
    bad.vm.provision "bootstrap", type:"shell", inline: <<-SHELL
        apt-get update -y
        apt-get install -y bind9 bind9-utils bind9-doc
    SHELL

    bad.vm.provision "bad", type:"shell", inline: <<-SHELL
      cp -v /vagrant/dns/bad/named* /etc/bind/
      cp -v /vagrant/dns/bad/*.dns /var/lib/bind/
      chown bind:bind /var/lib/bind/example.com.dns /var/lib/bind/192.168.57.dns
      chmod 644 /var/lib/bind/example.com.dns /var/lib/bind/192.168.57.dns
      systemctl restart bind9
    SHELL
  end 

end # Configure
