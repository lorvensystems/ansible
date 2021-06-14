# ansible


# Getting Started with Ansible (Ad Hoc Server Management)

## Creating Project Specific Ansible Configuration

The default configurations for ansible resides at /etc/ansible/ansible.cfg. Instead of relying on defaults, we are going to creates  a custom configuration file for our project. The advantage with that is we could take this configurations on any host and execute it the same way, without touching the default system configurations.  This custom configurations will essentially  override the values in /etc/ansible/ansible/cfg.

###  Ansible configuration file

Change into /vagrant/code/chap3 directory on your ansible host. Create a file called ansible.cfg  Add  the following contents to the file.

On Ansible Control node,

```
mkdir /vagrant/code/chap3
cd /vagrant/code/chap3
```

Create **ansible.cfg** in chap3

```
[defaults]
remote_user = vagrant
inventory   = myhosts.ini
```

## Creating Host Inventory

Create a new file called *myhosts.ini* in the same directory.
Let's create three groups as follows,

```
[local]
localhost ansible_connection=local

[app]
192.168.61.12
192.168.61.13

[db]
192.168.61.11
```

* First group contains the localhost, the control host. Since it does not need to be connected over ssh, it mandates we add ansible_connection=local option
* Second group contains  Application Servers. We will add  two app servers to this group.
* Third group holds the information about the database servers.

The inventory file should look like below.

## Setting up passwordless ssh access to inventory hosts

### Generating ssh keypair on control host

Now on control host, execute the following command

```
ssh-keygen -t rsa
```

Now press enter for the passphrase and other queries.

```
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):
Created directory '/root/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
c5:a5:6d:60:56:5a:7b:3c:60:23:b5:0f:1b:cf:f9:fd root@ansible
The key's randomart image is:
+--[ RSA 2048]----+
|          =oO    |
|         + X *   |
|          = B +  |
|         . . O o |
|        S   . =  |
|               ..|
|                o|
|                .|
|                E|
+-----------------+
```

### Copying public key to inventory hosts

Copy public key of control node to other hosts

```
ssh-copy-id vagrant@192.168.61.11

ssh-copy-id vagrant@192.168.61.12

ssh-copy-id vagrant@192.168.61.13

ssh-copy-id vagrant@192.168.61.14
```

See this example output to verify with your output

```
The authenticity of host '192.168.61.11 (192.168.61.11)' can't be established.
RSA key fingerprint is 32:7f:ad:d7:da:63:32:b6:a9:ff:59:af:09:1e:56:22.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.61.11' (RSA) to the list of known hosts.
```

The password for user *vagrant* is *vagrant*

### Validate the passwordless login

Let us check the connection of control node with other hosts

```
ssh vagrant@192.168.61.11

ssh vagrant@192.168.61.12

ssh vagrant@192.168.61.13

ssh vagrant@192.168.61.14
```

### Ansible ping

We will use Ansible to make sure all the hosts are reachable

```
ansible all -m ping

```

[Output]

```
192.168.61.13 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
192.168.61.11 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
192.168.61.12 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
localhost | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

## Ad Hoc commands

Try running following *fire-and-forget* Ad-Hoc commands...

### Run *hostname* command on all hosts

Let us print the hostname of all the hosts

```
ansible all -a hostname
```

[output]

```
localhost | SUCCESS | rc=0 >>
ansible

192.168.61.11 | SUCCESS | rc=0 >>
db

192.168.61.12 | SUCCESS | rc=0 >>
app

192.168.61.13 | SUCCESS | rc=0 >>
app
```

### Check the *uptime*

How long the hosts are *up*?

```
ansible all -a uptime
```

[Output]

```
localhost | SUCCESS | rc=0 >>
 13:17:13 up  2:21,  1 user,  load average: 0.16, 0.03, 0.01

192.168.61.12 | SUCCESS | rc=0 >>
 13:17:14 up  1:50,  2 users,  load average: 0.00, 0.00, 0.00

192.168.61.13 | SUCCESS | rc=0 >>
 13:17:14 up  1:47,  2 users,  load average: 0.00, 0.00, 0.00

192.168.61.11 | SUCCESS | rc=0 >>
 13:17:14 up  1:36,  2 users,  load average: 0.00, 0.00, 0.00
```

### Check memory info on app servers

Does my app servers have any disk space *free*?

```
ansible app -a free
```

[Output]

```
192.168.61.13 | SUCCESS | rc=0 >>
             total       used       free     shared    buffers     cached
Mem:        372916     121480     251436        776      11160      46304
-/+ buffers/cache:      64016     308900
Swap:      4128764          0    4128764

192.168.61.12 | SUCCESS | rc=0 >>
             total       used       free     shared    buffers     cached
Mem:        372916     121984     250932        776      11228      46336
-/+ buffers/cache:      64420     308496
Swap:      4128764          0    4128764
```

### Installing packages

Let us *install* Docker on app servers

```
ansible app -a "yum install -y docker-engine"
```

This command will fail.

[Output]

```
192.168.61.13 | FAILED | rc=1 >>
Loaded plugins: fastestmirror, prioritiesYou need to be root to perform this command.

192.168.61.12 | FAILED | rc=1 >>
Loaded plugins: fastestmirror, prioritiesYou need to be root to perform this command.
```

Run the fillowing command with sudo permissions.

```
ansible app -s -a "yum install -y docker-engine"
```

This will install docker in our app servers

[Output]

```
192.168.61.12 | SUCCESS | rc=0 >>
Loaded plugins: fastestmirror, priorities
Setting up Install Process
Loading mirror speeds from cached hostfile
 * base: mirrors.nhanhoa.com
 * epel: mirror.rise.ph
 * extras: mirror.fibergrid.in
 * updates: mirror.fibergrid.in
