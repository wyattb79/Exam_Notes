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

No curly braces are used in a conditional statement in a playbook or in a template


## Facts

**Custom Facts**

Defined in /etc/ansible/facts.d/FILENAME.fact

Available in ansible_facts.ansible_local

**Example Custom Fact File**

[my_custom_facts]

package = httpd
state = present


```
---
- name: install from custom fact file
  hosts: www.example.com
  become: yes
  tasks:
  - name: install packages from custom facts
    yum:
      name: "{{ ansible_local.FILENAME.my_custom_facts.package }}"
      state: "{{ ansible_local.FILENAME.my_custom_facts.state }}"
```




## Loops

## Conditional tasks

```
---
- name: restart sshd if httpd is running
  hosts: www.example.com
  tasks:
  - name: get httpd status
    command: systemctl is-active httpd
    ignore_errors: yes
    register httpd_status
  - name: restart sshd
    service:
      name: sshd
      state: restarted
    when: httpd_status.rc == 0
```

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

**Use a role**

```
---
- name: ...
  roles:
    - my_role
```

**Create a custom role**

```
ansible-galaxy init my_custom_role
```

**RHEL System Roles**

Install:
```
yum install rhel-system-roles
```

Docs:
```
/usr/share/doc/rhel-system-roles
```

## Use provided documentation to look up specific information about Ansible modules and commands

# Use roles and Ansible Content Collections

## Create and work with roles

## Install roles and use them in playbooks

## Install Content Collections and use them in playbooks

```
ansible-galxy collection install ansible.posix

---
- name: play name
  collections:
    - ansible.posix
  tasks:
  - name: ...
    selinux: 
```

## Obtain a set of related roles, supplementary modules, and other content from content collections, and use them in a playbook.

ansible-galaxy collection install -r requirements.yaml

```
requirements.yaml:

collections:
  - ansible.netcommon
    source: https://galaxy.ansible.com
  - ansible.posix
    source: https://galaxy.ansible.com
```

# Install and configure an Ansible control node

## Install required packages

subscription-manager repos --list | grep ansible

subscription-manager repos --enable=OUTPUT

yum install -y ansible

To add python to a managed node remotely using Ansible:
ansible -u root -i inventory HOST --ask-pass -m raw -a 'yum install python3 -y'

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

yum install python3

alternatives --set python /usr/bin/python3

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

#### Ignore errors

```
---
- hosts: www.example.com
  tasks:
    - name: pull initial index.html, ignoring errors
      fetch:
        src: /downloads/www.index.html
	dest: /var/www/html/index.html
      ignore_errors: yes
```

#### Block and rescue

```
---
- hosts: www.example.com
  tasks:
    - name: pull initial index.html, handle failure
      block:
        - fetch:
          src: /downloads/www/index.html
	  dest: /var/www/html/index.html
      rescue:
        - copy:
	    content: Webserver down for maintenance.  Check back later.
	    dest: /var/www/html/indx.html
      always:
        - service:
	    name: httpd
	    state: restarted

```

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

**Setup filesystem on logical volume**
```
---
- name: setup filesystem
  hosts: www.example.com
  tasks:
  - name: create filesystem on logical volume
    filesystem:
      dev: "/dev/vgstorage/lvlogs"
      fstype: xfs
```

## Storage devices

**Setup partitions**
```
---
- name: setup partitions
  hosts: www.example.com
  tasks:
  - name: create 1 GiB partition on /dev/sdb1
    parted:
      device: /dev/sdb
      state: present
      number: 1
      part_start: 1MiB
      part_end: 1GiB
```

**Setup volume group**
```
---
- name: setup logical volume group
  hosts: www.example.com
  tasks:
  - name: create volume group from PVs
    lvg:
      vg: /dev/sdb1
      name: vgstorage
```

**Setup logical volume on volume group**
```
---
- name: setup logical volume if needed
  hosts: www.example.com
  tasks:
  - name: create logical volume on volume group
    lvol:
      vg: vgstorage
      lv: lvlogs
      size: 512M
    when: lvlogs not in ansible_lvm["lvs"]
```
**Mount LVs**
```
---
- name: mount logical volume
  hosts: www.example.com
  tasks:
  - name: mount volume
    mount:
      path: /var/log
      source: "/dev/vgstorage/lvlogs"
      fstype: xfs
      state: mounted
```

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

