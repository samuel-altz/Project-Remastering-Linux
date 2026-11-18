# Linux Finance 1.0

**Distribusi Linux hasil remastering berbasis Ubuntu 20.04 LTS untuk staf keuangan, akuntan pemula, dan mahasiswa akuntansi.**

Nama kode: **OS Cawapres**

**Sistem Operasi**

---

## Daftar Isi

1. [Ringkasan Proyek](#1-ringkasan-proyek)
2. [Identitas Distribusi](#2-identitas-distribusi)
3. [Target Pengguna](#3-target-pengguna)
4. [Lingkungan dan Alat Kerja](#4-lingkungan-dan-alat-kerja)
5. [Fitur dan Modifikasi](#5-fitur-dan-modifikasi)
6. [Metode Remastering](#6-metode-remastering)
7. [Pengujian](#7-pengujian)
8. [Unduh dan Pemasangan](#8-unduh-dan-pemasangan)
9. [Kendala dan Solusi](#9-kendala-dan-solusi)
10. [Struktur Repositori](#10-struktur-repositori)
11. [Tim Pengembang](#11-tim-pengembang)
12. [Lisensi](#12-lisensi)

---

## 1. Ringkasan Proyek

Linux Finance adalah citra ISO Linux yang dibuat dengan teknik **remastering**, yaitu memodifikasi citra ISO Ubuntu yang sudah ada agar sesuai dengan kebutuhan kelompok pengguna tertentu. Dalam proyek ini, Ubuntu 20.04.6 LTS Desktop dijadikan basis, kemudian dikustomisasi untuk mendukung pekerjaan administrasi dan pembukuan keuangan.

Tujuan proyek:

- Menerapkan konsep sistem operasi (komponen OS, struktur OS, manajemen paket, *file system*, pengaturan sistem, dan konsep Linux) ke dalam produk nyata.
- Menghasilkan ISO yang dapat di-*boot*, dipasang sebagai instalasi baru pada mesin virtual, dan fitur utamanya dapat langsung digunakan.
- Mendokumentasikan seluruh langkah secara terstruktur sehingga proses dapat direproduksi.

## 2. Identitas Distribusi

| Atribut | Keterangan |
|---|---|
| Nama distribusi | Linux Finance |
| Nama kode | OS Cawapres |
| Versi | 1.0 |
| Basis | Ubuntu 20.04.6 LTS (Focal Fossa), amd64 |
| Lingkungan desktop | GNOME |
| Berkas ISO | `cawapres.iso` |
| Ukuran | 4.590.725.120 byte (sekitar 4,28 GiB) |
| Checksum MD5 | `cce2524f2597aac6ac85244943edd8f6` |
| Alat remastering | Cubic (Custom Ubuntu ISO Creator) |
| Lisensi | GPL-3.0 |

## 3. Target Pengguna

Distribusi ini ditujukan bagi:

- **Staf keuangan dan administrasi** yang membutuhkan aplikasi pembukuan dan lembar kerja.
- **Akuntan pemula** yang ingin belajar mencatat transaksi dengan prinsip akuntansi berpasangan (*double-entry*).
- **Mahasiswa akuntansi** yang membutuhkan lingkungan kerja yang rapi, aman, dan siap pakai.

Pemilihan aplikasi, penghapusan paket, dan pengaturan sistem seluruhnya disesuaikan dengan kebutuhan kelompok pengguna tersebut.

## 4. Lingkungan dan Alat Kerja

| Komponen | Keterangan |
|---|---|
| Sistem operasi host | Windows 11 |
| Virtualisasi | Oracle VirtualBox |
| VM pengembangan | Ubuntu 20.04 (nama VM: `Ubuntu0`), disk virtual diperbesar menjadi 60 GB |
| Alat remastering | Cubic 2026.09.110 |
| Citra dasar | `ubuntu-20.04.6-desktop-amd64.iso` |
| Pertukaran berkas | *Shared folder* VirtualBox (host ke VM) |
| Distribusi hasil | GitHub Releases |

Seluruh pekerjaan dilakukan di dalam mesin virtual sehingga sistem operasi utama tidak terpengaruh.

## 5. Fitur dan Modifikasi

### 5.1 Identitas dan visual

- Nama distribusi, versi, dan *volume ID* ISO disesuaikan melalui Cubic.
- Wallpaper bertema keuangan (grafik *candlestick*, koin Rp, $, dan %) dirancang sendiri dan ditetapkan sebagai wallpaper bawaan melalui berkas *override* skema GSettings.

### 5.2 Aplikasi tambahan

| Aplikasi | Fungsi |
|---|---|
| **GnuCash** | Perangkat lunak pembukuan dan akuntansi |
| **LibreOffice Calc** | Lembar kerja untuk perhitungan dan laporan |
| **LibreOffice Writer** | Pengolah kata untuk dokumen dan laporan |
| **Qalculate!** | Kalkulator lanjutan dan konversi mata uang |
| **KeePassXC** | Pengelola kata sandi untuk mengamankan data keuangan |

### 5.3 Paket yang dihapus

| Paket | Alasan |
|---|---|
| **Rhythmbox** | Pemutar musik tidak relevan untuk pekerjaan keuangan |
| **Transmission** | Klien BitTorrent tidak relevan dan berisiko terhadap keamanan data finansial |

### 5.4 Pengaturan sistem

| Pengaturan | Nilai | Alasan |
|---|---|---|
| Zona waktu | `Asia/Jakarta` (WIB) | Pencatatan transaksi memerlukan stempel waktu yang tepat |
| Bahasa dan format regional | `id_ID.UTF-8` | Format angka dan mata uang sesuai kebiasaan di Indonesia |
| Kunci layar otomatis | Aktif, setelah 5 menit tidak aktif | Melindungi data keuangan saat pengguna meninggalkan komputer |

### 5.5 Dokumen bawaan

Berkas `README.txt` ditempatkan di `/etc/skel/Desktop/` sehingga otomatis muncul di Desktop setiap pengguna baru. Isinya menjelaskan identitas distribusi, aplikasi utama, dan cara penggunaan.

### 5.6 Skrip shell

Skrip `cek-sistem.sh` terpasang di `/usr/local/bin/` dan dapat dijalankan dari terminal oleh semua pengguna untuk menampilkan informasi sistem.

```bash
#!/bin/bash
echo "=== Linux Finance: Info Sistem ==="
echo "Host     : $(hostname)"
echo "OS       : $(lsb_release -ds)"
echo "Kernel   : $(uname -r)"
echo "CPU      : $(lscpu | grep 'Model name' | sed 's/Model name: *//')"
echo "RAM      : $(free -h | awk '/Mem:/ {print $3 " / " $2}')"
echo "Disk (/) : $(df -h / | awk 'NR==2 {print $3 " / " $2}')"
echo "Waktu    : $(date '+%d-%m-%Y %H:%M:%S %Z')"
```

Cara menjalankan:

```bash
cek-sistem.sh
```

## 6. Metode Remastering

Alur kerja secara garis besar:

1. **Persiapan** — membuat VM Ubuntu di VirtualBox, memperbesar disk virtual, memasang Guest Additions, lalu memasang Cubic dari PPA.
2. **Pemuatan ISO** — memindahkan ISO Ubuntu 20.04.6 ke VM melalui *shared folder*, kemudian membuat proyek baru di Cubic.
3. **Kustomisasi di chroot** — seluruh perubahan dilakukan pada terminal lingkungan virtual (chroot) Cubic.
4. **Pembangkitan ISO** — memilih kompresi dan menjalankan *Generate* untuk menghasilkan ISO.
5. **Pengujian** — *boot* ISO pada VM baru dan memasangnya sebagai instalasi baru.

### 6.1 Memasang Cubic

```bash
sudo apt-add-repository ppa:cubic-wizard/release
sudo apt update
sudo apt install cubic
```

### 6.2 Aplikasi dan penghapusan paket (di chroot)

```bash
add-apt-repository -y universe
apt update
apt install -y gnucash libreoffice-calc libreoffice-writer qalculate-gtk keepassxc language-pack-id

apt remove --purge -y rhythmbox transmission-gtk
apt autoremove -y
apt clean
```

### 6.3 Zona waktu dan bahasa

```bash
ln -sf /usr/share/zoneinfo/Asia/Jakarta /etc/localtime
echo "Asia/Jakarta" > /etc/timezone
update-locale LANG=id_ID.UTF-8
```

### 6.4 Kunci layar otomatis

```bash
cat > /usr/share/glib-2.0/schemas/91_finance.gschema.override << 'EOF'
[org.gnome.desktop.screensaver]
lock-enabled=true
lock-delay=uint32 0

[org.gnome.desktop.session]
idle-delay=uint32 300
EOF
glib-compile-schemas /usr/share/glib-2.0/schemas/
```

### 6.5 Wallpaper bawaan

```bash
cp /root/finance.jpg /usr/share/backgrounds/finance.jpg
chmod 644 /usr/share/backgrounds/finance.jpg

cat > /usr/share/glib-2.0/schemas/92_wallpaper.gschema.override << 'EOF'
[org.gnome.desktop.background]
picture-uri='file:///usr/share/backgrounds/finance.jpg'
picture-options='zoom'

[org.gnome.desktop.screensaver]
picture-uri='file:///usr/share/backgrounds/finance.jpg'
picture-options='zoom'
EOF
glib-compile-schemas /usr/share/glib-2.0/schemas/
```

## 7. Pengujian

| Pengujian | Hasil |
|---|---|
| ISO berhasil di-*boot* pada VirtualBox | Berhasil, installer Ubuntu tampil |
| Aplikasi tambahan terpasang (`dpkg -l`) | GnuCash, LibreOffice Calc, Qalculate, dan KeePassXC terkonfirmasi |
| Paket yang dihapus | Rhythmbox dan Transmission terkonfirmasi terhapus |
| Skrip `cek-sistem.sh` | Berjalan dan menampilkan tujuh baris informasi sistem, termasuk zona waktu WIB |
| Dokumen `README.txt` bawaan | Tersimpan di `/etc/skel/Desktop/` |
| Integritas ISO | Checksum MD5 pada Windows sama dengan checksum yang dibuat Cubic |
| Pemasangan sebagai instalasi baru | `[ISI: hasil setelah instalasi selesai]` |

## 8. Unduh dan Pemasangan

### 8.1 Mengunduh

ISO dibagi menjadi tiga bagian karena GitHub membatasi ukuran berkas pada Releases hingga di bawah 2 GiB per berkas.

Unduh seluruh berkas dari halaman **[Releases](../../releases)**:

- `cawapres.iso.part-00`
- `cawapres.iso.part-01`
- `cawapres.iso.part-02`
- `cawapres.md5`

### 8.2 Menyatukan kembali ISO

**Windows (Command Prompt):**

```
copy /b cawapres.iso.part-00+cawapres.iso.part-01+cawapres.iso.part-02 cawapres.iso
```

**Linux:**

```bash
cat cawapres.iso.part-* > cawapres.iso
```

### 8.3 Memeriksa keutuhan

**Windows (PowerShell):**

```powershell
Get-FileHash -Algorithm MD5 .\cawapres.iso
```

**Linux:**

```bash
md5sum cawapres.iso
```

Hasilnya harus sama dengan isi `cawapres.md5`, yaitu `cce2524f2597aac6ac85244943edd8f6`.

### 8.4 Memasang pada VirtualBox

1. Buat VM baru dengan **RAM 4 GB**, **2 CPU**, dan **disk 25 GB atau lebih**.
2. Pilih `cawapres.iso` sebagai ISO instalasi dan centang *Skip Unattended Installation*.
3. Pada **Settings → Display**: *Graphics Controller* **VMSVGA**, *Video Memory* **128 MB**, *3D Acceleration* dinonaktifkan.
4. Jalankan VM, pilih **Try Ubuntu** untuk mencoba atau **Install Ubuntu** untuk memasang.
5. Setelah instalasi selesai, restart dan lepas ISO dari drive virtual.

### 8.5 Alternatif

Tautan unduhan ISO utuh: `[ISI: tautan Google Drive jika ada]`

## 9. Kendala dan Solusi

Bagian ini mendokumentasikan masalah yang ditemui selama pengerjaan.

| Kendala | Penyebab | Solusi |
|---|---|---|
| Paket `gnucash` dan `qalculate-gtk` tidak ditemukan di chroot | Repositori *universe* belum aktif | `add-apt-repository -y universe` lalu `apt update` |
| Pesan `couldn't connect to zsys daemon` | Layanan `zsys` tidak berjalan di dalam chroot | Diabaikan, tidak memengaruhi hasil |
| Wallpaper tidak berubah | Nama grup pada berkas *override* salah ketik (`backgrounds`, seharusnya `background`), dan izin berkas gambar terlalu ketat (`-rwxr-x---`) | Memperbaiki nama grup dan menjalankan `chmod 644` pada gambar |
| Tombol *Test* Cubic sangat lambat | QEMU berjalan di dalam VirtualBox tanpa akselerasi perangkat keras | Pengujian dilakukan langsung pada VM VirtualBox baru |
| Layar VM hitam setelah login | Masalah sesi grafis pada VM pengembangan | Menggunakan sesi **Ubuntu on Wayland** dan menyesuaikan pengaturan *Display* |
| Tidak dapat mengunggah ISO ke GitHub | Batas 25 MB (browser), 100 MiB (Git), dan 2 GiB per berkas (Releases) | ISO dipecah menjadi tiga bagian dan diunggah melalui Releases |
| Ruang disk VM tidak cukup untuk Cubic | Partisi awal hanya sekitar 27 GB | Disk virtual diperbesar menjadi 60 GB dan partisi diperluas dengan GParted |

## 10. Struktur Repositori

```
Project-Remastering-Linux/
├── README.md            # Dokumentasi proyek (berkas ini)
├── LICENSE              # GPL-3.0
├── cek-sistem.sh        # Skrip informasi sistem
├── README.txt           # Dokumen bawaan yang ikut di dalam ISO
├── finance.jpg          # Wallpaper bawaan
└── cawapres.md5         # Checksum ISO
```

> Berkas ISO tidak disimpan di repositori, tetapi dibagikan melalui **Releases**.

## 11. Tim Pengembang

Samuel Clement Tjahjono / samuel-altz

**Video dokumentasi:** `[ISI: tautan YouTube]`

## 12. Lisensi

Proyek ini dirilis di bawah lisensi **GNU General Public License v3.0** (GPL-3.0). Rincian lengkap tersedia pada berkas [`LICENSE`](LICENSE).

Ubuntu dan komponen perangkat lunak bebas lain di dalam ISO tetap tunduk pada lisensi masing-masing. Ubuntu adalah merek dagang terdaftar milik Canonical Ltd. Proyek ini adalah karya akademik dan tidak berafiliasi dengan Canonical.

---

