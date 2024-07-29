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
