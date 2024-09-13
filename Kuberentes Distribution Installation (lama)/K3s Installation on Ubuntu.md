## Apa itu K3s?

K3s merupakan versi lebih ringan dari K8s (Kubernetes) yang dapat digunakan sebagai simulasi production-level Kubernetes di local machine. K3s menghilangkan komponen - komponen yang jarang digunakan seperti legacy, alpha, dan fitur-fitur non-default serta mengantikan Kubernetes complex networking, load balancing, storage, dan DNS Server dengan alternatif yang lebih ringan. Ini membuat K3s berjalan lebih efisien di system local. Cluster K3s menyediakan environment yang dapat mensimulasikan K8s standar, ini memungkinkan kita untuk melakukan pengujian kubernetes seperti di production environment.

Referensi :

- https://thechief.io/c/editorial/k3s-vs-k3d/
- https://www.devopsschool.com/blog/a-basic-k3s-tutorial-for-kubernetes/
- https://k3s.io/

## Instalasi K3s 

Instalasi K3s terbilang mudah, K3s dapat dijalankan di Virtual Machine atau environment dengan basis ARM64 dan ARMv7. Disini saya mensimulasikan dengan menggunakan Virtual Machine dengan basis Ubuntu 22.04 untuk instalasi K3s. Pastikan memiliki hardware dengan spesifikasi sebagai berikut:

| Spec | Minimum | Recommended |
| ---- | ------- | ----------- |
| CPU  | 1 Core  | 2 Cores++   |
| RAM  | 512 MB  | 1 GB++      |

Disini kita bisa menggunakan script yang sudah disediakan di situs dokumentasi resmi K3s.

```bash
$ curl -sfL https://get.k3s.io | sh -
```

Setelah menjalankan perintah diatas :

- K3s akan dikonfigurasi agar melakukan restart otomatis setelah node mengalami reboot atau jika terjadi crash bahkan mati.
- Utilitas tambahan akan terinstall, termasuk `kubectl`, `crictl`, `ctr`, `k3s-killall.sh` dan `k3s-unsinstall.sh`.
- File `kubeconfig` otomatis terwrite di `/etc/rancher/k3d/k3s.yaml` dan kubectl yang diinstall oleh k3s secara otomatis akan menggunakannya.

Verifikasi instalasi dengan menjalankan:

```bash
$ sudo k3s kubectl get nodes
NAME           STATUS   ROLES                  AGE   VERSION
master-node1   Ready    control-plane,master   52m   v1.29.5+k3s1
```

Perintah diatas akan menampilkan status dari node2 K3s.

### Setup Kubectl & Kubeconfig

Secara default, K3s akan membuat kubeconfig file di /etc/rancher/k3s/k3s.yaml yang mengizinkan komunikasi ke cluster menggunakan kubectl. Agar kita dapat menggunakan konfigurasi ini dengan user kita, kita harus mengcopy file tersebut ke home directory dan memastikan agar tidak readable secara global.

Pertama setup environmental variable untuk `KUBECONFIG=~/.kube/config`

```bash
$ export KUBECONFIG=~/.kube/config
```

Jika direktori `~/.kube` tidak ada, kita bisa membuatnya terlebih dahulu, lalu generate file kubeconfig  dan melakukan modify file seperti dibawah

```bash
$ mkdir ~/.kube 2> /dev/null
$ sudo k3s kubectl config --raw > "$KUBECONFIG"
$ chmod 600 "$KUBECONFIG"
```

Selanjutnya agar dapat digunakan saat reboot, kita perlu menambahkan line dibawah pada `.bashrc`

```bash
export KUBECONFIG=~/.kube/config
```

Cara lain kita bisa memanfaatkan flag `--write-kubeconfig-mode` pada saat instalasi.

```bash
$ curl -sfL https://get.k3s.io | sh -s - --write-kubeconfig-mode 644
```

Verifikasi dengan menjalankan

```bash
$ kubectl get nodes
NAME           STATUS   ROLES                  AGE   VERSION
master-node1   Ready    control-plane,master   14s   v1.29.5+k3s1
```

Referensi :

- https://docs.k3s.io/quick-start
- https://devops.stackexchange.com/questions/16043/error-error-loading-config-file-etc-rancher-k3s-k3s-yaml-open-etc-rancher
- https://www.devopsschool.com/blog/a-basic-k3s-tutorial-for-kubernetes/
- https://github.com/k3s-io/k3s/issues/389



