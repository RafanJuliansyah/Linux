## **🧾 Soal :**
Anda diminta melakukan konfigurasi dan pengamanan dasar terhadap jaringan server
Ubuntu agar tidak mudah diserang dari luar. Fokus Anda adalah mengontrol
akses layanan dan port, serta mencegah pemindaian jaringan yang agresif.

Pasang firewall dan konfigurasi rule hanya untuk:
- SSH (port 2202)
- HTTP (port 80)
- HTTPS (port 443)
  Semua port selain itu harus ditutup.
- Nonaktifkan ICMP/ping agar tidak bisa diping dari luar.
- Pasang dan aktifkan TCP Wrappers (hosts.deny dan hosts.allow)
- Tolak semua default dari ALL
- Hanya izinkan akses SSH dari ip komputer
- Pasang dan jalankan psad atau portsentry
- Untuk mendeteksi port scan

--- 

### 📌 Langkah 1: Izinkan Hanya Port yang Diperlukan (SSH 2202, HTTP 80, HTTPS 443)

```bash
sudo apt update
```

- Memperbarui daftar paket dari repository agar sistem tahu versi terbaru dari setiap paket yang tersedia.
    

```bash
sudo apt install ufw -y
```

- Menginstal firewall UFW dan langsung menyetujui instalasi tanpa konfirmasi manual.
    

```bash
sudo ufw enable
```

- Mengaktifkan firewall agar langsung berjalan dan mulai memfilter koneksi masuk dan keluar.
    

```bash
sudo ufw default deny incoming
```

- Menolak semua koneksi masuk secara default agar tidak ada layanan yang dapat diakses dari luar kecuali diizinkan secara eksplisit.
    

```bash
sudo ufw default allow outgoing
```

- Mengizinkan semua koneksi keluar secara default agar server tetap dapat mengakses internet.
    

```bash
sudo ufw allow 2202/tcp
```

- Mengizinkan port 2202 (SSH custom) untuk koneksi masuk menggunakan protokol TCP.
    

```bash
sudo ufw allow 80/tcp
```

- Membuka akses HTTP agar server bisa diakses menggunakan protokol web biasa.
    

```bash
sudo ufw allow 443/tcp
```

- Membuka akses HTTPS agar server bisa diakses dengan koneksi aman menggunakan SSL/TLS.
    

---

### 📌 Langkah 2: Nonaktifkan ICMP (Ping)

```bash
sudo nano /etc/ufw/before.rules
```

- Membuka file konfigurasi awal UFW untuk menambahkan aturan khusus sebelum aturan utama dijalankan.
    

Tambahkan baris ini sebelum bagian `*filter`:

```
-A ufw-before-input -p icmp --icmp-type echo-request -j DROP
```

- Baris ini menolak semua paket ICMP jenis echo-request (yang digunakan untuk ping), agar server tidak merespons permintaan ping dari luar.
    

```bash
sudo ufw enable
sudo ufw reload
```

- Memuat ulang firewall agar aturan baru mengenai ICMP mulai diterapkan.
	

```
sudo iptables -L | grep icmp
```

- lalu selanjutnya kita coba cek apakah berhasil atau tidak

Output:
```
REJECT all -- anywhere anywhere reject-with icmp-port-unreachable
```

Menunjukkan bahwa:

- Semua permintaan ke port yang tidak dibuka akan ditolak dengan balasan `icmp-port-unreachable` (default dari UFW).
    
- **Tidak ada lagi aturan `ACCEPT icmp echo-request`** → artinya permintaan ping (`ping`/`ICMP echo-request`) sudah **tidak diterima**.
    

---

### 📌 Langkah 3: Konfigurasi TCP Wrappers

```bash
sudo nano /etc/hosts.deny
```

- Membuka file konfigurasi untuk menentukan koneksi mana saja yang harus ditolak secara default.
    

Isi file:

```
ALL: ALL
```