283 packages excluded due to repository priority protections
Resolving Dependencies
--> Running transaction check
---> Package docker-engine.x86_64 0:1.7.1-1.el6 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package             Arch         Version              Repository          Size
================================================================================
Installing:
 docker-engine       x86_64       1.7.1-1.el6          local_docker       4.5 M

Transaction Summary
================================================================================
Install       1 Package(s)

Total download size: 4.5 M
Installed size: 19 M
Downloading Packages:
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
  Installing : docker-engine-1.7.1-1.el6.x86_64                             1/1
  Verifying  : docker-engine-1.7.1-1.el6.x86_64                             1/1

Installed:
  docker-engine.x86_64 0:1.7.1-1.el6

Complete!

192.168.61.13 | SUCCESS | rc=0 >>
Loaded plugins: fastestmirror, priorities
Setting up Install Process
Loading mirror speeds from cached hostfile
 * base: mirror.fibergrid.in
 * epel: mirror.rise.ph
 * extras: mirror.fibergrid.in
 * updates: mirror.fibergrid.in
283 packages excluded due to repository priority protections
Resolving Dependencies
--> Running transaction check
---> Package docker-engine.x86_64 0:1.7.1-1.el6 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package             Arch         Version              Repository          Size
================================================================================
Installing:
 docker-engine       x86_64       1.7.1-1.el6          local_docker       4.5 M

Transaction Summary
================================================================================
Install       1 Package(s)

Total download size: 4.5 M
Installed size: 19 M
Downloading Packages:
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
  Installing : docker-engine-1.7.1-1.el6.x86_64                             1/1
  Verifying  : docker-engine-1.7.1-1.el6.x86_64                             1/1

Installed:
  docker-engine.x86_64 0:1.7.1-1.el6

Complete!
```

### Running commands one machine at a time

Do you want a command to run on *one machine at a time* ?

```
ansible all -f 1 -a "free"
```

## Using *modules* to manage the state of infrastructure

### Creating users and groups using *user* and *group*

To create a group

```
ansible app -s -m group -a "name=admin state=present"
```

The output will be,

```
192.168.61.13 | SUCCESS => {
    "changed": true,
    "gid": 501,
    "name": "admin",
    "state": "present",
    "system": false
}
192.168.61.12 | SUCCESS => {
    "changed": true,
    "gid": 501,
    "name": "admin",
    "state": "present",
    "system": false
}
```

To create a user

```
ansible app -s -m user -a "name=devops group=admin createhome=yes"
```

This will create user *devops*,

```
192.168.61.13 | SUCCESS => {
    "changed": true,
    "comment": "",
    "createhome": true,
    "group": 501,
    "home": "/home/devops",
    "name": "devops",
    "shell": "/bin/bash",
    "state": "present",
    "system": false,
    "uid": 501
}
192.168.61.12 | SUCCESS => {
    "changed": true,
    "comment": "",
    "createhome": true,
    "group": 501,
    "home": "/home/devops",
    "name": "devops",
    "shell": "/bin/bash",
    "state": "present",
    "system": false,
    "uid": 501
}
```

### Copy a file using *copy* modules

We will copy file from control node to app servers.

```
ansible app -m copy -a "src=/vagrant/test.txt dest=/tmp/test.txt"
```

File will be copied over to our app server machines...

```
192.168.61.13 | SUCCESS => {
    "changed": true,
    "checksum": "3160f8f941c330444aac253a9e6420cd1a65bfe2",
    "dest": "/tmp/test.txt",
    "gid": 500,
    "group": "vagrant",
    "md5sum": "9052de4cff7e8a18de586f785e711b97",
    "mode": "0664",
    "owner": "vagrant",
    "size": 11,
    "src": "/home/vagrant/.ansible/tmp/ansible-tmp-1472991990.29-63683023616899/source",
    "state": "file",
    "uid": 500
}
192.168.61.12 | SUCCESS => {
    "changed": true,
    "checksum": "3160f8f941c330444aac253a9e6420cd1a65bfe2",
    "dest": "/tmp/test.txt",
    "gid": 500,
    "group": "vagrant",
    "md5sum": "9052de4cff7e8a18de586f785e711b97",
    "mode": "0664",
    "owner": "vagrant",
    "size": 11,
    "src": "/home/vagrant/.ansible/tmp/ansible-tmp-1472991990.26-218089785548663/source",
    "state": "file",
    "uid": 500
}

```

## Exercises:
1. Add another system group (not inventory group) called *lb* in inventory with respective host ip
2. Add a system user called *joe* on all  app servers. Make sure that the user has a home directory
3. Install  package *vim* using the correct *Ad-Hoc* command
4. Examine all the available module
http://docs.ansible.com/ansible/modules_by_category.html
5. Find out the difference between the *command* module and *shell* module. Try running the following command with both these modules,

```
"free | grep -i swap"
```

6. Use command module to show *uptime* on the host  
7. Install docker-engine using the yum/apt module
8. Using docker-image module, pull *hello-world* image on web server


# Learning to Write Playbooks

In this tutorial we are going to create a simple playbook to add system users, install and start ntp service and some basic utilities.

## Anatomy of a Playbook

## YAML - The Language to write Playbooks


## Writing and executing our first Playbook

**Problem Statement**  

You have to create a playbook which will  
  * create a admin user with uid 5001
  * remove  user dojo
  * install tree  utility
  * install ntp

on all systems which belong to  prod group in the inventory

To create playbook,

  * Change working directory to /vagrant/code/chap5  


```
cd /vagrant/code/chap5
```



  * Edit myhosts.ini if required and comment the hosts which are absent.

  * Add the following configuration to ansible.cfg.   

```
retry_files_save_path = /tmp
```

  This defines the path where retry files are created in case of failed ansible run. We will learn about this in later in this chapter.


  * Create a new file with name *playbook.yml* and add the following content to it

```
---
  - name: Base Configurations for ALL hosts
    hosts: all
    become: true
    tasks:
      - name: create admin user
        user: name=admin state=present uid=5001

      - name: remove dojo
        user: name=dojo  state=absent

      - name: install tree
        yum:  name=tree  state=present

      - name: install ntp
        yum:  name=ntp   state=present

