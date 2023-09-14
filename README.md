Creating a README file for your GitHub repository is a great way to document your setup and instructions for others to use. Here's a README file based on the commands and instructions you provided for deploying Kubernetes on CentOS 7:

```markdown
# Kubernetes Deployment on CentOS 7

This guide outlines the steps to set up a Kubernetes cluster on CentOS 7 with one master node and one worker node.

## Prerequisites

- Master Node: t4.medium with 4 GB RAM
- Worker Node: t2.small with 2 GB RAM
- CentOS 7 installed on both nodes

## Setup Master Node

1. SSH into your master node as the root user:

   ```bash
   ssh root@<MasterServerIP>
   ```

2. Install necessary packages:

   ```bash
   yum install nano
   ```

3. Change the hostname to 'master' for better understanding:

   ```bash
   nano /etc/hostname
   ```

4. Reboot the system to apply the hostname change:

   ```bash
   systemctl reboot
   ```

5. Install Docker:

   ```bash
   yum install -y -q yum-utils device-mapper-persistent-data lvm2
   yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
   yum install -y -q docker-ce
   service docker start
   chkconfig docker on
   ```

6. Disable SELinux and set the firewall to permissive:

   ```bash
   setenforce 0
   sed -i --follow-symlinks 's/^SELINUX=enforcing/SELINUX=disabled/' /etc/sysconfig/selinux
   ```

7. Disable the firewall:

   ```bash
   systemctl disable firewalld
   ```

8. Disable swap:

   ```bash
   sed -i '/swap/d' /etc/fstab
   swapoff -a
   ```

9. Configure kernel parameters:

   ```bash
   cat >>/etc/sysctl.d/kubernetes.conf<<EOF
   net.bridge.bridge-nf-call-ip6tables = 1
   net.bridge.bridge-nf-call-iptables = 1
   EOF
   sysctl --system
   ```

10. Set up the Kubernetes repository:

    ```bash
    cat >>/etc/yum.repos.d/kubernetes.repo<<EOF
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

11. Install kubeadm, kubelet, and kubectl:

    ```bash
    yum install -y kubeadm-1.15.6-0.x86_64 kubelet-1.15.6-0.x86_64 kubectl-1.15.6-0.x86_64
    systemctl start kubelet
    systemctl enable kubelet
    ```

12. Initialize the Kubernetes master node:

    ```bash
    kubeadm init --apiserver-advertise-address=<MasterServerIP> --pod-network-cidr=192.168.0.0/16
    ```

13. Create a new user and copy Kubernetes configuration:

    ```bash
    useradd lija
    passwd lija
    mkdir /home/lija/.kube
    cp -i /etc/kubernetes/admin.conf /home/lija/.kube/config
    chown -R lija:lija /home/lija/.kube
    ```

14. Make sure the new user can run sudo commands:

    ```bash
    nano /etc/sudoers
    # Add the following line:
    user_name ALL=(ALL) ALL
    ```

## Setup Worker Node

1. SSH into your worker node as the root user:

   ```bash
   ssh root@<WorkerNodeIP>
   ```

2. Install necessary packages (follow steps 2-5 from the Master Node setup).

3. Join the worker node to the Kubernetes cluster (use the token obtained during Master Node setup):

   ```bash
   kubeadm join <MasterServerIP>:6443 --token <Token> --discovery-token-ca-cert-hash <Hash>
   ```

## Additional Configuration

1. Install wget and Calico networking (on the master node):

   ```bash
   yum install wget
   wget https://docs.projectcalico.org/v3.9/manifests/calico.yaml
   kubectl create -f calico.yaml
   ```

2. To obtain the token for adding more worker nodes, run the following command on the master node:

   ```bash
   kubeadm token create --print-join-command
   ```

## Verification

- Check cluster nodes:

  ```bash
  kubectl get nodes
  ```

- Verify Kubernetes version and cluster info:

  ```bash
  kubectl version --short
  kubectl cluster-info
  ```

- Check pods in the kube-system namespace:

  ```bash
  kubectl get pods -n kube-system -o wide
  ```

## Kubernetes Dashboard

Metricserver and InfluxDB are used for the Kubernetes dashboard.

Feel free to adapt these instructions to your specific setup and requirements. Good luck with your Kubernetes deployment!
```

Make sure to replace `<MasterServerIP>`, `<WorkerNodeIP>`, `<Token>`, `<Hash>`, and `user_name` with your actual values. Additionally, you may want to add more details or explanations where necessary.
