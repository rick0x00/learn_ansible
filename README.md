# Learning Ansible

[Official Documentation](https://docs.ansible.com/ansible/latest/getting_started/index.html)

Ansible need a control node, control node needs a ssh client ans Ansible softwares.
Ansible control node controller the all devices specified in INVENTORY
Ansible can execute tasks describes on playbooks

---
Setting up hostess machine
    - Configuring ssh client

```shell
    echo "PubkeyAcceptedKeyTypes +ssh-rsa" >> ~/.ssh/config
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

```bash
    #Start vagrant environment
    vagrant up
```

---
Testing VM on vagrant with ssh connection

```bash
    #show vagrant ssh configurations
    vagrant ssh-config
    #connect with ssh to vagrant vm
    vagrant ssh
```

---
Making Ansible Inventory

```bash
    echo -e "[wordpress]\n <host vagrant vm wordpress>" >> ./hosts.cfg
```

---
Testing Ansible connection to hosts

```bash
    ansible wordpress -u vagrant --private-key .vagrant/machines/wordpress/virtualbox/private_key -i ./hosts.cfg -m shell -a 'echo Hello, World'
```

---
Creating a first Ansible playbook(provisioning.yaml)

```bash
    ---
    - hosts: all
    tasks:
        - shell: "echo 'hello world' > /vagrant/first-task.txt"
```

---
Testing Ansible playbook

```bash
    ansible-playbook ./provisioning.yaml -u vagrant -i ./hosts.cfg --private-key ./.vagrant/machines/wordpress/virtualbox/private_key
```

---
---
start to build lab wordpress

---
Creating Ansible playbook(lab_playbook.yaml)

```yaml
---
- hosts: all
  tasks:
    - name: "Install PHP5"
    apt:
        name: php5
        state: latest
    become: yes
    - name: "Install Apache2"
    apt:
        name: apache2
        state: latest
    become: yes
    - name: "Install modphp"
    apt:
        name: libapache2-mod-php5
        state: latest
    become: yes
```

---
Executing Ansible playbook

```bash
    ansible-playbook ./lab_playbook.yaml -u vagrant -i ./hosts.cfg --private-key ./.vagrant/machines/wordpress/virtualbox/private_key
```

---
Updating Ansible Playbook(lab_playbook-v2.yaml)

```yaml
---
- hosts: all
  tasks:
    - name: "install packages"
    apt:
        name: "{{ item }}"
        state: latest
    become: yes
    with_items:
        - php5
        - apache2
        - libapache2-mod-php5
        - php5-gd
        - libssh2-php
        - php5-mcrypt
        - mysql-server-5.6
        - python-mysqldb
        - php5-mysql
```

better

```yaml
---
- hosts: all
  tasks:
    - name: "install packages"
      apt:
        name:
          - php5
          - apache2
          - libapache2-mod-php5
          - php5-gd
          - libssh2-php
          - php5-mcrypt
          - mysql-server-5.6
          - python-mysqldb
          - php5-mysql
        state: latest
      become: yes
```

---
Executing Ansible playbook

```bash
    ansible-playbook ./lab_playbook-v2.yaml -u vagrant -i ./hosts.cfg --private-key ./.vagrant/machines/wordpress/virtualbox/private_key
```

---
Updating Ansible Inventory

```bash
    echo -e "[wordpress]\n <host vagrant vm wordpress> ansible_user=vagrant ansible_ssh_private_key_file='./.vagrant/machines/wordpress/virtualbox/private_key'" >> ./hosts.cfg
```

---
Executing Ansible playbook

```bash
    ansible-playbook ./lab_playbook-v2.yaml -i ./hosts.cfg
```

---
Create Database

Add the following text to the end of the `./lab playbook-v2.yaml` file

```yaml
    - name: 'Making MySQL db'
      mysql_db: 
        name: wordpress_db
        login_user: root
        state: present
```

---
Execute Ansible playbook

```bash
    ansible-playbook ./lab_playbook-v2.yaml -i ./hosts.cfg
```

---
Create Database user

Add the following text to the end of the `./lab playbook-v2.yaml` file

```yaml
    - name: 'Creating MySQL database user'
      mysql_user:
        login_user: root
        name: wordpress_user
        password: 12345
        priv: 'wordpress_db.*:ALL'
        state: present
```

---
Execute Ansible playbook

```bash
    ansible-playbook ./lab_playbook-v2.yaml -i ./hosts.cfg
```

---
Install Wordpress

Add the following text to the end of the `./lab playbook-v2.yaml` file

```yaml
    - name: "Download Wordpress Package"
      get_url:
        url: 'https://wordpress.org/wordpress-5.0.tar.gz'
        dest: '/tmp/wordpress.tar.gz'

    - name: "Unpacking Wordpress Package"
      unarchive:
        src: '/tmp/wordpress.tar.gz'
        dest: '/var/www/'
        remote_src: yes
      become: yes
```

---
Execute Ansible playbook

```bash
    ansible-playbook ./lab_playbook-v2.yaml -i ./hosts.cfg
```

---
Configuring Wordpress

Add the following text to the end of the `./lab playbook-v2.yaml` file

```yaml
    - name: "Creating Wordpress Config file"
      copy:
        src: '/var/www/wordpress/wp-config-sample.php'
        dest: '/var/www/wordpress/wp-config.php'
        remote_src: yes
      become: yes

    - name: "Configuring Wordpress config file"
      replace:
        path: '/var/www/wordpress/wp-config.php'
        regexp: '{{ item.regex }}'
        replace: '{{ item.value }}'
      with_items:
        - { regex: 'database_name_here', value: 'wordpress_db' }
        - { regex: 'username_here', value: 'wordpress_user' }
        - { regex: 'password_here', value: '12345' }
      become: yes

```

---
Execute Ansible playbook

```bash
    ansible-playbook ./lab_playbook-v2.yaml -i ./hosts.cfg
```

---
Configuring Apache

Add the following text to the end of the `./lab playbook-v2.yaml` file

```yaml
    - name: "Configuring Apache "
      replace:
        path: '/etc/apache2/sites-available/000-default.conf'
        regexp: '/var/www/html'
        replace: '/var/www/wordpress'
      become: yes
      notify:
        - restart apache
```

Add the following text to the start of the `./lab playbook-v2.yaml` file

```yaml
  handlers:
    - name: restart apache
      service:
        name: apache2
        state: restarted
      become: yes
```

---
Execute Ansible playbook

```bash
    ansible-playbook ./lab_playbook-v2.yaml -i ./hosts.cfg
```
