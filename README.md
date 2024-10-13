# Ansible Project : NTP configuration

in this project i will use ansible to configure NTP on two types of machines Ubuntu and Centos. the configuration will be different for each type. i will be using 3 EC2 instance for this project, one as the control machine on witch ansible will be installed, one as ubuntu host and one as Centos host

here are the commands for installing ansible

```bash
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible
```

here is the config file 

```yaml
[defaults]
host_key_checking=False # 
inventory=./inventory.yaml
forks=5
log_path=./ansible.log
```

here is the inventory file

```yaml
# Inventory File

all:
  hosts:
    ubuntu01:
      ansible_host: 172.31.28.85 #private IP of the EC2 instance
    centos01:
      ansible_host: 54.198.72.46

  children:
    ubuntu_hosts:
      hosts:
        ubuntu01:
      vars:
        ansible_user: ubuntu

    centos_hosts:
      hosts:
        centos01:
      vars:
        ansible_user: ec2-user

  vars:
    ansible_ssh_private_key_file: sshkey.pem

```

here is the playbook file 

```yaml
---
- name: Install and configure NTP or Chrony
  hosts: all
  become: yes  # Use sudo to execute tasks
  tasks:
    - name: Set timezone to Europe/Paris
      community.general.timezone:
        name: Europe/Paris

    - name: Install NTP on Ubuntu
      apt:
        name: ntp
        state: present
        update_cache: yes
      when: ansible_facts['distribution'] == 'Ubuntu'

    - name: Ensure NTP service is started and enabled on Ubuntu
      service:
        name: ntp
        state: started
        enabled: yes
      when: ansible_facts['distribution'] == 'Ubuntu'

    - name: Install Chrony on CentOS
      yum:
        name: chrony
        state: present
        update_cache: yes
      when: ansible_facts['distribution'] == 'CentOS'

    - name: Ensure Chrony service is started and enabled on CentOS
      service:
        name: chronyd
        state: started
        enabled: yes
      when: ansible_facts['distribution'] == 'CentOS'

    - name: Deploy Chrony configuration file
      template:
        src: ./templates/ntp.conf
        dest: /etc/ntpsec/ntp.conf
        backup: yes
      when: ansible_facts['distribution'] == 'Ubuntu'
      notify:
        - Restart NTP
    - name: Deploy Chrony configuration file
      template:
        src: ./templates/chrony.conf
        dest: /etc/chrony.conf
        backup: yes
      when: ansible_facts['distribution'] == 'CentOS'
      notify:
        - Restart Chronyd

        
 handlers:
  - name: Restart Chronyd
    service:
      name: chronyd
      state: restarted
  
  - name: Restart NTP
    service:
      name: ntp
      state: restarted
      
```

variables in groups_vars/all  

```yaml
# all
ntp0: 0.fr.pool.ntp.org
ntp1: 1.fr.pool.ntp.org
ntp2: 2.fr.pool.ntp.org
ntp3: 3.fr.pool.ntp.org
```

add this to the ntp config files in the template directory

```yaml
pool "{{ntp0}}" iburst
pool "{{ntp1}}" iburst
pool "{{ntp2}}" iburst
pool "{{ntp3}}" iburst
```
