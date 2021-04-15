# ILKE Parameters

Below  you can find all the parameters you can use in this file, section by section.

## Global Section

This section is used to custom global ILKE settings.

| Parameter | Description | Values |
| --- | --- | --- |
| `ilke.global.data_path` | Path where ILKE saves all config/pki/service files on deploy machine | **/var/ilke/** *(default)* |

## Certificates & PKI section

This section is used to custom the PKI used for your deployment and manage Certificates lifecycle.

| Parameter | Description | Values |
| --- | --- | --- |
| `ilke_pki.infos.state` | State or province name added to PKI CSR | **Ile-De-France** *(default)* |
| `ilke_pki.infos.locality` | Locality added to PKI CSR | **Paris** *(default)* |
| `ilke_pki.infos.country` | Country added to PKI CSR | **FR** *(default)* |
| `ilke_pki.infos.root_cn` | CommonName used for Root CA | **ILKI Kubernetes Engine** *(default)* |
| `ilke_pki.infos.expirity` | Expirity for all PKI certificats | **+3650d** (default - 10 years)|
| `ilke_pki.management.rotate_certificats` | Boolean used to rotate certificates | **False** (default)|

## Main K8S Components Section

This section is used to custom K8S main components that will be deployed.

### ETCD

This section allows you to configure your ETCD deployment.

| Parameter | Description | Values |
| --- | --- | --- |
| `ilke_base_components.etcd.release` | ETCD release that will be installed on etcd hosts | **v3.4.14** *(default)* |
| `ilke_base_components.etcd.upgrade` | Upgrade current ETCD release to `ilke_base_components.etcd.release` | **False** *(default)* |
| `ilke_base_components.etcd.check` | Check ETCD cluster Status/Size/Health/Leader when running ilke run | **True** *(default)* |
| `ilke_base_components.etcd.data_path` | Path where ETCD save data on ETCD hosts | **/var/lib/etcd** *(default)* |
| `ilke_base_components.etcd.backup.enabled` | Enable etcd backup Pod | **False** *(default)* |
| `ilke_base_components.etcd.backup.crontab` | CronTab used to run ETCD Backup | **"*/30 * * * *"** *(default)* |
| `ilke_base_components.etcd.backup.storage.enabled` | Enable persistent Storage for ETCD Backups | **False** *(default)* |
| `ilke_base_components.etcd.backup.storage.capacity` | Storage Size used to store ETCD Backups | **10Gi** *(default)* |
| `ilke_base_components.etcd.backup.storage.type` | Type of Storage to use when `ilke_base_components.etcd.backup.storage.enabled` is set to **True** | **hostpath** *(default)*, storageclass, persistentvolume |
| `ilke_base_components.etcd.backup.storage.storageclass.name` | StorageClass name used to store ETCD Backups. Used only if `ilke_base_components.etcd.backup.storage.type` is set to **storageclass** | **default-jiva** *(default)* |
| `ilke_base_components.etcd.backup.storage.persistentvolume.name` | PersistentVolume name used to store ETCD Backups. Used only if `ilke_base_components.etcd.backup.storage.type` is set to **persistentvolume** | **my-pv-backup-etcd** *(default)* |
| `ilke_base_components.etcd.backup.storage.persistentvolume.storageclass` | StorageClass name used to create persistentvolume set in `ilke_base_components.etcd.backup.storage.persistentvolume.name`. Used only if `ike_base_components.etcd.backup.storage.type` is set to **persistentvolume** | **/var/lib/etcd** *(default)* |
| `ilke_base_components.etcd.backup.storage.hostpath.nodename` | K8S node (master/worker/storage) where backups are stored locally. Used only if `ilke_base_components.etcd.backup.storage.type` is set to **hostpath** | **master1** *(default)* |
| `ilke_base_components.etcd.backup.storage.hostpath.path` | Path on `ilke_base_components.etcd.backup.storage.hostpath.nodename` where ETCD backups are stored | **/var/etcd-backup** *(default)* |


### Kubernetes

This section allows you to configure your Kubernetes deployment.

| Parameter | Description | Values |
| --- | --- | --- |
| `ilke_base_components.kubernetes.release` | Kubernetes release that will be installed on *Master/Worker/Storage* hosts |  **v1.20.4** *(default)* |
| `ilke_base_components.kubernetes.upgrade` | Upgrade current Kubernetes release to `ilke_base_components.kubernetes.release` | **False** *(default)* |

### Container Engine

This section allows you to configure your Container Engine that will be deployed on all Master/Worker/Storage hosts.

| Parameter | Description | Values |
| --- | --- | --- |
| `ilke_base_components.container.engine`  | Container Engine to install (Containerd or Docker) on all Master/Worker/Storage hosts |  **containerd** *(default)*, or docker |
| `ilke_base_components.container.release` | Release of Container Engine to install - Supported only if `ilke_base_components.container.engine` set to *docker*  | If **""** install latest release *(default)* |
| `ilke_base_components.container.upgrade` | Upgrade current Container Engine release to `ilke_base_components.container.release` | **Will be available soon** (No effect) |