```

### Validating Syntax

Option 1 : Using --syntax-check option with ansible-playbook

```
ansible-playbook playbook.yml --syntax-check
```


**Exercise:**   Break the syntax, run playbook with --syntax check again, and learn how it works.

Option 2 : Using YAML Linter Online

Another way to validate syntax

  * Visit http://www.yamllinter.com
    ![YAML Linter](../images/ansible/yaml_lint.png)



### Using ansible-playbook utility
We will start using ansible-playbook utility to execute playbooks.

To learn how to use ansible-playbook execute the following command,

```
  ansible-playbook --help

```
```
[output]

Usage: ansible-playbook playbook.yml

Options:
  --ask-become-pass     ask for privilege escalation password
  -k, --ask-pass        ask for connection password
  --ask-su-pass         ask for su password (deprecated, use become)
  -K, --ask-sudo-pass   ask for sudo password (deprecated, use become)
  --ask-vault-pass      ask for vault password
  -b, --become          run operations with become (nopasswd implied)
  --become-method=BECOME_METHOD
                        privilege escalation method to use (default=sudo),
                        valid choices: [ sudo | su | pbrun | pfexec | runas |
                        doas ]

.......
```

#### Dry Run

To execute ansible in a check mode, which will simulate tasks on the remote nodes, without actually committing, ansible provides --check or -C option. This can be invoked as ,

```
ansible-playbook playbook.yml --check

```

or
```
ansible-playbook playbook.yml -C
```

### Listing Hosts, Tasks and Tags in a Playbook

```
ansible-playbook playbook.yml --list-hosts

ansible-playbook playbook.yml --list-tasks

ansible-playbook playbook.yml --list-tags
```

### Executing Actions with Playbooks  

To execute  the playbook, we are going to execute **ansible-playbook** comman with playbook  YAML file as an argument. Since we have already defined the inventory and configurations, additional options are not necessary at this time.

```
ansible-playbook playbook.yml
```

```
[output]

PLAY [Base Configurations for ALL hosts] ***************************************

TASK [setup] *******************************************************************
ok: [192.168.61.14]
ok: [192.168.61.11]
ok: [localhost]
ok: [192.168.61.12]
ok: [192.168.61.13]

TASK [create admin user] *******************************************************
changed: [192.168.61.13]
changed: [192.168.61.12]
changed: [localhost]
changed: [192.168.61.11]
changed: [192.168.61.14]

TASK [remove dojo] *************************************************************
changed: [192.168.61.14]
changed: [localhost]
changed: [192.168.61.12]
changed: [192.168.61.11]
changed: [192.168.61.13]

TASK [install tree] ************************************************************
ok: [localhost]
ok: [192.168.61.13]
ok: [192.168.61.12]
ok: [192.168.61.14]
ok: [192.168.61.11]

TASK [install ntp] *************************************************************
changed: [192.168.61.12]
changed: [192.168.61.13]
changed: [192.168.61.11]
changed: [localhost]
changed: [192.168.61.14]

```

## Error Handling and Debugging


We are now going to add a new task to the playbook that we created. This task would start ntp service on all prod hosts.

When you add this task, make sure the indentation is correct.

```

      - name: start ntp service
        service: name=ntp state=started enabled=yes

```

  * Apply playbook again, check the output


```
ansible-playbook playbook.yml

```

[output]
```
TASK [start ntp service] *******************************************************
fatal: [localhost]: FAILED! => {"changed": false, "failed": true, "msg": "no service or tool found for: ntp"}
fatal: [192.168.61.11]: FAILED! => {"changed": false, "failed": true, "msg": "no service or tool found for: ntp"}
fatal: [192.168.61.12]: FAILED! => {"changed": false, "failed": true, "msg": "no service or tool found for: ntp"}

NO MORE HOSTS LEFT *************************************************************
	to retry, use: --limit @/tmp/playbook.retry

PLAY RECAP *********************************************************************
192.168.61.11              : ok=5    changed=0    unreachable=0    failed=1
192.168.61.12              : ok=5    changed=0    unreachable=0    failed=1
localhost                  : ok=5    changed=0    unreachable=0    failed=1
```
Exercise : There was a intentional error introduced in the code. Identify the error from the log message above, correct it  and run the playbook again. This time you should run it  only on the failed hosts by limiting  with the retry file mentioned above (e.g. --limit @/tmp/playbook.retry )

### Debugging Technique : Step By Step Execution

Ansible provides a way to execute tasks step by step, asking you whether to run or skip each task. This can be useful while debugging issues.

```
ansible-playbook playbook.yml --step
```

[Output]

```
root@control:/workspace/chap5# ansible-playbook playbook.yml --step                             

