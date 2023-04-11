Kubernetes CKA Study
=======================

> Rodolfo R. de Almeida | November 11th, 2021

------------------------------------------

This repository will be used to post all topics related to Kubernetes CKA certification. My study will be based on the CLoud Guru training called "Certified Kubernetes Administrator (CKA)".

*[I am using this basic writing format to create this document.](https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax)*

------------------------------------------

# Table of Contents

1. [CKA Certification Exam](#CKA-Certification-Exam)
2. [K8S Getting Started](#k8s-Getting-Started)
3. [Building a Kubernetes Cluster](#building-a-kubernetes-cluster)
4. [LAB01 Building a Kubernetes Cluster](#lab01-building-a-kubernetes-cluster)
5. [Namespaces in K8s](#namespaces-in-k8s)
6. [LAB02 Working with Kubernetes Namespaces](#LAB02-working-with-kubernetes-namespaces)
7. [Kubernetes Management](#kubernetes-management)
8. [LAB03 Performing Kubernetes Upgrade](#LAB03-performing-kubernetes-upgrade)
9. [Backing Up and Restoring Etcd Cluster Data](#backing-up-and-restoring-etcd-cluster-data)
10. [LAB04 Backing Up and Restoring Etcd](#LAB04-backing-up-and-restoring-etcd)
11. [Working with kubectl](#working-with-kubectl)
12. [kubectl Tips](#kubectl-tips)
13. [RBAC in k8s](#rbac-in-k8s)
14. [Creating Service Accounts](#creating-service-accounts)
15. [Inspecting Pod Resource Usage](#inspecting-pod-resource-usage)
16. [Managing Application Configuration](#managing-application-configuration)
17. [Managing Containers Resources](#managing-containers-resources)
18. [Monitoring Container Health with Probes](#monitoring-container-health-with-probes)
19. [Building Self-Healing Pods](#building-self-healing-pods)

---------------

CKA Certification Exam
====================

## CKA Exam
The CKA exam is periodically updated. The current CKA version is 1.24 and the most significant change is that the exam now uses K8s version 1.24.
The exam version maches the kubernetes version.

## PSI Bridge Proctoring Platform
The exam platform is PSI Bridge Proctoring Platform and it uses the PSI Secure Browser that is required to do the exam.
It is permited to use Documentation link but no personal bookmarks.
It is permited only one active monitor during the exam.


K8s Getting Started
====================

## K8s Basics Quick Recap

* What is kubernetes?
 
  Kubernetes is a portable, extensible, open-source platform for manageing containerized workloads and services, that facilitates both declarative configuration and automation.

* What does "k8s" mean?

  K8s is simply a short for Kubernetes. The 8 represents the 8 letters between K ad S!
 
* What K8s actualy does?

  Kubernetes creates a cluster to manage all nodes (servers) and conteiners is a centralized way. When you run a conteiner inside a Kubernetes cluster you do not need to worry in which server the conteiner will run, because kubernetes will manage that.
 
* K8s Features

  Container Orchestration - The primary purpose of Kubernetes is to dynamicaly manage conteiners across multipe host systems.
  
  Application Reliability - Kubernetes makes it easier to build reliable, self-healing, and scalable applications.
  
  Automation - Kubernetes offers a variety of features to help automate the management of your container apps.

  
## K8s Architectural Overview

* K8s Control Plane

  It is what runs our kubernetes cluster.
  A collection of multiple components that are responsible to manage the cluster globally.
  Control plane components can run on any machine in the cluster, but usually are run on dedicated controller machines.
  
  * **kube-api-server** serves the Kubernetes API, the primary interface to the control plane and the cluster itself.
    When interacting with your Kubernetes cluster, you will usually do so using the Kubernetes API.
    
    *[If you do not know what is API please check this link.](/whatisAPI.md)*
   
    
  * **Etcd** is the backend data store for the kubernetes cluster, and provides high availability sotrage for all data relating to the state of the cluster.
   
  * **Kube-scheduler** handles scheduling, the process of selecting an available node in the cluster on which to run containers.
  
  * **kube-controller-mnager** runs a collection of multiple controller utilities in a single process. These controllers carry out a variety of automation-related tasks within the Kubernetes cluster.
   
  * **cloud-controller-manager** provides an interface between Kubernetes and various cloud platorms. It is only used when using cloud-based resources alongside Kubernetes.
  
* K8s Nodes
  
  Nodes are the machines where the conteiners managed by the cluster run. A cluster can have any number of nodes. Various node components manage conteiners on the machine and communicate with the control plane.
  
  * **Kubelet** is the Kubernetes agent that runs on each node. It communicates with the control plane and ensures that containers are run on its node as instructed by the control plane. It also handles the process of reporting container status and other data about containers back to the control plane.
  
  * **Container Runtime** is not built into Kubernetes. It is a separate piece of software that is responsible for actually running containers on the machine. Kubernetes supports multiple container runtime implementations. Some popular container runtimes are Docker and Containerd.
    
  * **kube-proxy** is a network proxy. It runs on each node and handles some tasks related to providing networking between containers and services in the cluster.

* A K8s Cluster

  This is what the hole kubernetes cluster lookslike.
  
![image](https://user-images.githubusercontent.com/113181949/201423697-3e8189af-fbf5-438d-93c8-f1fa11e643bd.png)

     
Building a Kubernetes Cluster
====================

### What is kubeadm?

It is a tool that will simplify the process of setting up our Kubernetes cluster. This tool provide `kubeadm init` and `kubeadm join` as best-practice "fast paths" for creating Kubernetes clusters. kubeadm performs the actions necessary to get a minimum viable cluster up and running. By design, it cares only about bootstrapping, not about provisioning machines. Likewise, installing various nice-to-have addons, like the Kubernetes Dashboard, monitoring solutions, and cloud-specific addons, is not in scope.


### Playground Server Setup

![image](https://user-images.githubusercontent.com/113181949/207026751-965e7f24-2f39-4123-8d01-07bd0aeca4d6.png)

LAB01 Building a Kubernetes cluster
====================

1. Deploy three nodes in your cloud provider.

2. Change the server host name to controller and worker nodes.

   `sudo hostnamectl set-hostname k8s-controller1`

3. Setup the host file with all servers hostname for name resolution.
   
   `sudo vi /etc/hosts`
   
   Exemple to add the IP Address and hostname server. 
   
   *Allways uses private IP Address because they are static. Public IP Address is dynamic and might change*
   
   ![image](https://user-images.githubusercontent.com/113181949/207029232-8bab9ee2-827b-4087-b79a-dd39c3a9045b.png)
   
   > __Note__ 
   > In order to hostname changes take effect, logout and login back to the server.

4. Install and configure conteinerd
   
   - Enable `overlay` and `br_netfilter`. Load kernel modules on startup and everytime the server boots `overlay` and `br_netfilter` will be enabled.
   
   ```
   cat << EOF | sudo tee /etc/modules-load.d/conteinerd.conf 
   overlay
   br-netfilter
   EOF
   ```
   
   - This commands will immediatly enable those modules and it is not required to reboot the host to enable them.
   
   ```
   sudo modprobe overlay
   sudo modprobe br_netfilter
   ```
   
   - Network configurations required that kubernetes is gonna need.
   
   ```
   cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
   net.bridge.bridge-nf-call-iptables = 1
   net.ipv4.ip_forward = 1
   net.bridge.bridge-nf-call-ip6tables = 1
   EOF
   ```
   
   - To apply all this configurations 

   ```
   sudo sysctl --system
   ```
   
5. Install and configure containerd.

   ```
   sudo apt-get update && sudo apt-get install -y containerd
   
   sudo mkdir -p /etc/containerd
   
   sudo containerd config default | sudo tee /etc/containerd/config.toml
   
   sudo systemctl restart containerd
   ```

6. On all nodes, disable swap.
  
   What is Swap on linux? 
   
   Swap space in Linux is a part of a machine's virtual memory used when the physical memory runs out of space. Especially when working with a large program or application where a need for extra memory arises, swap space is an ideal choice to hold temporarily inactive memory pages.
   Sometimes you need to disable Swap on a Linux server. Kubernetes worker nodes need to have Swap turned off in order to increase the performance and stability.
   
   ```
   sudo swapoff -a
   ```

7. On all nodes, install kubeadm, kubelet, and kubectl.

   ```
   sudo apt-get update && sudo apt-get install -y apt-transport-https curl
   
   curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
   
   cat << EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
   deb https://apt.kubernetes.io/ kubernetes-xenial main
   EOF
  
   sudo apt-get update
  
   sudo apt-get install -y kubelet=1.24.0-00 kubeadm=1.24.0-00 kubectl=1.24.0-00
  
   sudo apt-mark hold kubelet kubeadm kubectl
   ```

8. On the control plane node only, initialize the cluster and set up kubectl access.
  
   ```
   sudo kubeadm init --pod-network-cidr 192.168.0.0/16 --kubernetes-version 1.24.0
   
   mkdir -p $HOME/.kube
   
   sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  
   sudo chown $(id -u):$(id -g) $HOME/.kube/config
   ```
  
9. Verify the cluster is working.

    ```
    kubectl get nodes
    ```
   
10. Install the Calico network add-on.

    ```
    kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
    ```
   
11. Get the join command (this command is also printed during kubeadm init . Feel free to simply copy it from there).
    
    ```
    kubeadm token create --print-join-command
    ```
   
12. Copy the join command from the control plane node. Run it on each worker node as root (i.e. with sudo ).

    ```
    sudo kubeadm join ...
    ```
   
13. On the control plane node, verify all nodes in your cluster are ready. Note that it may take a few moments for all of the nodes to

    ```
    enter the READY state.
    kubectl get nodes
    ```
Namespaces in K8S
====================

Namespaces are virtual clusters backed by the same physical cluster. Kubernetes objects, such as pods and containers, live in namespaces. Namespaces are a way to separate and organize objects in your cluster.

## List existing Namespaces

You can list existing namespaces with kubectl:

```
kubectl get namespaces
```

All clusters have a default namespace. This is used when no namespace is specified. kubeadm also creates a kube-system namespace for system components.

## Specify a Namespace

When using kubectl, you may need to specify a namspace. You can do this with the `--namespace` flag.

Alternatively you can use the -n flag, which is the short form of the --namespace flag.

```
kubectl get pods --namespaces my-namespace
```

You can also get pods from all namespaces using the command below.

```
kubectl get pods --all-namespaces
```

> __Note__ 
> If you do not specify a namespace, the default namespace is assumed.


## Create a Namespace

You can create a namespace with kubectl.

```
kubectl create namespace my-namespace
```

LAB02 Working with Kubernetes Namespaces
========================================

## Introduction
Namespaces are a central component of any Kubernetes infrastructure. This lab will give you the opportunity to work with namespaces in a functioning cluster. You will be able to practice the process of creating, using, and navigating Kubernetes namespaces.

## Solution
Log in to the provided control plane node server using the credentials provided:
```
ssh cloud_user@<PUBLIC_IP_ADDRESS>
```

Create the dev Namespace
Create a namespace in the cluster called dev:
```
kubectl create namespace dev
```

Get a List of the Current Namespaces
List the current namespaces:
```
kubectl get namespace
```

Save the namespaces list to a file:
```
kubectl get namespace > /home/cloud_user/namespaces.txt
```

Verify the list saved to the file:
```
cat namespaces.txt
```

We should see the list of namespaces.

## Find the quark Pod's Namespace

Locate the quark pod:
```
kubectl get pods --all-namespaces
```

Copy the name of the namespace where the quark pod is located.

Create a file in which to save its name: :
```
vi /home/cloud_user/quark-namespace.txt
```

Paste in the name of the `quark` pod's namespace.

Save and exit the file by pressing Escape followed by `:wq.`

Kubernetes Management
====================

## Intruduction to High Availability in K8s

K8s favilitates high-availability applications, but you can also design the cluster itself to be high available. To do this, you need multiple control plane nodes.

![image](https://user-images.githubusercontent.com/113181949/209573691-b12e1c73-0391-400e-a931-e90eb36c2ce7.png)

## Stacked etcd
The etcd runs on the same nodes as the rest of control plane components.
This is the setup pattern used by clusters that are set up using kubeadm. So we used kubeadm to build our cluster and kubeadm uses this stacked etcd methodology. That means that our control plane node, which is running all of the different control plane components, is also running etcd, and in a high availability setup, each one of those individual control plane nodes would have its own etcd instance.

![image](https://user-images.githubusercontent.com/113181949/209573214-7246b411-2c80-4353-be8b-7e8e6209085f.png)

## External etcd

With external etcd, we have etcd running on completely separate nodes. In the context of high availability, we could then have multiple external etcd nodes in a high availability cluster, and that set of servers would be a completely different set of servers from the
ones that are running our normal Kubernetes control plane components. With the external etcd model, you can have any number of Kubernetes control plane instances and any number of SED nodes. So in this lesson, we've provided a brief high-level overview of what high availability looks
like in a Kubernetes cluster.


![image](https://user-images.githubusercontent.com/113181949/209573266-fc3a280c-fa21-4fee-ac19-f460ff6ea63e.png)


 > __Note__ 
 > Link to access [Options for High Availability Topolology](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/ha-topology/)


## Introduction to K8s Management Tools

What are k8s Managemente tools?

There is a variety of management tools available for kubernetes. THese tools interface with kubernetes to provide additional functionality. When using Kubernetes, it is a good idea to be aware of some of these tools.

- kubectl
  
  kubectl is the official command line interface for Kubernetes.

- kubeadm
  
  It is a tool for quickly and easily creating Kubernetes clusters.
  
- Minikube
  
  Allows you to automatically set up a local single-node Kubernetes cluster. It igreat for getting k8s up adn running quickly for test and development purposes.
  
- Helm
 
  Provides templating and package management for Kubernetes objects. Can be used to manage your own templates (known charts). You can also download and use shared templates.
  
- Kompose
  
  It helps you translate Docker compose files into Kubernetes objects. If you are using Docker compose for some part of your workflow, you can move your application to Kubernetes easily with Kompose.
  
- Kustomize
-
  It is a configuration management tool for managing Kubernetes object configurations. It allows you to share and re-use templated configurations for Kubernetes applications. It is similar to Helm.
  
  
## Safely Draining a K8s Node

#### What is Draining?

When performing maintenance, you may sometimes need to remove a Kubernetes node from service. To do this, you can drain the node. Containers running on the node will be gracefully terminated (and potentially rescheduled on another node).

#### Draining a Node

To drain a node, use the kubectl drain command.
```
kubectl drain <node name>
```

#### Ignoring DaemonSets

  When draining a node, you may need to ignore DaemonSets (pods that are tied to each node). If you have any DaemonSet pods running on the node, you will likely need to use the ```--ignore-daemonsets``` flag.
  
 ```
 kubectl drain <node name> --ignore-daemonsets
 ```
 
#### Uncordoning a Node

 If the node reamins part of the cluster, yo can allow pods to run on the node again when maintenance is complete using the ```kubectl uncordon``` command.

```
kubectl uncordon <node name>
```

  > __Note__  
   > Be aware that pods running on the Drained node will be started on the available nodes of the cluster, but once the node is uncordoned, it will not be started back in the uncordoned node. The pods will remain running on the nodes and just new pods will be started on the uncordoned node.

  > __Note__  
   > Pods will be started in other nodes if they were deployed as deployments. If a pod is deployed manually in a node, it cannot be started in a different node. The node will be deleted if using ```--force``` flag.


## Upgrading K8s with kubeadm

#### Upgrading with kubeadm
When using kubernetes, you will likely want to periodically upgrade Kubernetes to keep your cluster up to date and kubeadm makes this process easier.

#### Control Plane Upgrade Steps
- Upgrade kubeadm on the control plane node.
- Drain the control plane node.
- Plan the upgrade (kubeadm upgrade plan).
- Apply the upgrade (kubeadm upgrade apply).
- Upgrade kubelet and kubectl on the control plane node.
- Uncordon the control plane node.

#### Worker Node Upgrade Steps
- Drain the node.
- Upgrade kubeadm.
- Upgrade the kubelet configuration (kubeadm upgrade node).
- Upgrade kubelet and kubectl.
- Uncordon the node.

#### Demonstration Upgrading K8s with kubeadm

Drain the control plane node.
```
kubectl drain k8s-control --ignore-daemonsets
```

Upgrade kubeadm.
```
sudo apt-get update && \
sudo apt-get install -y --allow-change-held-packages kubeadm=1.22.2-00
```
Check the version
```
kubeadm version
```

Plan the upgrade.
```
sudo kubeadm upgrade plan v1.22.2
```

Upgrade the control plane components.
```
sudo kubeadm upgrade apply v1.22.2
```

Upgrade kubelet and kubectl on the control plane node.
```
sudo apt-get update && \
sudo apt-get install -y --allow-change-held-packages kubelet=1.22.2-00 kubectl=1.22.2-00
```

Restart kubelet.
```
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

Uncordon the control plane node.
```
kubectl uncordon k8s-control
```

Verify that the control plane is working.
```
kubectl get nodes
```

Upgrade the worker nodes.

> __Note__ : In a real-world scenario, you should not perform upgrades on all worker nodes at the same time. Make sure
enough nodes are available at any given time to provide uninterrupted service.

Run the following on the control plane node to drain worker node 1:
```
kubectl drain k8s-worker1 --ignore-daemonsets --force
```

Log in to the first worker node, then Upgrade kubeadm.
```
sudo apt-get update && \
sudo apt-get install -y --allow-change-held-packages kubeadm=1.22.2-00
```

Upgrade the kubelet configuration on the worker node.
```
sudo kubeadm upgrade node
```

Upgrade kubelet and kubectl on the worker node.
```
sudo apt-get update && \
sudo apt-get install -y --allow-change-held-packages kubelet=1.22.2-00 kubectl=1.22.2-00
```

Restart kubelet.
```
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

From the control plane node, uncordon worker node 1.
```
kubectl uncordon k8s-worker1
```

Repeat the upgrade process to the other worker nodes.

Verify that the cluster is upgraded and working.
kubectl get nodes


LAB03 Performing Kubernetes Upgrade
===============

Kubernetes Cluster

![image](https://user-images.githubusercontent.com/113181949/217599835-b9b34622-2cbe-4f06-83aa-60339c073d51.png)

In this cluster we have three servers; One controle plane and two worker nodes. The cluster was built with Kubeadm.
First we need to upgrade the control plane and then the worker nodes.


Backing Up and Restoring Etcd Cluster Data
====

### Why Back UP etcd?

**etcd** is the backend data data storage solution for your kubernetes cluster. As such, all your kubernetes objects, applications, and configurations are stored in etcd.

Therefore, you will likely data by backing up etcd.


### Backing UP etcd

You can back up etcd command line tool, **etcdctl**.

Use the **etcdctl snapshot save** command to back up the data. 

![image](https://user-images.githubusercontent.com/113181949/217636700-dc24e7bc-17e0-4dba-bb5f-f826dca6e458.png)


### Restoring etcd

You can restore etcd data from a backup using the **etcdctl snapshot restore** command.

You will need to supply some additional parameters, as the restore operation creates a new logical cluster.

![image](https://user-images.githubusercontent.com/113181949/217636989-6ddff387-e3a5-449e-9789-55e7ed755b82.png)


LAB04 Backing Up and Restoring Etcd
===

![image](https://user-images.githubusercontent.com/113181949/218560253-218ae9af-b973-4417-a331-31ac0bced929.png)

#### Step 01 - Check the variable

```
ETCDCTL_API=3 etcdctl get cluster.name \
--endpoints=https://10.0.1.101:2379 \
--cacert=/home/cloud_user/etcd-certs/etcd-ca.pem \
--cert=/home/cloud_user/etcd-certs/etcd-server.crt \
--key=/home/cloud_user/etcd-certs/etcd-server.key
```

The output should be something similar to:

![image](https://user-images.githubusercontent.com/113181949/218561066-5ed91258-9b4d-40b8-9ac7-79166822b4be.png)


#### Step 02 - Taking a Snapshot from Etcd data base

```
ETCDCTL_API=3 etcdctl snapshot save /home/cloud_user/etcd_backup.db \
--endpoints=https://10.0.1.101:2379 \
--cacert=/home/cloud_user/etcd-certs/etcd-ca.pem \
--cert=/home/cloud_user/etcd-certs/etcd-server.crt \
--key=/home/cloud_user/etcd-certs/etcd-server.key
```

The output should be something similar to:

![image](https://user-images.githubusercontent.com/113181949/218561856-c6ad1626-e12c-466c-8a0e-eedfdcd3c16b.png)


### Step 03 - Reset the cluster data

```
sudo systemctl stop etcd
```

```
sudo rm -rf /var/lib/etcd
```
r — recusrsive. all directories and subdirectories will be deleted;
f — force. Force delete all files;


```
sudo ETCDCTL_API=3 etcdctl snapshot restore /home/cloud_user/etcd_backup.db \
--initial-cluster etcd-restore=https://10.0.1.101:2380 \
--initial-advertise-peer-urls https://10.0.1.101:2380 \
--name etcd-restore \
--data-dir /var/lib/etcd
```

The output should be something similar to:

![image](https://user-images.githubusercontent.com/113181949/218563687-edee8404-604d-401e-b60d-e0c8510e3441.png)


### Step 04 - Check Etcd 

![image](https://user-images.githubusercontent.com/113181949/218564201-6d3414fd-1043-4ce6-a3da-66d9f72ddb63.png)


Working with kubectl
===

### What is kubectl?

kubectl is a command line tool that allows you to interact with Kubernetes. kubectl uses the Kubernetes API to communicate with the cluster and carry out your commands. You can use kubectl to deploy applications, inspect and manage cluster resources, and view logs.

### kubectl get

Use **kubectl get** to list objects in the Kubernetes cluster.

- `-o`  - Set the output format
   
   ```kubectl get pods -o wide``` >> Wide provides more detailed information about the command.
   
   ```kubectl get pods -o json``` >> Gives the output in json format
   
   ```kubectl get pods -o yaml``` >> Gives the output in yaml format
   
- `--sort-by`  - Sort output using a JSONPath expression.
   
- `--selector`  - Filter results by label.

   ```kubectl get pods -n kube-system --selector k8s-app=calico-node``` >> this will show the pods that contains this lable.


```
$ kubectl get <object type> <object name> -o <output> --sort-by <JSONPath> --selector <selector>
```

### kubectl describe

You can get detailed information about Kubernetes objects using kubectl describe.

```
kubectl describe <object type> <object name>
```

### kubectl create

Use **kubectl create** to create objects.

Supply a YAML file with -f create an object from a YAML descriptor stored in the file.
If you attempt to create an object that already exists, an error will occur.

```
kubectl create -f <file name>
```

### kubectl apply

kubectl apply is similar to kubectl create. However, if you use kubectl apply on an object that already exists, it will modify the existing object, if possible.

```
$ kubectl apply -f <file name>
```

### kubectl delete

Use **kubectl delete** to delete objects from the cluster.

```
kubectl delete <object type> <object name>
```

### kubectl exec

**kubectl exec** can be used to run commnds inside containers. Keep in mind that, in order for a command to succeed, the necessary software must exist within the container to run it.

```
kubectl exec <pod name> -c <container name> -- <command>
```

   > __Note__ 
   >  `-c` is used only if your pod has multiple containers, with that flag you can specify the container.


kubectl Tips
===

## Imperative Commands
There are the Declarative and the Imperative commands in kubernetes.
The **Declarative** command define objects using data structures such as YAML or JSON.
The **Imperative** command define objects using kubectl commands and flags. 

Example of Imperative command:

```
kubectl create deployment my-deployment --image=nginx
```

*[kubectl cheat sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)*

Run kubectl create to see a list of objects that can be created with imperative commands.

```
kubectl create
```

Create a deployment imperatively.

```
kubectl create deployment my-deployment --image=nginx
```

Do a dry run to get some sample yaml without creating the object.

```
kubectl create deployment my-deployment --image=nginx --dry-run -o yaml
```

   > __Note__ 
   > Dry run is an expression used to indicate a rehearsal of a performance or procedure before the real one. Dry run mode gives you the possibility of issuing a command without side effects for testing an actual command that you intend to run.


Save the yaml to a file.

```
kubectl create deployment my-deployment --image=nginx --dry-run -o yaml > deployment.yml
```


Create the object using the file.

```
kubectl create -f deployment.yml
```

Scale a deployment and record the command.

```
kubectl scale deployment my-deployment replicas=5 --record
kubectl describe deployment my-deployment
```

RBAC in k8s
===

Role-based access control (RBAC) in k8s allows you to control what users are allowed to do and access within your cluster. For example, you can use RBAC to allow developers to read metadata and logs from kubernetes pods but not make changes to them.

## RBAC Objects

**Roles** and **ClusterRoles** are kubernetes objects that define a set of permissions. These permissions determine what users can do in the cluster. A Role defines permissions within a particular namespace, and a ClusterRole defines cluster-wide permissions not specific to a single namespace.


**RoleBinding** and **ClusterRoleBinding** are objects that connect Roles and ClusterRoles to users.

![image](https://user-images.githubusercontent.com/113181949/229828229-9b8ed41c-41ba-4657-ba04-1e1c215291e8.png)

## Lesson Reference

Create a Role spec file.

```
vi role.yml
```
```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
namespace: default
name: pod-reader
rules:
- apiGroups: [""]
resources: ["pods", "pods/log"]
verbs: ["get", "watch", "list"]
```

Create the Role.

```
kubectl apply -f role.yml
```

Bind the role to the dev user.

```
vi rolebinding.yml
```
```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
name: pod-reader
namespace: default
subjects:
- kind: User
name: dev
apiGroup: rbac.authorization.k8s.io
roleRef:
kind: Role
name: pod-reader
apiGroup: rbac.authorization.k8s.io
Create the RoleBinding.
```
```
kubectl apply -f rolebinding.yml
```

Creating Service Accounts
===

## What is a Service Account
Is an account used by container processes within Pods to authenticate with the k8s API.
If your Pods need to communicate with the k8s API, you can use service accounts to control their access.

## Creating Service Accounts
A service account object can be created with some YAML just like any other k8s object.

## Binding Roles to Service Accounts
You can manage access control for service accounts, just like any other user, using RBAC objects. Bind service accounts with RoleBindings or ClusterRoleBindings to provide access to k8s API functionality.

Create a basic ServiceAccount.

```
vi my-serviceaccount.yml
```
```
apiVersion: v1
kind: ServiceAccount
metadata:
name: my-serviceaccount
```
```
kubectl create -f my-serviceaccount.yml
```

Create a ServiceAccount with an imperative command.

```
kubectl create sa my-serviceaccount2 -n default
```

View your ServiceAccount.

```
kubectl get sa
```

Attach a Role to the ServiceAccount with a RoleBinding.

```
vi sa-pod-reader.yml
```
```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
name: sa-pod-reader
namespace: default
subjects:
- kind: ServiceAccount
name: my-serviceaccount
namespace: default
roleRef:
kind: Role
name: pod-reader
apiGroup: rbac.authorization.k8s.io
```
```
kubectl create -f sa-pod-reader.yml
```

Get additional information for the ServiceAccount.

```
kubectl describe sa my-serviceaccount
```

Inspecting Pod Resource Usage
===

## Kubernetes Metrics Server

In order to view metrics about the resources pods and containers are using, we need an add-on to collect and provide that data. One such add-on is **Kubernetes Metrics Server**. This needs to be manually installed after a kubernetes cluster has been deployed.

## kubectl top
With kubectl top, you can view data about resource usage in your pods and nodes. kubectl top also support flags like --sort-by and --selector.

```
kubectl top pod --sort-by <JSONPATH> --selector <selector>
```
```
kubectl top node
```

## Installing Metrics Server and tesing top command

Install Kubernetes Metrics Server.
```
https://raw.githubusercontent.com/linuxacademy/content-cka-resources/master/metrics-server-components.yaml
```

Verify that the metrics server is responsive. Note that it may take a few minutes for the metrics server to become responsive to
requests.
```
kubectl get --raw /apis/metrics.k8s.io/
```

Create a pod to monitor.
```
vi pod.yml
```
```
apiVersion: v1
kind: Pod
metadata:
name: my-pod
labels:
 app: metrics-test
spec:
containers:
- name: busybox
image: radial/busyboxplus:curl
command: ['sh', '-c', 'while true; do sleep 3600; done']
```
```
kubectl apply -f pod.yml
```

Use  kubectl top  to view resource usage by pod.
```
kubectl top pod
```

Sort output with  --sort-by .
```
kubectl top pod --sort-by cpu
```
Filter output by label with  --selector .
```
kubectl top pod --selector app=metrics-test
```

Managing Application Configuration
===

## Application Configuration

When you are running applications in kubernetes, you may want to pass dynamic values to your applications at runtime to controll how they behave. This is known as **application configuration**
We will pass data to our containers that is going to control what those containers do and how they run.

## ConfigMaps
This is one of the primary ways to store configuration data in kubernetes.
You can store configuration data in kubernetes using Config Maps.
ConfigMaps store data in the form of a key-value map. ConfigMap data can be passed to your container applications.

![image](https://user-images.githubusercontent.com/113181949/230373131-cae47a00-4e6c-4b02-bf55-ae4220683e22.png)

IMPORTANT ITENS!!

**data section**
It is just a yaml data. It is a map of value key pairs

**multi-line data**
You can specify a configuration file to be applyied to one of your containers.


## Secrets
This is another way to store data in kubernetes. Secrets are similar to ConfigMaps but are designed to store sensitive data, such as passwords or API keys, more securely. They are created and used similarly to ConfigMaps.

![image](https://user-images.githubusercontent.com/113181949/230374033-fd18bfb8-c971-4fad-b0c2-bcdf4f52bee7.png)


## Environment Variables
You can pass ConfigMaps and Secret data to your containers as environment variables. These variables will be visable to your container process at runtime.

![image](https://user-images.githubusercontent.com/113181949/230376421-a93094a4-0ab4-42e2-95f0-ffa55be243cf.png)


## Configuration Volumes
Configuration data from ConfigMaps and Secrets can also be passed to containers in the form of **mounted volumes**. This will cause the configuration data to appear in files available to the container file system.
Each top-level key in the configuration data will appear as a file containing all keys below that top-lvel key.

![image](https://user-images.githubusercontent.com/113181949/230378303-d943637c-550d-4dcf-b1df-e1f18a1adda3.png)


## Lesson Reference

[Managing Application Configuration.pdf](https://github.com/rodolfoalmeid/Kubernetes-CKA-Study/files/11170643/1597437360824-devops-wb002.-.S05-L02.Managing.Application.Configuration.pdf)


Managing Containers Resources
===

## Resource Requests
It allow you to define an amount of resources (such CPU or memory) you expect a container to use. The Kubernetes scheduler will use resource requests to avoid scheduling pods on nodes that do not have nough available resources.

   > __Note__ 
   > Containers are allowed to use more (or less) than the requested resources. Resource requests only affect scheduling.

![image](https://user-images.githubusercontent.com/113181949/230402486-23120793-b732-49d4-8af6-3be54b381d46.png)

   > __Note__ 
   > Memory is measured in bytes and CPU is measured in CPU units.


## Resource Limits
Provide a way for you to limit the amount of resources your containers can use. The container runtime is responsible for enforcing these limits, and different container runtimes do this differently.

   > __Note__ 
   > Some runtimes will enforce these limits by terminating container process that attempt to use more than the allowed amount of resources.

![image](https://user-images.githubusercontent.com/113181949/230403521-30483b72-be09-40a5-ad3d-f3d6f119d93e.png)



Monitoring Container Health with Probes
===

## Container Health
k8s provides a number of features that allow you to build robust solutions, such as the ability to automatically restart unhealthy containers. To make the most of these features, k8s needs to be able to accurately determine the status of your applications. This means actively monitoring container health.

## Liveness Probes
Allow you to automatically determine wether or not a container application is in a healthy state. By default, k8s will only consider a container to be down if the container process stops. Liveness probes allow you rto customize this detection mechanism and make it more sphisticated.

![image](https://user-images.githubusercontent.com/113181949/230412110-4f02cf82-a046-4fd1-9047-bc0773649c99.png)


## Startup Probes
Are very similar to liveness probes. However, while liveness probes run contantly on a shcedule, startup probes run at container startup and stop running once they succeed. They are used to determine when the application has successfully started up. Startup probes are especially useful for legacy applications that can have long startup times.

![image](https://user-images.githubusercontent.com/113181949/230412952-892cdcb8-e56c-4322-b51a-d10b5e8a58bd.png)


## Readiness Probes
Are used to determine when a container is ready to accept requests. When you have a service backed by multiple container endpoints, user traffic will not be sent to a particular pod until its containers have all passed the readiness checks defined by their readiness probes. Use readiness probes to prevent user traffic from being sent to pods that are still in the process of starting up.

![image](https://user-images.githubusercontent.com/113181949/230413499-deb6f470-2887-4a93-88f5-50b6be1ce383.png)


## Lesson Reference

[Monitoring Container Health with Probes.pdf](https://github.com/rodolfoalmeid/Kubernetes-CKA-Study/files/11171607/1597437462380-devops-wb002.-.S05-L04.Monitoring.Container.Health.with.Probes.pdf)


Building Self-Healing Pods
===

## Restart Policies

K8s can automatically restart containers when they fail. **Restart Policies**allow you to customize this behavior by defining when you want a pod's containers to be automatically restarted. Restart policies are an important component of self-healing applications, which are automatically repaired when a problem araises. There are three possible values for a pod's restart policy in k8s: Always, OnFailure, and Never.

### Always
Always is the default restart policy in k8s. With this policy, containers will always be restarted if they stop, even if they completed successfully. Use this policy for applications that should always be running.

### OnFailure
The OnFailure restart policy will restart containers only if the container process exits with an error code or the container is determined to be unhealthy by a liveness probe. Use this policy for applications that need to run successfully and then stop.

### Never
The Never restart policy will cause the pod's containers to never be restarted, even if the containerexits or a liveness probe fails. Use this for applications that should run once and never be automatically restarted.

### Lesson Reference

Create a pod with the Always restart policy that completes after a few seconds.

```
vi always-pod.yml
```
```
apiVersion: v1
kind: Pod
metadata:
name: always-pod
spec:
restartPolicy: Always
containers:
- name: busybox
image: busybox
command: ['sh', '-c', 'sleep 10']
```
```
kubectl create -f always-pod.yml
```

Check the pod status. You should see it restarting after every 10-second completion.

```
kubectl get pod always-pod
```

Create a pod with the OnFailure restart policy.

```
vi onfailure-pod.yml
```
```
apiVersion: v1
kind: Pod
metadata:
name: onfailure-pod
spec:
restartPolicy: OnFailure
containers:
- name: busybox
image: busybox
command: ['sh', '-c', 'sleep 10']
```

```
kubectl create -f onfailure-pod.yml
```

Check the pod status. Note that the pod does not restart because it completed successfully.

```
kubectl get pod onfailure-pod
```

Delete, modify, and recreate the pod so that it fails.

```
vi onfailure-pod.yml
```
```
apiVersion: v1
kind: Pod
metadata:
name: onfailure-pod
spec:
restartPolicy: OnFailure
containers:
- name: busybox
image: busybox
command: ['sh', '-c', 'sleep 10; this is a bad command that will fail']
```
```
kubectl create -f onfailure-pod.yml
```

Check the pod status. Note that the pod restarts because it exited with an error code.

```
kubectl get pod onfailure-pod
```

Create a pod with the Never restart policy that completes after a few seconds.

```
vi never-pod.yml
```

```
apiVersion: v1
kind: Pod
metadata:
name: never-pod
spec:
restartPolicy: Never
containers:
- name: busybox
image: busybox
command: ['sh', '-c', 'sleep 10; this is a bad command that will fail']
```
```
kubectl create -f never-pod.yml
```

Check the pod status. You should see that it does not restart after completing.

```
kubectl get pod never-pod
```
