#Managing Docker Containers with Ansible


ocker and Ansible Overview

As leading DevOps technologies, Docker and Ansible complement each other very well. Docker is used as a way to encapsulate applications in predictable environments within a lightweight container, while Ansible can be used to configure the host server to support and orchestrate Docker deployments. In this guide, we will walk through steps to configure and manage Docker containers with Ansible.

Technology Note

The guide is drafted based on the current course material at Linux Academy, which allows you to get up to speed fairly quickly. However, as Docker tends to evolve very rapidly due to heavy development, there is a chance that whatever Docker implementation you are using could differ from the course. If that’s the case, we encourage you to post comments in our community forum if you have questions about the guide.

The following skills would be useful in following this guide, but aren’t strictly necessary:

Managing Linux or Unix-like systems (we will be mostly working within the terminal).
Having a working knowledge of Docker.
A basic understanding of Ansible.
Feel free to take advantage of the following resources available from Linux Academy to get up to speed:


In addition, you should have the following:

A workstation environment with Docker and Ansible 2.2 and above installed.
An OS server that supports Docker. Note that for this guide, we are using Ubuntu 16.04 and a login account "ubuntu" with password-less sudo privileges:
A Docker Hub account. Instructions can be found ahere.
Assuming that you have all of that, you may proceed.



Docker Development

As a first step, at the base of your home directory, create the directory src/docker/demo and then change over to that path.



stardust:~ rilindo$ mkdir -p src/docker/demo
stardust:~ rilindo$ cd src/docker/demo


Next, we will download the latest version of CentOS from Docker by running docker pull centos:latest:



stardust:demo rilindo$ docker pull centos:latest
The output should look similar to:



stardust:demo rilindo$ docker pull centos:latest
latest: Pulling from library/centos
8d30e94188e7: Pull complete 
Digest: sha256:2ae0d2c881c7123870114fb9cc7afabd1e31f9888dac8286884f6cf59373ed9b
Status: Downloaded newer image for centos:latest

Now we will test the image by instantiating it with the command docker run -d --name test_instance centos:latest:


stardust:demo rilindo$ docker run -d --name test_instance centos:latest


It will return a long uuid string:


stardust:demo docker run -d --name test_instance centos:latest
311f2825842f324e076d952384a3a6795250976ef6f3572feaf9c3c01ce5f978

We should be able to verify that it is running using the docker ps -a command:


stardust:demo rilindo$ docker ps -a

The output should be similar to the following:


stardust:demo rilindo$ docker ps -a
CONTAINER ID        IMAGE                    COMMAND                  CREATED             STATUS                     PORTS                  NAMES
311f2825842f        centos:latest            "/bin/bash"              6 seconds ago       Exited (0) 4 seconds ago                          test_instance

With basic functionality verified, we can go ahead and create a custom Docker image.



Customizing a Docker Image

In the directory src/docker/demo, create the following Dockerfile:


FROM centos:latest
Maintainer rilindo.foster@monzell.com

RUN yum update -y
RUN yum install -y httpd net tools

RUN echo "This is a custom index file built during the image creation" > /var/www/html/index.html

ENTRYPOINT apachectl "-DFOREGROUND"


When we build the image, the following actions will take place:

Update CentOS to the latest version
Install the httpd (apache) application and supporting tools
Create a custom index page
Setup the container to start Apache in the foreground
Save the file and build the image with this command:


docker build -t centos/apache:v1


The output should be similar to the following:


stardust:demo rilindo$ docker build -t centos/apache:v1 .
Sending build context to Docker daemon 2.048 kB
Step 1 : FROM centos:latest
 ---> 980e0e4c79ec
Step 2 : MAINTAINER rilindo.foster@monzell.com
 ---> Using cache
 ---> 418ea7dd7050
Step 3 : RUN yum update -y
 ---> Using cache
 ---> e2455d5eefd2
Step 4 : RUN yum install -y httpd net tools
 ---> Using cache
 ---> c81b8cd54ce8
Step 5 : RUN echo "This is a custom index file build during the image creation" > /var/www/html/index.html
 ---> Using cache
 ---> 362d569c3803
Step 6 : ENTRYPOINT apachectl "-DFOREGROUND"
 ---> Using cache
 ---> abcf05ec382a
Successfully built abcf05ec382a


Once the image is finished building, we are going to test it by spinning up a container based off of the image with:



