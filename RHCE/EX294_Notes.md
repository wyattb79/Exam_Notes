# Understand core components of Ansible

## Inventories

Default location: /etc/ansible/hosts (defined in /etc/ansible/ansible.cfg)

[webservers]

webserver1

webserver2

[databases]

db1

db2

[servers:children]

webservers

databases

ansible -i INVENTORY_FILE 

## Modules

## Variables

## Facts

## Loops

## Conditional tasks

## Plays

## Handling task failure

## Playbooks

## Configuration files

In order of precedence: (also listed in default cfg file)

ANSIBLE_CONFIG environment variable

./ansible.cfg

/etc/ansible/ansible.cfg (default file)

ansible-config list

ansible-config dump

ansible-config view

## Roles

## Use provided documentation to look up specific information about Ansible modules and commands

# Install and configure an Ansible control node

## Install required packages

subscription-manager repos --list | grep ansible

subscription-manager repos --enable OUTPUT

yum install -y ansible

## Create a static host inventory

Default file is in /etc/ansible/hosts

## Create a configuration file

Default file is in /etc/ansible/ansible.cfg

## Create and use static inventories to define groups of hosts

# Configure Ansible managed nodes

## Create and distribute SSH keys to managed nodes

ssh-keygen

ssh-copy-id wyatt@host1.example.com

## Configure privilege escalation on managed nodes

/etc/sudoers.d/ansible

ansible ALL=(ALL) NOPASSWD: ALL

## Deploy files to managed nodes

## Be able to analyze simple shell scripts and convert them to playbooks

# Create Ansible plays and playbooks

## Know how to work with commonly used Ansible modules

## Use variables to retrieve the results of running a command

```
---
- hosts: www.example.com
  tasks:
    - name: create a file
      file:
        path: /home/users/wyatt/file1.txt
	state: touch
      register: touch_output

    - name: show previous command output
      debug:
        msg: "Command output is {{ touch_output }}"
```

## Use conditionals to control play execution

#### Handlers

```
---
- hosts: www.example.com
  tasks:
    - name: update web configuration
      replace:
        path: /etc/httpd/conf/httpd.conf
	regexp: '^ServerAdmin.*$'
	replace: 'ServerAdmin wyatt@example.com'
	backup: yes
      notify: "restart httpd"
  handlers:
    - name: "restart web server"
      service:
        name: httpd
	state: restarted
      listen: "restart httpd"
```

#### When conditional

```
---
- hosts: all
  become: yes
  tasks:
    - name: setup webserver
      copy:
        src: /home/users/wyatt/www/index.html
	dest: /var/www/html/index.html
      when: ansible_hostname == "webserver.example.com"
    
```

#### Looping over a list

```
--- 
- hosts: www.example.com
  become: yes
  tasks:
    - name: add users
      user:
        name: "{{ item }}"
	state: present
	groups: wheel
      loop:
        - wyatt
	- wyatt_dev
 
```

## Configure error handling

## Create playbooks to configure systems to a specified state

# Automate standard RHCSA tasks using Ansible modules that work with:

## Software packages and repositories

#### Install packages

```
---
- hosts: www.example.com
  become: yes
  tasks:
    - name: install a package
      yum:
        name:
	  - at
	  - httpd
	state: latest
```

#### Add a repo
```
---
- hosts: www.example.com
  become: yes
  tasks:
    - name: configure EPEL
      yum_repository:
        name: epel
	descripion: Extra packages for Enterprise Linux
	baseurl: https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
```

## Services

**Start and enable firewalld using service**

```
---
- hosts: www.example.com
  become: yes
  tasks:
    - name: start and enable firewalld
      service:
        name: firewalld
	state: started
	enabled: yes
```

**Start and enable firewalld using systemd**
```
---
- hosts: www.example.com
  become: yes
  tasks:
    - name: start and enable firewalld using systemd
      systemd:
        name: firewalld
	state: started
	enabled: yes
```

## Firewall rules

**Rule for port**

```
---
- hosts: www.example.com
  become: yes
  tasks:
    - name: allow https
      firewalld:
        port:443/tcp
	permanent: yes
	immediate: yes
	state: enabled
```

