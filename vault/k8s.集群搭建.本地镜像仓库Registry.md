---
id: hrjqjecd57jzuvsm33a8lx9
title: 本地镜像仓库Registry
desc: ''
updated: 1723900742841
created: 1723900659025
---

## 本地镜像 Registry

参考: https://distribution.github.io/distribution/about/deploying/

```
nerdctl run -d --name registry -p 5000:5000 \
  -v /etc/docker/certs.d/master.node:/certs \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \
  --restart=always
  registry:2
```

### registry https 证书生成

参考: [Test an insecure registry](https://distribution.github.io/distribution/about/insecure/)

```
# 证书 SAN 配置
# /etc/docker/certs.d/master.node:5000/san.cnf
[ req ]
default_bits       = 2048
prompt             = no
default_md         = sha256
req_extensions     = req_ext
distinguished_name = dn

[ dn ]
C  = US
ST = CA
L  = San Francisco
O  = MyCompany
OU = MyOrg
CN = master.node

[ req_ext ]
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = master.node
IP.1 = 192.168.56.200  # Replace with your actual IP address
```

```
# 根据 san.cnf 生成 domain.csr
sudo mkdir -p /etc/docker/certs.d/master.node:5000
cd /etc/docker/certs.d/master.node:5000
sudo openssl req -newkey rsa:2048 -nodes -keyout domain.key -out domain.csr -config san.cnf
# 根据 domain.csr 生成 domain.crt
sudo openssl x509 -req -in domain.csr -signkey domain.key -out domain.crt -days 365 -extfile san.cnf -extensions req_ext
```

```
# 拷贝 ca.crt 给其他机器
cd ~ && scp master.node:/home/vboxuser/ca.crt .
sudo mkdir -p /etc/containerd/certs.d/master.node:5000
sudo  mv /home/vboxuser/ca.crt /etc/containerd/certs.d/master.node:5000
sudo chown root:root /etc/containerd/certs.d/master.node:5000/ca.crt
sudo ls -l /etc/containerd/certs.d/master.node:5000/ca.crt
```

### 系统信任证书

```
# ubuntu
$ cp certs/domain.crt /usr/local/share/ca-certificates/myregistrydomain.com.crt
update-ca-certificatesk
```

```
sudo cp /etc/containerd/certs.d/master.node\:5000/ca.crt /usr/local/share/ca-certificates/
sudo update-ca-certificates
sudo systemctl restart containerd
sudo systemctl restart kubelet
```