docker run -d --name apacheweb1 -p 8080:80  centos/apache:v1

When you run the above command, you should be able to see it in the Docker process list by running docker ps -a. The output should look similar to this:


CONTAINER ID        IMAGE                    COMMAND                  CREATED             STATUS              PORTS                  NAMES
b049ba2e829f        rilindo/myapacheweb:v1   "/bin/sh -c 'apachect"   4 days ago          Up 4 seconds        0.0.0.0:8080->80/tcp   apacheweb1
stardust:~ rilindo$

You should be able to view the custom index page by browsing to it or downloading it with a web utility. In our case, we are able to view the index page by simply using curl:


stardust:~ rilindo$ curl localhost:8080
This is a custom index file build during the image creation


With your setup, you may need to pull the page against the Docker ip, which you can retrieve by parsing for the IPAddress against the output of docker inspect:


stardust:~ rilindo$ docker inspect apacheweb1 | grep IPAddress
            "SecondaryIPAddresses": null,
            "IPAddress": "172.17.0.2",

Uploading The Custom Container

Now that you created a custom image, it is time to upload it to a private repository. If you haven’t done so, sign up for a Docker Hub account. When you are done, click on the button called “Create Registry”:


user_12286_580eac36d5515.png


Then specify the name of the registry, a brief description of the repository, and mark the repository as “private” (we will get to the reasons why in just a moment):



user_12286_580eacee84c88.png


Once you are done, go back to your terminal and run docker login to authenticate against your account:



stardust:~ rilindo$ docker login
Enter your Docker Hub username and password and it should return with Login Succeeded message: 


Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: rilindo
Password:
Login Succeeded
stardust:~ rilindo$


Now list the images you have locally with the command Docker images (it should return most of the images you have locally):



REPOSITORY TAG IMAGE ID CREATED SIZE
<none> <none> 28ade7a75856 23 hours ago 485.4 MB
centos/apache v1 abcf05ec382a 23 hours ago 485.4 MB
nginx latest ba6bed934df2 3 days ago 181.4 MB
centos latest 980e0e4c79ec 2 weeks ago 196.8 MB


We will then tag the image we created using its Image ID that was just returned to us from the previous command. The tag should match the name of the repository, like so:



stardust:~ rilindo$ docker tag abcf05ec382a rilindo/myapacheweb:v1
When that is done, upload it with:
docker push rilindo/myapacheweb:v1


It will take some time to upload, depending on your bandwidth. When it finishes, it should return with a hash at the end (note the bolded line):



stardust:~ rilindo$ docker push rilindo/myapacheweb:v1
The push refers to a repository [docker.io/rilindo/myapacheweb]
abfaedfcb05d: Mounted from rilindo/apache
c8bbf58a3299: Mounted from rilindo/apache
49715a5feaeb: Mounted from rilindo/apache
0aeb287b1ba9: Mounted from rilindo/apache
v1: digest: sha256:22136f81d1938870f19ec3a4773595de0660c39c1645e1a376855ec8d9897a6f size: 1160


Congratulations! You have successfully uploaded a Docker image to your account. 


Configuring Ansible with Docker

In addition to ensuring that Ansible is installed on your workstation, you also need to install the following module in order for Ansible to talk to the docker host:



stardust:ansible_docker_demo rilindo$ sudo pip install 'docker-py>=1.7.0'
Downloading/unpacking docker-py>=1.7.0
Successfully installed docker-py requests websocket-client docker-pycreds
Cleaning up...


Then for Docker service, we will need to install (for brevity’s sake, the output is cut short):



stardust:ansible_docker_demo rilindo$ pip install 'docker-compose>=1.7.0'
Downloading/unpacking docker-compose>=1.7.0
Downloading docker-compose-1.9.0rc1.tar.gz (155kB): 155kB downloaded
Running setup.py egg_info for package docker-compose
----------------------------------------
Command /usr/bin/python -c "import setuptools;__file__='/private/var/folders/9x/tsk9k8s56sg570g66qxr240w0000gn/T/pip-build-rilindo/docker-compose/setup.py';exec(compile(open(__file__).read().replace('\r\n', '\n'), __file__, 'exec'))" install --record /var/folders/9x/tsk9k8s56sg570g66qxr240w0000gn/T/pip-XnOJ2u-record/install-record.txt --single-version-externally-managed failed with error code 1 in /private/var/folders/9x/tsk9k8s56sg570g66qxr240w0000gn/T/pip-build-rilindo/docker-compose
Storing complete log in /var/folders/9x/tsk9k8s56sg570g66qxr240w0000gn/T/tmpgVicV2


