## Konfigurasi Keamanan Akun Ubuntu
---

### 🔢 Langkah 1: Buat user `user1`

```bash
sudo useradd -m -s /bin/bash user1
```

🔍 Penjelasan:

- `sudo` → Menjalankan perintah sebagai administrator
    
- `useradd` → Perintah untuk menambahkan user baru
    
- `-m` → Membuat direktori home (`/home/user1`)
    
- `-s /bin/bash` → Menentukan shell default untuk login (bash)
    

✅ Output: Tidak ada output jika berhasil

---

### 🔢 Langkah 2: Atur password untuk `user1`

```bash
echo "user1:password123" | sudo chpasswd
```

🔍 Penjelasan:

- `echo` → Menyusun teks `user1:password123`
    
- `|` → Menyambungkan output `echo` ke input `chpasswd`
    
- `chpasswd` → Perintah untuk mengatur password user secara massal
    

✅ Output: Tidak ada output jika berhasil

---

### 🔢 Langkah 3: Buat user `user2`

```bash
sudo useradd -m -s /bin/bash user2
```

🔍 Penjelasan: Sama seperti langkah 1, tetapi untuk user2

✅ Output: Tidak ada output jika berhasil

---

### 🔢 Langkah 4: Atur password untuk `user2`

```bash
echo "user2:password123" | sudo chpasswd
```

🔍 Penjelasan: Sama seperti langkah 2, tetapi untuk user2

✅ Output: Tidak ada output jika berhasil

---

### 🔢 Langkah 5: Buat user `user3`

```bash
sudo useradd -m -s /bin/bash user3
```

🔍 Penjelasan: Sama seperti sebelumnya, digunakan untuk user3

✅ Output: Tidak ada output jika berhasil

---

### 🔢 Langkah 6: Atur password untuk `user3`

```bash
echo "user3:password123" | sudo chpasswd
```

🔍 Penjelasan: Sama seperti sebelumnya

✅ Output: Tidak ada output jika berhasil

---

### 🔢 Langkah 7: Buat user `user4`

```bash
sudo useradd -m -s /bin/bash user4
```

🔍 Penjelasan: Sama seperti sebelumnya

✅ Output: Tidak ada output jika berhasil

---

### 🔢 Langkah 8: Atur password untuk `user4`

```bash
echo "user4:password123" | sudo chpasswd
```

🔍 Penjelasan: Sama seperti sebelumnya

✅ Output: Tidak ada output jika berhasil

---

### 📌 [Sampai Sini] Semua user sudah dibuat dan memiliki password

- `user1` dan `user2` = nanti akan diaktifkan SSH
    
- `user3` dan `user4` = nanti akan diblok login SSH
    

---

### 🔢 Langkah 9: Tambahkan user yang boleh akses SSH (optional tapi disarankan)

Edit config SSH:

```bash
sudo nano /etc/ssh/sshd_config
```

🔍 Penjelasan:

- `nano` → Editor teks terminal
    
- File `sshd_config` berisi pengaturan layanan SSH
    

🎯 Cari bagian paling bawah file dan tambahkan ini:

```
AllowUsers user1 user2
```

💾 Simpan dengan `CTRL+O`, lalu `Enter`, keluar pakai `CTRL+X`

---

### 🔢 Langkah 10: Restart SSH

```bash
sudo systemctl restart ssh
```

🔍 Penjelasan:

- `systemctl` → Digunakan untuk mengelola service
    
- `restart ssh` → Memuat ulang konfigurasi SSH supaya perubahan aktif
    

✅ Output: Tidak ada output jika berhasil

---

## 🧪 Uji Coba SSH

Gunakan terminal dari mesin lain atau dari lokal:

```bash
ssh user1@<ip-address>
```

```bash
ssh user2@<ip-address>
```

✅ Jika konfigurasi benar, dua user ini bisa login.

---

### 🔢 Langkah 11: Nonaktifkan login `user3` dengan mengunci password-nya

```bash
sudo usermod -L user3
```

🔍 Penjelasan:

- `usermod` → Digunakan untuk mengubah konfigurasi user
    
- `-L` → Lock: mengunci password user di `/etc/shadow`
    
