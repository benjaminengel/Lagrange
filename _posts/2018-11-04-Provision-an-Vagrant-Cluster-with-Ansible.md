---
layout: post
title: "Provision an Vagrant Cluster with Ansible"
author: "Ben Engel"
categories: devop
tags: ssh, vagrant, ansible
image: 
---

This time we will take a look on how to setup an dev environment to test an ansible deployment. 

Therefore we will leverage vagrant.



Vagrant is a toll to set up (in our case) vm's so that we can locally simulate a cluster.  It works with Virtualbox and VM Ware as provider. The provider detection is done automatically by vagrant the only case where we need to interfere is when we have both options installed.



But before we deal with those details some basic things about vagrant. The entire configuration for a vm or a cluster of vm's is in the `Vagrantfile` we discuss the essential parts of one that is adapted from [this](https://github.com/jandom/gromacs-slurm-openmpi-vagrant/blob/master/Vagrantfile) Vagrantfile. The important part for us in this is the distribution of an ssh-key across the cluster so that we can easily use ansible even if we switch out the base os. 

But let us take a look at the file: 

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
```
here we start the configuration by looping over the `Vagrant.configure` version 2 to vonfigure the vm's

```ruby
  config.vm.box = "ubuntu/bionic64"
# generate and share an ssh key across all hosts
  script = <<-SCRIPT
    set -x
    if [[ ! -e /etc/.provisioned ]]; then
      # we only generate the key on one of the nodes
      if [[ ! -e /vagrant/id_rsa ]]; then
        ssh-keygen -t rsa -f /vagrant/id_rsa -N ""
      fi
      install -m 600 -o vagrant -g vagrant /vagrant/id_rsa /home/vagrant/.ssh/
      # the extra 'echo' is needed because Vagrant inserts its own key without a
      # newline at the end
      (echo; cat /vagrant/id_rsa.pub) >> /home/vagrant/.ssh/authorized_keys
      touch /etc/.provisioned
    fi
  SCRIPT
```
these are settings for all vm that follow where we specify the os (other images can be found [here](https://app.vagrantup.com/boxes/search)), furthermore we declare a script that generates only on the first vm we touch an ssh-key which we distribute across all other vm's later.
```ruby
  # configure zookeeper cluster
  (1..3).each do |i|
    config.vm.provider "virtualbox" do |v|
      v.memory = 2048
    end
    config.vm.define "zookeeper#{i}" do |s|
      s.vm.hostname = "zookeeper#{i}"
      s.vm.network "private_network", ip: "10.30.3.1#{i}"
      s.vm.provision :docker
      s.vm.provision "shell", inline: script
    end
  end
  # configure brokers
  (1..3).each do |i|
    config.vm.provider "virtualbox" do |v|
      v.memory = 4096
    end
    config.vm.define "broker#{i}" do |s|
      s.vm.hostname = "broker#{i}"
      s.vm.network "private_network", ip: "10.30.3.2#{i}"
      s.vm.provision :docker
      s.vm.provision "shell", inline: script
    end
  end
```
in the loop we set up 3 zookeeper and 3 kafka broker vm's. 
* `v.memory` option sets the RAM for the vm
*  `s.vm.hostname` sets the hostname
*  `s.vm.network "private_network"` declares an internal network which doesn't expose/forwards ports to the host machine, the oiption `ip` sets the ip for the vm
*  `s.vm.provision :docker` installs docker on the vm
*  `s.vm.provision "shell", inline: script` executes our script on the vm
```ruby
  (1..3).each do |i|
    config.vm.define "manager" do |s|
      s.vm.hostname = "manager"
      s.vm.network "private_network", ip: "10.30.3.100"
      s.vm.provision :docker
      s.vm.provision "shell", inline: script
      s.vm.provision "shell", inline: <<-SHELL
        apt-get update
        apt-get install -y ansible sshpass python-pip
      SHELL
    end
  end
```
the only new option here is that we can use the `SHELL` tag to write multiline bash-scripts in `s.vm.provision "shell"`
```ruby
  config.vm.provider "virtualbox" do |v|
    #  This setting controls how much cpu time a virtual CPU can use. A value of 50 implies a single virtual CPU can use up to 50% of a single host CPU.
    v.customize ["modifyvm", :id, "--cpuexecutioncap", "50"]
    # additional config
    v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    v.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
  end
end
```
Here we control some virtualbox recource properties.


With this setup we can simply connect to the manager an provision our cluster via ansible. 
An ansible inventory could look like the following
```
#hosts
[all:vars]
#needed to set the python interpreter on remote host (py2/py3 problem)
ansible_python_interpreter=/usr/bin/python3

[zookeeper]
zookeeper1 ansible_host=10.30.3.11 id=1
zookeeper2 ansible_host=10.30.3.12 id=2
zookeeper3 ansible_host=10.30.3.13 id=3

[kafka]
broker1 ansible_host=10.30.3.21 id=1
broker2 ansible_host=10.30.3.22 id=2
broker3 ansible_host=10.30.3.23 id=3
```