With that done, we’ll go ahead and create a working directory for us to write our ansible code:



stardust:~ rilindo$ mkdir src/ansible_docker_demo
stardust:~ rilindo$ cd src/ansible_docker_demo/


Next, we will create a role directory:



stardust:ansible_docker_demo rilindo$ mkdir roles


Then we will create a configuration file that tells ansible to look into that role directory path:



[defaults]
roles_path=~/src/ansible_docker_demo/roles
And then now we will point ansible to that configuration file:



stardust:ansible_docker_demo rilindo$ export ANSIBLE_CONFIG=$(pwd)/ansible.cfg


Verify that the variable is set:



stardust:ansible_docker_demo rilindo$ echo $ANSIBLE_CONFIG
/Users/rilindo/src/ansible_docker_demo/ansible.cfg


Now we will create an inventory. Create a file called hosts and put in the IP of the docker host (in this case, 192.168.64.7), then set the ANSIBLE_HOST variable:



stardust:ansible_docker_demo rilindo$ export ANSIBLE_HOSTS=$(pwd)/hosts


Verify that it is set:



stardust:ansible_docker_demo rilindo$ echo $ANSIBLE_HOSTS
/Users/rilindo/src/ansible_docker_demo/hosts

Installing Docker on the Host Server

By itself, Ansible will not install docker on the host server, so we’ll need to write up some code to go it for us. Fortunately, we can use one of the roles available online. In this case, we’ll use this role here:

https://github.com/angstwad/docker.ubuntu

Create a file call requirements.yml with the following content:



- src: https://github.com/angstwad/docker.ubuntu
  name: angstwad.docker.ubuntu
  version: master
Run the ansible-galaxy command to install the role:



stardust:ansible_docker_demo rilindo$ ansible-galaxy install -r requirements.yml
- extracting angstwad.docker.ubuntu to /Users/rilindo/src/ansible_docker_demo/roles/angstwad.docker.ubuntu
- angstwad.docker.ubuntu was installed successfully
stardust:ansible_docker_demo rilindo$


Now we will create playbook  install_docker.yml to install the docker on the host. Create a playbook to install docker with:



---
- hosts: all
  vars:
  docker_opts: >
    -H unix://
    -H tcp://0.0.0.0:2375
    --log-level=debug
  remote_user: ubuntu
  become: yes
  become_method: sudo
  roles:
  - angstwad.docker.ubuntu


What is will do is to install Docker and then and expose the docker port API. This will allow ansible to directly communicate with the Docker API to create and destroy docker instances.

Verify that the syntax is correct with ansible-playbook –syntax-check:



stardust:ansible_docker_demo rilindo$ ansible-playbook --syntax-check install_docker.yml
statically included: /Users/rilindo/src/ansible_docker_demo/roles/angstwad.docker.ubuntu/tasks/kernel_check_and_update.yml
playbook: install_docker.yml


Since we did not see any error, we will run the play. The following output should appear (once again, the output is cut due to verbosity):



stardust:ansible_docker_demo rilindo$ ansible-playbook install_docker.yml
statically included: /Users/rilindo/src/ansible_docker_demo/roles/angstwad.docker.ubuntu/tasks/kernel_check_and_update.yml
PLAY [all] *********************************************************************
TASK [setup] *******************************************************************
ok: [192.168.64.8]
TASK [angstwad.docker.ubuntu : Start docker] ***********************************
ok: [192.168.64.8]
TASK [angstwad.docker.ubuntu : Start docker.io] ********************************
skipping: [192.168.64.8]
TASK [angstwad.docker.ubuntu : Add users to the docker group] ******************
TASK [angstwad.docker.ubuntu : update facts if docker0 is not defined] *********
ok: [192.168.64.8]
PLAY RECAP *********************************************************************
192.168.64.8 : ok=17 changed=10 unreachable=0 failed=0
Verify that you are able to access the DOCKER API port. In this case, we’ll use netcat to test:



