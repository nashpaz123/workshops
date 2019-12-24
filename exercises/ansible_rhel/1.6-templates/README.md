# Exercise 1.6 - Templates

**Read this in other languages**: ![uk](../../../images/uk.png) [English](README.md),  ![japan](../../../images/japan.png) [日本語](README.ja.md).

Ansible uses Jinja2 templating to modify files before they are distributed to managed hosts. Jinja2 is one of the most used template engines for Python (<http://jinja.pocoo.org/>).

## Step 6.1 - Using Templates in Playbooks

When a template for a file has been created, it can be deployed to the managed hosts using the `template` module, which supports the transfer of a local file from the control node to the managed hosts.

As an example of using templates you will change the motd file to contain host-specific data.

First in the `~/ansible-files/` directory create the template file `motd-facts.j2`:

<!-- {% raw %} -->
```html+jinja
Welcome to {{ ansible_hostname }}.
{{ ansible_distribution }} {{ ansible_distribution_version}}
deployed on {{ ansible_architecture }} architecture.
```
<!-- {% endraw %} -->

The template file contains the basic text that will later be copied over. It also contains variables which will be replaced on the target machines individually.

Next we need a playbook to use this template. In the `~/ansible-files/` directory create the Playbook `motd-facts.yml`:

```yaml
---
- name: Fill motd file with host data
  hosts: node1
  become: yes
  tasks:
    - template:
        src: motd-facts.j2
        dest: /etc/motd
        owner: root
        group: root
        mode: 0644
```

You have done this a couple of times by now:

  - Understand what the Playbook does.

  - Execute the Playbook `motd-facts.yml`.

  - Login to node1 via SSH and check the message of the day content.

  - Log out of node1.

You should see how Ansible replaces the variables with the facts it discovered from the system.

## Step 6.2 - Challenge Lab

Add a line to the template to list the current kernel of the managed node.

  - Find a fact that contains the kernel version using the commands you learned in the "Ansible Facts" chapter.

> **Tip**
>
> Do a `grep -i` for kernel

  - Change the template to use the fact you found.

  - Run the Playbook again.

  - Check motd by logging in to node1

> **Warning**
>
> **Solution below\!**


  - Find the fact:

```bash
[root@ansible ansible-files]$ ansible node1 -m setup|grep -i kernel
       "ansible_kernel": "3.10.0-693.el7.x86_64",
```

  - Modify the template `motd-facts.j2`:

<!-- {% raw %} -->
```html+jinja
Welcome to {{ ansible_hostname }}.
{{ ansible_distribution }} {{ ansible_distribution_version}}
deployed on {{ ansible_architecture }} architecture
running kernel {{ ansible_kernel }}.
```
<!-- {% endraw %} -->

  - Run the playbook.
  - Verify the new message via SSH login to `node1`.


## Extra Jinja2 - What does the Ansible template module do?
Ansible’s template module transfers templated files to remote hosts. It works similarly to the copy module, but with 2 major differences:

template looks for templates in ./templates/ when you supply a relative path for src (instead of ./files/ for copy)
You can use the jinja2 templating language in your files, which will be templated out separately for each remote host
The template module is extremely useful for doing machine-specific configuration based on Ansible variables.

Given the following file structure:

```yaml
.
├── ansible.cfg
├── templates
│   └── my_app.conf.j2
└── playbook.yml
```

With ./templates/my_app.conf.j2 containing:

```yaml
local_ip = {{ ansible_default_ipv4["address"] }}
local_user = {{ ansible_user }}
```

Running the following task in playbook.yml:

```yaml
- name: install my_app configuration file from template
  template:
    src: my_app.conf.j2
    dest: $HOME/my_app.conf
```    
    
Produces the following on my Ubuntu 18.04 test host:

```yaml
local_ip = 10.1.11.72
local_user = ubuntu
```

And the following on my Centos 7.5 test host:

```yaml
local_ip = 10.1.11.62
local_user = centos
```

## Special variables available inside templates
The following variables are available inside templates in addition to all available Ansible variables:

ansible_managed - Usually put at the start of a templated file to indicate that the file is managed by Ansible and should not be modified manually. Configurable in ansible.cfg, please see the ansible_managed section of Ansible’s configuration guide for more details.
template_host - The node name of the host that executed the template (the local host).
template_uid - The numeric user id of the template file owner on the local host.
template_path- The absolute path to the template on the local host.
template_fullpath - Same as template_path in my experience.
template_run_date - The date the template was rendered.

Given the following file structure:

```yaml
.
├── ansible.cfg
├── templates
│   └── special_variables.j2
└── playbook.yml
```

With ./templates/special_variables.j2 containing:

```yaml
ansible_managed = "{{ ansible_managed }}"

template_host = "{{ template_host }}"

template_uid = "{{ template_uid }}"

template_path = "{{ template_path }}"

template_fullpath = "{{ template_fullpath }}"

template_run_date = "{{ template_run_date }}"
```

Will produce the following templated content:

```yaml
ansible_managed = "Ansible managed"

template_host = "AnsibleController.local"

template_uid = "root"

template_path = "/path/to/ansible/templates/special_variables.j2"

template_fullpath = "/path/to/ansible/templates/special_variables.j2"

template_run_date = "2019-12-18 15:24:05.185182"
```

## Examples
#### How to template a file onto a remote host
Set the src parameter to a jinja2 template in your ./templates/ directory and dest to the location on the remote host.
```yaml
- name: template file to remote host
  template:
    src: my_app.conf.j2
    dest: $HOME/my_app.conf
```
#### How to set ownership and file permissions when templating a file onto a remote host
The owner, group and mode parameters give you fine control over the ownership and file permissions of the file(s) created by the template module. Remember that you will need become: true if you are setting the owner or group to values that don’t match the ansible_user.

The mode parameter accepts the file permissions setting in the following ways:

An octal number: e.g. 0644, 0600
Symbolic mode format: e.g. u=rw,g=r,o=r (where u is the owner, g is the group, and o is others). The permissions are r for read, w for write and x for execute.
With octal mode:
```yaml
- name: template with ownership and octal mode
  template:
    src: my_app.conf.j2
    dest: $HOME/my_app.conf
    owner: "{{ ansible_user }}"
    group: "{{ ansible_user }}"
    mode: 0644
```
With symbolic mode:
```yaml
- name: template with ownership and symbolic mode
  template:
    src: my_app.conf.j2
    dest: $HOME/my_app.conf
    owner: "{{ ansible_user }}"
    group: "{{ ansible_user }}"
    mode: u=rw,g=r,o=r
```
Setting a different owner with become: true:
```yaml
- name: template with different owner (become required)
  template:
    src: my_app.conf.j2
    dest: /etc/my_app.conf
    owner: root
    group: root
    mode: 0644
  become: true
```  
#### How to test that a file is valid before templating onto the remote host
Use the validate parameter to to check whether a file is valid on the remote host before templating the file into place. This parameter is highly recommended if you have some program that can confirm the whether a target file is valid.

%s in the validate string refers to the file to be checked.

#### How to check the validity of a sudoers file before templating
The /usr/sbin/visudo -cf command checks the validity of a sudoers file:
```yaml
- name: template /etc/sudoers with validation
  template:
    src: sudoers.j2
    dest: /etc/sudoers
    owner: root
    group: root
    mode: 0400
    validate: /usr/sbin/visudo -cf %s
  become: true
```
#### How to check the validity of an nginx configuration file before templating
The /usr/bin/nginx -t -c command checks the validity of an nginx configuration file:
```yaml
- name: template /etc/nginx/nginx.conf with validation
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
    owner: root
    group: root
    mode: 0644
    validate: /usr/bin/nginx -t -c %s
  become: true
```
#### How to backup a file if template needs to overwrite it
By default, the template module will overwrite a file if it already exists on the remote host. Use the backup parameter if you want to backup the file before it is overwritten:
```yaml
- name: template file to remote host with backup
  template:
    src: my_app.conf.j2
    dest: $HOME/my_app.conf
    backup: true
```
With backup: true, template will make a backup and append the timestamp before overwriting the file:
```yaml
.
├── my_app.conf
└── my_app.conf.26507.2018-12-28@06:39:29~
```
#### How to template multiple files with a loop
Use the loop keyword to template multiple files. Remember that you can use the loop variables in your template file as well!

In the example below I’d like to create a .gitconfig for each user on the system. Here’s the template:

templates/gitconfig.j2
```yaml
[user]
  name = {{ user.name }}
  username = {{ user.username }}
  email = {{ user.username }}@example.com

[core]
  excludesfile = /home/{{ user.username }}/.gitignore
  
```  
In the template we reference the user variable, which is going to come from each item in a loop. We can template this into each user’s home directory with the following task:
```yaml
- name: create .gitconfig for each user
  template:
    src: .gitconfig.j2
    dest: "/home/{{ user.username }}/.gitconfig"
    owner: "{{ user.username }}"
    group: "{{ user.username }}"
    mode: 0644
  become: true
  loop:
    - name: John Smith
      username: jsmith
    - name: Jane Doe
      username: jdoe
  loop_control:
    loop_var: user
```    
Notice that we’ve used the loop_control and loop_var directives to change the loop variable to user instead of item.

#### How to capture template module output
Use the register keyword to capture the output of the template module.
```yaml
- name: template file to remote host
  template:
    src: my_app.conf.j2
    dest: $HOME/my_app.conf
  register: template_output

- debug: var=template_output
```
The debug task above will output the following:
```yaml
ok: [123.123.123.123] => {
    "template_output": {
        "changed": true,
        "checksum": "120ec4bdd3cea15350829527ecbdb59604fafa11",
        "dest": "/home/ubuntu/my_app.conf",
        "diff": [],
        "failed": false,
        "gid": 1000,
        "group": "ubuntu",
        "md5sum": "62ca3280a599503f968d8ba8fa2fefd8",
        "mode": "0664",
        "owner": "ubuntu",
        "size": 43,
        "src": "/home/ubuntu/.ansible/tmp/ansible-tmp-1545979819.76-238399833013342/source",
        "state": "file",
        "uid": 1000
    }
}
```
#### How to capture template module output from a loop
When using the template module in a loop, the output of each item in the loop will go into the results key of the registered variable.
```yaml
- name: template multiple file to remote host
  template:
    src: "{{ item }}.j2"
    dest: "$HOME/{{ item }}"
  register: template_output
  loop:
    - my_app.conf
    - welcome_message

- debug: var=template_output
```
```yaml
ok: [54.206.30.148] => {
    "template_output": {
        "changed": true,
        "msg": "All items completed",
        "results": [
            {
                ...
                "changed": true,
                "checksum": "120ec4bdd3cea15350829527ecbdb59604fafa11",
                "dest": "/home/ubuntu/my_app.conf",
                "diff": [],
                "failed": false,
                "gid": 1000,
                "group": "ubuntu",
                "invocation": {
                    "module_args": {...},
                "item": "my_app.conf",
                "md5sum": "62ca3280a599503f968d8ba8fa2fefd8",
                "mode": "0664",
                "owner": "ubuntu",
                "size": 43,
                "src": "/home/ubuntu/.ansible/tmp/ansible-tmp-1545979954.53-190341758633466/source",
                "state": "file",
                "uid": 1000
            },
            {
                ...
```

#### Working With Filters
Filters are a way of transforming template expressions from one kind of data into another. They take some arguments, process them, and return the result. There are a lot of builtin filters which can be used in the Ansible playbook. For example, in the following code myvar is a variable; Ansible will pass myvar to the Jinja2 filter as an argument. The Jinja2 filter will then process it and return the resulting data.

		    {{ myvar | filter }}
Templating happens on the Ansible controller, not on the task’s target host, so filters also execute on the controller as they manipulate local data. Some common uses of filters are as follows :

For formatting data – These filters will take a data structure in a template and render it in a slightly different format. These are occasionally useful for debugging. It may be reading in some already formatted data. Example:

         {{ some_variable | to_json }}
         {{ some_variable | from_json }}.
         
IP address filter – Used to test if a string is a valid IP address. IP address filter can also be used to extract specific information from an IP address. Example:

         {{ myvar | ipaddr }}.
         
URL Split Filter – The urlsplit filter extracts the fragment, hostname, netloc, password, path, port, query, scheme, and username from an URL. With no arguments, returns a dictionary of all the fields. Example:

          {{ "http://user:password@www.acme.com:9000/dir/index.html?query=term#frament" | urlsplit('hostname') }}
Regular Expression Filters – Used to search a string with a regex, use the “regex_search” filter. Example:

          {{ 'foobar' | regex_search('(foo)') }}
List Filters – These filters all operate on list variables. Example:

          {{ list1 | min }}
          {{ list1 | unique }}
          {{ list1 | union(list2) }}.
Defining default values for undefined variables: Jinja2 provides a useful ‘default’ filter, that is often a better approach to failing if a variable is not defined. Example:

          {{ some_variable | default(5) }}

----

[Click here to return to the Ansible for Red Hat Enterprise Linux Workshop](../README.md#section-1---ansible-engine-exercises)
