## MinIO Overview

MinIO adalah sebuah solusi Object Storage yang menyediakan AWS S3-compatible API dan mendukung semua inti fitur dari AWS S3. MinIO dibuat agar mudah dideploy dibebagai macam environment seperti public atau private cloud, baremetal infrastructure, orchestrated infrastructure dan lain2.

Lalu apa itu Object Storage?

Object Storage atau object-based storage adalah salah satu jenis arsitektur penyimpanan data seperti gambar, video atau format file lainnya. Object storage melihat file yang tersimpan di dalamnya sebagai objek atau benda. setiap unit data disimpan di dalam satu lokasi sehingga bisa dicari dan diakses dengan mudah. Tidak seperti filesystem yang membaca file dalam hirarki folder, object-based storage menyimpan data sebagai objek yang memiliki komponen2 seperti metadata dan Unique Identifier (ID) yang dapat mempermudah dalam mencari file2 yang tidak terstruktur.

## MinIO Installation

Instalasi MinIO bisa diinstall di NFS-Server untuk mempermudah akses. Dibawah saya hanya memberikan demo untuk single-node single-drive MinIO server ke kubernetes. Dalam production sangat direkomendasikan untuk menggunakan MinIO Operator.

### Step 1 - NFS Server Preparation

Siapkan Node khusus sebagai NFS Server tempat meletakan shared directory yang nantinya digunakan oleh MinIO.

1. 
