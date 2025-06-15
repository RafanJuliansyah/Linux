## **🧾 Soal :**

Anda adalah seorang administrator sistem baru di sebuah organisasi.
Tugas Anda adalah melakukan hardening dasar terhadap sistem Ubuntu Server 22.04
yang baru saja terinstall. Sistem ini belum dilindungi dan rawan terhadap eksploitasi umum.

Langkah-langkah hardening yang harus Anda lakukan:

1. Nonaktifkan login root via SSH
2. Ganti port default SSH ke 2202
3. Aktifkan firewall dan hanya izinkan port:
   - 2202 (SSH baru)
   - 80 (HTTP)
   - 443 (HTTPS)
4. Nonaktifkan dan matikan layanan cups dan avahi-daemon (jika ada)
5. Buat user baru bernama adminlks dengan hak sudo tanpa password
6. Pasang dan aktifkan fail2ban untuk SSH
7. Aktifkan auditd dan pastikan log audit berjalan

---

## 🔐 **1. Nonaktifkan Login Root via SSH**

### 📌 Perintah:

```
sudo nano /etc/ssh/sshd_config
```

- Perintah ini menjalankan editor teks `nano` dengan hak akses **superuser (sudo)** untuk mengedit file konfigurasi **SSH server**.
    
- File `/etc/ssh/sshd_config` berisi pengaturan utama untuk bagaimana server SSH (`sshd`) berjalan dan merespons koneksi masuk.
    

### 📌 Ubah/aktifkan baris berikut:

```
PermitRootLogin no
```

- Baris ini **melarang login langsung sebagai user root** melalui SSH.
    
- Hal ini sangat penting karena root adalah akun dengan hak tertinggi dan sering menjadi target brute-force attack.
    

📷 **Screenshot hasil:**

