## Kubernetes Maintenance Mode

Kubernetes menyediakan cara untuk memelihara/maintain node dalam cluster secara manual. Dengan kubectl kita bisa dengan mudah membuat sebuah object node dan memodifikasinya, sebagai contoh kita mark node tersebut `unschedulable` agar tidak ada pod yang bisa dideploy disana. Contoh kasusnya adalah semisal kita ingin menambahkan resource atau bahkan mengganti node dalam cluster kubernetes, best practicenya adalah kita harus menandai node tersebut `unschedulable` dan dilanjutkan dengan memindahkan atau nge`drain` semua Pod dalam node tersebut ke node lain atau biasa disebut dengan kubernetes `cordon` dan `drain`.

### Kubernetes Cordon



