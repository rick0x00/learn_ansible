# Learning Ansible

[Official Doccumentation](https://docs.ansible.com/ansible/latest/getting_started/index.html)

Ansible need a control node, control node needs a ssh client ans Ansible softwares.
Ansible control node controller the all devices specified in INVENTORY
Ansible can execute tasks describes on playbooks

---
Setting up hostess machine
    - Configuring ssh client

```shell
    echo -e "Host 127.0.0.1 \n    PubkeyAcceptedKeyTypes +ssh-rsa" >> ~/.ssh/config
```

---
Provisioning lab

```text
    #Add text to ./Vagrant file
    Vagrant.configure("2") do |config|

        config.vm.box = "ubuntu/trusty64"

        config.vm.provider "virtualbox" do |v|
            v.memory = 1024
        end

        config.vm.define "wordpress" do |m|
            m.vm.network "private_network", ip: "<host vagrant vm wordpress>"
        end
    end
```

```shell
    #Start vagrant environment
    vagrant up
```

---
Testing VM on vagrant with ssh connection

```shell
    #show vagrant ssh configurations
    vagrant ssh-config
    #connect with ssh to vagrant vm
    vagrant ssh
```

---
Making Ansible Inventory

```shell
    echo -e "[wordpress]\n <host vagrant vm wordpress>" >> ./hosts
```
