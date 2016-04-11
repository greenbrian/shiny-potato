# -*- mode: ruby -*-
# vi: set ft=ruby :

$bootstrap = <<ANSIBLE
apt-get update  -y -qq
apt-get install -y -qq software-properties-common
apt-add-repository ppa:ansible/ansible
apt-get update  -y -qq
apt-get -y -qq install ansible
ANSIBLE

$hack = <<SCRIPT
GALAXY=/usr/local/bin/ansible-galaxy
echo '#!/usr/bin/env python2
import sys
import os

args = sys.argv
if args[1:] == ["--help"]:
  args.insert(1, "info")

os.execv("/usr/bin/ansible-galaxy", args)
' | sudo tee $GALAXY
sudo chmod 0755 $GALAXY
SCRIPT

VAGRANTFILE_API_VERSION = "2"
Vagrant.require_version ">= 1.8.1"
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "bento/ubuntu-14.04"
  config.vm.define "ubuntu" do |ubuntu|
    ubuntu.vm.provider(:virtualbox) do |v|
      v.customize ['modifyvm', :id, '--cpus', '2']
      v.customize ['modifyvm', :id, '--memory', '2048']
      v.customize ["modifyvm", :id, "--cpuexecutioncap", "50"]
      v.customize ["modifyvm", :id, "--chipset", "ich9"]
      v.customize ["modifyvm", :id, "--vtxvpid", "on"]
      v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
      v.customize ["modifyvm", :id, "--ioapic", "on"]
    end

    ubuntu.vm.provision :shell, inline: $bootstrap
    # hack to fix ansible detection
    # https://github.com/mitchellh/vagrant/issues/6793
    # http://stackoverflow.com/questions/35299304/automatically-installing-and-running-ansible-local-via-vagrant
    ubuntu.vm.provision :shell, inline: $hack


    ubuntu.vm.provision "ansible_local" do |ansible|
      ansible.install           = false
      ansible.verbose           = true
      ansible.provisioning_path = "/vagrant/vagrant_setup"
      ansible.playbook          = "vagrant-setup.yml"
    end
    ubuntu.vm.hostname          = "ubuntu"
  end

end
