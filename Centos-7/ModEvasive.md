
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

```
yum install -y mod_evasive mod_security
```

### 🔧 Langkah 3: Buat Direktori dan Halaman Index
```
mkdir -p /var/www/html
nano /var/www/html/index.html
```
Isi file index.html bisa sederhana, misalnya:
```
<h1>Testing mod_evasive</h1>
```
### 🔧 Langkah 4: Jalankan dan Aktifkan Apache
```
systemctl start httpd
systemctl enable httpd
```

### 🔧 Langkah 5: Nonaktifkan Sementara SELinux
```
setenforce 0
```

---

## 🔨 Simulasi Serangan DoS (Testing mod\_evasive)

### ✍️ Buat Script Perl `dos.pl`

```bash
nano dos.pl
```

> Contoh isi script sederhana (ubah `TARGET_IP` ke IP server Apache kamu):

```perl
#!/usr/bin/perl
use IO::Socket;
use strict;

my $target = "TARGET_IP";
my $port = 80;

for(my $i=0; $i<1000; $i++) {
    my $sock = IO::Socket::INET->new(PeerAddr => $target,
                                     PeerPort => $port,
                                     Proto    => 'tcp') or next;
    print $sock "GET / HTTP/1.1\r\nHost: $target\r\n\r\n";
    close($sock);
}
```

### ▶️ Jalankan Script

```bash
perl dos.pl
```

---

## ⚙️ Konfigurasi mod\_evasive dan mod\_security

### ✍️ Tambahkan Baris Berikut ke Apache Config

#### `mod_evasive.conf`

```apache
<IfModule mod_evasive20.c>
    DOSHashTableSize    3097
    DOSPageCount        4
    DOSSiteCount        50
    DOSPageInterval     1
    DOSSiteInterval     1
    DOSBlockingPeriod   10
</IfModule>
```

#### tambah kan ini pada configurasi mod_evasive.conf dan mod_security.conf:

```apache
LoadModule evasive20_module modules/mod_evasive24.so
LoadModule security2_module modules/mod_security2.so
```

---

## 🔁 Tes Ulang Setelah Konfigurasi

1. **Restart Apache**

```bash
systemctl restart httpd
```

2. **Jalankan lagi script `dos.pl`**

```bash
perl dos.pl
```

### ✅ Hasil:

* IP yang sebelumnya bisa spam sekarang akan diblokir.
* Server akan mengembalikan respons `403 Forbidden`.

---

## 🧠 Catatan Penting

* Untuk **pemblokiran permanen**, integrasikan dengan `iptables` atau Fail2Ban.
* File log mod\_evasive biasanya berada di `/var/log/httpd/mod_evasive.log`.
* Email notifikasi dapat diatur melalui konfigurasi tambahan.

---

## 📚 Referensi

* [Apache mod\_evasive Official](https://github.com/jzdziarski/mod_evasive)
* [RedHat/CentOS Docs](https://access.redhat.com/)


