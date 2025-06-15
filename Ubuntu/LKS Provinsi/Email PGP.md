## **🧾 Soal :**
Anda bekerja sebagai teknisi keamanan informasi di sebuah perusahaan. Divisi Anda sedang melakukan audit terhadap praktik pengiriman e-mail aman. Anda diminta untuk menyiapkan skema pengiriman email terenkripsi menggunakan PGP di sistem Ubuntu.

✅ Instruksi:
Install GnuPG (GPG):
Pastikan sistem memiliki gpg versi terbaru.
Buat sepasang key PGP:

```bash
Nama: LKS SMK

Email: lks@smkprovinsi.id

Key type: RSA & RSA

Ukuran kunci: 4096 bit

Expiry: 0 (never expire)

```

- Ekspor public key Anda ke file public-lks.asc
- Buat file pesan pesan.txt berisi teks:
  "Ini adalah pesan rahasia untuk panitia LKS."
Enkripsi file tersebut menggunakan public key Anda sendiri. Simpan hasilnya sebagai pesan.txt.gpg
Buat digital signature terhadap file pesan.txt dan hasilkan file pesan.txt.sig
Verifikasi kembali signature yang telah Anda buat

---

## 🔧 1. Install GnuPG (GPG)

```
sudo apt update
sudo apt install gnupg -y
```

📝 **Fungsi dan Penjelasan Lengkap:**

- `sudo apt update`:  
    Perintah ini menyegarkan daftar paket dari repository. Dengan begini, kita memastikan bahwa semua paket yang tersedia untuk diinstall merupakan versi terbaru yang disediakan oleh pengelola distribusi Linux. Ini langkah awal wajib sebelum install software.
    
- `sudo apt install gnupg -y`:  
    Ini akan menginstal **GNU Privacy Guard (GPG)**, alat kriptografi berbasis standar **OpenPGP (RFC 4880)**. GPG memungkinkan kita membuat kunci digital, mengenkripsi data, dan menandatangani file agar keasliannya bisa diverifikasi.  
    Flag `-y` digunakan agar kita tidak perlu mengetik `y` saat konfirmasi instalasi—berguna saat proses otomatisasi.
    

---

## 🔧 2. Buat PGP Key Pair

```
gpg --full-generate-key
```

📝 **Fungsi dan Penjelasan Lengkap:**

Perintah ini memulai wizard interaktif untuk membuat **key pair** (pasangan kunci) yaitu **Public Key** dan **Private Key**.

### Input Rekomendasi:

```
Key type:        RSA and RSA          # Algoritma kriptografi populer & kuat
Key size:        4096                 # Ukuran bit besar untuk keamanan maksimal
Expiry:          0 (Never Expire)     # Supaya gak kadaluarsa (bisa diubah nanti)
Name:            LKS SMK              # Identitas pemilik key
Email:           lks@smkprovinsi.id   # Email pemilik key
```

🔐 **Public Key**: Dipakai orang lain untuk mengenkripsi pesan ke kamu.  
🔑 **Private Key**: Hanya kamu yang punya; dipakai untuk decrypt dan tanda tangan digital.

> 💡 Gunakan identitas sesuai agar pengelolaan kunci jadi mudah dan profesional.

---

## 🔧 3. Ekspor Public Key ke File

```
gpg --export -a LKS SMK > public-lks.asc
```

📝 **Fungsi dan Penjelasan Lengkap:**

- `--export`: Mengekspor public key berdasarkan identitas (nama/email).
    
- `-a / --armor`: Mengubah format output jadi **ASCII Armor (.asc)** — cocok buat dikirim via teks/email.
    
- `> public-lks.asc`: Output-nya disimpan di file bernama `public-lks.asc`.
    

📬 **Gunanya:**  
File public key ini bisa kamu kirim ke orang lain agar mereka bisa mengenkripsi pesan untukmu atau memverifikasi signature darimu.

> 🧠 Aman dibagikan ke publik. Tapi **private key jangan pernah dibagikan!**

---

## 🔧 4. Buat File pesan.txt

```
echo "Ini adalah pesan rahasia untuk panitia LKS." > pesan.txt
```

📝 **Fungsi dan Penjelasan Lengkap:**

Perintah `echo` dipakai untuk menuliskan kalimat ke dalam file baru bernama `pesan.txt`.

📄 File ini adalah contoh **data plaintext** atau file asli yang nantinya akan:

- **Dienkripsi** menjadi `.gpg`
    
- **Ditandatangani** secara digital menjadi `.sig`
    

> ✍️ Kamu bebas ganti isi pesannya sesuai kebutuhan.

---

## 🔧 5. Enkripsi File pesan.txt

```
gpg --output pesan.txt.gpg --encrypt --recipient lks@smkprovinsi.id pesan.txt
```

📝 **Fungsi dan Penjelasan Lengkap:**

- `--output pesan.txt.gpg`: Nama file hasil enkripsi.
    
- `--encrypt`: Menyuruh GPG untuk melakukan enkripsi data.
    
- `--recipient lks@smkprovinsi.id`: Tentukan **public key penerima** yang akan digunakan untuk enkripsi.
    
- `pesan.txt`: File yang akan dienkripsi.
    

🔐 **Hasilnya adalah file `.gpg`** yang hanya bisa dibuka oleh pemilik **private key** sesuai `recipient`.

Alternatif singkat:

```
gpg -e -r lks@smkprovinsi.id pesan.txt
```

📦 Tapi perintah ini gak kasih nama output file, jadi hasilnya default (langsung tertimpa di CLI atau file `.gpg` dengan nama sama).

> 🧠 Aman dikirim lewat email, karena hanya penerima (dengan private key) yang bisa baca isinya.

---

## 🔧 6. Buat Digital Signature untuk File

```
gpg --output pesan.txt.sig --detach-sign pesan.txt
```

📝 **Fungsi dan Penjelasan Lengkap:**

- `--detach-sign`: Signature dibuat sebagai file terpisah, **tidak menyatu dengan file aslinya**. Jadi lebih fleksibel.
    
- `--output pesan.txt.sig`: Nama file signature yang disimpan.
    
- `pesan.txt`: File yang akan ditandatangani.
    

🧾 Tanda tangan digital diproses menggunakan **private key kamu** dan menciptakan signature unik berdasarkan isi file.

> ⚠️ Jika isi `pesan.txt` berubah sedikit saja, signature akan dianggap **tidak valid**.

---

## 🔧 7. Verifikasi Signature

```
gpg --verify pesan.txt.sig pesan.txt
```

📝 **Fungsi dan Penjelasan Lengkap:**

Perintah ini digunakan untuk **memverifikasi** tanda tangan digital (`.sig`) terhadap file aslinya.

📋 GPG akan:

- Mengecek apakah signature cocok dengan isi file.
    
- Mengecek apakah signature dibuat oleh **public key** yang valid.
    

Contoh hasil yang sukses:

```
gpg: Good signature from "LKS SMK <lks@smkprovinsi.id>"
```

> 💡 Signature menjamin **integritas** dan **keaslian** file, bukan enkripsi.

---

## 📁 Struktur File Akhir

```bash
.
├── pesan.txt           # File asli (plaintext)
├── pesan.txt.gpg       # Hasil enkripsi menggunakan public key
├── pesan.txt.sig       # File signature yang dibuat dengan private key
└── public-lks.asc      # Public key yang bisa dibagikan ke siapa saja
```

📂 File-file ini bisa:

- Dipresentasikan ke juri
    
- Dikirim via email
    
- Diupload ke web demo
    
- Dipakai untuk menunjukkan **kekuatan kriptografi PGP**
    

---
