## Kubernetes Volumes

File didalam sebuah container sifatnya sementara, ketika container dimatikan maka data yang berada dalam container tersebut juga menghilang. Maka dari itu dalam kubernetes terdapat sebuah mekanisme untuk menyediakan penyimpanan data yang dimana nantinya dapat diakses oleh Pod. Kubernetes mensupport banyak jenis Volumes, sebuah Pod dapat menggunakan berbagai jenis Volumes secara bersamaan. Terdapat dua klasifikasi Volumes di kubernetes yakni, Ephemeral Volumes dan Persistent Volumes. 

Referensi :

- https://kubernetes.io/docs/concepts/storage/volumes/
- https://bluexp.netapp.com/blog/cvo-blg-5-types-of-kubernetes-volumes-and-how-to-work-with-them

### Ephemeral Volumes

Beberapa aplikasi membutuhkan penyimpanan tambahan namun tidak memperdulikan data tersebut tetap persisten atau tidak ketika restart. Sebagai contoh seperti *caching* yang dibatasi karena ukuran memori. Aplikasi lain juga memerlukan beberapa read-only input data yang ada dalam sebuah file, seperti data konfigurasi dan secret keys. **Ephemeral Volumes** didesain untuk kasus - kasus seperti diatas karena Ephemeral Volume memiliki sifat yang mengikuti umur Pod yang dibuat serta dihapus bersamaan dengan Pod tersebut. Ephemeral Volume ditentukan dalam line PodSpec yang dapat mempermudah dalam managemen hingga deploymennya.

Kubernetes memiliki beberapa jenis dari Ephemeral Volume yang digunakan untuk kebutuhan masing2, yakni:

- [emptyDir](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir)

  Sesuai dengan namanya, emptyDir akan membuat direktori kosong ketika Pod hidup, dengan storage yang datang dari base directory kubelet (biasanya root disk) atau RAM. emptyDir akan dihapus ketika Pod dihentikan. emptyDir cocok untuk penyimpanan yang sifatnya sementara seperti cache yang dimana tidak diperlukan setelah Pod dihentikan.

