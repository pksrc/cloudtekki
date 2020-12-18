---
title: "Beginner's Guide: Bootstrapping kubernetes on Ubuntu 20.04 using kubeadm w/ Calico"
date: 2020-10-24T00:00:00-00:00
draft: false
categories: [KUBERNETES]
tags:
- Kubernetes
- Automation
- Ubuntu
- Calico
---

This is not going to be an eye-opening new post about bootstrapping Kubernetes - a lot of people have done it and several blog posts already exists. I'm hoping to capture the information here to serve as an easy documentation for myself and other beginners in the area of Kubernetes and Cloud Native

## Getting Started

As a baseline, here are my assumptions about those following along or to my future self

- You already have some (atleast 2) VMs or Physical hosts (any x86 computer) available
- You are able to install some linux distro on those VMs or Physical hosts - For the purpose of this blog, I'm going to be using [Ubuntu 20.04](https://releases.ubuntu.com/20.04/) and providing steps for that. 
- You can setup static IP on these servers after installing the OS. I'll provide the steps for Ubuntu 20.04 here which uses `Netplan` to configure network

## Installing Ubuntu

Installing Ubuntu is pretty straight forward using the ISO. if you want to skip ahead, click [here](#clone-to-a-vm)

Download 20.04 live server iso - https://releases.ubuntu.com/20.04/

**Machine Requirements (that i'm going to be using)**
- CPU: 1 gigahertz or better
- RAM: 2 gigabyte or more
- Disk: 16 gigabytes

**Procedure** (excerpt below and detailed here https://ubuntu.com/server/docs/install/step-by-step)
- Choose your language
- Update the installer (if offered)
- Select your keyboard layout
- Configure networking (the installer attempts to configure wired network interfaces via DHCP which is fine is you are using a VM and are going to create a template)
- Enter a hostname, username and password
- For storage, leave “use an entire disk” checked, and choose a disk to install to, then select “Done” on the configuration screen and confirm the install
- Configure a proxy or custom mirror if you have to in your network
- Select OpenSSH server from the list of packages to be installed
- You will now see log messages as the install is completed
- Select restart when this is complete, and log in using the username and password provided

{{<youtube RIz359xF-8U>}}

Unlike the sped up video above, this install can be a time consuming process to manually attempt on a number of servers (20-30 minutes per install). If you are using VMs, you should consider taking a snapshot and creating a template of your Base Ubuntu VM right after the install. This way when you need more copies in the future or want to start over, you have a clean Ubuntu install readily available. 

## Clone to a VM

From the Base Ubuntu template that you have, create two clones one for the master and one for the worker node. If you have gone the route of installing Ubuntu from ISO on two machines without cloning, that works too.. 

- k8s-m1: will be the control node
- k8s-w1: will be the capacity node

For Highly Available clusters for Kubernetes using kubeadm, refer to this [kubernetes.io documentation](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/)

## Configuring Ubuntu for Kubernetes

With our two machines now ready, we need to do some configuration that is specific to kubernetes and docker on these servers. 

> There are not a lot of changes that needs to be done and can be done manually using multi window editing capabilities that iTerm2 offers 
> - shortcuts cmd-d and cmd-shift-d divide an existing session vertically or horizontally, respectively 
> - shortcuts cmd-shift-i and cmd-i to edit in multiple pane or go back to editing in a single pane, respectively

### Step 1 - Disable swapoff 


```bash
#disables swap immediately
sudo swapoff –a 

#ensures swap remains disabled after reboot
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

There is a larger and better thread that conveys why this has to be done (or shouldn't be) and where this falls within the scope of Kubernetes project on the things to get done - https://github.com/kubernetes/kubernetes/issues/53533 

### Step 2 - Setup Static IP

Ubuntu networking is configured with [Netplan](https://netplan.io/examples/). Succinctly, you want to edit `/etc/netplan/01-netcfg.yaml` to look like this

```bash
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
     dhcp4: no
     addresses: [10.10.10.10/24]
     gateway4: 10.10.10.1
     nameservers:
       addresses: [8.8.8.8,8.8.4.4]
```
Once you have the Netplan config updated as required for your network, go ahead and run 

```bash
sudo netplan apply
```

> Here is another article that helps guide you through this change - https://linuxize.com/post/how-to-configure-static-ip-address-on-ubuntu-20-04/#configuring-static-ip-address-on-ubuntu-server

### Step 3 - Install Docker

Next step is to install Docker which Kubernetes will be using as the container runtime. 

```bash
#install docker
sudo apt install docker.io -y

#Add your user to the docker group
groupadd docker
sudo usermod -aG docker $USER

# enable and start docker
sudo systemctl enable docker
```

### Step 4 - Prepping for Kubernetes

The last item remaining to prep our servers is to download the components for Kubernetes and kubeadm as described [here](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/) and shown below

**Steps**:
- Add the signing key to verify k8s download
- Add kubernetes repos to the repo list
- Install these on all servers
    - **Kubeadm** which is going to bootstrap the cluster
    - **Kubectl** which is the CLI used to manage the cluster and 
    - **Kubelet** which is the component that needs to run on all machines to make kubernetes work

```bash
# Install signing keys
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add

# Adding kubenetes repository for APT
apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"

# install k8s components
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

apt-get update

apt-get install kubeadm kubelet kubectl -y
apt-mark hold kubelet kubeadm kubectl
```

## Bootstrap Kubernetes - Control Plane
 
All that is left to do now is to get Kubernetes working! 

### Step 1: Getting the Control Node ready (a.k.a master node)

To do that we are going to leverage `kubeadm` to bootstrap kubernetes using a single command (maybe a couple..). First we want to get our Control node up and running with Kubernetes. This Control Node will run 
- API Server (**kube-apiserver**): This is the API gateway (if you will) for all the components to interact with
- Etcd (**etcd**): This is the database (if you will) for all the kubernetes cluster data (from nodes to pods to everything)
- Scheduler (**kube-scheduler**): This is the backend scheduler/job(if you will) that continues to ensures the pods are running and properly distributed to all the nodes
- Controller Manager (**kube-controller-manager**): For every type of object within Kubernetes, there are Controllers available that specialize in managing that object (for example: Node, Replication, Endpoint etc)

> This is the noob version of describing kubernetes. If you choose to, you can always learn more at https://kubernetes.io/docs/concepts/overview/components/

We start by initializing kubernetes on the Control Node and all the info that we need the pod network CIDR at this point. Ensure that the POD CIDR does not overlap with any of your existing networking IP ranges. 

```bash
# using the default POD NETWORK CIDR for calico
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
```

If you have more than one NICs you will need to specify which IP address kubernetes should use for the API address (this is critical since all the other components need to talk to it for proper functionality). If you have it all working properly, you should get something similar to what i have below

<image src="/img/k8s/bootstrap-k8s/bootstrap.png">

### Step 2: Setup Kubectl for the user account

Kubeadm will also provide some instructions to setup the user account with the right kube config. Run the following as a regular user:

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### Step 3: Setup Pod networking

Once you have kubeconfig setup, you should now have kubectl working and be able to deploy a pod network to the cluster. I'm going to using `calico` for the networking and the networking policy. 

```bash
curl https://docs.projectcalico.org/manifests/calico-etcd.yaml -o calico.yaml
```
In my case, since I'm using `etcd` for the datastore and also using TLS, i had to update the `calico.yaml` with the right information (as shown). I had missed/messed up this step and ended up troubleshooting for half a day. 

```yaml
---
# Source: calico/templates/calico-etcd-secrets.yaml
# The following contains k8s Secrets for use with a TLS enabled etcd cluster.
# For information on populating Secrets, see http://kubernetes.io/docs/user-guide/secrets/
apiVersion: v1
kind: Secret
type: Opaque
metadata:
  name: calico-etcd-secrets
  namespace: kube-system
data:
  # Populate the following with etcd TLS configuration if desired, but leave blank if
  # not using TLS for etcd.
  # The keys below should be uncommented and the values populated with the base64
  # encoded contents of each file that would be associated with the TLS data.
  # Example command for encoding a file contents: cat <file> | base64 -w 0
  etcd-key: <base 64 encoded etcd Key>
  etcd-cert: <base 64 encoded etcd Certificate>
  etcd-ca: <base 64 encoded etcd CA>
---
```
You can get the base 64 encoded values by using the commands provided below. The file locations is well known and can also be obtained by running a kubectl describe command on the etcd pod. 

```bash
# CA root certificate
cat /etc/kubernetes/pki/etcd/ca.crt | base64 -w 0

# ETCD private and public key
sudo cat /etc/kubernetes/pki/etcd/server.key | base64 -w 0
cat /etc/kubernetes/pki/etcd/server.crt | base64 -w 0
```
Once you have that updated you are also going to update the `calico.yaml` to look like the below. The `calico-secrets/etcd-xx` are created using the base64 encoded information that we updated above. 

```yaml
# Source: calico/templates/calico-config.yaml
# This ConfigMap is used to configure a self-hosted Calico installation.
kind: ConfigMap
apiVersion: v1
metadata:
  name: calico-config
  namespace: kube-system
data:
  # Configure this with the location of your etcd cluster.
  etcd_endpoints: "https://<API IP>:2379"
  # If you're using TLS enabled etcd uncomment the following.
  # You must also populate the Secret below with these files.
  etcd_ca: "/calico-secrets/etcd-ca"
  etcd_cert: "/calico-secrets/etcd-cert"
  etcd_key: "/calico-secrets/etcd-key"
```
With these edits in place, you should now be good to apply the calico.yaml to your kubernetes cluster. 

```bash
kubectl apply -f calico.yaml
```

> More information on how to set up calidco can be found here - https://docs.projectcalico.org/getting-started/kubernetes/self-managed-onprem/onpremises

## Bootstrap Kubernetes - Capacity Nodes

Once you have the Control Plane Node up and running, we can proceed to get the Capacity Node joined to the Kubernetes Cluster. To do so, you'll need to run the command provided below. If you do this immediately after running the `kubeadm init` on the Control Plane node, kubeadm will provide the exact command that needs to be run on the Capacity Nodes. 

`kubeadm join --token <token> <control-plane-host>:<control-plane-port> --discovery-token-ca-cert-hash sha256:<hash>`

In the event you need to add more nodes after and if you need to get the token and the CA certificate hash, here is how you can get them

```bash
# list the tokens
kubeadm token list
# create a new token
kubeadm token create

# to get the hash
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
```

After running the `kubeadm join` command you should get a response like below if your Capacity node was able to join the Kubernetes Controller

<img src="/img/k8s/bootstrap-k8s/node.png">

Hope you are successfully able to follow these instructions and get a working Kubernetes cluster with Calico on Ubuntu 20.04 using kubeadm. 

Please provide feedback by leaving a comment and share this post using the social media icons shown below. That's all folks! 