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



# Hardening a kops Default Deployment with Kube-bench
## Use kops to Create the Cluster

A script has been provided to run kops and create the cluster.

To run the script, type:

$ . ./k8s-create.sh
To actually create the cluster, type:

$ kops update cluster --yes
To validate the cluster, type:

$ kops validate cluster

## Retrieve a connect string from Amazon AWS and Connect to the Master Node
Use the Amazon AWS Console and retrieve a connect string for the master node. Then use ssh to connect to the master node.

$ ssh -i [YOUR .pem HERE] admin@[YOUR dns name HERE]

## Run the AquaSec kube-bench utility on the Master Node
From your terminal session on the master node...

To install the kube-bench utility on the master node, type:

$ sudo docker run --rm -v `pwd`:/host aquasec/kube-bench:latest install
Then to run the utility and output the report to a file, type:

$ ./kube-bench master > orig.report

## Remediate Failed Tests 1.1.2, 1.1.5 and 1.1.6
Use the process status command to look at the current apiserver arguments.

$ ps -aef | grep kube-apiserver
Use the vi editory as super user to edit the kube apiserver manifest.

$ sudo vi /etc/kubernetes/manifests/mainfest-kube-apiserver.conf
Edit the test recommendations per the remediation listed in the kube-bench report.

Rerun the kube=bench report after the api server has restarted.

$ ./kube-bench master > new.report
Compare the old and new reports using the Linux diff command.

$ diff orig.report new.report





