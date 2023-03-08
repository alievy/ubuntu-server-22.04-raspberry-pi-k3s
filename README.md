# How to install ubuntu 22.04 on raspberry pi and k3s.

How to install ubuntu server 22.04 on raspberry pi and k3s agent.

## Update the system

With the raspberry pi installed with ubuntu 22.04, run update and upgrade the system

```bash
sudo apt update && 
apt install linux-modules-extra-raspi -y &&
apt upgrade -y &&
apt dist-upgrade -y
```

Reboot system when finished

```
sudo systemctl reboot
```

## Fix cgroup for raspberry pi

Add this to the beginning of /boot/firmware/cmdline.txt:

```
cgroup_enable=cpuset cgroup_memory=1 cgroup_enable=memory
```

```
sudo vi /boot/firmware/cmdline.txt
```

Reboot after change

## Set hostname and hosts

Give your raspberry pi a hostname and also update hosts file other Pi's can find each other

Update hostname

```
sudo hostnamectl set-hostname worker1
```

Edit /etc/hosts and add following:

```
$ sudo vi /etc/hosts


127.0.0.1       worker1
192.168.10.21   master
192.168.10.22   worker1
```

## Set static ip

Update /etc/cloud/cloud.cfg
Change the preserve_hostname value to true.

Edit sudo vi /etc/netplan/50-cloud-init.yaml

```
network:
    ethernets:
        eno1:
            addresses:
                - 192.168.10.21/24
            nameservers:
                addresses: [8.8.4.4, 8.8.8.8]            
            routes:
                - to: default
                via: 192.168.10.1
                dhcp4: false
    version: 2
```

When done run sudo netplan apply and reboot device

## Install docker

```
sudo snap install docker
sudo addgroup --system docker
sudo adduser $USER docker
newgrp docker

sudo snap disable docker
sudo snap enable docker
```


## Allow ports on firewall

We need to allow ports that will will be used to communicate between the master and the worker nodes. The ports are 443 and 6443.

```
sudo ufw allow 6443/tcp
sudo ufw allow 443/tcp
```

I want to make sure the firewall is disabled or ports are allowed at this point. I will just disable the firewall for now

```
sudo ufw disable
```

## Install k3s on worker nodes

We need to extract the token from the master node that will be used to join the the worker nodes to the master

```
sudo cat /var/lib/rancher/k3s/server/node-token
```

Then run the k3s script to install k3s

```
curl -sfL https://get.k3s.io | K3S_URL=https://myserver:6443 K3S_TOKEN=mynodetoken sh -
```


## Kubeconfig

Back on primary node

```
mkdir ~/.k3s
sudo cp /etc/rancher/k3s/k3s.yaml ~/.k3s/config
sudo chown $USER.$USER ~/.k3s/config
export KUBECONFIG=$HOME/.k3s/config
```