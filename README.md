# WebServerCluster
It's my assignment.<br/>

## About The Assignment

Use Docker(-compose) and Ansible to deploy and provision a simple web server cluster having 3 servers.

- 1 load balancer using nginx or haproxy
- 2 web servers running a simple hello world using PHP

Please use centos or ubuntu as a base docker image for the instances.<br/>


![Diagram Screen Shot](diagram-web-server-cluster.png?raw=true "Title")<br/>

<!-- GETTING STARTED -->
## Getting Started

### Project's directory/files structure

The main folder in the project is the 'ansible-playbook' directory that includes all the files we need to raise the cluster. I have used the 'ansible roles directory structure' standard and you can find some folders like:

- site.yml, for master playbook
- roles, for ansible roles 
- group_vars, for assign variables to particular groups
- inventory.yml, for the load balancer and webservers servers

#### roles directory
``` bash
ansible-playbook
├── ansible.cfg
├── group_vars
│   ├── loadbalancers.yml
│   └── webservers.yaml
├── inventory.yml
├── requirements.yaml
├── roles
│   ├── ansible-docker
│   ├── ansible-docker-compose
│   ├── ansible-haproxy
│   └── ansible-php
```


We will use four prominent roles to create a web cluster with the ansible:


- ansible-docker: This role installs and configures Docker and Docker Compos for us. 
```bash
── ansible-docker
   ├── defaults
   │   └── main.yml
   ├── handlers
   │   └── main.yml
   ├── tasks
   │   ├── config.yml
   │   ├── install.yml
   │   └── main.yml
   └── templates
       └── etc
```
- ansible-docker-compose: This role manages services prerequisites and counters that run with Docker Compos . in the 'files' directory, you can see project folder includes Dockerfile, docker-compose.yml and other project's files. 
``` bash
── ansible-docker-compose
   ├── defaults
   │   └── main.yml
   ├── files
   │   ├── loadbalancer
   │   └── webserver
   └── tasks
       └── main.yml
```
- ansible-haproxy: It creates haproxy.cfg file in place(roles/ansible-docker-compose/files/loadbalancer/haproxy.cfg) 
``` bash 
── ansible-haproxy
   ├── defaults
   │   └── main.yml
   ├── tasks
   │   └── main.yml
   └── templates
       ├── backend.cfg.j2
       ├── frontend.cfg.j2
       ├── haproxy.cfg.j2
       └── listen.cfg.j2
```
- ansible-php: It creates php.ini file in place(roles/ansible-docker-compose/files/webserver/php.ini) 
``` bash 
── ansible-php
   ├── defaults
   │   └── main.yml
   ├── tasks
   │   ├── configure.yml
   │   └── main.yml
   └── templates
       └── php.ini.j2
```

#### site.yml file

In the site.yml file, you can see how roles are categorized based on host inventory, and the following tags have been given:

``` yaml
- hosts: webservers
  roles:
    - role: ansible-docker
      tags: ['docker-role']

    - role: ansible-php
      tags: ['php-role','update']

    - role: ansible-docker-compose
      tags: ['docker-compose-role','update']

- hosts: loadbalancers
  roles:
    - role: ansible-docker
      tags: ['docker-role']

    - role: ansible-haproxy
      tags: ['haproxy-role','update']

    - role: ansible-docker-compose
      tags: ['docker-compose-role','update']
```

### Usage

Run project like this:

``` bash
cd ansible-playbook

## All nodes 
ansible-playbook -i inventory.yml -l all site.yml --become 

## Webserver 
ansible-playbook -i inventory.yml -l webservers site.yml --become 

## Loadbalanceer
ansible-playbook -i inventory.yml -l loadbalancers site.yml --become 
```

## Ansible tags

The following tags are defined in playbooks:

|                       Tag name | Used for                                        
|--------------------------------|-------------------------------------------------
|                    docker-role | Installing/Configuring docker and docker-compose services 
|                         update | Run dokcer compose and Configuring files  (image tag sould be update befor run) 
|                   haproxy-role | Configuring haproxy.cfg
|                       php-role | Configuring php.ini
|            docker-compose-role | update/build docker-compose, for restart set 'docker_compose__restart', yes.

### Test

Before running the project in production, you can test it with vagrant :

``` bash
cd vagrant
vagrant up
ssh-copy-id vagrant@10.0.0,1.11,12  #password vagrant
cd ../ansible-playbook 
ansible-playbook -i ../vagrant/inventory.yml -l all --tags dokcer-role site.yml --become 
```

### monitoring
You can forward port to see the haproxy dashboard:

```bash
ssh -NL 127.0.0.1:1936:127.0.0.1:1936 (haproxy server ip)
```

![Haproxy dashboard Screen Shot](haproxy-dashboard.png?raw=true )<br/>

### Security / reliability tips

#### relibility

1- Set resorce limitashion for our contaners. We can also regulate the number of connections with the number of resources.
``` yaml
    deploy:
      resources:
        limits:
          cpus: '1'
          memory: 1000M
        reservations:
          cpus: '1'
          memory: 1000M
```
2- We can also regulate the number of connections with resources.

``` bash
/etc/php/7.4/fpm/pool.d/www.conf
pm.max_children
pm.max_requests

```
3- Set 'restart: always' for docker compose.
``` yaml
services:
  haproxy:
    image: haproxy:1.3
    build: 
      context: ./
    restart: always  # <---
```
#### Security

1- Remove public IPs from machines that do not need.

2- Enable firewall (aws, deploy network security across your Amazon VPCs)

