## Kubernetes Maintenance Mode

Kubernetes menyediakan cara untuk memelihara/maintain node dalam cluster secara manual. Dengan kubectl kita bisa dengan mudah membuat sebuah object node dan memodifikasinya, sebagai contoh kita mark node tersebut `unschedulable` agar tidak ada pod yang bisa dideploy disana. Contoh kasusnya adalah semisal kita ingin menambahkan resource atau bahkan mengganti node dalam cluster kubernetes, best practicenya adalah kita harus menandai node tersebut `unschedulable` dan dilanjutkan dengan memindahkan atau nge`drain` semua Pod dalam node tersebut ke node lain atau biasa disebut dengan kubernetes `cordon` dan `drain`.

### Kubernetes Cordon

Kuberentes cordon adalah sebuah cara untuk menandai atau `taint` sebuah node sebagai unschedulable. Dengan menggunakannya pada node, kita bisa memastikan tidak akan ada Pod yang akan dischedule pada node tersebut. Kubernetes cordon mencegah scheduler meletakan pod baru pada node tersebut, akan tetapi tidak mempengaruhi Pod yang sebelumnya sudah berjalan sebelum node tersebut di cordon.

Berikut adalah contoh melakukan cordon dan drain pada node.

#### Step 1 - Cordoning a Node

Seperti yang dijelaskan diatas, kita menggunakan `cordon` untuk manandai atau menambahkan `taint` pada node untuk memberitahu ke `scheduler` untuk tidak meletakan Pod baru di node yang dicordon. Untuk cordoning sebuah node kita menggunakan command dibawah

```bash
$ kubectl cordon <nama-node>
```

Selanjutnya kita bisa cek status dari node tersebut.

![image-20240914125205336](https://github.com/user-attachments/assets/c841f921-7ef9-47e3-9189-25acac6681fd)

Seperti kita lihat pada gambar diatas, node `k8s-worker-2` ketika dicordon statusnya berbuah menjadi `SchedulingDisabled` yang berarti cordon sudah menandai node tersebut agar tidak diletakan Pod baru. Tapi kita bisa lihat juga bahwa Pod yang sebelumnya sudah berjalan itu tetap berjalan dengan normal. Maka dari itu kita perlu **memindahkan** pod tersebut ke node lain dengan menggunakan `kubectl drain`.

#### Step 2 - Draining a Node

Untuk mengosongkan Node dari Pod yang sudah berjalan, kita perlu **mengeringkan** atau **drain** node tersebut. Proses ini akan mengeluarkan semua Pod dalam node sehingga Pod tersebut bisa direschedule pada Node yang lain oleh scheduler. 

Drain node yang sebelumnya sudah di cordon.

```bash
$ kubectl drain <nama-node> --ignore-daemonsets
```

> **--ignore-daemonsets** berguna untuk memastikan DaemonSets tidak ikut dikeluarkan dari node tersebut. Karena kita tahu bahwa DaemonSets Controller akan membuat ulang Pod DaemonSets jika sebelumnya kita paksa keluarkan. DaemonSets Controller tidak memperdulikan adanya taint uncheduled yang membuat proses draining tidak bisa berjalan dengan baik.

Jika berjalan dengan normal, akan terlihat output seperti dibawah.

![image-20240914125205336](https://github.com/user-attachments/assets/e1a88c37-e0b7-405a-a0e3-a2082ba13bf8)

Jika tidak terdapat error kita bisa lanjutkan untuk maintenance node tersebut. Sebagai contoh saya akan mematikan node tersebut karena ingin menambahkan jumlah resource pada node tersebut.

#### Step 3 - Uncordoning a Node

Selanjutnya jika sudah selesai melakukan maintenance node & agar node tersebut dapat digunakan kembali kita bisa melakukan `uncordoning` node. 

```bash
$ kubectl uncordon <nama-node>
```

![image-20240914131256349](https://github.com/user-attachments/assets/fa920fad-914c-4497-829f-ef4ab39acfec)

Pada gambar diatas bisa dilihat status dari node `k8s-worker-2` sudah kembali normal, dan kita bisa menschedule Pod pada node tersebut lagi.

Referensi:

- https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/
- https://cast.ai/blog/kubernetes-cordon-how-it-works-and-when-to-use-it/
- https://medium.com/@abhijeetjha916/everything-about-kubernetes-cluster-maintenance-0892a5ac6374
- https://kubernetes.io/docs/reference/kubectl/generated/kubectl_cordon/