![Screenshot 2025-06-14 131206](https://github.com/user-attachments/assets/3c08839f-ac80-4276-9dc8-703b3c8251b0)

### 📌 Terapkan perubahan:

```
sudo systemctl restart ssh
```

- Perintah ini **me-restart service SSH**, sehingga perubahan di konfigurasi tadi akan langsung diterapkan.
    

📷 **Screenshot hasil:**

![[Screenshot 2025-06-14 131524 1.png]]

---

## 🔐 **2. Ganti Port Default SSH ke 2202**

### 📌 Perintah:

```
sudo nano /etc/ssh/sshd_config
```

- Kembali mengedit file konfigurasi SSH untuk menyesuaikan port akses.
    

### 📌 Tambahkan atau ubah baris ini:

```bash
Port 2202
```

- Artinya, SSH server akan mendengarkan koneksi pada **port 2202** alih-alih default port 22.
    
- Port ini digunakan agar server tidak mudah dideteksi oleh port scanner yang menargetkan port default.
    

📷 **Screenshot hasil:**

![[Screenshot 2025-06-14 131605.png]]
### 📌 Terapkan perubahan:

```
sudo systemctl restart ssh
```

- Perlu dilakukan agar **konfigurasi port baru** langsung aktif.
    

📷 **Screenshot hasil:**
###### Status sebelum di restart
![[Screenshot 2025-06-14 131652.png]]
###### Status sesuad di restart
![[Screenshot 2025-06-14 131713.png]]

---

## 🧪 **Uji Akses SSH ke Port Baru**

### 📌 Perintah:

```
ssh -p 2202 ubuntu@ip-server
```

- Flag `-p 2202` memberitahu client SSH agar menyambung ke **port 2202**, bukan port default (22).
    
- `user@ip-server` adalah **user** yang akan login dan **alamat IP/server** tujuan.
    

📷 **Screenshot hasil:**

![[Screenshot 2025-06-14 131926.png]]

---

## 🔐 **3. Aktifkan Firewall (UFW) dan Izinkan Port Penting**

### 📌 Perintah:

```
sudo ufw allow 2202/tcp
```

- Membuka port 2202 untuk protokol TCP (digunakan oleh SSH).
    
- UFW adalah firewall default di Ubuntu yang mudah dikonfigurasi.
    

```
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

- Port 80 (HTTP) dan 443 (HTTPS) dibuka untuk akses web normal jika server menjalankan aplikasi web.
    

```
sudo ufw enable
```

- Mengaktifkan UFW secara permanen.
    
- Semua aturan yang telah diatur sebelumnya akan mulai diterapkan.
    

📷 **Screenshot hasil:**

![[Screenshot 2025-06-14 132840.png]]

---

## 🔧 **4. Nonaktifkan `cups` dan `avahi-daemon` (Jika Ada)**

### 📌 Perintah:

```
sudo systemctl stop cups
```

- Menghentikan layanan **Common Unix Printing System (CUPS)**, tidak dibutuhkan di server.
    

```
sudo systemctl disable cups
```

- Menonaktifkan agar layanan **tidak otomatis aktif** saat booting.
    

```
sudo systemctl stop avahi-daemon
sudo systemctl disable avahi-daemon
```

- `avahi-daemon` adalah layanan jaringan lokal (mDNS) yang juga tidak perlu di server produksi.
    

---

## 👤 **5. Buat User `adminlks` dengan Sudo Tanpa Password**

### 📌 Perintah:

```
sudo adduser adminlks
```

- Membuat akun baru bernama `adminlks` dengan folder home dan shell login.
    

```
sudo usermod -aG sudo adminlks
```

- Menambahkan user ke grup `sudo` sehingga punya hak istimewa administratif.
    

```
sudo visudo
```

- Digunakan untuk mengedit file `/etc/sudoers` secara **aman** (mencegah kesalahan fatal).
    
- Di dalamnya, tambahkan:
    

```bash
adminlks ALL=(ALL) NOPASSWD:ALL
```

- User `adminlks` bisa menjalankan semua perintah `sudo` **tanpa harus memasukkan password**.
    

📷 **Screenshot hasil:**

![[Screenshot 2025-06-14 133442.png]]

> jika saat sudo su masih di mintai password coba nonaktifan dengan command **#** ini pada visudo command *%sudo ALL=(ALL:ALL) ALL_* 

---

## 🛡️ **6. Pasang dan Aktifkan Fail2Ban untuk SSH**

### 📌 Perintah:

```
sudo apt install fail2ban -y
```

- Menginstal tool keamanan yang secara otomatis memblokir IP yang gagal login berulang kali.
    

```
cd /etc/fail2ban
sudo cp jail.conf jail.local
```

- Menyalin file konfigurasi utama sebagai `jail.local`, yang akan digunakan agar perubahan tidak hilang saat update.
    

```
sudo nano jail.local
```

- Edit bagian `[sshd]` agar konfigurasi proteksi SSH diaktifkan:
    

```bash
[sshd]
enabled = true
port = ssh
logpath = %(sshd_log)s
maxretry = 5
backend = systemd
```

- `enabled = true`: aktifkan proteksi
    
- `maxretry = 5`: blokir IP jika gagal login 5 kali
    
- `backend = systemd`: gunakan log systemd (`journalctl`)
    

```
sudo systemctl enable fail2ban
sudo systemctl restart fail2ban
```

- Aktifkan dan jalankan `fail2ban`.
    

```
sudo systemctl status fail2ban
```

- Pastikan fail2ban aktif dan tidak error.
    

```
sudo fail2ban-client status
sudo fail2ban-client status sshd
```

- Menampilkan **status proteksi global** dan **spesifik SSH**, termasuk IP yang diblokir.
    

---

## 📝 **7. Aktifkan `auditd` dan Cek Aktivitas**

### 📌 Perintah:

```
sudo apt install auditd -y
```

- Menginstal **audit daemon** bawaan Linux untuk log aktivitas penting.
    

```
sudo systemctl enable auditd --now
```

- Mengaktifkan dan langsung menjalankan `auditd`.
    

### 📌 Periksa Aktivitas Audit:

```
last
```

- Menampilkan **riwayat login pengguna**.
    

```
journalctl -u ssh
```

- Menampilkan **log layanan SSH**, termasuk login gagal.
    

```
ausearch -x sudo
```

- Mencari log semua aktivitas `sudo`.
    

```
aureport -au
```

- Ringkasan laporan autentikasi dari `auditd`.
    

---
