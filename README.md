# kubedmkubernetesdemo
Kubernetes Cluster Creation Using kubeadm


### Prerequisites:
- You have a master node with at least 4 GB of RAM.
- Two worker nodes, each with 2GB of RAM (t2.micro).
- You are using CentOS 7.
- You have root or sudo privileges on the master node.
- You have worker nodes prepared (not covered in this guide).
![Screenshot 2023-09-11 182945](https://github.com/lija12-3/kubedmkubernetesdemo/assets/105269384/139f9c0e-7e40-4047-ac44-bcfe629485b3)

### Step 1: Change Hostname
1. SSH into your master node or access it directly.
2. Open the hostname file for editing:
   ```bash
   sudo su
   yum install nano
   nano /etc/hostname
   ```
3. Change the hostname to "master" and save the file.
4. Reboot the master node to apply the new hostname:
   ```bash
   systemctl reboot
   ```

### Step 2: Install Docker
1. Log in as the root user:
   ```bash
   sudo su
   ```
2. Install Docker and start the service:
   ```bash
   yum install -y -q yum-utils device-mapper-persistent-data lvm2
   yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
   yum install -y -q docker-ce
   chkconfig docker on
   service docker start
   ```

### Step 3: Disable SELinux and Firewall
1. Disable SELinux by editing the SELinux configuration file:
   ```bash
   nano /etc/sysconfig/selinux
   ```
   Change `SELINUX=enforcing` to `SELINUX=disabled` and save the file.
   
2. Disable the firewall:
   ```bash
   systemctl disable firewalld
   ```
3. Disable swap
4. ```bash
   sed -i '/swap/d' /etc/fstab
   swapoff -a
```

### Step 4: Configure Kernel Parameters
1. Enable required kernel parameters by creating a configuration file:
   ```bash
   cat >> /etc/sysctl.d/kubernetes.conf <<EOF
   net.bridge.bridge-nf-call-ip6tables = 1
   net.bridge.bridge-nf-call-iptables = 1
   EOF
   ```

2. Reload the sysctl configuration:
   ```bash
   sysctl --system
   ```

### Step 5: Add Kubernetes Repository and Install Kubernetes Tools
1. Add the Kubernetes repository by creating a Kubernetes repo file:
   ```bash
   cat >> /etc/yum.repos.d/kubernetes.repo <<EOF
   [kubernetes]
   name=Kubernetes
   baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
   enabled=1
   gpgcheck=1
   repo_gpgcheck=1
   gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
           https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
   EOF
   ```

2. Install Kubernetes tools (kubeadm, kubelet, kubectl):
   ```bash
   yum install -y kubeadm kubelet kubectl
   ```

3. Enable and start the kubelet service:
   ```bash
   service kubelet status
   service kubelet start
   chkconfig kubelet on
  
```

### Step 6: Initialize the Kubernetes Cluster (Master Node)
ifconfig
on master node
kubeadm init --apiserver-advertise-address=<MasterServerIP> --pod-network-cidr=192.168.0.0/16
1. Sometimes, you might need to reset the Kubernetes configuration on the master node:
   ```bash
   kubeadm reset
   ```

2. Initialize the Kubernetes cluster on the master node:
   ```bash
   kubeadm init
   ```

3. After initialization, you'll receive a command that includes a token to join worker nodes to the cluster. Copy this command for later use.

### Step 7: Configure a Non-Root User
1. Create a non-root user (e.g., "lija"):
   ```bash
   useradd lija
   passwd lija
   # Enter a new password for the user.
   ```

2. Create the `.kube` directory for the user:
   ```bash
   mkdir /home/lija/.kube
   ```

3. Copy the Kubernetes configuration to the user's directory:
   ```bash
   cp -i /etc/kubernetes/admin.conf /home/lija/.kube/config
   ```

4. Set the ownership of the `.kube` directory to the user:
   ```bash
   chown -R lija:lija /home/lija/.kube
   ```

### Step 8: Install a Network Plugin (Calico)
1. Switch to the "lija" user:
   ```bash
   su - lija
   ```

2. Download the Calico network plugin YAML file:
   ```bash
   wget https://docs.projectcalico.org/v3.9/manifests/calico.yaml
   ```

3. Move the Calico YAML file to the user's home directory:
   ```bash
   mv calico.yaml /home/lija
   ```

4. Apply the Calico network plugin:
   ```bash
   kubectl create -f /home/lija/calico.yaml
   ```

### Step 9: Join Worker Nodes
1. On each worker node, run the "kubeadm join" command that you obtained from the master node during initialization. This will join the worker node to the cluster.

### Step 10: Verify Cluster
1. On the master node, you can verify that the cluster is running and that the nodes are joined by running:
   ```bash
   kubectl get nodes
   ```

This should display a list of nodes, including the master node and any worker nodes you've joined.

That's it! You've now set up a basic Kubernetes cluster on CentOS 7 using `kubeadm`. You can proceed to deploy applications and manage your cluster using `kubectl`. Make sure to refer to the official Kubernetes documentation for more advanced configurations and security considerations.