PLAY [Base Configurations for ALL hosts] ***************************************                
Perform task: TASK: setup (N)o/(y)es/(c)ontinue: y                                              


TASK [setup] *******************************************************************                
y                                                                                               
ok: [app1]                                                                                      
ok: [db]                                                                                        
ok: [app2]                                                                                      
ok: [lb]                                                                                        
Perform task: TASK: create admin user (N)o/(y)es/(c)ontinue:                                    

TASK [create admin user] *******************************************************                
yok: [app2]                                                                                     
ok: [app1]                                                                                      
ok: [db]                                                                                        
ok: [lb]                                                                                        

Perform task: TASK: install tree (N)o/(y)es/(c)ontinue: y                                       

TASK [install tree] ************************************************************                
ok: [app2]                                                                                      
ok: [lb]                                                                                        
ok: [app1]                                                                                      
ok: [db]                                                  
```


### Adding Additional  Play

Problem Statement:

You have to a new play to existing playbook  which will

  * create a app user with uid 5003
  * install git
  * on all app servers in the inventory

Lets add a second play specific to app servers. Add the following block of code in playbook.yml file and save   

```
- name: App Server Configurations
  hosts: app
  become: true
  tasks:
    - name: create app user
      user: name=app state=present uid=5003

    - name: install git
      yum:  name=git  state=present
```

Run the playbook again...  

```
ansible-playbook playbook.yml
```

```
.......

PLAY [App Server Configurations] ***********************************************

TASK [setup] *******************************************************************
ok: [192.168.61.13]
ok: [192.168.61.12]

TASK [create app user] *********************************************************
changed: [192.168.61.12]
changed: [192.168.61.13]

TASK [install git] *************************************************************
ok: [192.168.61.13]
ok: [192.168.61.12]

PLAY RECAP *********************************************************************
192.168.61.11              : ok=6    changed=0    unreachable=0    failed=0
192.168.61.12              : ok=9    changed=1    unreachable=0    failed=0
192.168.61.13              : ok=9    changed=1    unreachable=0    failed=0
192.168.61.14              : ok=6    changed=0    unreachable=0    failed=0
localhost                  : ok=6    changed=0    unreachable=0    failed=0
```

### Limiting the execution to a particular group  

Now run the following command to restrict the playbook execution to *app servers*  

```
ansible-playbook playbook.yml --limit app
```

This will give us the following output, plays will be executed only on app servers...  

```

PLAY [Base Configurations for ALL hosts] ***************************************

TASK [setup] *******************************************************************
ok: [192.168.61.13]
ok: [192.168.61.12]

.........

TASK [start ntp service] *******************************************************
ok: [192.168.61.12]
ok: [192.168.61.13]

PLAY [App Server Configurations] ***********************************************

........

TASK [install git] *************************************************************
ok: [192.168.61.12]
ok: [192.168.61.13]

PLAY RECAP *********************************************************************
192.168.61.12              : ok=9    changed=0    unreachable=0    failed=0
192.168.61.13              : ok=9    changed=0    unreachable=0    failed=0

```


## Exercise:

### **Exercise1**: Create a Playbook with the following specifications,  

  * It should apply only on local host (ansible host)
  * Should use become method
  * Should create a **user** called webadmin with shell as "/bin/sh"
  * Should install and start **nginx** service
  * Should **deploy** a sample html app into the default web root directory of nginx using ansible's **git** module.
    * Source repo:  https://github.com/schoolofdevops/html-sample-app
    * Deploy Path : /usr/share/nginx/html/app
 * Once deployed, validate the site by visting http://CONTROL_HOST_IP/app

### **Exercise 2**: Disable Facts Gathering  

  * Run ansible playbook and observe the output
  * Add the following configuration parameter to ansible.cfg

```
gathering = explicit
```

  * Launch ansible playbook run again, observe the output and compare it with the previous run.
# Chapter 6  : Working with Roles

In this tutorial we are going to create simple, static role for apache which will,
  * Install **httpd** package
  * Configure **httpd.conf**, manage it as a static file
  * Start httpd service
  * Add a notification and a  handler so that whenever the configuration is updated, service is automatically restarted.

### 6.1 Creating Role Scaffolding for Apache  
  * Change working  directory to **/vagrant/code/chap5**

```
cd  chap6
```

  * Create roles directory

```
mkdir roles
```

  * Generate role scaffolding using ansible-galaxy

```
ansible-galaxy init --offline --init-path=roles  apache
```

  * Validate

```
tree roles/
```

[Output]

```
  roles/
    └── apache
        ├── defaults
        │   └── main.yml
        ├── files
        ├── handlers
        │   └── main.yml
        ├── meta
        │   └── main.yml
        ├── README.md
        ├── tasks
        │   └── main.yml
        ├── templates
        ├── tests
        │   ├── inventory
        │   └── test.yml
        └── vars
            └── main.yml

```


### 6.2 Writing Tasks to Install and Start  Apache Web Service

We are going to create three different tasks files, one for each phase of application lifecycle
  * Install
  * Configure
  * Start Service

To begin with, in this part, we will install and start apache.

  *  To install apache, Create **roles/apache/tasks/install.yml**

```
---
- name: Install Apache...
  yum: name=httpd state=latest
```  


  * To start the service, create  **roles/apache/tasks/start.yml** with the following content  

```
---
- name: Starting Apache...
  service: name=httpd state=started
```  


To have these tasks being called, include them into main task.

  * Edit roles/apache/tasks/main.yml

```
---
# tasks file for apache
- include: install.yml
- include: start.yml
```

  * Create a playbook for app servers at /vagrant/chap5/app.yml with following contents

```
  ---
  - hosts: app
    become: true
    roles:
      - apache
