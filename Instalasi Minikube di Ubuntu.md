## Instalasi Minikube di Ubuntu (KVM/VirtualBox)

### Step 1 - Update Packages & Download Minikube

Update semua package repository agar saat instalasi semua package dalam versi terbaru.

```bash
$ sudo apt-get update
```

Donwload file binary minikube dan letakan di `/usr/loca/bin`

```bash
# Download Minikube Binary File using wget
$ wget https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64

# Make the binary file executable.
$ chmod +x minikube-linux-amd64

# Move file and save as minikube
$ sudo mv minikube-linux-amd64 /usr/local/bin/minikube
```

Verifikasi instalasi minikube dengan menjalankan

```bash
$ minikube version
minikube version: v1.33.1
commit: 5883c09216182566a63dff4c326a6fc9ed2982ff
```

### Step 2 - Kubectl Installation

Download kubectl, kubectl digunakan untuk berinteraksi dengan cluster kubernetes. 

```bash
$ curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
```

Buat binary file kubectl menjadi executable

```bash
$ chmod +x ./kubectl
```

Pindahkan binary file kubectl ke PATH

```bash
$ sudo mv ./kubectl /usr/local/bin/kubectl
```

Verifikasi dengan menjalankan

```bash
$ kubectl version -o json --client
```

### Step 3 - Creating Cluster & Set Default Driver

Secara default minikube menggunakan docker sebagai driver, dan disini saya ingin menggunakan kvm sebagai driver. 

```bash
# Running Kubernetes Cluster with KVM Drivers
$ minikube start --driver=kvm2 --memory=2000m --cpus=2 --disk-size="15000mb" -p kvm-cluster

# Running Kubernetes Cluster with VirtualBox Drivers
$ minikube start --driver=virtualbox --memory=2000m --cpus=2 --disk-size="15000mb" -p virtualbox-cluster
```

> Note : sesuaikan dengan resource yang dimiliki. Jika tidak ingin menggunakan vtx, tambahkan command `--no-vtx-check` diakhir
>
> `-p kvm-cluster` merupakan nama cluster yang akan dibuat di minikube.

#### Optional

Jika ingin menggunakan driver kvm2 atau lainnya secara default, jalankan command dibawah

```bash
$ minikube config set driver kvm
```

Verifikasi dengan menjalankan 

```bash
$ kubectl get nodes
NAME       STATUS   ROLES           AGE    VERSION
minikube   Ready    control-plane   105s   v1.30.0

$ minikube status
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

Akses minikube VM menggunakan ssh

```bash
$ minikube ssh
                         _             _            
            _         _ ( )           ( )           
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __  
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ sudo su -
# 
```

Untuk managemen cluster jalankan

```bash
# Stop Cluster
$ minikube stop -p <cluster-name>

# Pause Cluster
$ minikube pause -p <cluster-name>

# Delete Cluster
$ minikube delete -p <cluster-name>

# View All Profile
$ minikube profile list

# Access Kubernetes Dashboard
$ minikube dashboard --url
```



Referensi :

- https://computingforgeeks.com/how-to-install-minikube-on-ubuntu/
- https://minikube.sigs.k8s.io/docs/drivers/
