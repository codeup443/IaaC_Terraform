
## 5th install inside the HAProxy instance: the latest stable version of the 2.2 branch with TProxy and SSL

```
$vim 3_install_configure_haproxy.sh

#!/bin/bash

sudo apt-get install build-essential

echo deb http://deb.debian.org/debian buster-backports main \
      > /etc/apt/sources.list.d/backports.list

sudo apt-get update

sudo apt-get install haproxy=2.2.\* -t buster-backports

sudo systemctl restart haproxy

mkdir /etc/haproxy
vim -E -s vim haproxy-2.2.6.conf << EOF

global
	log /dev/log	local0
	log /dev/log	local1 notice
	chroot /var/lib/haproxy
	stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
	stats timeout 30s
	user haproxy
	group haproxy
	daemon

# Default SSL material locations
	ca-base /etc/ssl/certs
	crt-base /etc/ssl/private
	
# See: https://ssl-config.mozilla.org/#server=haproxy&server-version=2.0.3&config=intermediate
	    ssl-default-bind-ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384
	    ssl-default-bind-ciphersuites TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256
	    ssl-default-bind-options ssl-min-ver TLSv1.2 no-tls-tickets


defaults
	log	global
	mode	http
	option	httplog
	option	dontlognull
        timeout connect 5000
        timeout client  50000
        timeout server  50000
	errorfile 400 /etc/haproxy/errors/400.http
	errorfile 403 /etc/haproxy/errors/403.http
	errorfile 408 /etc/haproxy/errors/408.http
	errorfile 500 /etc/haproxy/errors/500.http
	errorfile 502 /etc/haproxy/errors/502.http
	errorfile 503 /etc/haproxy/errors/503.http
	errorfile 504 /etc/haproxy/errors/504.http

EOF

echo "OK 3_install_configure_haproxy"
```

#### Create the script to run the 3 before scripts in order into instance EC2:

```
$cd init_scripts
$vim configure.sh

#!/bin/bash

echo "Configuring system"

sudo sh /tmp/init_scripts/1_update_and_install.sh 
sudo sh /tmp/init_scripts/2_configure_kernel.sh
sudo sh /tmp/init_scripts/3_install_configure_haproxy.sh
```



#### Add the provisioners in configuration file of Terraform  (add to the end of the file):

```
$vim EC2_config.tf

provisioner "remote-exec" {
   inline = [
     "sudo mkdir /tmp/init_scripts",
     "sudo chmod -R 777 /tmp/init_scripts"
   ]
   connection {
     host        = self.public_ip
     type        = "ssh"
     agent       = false
     private_key = file(var.ec2_private_key_file_path)
     user        = "ec2-user"
   }
 }

 provisioner "file" {
   source      = "init_scripts/"
   destination = "/tmp/init_scripts"
   connection {
     host        = self.public_ip
     type        = "ssh"
     agent       = false
     private_key = file(var.ec2_private_key_file_path)
     user        = "ec2-user"
   }
 }

 provisioner "remote-exec" {
   inline = [
     "sudo sh /tmp/init_scripts/configure.sh",
   ]
   connection {
     host        = self.public_ip
     type        = "ssh"
     agent       = false
     private_key = file(var.ec2_private_key_file_path)
     user        = "ec2-user"
   }
 }
}


```

#### Generate resources of Terraform:

```
$terraform apply
```
