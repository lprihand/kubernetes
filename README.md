# KUBERNETES INSTALLATION 

This is tutorial to setup kubernetes cluster one master with two worker nodes on VMWare Workstation. I used Linux Ubuntu 24.04 Server edition with setup of two LAN ethernet. First one using NAT and second one using Bridge with manual address. 
Since I like to use NetworkManager in order to use nmcli command therefore some configuration to enable NetworkManager prior installing Kubernetes. Ubuntu Server doesn't come with network-manager by default unlike desktop version.

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
Apply below on all VM nodes, kindly note that here I used containerd.

```
#sysctl params required by setup, params persist across reboots

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
EOF

#Apply sysctl params without reboot
sudo sysctl --system

sysctl net.ipv4.ip_forward


sudo apt install ca-certificates curl -y
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install containerd.io


containerd config default | sed 's/SystemdCgroup = false/SystemdCgroup = true/' | sed 's/sandbox_image = "registry.k8s.io\/pause:3.6"/sandbox_image = "registry.k8s.io\/pause:3.9"/' | sudo tee /etc/containerd/config.toml

sudo systemctl restart containerd

sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list


sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

Apply below on Master node only, note that below I used IP address of Bridge interface used on VM. 

```
sudo kubeadm init --pod-network-cidr=10.200.0.0/16 --apiserver-advertise-address=192.168.116.215 

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

I also need to ensure cluster IP using bridge interface by doing additional configuration under */var/lib/kubelet/kubeadm-flags.env*. Apply on all node with respective of IP address assign on each VM.

```
#add below
--node-ip=192.168.116.215

#below content after modification
KUBELET_KUBEADM_ARGS="--node-ip=192.168.116.215 --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock --pod-infra-container-image=registry.k8s.io/pause:3.9"

#restart kubelet
sudo systemctl restart kubelet
```

Finally apply below command on each Worker node
```
sudo kubeadm join 192.168.116.215:6443 --token axunvk.92d9uxlk2rvlmk6l \
        --discovery-token-ca-cert-hash sha256:941d4d2190cbb615faa944b57de3933eb1bc1ea98e21d1ff30224b141d5ae342 

```

Check on Master node if cluster are ready

```
lprihand@master-k8s:~$ kubectl get node 
NAME         STATUS   ROLES           AGE    VERSION
master-k8s   Ready    control-plane   2d2h   v1.30.3
node1-k8s    Ready    <none>          2d2h   v1.30.3
node2-k8s    Ready    <none>          2d2h   v1.30.3
lprihand@master-k8s:~$ kubectl get node -o wide
NAME         STATUS   ROLES           AGE    VERSION   INTERNAL-IP       EXTERNAL-IP   OS-IMAGE           KERNEL-VERSION     CONTAINER-RUNTIME
master-k8s   Ready    control-plane   2d2h   v1.30.3   192.168.116.215   <none>        Ubuntu 24.04 LTS   6.8.0-39-generic   containerd://1.7.19
node1-k8s    Ready    <none>          2d2h   v1.30.3   192.168.116.216   <none>        Ubuntu 24.04 LTS   6.8.0-39-generic   containerd://1.7.19
node2-k8s    Ready    <none>          2d2h   v1.30.3   192.168.116.217   <none>        Ubuntu 24.04 LTS   6.8.0-39-generic   containerd://1.7.19
```
