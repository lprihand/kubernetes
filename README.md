# kubernetes

This is tutorial to setup kubernetes cluster one master with two worker nodes on VMWare Workstation. I used Linux Ubuntu 24.04 Server edition with setup of two LAN ethernet. First one using NAT and second one using Bridge with manual address. 
Since I like to use NetworkManager in order to use nmcli command therefore some configuration to enable NetworkManager prior installing Kubernetes.

```
sudo apt update
sudo apt install network-manager -y
```

After that we need to address netplan configuration to use NetworkManager, please comment everything inside */etc/netplan/50-cloud-init.yaml* file and replace with below

```
network:
    version: 2
    renderer: NetworkManager
```

and then do below command line 

```
sudo netplan apply
sudo netplan generate

sudo systemctl disable systemd-networkd.service
sudo systemctl mask systemd-networkd.service
sudo systemctl stop systemd-networkd.service

sudo reboot 
```

check if you now are able to use below command line 

```
nmcli 
nmcli device
nmcli connection
# if you like to edit configuration in UI mode
nmtui 
```

Now we need to disable SWAP, reason to disable it is Kubernetes need to ensure performance, as using SWAP will slow the performance (for more detail please dig it from google). 

```
sudo swapoff -a
```

to disable SWAP permanently, kindly modify swap line configuration with commenting the line 

```
sudo vim /etc/fstab

#/swap.img      none    swap    sw      0       0
```

and then mount it again 

```
mount -a
```