stardust:ansible_docker_demo rilindo$ nc -zv 192.168.64.8 2375
found 0 associations
found 1 connections:
1:flags=82<CONNECTED,PREFERRED>
outif bridge100
src 192.168.64.1 port 55979
dst 192.168.64.8 port 2375
rank info not available
TCP aux info available
Connection to 192.168.64.8 port 2375 [tcp/*] succeeded!


Now we are ready to deploy docker containers.


Managing Docker Images

Let us now create a playbook called download_docker_image.yml with the following content:



---
- hosts: all
  remote_user: ubuntu
  become: yes
  become_method: sudo
  tasks:
  - name: pull image
    docker_image:
    name: ubuntu
    state: present


This playbook will ensure that Ubuntu image should exist (hence, the state present). From there, Ansible will talk to the docker API and initiate the download of the container.

Verify that the Ansible syntax. If there are no errors, it should return playbook: filename, like so:



stardust:ansible_docker_demo rilindo$ ansible-playbook --syntax-check download_docker_image.yml
playbook: download_docker_image.yml


And then run it:



stardust:ansible_docker_demo rilindo$ ansible-playbook download_docker_image.yml
PLAY [all] *********************************************************************
TASK [setup] *******************************************************************
ok: [192.168.64.8]
TASK [pull image] **************************************************************
changed: [192.168.64.8]
PLAY RECAP *********************************************************************
192.168.64.8 : ok=2 changed=1 unreachable=0 failed=0


You should be able to verify that the image has been downloaded from the docker server:



ubuntu@ubuntu:~$ sudo docker images
REPOSITORY TAG IMAGE ID CREATED SIZE
ubuntu latest f753707788c5 2 weeks ago 127.2 MB
ubuntu@ubuntu:~$
Let us move on to creating and destroying containers.


Running and Removing Containers

Create the file create_docker_container.yml with the following code:


---
- hosts: all
  remote_user: ubuntu
  become: yes
  become_method: sudo
  tasks:
  - name: create docker container
    docker_container:
      name: mycontainer
      image: ubuntu
      state: started

This will create a new container called mycontainer using the Ubuntu image.

Check for syntax errors:


stardust:ansible_docker_demo rilindo$ ansible-playbook --syntax-check create_docker_container.yml 

playbook: create_docker_container.yml
And then run it:


stardust:ansible_docker_demo rilindo$ ansible-playbook create_docker_container.yml 

PLAY [all] *********************************************************************

TASK [setup] *******************************************************************
ok: [192.168.64.8]

TASK [create docker container] *************************************************
changed: [192.168.64.8]

PLAY RECAP *********************************************************************
192.168.64.8               : ok=2    changed=1    unreachable=0    failed=0

The fact that it says changed in the task tell us that it is able to create the container. Sure enough, when we logged in, we could see the container in docker ps –a output:


CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
ce6a6e6662a4        ubuntu              "/bin/bash"         4 seconds ago       Exited (0) 3 seconds ago                       mycontainer

Now we will remove it. Create the file call destroy_docker_container.yml with the following:


---
- hosts: all
  remote_user: ubuntu
  become: yes
  become_method: sudo
  tasks:
  - name: create docker container
    docker_container:
      name: mycontainer
      image: ubuntu
      state: absent

The only difference with this and create_docker_container.yml is that we are now setting the state to absent, meaning that the container should no longer exist.

Once again, validate the syntax:


stardust:ansible_docker_demo rilindo$ ansible-playbook --syntax-check destroy_docker_container.yml 

playbook: destroy_docker_container.yml
stardust:ansible_docker_demo rilindo$
And then run it:


stardust:ansible_docker_demo rilindo$ ansible-playbook destroy_docker_container.yml 

PLAY [all] *********************************************************************

TASK [setup] *******************************************************************
ok: [192.168.64.8]

TASK [create docker container] *************************************************
changed: [192.168.64.8]

PLAY RECAP *********************************************************************
192.168.64.8               : ok=2    changed=1    unreachable=0    failed=0

Where it says changed, it means that it able to remove the container. Sure enough, when we run docker ps –a, the container no longer exists:


root@ubuntu:~# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
root@ubuntu:~#

At this point, let us now deploy our custom container.


Deploying and Destroying Custom Docker Containers


Since we need to login to Docker registry, we need to be able to put our secrets in the code. To avoid exposing credentials in the clear, we will use ansible-vault to encrypt them. Create a file called secrets.yml with your login info:


---
docker_hub_username: yourusername
docker_hub_password: yourpassword
docker_hub_email: youremail@example.com

Run ansible-vault against the file. It will prompt for a password to encrypt it:


stardust:ansible_docker_demo rilindo$ ansible-vault encrypt secret.yml 
New Vault password: 
Confirm New Vault password: 
Encryption successful
The file will look something like this:


$ANSIBLE_VAULT;1.1;AES256
36626666333261633834313438336264383534353036633135333837633035303432366136353632
3839336362303934393462643665366438653432633232300a336264353063316136663433343435
39393231333864663230373836303865616163353034323262333035363336333537373265333932
3863343532636161660a653435623135336436656533346632626132656130633136356565303139
3338

Now create the file call create_custom_docker_container.yml with the following code:


---
- hosts: all
  remote_user: ubuntu
  vars_files:
    - secret.yml
  become: yes
  become_method: sudo
  tasks:
  - name: login to docker registry
    docker_login:
      username: "{{ docker_hub_username }}"
      password: "{{ docker_hub_password }}"
      email: "{{ docker_hub_email }}"
  - name: create custom docker container
    docker_container:
      name: mycustomcontainer
      image: rilindo/myapacheweb:v1
      state: started
      exposed_ports:
        - 80
      ports:
        "80:80"

This will pull the credentials from your encrypted secrets file, login into the docker container and then download our container image, create it and expose the port for access.

Now validate the syntax. Because the credentials are now encrypted, you need to pass the --ask-vault-pass parameter to the syntax command. If you do not, you will get the following error:


stardust:ansible_docker_demo rilindo$ ansible-playbook --syntax-check create_custom_docker_container.yml 
ERROR! Decryption failed on /Users/rilindo/src/ansible_docker_demo/secret.yml
Otherwise, assuming you did pass the parameter and you entered the correct vault password, the output should return as follows:


stardust:ansible_docker_demo rilindo$ ansible-playbook --syntax-check create_custom_docker_container.yml --ask-vault-pass
Vault password: 

playbook: create_custom_docker_container.yml

With no errors, run the play (make sure that you pass the –ask-vault-pass to decrypt credentials. It should return with:


stardust:ansible_docker_demo rilindo$ ansible-playbook create_custom_docker_container.yml --ask-vault-pass
Vault password: 

PLAY [all] *********************************************************************

TASK [setup] *******************************************************************
ok: [192.168.64.8]

TASK [login to docker registry] ************************************************
changed: [192.168.64.8]

TASK [create custom docker container] ******************************************
changed: [192.168.64.8]

PLAY RECAP *********************************************************************
192.168.64.8               : ok=3    changed=2    unreachable=0    failed=0   

stardust:ansible_docker_demo rilindo$

You should be able to confirm that the customer container is running on the docker host:


CONTAINER ID        IMAGE                    COMMAND                  CREATED             STATUS              PORTS                NAMES
9adffc6184f1        rilindo/myapacheweb:v1   "/bin/sh -c 'apachect"   About an hour ago   Up About an hour    0.0.0.0:80->80/tcp   mycustomcontainer


And verify that you can access the web port:


ubuntu@ubuntu:~$ curl http://localhost
This is a custom index file build during the image creation
ubuntu@ubuntu:~$

To tear down the docker container, you can do the reverse of what you have done by copying the file create_custom_docker_container.yml to destroy_custom_docker_container.yml and change the state line to absent in the task create custom docker container:


---
- hosts: all
  remote_user: ubuntu
  vars_files:
    - secret.yml
  become: yes
  become_method: sudo
  tasks:
  - name: login to docker registry
    docker_login:
      username: "{{ docker_hub_username }}"
      password: "{{ docker_hub_password }}"
      email: "{{ docker_hub_email }}"
  - name: create custom docker container
    docker_container:
      name: mycustomcontainer
      image: rilindo/myapacheweb:v1
      state: absent
      exposed_ports:
        - 80
      ports:
        "80:80"
Run it and it will remove the docker container:


stardust:ansible_docker_demo rilindo$ ansible-playbook destroy_custom_docker_container.yml --ask-vault-pass
Vault password: 

PLAY [all] *********************************************************************

TASK [setup] *******************************************************************
ok: [192.168.64.8]

TASK [login to docker registry] ************************************************
ok: [192.168.64.8]

TASK [create custom docker container] ******************************************
changed: [192.168.64.8]

PLAY RECAP *********************************************************************
192.168.64.8               : ok=3    changed=1    unreachable=0    failed=0
This is how you can manage Docker Containers with Ansible.