```

  * Apply app.yml with ansible-playbook

```
  ansible-playbook app.yml
```

[Output]

```
PLAY [Playbook to configure App Servers] *********************************************************************

TASK [setup] *******************************************************************
ok: [192.168.61.12]
ok: [192.168.61.13]

TASK [apache : Install Apache...] **********************************************
changed: [192.168.61.13]
changed: [192.168.61.12]

TASK [apache : Starting Apache...] *********************************************
changed: [192.168.61.13]
changed: [192.168.61.12]

PLAY RECAP *********************************************************************
192.168.61.12              : ok=3    changed=2    unreachable=0    failed=0
192.168.61.13              : ok=3    changed=2    unreachable=0    failed=0
```


### 6.3 Managing Configuration files for Apache
  * Copy **index.html** and **httpd.conf** from **chap5/helper** to **/roles/apache/files/** directory    

```
    cp helper/index.html helper/httpd.conf roles/apache/files/  
```  

  * Create a task file at **roles/apache/tasks/config.yml** to manage files.    

```
---
- name: Copying configuration files...
  copy: src=httpd.conf
        dest=/etc/httpd.conf
        owner=root group=root mode=0644

- name: Copying index.html file...
  copy: src=index.html
        dest=/var/www/html/index.html
        mode=0777
```  

#### 6.3.2 Adding Notifications and Handlers   

  * Previously we have create a task in roles/apache/tasks/config.yml to copy over httpd.conf to the app server. Update this file to send a notification to restart  service on configuration update.  You simply have to add the line which starts with **notify**

```
  - name: Copying configuration files...
    copy: src=httpd.conf
          dest=/etc/httpd.conf
          owner=root group=root mode=0644
    notify: Restart apache service
```

  * Create the notification handler by updating   **roles/apache/handlers/main.yml**  

```
---
- name: Restart apache service
  service: name=httpd state=restarted
```  

```
  ansible-playbook app.yml
```   


[Output]  
```

PLAY [Playbook to configure App Servers] ***************************************

TASK [setup] *******************************************************************
ok: [192.168.61.13]
ok: [192.168.61.12]

TASK [apache : Installing Apache...] *******************************************
ok: [192.168.61.12]
ok: [192.168.61.13]

TASK [apache : Starting Apache...] *********************************************
ok: [192.168.61.13]
ok: [192.168.61.12]

TASK [apache : Copying configuration files...] *********************************
changed: [192.168.61.12]
changed: [192.168.61.13]

TASK [apache : Copying index.html file...] *************************************
changed: [192.168.61.12]
changed: [192.168.61.13]

RUNNING HANDLER [apache : Restart apache service] ******************************
changed: [192.168.61.12]
changed: [192.168.61.13]

PLAY RECAP *********************************************************************
192.168.61.12              : ok=6    changed=3    unreachable=0    failed=0
192.168.61.13              : ok=6    changed=3    unreachable=0    failed=0

```  

## Troubleshooting Exercise

Did the above command added the configuration files and restarted the service ? But we have already written **config.yml**. Troubleshoot why its not being run and fix it before you proceed.


### 6.4 Base Role and Role Nesting

  * Create a base role with ansible-galaxy utility,  
```
  ansible-galaxy init --offline --init-path=roles base
```  

  * Create tasks for base role by editing  **/roles/base/tasks/main.yml**  

```
---
# tasks file for base
# file: roles/base/tasks/main.yml
  - name: create admin user
    user: name=admin state=present uid=5001

  - name: remove dojo
    user: name=dojo  state=present

  - name: install tree
    yum:  name=tree  state=present

  - name: install ntp
    yum:  name=ntp   state=present

  - name: start ntp service
    service: name=ntpd state=started enabled=yes

```  

  * Define base role as a dependency for  apache role,  
  * Update meta data for Apache by editing **roles/apache/meta/main.yml** and adding the following
```
---
dependencies:
 - {role: base}
```  




### 6.5  Creating a Site Wide Playbook

We will create a site wide playbook, which will call all the plays required to configure the complete infrastructure. Currently we have a single  playbook for App Servers. However, in future we would create many.

  * Create **site.yml** in /vagrant/chap5 directory and add the following content

```
  ---
  # This is a sitewide playbook
  # filename: site.yml
  - include: app.yml

```  



  * Execute sitewide playbook as


```
ansible-playbook site.yml
```

[Output]

```

PLAY [Playbook to configure App Servers] ***************************************

TASK [setup] *******************************************************************
ok: [192.168.61.12]
ok: [192.168.61.13]

TASK [base : create admin user] ************************************************
ok: [192.168.61.12]
ok: [192.168.61.13]

TASK [base : remove dojo] ******************************************************
ok: [192.168.61.12]
ok: [192.168.61.13]

TASK [base : install tree] *****************************************************
ok: [192.168.61.13]
ok: [192.168.61.12]

TASK [base : install ntp] ******************************************************
ok: [192.168.61.13]
ok: [192.168.61.12]

TASK [base : start ntp service] ************************************************
ok: [192.168.61.13]
ok: [192.168.61.12]

TASK [apache : Installing Apache...] *******************************************
ok: [192.168.61.13]
ok: [192.168.61.12]

TASK [apache : Starting Apache...] *********************************************
ok: [192.168.61.13]
ok: [192.168.61.12]

TASK [apache : Copying configuration files...] *********************************
ok: [192.168.61.12]
ok: [192.168.61.13]

