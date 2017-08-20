---
layout: post
title: DCOS Vagrant Ubuntu
tags: [dcos, mesos, ubuntu, vagrant]
---

Installation steps to setup DCOS on Ubuntu using Vagrant.

```bash
sudo apt-get install python3
sudo apt-get install python3-pip
pip3 install virtualenv
vagrant plugin install vagrant-hostmanager
cd ~/dev/tools/dcos/dcos-vagrant-1.2.0
cp VagrantConfig.yaml.example VagrantConfig.yaml
```

* Reduce the memory of agent node to 1GB i.s.o. 6GB

* Update etc/config-1.9.yml add _oauth_enabled: false_

* Start DC/OS

```bash
vagrant up
```

* Go to http://m1.dcos/

* To destroy the setup

```bash
vagrant destroy -f
```
