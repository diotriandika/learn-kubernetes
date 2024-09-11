## Apa itu MetalLB?

![metal-lb use cases](https://kubernetes.github.io/ingress-nginx/images/baremetal/metallb.jpg)

MetalLB adalah sebuah implementasi dari LoadBalancer untuk cluster Kubernetes yang berjalan di infrastruktur Bare-Metal atau Virtualized. Biasanya, service LoadBalancer di Kuberentes secara default hanya tersedia dalam lingkungan Public Cloud yang mendukung integrasi khusus untuk memprovisioning load balancer eksternal. Namun, ketika menggunakan di lingkungan Bare-Metal atau virtualized, tidak ada layanan LoadBalancer yang otomatis tersedia. Inilah mengapa MetalLB hadir sebagai solusi alternatif yang memungkinkan kita untuk menggunakan IP LoadBalancer dilingkungan infrastruktur Bare-Metal atau Virtualized. Dengan MetalLB kita bisa mengkonfigurasi IP Address LoadBalancer dan mengarahkannya ke backend Pod yang diinginkan. 

MetalLB bekerja dengan melibatkan penggunakan protocol ARP (Address Resouliton Protocol) atau BGP (Border Gateway Procotol) untuk mengiklankan alamat IP LoadBalancer yang tersedia ke network kita. Ketika request masu ke alamat IP LoadBalancer, MetalLB akan menerukan request tersebut ke backend Pod yang sesuai dengan metode load-balancing yang sudah dikonfigurasi. Dengan MetalLB kita bisa mengatur rentang IP Address yang tersedia, metode load balancing yang akan digunakan dll. 

## MetalLB Installation

Seperti plugin pada umumnya, MetalLB dapat diinstall dengan berbagai cara seperti menggunakan manifest, Kustomize dan juga Helm, namun disini saya akan menggunakan metode instalasi dengan manifest. Pastikan juga untuk memenuhi semua [requirements](https://metallb.universe.tf/#requirements:~:text=much%20as%20possible.-,Requirements,-MetalLB%20requires%20the) yang ada.

### Step 1 - Preparation

Enable strictARP Mode, karena sejak Kubernetes v1.14.2 kita harus mengaktifkan mode tersebut secara manual jika menggunakan kube-proxy di IPVS mode.

```bash
kubectl edit configmap -n kube-system kube-proxy
```

### Step 2 - MetalLB Installation

Apply manifest untuk instalasi MetalLB

```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.8/config/manifests/metallb-native.yaml
```

Atau jika ingin mendeploy MetalLB dengan menggunakan [FRR Mode](https://metallb.io/configuration/#enabling-bfd-support-for-bgp-sessions)

```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.8/config/manifests/metallb-frr.yaml
```

Manifest ini akan mendeploy MetalLB ke dalam cluster Kubernetes dibawah namespace `metallb-system`. Didalam manifest tersebut terdapat dua komponen penting yakni:

- `metallb-system/controller` deployment. Ini adalah sebuah cluster-wide controller yang menghandle pengalamatan IP.
- `metallb-system/speaker` daemonses. Ini adalah sebuah komponen yang berbicara dengan protocol yang dipilih untuk dapat menjangkau services.
- Service accounts untuk controller dan speaker, bersama dengan RBAC Permissions yang komponen2 butuhkan untuk dapat berfungsi.

Instalasi diatas tidak termasuk dengan file konfigurasi MetalLB, maka dari itu MetalLB akan dalam state Idle setelah start.

## How to use MetalLB

![metallb](https://access.redhat.com/webassets/avalon/d/OpenShift_Container_Platform-4.9-Networking-en-US/images/41d8213d50dd7449fd60c44c47ffd01f/nw-metallb-layer2.png)

Sebelum itu kita harus medefinisikan dulu IP yang akan diassign oleh LoadBalancer ke service. Setelah itu, IP tersebut juga harus diannounced. Disini saya menggunakan protocol L2 untuk mengannounce IP service.

```bash
$ vim metallb-ip_pool.yaml
---
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: local-pool
  namespace: metallb-system
spec:
  addresses:
  - 10.20.1.50 - 10.20.1.100
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: first-pool-l2
  namespace: metallb-system
spec:
  ipAddressPools:
  - local-pool
```

Apply ip-pool tersebut

```bash
kubectl apply -f metallb-ip_pool.yaml
```

IP Pool diatas adalah dalam subnet yang sama dengan Node Cluster yang saya miliki. kita bisa lihat IP pool yang dibuat dengan 

```bash
kubectl get ipaddresspools.metallb.io -n metallb-system
NAME           AUTO ASSIGN   AVOID BUGGY IPS   ADDRESSES
local-pool   true          false             ["10.20.1.50 - 10.20.1.100"]
```

Untuk pengetesan, disini saya akan membuat sebuah deployment nginx dengan type service LoadBalancer.

```bash
$ vim nginx-with-loadbalancer.yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 3
  template:
    metadata:
      labels:
        app:  nginx
    spec:
      containers:
      - name:  nginx
        image:  nginx
        ports:
        - containerPort:  80
          name:  nginx
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  type: LoadBalancer
  ports:
  - name: nginx-service
    port: 80
    targetPort: nginx
```

lalu deploy

```bash
kubectl apply -f nginx-with-loadbalancer.yaml
```

```bash
$ kubectl get pod 
NAME                     READY   STATUS    RESTARTS   AGE
nginx-696ff7b4cd-lsbjn   1/1     Running   0          143m
nginx-696ff7b4cd-s6zvb   1/1     Running   0          143m
nginx-696ff7b4cd-tx4x5   1/1     Running   0          143m
$ kubectl get svc
NAME            TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes      ClusterIP      10.96.0.1        <none>        443/TCP        3d21h
nginx-service   LoadBalancer   10.101.200.133   10.20.1.51    80:30974/TCP   143m
```

Diatas kita bisa lihat bahwa service `nginx-service` kini memiliki sebuah External IP yang dimana diassign oleh MetalLB. Kita bisa melakukan curl melalui semua node yang satu network dengan ip tersebut

```bash
lnearher@other-node:~$ curl http://10.20.1.51
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

Referensi:

- https://medium.com/@maulanamalikjb147/metallb-kubernetes-990b3e5227d1
- https://metallb.universe.tf/installation/
- https://metallb.universe.tf/concepts/bgp/
- https://metallb.universe.tf/configuration/
- https://docs.redhat.com/en/documentation/openshift_container_platform/4.10/html/networking/load-balancing-with-metallb#about-metallb
- https://ubuntu.com/kubernetes/docs/metallb
