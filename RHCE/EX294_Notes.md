# Automate standard RHCSA tasks using Ansible modules that work with:

* Software packages and repositories

* Services

* Firewall rules

* File systems

* Storage devices

* File content

* Archiving

** Directory archive **

```
- hosts: www.example.com
  tasks:
    - name: archive /home/users/wyatt
      archive:
        path: /home/users/wyatt
	format: gz
	dest: /tmp/wyatt.tgz
```

** Multiple file archive **

```
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

** Wildcard archive **

```
- hosts: www.example.com
  tasks:
    - name: archive /home/users/wyatt/files/file[1,2,4] but skip file3
      archive:
        path: /home/users/wyatt/files/file*
	excluse_path: /home/users/wyatt/files/file3
      format: gz
      dest: /tmp/file124.tgz
```

** Unarchive **

```
- hosts: www.example.com
  tasks:
    - name: unarchive /tmp/file124.tgz
      unarchive:
        src: /tmp/file124.tgz
	dest: /home/users/wyatt/new_files
	remote_src: yes # remote_src says the file is located on the remote box
```

* Task scheduling

* Security

* Users and groups