## Network Settings

This section allows you to configure your K8S cluster network settings.

| Parameter | Description | Values |
| --- | --- | --- |
| `ilke_network.cni_plugin` | CNI plugin used to enable K8S hosts Networking | **calico** *(default)*, kube-router |
| `ilke_network.mtu` | MTU for CNI plugin. Auto-MTU if set to **0**. Only used if `ilke_network.cni_plugin` is set to **calico** | **0** *(default)* |
| `ilke_network.cidr.pod` | PODs CIDR network | **10.33.0.0/16** *(default)* |
| `ilke_network.cidr.service` | Service CIDR network | **10.32.0.0/24** *(default)* |
| `ilke_network.service_ip.kubernetes` | ClusterIP of *default.kubernetes* service. Should be the first IP available in `ilke_network.cidr.service` | **10.32.0.1** *(default)* |
| `ilke_network.service_ip.coredns` | ClusterIP of *kube-system.kube-dns* service. | **10.32.0.10** *(default)* |
| `ilke_network.nodeport.range` | Range of allowed ports usable by NodePort services | **30000-32000** *(default)* |
| `ilke_network.external_loadbalancing.enabled` | Enable External LoadBalancing in ARP mode. Working only if On-Prem deployments | **False** *(default)* |
| `ilke_network.external_loadbalancing.ip_range` | IPs Range, or CIDR used by External LoadBalancer to assign External IPs  | **10.10.20.50-10.10.20.250** *(default range)* |
| `ilke_network.external_loadbalancing.secret_key` | Security Key : Generate a custom key with : `openssl rand -base64 128` | **a default insecure key** *(Change it !)* |
| `ilke_network.kube_proxy.mode` | Kube-Proxy mode. iptables/ipvs. IPVS > IPTABLES | **ipvs** *(default)* |
| `ilke_network.kube_proxy.algorithm` | Default ClusterIP loadBalancing Algorithm : rr,lc,dh,sh,sed,nq. Only supported if IPVS | **rr** *(default Round-Robin)* |


## ILKE features

This section allows you to configure your K8S features.

| Parameter | Description | Values |
| --- | --- | --- |
| `ilke_features.storage.enabled` | Enable Storage feature - OpenEBS based | **False** *(default)* |
| `ilke_features.storage.release` | OpenEBS release to be installed | **2.5.0** *(default)* |
| `ilke_features.storage.jiva.data_path` | Path where OpenEBS store Jiva volumes on Storage Nodes | **/var/openebse** *(default)* |
| `ilke_features.storage.jiva.fs_type` | Jiva FS types | **ext4** *(default)* |
| `ilke_features.storage.hostpath.data_path` | Path where OpenEBS store HostPath volumes on Pod node | **False** *(default)* |
| `ilke_features.dashboard.enabled` | Enable Kubernetes dashboard | **False** *(default)* |
| `ilke_features.dashboard.generate_admin_token` | Generate a default admin user + save token to /root/.kube/dashboardamin on Deploy node | **False** *(default)* |
| `ilke_features.metrics_server.enabled` | Enable Metrics-Server | **False** *(default)* |
| `ilke_features.ingress.controller` | Ingress Controller to install : nginx, ha-proxy, traefik | **nginx** *(default)* |
| `ilke_features.ingress.release` | Ingress controller release to install. Only used if `ilke_features.ingress.controller` set to "nginx" | **False** *(default)* |
| `ilke_features.monitoring.enabled` | Enable Monitoring | **False** *(default)* |
| `ilke_features.monitoring.persistent` | Persist Monitoring Data | **False** *(default)* |
| `ilke_features.monitoring.admin.user` | Default Grafana admin user | **administrator** *(default)* |
| `ilke_features.monitoring.admin.password` | Default grafana admin password | **P@ssw0rd** *(default)* |

## ILKE other settings
This section allows you to configure some other settings

| Parameter | Description | Values |
| --- | --- | --- |
| `ilke_populate_etc_hosts` | Add to all hostname/IPs of ILKE Cluster to /etc/hosts file of all hosts. Use it only if you don't have DNS server. | **False** *(default)* |
| `ilke_encrypt_etcd_keys` | Array of keys/algorith used to crypt/decrypt data in etcd? Generate with : `head -c 32 /dev/urandom | base64` | **changeME !** *(default)* |
| `restoration_snapshot_file` | ETCD backup path to be restored | **none** *(default)* |

# Kubernetes deployment

Once all configuration files are set, run the following command to launch the Ansible playbook that will deploy the pre-configured Kubernetes cluster :

```
sudo ansible-playbook ilke.yaml
```
