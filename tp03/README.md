# TP part 03

### 3-1 Document your inventory and base commands

Ansible uses an inventory file to define the remote hosts to manage: `ansible/inventories/setup.yml`

```
all:
  vars:
    ansible_user: admin
    ansible_ssh_private_key_file: /absolute/path/to/id_rsa
  children:
    prod:
      hosts:
        florentin.millot.takima.cloud
```
---

#### Useful Flags

* `--become`: Run commands with sudo privileges
* `-u <user>`: Specify remote SSH user (overrides inventory)
* `--private-key <path>`: Specify SSH private key path (overrides inventory)
* `-i <inventory_file>`: Specify the inventory file location
* `all`: To run on all domain names specified into hosts configuration

Check if Ansible can reach my server and authenticate properly:

```
ansible all -i ansible/inventories/setup.yml -m ping
```

Expected response: `"pong"` means successful connection.

---


Retrieve detailed system information (called facts) from remote hosts:

```
ansible all -i ansible/inventories/setup.yml -m setup
```

To filter specific facts, for example to get OS distribution info:

```
ansible all -i ansible/inventories/setup.yml -m setup -a "filter=ansible_distribution*"
```

---


Refresh the package manager cache on remote hosts (usefull to install apache2):

```
ansible all -i ansible/inventories/setup.yml -m apt -a "update_cache=yes" --become
```

---


Install software packages on remote hosts:

```
ansible all -i ansible/inventories/setup.yml -m apt -a "name=apache2 state=present" --become
```

---


Remove software packages from remote hosts:

```
ansible all -i ansible/inventories/setup.yml -m apt -a "name=apache2 state=absent" --become
```

---

Start a system service (Apache2):

```
ansible all -i ansible/inventories/setup.yml -m service -a "name=apache2 state=started" --become
```

---


Stop a system service:

```
ansible all -i ansible/inventories/setup.yml -m service -a "name=apache2 state=stopped" --become
```

---


Restart a system service:

```
ansible all -i ansible/inventories/setup.yml -m service -a "name=apache2 state=restarted" --become
```

---

Example: Create or overwrite a file:

```
ansible all -i ansible/inventories/setup.yml -m shell -a 'echo "<html><h1>Hello World</h1></html>" > /var/www/html/index.html' --become
```

---


Copy files from your local machine to remote hosts:

```
ansible all -i ansible/inventories/setup.yml -m copy -a "src=/local/path/to/file dest=/remote/path owner=root group=root mode=0644" --become
```

### 3-2 Document your playbook

```
- hosts: all                   #Target all hosts in your inventory
  gather_facts: true           #Collect system facts for use in the playbook
  become: true                 #Execute tasks with sudo (root privileges)

  roles:
    - docker                  #Call the 'docker' role to run its tasks
```