## Prasyarat
Sistem Operasi: Ubuntu 20.04 LTS atau 22.04 LTS.

Hak Akses Root: Pastikan Anda memiliki akses sudo untuk menginstal paket dan dependensi.

Koneksi Internet: Diperlukan untuk mengunduh paket dan dependensi.
----------------------------------------------------------------------------------------------------------

## Langkah 1: Perbarui Sistem
Sebelum memulai, pastikan sistem Anda diperbarui:

bash
Copy
sudo apt update
sudo apt upgrade -y
----------------------------------------------------------------------------------------------------------

## Langkah 2: Instal Dependensi
Instal paket-paket yang diperlukan untuk membangun dan menjalankan Open5GS:

bash
Copy
sudo apt install -y software-properties-common
sudo add-apt-repository ppa:open5gs/latest
sudo apt update
sudo apt install -y open5gs
______________________________________________________________________________________________________________
## Langkah 3: Instal MongoDB
Open5GS menggunakan MongoDB sebagai database untuk menyimpan data pelanggan dan informasi jaringan. Instal MongoDB dengan perintah berikut:

# Install MongoDB Tools on Ubuntu 22.04

## 1. Install Curl and GNUPG
```sh
sudo apt-get install gnupg curl
```

## 2. Download MongoDB's GPG keys
```sh
curl -fsSL https://pgp.mongodb.com/server-7.0.asc | \
   sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg \
   --dearmor
```

## 3. Add new source list for MongoDB
```sh
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" \
  | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
```

## 4. Update package db
```sh
sudo apt-get update
```

## 5. Install the tools
```sh
sudo apt install mongodb-org-tools
```
______________________________________________________________________________________________________________
##Langkah 4: Unduh dan Konfigurasi Open5GS
Unduh Open5GS:
Jika Anda ingin mengunduh source code dan membangunnya secara manual, gunakan perintah berikut:

bash
Copy
git clone https://github.com/open5gs/open5gs
cd open5gs
meson build --prefix=/usr/local
ninja -C build
sudo ninja -C build install
Konfigurasi Open5GS:

File konfigurasi Open5GS terletak di /etc/open5gs/.

Sesuaikan file konfigurasi sesuai kebutuhan Anda, seperti mengatur alamat IP, PLMN, dan parameter jaringan lainnya.

Contoh file konfigurasi:

AMF: /etc/open5gs/amf.yaml

SMF: /etc/open5gs/smf.yaml

UPF: /etc/open5gs/upf.yaml
______________________________________________________________________________________________________________
## Langkah 5: Jalankan Open5GS
Setelah instalasi selesai, Anda dapat menjalankan komponen Open5GS secara individual atau menggunakan skrip untuk menjalankan semuanya.

Jalankan Semua Komponen:

bash
Copy
sudo open5gs
Jalankan Komponen Secara Individual:
Contoh untuk menjalankan AMF:

bash
Copy
sudo open5gs-amfd
______________________________________________________________________________________________________________
## Langkah 6: Verifikasi Instalasi
Periksa Status Layanan:
Gunakan perintah berikut untuk memeriksa status layanan Open5GS:

bash
Copy
sudo systemctl status open5gs-*
Periksa Log:
Log Open5GS dapat ditemukan di /var/log/open5gs/. Periksa log untuk memastikan tidak ada kesalahan.

Uji Koneksi:

Gunakan alat seperti curl atau wget untuk menguji koneksi ke komponen Open5GS.

Jika Anda menggunakan UERANSIM (simulator UE dan gNodeB), pastikan UE dapat terdaftar di jaringan Open5GS.
______________________________________________________________________________________________________________
## Langkah 7: Konfigurasi Tambahan (Opsional)
Tambahkan Pelanggan:

Gunakan open5gs-dbctl untuk menambahkan pelanggan ke database MongoDB.

Contoh:

bash
Copy
open5gs-dbctl add 901700123456789 00112233445566778899aabbccddeeff
Integrasi dengan UERANSIM:

Jika Anda ingin menguji Open5GS dengan simulator UE dan gNodeB, Anda dapat menggunakan UERANSIM.
______________________________________________________________________________________________________________
## Langkah 8: Hentikan atau Mulai Ulang Layanan
Untuk menghentikan Open5GS:

bash
Copy
sudo systemctl stop open5gs-*
Untuk memulai ulang Open5GS:

bash
Copy
sudo systemctl restart open5gs-*
