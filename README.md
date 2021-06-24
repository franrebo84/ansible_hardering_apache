# ansible_hardering_apache

This playbook is intended to be executed on ubuntu (for now).
Things the playbook does:
- Install recommended packages like sysstat/ncdu/iotop and more
- Change SSH port to port defined in ansible_port variable. By default connects to 22 and then change it to the new one.
- Secure SSH, disableing root login and more.
- Copy ssh public keys to desired user. Put your key in public_kyes/ directory and edit hardering.yml
- Secure Ubuntu using  Center for Internet Security (CIS) and florianutz role from ansible galaxy.
- Install and configure Fail2Ban using oefenweb role. Jails for ssh and apache.
- Secure apache config using best practices from juju4/ansible-harden-apache role. Modified to allow ip whitelisting.

## Installing Ansible

```bash
# sudo apt install software-properties-common 
# sudo apt-add-repository --yes --update ppa:ansible/ansible 
# sudo apt install ansible sshpass python3-selinux python3-semanage
```

## Clone this repository 

```bash
# git clone https://github.com/franrebo84/ansible_hardering_apache.git
# cd ansible_hardering_apache
```

## Preerequisites

Install required roles:

```bash 
# ansible-galaxy install -r requirements.yml
```


## Setting inventory file

Edit inventory file and change IP address, username, password and desired ssh port.

```
[servers]
myserver ansible_host=192.168.88.21 ansible_user=ubuntu ansible_password=ubuntu ansible_become_pass=ubuntu ansible_port=1234
```


## Edit Hardering.yaml variables

### User 

```
 - name: Set up multiple authorized keys
     authorized_key:
       user: ubuntu <<<<<<<<<<<<<<< CHANGE THIS to your user
       state: present
       key: '{{ item }}'
     with_file:
       - public_keys/fran.pub
```


### IP Whitelist for modsecurity

ModSecurity blocks request with IP instead full URL in headers. To whitelist clients, edit the following variables list:


```
   roles:
   - role: ssh_hardering
     gather_facts: no
   - role: ansible-harden-apache
     vars:
       whitelist_headers:
         - 192.168.88.37   <<<<< Whitelist IP
         - 192.168.88.38
     tags: apache
```


## Runing the Playbooks

Firt you must run ssh.yml playbook. This will change default ssh port to whatever is defined in inventory.

```bash
# ansible-playbook -i inventory ssh.yml -v
```

Then, execute the following playbook:

```bash
# ansible-playbook -i inventory hardering.yml -v
```





