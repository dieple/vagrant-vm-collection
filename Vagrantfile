# -*- mode: ruby -*-
# vi: set ft=ruby :
# Sample Vagranfile to setup Learning Environment
# for Ansible Playbook Essentials

require_relative 'vagrant_rancheros_guest_plugin.rb'

# To enable rsync folder share change to false
$rsync_folder_disabled = false
$number_of_nodes = 1
$vm_mem = "1024"
$vb_gui = false



VAGRANTFILE_API_VERSION = "2"
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "ansible-ubuntu-1404-amd64"
  config.vm.box_url = "https://cloud-images.ubuntu.com/vagrant/trusty/current/trusty-server-cloudimg-amd64-vagrant-disk1.box"

  config.vm.define "control" do |control|
    control.vm.network :private_network, ip: "192.168.61.10"
  end

  config.vm.define "db" do |db|
    db.vm.network :private_network, ip: "192.168.61.11"
  end

  config.vm.define "dbel" do |db|
    db.vm.network :private_network, ip: "192.168.61.14"
    db.vm.box = "opscode_centos-6.5-i386"
    db.vm.box = "http://opscode-vm-bento.s3.amazonaws.com/vagrant/virtualbox/opscode_centos-6.5_chef-provisionerless.box"
  end

  config.vm.define "www" do |www|
    www.vm.network :private_network, ip: "192.168.61.12"
  end

  config.vm.define "lb" do |lb|
    lb.vm.network :private_network, ip: "192.168.61.13"
  end

  # rkt
  config.vm.define "rkt" do |rkt|
    # grab Ubuntu 14.10 official image
    rkt.vm.box = "ubuntu/trusty64" # Ubuntu 14.10
    rkt.vm.network :private_network, ip: "192.168.61.20"

    # fix issues with slow dns http://serverfault.com/a/595010
    rkt.vm.provider :virtualbox do |vb, override|
        vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
        vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
        # add more ram, the default isn't enough for the build
        vb.customize ["modifyvm", :id, "--memory", "2048"]
    end
    # install Build Dependencies (GOLANG)
    rkt.vm.provision :shell, :privileged => false, :path => "scripts/vagrant/install-go.sh"
    # Install rkt
    rkt.vm.provision :shell, :privileged => false, :path => "scripts/vagrant/install-rkt.sh"
    # sync external host
    rkt.vm.synced_folder "/Users/devops/sandboxes/", "/home/vagrant/sandboxes"
  end

  # rancher-os
  config.vm.define "ros" do |ros|
    ros.vm.box   = "rancherio/rancheros"
    ros.vm.box_version = ">=0.3.3"

    (1..$number_of_nodes).each do |i|
      hostname = "rancher-%02d" % i
      ros.vm.define hostname do |node|
          node.vm.provider "virtualbox" do |vb|
              vb.memory = $vm_mem
              vb.gui = $vb_gui
          end
          ip = "192.168.61#{i+30}"
          node.vm.network "private_network", ip: ip
          node.vm.hostname = hostname
          # Disabling compression because OS X has an ancient version of rsync installed.
          # Add -z or remove rsync__args below if you have a newer version of rsync on your machine.
          node.vm.synced_folder ".", "/opt/rancher", type: "rsync",
              rsync__exclude: ".git/", rsync__args: ["--verbose", "--archive", "--delete", "--copy-links"],
              disabled: $rsync_folder_disabled
      end
    end
  end

end
