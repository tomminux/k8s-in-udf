# Building a Kubernetes Cluster in UDF

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
echo "devsecops-server" > /etc/hostname
echo -e '#!/bin/bash\nifconfig eth1 10.1.20.4/24 up' > /etc/rc.local
chmod 755 /etc/rc.local
```

Update the Ansible package in order to use the "reboot" module (ansible > v2.7)

```
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 93C4A3FD7BB9C367
sudo apt-add-repository "deb http://ppa.launchpad.net/ansible/ansible/ubuntu bionic main"
sudo apt install ansible
```

If you want to run "kubectl" from DevSecOps Server, run this as root

```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add
sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
apt install kubectl
```

Reboot the DevSecOps Server

    reboot

## Create three linxu Ubuntu Server 18.04, t2.large
* k8s-master -  Bind a second interface to internal IP address 10.1.20.60
* k8s-node1 -  Bind a second interface to internal IP address 10.1.20.61
* k8s-node2 -  Bind a second interface to internal IP address 10.1.20.62

**Copy id_rsa.pub from user ansible server's ubuntu user to ".ssh/authorized_keys" on user root on each k8s node and master**

## Clone the ansible-udf-k8s repository

    git clone https://github.com/tomminux/k8s-in-udf.git
    
Modify 

    ansible/inventory/hosts

according to your UDF configuration. 

Modify

    ansible/playbooks/files/k8s-files/hosts

according to your UDF configuration. 

Change directory into

    cd ansible/playbooks/files/k8s-files
    
And download the updated flannel YAML file: kube-flannel.yml and modify it to run flannel on the eth1 interface in UDF:

    curl -s https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml | sed '/.*kube-subnet-mgr/a\ \ \ \ \ \ \ \ - --iface=eth1' > kube-flannel.yml