- Artinya: user tidak bisa login dengan password
    

✅ Output: Tidak ada output jika berhasil

---

### 🔢 Langkah 12: Nonaktifkan login `user4` dengan mengunci password-nya

```bash
sudo usermod -L user4
```

🔍 Penjelasan: Sama seperti langkah sebelumnya, tapi untuk `user4`

✅ Output: Tidak ada output jika berhasil

---

### 🔢 Langkah 13: Nonaktifkan shell login untuk `user3`

```bash
sudo chsh -s /usr/sbin/nologin user3
```

🔍 Penjelasan:

- `chsh` → Ganti shell default user
    
- `/usr/sbin/nologin` → Shell ini akan menolak login jika dicoba
    
- Efeknya: meskipun password terbuka, user tetap tidak bisa login karena shell-nya dummy
    

✅ Output: Tidak ada output jika berhasil

---

### 🔢 Langkah 14: Nonaktifkan shell login untuk `user4`

```bash
sudo chsh -s /usr/sbin/nologin user4
```

🔍 Penjelasan: Sama seperti langkah sebelumnya, tapi untuk `user4`

✅ Output: Tidak ada output jika berhasil

---

📌 Sampai tahap ini:

- `user3` dan `user4` telah **dikunci** dan shell-nya diubah menjadi `nologin`
    
- Mereka **tidak akan bisa login SSH**, bahkan jika tahu password-nya
    

---

## 🔍 Verifikasi Status Akun (Opsional tapi Disarankan)

---

### 🔢 Langkah 15: Cek status akun `user1`

```bash
sudo passwd -S user1
```

📥 Contoh Output:

```
user1 P 06/04/2025 0 99999 7 -1
```

🔍 Keterangan:

- `P` = password aktif
    
- Artinya `user1` bisa login
    

---

### 🔢 Langkah 16: Cek status akun `user3`

```bash
sudo passwd -S user3
```

📥 Contoh Output:

```
user3 L 06/04/2025 0 99999 7 -1
```

🔍 Keterangan:

- `L` = password terkunci (Locked)
    
- Artinya user tidak bisa login
    

---

### 🔢 Langkah 17: Cek shell dari semua user

```bash
grep '/home' /etc/passwd
```

📥 Contoh Output (potongan saja):

```
user1:x:1001:1001::/home/user1:/bin/bash
user2:x:1002:1002::/home/user2:/bin/bash
user3:x:1003:1003::/home/user3:/usr/sbin/nologin
user4:x:1004:1004::/home/user4:/usr/sbin/nologin
```

🔍 Keterangan:

- `user1` dan `user2` = masih pakai bash → bisa login
    
- `user3` dan `user4` = pakai `nologin` → tidak bisa login
    

---

## 🧪 Tes SSH Login User yang Diblok

---

### 🔢 Langkah 18: Tes SSH login sebagai `user3`

```bash
ssh user3@<ip-address>
```

📥 **Hasil yang Mungkin Muncul:**

1. 
```
This account is currently not available.
```
🔒 Ini terjadi jika shell user di-set ke `/usr/sbin/nologin`, artinya shell tidak bisa dipakai login. ✅

2. 
```
Permission denied, please try again.
```
🔐 Ini muncul jika password user dikunci dengan `usermod -L`, sehingga login akan selalu gagal walau password benar. ✅

🧠 Artinya:

- Shell `nologin` aktif → user tidak bisa login.
- Password dikunci → login gagal total.
- Kedua metode berhasil mencegah user masuk via SSH. 🔥

---

### 🔢 Langkah 19: Tes SSH login sebagai `user4`

```bash
ssh user4@<ip-address>
```

📥 **Hasil yang Mungkin Muncul:**

1. 
```
This account is currently not available.
```
🔒 Shell login diblok, user tidak bisa masuk.

2. 
```
Permission denied, please try again.
```
🔐 Password dikunci walaupun benar.

🧠 Sama seperti `user3`, akun `user4` benar-benar diblok dari akses SSH baik melalui shell maupun password. Ini menunjukkan bahwa metode pengamanan berhasil dengan sempurna. ✅