TASK [apache : Copying index.html file...] *************************************
ok: [192.168.61.12]
ok: [192.168.61.13]

PLAY RECAP *********************************************************************
192.168.61.12              : ok=10   changed=0    unreachable=0    failed=0
192.168.61.13              : ok=10   changed=0    unreachable=0    failed=0
```


## Exercises

##### Exercise 1: Update Apache Configurations
  * Update httpd.conf and change some configuration parameters. Validate the service restarts on configuration updates by applying the sitewide playbook.


##### Exercise 2: Create MySQL Role
  * Create a Role to install and configure MySQL server   
     *    Create role scaffold for mysql  using ansible-galaxy init  
     *   Create task to install "mysql-server" and "MySQL-python" packages using yum module   
     *    Create a task to start mysqld service   
     *   Manage my.cnf by creating a centralized copy in role and writing a task to copy it to all db hosts. Use helper/my.cnf as a reference. The destination for this file is /etv/my.cnf on db servers.
     *    Write a handler to restart the service on configuration change. Add a notification from the copy resource created earlier.
     * Add a dependency on base role in the metadata for mysql role.  
     *     Create  **db.yml** playbook for configuring all database servers. Create definitions to configure **db** group and to apply **mysql** role.   
# Templates and Variables

In this  tutorial, we  are going to make the roles that we created earlier dynamically by adding templates and defining variables.

## Variables

Variables are of two types

* Automatic Variables/ Facts
* User Defined Variables

Lets try to discover information about our systems by using facts.

### Finding Facts About Systems

* Run the following command to see to facts of db servers  
```
ansible db -m setup
```

[Output]

```
192.168.61.11 | SUCCESS => {
        "ansible_facts": {
            "ansible_all_ipv4_addresses": [
                "10.0.2.15",
                "192.168.61.11"
            ],
            "ansible_all_ipv6_addresses": [
                "fe80::a00:27ff:fe30:3251",
                "fe80::a00:27ff:fe8e:83e0"
.....
                "tz_offset": "+0100",
                "weekday": "Monday",
                "weekday_number": "1",
                "weeknumber": "36",
                "year": "2016"
            }
```

### Filtering facts

* Use filter attribute to extract specific data

```
ansible db -m setup -a "filter=ansible_distribution"
```

[Output]

```
192.168.61.11 | SUCCESS => {
  "ansible_facts": {
      "ansible_distribution": "CentOS"
  },
  "changed": false
}  
```

## Creating Templates for Apache

* Create template for apache configuration    
* This template will change **port number**, **document root** and **index.html** for  apache server    
* Copy **httpd.conf** file from **roles/apache/files/** to **roles/apache/templates**    

```
cp roles/apache/files/httpd.conf roles/apache/templates/httpd.conf.j2
```

* Change your working directory to templates

```
cd roles/apache/templates
```

* Change values of following  parameters by using template variables in **httpd.conf.j2**  
  * Listen
  * DocumentRoot
  * DirectoryIndex

Following code depicts only the parameters changed. Rest of the configurations in *httpd.conf.j2* remain as is

```
Listen {{ apache_port }}
DocumentRoot "{{ custom_root }}"
DirectoryIndex {{ apache_index }} index.html.var
```

* Create a template for index.html as well

```
cp roles/apache/files/index.html roles/apache/templates/index.html.j2
```

* Add the following contents to index.html.j2

```
<html>
<body>
  <h1>  Welcome to Ansible training! </h1>

  <h2> SYSTEM INFO </h2>
  <h4>  ========================= </h4>
  <h3> Operating System : {{ ansible_distribution }} </h3>
  <h3> IP Address : {{ ansible_eth0['ipv4']['address'] }} </h3>

  <h2>  My Favourites </h2>
  <h4>  ========================= </h4>

  <h3> color     : {{ fav['color'] }} </h3>
  <h3> fruit     : {{ fav['fruit']   }} </h3>
  <h3> car       : {{ fav['car']   }} </h3>
  <h3> laptop    : {{ fav['laptop']   }} </h3>
  <h4>  ========================= </h4>

</body>
</html>

```

### Defining Default Variables

* Define values of the variables used in the templates above.  The default values are defined in *roles/apache/defaults/main.yml* . Lets edit that file and add the following,

```
apache_port: 80
custom_root: /var/www/html
apache_index: index.html
fav:
  color: white
  car: fiat
  laptop: dell
  fruit: apple
```

### Updating Tasks to use Templates

* Since we are now using template instead of static file, we need to edit *roles/apache/tasks/config.yml* file and use template module
* Replace **copy** module with **template** modules as follows,

```
---
- name: Creating configuration from templates...
  template: >
    src=httpd.conf.j2
    dest=/etc/httpd.conf
    owner=root
    group=root
    mode=0644
  notify: Restart apache service

- name: Copying index.html file...
  template: >
    src=index.html.j2
    dest=/var/www/html/index.html
    mode=0777

```

* Delete httpd.conf and index.html in files directory

```
  rm roles/apache/files/httpd.conf
  rm roles/apache/files/index.html
```

### Validating

* Let's test this template in action

```
ansible-playbook app.yml
```

[Output]

```
PLAY [Playbook to configure App Servers] ***************************************

TASK [setup] *******************************************************************
ok: [192.168.61.13]
ok: [192.168.61.12]

.....

RUNNING HANDLER [apache : Restart apache service] ******************************
changed: [192.168.61.12]
changed: [192.168.61.13]

PLAY RECAP *********************************************************************
192.168.61.12              : ok=11   changed=3    unreachable=0    failed=0
192.168.61.13              : ok=11   changed=3    unreachable=0    failed=0
```

## Variable Precedence in Action

Lets define the variables from couple of other places, to learn about the Precedence rules. We will create,
* group_vars
* playbook vars

Since we are going to define the variables using multi level hashes, lets define the way hashes behave when defined from multiple places.

Update chap7/ansible.cfg and add the following,

```
hash_behaviour=merge
```

Lets create group_vars and create a group **prod** to define vars common to all prod hosts.

```
cd chap7
mkdir group_vars
cd group_vars
touch prod.yml
```

Edit **group_vars/prod.yml** file and add the following contents,

```
---
  fav:
    color: blue
    fruit: peach
```

Lets also add vars to playbook. Edit app.yml and add vars as below,

```
---
  - name: Playbook to configure App Servers
    hosts: app
    become: true
    vars:
      fav:
        fruit: mango
    roles:
    - apache
```

Execute the playbook and check the output

```
ansible-playbook app.yml
```

If you view the content of the html file generated, you would notice the following,

```
<h3> color     : blue </h3>
<h3> fruit     : mango </h3>
<h3> car       : fiat </h3>
<h3> laptop    : dell </h3>
```


| fav item | role defaults     | group_vars     | playbook_vars |
| :------------- | :------------- | :------------- | :------------- |
| color | white | **blue** |   |
| fruit | fiat      |  peach     | **mango** |
| car | **fiat**       |        |  |
| laptop | **apple**       |        |  |

* value of color comes from group_vars/all.yml
* value of fruit comes from playbook vars
* value of car and laptop comes from role defaults

## Registered  Variables

Lets create a playbook to run a shell command, register the result and display the value of registered variable.

Create **register.yml** in chap6 directory

```
---
  - name: register variable example
    hosts: local
    tasks:
      - name: run a shell command and register result
        shell: "/sbin/ifconfig eth1"
        register: result

      - name: print registered variable
        debug: var=result
```

Execute the playbook to display information about the registered variable.

```
ansible-playbook  register.yml
```

## Adding support for Ubuntu

Apache role that we have developed supports only RedHat based systems at the moment. To add support for ubuntu (app2), we must handle platform specific differences.

e.g.

|   | RedHat | Debian     |
| :------------- | :------------- | :------------- |
| Package Name | httpd       | apache2       |
| Service Name | httpd       | apache2       |

OS specific configurations can be defined by creating role vars and by including those in tasks.

file: roles/apache/vars/RedHat.yml

```
---
apache:
  package:
    name: httpd
  service:
    name: httpd
    status: started
```

file: roles/apache/vars/Debian.yml

```
---
apache:
  package:
    name: apache2
  service:
    name: apache2
    status: started
```

Lets now selectively include those var files from tasks/main.yml .  Also selectively call configurations.
file: role/apache/tasks/main.yml

```
---
# tasks file for apache
  - include_vars: "{{ ansible_os_family }}.yml"
  - include: install.yml
  - include: start.yml
  - include: config_{{ ansible_os_family }}.yml  
```

We are now going to create two different config tasks. Since the current config is applicable to RedHat, lets rename it to config_RedHat.yml

```
mv roles/apache/tasks/config.yml roles/apache/tasks/config_RedHat.yml
```

We will now create a new config for Debian

file: roles/apache/tasks/config_Debian.yml

```
- name: Copying index.html file...
  template: >
    src=index.html.j2
    dest=/var/www/html/index.html
    mode=0777
```

Update tasks and handlers to install and start the correct service

tasks/install.yml

```
---
  - name: install httpd on centos
    package: >
      name={{ apache['package']['name']}}
      state=installed
```

tasks/start.yml

```
---
  - name: start httpd service
    service: >
      name={{ apache['service']['name']}}
      state={{ apache['service']['status']}}
```

handlers/main.yml

```
---
# handlers file for apache
  - name: restart apache service
    service: >
      name={{ apache['service']['name']}}
      state=restarted
```

## Exercises

* Create host specific variables in host_vars/HOSTNAME for one of the app servers, and define some variables values specific to the host. See the output after applying playbook on this node.
* Generate MySQL Configurations dynamically using templates and modules.
  * Create a template for my.cnf.  Name it as roles/mysql/templates/my.cnf.j2
  * Replace parameter values with templates variables
  * Define variables in role defaults.

# Control Structures

In Chapter 7, we will learn about the aspects of conditionals and iterations that affects program's execution flow in Ansible  
Control structures are of two different type

* Conditional  
* Iterative  

## Conditionals

Conditionals structures allow Ansible to choose an alternate path. Ansible does this by using *when* statements

## **When** statements

When statement becomes helpful, when you will want to skip a particular step on a particular host

### Selectively calling install tasks based on platform

* Edit *roles/apache/tasks/main.yml*,

```
---
- include: install.yml
  when: ansible_os_family == 'RedHat'
- include: start.yml
- include: config.yml
```

* This will include *install.yml* only if the OS family is Redhat, otherwise it will skip the installation playbook

### Configuring MySQL server based on boolean flag

* Edit *roles/mysql/tasks/main.yml* and add when statements,

```
---
- include: install.yml

- include: start.yml
  when: mysql.server

- include: config.yml
  when: mysql.server
```

* Edit *db.yml* as follows,

```
---
- name: Playbook to configure DB Servers
  hosts: db
  become: true
  roles:
  - mysql
  vars:
    mysql:
      server: true
      config:
        bind: "{{ ansible_eth0.ipv4.address }}"
```


### Adding conditionals in Jinja2 templates

* Put the following content in *roles/mysql/templates/my.cnf.j2*

```
[mysqld]

{% if mysql.config.datadir is defined %}
datadir={{ mysql['config']['datadir'] }}
{% endif %}

{% if mysql.config.socket is defined %}
socket={{ mysql['config']['socket'] }}
{% endif %}

symbolic-links=0
log-error=/var/log/mysqld.log

{% if mysql.config.pid is defined %}
pid-file={{ mysql['config']['pid']}}
{% endif %}

[client]
user=root
password={{ mysql_root_db_pass }}
```

* These conditions will run flawlessly, because we have already defined these Variables

### Running One Time Tasks

* To see how this works, lets take a look at the code in *roles/mysql/tasks/config.yml*

```
    [...]
- name: reset default root password
shell: mysql --user=root --password="{{ MYSQL_DEFAULT_PASS }}" --connect-expired-password mysql < /root/.mysql_reset_pass.sql
run_once: true
ignore_errors: yes
     [...]
```

* In some cases there may be a need to only run a task one time and only on one host. This can be achieved by configuring “run_once” on a task

### Conditional Execution of Roles

* This will execute app playbook only if the node is running **RedHat** family
* Update app.yml to restrict role to be run only on RedHat platform.

```
---
  - name: Playbook to configure App Servers
    hosts: app
    become: true
    vars:
      fav:
        fruit: mango
    roles:
    - { role: apache, when: ansible_os_family == 'RedHat' }
```

* Let's run this code

```
ansible-playbook site.yml
```
[Output]

```
TASK [setup] *******************************************************************
ok: [192.168.61.12]
ok: [192.168.61.13]

TASK [base : create admin user] ************************************************
skipping: [192.168.61.12]
skipping: [192.168.61.13]

TASK [base : remove dojo] ******************************************************
skipping: [192.168.61.12]
skipping: [192.168.61.13]

TASK [base : install tree] *****************************************************
skipping: [192.168.61.12]
skipping: [192.168.61.13]

TASK [base : install ntp] ******************************************************
skipping: [192.168.61.12]
skipping: [192.168.61.13]

TASK [base : start ntp service] ************************************************
skipping: [192.168.61.12]
skipping: [192.168.61.13]

TASK [apache : Installing Apache...] *******************************************
skipping: [192.168.61.12]
skipping: [192.168.61.13]

TASK [apache : Starting Apache...] *********************************************
skipping: [192.168.61.12]
skipping: [192.168.61.13]

TASK [apache : Creating configuration from templates...] ***********************
skipping: [192.168.61.12]
skipping: [192.168.61.13]

TASK [apache : Copying index.html file...] *************************************
skipping: [192.168.61.12]
skipping: [192.168.61.13]

```

**Exercise**: Try using **Debian** instead of **RedHat** . You shall see app role being skipped altogether. Don't forget to put it back after you try this out.

## Iterations

### Iteration over list

* Create a list of packages  
* Let us create the following list of packages in base role.  
* Edit *roles/base/defaults/main.yml* and put

```
---
# packages list
demolist:
  packages:
    - atk
    - flac
    - eggdbus
    - pixman
    - polkit

```

* Also edit *roles/base/tasks/main.yml* to iterate over this list of items and install packages

```
- name: install a list of packages
  yum:
    name: "{{ item }}"
  with_items: {{ demolist.packages }}
```

* Let's check the output

```
TASK [base : install a list of packages] ***************************************
changed: [192.168.61.12] => (item=[u'atk', u'flac', u'eggdbus', u'polkit', u'pixman'])
changed: [192.168.61.13] => (item=[u'atk', u'flac', u'eggdbus', u'polkit', u'pixman'])
```

### Iterating over a Hash Table/Dictionary

* This iteration can be done with using **with_dict** statement, let us see how.
* Edit *group_vars/all* file from the **parent directory** and define a dictionary of mysql databases and users to be created

```
---
  fav:
    color: blue
    fruit: peach
  mysql_bind: "{{ ansible_eth0.ipv4.address }}"
  mysql:
    databases:
      infinity:
        state: present
      peace:
        state: present
    users:
      dojo:
        pass: PassWord@1234
        host: '%'
        priv: '*.*:ALL'
        state: present
      koko:
        pass: f8Usg3ord@1we28
        host: '%'
        priv: '*.*:ALL'
        state: present

```

* **Append** the following iteration in *roles/mysql/tasks/config.yml*

```
- name: create mysql databases
  mysql_db:
    name: "{{ item.key }}"
    state: "{{ item.value.state }}"
  with_dict: "{{ mysql['databases'] }}"

- name: create mysql users
  mysql_user:
    name: "{{ item.key }}"
    host: "{{ item.value.host }}"
    password: "{{ item.value.pass }}"
    priv: "{{ item.value.priv }}"
    state: "{{ item.value.state }}"
  with_dict: "{{ mysql['users'] }}"
```

* Execute the *db* playbook to verify the output

```
ansible-playbook db.yml
```

## Exercises

* Define dictionary of properties for a new database user  in group_vars/all. Observe if it gets created automatically  output by running db.yml playbook. Validate if the user is been actually present by logging on to the mysql server and checking status.
* Update index.html.j2 to iterate over the dictionary of favorites and generate html content to display it instead of adding multiple lines.
* Define a hash/dictionary  of apache virtual hosts to be created  and create a template which would iterate over that dictionary and create vhost configurations.
* Learn about what else you could loop over, as well as how to do so by reading this document http://docs.ansible.com/ansible/playbooks_loops.html#id12
