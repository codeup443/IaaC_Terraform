
## 4th configure the instance so that it is capable of accepting at least 2,000 requests / second

##### Configure kernel linux in Debian ami into instance EC2, to accept minimum 2.000 petitions/ second, (to increase The Maximum Number Of Open Files / File Descriptors):

```
$vim 2_configure_kernel.sh

#!/bin/bash

vim -E -s /etc/sysctl.conf << EOF
fs.file-max = 65535
fs.nr_open = 65535
:update
:quit
EOF

vim -E -s /etc/security/limits.conf << EOF

- soft nproc 65535
- hard nproc 65535
- soft nofile 65535
- hard nofile 65535
  root soft nproc 65535
  root hard nproc 65535
  root soft nofile 65535
  root hard nofile 65535
  :update
  :quit
  EOF

vim -E -s /etc/systemd/user.conf << EOF
DefaultLimitNOFILE=65535
:update
:quit
EOF

sysctl -p

echo "OK 2_configure_kernel"
```

