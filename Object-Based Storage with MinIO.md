## MinIO Overview

MinIO adalah sebuah solusi Object Storage yang menyediakan AWS S3-compatible API dan mendukung semua inti fitur dari AWS S3. MinIO dibuat agar mudah dideploy dibebagai macam environment seperti public atau private cloud, baremetal infrastructure, orchestrated infrastructure dan lain2.

Lalu apa itu Object Storage?

Object Storage atau object-based storage adalah salah satu jenis arsitektur penyimpanan data seperti gambar, video atau format file lainnya. Object storage melihat file yang tersimpan di dalamnya sebagai objek atau benda. setiap unit data disimpan di dalam satu lokasi sehingga bisa dicari dan diakses dengan mudah. Tidak seperti filesystem yang membaca file dalam hirarki folder, object-based storage menyimpan data sebagai objek yang memiliki komponen2 seperti metadata dan Unique Identifier (ID) yang dapat mempermudah dalam mencari file2 yang tidak terstruktur.

## MinIO Installation

Instalasi MinIO bisa diinstall di NFS-Server untuk mempermudah akses. Dibawah saya hanya memberikan demo untuk single-node single-drive MinIO server ke kubernetes. Dalam production sangat direkomendasikan untuk menggunakan MinIO Operator.

### Step 1 - NFS Server Preparation

Siapkan Node khusus sebagai NFS Server tempat meletakan shared directory yang nantinya digunakan oleh MinIO.

**Run on NFS Server Node**

1. SSH ke node yang ingin digunakan lalu install package NFS Server

   ```
   $ sudo apt-get install nfs-kernel-server
   ```

2. Buat direktori yang akan diexport

   ```bash
   ## Create Export Directory
   $ sudo mkdir /mnt/minio
   
   ## Set directory privileges
   $ sudo chmod 777 /mnt/minio
   ```

3. Export direktori

   ```bash
   $ sudo vim /etc/exports
   ```

   ```bash
   ## Add these following lines
   /mnt/minio  10.20.1.0/24(rw,sync,no_subtree_check)
   ```

   ```bash
   $ sudo exportfs -av
   exporting 172.16.1.0/24:/mnt/minio
   ```

4. Restart NFS Server kemudian cek detail dari direktori yang diexport

  ```bash
## Restart NFS Server
$ sudo systemctl restart nfs-kernel-server

## Show export details
$ /sbin/showmount -e nfs-server
Export list for nfs-server:
/mnt/minio 172.16.1.0/24
  ```

  > `nfs-server` adalah static dns yang mengarah ke 172.16.1.16 atau node nfs-server

**Run on All Control Plane and Worker Nodes**

1. Install nfs client disemua node dalam cluster

   ```bash
   $ sudo apt-get install nfs-client -y
   ```

   > Bisa menggunakan multipane tmux atau ansible untuk mempermudah instalasi

**Jalankan di Node Control Plane**

1. Tambahkan repo helm nfs subdir external provisioner

   ```bash
   ## Add nfs helm repo
   $ helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner
   
   ## Update helm repo
   $ helm repo update
   ```

2. Install nfs subdir external provisioner

   ```bash
   $ helm install nfs-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
   --set nfs.server=172.16.1.16 \
   --set nfs.path=/mnt/minio
   ```

3. Cek pod dan storageclass nfs

   ```bash
   ## Check nfs provisioner pod
   $ kubectl get pod 
   NAME                                                              READY   STATUS    RESTARTS      AGE
   nfs-provisioner-nfs-subdir-external-provisioner-96f4f677f-9xvl7   1/1     Running   0             91s
   
   ## Check nfs provisioner storageclass
   $ kubectl get sc
   NAME         PROVISIONER                                                     RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
   nfs-client   cluster.local/nfs-provisioner-nfs-subdir-external-provisioner   Delete          Immediate           true                   14s
   ```

4. Test dengan membuat pvc

   ```yaml
   apiVersion: v1
   kind: PersistentVolumeClaim
   metadata:
     name: nfs-pvc
   spec:
     accessModes:
     - ReadWriteOnce
     storageClassName: nfs-client
     resources:
       requests:
         storage: 1Gi
   ```

5. Cek PVC yang dibuat dan apakah sudah bisa auto create pv

   ```bash
   $ kubectl get pvc 
   NAME      STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
   nfs-pvc   Bound    pvc-f2d7ab07-bbe3-4720-aec4-f4675c05848b   1Gi        RWO            nfs-client     <unset>                 3s
   
   $ kubectl get pv
   NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM             STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
   pvc-f2d7ab07-bbe3-4720-aec4-f4675c05848b   1Gi        RWO            Delete           Bound    default/nfs-pvc   nfs-client     <unset>                          9s   
   ```

### 
