About
---
Objectives were copied from
https://www.redhat.com/en/services/training/ex407-red-hat-certificate-expertise-ansible-automation
on 2017-08-11. The exam uses RHEL 7. I have no idea what version of ansible its using
but I'll assume ansible >= 2.0.

Objectives
---
* Understand core components of Ansible
** Inventories
*** [Simple Inventories](https://docs.ansible.com/ansible/latest/intro_inventory.html)
 
    The inventory is formatted like an INI file. You can just have a list of
    servers like below and you can specify variables for them on their line in
    the inventory like below:
 
    ```
    server1.example.com
    server2.example.com
    
    # if they have a non standard ssh port or ssh user 
    server2.example.com ansible_port=2222 ansible_user=mmckinst 
 
    # you can specify vars lke this
    server3.example.com timezone=America/Detroit
    ```
    
    The far more likely situation is you'll need to group hosts together in some
    logical fashion. In the below example they're grouped together by
    geography. You specify the groups with `[groupname]` and the servers are
    listed below it.
    
    In addition to specifying variables on individual lines, you an specify them
    for an entire group using `[groupname:vars]` like below.
    
    You can further group groups together using `[bigger_groupname:children]`. In
    the below example, the *michigan* and *ohio* groups are part of a bigger
    group called *midwest*.
    
    ```
    [michigan]
    www1.example.com
    www2.example.com
    db1.example.com
    
    [ohio]
    mail.example.com
    www3.example.com
    db2.example.com
    
    [california]
    wsgi1.example.com
    wsgi2.example.com
    
    [oregon]
    perl1.example.com
    db4.example.com
    
    [mailservers]
    mail[01:75].example.com
    
    [midwest:children]
    michigan
    ohio
    
    [west:children]
    california
    oregon
    
    [usa:children]
    midwest
    west
 
    [midewst:vars]
    timezone='America/Detroit'
 
    [west:vars]
    timezone='America/Los_Angeles'
    ```
*** [Dynamic Inventories](https://docs.ansible.com/ansible/latest/intro_dynamic_inventory.html)

	A dynamic inventory is just a script that returns information in the
    INI-like format described above. It could be a python script, a shell
    script, etc that utilizes an existing inventory system you already have.
	
** [Modules](https://docs.ansible.com/ansible/latest/modules.html)

   Modules are used to do things like install packages, start services, copy
   config files, etc. When you use a module, you pass it arguments. In the below
   example there's two different ways to start and enable *httpd* using the
   *service* module.
   
   ```
   - name: start and enable httpd
     service:
	   name: httpd
	   state: started
	   enabled: yes

   # you can squash all of the above in to one line too
   - name: start and enable httpd
     service: name=httpd state=started enabled=yes
   ```

** [Variables](https://docs.ansible.com/ansible/latest/playbooks_variables.html)
   
   You can specify variables in the inventory like shown earlier.
   
   You can specify variables in a play like below:
   
   ```
   - hosts: webservers
     vars:
       http_port: 80
     tasks:
     - name: open http port {{ http_port }}
	   firewalld:
         port: "{{ http_port }}"
         permanent: true
         state: enabled
   ```

   One common gotcha is if you start the line with a variable like `{{ foo }}`
   you need to quote the entire line like below:

   ```
   # BAD
   - hosts: app_servers
     vars:
       app_path: {{ base_path }}/22

   # GOOD
   - hosts: app_servers
     vars:
       app_path: "{{ base_path }}/22"
   ```

   Facts described below are just variables you can use like `{{ ansible_python_version }}`.

   You can have a vars_files in a play like below:
   
   ```
   vars_files:
    - /vars/external_vars.yml
   ```

** [Facts](https://docs.ansible.com/ansible/latest/playbooks_variables.html#information-discovered-from-systems-facts)

** Plays

   A play consist of one or more *tasks* to run on one or more *hosts*. An example of a play is below:
   
   ```
   ---
   - hosts: webservers
     vars:
       http_port: 80
     tasks:
     - name: open http port {{ http_port }}
	   firewalld:
         port: "{{ http_port }}"
         permanent: true
         state: enabled
		 
     - name: ensure apache is installed
       package:
	     name: httpd 
		 state: present

     - name: put index.html in place
       copy:
	     name: 
         content: '<html><head></head><body>test</body></html>'
         dest: /var/www/html/index.html
         owner: root
         group: root
         mode: 0644
       notify: restart apache

     - name: ensure apache is running (and enable it at boot)
       service: 
	     name: httpd
		 state: started
		 enabled: yes
		 
     handlers:
       - name: restart apache
         service: name=httpd state=restarted
```

** Playbooks

   A playbook has one or more plays in it. Often times it only has one play in it.
   
** Configuration files

   ansible will use the first config file found in this order. It doesn't merge
   settings or anything like that, just uses the first one.

   ```
   * ANSIBLE_CONFIG (an environment variable)
   * ansible.cfg (in the current directory)
   * .ansible.cfg (in the home directory)
   * /etc/ansible/ansible.cfg
   ```

* [Run ad-hoc Ansible commands](https://docs.ansible.com/ansible/latest/intro_adhoc.html)

  ```
  # run a simple command on 'group_name' of servers in the inventory
  ansible -i ~/path/to/inventory group_name -m command -a 'uptime'
  
  # use a module to restart a service on 'group_name' of servers in the inventory
  ansible -i ~/path/to/inventory group_name -m service -a name=httpd state=restarted'

  # use a module to restart a service on all servers
  ansible -i ~/path/to/inventory '*' -m service -a name=httpd state=restarted'
  
  # specify extra-vars
  ansible -i ~/path/to/inventory '*' --extra-vars "http_port=80 service_name=httpd" -m service -a name={{service_name}} state=restarted'

  # specify extra-vars file with '@some_file.json'
  ansible -i ~/path/to/inventory '*' --extra-vars "@some_file.json" -m service -a name={{service_name}} state=restarted'
  ```

* Use both static and dynamic inventories to define groups of hosts
* Utilize an existing dynamic inventory script

  ```
  # static inventory
  ansible -i ~/path/to/inventory group_name -m command -a 'uptime'

  # dynamic inventory
  ansible -i ~/path/to/inventory_script.py group_name -m command -a 'uptime'

  # https://docs.ansible.com/ansible/latest/intro_dynamic_inventory.html#using-inventory-directories-and-multiple-inventory-sources
  #
  # if you need to use multiple inventories or inventory scripts, put them all
  # in ~/path/to/inventory_directory. files marked as executable will be treated
  # as dynamic inventories. don't give a static inventory a .ini extension, it
  # will be ignored, anything else should be fine
  ansible -i ~/path/to/inventory_directory group_name -m command -a 'uptime'
  ```

* Create Ansible plays and playbooks
** Know how to work with commonly used Ansible modules
   
   If you `yum install ansible-doc` you can pull up
   the [index.html](file:///usr/share/doc/ansible-doc/html/index.html) in a
   browser and find a list of all modules as well as their documentation.
   
   If you can't do that, `ansible-doc -l` will list all the modules and a brief
   summary.
   
   To view the docs for a specific module you use `ansible-doc service`.
   
   Commonly used ansible modules: service, package, copy, template, command, shell

** [Use variables to retrieve the results of running a commands](https://docs.ansible.com/ansible/latest/playbooks_variables.html#registered-variables)
** [Use conditionals to control play execution](https://docs.ansible.com/ansible/latest/playbooks_conditionals.html#register-variables)
   ```
   - hosts: web_servers
     tasks:
     - shell: cat /etc/hosts
       register: hosts_result

     - name: touch test_file if /etc/hosts has localhost in it
	   file:
         path: /etc/foo.conf
         owner: root
         group: root
         mode: 0644
       when: hosts_result.stdout.find('localhost') == 1

   - hosts: web_servers
     tasks:
     - shell: python --version
       register: python_version

     - name: touch /etc/cool if python is version 2.7.13
	   file:
         path: /etc/cool
         owner: root
         group: root
         mode: 0644
       when: python_version.stdout == 'Python 2.7.13'
   ```

** [Configure error handling](https://docs.ansible.com/ansible/latest/playbooks_error_handling.html)

   Normally ansible will stop executing when a command fails (it checks the $? exit status) but we can tell it
   to keep going with the *ignore_errors* option like below:

   ```
   - name: this will not be counted as a failure
     command: /bin/false
     ignore_errors: yes
   ```

   A command might always exit with a 0 status since the command ran
   successfully but if there was an error or failure in the output you can use
   the below to determine what qualifies as failure from its stdout.

   ```
   - name: Fail task when the command error output prints FAILED
     command: /usr/bin/example-command -x -y -z
     register: command_result
     failed_when: "'FAILED' in command_result.stderr"
   ```

   You can also define a custom failure by checking the return code:

   ```
   - name: Fail task when both files are identical
     raw: diff foo/file1 bar/file2
     register: diff_cmd
     failed_when: diff_cmd.rc == 0 or diff_cmd.rc >= 2
   ```

** Create playbooks to configure systems to a specified state
** Selectively run specific tasks in playbooks using tags

   You can apply tags to tasks in a playbook and ten selectively the parts of
   the playbook with the tags. In the below example you can run only the package
   installations with `ansible-playbook example.yml --tags "packages"`.

   ```
   - hosts: web_servers
     tasks:
     - name: install httpd
       package:
         name: httpd
         state: present
       tags:
         - packages
     - name: install emacs
       package:
         name: emacs
         state: present
       tags:
         - packages
  
     - name: start and enable httpd
       service:
  	   name: httpd
  	   state: started
  	   enabled: yes
       tags:
  	   - services
   ```

* Create and use templates to create customized configuration files
* Work with Ansible variables and facts
* Create and work with roles

  Create a role:

  `ansible-galaxy init role_name`
  
  You can use installed roles in a playbook like the below
  
  ```
  ---
  - hosts: all
    roles:
    - geerlingguy.ntp
  ```

* Download roles from an Ansible Galaxy and use them

  To install a role in the default role location (`/etc/ansible/roles/`):

  `ansible-galaxy install username.role_name`

  To install a role in a different role location:

  `ansible-galaxy install username.role_name -p /some/dir`

* Manage parallelism

  On the CLI, you can use the `-f` flag to do this. The default is 5 hosts.
  
  You can also specify the number of forks in the config file using `forks = 20`
  or whatever you want.

* [Use Ansible Vault in playbooks to protect sensitive data](https://docs.ansible.com/ansible/latest/playbooks_vault.html)

  ```
  ansible-vault create foo.yml
  ansible-vault edit foo.yml
  ```
  
  You can use the password vault in the *vars* section specifying it as a file
  like normal. Specify `--ask-vault-pass` to be prompted for the vault password
  or use a file that has the password with `--vault-password-file=VAULT_PASSWORD_FILE`.

* Install Ansible Tower and use it to manage systems

  1. Download tarball
  2. Modify inventory to set vars for
  3. Run setup.sh
  
  Project - collection of playbooks
  Inventories - collection of hosts
  Template - has job templates telling which playbooks (from projects) should run on which inventories
  
* Use provided documentation to look up specific information about Ansible modules and commands
   
  If you `yum install ansible-doc` you can pull up
  the [index.html](file:///usr/share/doc/ansible-doc/html/index.html) in a
  browser and find a list of all modules as well as their documentation.
   
  If you can't do that, `ansible-doc -l` will list all the modules and a brief
  summary.
   
  To view the docs for a specific module you use `ansible-doc service`.

Practice
---
* Download the geerlingguy.ntp role from ansible galaxy and use it in a playbook with the following settings:
** Set a custom timezone variable in the playbook
** Set custom NTP servers in a yaml file encrypted with ansible-vault
* Register the integer in `/etc/specialness` as a variable and add conditionals that do the following:
** If the number is greater than 5, create a file at /tmp/too_high
** If the number is less than 5, create a file at /tmp/too_low
** If the number is equal to 5, create a file at /tmp/just_right
* Create an ansible role that does the following:
** Installs httpd package
** Starts and enables the httpd service
** Installs a file at `/var/www/html/index.html` that is generated from a jinja2 template using a variable.
* Run the `/usr/bin/always_exit_one` command but don't fail when it exits 1




Copyright
---
<a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-sa/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/">Creative Commons Attribution-ShareAlike 4.0 International License</a>.
 
```
(c) by Mark McKinstry

This work is licensed under a
Creative Commons Attribution-ShareAlike 4.0 International License.

You should have received a copy of the license along with this
work. If not, see <http://creativecommons.org/licenses/by-sa/4.0/>.
```
