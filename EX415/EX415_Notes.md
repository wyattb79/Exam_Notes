# Use Red Hat Ansible® Engine

## Install Red Hat Ansible Engine on a control node.

## Configure managed nodes.

## Configure simple inventories.

## Perform basic management of systems.

## Run a provided playbook against specified nodes.

# Configure intrusion detection 

## Install AIDE

## Configure AIDE to monitor critical system files.

# Configure encrypted storage 

## Encrypt and decrypt block devices using LUKS.

## Configure encrypted storage persistence using NBDE.

## Change encrypted storage passphrases.

# Restrict USB devices 

## Install USBGuard
```
yum install usbguard -y
usbguard generate-policy /etc/usbguard/rules.conf
systemctl start usbguard.service
systemctl enabled usbguard.service
```
## Write device policy rules with specific criteria to manage devices

Show devices

```
usbguard list-devices
```

Allow devices 'n' (example 2)
```
usbguard allow-device 2
```

Block devices 'n' (example 1)
```
usbguard block-device 1
```

Show usbguard activity
```
usbguard watch
```
To create rules:

Edit a temporary rules.conf file

install -m 0600 -o root -g root rules.conf /etc/usbguard/rules.conf

systemctl restart usbguard.service

## Manage administrative policy and daemon configuration

USBGuard config file is located at: /etc/usbguard/usbguard-daemon.conf

For devices that match no defined rules, ImplicitPolicyTarget=allow|block|reject

Settings can be modified via IPC by the IPCAllowedUsers or IPCAllowedGroups

Devices attached when daemon starts: PresentDevicePolicy

# Manage system login security using pluggable authentication modules (PAMs) 

## Configure password quality requirements

Password quality file: /etc/security/pwquality.conf

To use, edit etc/pam.d/passwd:

```
password required pam_pwquality.so retry=5
```

To create a password history, add after pam_pwquality.so to /etc/pam.d/system-auth and /etc/pam.d/password-auth:

```
password requisite pam_pwhistory.so remember=5 use_authtok
```

## Configure failed login policy

Edit /etc/pam.d/password-auth:

```
auth required pam_faillock.so preauth silent audit deny=5 unlock_time=300

auth [default=die] pam_faillock.so authfail audit deny=5 unlock_time=300

account require pam_faillock.so
```


Edit /etc/pam.d/system-auth:

```
auth required pam_faillock.so preauth silent audit deny=5 unlock_time=300

auth [default=die] pam_faillock.so authfail audit deny=5 unlock_time=300

account require pam_faillock.so
```

To lock root out, add to the auth lines:

```
even_deny_root
```

To exempt users from lockout, precede first faillock with:

```
auth [success=1 default=ignore] pam_succeed_if.so user in user1:user2:user3
```

View failed logins:

```
faillock
```

Failed login attempted logged at:

```
/var/run/faillock
```

Unlock locked user:

```
faillock --user USER --reset
```

## Modify PAM configuration files and parameters

# Configure system auditing

## Write rules to log auditable events

## Enable prepackaged rules

## Produce audit reports

# Configure SELinux

## Enable SELinux on a host running a simple application

## Interpret SELinux violations and determine remedial action

SELinux logs to /var/log/audit/audit.log

**Analyze audit log**

```
sealert -a /var/log/audit/audit.log
```

**Setup SELinux for a new application**

```
setenforce permissive
```

Run application

```
sealert -a /var/log/audit/audit.log

grep httpd /var/log/audit/audit.log | audit2allow -M se_policy

semodule -i se_policy.pp

setenforce enforcing
```

Journalctl -xe can also give clues

## Restrict user activity with SELinux user mappings

**View user mappings**

```
semanage login -l
```

**Identify user's mapping**

```
id -Z
```

**Map linux user to SELinux user**

```
semanage login -a -s SEUser User
```

**Delete user to SELinux mapping**

```
semanage -d User
```

**Change user mapping**

```
semanage login -m -S targeted -s "SELINUX_USER" -r MLC/MCS LOGIN
```

**sudo limitations*

Only roles sysadm_r and staff_r can use sudo when a user is not unconfined_r

**Allow user to sudo by changing user mapping**

```
semanage login -a -s "staff_r" wyatt
```

Add to /etc/sudoers:

wyatt ALL=(ALL) TYPE=administrator_t
ROLE=administrator_r /bin/sh

Update home directory SEContext:

restorecon -FR -v /home/wyatt

**Allow SEuser to sudo by changing SE mapping**

```
semanage login -m -R "staff_r" SEUSER
```

## Analyze and correct existing SELinux configurations

**List booleans**

```
getsebool -a
```

**List booleans (including description):**

```
semanage boolean -l 
```

**Set boolean persistently**

```
setsebool -P BOOLEAN on
```

# Enforce security compliance

## Install OpenSCAP and Workbench

## Use OpenSCAP and Red Hat Insights to scan hosts for security compliance

## Use OpenSCAP Workbench to tailor policy

## Use OpenSCAP Workbench to scan an individual host for security compliance

## Use Red Hat Satellite server to implement an OpenSCAP policy

## Apply OpenSCAP remediation scripts to hosts