- Menolak semua koneksi ke semua layanan untuk semua alamat IP secara default.
    

```bash
sudo nano /etc/hosts.allow
```

- Membuka file konfigurasi pengecualian untuk koneksi yang diizinkan.
    

Isi file:

```
sshd: 192.168.1.100
```

- Mengizinkan hanya alamat IP lokal `192.168.1.100` untuk mengakses layanan SSH. untuk IP nya sendiri bisa kalian sesuaikan dengan yang kalian mau
    

> ⚠️ Catatan: Hanya layanan yang mendukung `libwrap` seperti `sshd`, `vsftpd`, `telnetd` yang bisa dikontrol dengan TCP Wrappers.

---

### 📌 Langkah 4: Ubah Port SSH Menjadi 2202

```bash
sudo nano /etc/ssh/sshd_config
```

- Membuka file konfigurasi utama SSH untuk mengganti port dan meningkatkan keamanan.
    

Edit atau pastikan baris berikut:

```
Port 2202
```

- Mengubah port default SSH dari 22 ke 2202 agar tidak mudah dipindai oleh attacker.
    

```
PermitRootLogin no
```

- Melarang login langsung sebagai root untuk mengurangi risiko kompromi akun administrator.
    

```
PasswordAuthentication yes
```

- Mengaktifkan metode login menggunakan password (bisa disesuaikan dengan metode key-based authentication jika lebih aman).
    

```bash
sudo systemctl restart ssh
```

- Me-restart layanan SSH agar perubahan konfigurasi diterapkan.
    

---

### 📌 Langkah 5: Install & Konfigurasi PSAD (Port Scan Attack Detector)

```bash
sudo apt install psad -y
```

- Menginstal PSAD, tool yang menganalisis log firewall (iptables/UFW) untuk mendeteksi pola port scanning.
    

```bash
sudo ufw logging on
```

- Mengaktifkan logging UFW agar PSAD dapat membaca dan menganalisis lalu lintas masuk.
    

```bash
sudo nano /etc/ufw/before.rules
```

- Membuka file rule UFW untuk menambahkan aturan log secara eksplisit.
    

Tambahkan sebelum baris `COMMIT`:

```
-A INPUT -j LOG
-A FORWARD -j LOG
```

- Menambahkan aturan logging pada seluruh koneksi masuk (INPUT) dan koneksi forwarding (FORWARD).
>⚠️ Catatan:  jika ada error di line 11 tinggal kalian command saja pada baris line 11

    

```bash
sudo ufw enable
sudo ufw reload
```

- Memuat ulang konfigurasi UFW agar aturan logging aktif.
    

```bash
sudo nano /etc/psad/psad.conf
```

- Membuka file konfigurasi PSAD untuk mengatur notifikasi email dan nama host.
    

Edit baris berikut:

```
EMAIL_ADDRESSES your@email.com
HOSTNAME your-hostname
ENABLE_PSAD_EMAIL_ALERTS Y
```

- `EMAIL_ADDRESSES`: Alamat email tujuan untuk peringatan jika terdeteksi scanning.
    
- `HOSTNAME`: Nama host server yang muncul di laporan.
    
- `ENABLE_PSAD_EMAIL_ALERTS`: Mengaktifkan fitur pengiriman email peringatan.
    

```bash
sudo psad -R
```

- Me-reload konfigurasi PSAD setelah disimpan.
    

```bash
sudo psad --sig-update
```

- Mengupdate signature database agar mendeteksi jenis scanning terbaru.
    

```bash
sudo psad -H
```

- Menjalankan daemon PSAD agar aktif dan mulai memantau log UFW.
    

---

### 📌 Verifikasi Konfigurasi

```bash
sudo ufw status verbose
```

- Menampilkan status firewall dan port-port mana yang sedang terbuka.
    

```bash
sudo psad --Status
```

- Menampilkan status PSAD, termasuk apakah ada aktivitas port scan yang terdeteksi.