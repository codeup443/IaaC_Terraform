
## 1st selection operating system: Debian

##### Create a instance EC2 (the ami ami-0e0ed860953d90a95 content the image: debian-10-amd64-20200610-293 ) in config file of Terraform (add to the end of the file):

```
$vim EC2_config.tf

resource "aws_instance" "ec2_haproxy_server" {
 ami                         = "ami-0bb3fad3c0286ebd5"
 instance_type               = "t2.micro"
 associate_public_ip_address = true
 iam_instance_profile        = aws_iam_instance_profile.ec2_profile.name
 key_name = var.ec2_key_name
 security_groups = ["allow_ssh"]
}
```
