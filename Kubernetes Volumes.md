## Kubernetes Volumes

File didalam sebuah container sifatnya sementara, ketika container dimatikan maka data yang berada dalam container tersebut juga menghilang. Maka dari itu dalam kubernetes terdapat sebuah mekanisme untuk menyediakan penyimpanan data yang dimana nantinya dapat diakses oleh Pod. Kubernetes mensupport banyak jenis Volumes, sebuah Pod dapat menggunakan berbagai jenis Volumes secara bersamaan. Terdapat dua klasifikasi Volumes di kubernetes yakni, Ephemeral Volumes dan Persistent Volumes. 

Referensi :

- https://kubernetes.io/docs/concepts/storage/volumes/
- https://bluexp.netapp.com/blog/cvo-blg-5-types-of-kubernetes-volumes-and-how-to-work-with-them

### Ephemeral Volumes

Beberapa aplikasi membutuhkan penyimpanan tambahan namun tidak memperdulikan data tersebut tetap persisten atau tidak ketika restart. Sebagai contoh seperti *caching* yang dibatasi karena ukuran memori. Aplikasi lain juga memerlukan beberapa read-only input data yang ada dalam sebuah file, seperti data konfigurasi dan secret keys. **Ephemeral Volumes** didesain untuk kasus - kasus seperti diatas karena Ephemeral Volume memiliki sifat yang mengikuti umur Pod yang dibuat serta dihapus bersamaan dengan Pod tersebut. Ephemeral Volume ditentukan dalam line PodSpec yang dapat mempermudah dalam managemen hingga deploymennya.

Kubernetes memiliki beberapa jenis dari Ephemeral Volume yang digunakan untuk kebutuhan masing2, yakni:

- emptyDir

  Sesuai dengan namanya, emptyDir akan membuat direktori kosong ketika Pod hidup, dengan storage yang datang dari base directory kubelet (biasanya root disk) atau RAM. emptyDir akan dihapus ketika Pod dihentikan. emptyDir cocok untuk penyimpanan yang sifatnya sementara seperti cache yang dimana tidak diperlukan setelah Pod dihentikan.

- 



Referensi :

- https://kubernetes.io/docs/concepts/storage/ephemeral-volumes/
- https://kubernetes.io/docs/concepts/storage/volumes/#emptydir
- 
