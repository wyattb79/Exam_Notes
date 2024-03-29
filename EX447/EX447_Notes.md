# Understand and use Git

## Clone a Git repository

## Update, modify and create files in a Git repository

## Add those modified files back into the Git repository

# Manage inventory variables

## Structure host and group variables using multiple files per host or group

## Use special variables to override the host, port, or remote user Ansible uses for a specific host

## Set up directories containing multiple host variable files for some of your managed hosts

## Override the name used in the inventory file with a different name or IP address

# Manage task execution

## Control privilege execution

Create ssh keys with passwords
```
ssh-keygen ...
```

Use ssh-agent after login to store passphrases
```
eval ssh-agent /bin/bash
ssh-add ~/.ssh/id_rsa
ssh-add -l
```

Or add it to ~/.bash_profile
```
if [ -z "$SSH_AUTH_SOCK" ]
  eval `ssh-agent -s`
  ssh-add
fi
```

Remove NOPASSWD from /etc/sudoers and increase timestamp

```
/etc/sudoers:
Defaults timestamp_type=global,timestamp_timeout=120,!requiretty
```

## Run selected tasks

# Transform data with filters and plugins

## Populate variables with data from external sources using lookup plugins

Assert variable:
```
assert:
  that: task_succeeded
  success_msg: Task Success
  fail_msg: Task Failed
```

Check a given value:
```
---
- hosts: localhost
  vars_prompt:
  - name: age
    prompt: "Enter your age"
    private: no
  tasks: 
  - name: Check if age is over 21
    assert:
      that: 
      - "{{  (age | int) <= 120 }}"
      - "{{  (age | int) >= 21 }}"
    fail_msg: "Age must be between 21 and 120"
    success_msg: "Age fits criteria"
```

Fail if value has no value:
```
{{ my_var | mandatory }}
```

Set a default value for var2 if it is empty or has a boolean false value:
```
{{ my_var | default_value(var2, True) }}
```

Capitalize a string:
```
{{ my_var | capitalize }}
```

Create a single list from many lists:
```
union
```

Get a random element:
```
{{ [0, 1, 2] | random }}
```

Sort list:
```
{{ [1, 4, 2, 3] | sort }}
```

Generate password:
```
password_hash
```

Quote:
```
quote
```

## Use lookup and query functions to template data from external sources into playbooks and deployed template files

## Implement loops using structures other than simple lists using lookup plugins and filters

```
vars:
- logdir: "/var/log"
tasks:
  - name: show logfiles
    find:
      paths: "{{ logdir }}"
      recurse: yes
    register: logfiles
  - name: show raw data
    debug:
      var: logfiles
  - name: show files with map and list
    set_fact:
      logfilelist: "{{ logfiles['files'] | map(attribute='path') | list }}"
  - name: show new data
    debug:
      var: logfilelist
```

Flatten: a flat list from a nested struture
```
vars:
  usa:
    - newyork:
        cities:
        - albany
        - buffalo
    - california
        cities:
          losangeles
          sacramento
tasks:
- name: produce a flat list of cities
  debug: 
    msg: "{{ item }}"
  loop: "{{ usa | map(attribute='cities') | flatten }}"
```

Subelements:

```
vars:
  usa:
    - state: newyork
      cities:
        - albany
        - buffalo
    - state: california
      cities:
        - losangeles
        - sacramento
tasks:
  - name: show cities in each state
    debug:
      msg: >
        In state "{{ item.0.state }}"
        there is a city called "{{ item.1.* }}"
    loop: "{{ usa | subelements('cities') }}"
```

dict2items:

```
vars:
  users:
    name: bob
      shell: sh
      groups: admins
    name: jane
      shell: bash
      groups: developers
tasks:
  - name: add users
    user:
      name: "{{ item.key }}"
      shell: "{{ item.value.shell }}"
      groups: "{{ item.value.groups }}"
    loop: "{{ users | dict2items }}"
    

```

## Inspect, validate, and manipulate variables containing networking information with filters

Install required packages for network related filters:

```
yum install python3-netaddr python3-dns -y
```

ipaddr filter examples:

ip4_hex examples:

# Delegate tasks

## Run a task for a managed host on a different host, then control whether facts gathered by that task are delegated to the managed host or the other host

```
- name: test web url
  uri:
    url: http://www2
  delegate_to: www.example.com
```

Copy from www1 to control_node
```
...
hosts: control_node
tasks:
- name: copy /tmp/hosts to /etc/hosts
  copy:
    src: /tmp/hosts
    dest: /etc/hosts
  delegate_to: www1
```

# Install Ansible Tower

## Perform basic configuration of Ansible Tower after configuration

# Manage access for Ansible Tower

## Create Ansible Tower users and teams and make associations of one to the other

# Manage inventories and credentials

## Manage advanced inventories

## Create a dynamic inventory from an identity management server or a database server

## Create machine credentials to access inventory hosts

## Create a source control credential

# Manage projects

## Create a job template

# Manage job workflows

## Create a job workflow template

# Work with the Ansible Tower API

## Write an API scriptlet to launch a job

# Back up Ansible Tower

## Back up an instance of Ansible Tower
