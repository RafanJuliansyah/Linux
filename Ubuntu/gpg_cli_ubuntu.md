
# 💻 Panduan Lengkap GPG via CLI di Linux Ubuntu

Dokumentasi ini membahas penggunaan GPG (GNU Privacy Guard) **murni melalui terminal (CLI)** khusus untuk pengguna **Ubuntu**. Cocok untuk sysadmin, hacker, atau siapa pun yang ingin mengamankan komunikasi dan file mereka tanpa GUI.

---

## 📦 1. Instalasi GPG di Ubuntu

```bash
sudo apt update
sudo apt install gnupg
```

Cek versi:
```bash
gpg --version
```

---

## 🔑 2. Membuat Keypair (Public & Private Key)

```bash
gpg --full-generate-key
```

### Langkah:
1. Pilih `(1) RSA and RSA (default)`
2. Masukkan panjang kunci: `3072` atau `4096`
3. Masukkan masa berlaku (misalnya `0` untuk selamanya)
4. Isi nama, email, dan comment
5. Masukkan passphrase untuk proteksi private key

---

## 🧾 3. Melihat Key

### Public key:
```bash
gpg --list-keys
```

### Private key:
```bash
gpg --list-secret-keys
```

---

## 📤 4. Mengekspor Key

### Public Key:
```bash
gpg --export -a "Nama Kamu" > publickey.asc
```

### Private Key (untuk backup pribadi):
```bash
gpg --export-secret-keys -a "Nama Kamu" > privatekey.asc
```

---

## 📥 5. Mengimpor Key

```bash
gpg --import namafile.asc
```

---

## 🔐 6. Enkripsi File

### Enkripsi file:
```bash
gpg -e -r "email@penerima.com" file.txt
```

### Enkripsi dan tanda tangan:
```bash
gpg -se -r "email@penerima.com" file.txt
```

---

## 🔓 7. Dekripsi File

```bash
gpg -d file.txt.gpg
```

### Simpan hasil decrypt:
```bash
gpg -o hasil.txt -d file.txt.gpg
```

---

## ✍️ 8. Tanda Tangan Digital

### Clear sign:
```bash
gpg --clearsign file.txt
```

---

## ✅ 9. Verifikasi Tanda Tangan

```bash
gpg --verify file.txt.asc
```

---

## 📂 10. Backup & Restore Seluruh Key

### Backup:
```bash
gpg --export -a > all-public.asc
gpg --export-secret-keys -a > all-private.asc
```

### Restore:
```bash
gpg --import all-public.asc
gpg --import all-private.asc
```

---

## 🧠 11. Tips Tambahan

- Gunakan kunci 4096-bit untuk keamanan maksimal
- Jangan kirim private key ke siapa pun
- Simpan key di flashdisk terenkripsi atau cloud privat
- Gunakan password manager untuk simpan passphrase & fingerprint

---

## 🔍 12. Cek Fingerprint

```bash
gpg --fingerprint "Nama Kamu"
```

---

## 🎯 Contoh: Backup Aman dengan GPG

### Enkripsi:
```bash
tar czf backup.tar.gz important-folder/
gpg -e -r kamu@email.com backup.tar.gz
```

### Dekripsi:
```bash
gpg -d backup.tar.gz.gpg > backup.tar.gz
tar xzf backup.tar.gz
```

---

> 🔐 GPG CLI di Ubuntu sangat cocok untuk otomasi, backup aman, komunikasi rahasia, dan praktik keamanan digital profesional.
