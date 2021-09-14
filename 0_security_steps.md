## 0st prepare the security environment

#### Install the latest version of the AWS CLI with sudo:

```
$curl "https://s3.amazonaws.com/aws-cli/awscli-bundle.zip" -o "awscli-bundle.zip"
$unzip awscli-bundle.zip
$sudo ./awscli-bundle/install -i /usr/local/aws -b /usr/local/bin/aws
```


#### Verify that the AWS CLI installed correctly:

```
$ aws --version
```



#### Configure AWS CLI:

```
$aws configure

AWS Access Key ID [None]: aaa
AWS Secret Access Key [None]: bbb
Default region name [None]: ccc
Default output format [None]: json
```



#### Verify that the configuration is correct:

```
$aws configure list
```



```
​```
  Name                    Value             Type    Location
  ----                    -----             ----    --------
​```

   profile                <not set>             None    None
access_key     ****************ABCD  shared-credentials-file    
secret_key     ****************ABCD  shared-credentials-file    
    region                ccc                  env    AWS_DEFAULT_REGION
```

#### Install Terraform:

##### Add the HashiCorp GPG key:

```
$curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
```



##### Add the official HashiCorp Linux repository

```
$sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
```



##### Update and install

```
$sudo apt-get update && sudo apt-get install terraform
```



##### Verify the installation worked by opening a new terminal session and listing Terraform's available subcommands

```
$terraform -help
```



##### Create a work directory for Terraform

```
$mkdir terraform
$cd terraform
```

##### Configure  Terraform ( AWS profile and provider)

```
$vim setup.tf

setup.tf
terraform {
 required_providers {
   aws = {
     source  = "hashicorp/aws"
     version = "~> 2.70"
   }
 }
}
```



#### Configure AWS account settings
```
provider "aws" {
 profile = "default"
 region  = var.region
}
```



#### Configure variable of region of AWS

```
$vim variables.tf
variable "region" {
 default = "ccc"
}
```



#### Inicialization Terraform

```
$terraform init
```

This command creates a .terraform directory

#### Configure instance EC2 of AWS in Terraform

```
$vim EC2_config.tf

#Policy for EC2 Role

resource "aws_iam_role_policy" "ec2_policy" {
 name = "ec2_policy"
 role = aws_iam_role.ec2_role.id

#EC2 Role

resource "aws_iam_role" "ec2_role" {
 name = "ec2_role"

 assume_role_policy = <<-EOF
{
   "Version": "2012-10-17",
   "Statement": [
     {
       "Action": "sts:AssumeRole",
       "Principal": {
         "Service": "ec2.amazonaws.com"
       },
       "Effect": "Allow",
       "Sid": ""
     }
   ]
}
EOF
}

resource "aws_iam_instance_profile" "ec2_profile" {
 name = "ec2_profile"
 role = aws_iam_role.ec2_role.name
}
```



#### Generate keys to connect by SSH to EC2 instance

```
$ssh-keygen -t rsa -b 4096
```



##### Copying the Public Key to the Server

```
$ssh-copy-id -i ~/.ssh/tatu-key-rsa user@host
```



##### Verify that the keys is save in the correct directory

```
$ls -latrh /etc/ssh/ssh_host_rsa_key
```



#### Import your own public key to Amazon EC2 in AWS CLI :


To import the public key

Use the import-key-pair AWS CLI command.

To verify that the key pair was imported successfully

Use the describe-key-pairs AWS CLI command.

Verify that the keys is save in the correct directory

```
$~/.ssh/authorized_keys
```



#### Add the keys into the variables file of Terraform (add to the end of the file):

```
$vim variables.tf

variable "ec2_key_name" {
 default = "<id_rsa>"
}
variable "ec2_public_key" {
 default = "<xxxxxxxxxxxxxxxxxxx>" #PUBLIC_KEY variable is the content of id_rsa.pub
}
variable "ec2_private_key_file_path" {
 default = "<~/.ssh/id_rsa>"
}
```

#### Add the keys into the config file of the instance EC2 in Terraform (add to the end of the file):

```
$vim EC2_config.tf
	
resource "aws_key_pair" "deployer_key" {
 key_name   = var.ec2_key_name
 public_key = var.ec2_public_key
}


```

##### Create a security group into the config file of the instance EC2 in Terraform (allowing tcp requests to port 22 from any IP (version 4)) (add to the end of the file):

```
$vim EC2_config.tf

resource "aws_security_group" "allow_ssh" {
 name        = "allow_ssh"
 description = "Allow SSH inbound traffic"

 ingress {
   description = "SSH from outside"
   from_port   = 22
   to_port     = 22
   protocol    = "tcp"
   cidr_blocks = ["0.0.0.0/0"]
 }

 egress {
   from_port   = 0
   to_port     = 0
   protocol    = "-1"
   cidr_blocks = ["0.0.0.0/0"]
 }

 tags = {
   Name = "allow_ssh"
 }
}
```
