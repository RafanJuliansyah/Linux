

# 🛡️ Panduan Lengkap Tiap Langkah - Langkah Keamanan Port Server

---

## ✅ 1. `sudo ss -tuln`

### 📌 Fungsi:

Melihat port apa saja yang sedang **mendengarkan koneksi masuk (listening)**.

### 🧪 Contoh Output:

```bash
Netid  State   Recv-Q  Send-Q   Local Address:Port    Peer Address:Port
tcp    LISTEN  0       128      0.0.0.0:22            0.0.0.0:*
tcp    LISTEN  0       128      127.0.0.1:5432        0.0.0.0:*
tcp    LISTEN  0       128      0.0.0.0:80            0.0.0.0:*
```

### 🧠 Penjelasan:

- `0.0.0.0:22` → port SSH (default)
    
- `127.0.0.1:5432` → hanya bisa diakses dari localhost (PostgreSQL)
    
- `0.0.0.0:80` → port HTTP (web server) bisa diakses dari mana saja
    

---

## 🔒 2. Set Default Policy Firewall

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

### 📌 Fungsi:

- Menolak **semua koneksi masuk** secara default
    
- Mengizinkan **semua koneksi keluar** (supaya server bisa update, akses internet, dsb.)
    

🧠 **Prinsip penting**:

> "Block everything, then allow what’s needed." 🔐

---

## ✅ 3. Buka Port-Penting Saja

### A. Buka Port SSH Default

```bash
sudo ufw allow ssh
```

Alias untuk:

```bash
sudo ufw allow 22/tcp
```

### B. Kalau Ganti Port SSH

```bash
sudo ufw allow 2222/tcp
```

> Gunakan ini kalau kamu ubah port SSH dari 22 ke 2222.

### C. Buka Port Web (HTTP & HTTPS)

```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

> Untuk menjalankan website dan SSL.

---

## 🟢 4. Aktifkan Firewall

```bash
sudo ufw enable
```

📌 **Fungsi:** Mengaktifkan semua aturan `ufw`.

🧠 Jangan panik kalau koneksi putus saat aktifin — pastikan port SSH kamu sudah di-_allow_ dulu sebelum enable ya!

---

## 🔍 5. Cek Status Firewall

```bash
sudo ufw status verbose
```

### 📌 Fungsi:

Melihat **semua aturan aktif** saat ini.

### 🧪 Contoh Output:

```bash
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing)
To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
443/tcp                    ALLOW       Anywhere
```

🧠 Artinya:

- Port 22, 80, dan 443 diizinkan dari semua IP
    
- Semua port lainnya ditolak secara default
    

---

## ❌ 6. Blokir Port Spesifik (Misalnya 80/tcp)

```bash
sudo ufw deny 80/tcp
```

📌 **Fungsi:** Menolak koneksi ke port 80. Cocok kalau web server kamu udah gak aktif dan ingin menutup akses dari luar.

### 🔁 Tes Hasilnya:

```bash
sudo ufw status verbose
```

Hasil:

```bash
80/tcp                     DENY        Anywhere
```

🧪 Scan pakai Nmap dari client:

```bash
nmap 172.16.3.50
```

Sebelum deny:

```
80/tcp open  http
```

Setelah deny:

```
80/tcp filtered
```

> Ini menunjukkan firewall menyaring koneksi ke port tersebut.

---

## 🔁 7. Hapus Aturan Tertentu

```bash
sudo ufw delete allow 80/tcp
```

📌 **Fungsi:** Menghapus aturan `allow` tanpa mencatat sebagai `deny`.

🧠 Biasanya digunakan untuk bersih-bersih aturan yang tidak perlu lagi.

---

## 🔄 8. Ganti Port SSH Default

### A. Edit Config SSH:

```bash
sudo nano /etc/ssh/sshd_config
```

Ubah:

```
#Port 22
```

Jadi:

```
Port 2222
```

### B. Restart SSH

```bash
sudo systemctl restart ssh
```

### C. Update Firewall:

```bash
sudo ufw allow 2222/tcp
sudo ufw delete allow 22/tcp
```

🧠 Tujuannya:

- Port 22 adalah target umum brute-force.
    
- Dengan mengganti port, kamu **menyembunyikan SSH** dari scanner otomatis.
    

---

## 🛡️ 9. Pasang Fail2Ban

```bash
sudo apt install fail2ban -y
sudo systemctl enable --now fail2ban
```

### 🔍 Cek Status Jail SSH:

```bash
sudo fail2ban-client status sshd
```

🧠 Fail2Ban akan otomatis:

- Memantau file log SSH
    
- Blokir IP yang mencoba login terus-menerus (brute-force attack)
    

---

# 📌 Rangkuman Fungsi UFW Paling Penting

|Perintah|Fungsi|
|---|---|
|`sudo ufw allow 80/tcp`|Buka port HTTP (web server)|
|`sudo ufw deny 80/tcp`|Blokir port HTTP|
|`sudo ufw delete allow 80/tcp`|Hapus aturan allow port HTTP|
|`sudo ufw default deny incoming`|Tolak semua koneksi masuk secara default|
|`sudo ufw default allow outgoing`|Izinkan semua koneksi keluar|
|`sudo ufw enable`|Aktifkan firewall|
|`sudo ufw status verbose`|Lihat semua aturan firewall dengan detail|

---

