
## 3rd update the instance

##### Install the libraries (unattended):

```
$vim 1_update_and_install.sh

#!/bin/bash

echo "1. Updating and installing libraries..."
sudo apt update -y && 
sudo apt install -y
vim
wget
git
make
gcc
libsystemd-dev
libssl-dev
lua5.3
liblua5.3-dev
libpcre3-dev
zlib1g-dev

echo "OK 1_update_and_install"
```

`$./1_update_and_install.sh`
