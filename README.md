# Docker + AWS + Ansible

### Diversion task: Changing the Keys

First task of the day was to create another key for myself to utilise.
The point of doing this is, if a breach or compromise of one key occurs another one is spare to be utilised.

Navigate: Search for **IAM** -> **My security credentials** -> *"Create access key"*

![key](pictures/pass.png) 

This is where creation and deletion of the access codes can be made.

#### Ansible

After AWS keys have been configured, the proceeding task would be then to update the keys within ansibles *pass.yml*

My steps go as follow:
*Onto my vagrant controller*, navigate: */etc/ansible/public_var/all/*
within this directory we can see the *pass.yml* file.
We can now approach this file with the command: `sudo ansible-vault edit pass.yml` as accessing the file by normal means will output, garbled encryption

## EC2 instance with Ansible

To do this a playbook needs to be created first in ansible.

```ansible
---

- name: EC2 Making an instance
  hosts: local
  gather_facts: yes
  connection: local
  become: true

  tasks:
    - name: Inputting the setting for EC2
      tags: eng99_ivan_ansible
      ec2:
        aws_access_key: "{{aws_access_key}}"
        aws_secret_key: "{{aws_secret_key}}"
        key_name: eng99
        image: ami-07d8796a2b0f8d29c
        group_id: sg-09727510cb431c8cf
        instance_type: t2.micro
        vpc_subnet_id: subnet-05b84fc5570a5152b
        region: eu-west-1
        count: 1
        wait: yes
        assign_public_ip: yes

        instance_tags:
          Name: eng99_ivan_ansible

```
## Dockerfile

[Additional documentation for Dockerfile](https://docs.docker.com/engine/reference/builder/)

To **my** understanding a *Dockerfile* is another provisioning tool, this one just exclusively for docker and specifically the inside of a docker image.
```Dockerfile
FROM nginx:1.11-alpine
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]

```
## Running with docker 

```
# Provisioning docker
# installing docker as well as running an image
---
- hosts: aws
  gather_facts: true
  become: true

  tasks:
  - name: Retrive github
    git:
      repo: https://github.com/GNUKalashnikov/docker-ansible
      dest: /home/ubuntu/repo
      clone: yes
      update: yes
  - name: Install docker
    apt: pkg=docker state=present
  - name: Docker  
    shell: |
        cd repo/
        docker rmi my-nginx -f
        docker build -t my-nginx:latest .
        docker run -d -p 80:80 my-nginx

```

These two playbooks in conjunction should work to make a ec2 instance and then be able to run a docker file on top of it.

## Footnotes

**Note** within the code for *Making an instance* under the tasks, both "*aws_access_key"* and *"aws_secret_key"* are suggested to be variables, this is not the case within my production reality. They should hypothetically retrieve the keys from *ansible-vault*, but unfortunately in my reality they don't. For that segment to work please **hardcode** the keys in their respective places.
This is generally not recommended as this method does expose the keys for anyone on the same machine.

**Not a continuum***: In a future iteration I would proceed in adding a way to display the ip of the newly added EC2 instance and automatically add it to the hosts file. As of this iteration the user would need to find the instance within *aws* and manually have to add it to the hosts file. 
