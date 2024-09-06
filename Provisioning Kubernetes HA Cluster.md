## Provisioning Kubernetes HA Cluster

Sebuah Cluster HA (High Availability) terdiri dari kumpulan (minimal 2) control plane (master nodes), worker nodes dan sebuah loadbalancer. Control Plane atau Master Nodes adalah inti dari sebuah Kubernetes cluster yang memanage semua operasi di cluster tersebut. Ada kalanya master nodes tersebut down dan jika kita menggunakan Single-Node Control Plane, sudah dipastikan cluster tersebut juga akan mati berserta semua aplikasi yang berjalan didalamnya atau worst-casenya data hilang. Maka dari itu sangatlah penting membuat infrastruktur kubernetes cluster yang menggunakan High Availability atau Multi-Node Control Plane untuk meminimalisir terjadinya downtime. 

## Infrastructure

Specs:

| Requirements  | Specs       |
| ------------- | ----------- |
| Load Balancer | 1 Nodes     |
| Control Plane | 2 Nodes     |
| Worker        | 2 Nodes     |
| RAM           | 4096Mi each |
| CPUs          | 2           |
| Storage       | 20G         |

Topology: 

![image-20240906113142948](https://github.com/user-attachments/assets/e6eb86e3-cc69-4914-9ad7-567ebc0dd04e)

## Setup Load Balancer

Load balancer disini diperlukan untuk memproses semua traffic masuk kedalam cluster. Dalam konteks untuk ini, saya menggunakan HAProxy sebagai loadbalancer untuk membagi traffic ke semua control planes. Dan jika salah satu control plane mati, aplikasi tetap bisa diakses karena HAProxy secara otomatis mengalihkan semua traffic ke available server.

### Step 1 - Update Repository & Install HAProxy 

Update packages repository untuk memastikan semua package dalam versi terbaru, kemudian install haproxy di **Node Load Balancer.**

```bash
$ sudo apt-get update && sudo apt-get install -y
```

Edit configuration file `/etc/haproxy/haproxy.cfg` dan sesuaikan seperti dibawah (letakan dibawah option defaults)

```bash
$ sudo nano /etc/haproxy/haproxy.cfg
---
defaults
        ...
frontend k8s-master-frontend
        bind :6443
        mode tcp
        option tcplog
        default_backend k8s-master-backend

backend k8s-master-backend
        mode tcp
        option tcp-check
        balance roundrobin
        default-server inter 10s downinter 5s
        server k8s-master-1 10.20.1.10:6443 check
        server k8s-master-2 10.20.1.11:6443 check
```

> Notes: Port `6443` merupakan port dari Kubernetes API Server. [Reference here](https://kubernetes.io/docs/concepts/security/controlling-access/)

Reload konfigurasi dengan merestart service HAProxy & enable haproxy startup at boot.

```bash
$ sudo systemctl restart haproxy && sudo systemctl enable haproxy
```

Referensi:

- https://facsiaginsa.com/kubernetes/haproxy-for-k8s-load-balancer

## Setup Kubernetes Cluster

Pertama pastikan seluruh Node sudah saling terhubung satu sama lain dan jika perlu bisa lakukan `ssh-copy-id` disetiap node untuk mempermudah akses antar vm

### Step 1 - Setup Containerd as CRI

Install containerd disemua master dan worker node sebagai CRI (Container Runtime Interface) mengingat docker sudah mendapat deprecation warning untuk versi 1.20 keatas.

#### Jalankan di semua Master dan Worker Node!!

Load module `br_netfilter` & `overlay` yang mana digunakan untuk networking nantinya.

```
# Load overlay & br_netfilter module into running kernel.
$ sudo modprobe overlay && sudo modprobe br_netfilter
```









Semua Referensi:

- https://medium.com/@sven_50828/setting-up-a-high-availability-kubernetes-cluster-with-multiple-masters-31eec45701a2
- https://overcast.blog/disaster-recovery-and-high-availability-in-kubernetes-a-guide-21996d80ac39
- https://avinetworks.com/glossary/kubernetes-load-balancer/
- https://facsiaginsa.com/kubernetes/haproxy-for-k8s-load-balancer
- https://kubernetes.io/docs/concepts/security/controlling-access/
- 