**Rule for service**
```
---
- hosts: www.example.com
  become: yes
  tasks:
    - name: allow https
      firewalld:
        zone: public
	service: http
	permanent: yes
	immediate: yes
	state: enabled
```

**Rich Rule for port forwarding**
```
---
- hosts: www.example.com
  become: yes
  tasks:
    - name: forward 443 to 8443
      firewalld:
        rich_rule: rule family=ipv4 forward-port port=443 protocol=tcp to-port=8443
	permanent: yes
	immediate: yes
	state: enabled
```

## File systems

## Storage devices

## File content

**Create a file**

```
---
- hosts: www.example.com
  tasks:
    - name: create /home/wyatt/file.txt
      file:
        path: /home/wyatt/file.txt
	state: touch
```

**Add a line to a file**
```
---
- hosts: www.example.com
  tasks:
    - name: add line to file
      lineinfile:
        path: /tmp/file1
	line: This line was added by ansible
	create: yes
```

**Create a file with copy**
```
---
- hosts: www.example.com
  tasks: 
    - name: create file content with copy
      copy:
        content: ### Added by ansible ###
	dest: /tmp/ansible.text
```

**Create a file using a template**
```
---
- hosts: www.example.com
  tasks:
    - name: create a file using a template
      template:
        src: /home/users/wyatt/templates/host_template.j2
	dest: /var/www/html/index.html
```

**host_template.j2**
```
Welcome to {{ ansible_hostname }}
```

**Replace a line in a file**
```
---
- hosts: www.example.com
  tasks:
    - name: replace a line in a file
      replace:
        path: /home/users/wyatt/lines.txt
	regexp: ".*Wyatt$"
	replace: "This line replaced by Ansible"
```

**Replace a line using lineinfile**
```
---
- hosts: www.example.com
  tasks:
    - name: replace using lineinfile
      lineinfile:
        path: /home/users/wyatt/lines.txt
	regexp: "^Wyatt.*$"
	line: "Line that begins with Wyatt was removed"A
```

## Archiving

**Directory archive**

```
---
- hosts: www.example.com
  tasks:
    - name: archive /home/users/wyatt
      archive:
        path: /home/users/wyatt
	format: gz
	dest: /tmp/wyatt.tgz
```

**Multiple file archive**

```
---
- hosts: www.example.com
  tasks:
    - name: archive /home/users/wyatt/files/file[1,2,3,12]
      archive:
        path: 
	  - /home/users/wyatt/files/file1
	  - /home/users/wyatt/files/file2
	  - /home/users/wyatt/files/file3
	  - /home/users/wyatt/files/file12
        format: bz2
	dest: /tmp/files.bz2
```

**Wildcard archive**

```
---
- hosts: www.example.com
  tasks:
    - name: archive /home/users/wyatt/files/file[1,2,4] but skip file3
      archive:
        path: /home/users/wyatt/files/file*
	exclude_path: /home/users/wyatt/files/file3
        format: gz
        dest: /tmp/file124.tgz
```

**Unarchive**

```
---
- hosts: www.example.com
  tasks:
    - name: unarchive /tmp/file124.tgz
      unarchive:
        src: /tmp/file124.tgz
	dest: /home/users/wyatt/new_files
	remote_src: yes # remote_src says the file is located on the remote box
```

## Task scheduling

## Security

## Users and groups

**Create groups**

```
--- 
- hosts: www.example.com
  tasks:
    - name: Create developers and sysadmins groups
      group:
        name: "{{ item }}"
	state: present
      loop:
        - developers
	- sysadmins
```

**Add user**

```
---
- hosts: www.example.com
  tasks:
    - name: Create user wyatt
      user:
        name: wyatt
	comment: Wyatt developer account
	shell: /bin/bash
	groups: developers
	append: yes
	state: present
	uid: 1050
	append: yes
```

**Remove user**
```
---
- hosts: www.example.com
  tasks:
    - name: Remove user bob and their directories
      user:
        name: bob
	state: absent
	remove: yes  # get rid of all directories associated with this user
```
