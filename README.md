# Building a Kubernetes Cluster in UDF

## UDF - Create two more networks
* external on 10.1.10.0/24
* internal on 10.1.20.0/24

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

_Copy id_rsa.pub from user ansible server's ubuntu user to ".ssh/authorized_keys" **on user root** on each k8s node and master_

## Clone the ansible-udf-k8s repository

    git clone https://github.com/tomminux/k8s-in-udf.git
    
Change directory

    cd k8s-in-udf/ansible
    
Modify 

    inventory/hosts

according to your UDF configuration: please be carefull on second interafce name, it could be eth1 or ens6, check on your boxes. 

Modify

    playbooks/files/k8s-files/hosts

according to your UDF configuration. 
    
And download the updated flannel YAML file: kube-flannel.yml and modify it to run flannel on the eth1 interface in UDF (_make sure to execute this in k8s-in-udf/ansible directory_):

If you are running your Deployment in AWS please use:

    curl -s https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml | sed '/.*kube-subnet-mgr/a\ \ \ \ \ \ \ \ - --iface=eth1' > playbooks/files/k8s-files/kube-flannel.yml

If you are running your Deployment in UDF Hypervisor please use:

    curl -s https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml | sed '/.*kube-subnet-mgr/a\ \ \ \ \ \ \ \ - --iface=ens6' > playbooks/files/k8s-files/kube-flannel.yml

## Run the playbook

In order to finally build the cluster:

    ansible-playbook playbooks/install-k8s-cluster.yaml
    
At the end of the run, if eveything was good, you should be able to "see" the cluster running, directly from your Linux DevSecOps Server:

    kubectl get pods -A -o wide
    
And you can verify that all system pods, including Flannel pods, are running on the correct internal interface. 

**ENJOY YOUR BRAND NEW CLUSTER**

