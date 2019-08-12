# Using Packer and Ansible for Hardening

## KOPS 

# Step 1

# Grab the aws keys and Install the Ansible Role

Use the directory list command to verify the files in your home directory.

$ ls -l
Verify that ansible is installed by typing:

$ ansible --version
Display the AWS Credentials setup for your lab and make note of them.

IMPORTANT: DO NOT STORE THESE KEYS ANYWHERE BUT ON YOUR PRIVATE CLIENT SYSTEM.

$ cat .aws/credentials
Generate an RSA key for use by the ansible playbook that will be used to harden the OS system.

$ ssh-keygen -b 4096
Note: Use defaults (blank return responses) to the prompts.

Create a default vpc for your lab instance:

$ aws ec2 create-default-vpc
Use the anisible-galaxy to install the sample plybook we will use in this lab:

$ ansible-galaxy install githubixx.harden-linux


# Step 2 
# Edit the keys into the Packer run script, and Run the Packer Job

Use the editor of your choice to edit the ksac-packer-build.sh file and place the actual aws key id and secret key that you recorded into the place where the environment variables appear now.

$ vi ksac-packer-build.sh
Use the cat command to examine the contents of the ksac-packer.json file.

$ cat ksac-packer.json
Use the cat command to examine the contents of the playbook.yml file.

$ cat playbook.yml
When you are ready to execute the packer build and provisioning process, enter:

$ ksac-packer-build.sh
When the output is complete, use the amazon console to view the ami images created.

