## Namespace

Apa itu namespace? Mudahnya namespace digunakan untuk mengelompokan resource yang ada di kubernetes. Pengelompokan Namespace biasanya dipengaruhi oleh besarnya resource yang ada disebuah cluster kubernetes. Dengan menggunakan Namespace ini dapat mempermudah team untuk mengelola resource masing2 tim tanpa mempengaruhi resource yang lain. Sebagai contoh kita tahu bahwa kita tidak dapat membuat resource dengan nama yang sama didalam satu Namespace, akan tetapi jika berada dalam namespace yang berbeda resource tersebut bisa dibuat. (benerin nanti)

### Membuat Namespace

Untuk membuat Namespace, sama seperti resource lainnya kita perlu membuat sebuah configuration file.

```yaml
apiVersion: v1 
kind: Namespace
metadata:
  name: namespace-name
```

Sebagai contoh saya akan membuat sebuah namespace untuk tim QA

```yaml
apiVersion: v1 
kind: Namespace
metadata:
  name: quality-assurance
```

buat resource dengan menggunakan `kubectl create` atau `kubectl create`

```bash
$ kubectl create -f qa-namespace.yaml
namespace/quality-assurance created
```

Cek namespace

```bash
$ kubectl get namespaces
NAME                STATUS   AGE
default             Active   86m
kube-node-lease     Active   86m
kube-public         Active   86m
kube-system         Active   86m
quality-assurance   Active   64s
```

### Menggunakan Namespace

Sebagai contoh, dibawah saya membuat sebuah pod dengan nama `nginx-pod`. Seperti yang kita ketahui jika kita tidak mendefinisikan namespace diconfiguration file resource tersebut maka akan dimasukan ke namespace `default` dan jika saya mencoba membuat pod yang sama maka tidak akan terjadi perubahan dikarenakan sudah terdapat pod yang sama di namespace tersebut.

```bash
$ kubectl create -f pod.yaml
pod/nginx-pod created
$ kubectl create -f pod.yaml
pod/nginx-pod unchanged
```

Untuk itu kita perlu mendeploy pod tersebut di namespace yang berbeda dengan cara menambahkan `--namespace <namespace-name>` untuk mendefinisikan namespace

```bash
$ kubectl create -f pod.yaml --namespace quality-assurance
pod/nginx-pod created
```

Kita bisa melihat bahwa pod bisa berjalan dengan normal bahkan dengan nama yang sama. Untuk mengecek status pod pada sebuah namespace, jalankan perintah dibawah

```bash
$ kubectl get pod --namespace quality-assurance
NAME        READY   STATUS    RESTARTS   AGE
nginx-pod   1/1     Running   0          95s
```

### Menghapus Namespace

Untuk dapat menghapus sebuah namespace, kita bisa menggunakan `kubectl delete namespace <namespace-name>` 

```bash
$ kubectl delete namespace quality-assurance
namespace "quality-assurance" deleted
```

> Perlu diingat jika menghapus namespace, maka semua resource yang ada didalam namespace tersebut akan ikut terhapus

### Yang Perlu Diketahui Tentang Namespace

- Pod dengan nama yang sama boleh berjalan asalkan di Namespace yang berbeda
- Namespace bukanlah cara untuk mengisolasi resource
- Walaupu berbeda namespace, pod akan tetap bisa saling berkomunikasi dengan pod lain di namespace yang berbeda.