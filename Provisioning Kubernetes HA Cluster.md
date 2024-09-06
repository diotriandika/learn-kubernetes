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

![image-20240906113142948](C:\Users\Dio Tri Andika\AppData\Roaming\Typora\typora-user-images\image-20240906113142948.png)

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

CRI diperlukan disetiap node dalam cluster sehingga Pod dama berjalan diatasnya. Install containerd disemua master dan worker node sebagai CRI (Container Runtime Interface) mengingat docker sudah mendapat deprecation warning untuk versi 1.20 keatas, [baca disini.](https://kubernetes.io/blog/2020/12/02/dont-panic-kubernetes-and-docker/)

#### Jalankan di semua Master dan Worker Node!!

Load module `br_netfilter` & `overlay` yang mana digunakan untuk networking nantinya.

```
# Load overlay & br_netfilter module into running kernel.
$ sudo modprobe overlay && sudo modprobe br_netfilter

# Load on boot modules for containerd
$ cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF
```

Konfigurasi `sysctl` yang diperlukan dan pastikan persisten setiap system reboot.

```bash
$ cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
```

Apply semua perubahan parameter `sysctl` tanpa reboot.

```bash
$ sudo sysctl --system
```

Selanjutnya install package-package yang diperlukan untuk menggunakan repository melaui HTTPS dan tambahkan apt-key serta repository yang dibutuhkan untuk instalasi containerd.

```bash
# Add Docker's offical GPG key: (the same repo as containerd used.)
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg apt-transport-https -y
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Update current repository & install containerd
$ sudo apt-get update & sudo apt-get install containerd.io
```

Buat configuration file default dari containerd

```bash
$ sudo mkdir -p /etc/containerd
$ sudo containerd config default | sudo tee /etc/containerd/config.toml
```

Selanjutnya set containerd configuration file diatas agar  `cgroupDriver` untuk runc diset ke systemd yang mana nantinya diperlukan oleh kubelet.

```bash
$ sudo sed -i '/SystemdCgroup/s/false/true/g' /etc/containerd/config.toml
```

Restart containerd dan cek apakah containerd sudah berjalan atau tidak serta enable startup to system boot.

```bash
# restart containerd service
$ sudo systemctl restart containerd

# check containerd status & enable automatic startup while system boot
$ sudo systemctl status containerd && sudo systemctl enable --now containerd
```

> Quick Info: CRI atau Container Runtime Interface adalah software yang bertugas untuk menjalankan container dikubernetes. Saat ini kubernetes mendukung beberapa CRI seperti containerd, CRI-O dan juga Docker. [Lengkapnya disini](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#cri-versions)

Referensi: 

- https://medium.com/@sven_50828/setting-up-a-high-availability-kubernetes-cluster-with-multiple-masters-31eec45701a2
- https://blog.assistanz.com/how-to-create-the-k8s-cluster-with-containerd/
- https://www.nocentino.com/posts/2021-12-27-installing-and-configuring-containerd-as-a-kubernetes-container-runtime/
- https://saurabhkharkate05.medium.com/kubernetes-cluster-setup-with-containerd-945214a0d02c

### Step 2 - Kubernetes Installation





Semua Referensi:

- https://medium.com/@sven_50828/setting-up-a-high-availability-kubernetes-cluster-with-multiple-masters-31eec45701a2
- https://overcast.blog/disaster-recovery-and-high-availability-in-kubernetes-a-guide-21996d80ac39
- https://avinetworks.com/glossary/kubernetes-load-balancer/
- https://facsiaginsa.com/kubernetes/haproxy-for-k8s-load-balancer
- https://kubernetes.io/docs/concepts/security/controlling-access/
- 

