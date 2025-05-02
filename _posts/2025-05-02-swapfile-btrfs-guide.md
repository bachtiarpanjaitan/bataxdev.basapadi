---
title: "Error swapon: /swap/swapfile: swapon failed: Invalid argument"
excerpt: "Mengalami error 'swapon failed: Invalid argument' ketika melakukan perubahan atau membuat swap file di linux"
share: true
comments: true
header:
  image: /assets/images/swap_space.png
  teaser: /assets/images/swap_space.png
  overlay_image: /assets/images/unsplash-image-2.jpg
  og_image: /assets/images/swap_space.png
categories:
  - TipsAndTricks
tags:
  - Linux
  - Debian
  - Arch
image: /assets/images/swap_space.png
---


# Catatan Swapfile Kompatibel dengan BTRFS di Linux Manjaro

## Masalah dan Error yang Dihadapi

**Masalah:**
- Swapfile gagal aktif di sistem BTRFS.
- Swapfile yang sebelumnya sudah dibuat (512MB) tidak dapat menangani beban kerja yang lebih berat, seperti compile package AUR.
- Saat mencoba membuat swapfile baru dengan ukuran lebih besar (8GB), selalu gagal pada langkah `swapon /swap/swapfile`.

**Error yang Ditemui:**
```
swapon: /swap/swapfile: swapon failed: Invalid argument
```

**Penyebab:**
- **BTRFS** tidak mendukung swapfile di subvolume root (/) yang memiliki fitur Copy-on-Write (CoW) dan compression.
- File swap yang dibuat dengan `fallocate` atau teknik lain yang menghasilkan sparse file tidak kompatibel dengan BTRFS.

## Langkah-langkah Solusi (Swapfile Kompatibel BTRFS)

1. **Matikan swap yang lama dan hapus file swap sebelumnya:**
   ```
   sudo swapoff /swap/swapfile
   sudo rm -f /swap/swapfile
   ```

2. **Buat folder khusus untuk swap (di luar root /):**
   ```
   sudo mkdir -p /swap
   sudo chattr +C /swap     # Matikan Copy-on-Write (CoW)
   ```

3. **Buat file swap baru dengan ukuran 8GB menggunakan dd:**
   ```
   sudo dd if=/dev/zero of=/swap/swapfile bs=1M count=8192 status=progress
   ```
   Hindari penggunaan `fallocate` karena menghasilkan sparse file yang tidak didukung.

4. **Set permission yang aman untuk swapfile:**
   ```
   sudo chmod 600 /swap/swapfile
   ```

5. **Format swapfile menjadi swap:**
   ```
   sudo mkswap /swap/swapfile
   ```

6. **Aktifkan swap:**
   ```
   sudo swapon /swap/swapfile
   ```

7. **Tambahkan swap ke /etc/fstab agar aktif otomatis tiap boot:**
   ```
   echo '/swap/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
   ```

8. **Cek status swap:**
   ```
   swapon --show
   free -h
   ```

## Catatan Penting
- **BTRFS** tidak kompatibel dengan swapfile di root (/) atau di subvolume lainnya yang memakai CoW atau compression.
- Untuk membuat swapfile yang kompatibel, pastikan:
  - File swap tidak berada di root (/).
  - Gunakan perintah `chattr +C` untuk menonaktifkan CoW pada direktori tempat file swap disimpan.
  - Jangan gunakan `fallocate` untuk membuat swapfile, karena itu menghasilkan sparse file yang tidak cocok dengan BTRFS.
  - Pastikan `lsattr` menunjukkan file swap tanpa huruf `C` (Copy-on-Write).
