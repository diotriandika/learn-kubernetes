## ConfigMaps

ConfigMap adalah API Object yang digunakan untuk menyimpan non-confidential (tidak rahasia) data dalam key-value pairs. Pods dapat menggunakan ConfigMap sebagai variabel environment, command-line arguments, atau sebagai configuration files didalam sebuah volume. Kegunaan utama dari ConfigMap adalah untuk memisahkan detail konfigurasi dari container image yang dimana dapat meningkatkan fleksibilitas container tersebut. Konteks nyatanya biasanya ConfigMaps digunakan untuk khusus ke environment yang berbeda seperti development, testing dan production

ConfigMap mempermudah dalam menkonfiugrasi dan management aplikasi. Sehingga tidak lagi memerlukan hardcode data konfigurasi satu per satu ke kontainer.

![image](https://images.prismic.io/qovery/655e12b4531ac2845a255234_unnamed-6-.png?auto=format,compress)

