# Table of Contents
This is a list of points that will be explained in this instructions file for the ILKE project :

- [High-level Architecture](#high-level-architecture)
- [Prerequisites](#prerequisites)
- [Nodes Setup](#nodes-setup)
- [K8S Cluster Configuration](#k8s-cluster-configuration)
- [ILKE Parameters](#ilke-parameters)
- [Kubernetes deployment](#kubernetes-deployment)
- [Manage ETCD Cluster](./manage_etcd.md)
- [Create Pod](#create-pod)
- [Storage Benchmark](#storage-benchmark)
- [Uninstall ILKE](#uninstall-ilke)


# High-level Architecture

Below a diagram of the high-level architecture deployed by ILKE :
![Architecture](../images/ILKE_diagram.png)

**Notes :** This distibution is aimed to be customizable so you can choose : 
 - Where the **etcd** will be deployed (with the master or not) 
 - The number of **master** nodes to deploy (from 1 to many - 5 nodes for production)
 - The number of **etcd** nodes to deploy (from 1 to many - 5 nodes for production)
 - The number of **worker** nodes to deploy (from 1 to many)
 - The number of **storage** nodes to deploy (from 0 to many - 3 nodes for production needs)
 
 # Prerequisites

This section explains what are the prerequisites to install ILKE in your environment.

## OS

Below the OS currently supported on all the machines :
  - Ubuntu 18.04 & 20.04 - amd64
  - Centos 7 - amd64
  - Debian 10 - amd64

## Network

- Full network connectivity between all machines in the cluster (public or private network is fine)
- Full internet access
- Unique hostname, MAC address, and product_uuid for every node. See here for more [details](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#verify-the-mac-address-and-product-uuid-are-unique-for-every-node).
- Certain ports are open on your machines. See here for more [details](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#check-required-ports).

## Node Sizing

Node sizing indicated here is for production environment. You can custom it according to sweet your needs.

It is a best-practice to install ETCD and MASTERS on separate hosts.

| ILKE Type | no HA or all-in-one | no-production | production |
| --- | --- | --- | --- |
| MASTER | 1 | 3 | 3+ |
| ETCD | 1 | 3 | 5 |
| WORKER | 1 | X | X |
| STORAGE | 0 - 1 | 3 | 3+ |

We actually configure the proper VM size for your MASTER depending on the number of nodes (Workers + Storage) in your cluster

| nodes | Master Size |
| --- | --- |
| 1-5 | 1 CPU - 3,75 Go RAM |
| 6-10 | 2 CPU - 7,50 Go RAM |
| 11-100 | 4 CPU - 15 Go RAM |
| 101-250 | 8 CPU - 30 Go RAM |
| 251-500 | 16 CPU - 60 Go RAM |
| more than 500 | 32 CPU - 120 Go RAM |

We actually configure the proper VM size for your ETCD depending on the number of nodes (Workers + Storage + Masters) in your cluster

| nodes | ETCD Size | Notes |
| --- | --- | --- |
| 0-50 | 2 CPU - 8 Go RAM | A small cluster serves fewer than 100 clients, fewer than 200 of requests per second, and stores no more than 100MB of data |
| 50-250 | 4 CPU - 16 Go RAM | A medium cluster serves fewer than 500 clients, fewer than 1,000 of requests per second, and stores no more than 500MB of data |
| 250-1000 | 8 CPU - 32 Go RAM | A large cluster serves fewer than 1,500 clients, fewer than 10,000 of requests per second, and stores no more than 1GB of data |
| 1000-3000 | 16 CPU - 64 Go RAM | An xLarge cluster serves more than 1,500 clients, more than 10,000 of requests per second, and stores more than 1GB data |

# Nodes Setup

This section explains how to setup nodes before deploying Kubernetes Clusters with ILKE.

## Deployment node

The deployment node is an Ansible server which contains all Ansible roles and variables used to deploy and configure Kubernetes Clusters with ILKE distribution.

The prerequisites are:
- SSH Server (like openssh-server)
- Python3 & pip3
- git
- curl
- with pip3 : ansible, netaddr

Then clone or download the ILKE git branch / release you want to use.

You can run the following command to automatically install those packages and clone the latest stable ILKE distribution:
```
bash <(curl -s https://raw.githubusercontent.com/ilkilabs/ilke/master/setup-deploy.sh)
```

### Use Python Virtual Environment

Sometimes it is better to run Ansible and all its dependences into a specific *Python Virtual Environment*. This will make it easier for you to install Ansible and all its dependences needed by ILKE without take the risk to break your existing Python/Python3 installation.


You can create your own *Python Virtual Environment* from scratch by following:

```
# Install on deploy machine python3, pyhton3-pip and python3-venv
# On Ubuntu (18.04,20.04) or Debian10 use the following commands:
apt update
apt install -yqq python3 python3-pip python3-venv

# Only on Centos7
yum install -y libselinux-python3

# Create a Python Virtual Environment
python3 -m venv /usr/local/ilke-env

# Tell to your shell to use this Python Virtual Environment
source /usr/local/ilke-env/bin/activate

# Update PIP
pip3 install --upgrade pip

# Then install Ansible and Netaddr (needed by ILKE)
pip3 install ansible
pip3 install netaddr
pip3 install selinux

# You can alternatively install packages with "ilke/requirements.txt" file located on ILKE
pip3 install -r ilke/requirements.txt

# Validate ansible is installed and use your Python Virtual Environment
ansible --version

#ansible 2.10.5
#  config file = None
#  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
#  ansible python module location = /usr/local/ilke-env/lib/python3.8/site-packages/ansible
#  executable location = /usr/local/ilke-env/bin/ansible
#  python version = 3.8.5 (default, Jul 28 2020, 12:59:40) [GCC 9.3.0]


# If you whant to stop using the Python Virtual Environment, just execute the following command:
deactivate
```


## K8S nodes

The K8S nodes will host all the components needed for a Kubernetes cluster Control and Data planes.

The prerequisites are:
- SSH Server (lilke openssh-server)
- Python3
- curl

You can run the following command to automatically install those packages :
```
bash <(curl -s https://raw.githubusercontent.com/ilkilabs/ilke/master/setup-hosts.sh)
```

## SSH keys creation

ILKE is using Ansible to deploy Kubernetes. You have to configure SSH keys to ensure the communication between the deploy machine and the others.

On the deploy machine, create the SSH keys :
```
ssh-keygen
```
You can let everything by default.

When your keys are created, you have to copy the public key in the other machine in the folder /home/yourUser/.ssh/authorized_keys, or you can use the following commands to copy the key :
```
ssh-copy-id -i .ssh/id_rsa.pub yourUser@IP_OF_THE_HOST
```
You have to execute this command for each node of your cluster

Once your ssh keys have been pushed to all nodes, modify the file "ilke/hosts" to add the user/ssh-key (in section **SSH Connection settings**) that ILKE will use to connect to all nodes

# K8S Cluster Configuration

ILKE enables an easy way to deploy and manage customizable K8S clusters.

## ansible.cfg file

This file alows you to configure default settings for your Ansible server.

**If you are using CentOS-7, make sure to set "interpreter_python = /usr/bin/python2.7" !!** Ansible on CentOS-7 don't fully support Python3. 



## Inventory file

The first file to modify is ["./hosts"](../hosts). This file contains all architecture information about your K8S Cluster.

**All K8S servers names must be filled in by their FQDN**. You can run ```hostname -f``` on your hosts to get it.

The next Sample deploys K8S components in HA mode on 6 nodes (3 **etcd/masters** nodes, 3 **workers** nodes) :

```
[deploy]
deploy ansible_connection=local ansible_python_interpreter=/usr/bin/python3

[masters]
master1  ansible_host=10.10.20.4

[etcd]
master1  ansible_host=10.10.20.4

[workers]
worker2  ansible_host=10.10.20.5
worker3  ansible_host=10.10.20.6

[storage]
worker4 ansible_host=10.10.20.20

[all:vars]
advertise_masters=10.10.20.4
#advertise_masters=kubernetes.localcluster.lan

# SSH connection settings
ansible_ssh_extra_args=-o StrictHostKeyChecking=no
ansible_user=vagrant
ansible_ssh_private_key_file=/home/vagrant/ssh-private-key.pem

# Python version

# If centOS-7, use python2.7
# If no-CentOS-7, like ubuntu (18.04, 20.04) or If Debian 10, use Python3
ansible_python_interpreter=/usr/bin/python3


[etc_hosts]
#kubernetes.localcluster.lan ansible_host=10.10.20.4
```

The **deploy** section contains information about how to connect to the deployment machine.

The **etcd** section contains information about the etcd machine(s) instances.

The **masters** section contains information about the masters nodes (K8S Control Plane).

The **workers** section contains information about the workers nodes (K8S Data Plane).

The **storage** section contains information about the storage nodes (K8S Storage Plane ).

The **etc_hosts** section contains a list of DNS entries that will be injected to /etc/hosts files of all hosts. Use it only if you don't have DNS server.

The **all:vars** section contains information about how to connect to K8S nodes.

The **advertise_masters** parameter configure the Advertising IP of control Plan. Actually it is the IP of a frontal LB that expose Master nodes on port TCP/6643. It can also be a Master's IP if you don't have LB. In this case, HA is not enabled even if you got multiple Masters...

The **SSH Connection settings** section contain information about the SSH connexion. You have to modify the variable **ansible_ssh_private_key_file** with the path where your public key is stored.
**ansible_user** User used as service account by ILKE to connect to all nodes. **User must be sudoer**.

## Configuration file

The [../group_vars/all.yaml](../group_vars/all.yaml) file contains all configuration variables that you can customize to make your K8S Cluster fit your needs.

Sample file will deploy **containerd** as container runtime, **calico** as CNI plugin and enable all ILKE features (storage, dashboard, monitoring, LB, ingress, ....).

```
---
ilke:
  global:
    data_path: /var/ilke

ilke_pki:
  infos:
    state: "Ile-De-France"
    locality: "Paris"
    country: "FR"
    root_cn: "ILKI Kubernetes Engine"
    expirity: "+3650d"
  management:
    rotate_certificats: False

ilke_base_components:
  etcd:
    release: v3.4.14
    upgrade: False
    check: true
    data_path: /var/lib/etcd
    backup:
      enabled: False
      crontab: "*/30 * * * *"
      storage:
        capacity: 10Gi
        enabled: False
        type: "storageclass"
        storageclass:
          name: "default-jiva"
        persistentvolume:
          name: "my-pv-backup-etcd"
          storageclass: "my-storageclass-name"
        hostpath:
          nodename: "master1"
          path: /var/etcd-backup
  kubernetes:
    release: v1.20.2
    upgrade: false
  container:
    engine: containerd
# release : Only Supported if container engine is set to docker
    release: ""
#    upgrade: false

ilke_network:
  cni_plugin: calico
  mtu: 0
  cidr:
    pod: 10.33.0.0/16
    service: 10.32.0.0/24
  service_ip:
    kubernetes: 10.32.0.1 
    coredns: 10.32.0.10
  nodeport:
    range: 30000-32000
  external_loadbalancing:
    enabled: False
    ip_range: 10.10.20.50-10.10.20.250
    secret_key: LGyt2l9XftOxEUIeFf2w0eCM7KjyQdkHform0gldYBKMORWkfQIsfXW0sQlo1VjJBB17shY5RtLg0klDNqNq4PAhNaub+olSka61LxV73KN2VaJY/snrZmHbdf/a7DfdzaeQ5pzP6D5O7zbUZwfb5ASOhNrG8aDMY3rkf4ZzHkc=
  kube_proxy:
    mode: ipvs
    algorithm: rr

ilke_features:
  coredns:
    release: "1.8.0"
    replicas: 2
  storage:
    enabled: false
    release: "2.6.0"
    jiva:
      data_path: /var/openebs
      fs_type: ext4
    hostpath:
      data_path: /var/local-hostpath
  dashboard:
    enabled: false
    generate_admin_token: false
    release: v2.1.0
  metrics_server:
    enabled: false
  ingress:
    controller: nginx
    release: v0.44.0
  monitoring:
    enabled: false
    persistent: false
    admin:
      user: administrator
      password: P@ssw0rd

ilke_populate_etc_hosts: false

# Security
ilke_encrypt_etcd_keys:
# Warrning: If multiple keys are defined ONLY LAST KEY is used for encrypt and decrypt.
# Other keys are used only for decrypt purpose. Keys can be generated with command: head -c 32 /dev/urandom | base64
  key1:
    secret: 1fJcKt6vBxMt+AkBanoaxFF2O6ytHIkETNgQWv4b/+Q=

#restoration_snapshot_file: /path/snopshot/file Located on {{ etcd_data_directory }}

```

**Note :** You can also modify the IPs-CIDR if you want.

# Kubernetes deployment

Once all configuration files are set, run the following command to launch the Ansible playbook that will deploy the pre-configured Kubernetes cluster :

```
sudo ansible-playbook agorakube.yaml
```

