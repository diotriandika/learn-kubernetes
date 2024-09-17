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

testtesttest

Referensi:

- https://kubernetes.io/docs/concepts/storage/persistent-volumes/

### Penggunan Volume



