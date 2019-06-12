# Building a Kubernetes Cluster in UDF

## UDF - Create two more networks
* internal on 10.1.10.0/24
* external on 10.1.20.0/24

## UDF - Create a linux Ubuntu Server 18.04, t2.large

* Bind a second interface to internal IP address 10.1.20.4

```bash
ssh-keygen -b 2048
sudo su -
apt update
apt upgrade -y
apt install ansible -y
echo "devsecops-server" > /etc/hostname
echo -e '#!/bin/bash\nifconfig eth1 10.1.20.4/24 up' > /etc/rc.local
chmod 755 /etc/rc.local
```

If you want to run "kubectl" from DevSecOps Server, run this as root

```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add
sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
apt install kubectl
```

## Create three linxu Ubuntu Server 18.04, t2.large
* k8s-master -  Bind a second interface to internal IP address 10.1.20.60
* k8s-node1 -  Bind a second interface to internal IP address 10.1.20.61
* k8s-node2 -  Bind a second interface to internal IP address 10.1.20.62

*Copy id_rsa.pub from user ansible server's ubuntu user to ".ssh/authorized_keys" on user root on each k8s node and master*

## Clone the ansible-udf-k8s repository
