
# 🛡️ Write-Up: mod_evasive di Apache (CentOS 7)

## 📌 Pengertian mod_evasive

**mod_evasive** adalah modul keamanan untuk Apache Web Server yang digunakan untuk mendeteksi dan merespons serangan seperti:

- DoS (Denial of Service)
- DDoS (Distributed Denial of Service)
- Brute-force attack

Modul ini memantau pola traffic HTTP mencurigakan seperti:
- Request berulang dalam waktu singkat dari IP yang sama
- Lonjakan akses ke halaman tertentu secara terus-menerus

> ⚙️ mod_evasive ringan, efisien, dan efektif sebagai lapisan pertahanan pertama sebelum serangan mencapai aplikasi backend atau database.

---

## 🎯 Fungsi Utama mod_evasive

1. **Mendeteksi Request Berulang**
   - Memantau frekuensi request dari IP yang sama dalam waktu singkat.

2. **Memblokir IP Sementara atau Permanen**
   - IP yang melebihi ambang batas request akan diblokir sementara (`DOSBlockingPeriod`) atau permanen dengan bantuan sistem eksternal seperti `iptables`.

3. **Mengirim Email Notifikasi**
   - Otomatis mengirim email saat ada IP yang diblokir.

4. **Mencatat Aktivitas ke File Log**
   - Semua pemblokiran dicatat, berguna untuk audit atau integrasi dengan sistem lain seperti Fail2Ban.

5. **Menjalankan Custom Command**
   - Bisa menjalankan perintah shell otomatis (contohnya untuk block IP dengan `iptables`).

---

## 🖥️ Instalasi mod_evasive di CentOS 7

### 🔧 Langkah 1: Install Paket Dasar

```
yum install -y httpd epel-release perl nc
```

### 🔧 Langkah 2: Install modul mod_evasive dan mod_security