- [configMap](https://kubernetes.io/docs/concepts/storage/volumes/#configmap), [downwardAPI](https://kubernetes.io/docs/concepts/storage/volumes/#downwardapi) dan [secret]().

  Ketiga jenis volumes diatas bekerja dengan menginject data kubernetes kedalam Pod.

  Sedikit penjelasan mengenai ketiga jenis volume diatas:

  - configMap: digunakan untuk menyimpan konfigurasi yang bersifat tidak rhasia seperti (key-value) yang dapat digunakan oleh Pod.
  - downwardAPI: singkatnya ini adalah salah satu cara untuk mengekspos infromasi terkait Pod itu sendiri ke container. [Cek disini](https://kubernetes.io/docs/tasks/inject-data-application/downward-api-volume-expose-pod-information/)
  - secret: digunakan untuk menyimpan data sensitif seperti password, API token atau encryption key.

- [CSI epehemeral volumes](https://kubernetes.io/docs/concepts/storage/ephemeral-volumes/#csi-ephemeral-volumes)

  Secara konsep, CSI epehemeral volumes mirip dengan configMap, downwardAPI dan secret. Volume ini menggunakan driver CSI (Container Storage Interface) untuk menyediakan penyimpanan sementara untuk Pod tanpa memerlukan adanya PV (PersistentVolume) atau PVC (PersistentVolumeClaim). ??? well belum terlalu paham sih

- [Generic Ephemeral Volumes](https://kubernetes.io/docs/concepts/storage/ephemeral-volumes/#generic-ephemeral-volumes)
  Volumes ini mirip dengan emptyDir yang dimanan menyediakan direktori per-Pod dari nol dan biasanya kosong setelah dibuat. Akan tetapi Generic Ephemeral Volumes memiliki berberapa fitur tambahan, diantaranya:

  - Penyimpanannya dalam lokal atau network-attached.
  - Dapat menetapkan size dari volumes.
  - Operasi khusus dalam volume dapat didukung karena diasumsikan bahwa drivernya mendukung. sperti snapshooting, cloning, resizing dan tracking kapasitas storage.

emptyDir, configMap, downardAPI dan secret diatur oleh kubelet dalam setiap node dan disediakan sebagai [local ephemeral storage](https://kubernetes.io/docs/concepts/storage/ephemeral-volumes/), sedangkan CSI ephemeral storage harus disediakan oleh third-party storage drivernya CSI.

Referensi :

- https://kubernetes.io/docs/concepts/storage/ephemeral-volumes/

### Persistent Volumes

Berbeda dengan Ephemeral Volumes, sesuai dengan namanya PersistentVolumes menyimpan data secara persistent yang dimana tidak mengikuti umur Pod. PersistentVolumes sub-system menyediakan API untuk user dan administrator yang mengabstraksikan detail dari bagaimana storage berasal dan bagaimana storage tersebut digunakan. Untuk itu kita menggunakan dua resources API yakni PersistentVolume dan PersistentVolumeClaim.

- [PersistentVolume](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#:~:text=A%20PersistentVolume%20(PV))

  PersistentVolume atau PV adalah sebuah storage pool dalam cluster yang sudah disiapkan oleh administrator atau disiapkan secara dinamis menggunakan [StorageClasses](https://kubernetes.io/docs/concepts/storage/storage-classes/). PV digunakan oleh user untuk mendeploy aplikasi didalam cluster, PV bertindak sebagai sebuah resource dalam cluster layaknya seperti sebuah node didalam sebuah cluster resource. Users dapat memilih penyimpanan dari pool tersebut menggunakan PersistentVolumeClaim atau PVC.

- [PersistentVolumeClaim](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#:~:text=A%20PersistentVolumeClaim%20(PVC))

  PersistentVolumeClaim atau PVC adalah sebuah permintaan storage yang dibuat oleh user atau aplikasi yang berjalan di cluster. Cara kerja PVC mirip dengan sebuah Pod. Pod menggunakan resource dari Node dan PVC menggunakan resource dari PV. Pods dapat meminta secara spesifik berapa resource yang diinginkan (cpu dan memory), dan PVC dapat meminta secara spesifik berapa size dan apa [access modes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes) storage.

#### Lifecycle of a Volume and Claim

PV merupakan resources didalam cluster dan PVC merupakan request untuk resources tersebut and juga bertugas untuk mengecek klaim ke resources tersebut. Interaksi diatara PV dan PVC mengikuti lifecycle dibawah

#### Provisioning

Terdapat 2 cara PV dapat disiapkan, secara statis atau dinamis.

#### Static PV Provisioning

Dalam Static PV Provisioning, seorang cluster administrator membuat bebrapa jumlah PV, membawa detail dari storage aslinya yang nantinya bisa digunakan oleh cluster users. PV akan ada di Kubernetes API dan tersedia untuk digunakan.

#### Dynamic PV Provisioning

Ketika tidak ada satupun Static PV yang dibuat oleh administrator sesuai dengan PVC yang user buat, maka cluster akan mencoba secara dinamis  menyediakan sebuah volume yang secara khusus untuk PVC tersebut. Proses penyediaan ini bergantung dengan [StorageClasses](https://kubernetes.io/docs/concepts/storage/storage-classes/). PVC harus mereminta sebuah storage class dan administrator juga harus membuat serta mengkonfigurasi class tersebut agar dynamic provisioning terjadi. Claims yang merequest class kosong `""` secara otomatis akan mendisable dynamic provisioning untuk claims tersebut.

#### StorageClasses

Sebuah StorageClass menyediakan cara untuk administrator untuk mendeskripsikan classes dari storage yang mereka berikan. Class yang berbeda bisa digunakan untuk quality-of-service level atau backup policies yang ditentukan oleh cluster administrator. Konsep dari storage class ini mirip dengan `profiles` di sistem storage yang lain.

Referensi:

- https://kubernetes.io/docs/concepts/storage/persistent-volumes/
- https://kubernetes.io/docs/concepts/storage/persistent-volumes/#:~:text=working%20examples.-,Lifecycle%20of%20a%20volume%20and%20claim,-PVs%20are%20resources
- https://www.squadcast.com/blog/introduction-to-kubernetes-storage
- https://kubernetes.io/docs/concepts/storage/storage-classes/

## Kuberentes Volumes Example

Berikut adalah beberapa contoh menggunakan Volume pada Pod :)

### Ephemeral Volumes

Seperti yang dijelaskan diatas, ephemeral volume memiliki sifat yang tidak kekal atau keberadaanya menyesuaikan dengan umur dari Pod. Sehingga ini cocok digunakan untuk caching service. Type volume yang sering digunakan diantaranya emptyDir, configMap, downwardAPI, secrets dan generic ephemeral volumes.

#### emptyDir Example

Buat manifest untuk Pod, sebagai contoh dibawah saya menggunakan image `nginx`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-emptydir
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - mountPath: /cache
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir:
      sizeLimit: 10Mi
```

Diatas saya membuat volume untuk cache dengan nama `cache-volume` dan dimount dalam container kepath `/cache`.  Selanjutnya apply manifest tersebut dan describe Pod untuk melihat lebih detail terkait pod yang sudah dibuat.

```bash
$ kubectl describe pod nginx-emptydir
Name:             nginx-emptydir
Namespace:        default
Priority:         0
Service Account:  default
Node:             k8s-worker-2/10.20.1.21
Start Time:       Wed, 18 Sep 2024 11:38:34 +0800
Labels:           <none>
Annotations:      cni.projectcalico.org/containerID: d28e715d42c4d4aa77536d7d955706b1148a23d0954049dff7e057dbb3dccfa6
                  cni.projectcalico.org/podIP: 192.168.140.45/32
                  cni.projectcalico.org/podIPs: 192.168.140.45/32
Status:           Running
...
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /cache from cache-volume (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-jtzhr (ro)
...
Volumes:
  cache-volume:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:
    SizeLimit:  10Mi
```

Coba untuk masuk kedalam Pod

```shell
## Exec into Pod
$ kubectl exec -ti nginx-emptydir -- /bin/bash

## Check the volume
root@nginx-emptydir:/# cd /cache 
root@nginx-emptydir:/cache# ls

## Create a file
root@nginx-emptydir:/cache# cat << EOF | tee cache.file 
> this is caches
> this is caches
> this is caches
> EOF
root@nginx-emptydir:/cache# ls
cache.file
```

Sekarang coba untuk menhapus Pod tersebut kemudian jalankan lagi

```shell
## Delete Pod
$ kubectl delete pod nginx-emptydir

## Start Pod
$ kubectl apply -f nginx-emptydir.yaml
```

Masuk ke dalam Pod dan cek file yang dibuat tadi.

```shell
## Exec into Pod
$ kubectl exec -ti nginx-emptydir -- /bin/bash

## Check the file
root@nginx-emptydir:/# cd /cache
root@nginx-emptydir:/cache# ls
root@nginx-emptydir:/cache#
```

Diatas kita bisa lihat bahwa file (cache) yang sebelumnya dibuat ikut terhapus ketika Pod mati  atau direstart. Itu adalah salah satu contoh dari penggunaan ephemeral volume tipe emptyDir. untuk contoh lain bisa dibaca di [kubernetes.io](https://kubernetes.io/docs/concepts/storage/volumes/)

Referensi:

- https://kubernetes.io/docs/concepts/storage/volumes/
- https://www.kubermatic.com/blog/keeping-the-state-of-apps-1-introduction-to-volume-and-volumemounts/

### Persistent Volume

Berbalik dengan ephemeral volume yang hanya menyediakan volume sementara yang mengikuti umur dari Pod, Persistent Volume seperti namanya memiliki sifat yang persisten dan akan tetap ada walau Pod sudah dihapus. Salah satu contoh yang paling mudah adalah menggunakan hostPath, yang dimana volume ini akan mounting sebuah direktori dari host node yang menjalankan Pod kedalam Pod tersebut.

#### hostPath Example

 Buat manifest untuk Pod dengan volume tipe hostPath.

```bash
apiVersion: v1
kind: Pod
metadata:
  name: nginx-hostpath
spec:
  containers:
  - name: my-app
    image: nginx
    ports:
    - containerPort: 8080
    volumeMounts:
    - name: my-volume
      mountPath: /app
  volumes:
  - name: my-volume
    hostPath:
      path: /mnt/vpath 
```

Apply manifest tersebut lalu describe pod untuk melihat informasi terkait dimana Pod tersebut dideploy, volume yang digunakan  dan path dari volume dalam node.

> Bisa menggunakan node selector jika ingin mendeploy Pod dinode tertentu.

```bash
$ kubectl describe pod nginx-hostpath
Name:             nginx-hostpath
Namespace:        default
Priority:         0
Service Account:  default
Node:             k8s-worker-2/10.20.1.21
...
    Mounts:
      /app from nginx-volume (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-9rj5m (ro)
...
Volumes:
  nginx-volume:
    Type:          HostPath (bare host directory volume)
    Path:          /mnt/hostpath
```

Coba masuk kedalam Pod

```bash
## Exec into Pod
$ kubectl exec -ti nginx-hostpath -- /bin/bash

## Move to mounted volume then create a file
root@nginx-hostpath:/# cd /app
root@nginx-hostpath:/app# echo "test test test" > hostpath.file
root@nginx-hostpath:/app# ls
hostpath.file
```

Kemudian coba masuk ke node worker letak Pod dideploy.

```bash
lnearher@k8s-worker-2:~$ cd /mnt/hostpath
lnearher@k8s-worker-2:/mnt/hostpath$ ls | cat hostpath.file
test test test
```

Bisa kita lihat bahwa file yang ada dalam Pod juga ada di direktori node. Selanjutnya coba untuk menghapus Pod tersebut dan cek apakah file tersebut masih ada atau ikut hilang seperti emptyDir.

```bash
## Delete Pod
$ kubectl delete pod nginx-hostpath

## SSH into Node that volume has mounted before
lnearher@k8s-worker-2:~$ cd /mnt/hostpath
lnearher@k8s-worker-2:/mnt/hostpath$ ls | cat hostpath.file
test test test
```

Kita bisa lihat file yang sebelumnya dibuat masih ada di direktori pada node bahkan setelah Pod tersebut dihapus.

> Note: Menggunakan hostPath sebagai penyimpanan di stage production sangatlah tidak di rekomendasikan karena seperti kita tahu sifat dari hostPath sangat terikat dengan Node tempat Pod dijalankan. Jika Pod direstart atau dipindahkan ke node lain maka Pod tidak lagi memiliki akses ke data yang tersimpan di volume hostPath sebelumnya. 

Referensi:

- https://kubernetes.io/docs/concepts/storage/volumes/#:~:text=the%20v1.26%20release.-,hostPath,-A%20hostPath%20volume
- https://www.kubermatic.com/blog/keeping-the-state-of-apps-1-introduction-to-volume-and-volumemounts/

### PersistentVolume (PV) and PersistentVolumeClaim (PVC)

PersistentVolume dan PersistentVolumeClaim bekerja untuk membuat penyimpanan yang terbebas dari node, sehingga memungkinkan data dalam volume tersebut dapat diakses oleh Pod walaupun Pod tersebut dipindahkan ke Node lainnya. Namun tidak hanya itu saja, masih banyak keunggulan lain menggunakan kedua tipe volume ini seperti halnya adanya dynamic provisioner yang dapat mempermudah user dalam memanagemen storage.  

Seperti yang sudah saya mention diatas, dalam menyiapkan PersistentVolume kita bisa melakukannya dengan 2 cara, yakni dengan Static Provisioning dan Dynamic Provisioning.

#### Static Provisioning (Manual)

Buat manifest untuk mendeploy PV terlebih dahulu

```yaml

```