**Add block to file**
```
---
- hosts: www.example.com
  tasks:
    - name: add block to file
      blockinfile:
        path: /tmp/index.html
	block: |
	  Welcome to the webserver
	  This page under construction
	state: present    
```

**Checksum file**

```
---
- hosts: www.example.com
  tasks:
  - name: hash file
    stat:
      path: /tmp/index.html
      checksum_algorithm: md5
    register: result
  - name: show checksum
    debug:
      msg: Checksum is {{ result.stat.checksum }}
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

#### Cron

```
---
- hosts: www.example.com
  become: yes
  tasks:
    - name: perform daily yum update
      cron:
        name: "daily yum update"
	minute: "*"
	hour: "3"
	month: "*"
	weekday: "*"
	user: root
	state: present
	job: "yum update -y"
```

#### At
```
---
- hosts: www.example.com
  become: yes
  tasks:
    - name install at
      yum:
        name: at
	state: present
    - name: run yum update in 2 hours
      at:
        command: "yum update -y"
	count: 2
	units: hours
	state: present
```

## Security

**SELinux**

Package policycoreutils-python-utils need to be present to use SELinux

**Enable SELinux**

```
- name: ensure SELinux is enabled and enforcing
  selinux:
    policy: targeted
    state: enforcing
```
**Set context on policy**

```
- name: set context on new documentroot
  sefcontext:
    target: '/web(/.*)?'
    setype: httpd_sys_content_t
    state: present
```

**Restorecon**

```
- name: run restorecon
  command: restorecon -Rv /web
```

**SEBoolean**

```
- name: allow the web server to run user content
  seboolean:
    name: httpd_read_user_content
    state: yes
    persistent: yes
```

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
**Add authorized key**
```
---
- hosts: www.example.com
  tasks:
  - name: add authrozed key
    authorized_key:
      user: {{ item.name }}
      key: "{{ lookup('file', 'keys/' + item.name + '/id_rsa.pub') }}" 
```

**Set default multiuser state**
```
---
- hosts: www.example.com
  tasks:
  - name: set default boot target
    file:
      src: /usr/lib/systemd/system/multi-user.target
      dest: /etc/systemd/system/default.target
      state: link
```

# Manage content

## Create and use templates to create customized configuration files

**Jinja2 Template Example**

```
hosts.j2:

{% for host in groups['db_servers'] %}
  {{ hostvars[host]['ansible_default_ipv4']['address'] }} {{ hostvars[host]['ansible_fqdn'] }} {{hostvars[host]['ansible_hostname'] }}
{% endfor %}

playbook template:

- name: install hosts file
  template:
    src: hosts.j2
    dest: /etc/hosts
    owner: root
    group: root
    mode: 0644
```

**Template control structures**

```
{% if ... %}
...
{% else %}
...
{% endif %}

{% for host in groups['webservers'] %}
...
{% endfor %}

```



## Use Ansible Vault in playbooks to protect sensitive data

**Create an encrypted file**

```
ansible-vault create playbook.yml
```

**Create encrypted file with vault password file**

```
ansible-vault create --vault-password-file=vault-pass playbook.yaml
```

**View encrypted file**

```
ansible-vault view playbook.yaml
```

**Edit encrypted file**

```
ansible-vault edit playbook.yaml
```

**Encrypt existing file**

```
ansible-vault encrypt playbook.yaml
```

**Decrypt existing file**

```
ansible-vault decrypt playbook.yaml
```

**Change password of file**

```
ansible-vault rekey
```

**Run playbook that accesses vault encrypted files**

```
--vault-id @prompt
```

**Run with a vault password file**

```
ansible-playbook --vault-password-file=vault-pass playbook.yaml
```

**Run with a vault password**

```
ansible-playbook --ask-vault-pass playbook.yaml
```
