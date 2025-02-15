Menghubungkan OSMOHLR dengan jaringan nyata dalam skenario merged telco membutuhkan beberapa langkah penting, termasuk konfigurasi SS7, Diameter, atau SIGTRAN untuk integrasi dengan HLR/HSS dari operator yang sudah ada. Berikut langkah-langkahnya

## 1. Persiapan dan Instalasi OSMOHLR

Jika belum menginstal OSMOHLR, lakukan instalasi menggunakan Osmocom repository:

```
sudo apt update
sudo apt install osmo-hlr
```
Pastikan juga menginstal dependensi yang dibutuhkan untuk SS7/SIGTRAN:
```
sudo apt install osmo-stp osmo-mgw osmo-msc osmo-bsc
```
___________________________________________________________________________________________________________________

## 2. Konfigurasi OSMOHLR

File konfigurasi utama berada di /etc/osmocom/osmo-hlr.cfg. Edit file ini untuk menyesuaikan dengan jaringan yang akan diintegrasikan:

```
hlr
 database "/var/lib/osmocom/hlr.db"
 gt 1 1.2.3.4  # Global Title untuk OSMOHLR (sesuaikan dengan jaringan SS7)
```

Jika menggunakan SIGTRAN, tambahkan koneksi ke STP:
```
ss7-instance 1
 point-code 1234
 asp asp-clnt-1 2905 1
```
Pastikan jaringan SS7/SIGTRAN dari operator mendukung koneksi ini! 🔄
___________________________________________________________________________________________________________________


## 3. Menghubungkan ke Jaringan Nyata

OSMOHLR harus bisa berkomunikasi dengan jaringan operator. Ada beberapa metode:
a. SIGTRAN untuk SS7 Core Networks

Jika jaringan menggunakan SS7/SIGTRAN, OSMOHLR dapat terhubung melalui STP (Signaling Transfer Point) operator:

  1.  Pastikan ada koneksi SIGTRAN antara OSMOHLR dan STP operator.
  2.  Gunakan osmo-stp sebagai router SIGTRAN.
      Contoh konfigurasi /etc/osmocom/osmo-stp.cfg:

    ```
    cs7-instance 0
     listen m3ua 2905
     route 1 2.3.4.5
    ```
  3.  Pastikan OSMOHLR terhubung ke STP dengan perintah:
    ```
    osmo-hlr -c /etc/osmocom/osmo-hlr.cfg
    ```

b. Diameter untuk Integrasi dengan HSS

Jika jaringan mendukung Diameter, OSMOHLR dapat berfungsi sebagai pengganti HLR untuk subscriber tertentu:

  1.  Konfigurasikan Diameter Proxy atau Relay untuk menghubungkan OSMOHLR ke HSS/HLR utama.
  2.  Gunakan perangkat lunak seperti FreeDiameter untuk meneruskan permintaan antara OSMOHLR dan jaringan utama.

___________________________________________________________________________________________________________________

## 4. Pengujian dan Validasi

Setelah koneksi berhasil, uji fungsionalitasnya:
✅ Cek apakah Subscriber Terdaftar di HLR
```
osmo-hlr -s
```

✅ Uji Koneksi SS7/SIGTRAN
```
osmo-stp -s
```

✅ Verifikasi Diameter melalui Wireshark
Pantau lalu lintas Diameter:
```
tcpdump -i any port 3868 -w diameter_trace.pcap
```

___________________________________________________________________________________________________________________

## 5. Skenario Merged Telco

Dalam merged telco, OSMOHLR dapat digunakan untuk:

  1.  Menyatukan dua jaringan operator dalam satu HLR/HSS.
  2.  Mengelola roaming nasional antar jaringan.
  3.  Menangani subscriber migrasi dari satu operator ke yang lain.

Skenario Merged Telco dapat diuji dengan:

  *  Menambahkan IMSI dari kedua jaringan ke OSMOHLR.
  *  Mengatur routing SS7/Diameter agar subscriber bisa berpindah jaringan dengan seamless.
  *  Menggunakan MAP atau Diameter untuk melakukan update location antar HLR/HSS.

## A Apa Itu Skenario Merged Telco?

Skenario Merged Telco adalah ketika dua atau lebih operator telekomunikasi bergabung atau bekerja sama dalam infrastruktur yang sama. Dalam kasus ini, OSMOHLR bertindak sebagai HLR/HSS terpusat untuk menangani subscriber dari berbagai operator, memungkinkan:

  1.  Interoperabilitas antar jaringan.
  2.  Roaming nasional antar operator.
  3.  Migrasi pelanggan dari satu operator ke operator lain tanpa perubahan besar di jaringan.


## B Arsitektur OSMOHLR dalam Merged Telco

Dalam arsitektur Merged Telco, OSMOHLR harus bisa:

  1.  Menangani database pelanggan dari kedua operator dalam satu HLR/HSS.
  2.  Berkomunikasi dengan dua MSC/VLR atau dua MME/SGSN dari operator yang berbeda.
  3.  Mengelola Update Location agar pelanggan bisa berpindah antara jaringan operator dengan seamless.
  4.  Menyediakan koneksi SS7/SIGTRAN atau Diameter ke jaringan inti masing-masing operator.


+----------------------+
|     OSMOHLR         |
|   (HLR/HSS Terpusat) |
+----------------------+
      |         |
      |         |
