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
    - docker                   #Run the docker role
    - env                      #Copy env file with variable to the server
    - create_network           #Create 2 network to separate the back and the front
    - launch_database          #Lauch the database container to the server from the docker hub image
    - launch_app               #Lauch the simpleapi (backend) container to the server from the docker hub image
    - launch_proxy             #Lauch the http container to the server from the docker hub image
```

---

You can check every main from different roles. The code is commented

### 3-3 Document your docker_container tasks configuration.

#### **docker**
- [tasks/main.yml](../ansible/roles/docker/tasks/main.yml)  
  Tasks to install Docker and related dependencies.

---

#### **env**
- [tasks/main.yml](../ansible/roles/env/tasks/main.yml)  
  Task to copy environment files or set environment variables. **COMMENTED** Doesnt work maybe because there is an autorization issue

---

#### **create_network**
- [tasks/main.yml](../ansible/roles/create_network/tasks/main.yml)  
  Tasks to create Docker networks (internal, public, front).

---

#### **launch_database**
- [tasks/main.yml](../ansible/roles/launch_database/tasks/main.yml)  
  Tasks to launch and configure the Postgres container.

---

#### **launch_app**
- [tasks/main.yml](../ansible/roles/launch_app/tasks/main.yml)  
  Tasks to launch the backend application container.

---

#### **launch_proxy**
- [tasks/main.yml](../ansible/roles/launch_proxy/tasks/main.yml)  
  Tasks to launch the HTTPD or Nginx reverse proxy container.

---

##### **launch_front**
- [tasks/main.yml](../ansible/roles/launch_front/tasks/main.yml)  
  Tasks to launch the frontend container (static files served by Nginx).

---

### Is it really safe to deploy automatically every new image on the hub ? explain. What can I do to make it more secure?

Automatically deploying every new image from Docker Hub is risky because it can introduce vulnerabilities, bugs, or malicious code. To make it more secure, we can do:

- **Pin Specific Image Versions**: Avoid using `latest` and we can specify version
- **Test in CI/CD**
- **Limit Permissions**: Ensure least privilege access to pull/deploy images and hide variables
- **Enable Image Signing**: use Docker Content Trust or Notary to verify image authenticity
- **Rollback Mechanisms**: Ensure you can revert to a previous stable version if needed
