### Overview 

Di projek kalii ini saya akan menjelaskan bagiamana cara membangun sebuah infrastructure cloud aws tanpa mengeluarkan uang karena full gratis!!,sebenarnya ini adalah projek
phase 2 dari projek sebelumnya yang [ini](https://rizkiyusupk.github.io/devops/clouds/linux/server/iac/infrastructure/aws1/) dan menggabungkan projek [KVM/QEMU PHASE 2](https://rizkiyusupk.github.io/devops/kubernetes/linux/server/iac/infrastructure/observ-tools/),
di projek ini saya akan melengkapi projek 
sebelumnya dengan infrastructure yang sebelumnya sudah saya bangun,dengan menambahkan tools seperti k8s,jenkins,dan observ tools ditambah alert lewat telegram,kurang lebih 
begitu langsung saja masuk ke pembahasanya


| Node        | CPU     | RAM  | Storage | Network                             |
|-------------|---------|------|---------|------------------------------------ |
| **Master**  | 2 cores | 2GB  | 10GB    | 1 Adapters   ( DHCP  Ip )           |
| **Worker 1**| 2 cores | 2GB  | 10GB    | 1 Adapters   ( DHCP  Ip )           |
| **Worker 2**| 2 cores | 2GB  | 10GB    | 1 Adapters   ( DHCP  Ip )           |
| **Jenkins** | 7 cores | 7GB  | 240GB   |                Wlan                 |


![sdub](/assets/images/aws2/workflow-aws-2.png)

### Tools
- **Wsl** : v0.2.1
- **Terraform** : v1.16
- **Localstack** : v2026.6.3
- **Docker** : v29.1.3
- **Python** : v3.12.3
- **Aws Cli** : v1.45.52
- **OS Laptop 1** : Windows 11 Pro
- **OS WSL2 Subsystem** : Ubuntu 24.04 LTS
- **OS Ubuntu VM (K8s Nodes)** : 22.04 LTS
- **OS Ubuntu Jenkins Node (Bare Metal)** : 25.04
- **Kubernetes** : 1.28
- **Containerd** : 2.2.4
- **Java** : 21
- **Jenkins** : 2.56
- **Network** : Flannel
- **Ansible** : v2.16+
- **Helm** : v3.21
- **Telegram Bot** :
- **KVM** : 8.2.2
- **QEMU** : 8.2.2
- **Libvirt** : 10.0.0
- **Alertmanager** : v0.34.0
- **Loki** : 3.6.11
- **Loki Canary** : 3.6.12
- **Loki Gateway (nginx)** : 1.29-alpine
- **Grafana** : 13.1.0
- **Prometheus Operator** : v0.93.1
- **Prometheus Config Reloader** : v0.93.1
- **Kube State Metrics** : v2.20.0
- **Prometheus** : v3.14.1
- **Node Exporter** : v1.12.0
- **Promtail** : 3.5.1
- **Ngrok**
- **Gitlab** : SaaS

  
### Reasoning
Kenapa saya memilih untuk membangun phase 2? karena saya penasaran apakah bisa menintegrasikan aws environtment dengan tools tools stand alone dan ini juga sebagai bentuk 
pembelajaran bagi saya agar saya familiar dengan service yang ada di aws,dan kenepaa saya memilih untuk menggunakan localstack alasan utamanya  karena gratis tapi perlu
melakukan login user account terlebih dahulu agar mendapatkan auth token

### STRUCTURE FOLDER 

Untuk structure folder yang digunakan dalam projek ini ada tiga yang pertama itu untuk terraform dan yang kedua itu ansible,terakhir itu ada di cluster,

```
terraform-setup/
├── compute.tf
├── main.tf
├── prep-vm.tf
├── s3.tf
├── sns.tf
├── sqs.tf
├── sqs-trigger-lambda.tf
├── iam-attachment-role.tf
├── iam-attachment-role-consumer.tf
├── lambda_function_consumer.py
├── lambda_function.py
├── lambda_function_alert.py
├── lambda-permission.tf
├── lambda.tf
├── lambda-2.tf
├── lambda-alert.tf
├── cloud-watch.tf
├── cloud-watch-metrics.tf
├── dynamodb.tf
```


lalu yang kedua

```
k8s/
├── ansible.cfg
├── inventory
├── playbook-allow-port-native.yaml
├── playbook-ip-native.yaml
├── playbook-install-java-baremetal-native.yaml
├── playbook-install-jenkins-baremetal-native.yaml
├── playbook-install-kubectl-baremetal-native.yaml
├── playbook-join-native.yaml
├── playbook-kubernetes-native.yaml
├── playbook-pkg-native.yaml
├── playbook-swap-native.yaml
├── playbook-config-native.yaml
├── playbook-install-terraform-baremetal-native.yaml
├── -native.yaml
├── _2-native.yaml
```

dan yang terakhir yang ketiga di cluster 

```
cluster-side/
├── alertmanager-telegram.yaml
```

### Catatan Kompatibilitas
Dites pada: VM Ubuntu 22.04 (jammy, dari image cloud) dan node Jenkins Ubuntu 25.04.
Kalau memakai rilis Ubuntu lain, alur playbook dan module-nya sama. Yang perlu
disesuaikan hanya yang bergantung pada codename rilis:
- HashiCorp: ubah `hashicorp_suite` bila suite rilis belum ada di repo mereka.
- Docker: pastikan `download.docker.com` sudah punya suite untuk rilis tersebut.
- Terraform: ganti URL image di `prep-vm.tf`.

### prerequisites 
Oke sebelum masuk ke setup saya harapkan anda semua membaca ini,semua kondisi harus memenuhi 

 - **kubernetes.core**
 - **community.general**
 - **ansible.posix**

collection yang dibutuhkan tapi jika install ansible lewat pip ansible berarti semuanya 
