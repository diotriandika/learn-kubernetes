## Apa itu MetalLB?

![metal-lb use cases](https://kubernetes.github.io/ingress-nginx/images/baremetal/metallb.jpg)

MetalLB adalah sebuah implementasi dari LoadBalancer untuk cluster Kubernetes yang berjalan di infrastruktur Bare-Metal atau Virtualized. Biasanya, service LoadBalancer di Kuberentes secara default hanya tersedia dalam lingkungan Public Cloud yang mendukung integrasi khusus untuk memprovisioning load balancer eksternal. Namun, ketika menggunakan di lingkungan Bare-Metal atau virtualized, tidak ada layanan LoadBalancer yang otomatis tersedia. Inilah mengapa MetalLB hadir sebagai solusi alternatif yang memungkinkan kita untuk menggunakan IP LoadBalancer dilingkungan infrastruktur Bare-Metal atau Virtualized. Dengan MetalLB kita bisa mengkonfigurasi IP Address LoadBalancer dan mengarahkannya ke backend Pod yang diinginkan. 

MetalLB memiliki dukungan untuk Local Traffic, yang berarti hanya nodes yang menerima data akan melayani request tersebut. Oleh karena itu, sangat tidak direkomendasikan menggunakan Virtual IP dengan High Traffic workloads karena hanya satu nodes yang akan menerima traffic untuk sebuah service, nodes lain hanya digunakan untuk failover. Baca selengkapnya [disini.](https://ubuntu.com/kubernetes/docs/metallb)

MetalLB bekerja dengan melibatkan penggunakan protocol ARP (Address Resouliton Protocol) atau BGP (Border Gateway Procotol) untuk mengiklankan alamat IP LoadBalancer yang tersedia ke network kita. Ketika request masu ke alamat IP LoadBalancer, MetalLB akan menerukan request tersebut ke backend Pod yang sesuai dengan metode load-balancing yang sudah dikonfigurasi. Dengan MetalLB kita bisa mengatur rentang IP Address yang tersedia, metode load balancing yang akan digunakan dll. 



