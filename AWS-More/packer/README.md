
# Instructions & Tasks
  1.  Log in to the AWS Console and navigate to Cloud9.

  2.  Sudo to root and Install packer into /usr/local/bin. Then drop out of root.

  3.  Use Cloud9 to create a packerfile.json with the following contents:

  4.  Create a variable called instance_size; the default values should be t2.small.
  5.  Create a variable called ami_name; the default value should be ami_base.
  6.  Create a variable called ssh_username; the default value should be ec2_user.
  7.  Create a variable called vpc_id, there should be no default value.
  8.  Create a variable called subnet_id, there should be no default value.
  9.  It should use the AWS builder.
  10. The region should be set to us-east-1.
  11. Source AMI should use the ami_base variable.
  12. Instance Type should use the instance_size variable.
  13. SSH username should use the ssh_username variable.
  14. The ssh timeout should be set to 20m.
  15. The AMI name should use the ami_name variable.
  16. Set SSH pty to true.
  17. Set the VPC ID to use the vpc_id variable.
  18. Set the Subnet ID to use the subnet_id variable.
      Create two tags:
  19.Call the first one Name with a value of “Base AMI”.
  20. Call the second BuiltBy and set it to “Packer”.
      The description should be set to “AWS image”.
  21. Create an inline shell provisioner that will execute a yum update and install git.
  22. Validate the packerfile.json.

        Execute the packer build and supply the VPC ID and Subnet ID as variables. Use AMI ami-1853ac65 for the base_ami.

        After the AMI has completed building, create an EC2 instance using the AMI that was built and set the instance type to t2.small.
