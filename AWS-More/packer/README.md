
Instructions & Tasks
Log in to the AWS Console and navigate to Cloud9.

Sudo to root and Install packer into /usr/local/bin. Then drop out of root.

Use Cloud9 to create a packerfile.json with the following contents:

Create a variable called instance_size; the default values should be t2.small.
Create a variable called ami_name; the default value should be ami_base.
Create a variable called ssh_username; the default value should be ec2_user.
Create a variable called vpc_id, there should be no default value.
Create a variable called subnet_id, there should be no default value.
It should use the AWS builder.
The region should be set to us-east-1.
Source AMI should use the ami_base variable.
Instance Type should use the instance_size variable.
SSH username should use the ssh_username variable.
The ssh timeout should be set to 20m.
The AMI name should use the ami_name variable.
Set SSH pty to true.
Set the VPC ID to use the vpc_id variable.
Set the Subnet ID to use the subnet_id variable.
Create two tags:
Call the first one Name with a value of “Base AMI”.
Call the second BuiltBy and set it to “Packer”.
The description should be set to “AWS image”.
Create an inline shell provisioner that will execute a yum update and install git.
Validate the packerfile.json.

Execute the packer build and supply the VPC ID and Subnet ID as variables. Use AMI ami-1853ac65 for the base_ami.

After the AMI has completed building, create an EC2 instance using the AMI that was built and set the instance type to t2.small.
