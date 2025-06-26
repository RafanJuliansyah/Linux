## **🧾 Soal :**
Anda bertugas sebagai analis keamanan yang diminta melakukan investigasi digital pada sistem
Ubuntu yang dicurigai telah diserang. Tugas Anda adalah menemukan bukti, seperti file log,
proses mencurigakan, dan aktivitas akses file, lalu membuat laporan temuan.

✅ Instruksi Kasar:
- Cari dan tampilkan log login terakhir menggunakan last dan lastb.
- Analisa file log auth.log di /var/log untuk aktivitas login yang mencurigakan (misal: percobaan login gagal berulang).
- Cari file-file yang diubah atau dibuat dalam 24 jam terakhir di file anda.
- Periksa proses berjalan yang tidak biasa atau mencurigakan (ps aux + grep).
- Cari file yang memiliki permission tidak biasa (misal: file skrip yang bisa dijalankan oleh semua orang).
Buat laporan singkat (maks 1 paragraf) berisi ringkasan hasil analisis dan rekomendasi tindakan keamanan.


---

## ✅ 1. Pemeriksaan Log Login (`lastlog.txt`)

**Isi file `lastlog.txt`:**

```
Username         Port     From             Latest
root             pts/0    192.168.56.1     Wed Jun  4 09:12:10 +0700 2025
user1            pts/1    192.168.56.10    Wed Jun  4 08:43:05 +0700 2025
user1            pts/2    192.168.56.12    Wed Jun  3 22:43:05 +0700 2025
user3            pts/0    10.0.2.15        Wed May 29 19:05:45 +0700 2025
```

📌 **Analisis:**

- `user1` terlihat login dari **dua IP berbeda** dalam kurun waktu semalam.
    
- Akun `root` login terakhir dari IP `192.168.56.1`.
    
- Ada satu entri lama dari `user3` — perlu dicek apakah akun ini masih aktif atau tidak.
    

---

## 🚫 2. Pemeriksaan Log Kegagalan Login (`lastb.txt`)

**Isi file `lastb.txt`:**

```
user1    tty1     192.168.56.100    Wed Jun  4 08:10 - 08:10  (00:00)
user1    tty1     192.168.56.100    Wed Jun  4 08:09 - 08:09  (00:00)
user1    tty1     192.168.56.100    Wed Jun  4 08:08 - 08:08  (00:00)
user1    tty1     192.168.56.100    Wed Jun  4 08:07 - 08:07  (00:00)
user1    tty1     192.168.56.100    Wed Jun  4 08:06 - 08:06  (00:00)
```

📌 **Analisis:**

- Terjadi **5 kali percobaan login gagal** oleh `user1` dari IP `192.168.56.100` dalam waktu sangat berdekatan (indikasi brute force attack).
    
- Semua terjadi pada pagi hari 4 Juni 2025.
    

---

## 🔐 3. Analisis File Log `/var/log/auth.log`

**Cuplikan isi file `auth.log`:**

```
Jun  4 08:06:01 ubuntu-server sshd[1221]: Failed password for user1 from 192.168.56.100 port 55234 ssh2
Jun  4 08:07:13 ubuntu-server sshd[1225]: Failed password for user1 from 192.168.56.100 port 55235 ssh2
Jun  4 08:08:33 ubuntu-server sshd[1230]: Failed password for user1 from 192.168.56.100 port 55236 ssh2
Jun  4 08:09:15 ubuntu-server sshd[1237]: Failed password for user1 from 192.168.56.100 port 55237 ssh2
Jun  4 08:10:44 ubuntu-server sshd[1243]: Failed password for user1 from 192.168.56.100 port 55238 ssh2
Jun  4 08:12:01 ubuntu-server sshd[1249]: Accepted password for user1 from 192.168.56.100 port 55239 ssh2
```

📌 **Analisis:**

- Terdapat 5 kali "Failed password" sebelum akhirnya berhasil login.
    
- Pola waktu dan IP menunjukkan brute force **berhasil** menembus akun `user1`.
    

---

## 🧪 4. Pemeriksaan Proses Berjalan (`proses.txt`)

**Isi file `proses.txt`:**

```
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.1  22596  4100 ?        Ss   07:45   0:01 /sbin/init
user1     1123  0.0  0.3  92840  6020 ?        S    08:43   0:00 /usr/bin/python3 suspicious.py
user1     1150  0.1  0.1  47532  1300 ?        S    08:45   0:00 ./suspicious.sh
user3     1200  0.0  0.2  63500  3000 ?        S    08:46   0:00 /usr/bin/nano
```

📌 **Analisis:**

- `user1` menjalankan 2 file mencurigakan: `suspicious.py` dan `suspicious.sh`.
    
- Proses tersebut **tidak umum**, bisa jadi script otomatisasi atau malware.
    
- Perlu dilakukan analisis lanjutan isi file dan siapa yang membuatnya.
    

---

## 🧾 5. Laporan Singkat (Ringkasan & Rekomendasi)

📋 **Ringkasan Temuan:**  
Telah terjadi serangan brute-force SSH terhadap akun user1 dari IP 192.168.56.100 yang berujung pada login berhasil. Setelah login, user tersebut menjalankan dua skrip mencurigakan (suspicious.py dan suspicious.sh). Hal ini menunjukkan sistem telah berhasil diakses oleh penyerang dan dimungkinkan sedang dieksploitasi lebih lanjut. Tidak ditemukan aktivitas login mencurigakan oleh user lain, namun perlu investigasi lanjutan pada file yang dimodifikasi serta skrip yang dijalankan.

🛡️ **Rekomendasi Tindakan:**

- Nonaktifkan akun `user1` untuk investigasi:
    
    ```bash
    sudo usermod -L user1
    ```
    
- Cek isi file `suspicious.py` dan `suspicious.sh`:
    
    ```bash
    cat suspicious.py
    cat suspicious.sh
    ```
	
- Hapus proses dan file skrip mencurigakan (suspicious.py, suspicious.sh).
	
- Instal dan aktifkan `fail2ban` untuk mencegah brute force SSH:
    
    ```bash
    sudo apt install fail2ban
    sudo systemctl enable --now fail2ban
    ```
    
- Audit permission file dan backup sistem log:
    
    ```bash
    find / -type f -perm -111 2>/dev/null
    cp /var/log/auth.log /home/ubuntu/backup/
    ```
    
- Ubah konfigurasi SSH:
  Nonaktifkan login password (PasswordAuthentication no).
  Gunakan login dengan SSH key.
	
- Periksa file permission seluruh direktori home dan script.
	

---