---------------------------------
|                               |
|                               |
V                               V
Operator A (MSC/VLR)   Operator B (MSC/VLR)
Operator A (MME/SGSN)  Operator B (MME/SGSN)

Dalam skenario ini, OSMOHLR mengelola subscriber dari kedua operator dan meneruskan sinyal ke jaringan masing-masing.

## C Konfigurasi OSMOHLR untuk Merged Telco

Untuk memungkinkan OSMOHLR menangani pelanggan dari beberapa operator, kita perlu:
##A. Menambahkan Subscriber dari Kedua Operator

OSMOHLR menggunakan database SQLite untuk menyimpan subscriber. Kita bisa menambahkan pelanggan dari operator A dan B dengan perintah:
```
osmo-hlr -c /etc/osmocom/osmo-hlr.cfg
```
Kemudian, tambahkan IMSI pelanggan:
```
sqlite3 /var/lib/osmocom/hlr.db "INSERT INTO subscriber (imsi, msisdn, ki, opc, amf) VALUES ('001010123456789', '6281234567890', '00112233445566778899AABBCCDDEEFF', 'E56A3FCD4E5731D2345F67789ABCD123', '8000');"
```
Lakukan hal yang sama untuk pelanggan dari operator lain.

##B. Menghubungkan OSMOHLR ke Dua Jaringan Operator

Untuk komunikasi dengan dua jaringan yang berbeda, kita harus memastikan OSMOHLR bisa berbicara dengan dua MSC/VLR atau MME/SGSN.
1️⃣ Jika menggunakan SS7/SIGTRAN

  *  Konfigurasi Global Title (GT) agar OSMOHLR bisa berbicara dengan dua MSC/VLR.
  *  Konfigurasi /etc/osmocom/osmo-stp.cfg untuk mendukung routing ke dua jaringan:

```
cs7-instance 0
 point-code 1234
 asp asp-clnt-1 2905 1
 route 1 2.3.4.5
 route 2 5.6.7.8
```
Di sini, 2.3.4.5 dan 5.6.7.8 adalah MSC dari dua operator yang berbeda.
2️⃣ Jika menggunakan Diameter untuk jaringan 4G/5G

  *  Pastikan OSMOHLR memiliki koneksi Diameter ke dua MME/HSS.
  *  Gunakan Diameter proxy untuk mengarahkan trafik ke operator yang benar.

Contoh konfigurasi Diameter di /etc/freediameter/freeDiameter.conf:
```
ConnectPeer = "mme.operatorA.com" { ConnectPort = 3868; No_TLS; };
ConnectPeer = "mme.operatorB.com" { ConnectPort = 3868; No_TLS; };
```

## D. Pengujian dan Validasi

Setelah konfigurasi, lakukan pengujian berikut:
✅ 1. Uji Registrasi Pelanggan dari Kedua Operator

Gunakan perintah berikut untuk memastikan OSMOHLR menangani subscriber dari kedua operator:
```
osmo-hlr -s
```
Jika berhasil, seharusnya muncul daftar pelanggan dari Operator A dan Operator B.
✅ 2. Uji Koneksi SS7/SIGTRAN ke Dua MSC

Gunakan perintah berikut untuk memeriksa apakah OSMOHLR terhubung ke dua MSC:
```
osmo-stp -s
```
Jika berhasil, akan terlihat log bahwa pesan MAP dikirim ke MSC yang benar.
✅ 3. Uji Update Location Antar Operator

Untuk menguji apakah pelanggan bisa berpindah jaringan dengan seamless:

  *  Pastikan IMSI pelanggan ada di database OSMOHLR.
  *  Jalankan debug SS7 dengan Wireshark:

     ```
     tshark -i any -Y "sccp || map || diameter"
     ```
  *  Dari MSC/VLR Operator A, kirim Update Location untuk IMSI pelanggan dari Operator B.
  *  Jika berhasil, pesan MAP Update Location akan diteruskan ke MSC Operator B.

✅ 4. Uji Panggilan dan SMS Antar Operator

  *  Coba lakukan panggilan antar pelanggan dari operator A dan B.
  *  Kirim SMS antara pelanggan dari operator yang berbeda.
  *  Jika berhasil, berarti OSMOHLR telah menangani merged telco dengan baik.

   5. Manfaat dan Tantangan
🎯 Manfaat:

✅ Pengelolaan subscriber terpusat, mengurangi biaya operasional.
✅ Roaming nasional lebih mudah, memungkinkan pelanggan berpindah jaringan dengan seamless.
✅ Dukungan untuk 2G/3G (SS7) dan 4G/5G (Diameter) dalam satu sistem.
⚠️ Tantangan:

⚠️ Interoperabilitas SS7 dan Diameter bisa menjadi kompleks, terutama jika dua operator menggunakan protokol yang berbeda.
⚠️ Manajemen database subscriber harus dilakukan dengan hati-hati agar tidak ada duplikasi atau konflik IMSI.
⚠️ Regulasi dan perjanjian antar operator harus diselesaikan sebelum implementasi skenario merged telco.



Kesimpulan

Dengan konfigurasi yang tepat, OSMOHLR dapat digunakan sebagai solusi merged telco, memungkinkan dua operator untuk berbagi satu infrastruktur HLR/HSS.

🚀 Langkah selanjutnya:
Apakah ingin mengintegrasikan fitur VoLTE atau SMS over IMS dalam jaringan merged telco ini
