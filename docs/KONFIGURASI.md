# Tata Cara Konfigurasi EMO RUSH Photobooth

Panduan ini menjelaskan cara menyiapkan EMO RUSH Photobooth di Windows sampai bisa dijalankan. Semua perintah ditulis untuk PowerShell dan dijalankan dari folder utama proyek, yaitu folder yang berisi `DRAFT0_EC.py`.

## 1. Kebutuhan

| Komponen | Keterangan |
| --- | --- |
| Sistem operasi | Windows 10 atau 11, 64-bit |
| Python | 3.11, 64-bit, dengan Python launcher (`py`) |
| Git | Git for Windows |
| Kamera | Webcam yang dikenali Windows |
| Ruang disk | Minimal 5 GB, disarankan 10 GB |
| Internet | Hanya diperlukan saat instalasi |

GPU tidak diperlukan karena TensorFlow berjalan di CPU.

Periksa bahwa Python 3.11 tersedia:

```powershell
py -0p
py -3.11 --version
```

Selalu gunakan `py -3.11`. Perintah `python` biasa bisa mengarah ke versi lain, dan versi yang lebih baru seperti 3.14 belum tentu didukung oleh dependensi proyek ini.

## 2. Pilih lokasi virtual environment

TensorFlow berisi file dengan nama dan folder yang sangat panjang. Jika Windows Long Paths belum aktif, batas panjang path adalah 260 karakter dan instalasi bisa gagal dengan pesan seperti ini:

```text
ERROR: Could not install packages due to an OSError: [Errno 2] No such file or directory: '...\tensorflow\include\external\com_github_grpc_grpc\...'
HINT: This error might have occurred since this system does not have Windows Long Path support enabled.
```

Periksa status Long Paths:

```powershell
(Get-ItemProperty 'HKLM:\SYSTEM\CurrentControlSet\Control\FileSystem').LongPathsEnabled
```

- **Hasilnya `1`:** Long Paths aktif. Ikuti [Opsi B](#opsi-b-venv-di-dalam-folder-proyek).
- **Hasilnya `0` atau kosong:** ikuti [Opsi A](#opsi-a-venv-di-path-pendek-tanpa-hak-admin) jika path proyek panjang. Ini adalah opsi yang dipakai pada komputer pengembangan saat ini.

## 3. Pasang dependensi

### Opsi A: venv di path pendek, tanpa hak admin

Virtual environment dibuat di `%USERPROFILE%\.venvs\emorush`. Folder `venv` di proyek lalu dibuat sebagai junction yang menunjuk ke sana, sehingga `run_game.bat` tetap bekerja tanpa perubahan.

```powershell
$venv = "$env:USERPROFILE\.venvs\emorush"
py -3.11 -m venv $venv
& "$venv\Scripts\python.exe" -m pip install --upgrade pip
& "$venv\Scripts\python.exe" -m pip install -r requirements.txt
New-Item -ItemType Junction -Path venv -Target $venv
```

Jika folder `venv` sisa instalasi yang gagal masih ada, hapus dulu sebelum membuat junction. Lihat [Mengulang instalasi](#mengulang-instalasi).

### Opsi B: venv di dalam folder proyek

Gunakan opsi ini jika Long Paths sudah aktif atau path proyek cukup pendek.

```powershell
py -3.11 -m venv venv
venv\Scripts\python.exe -m pip install --upgrade pip
venv\Scripts\python.exe -m pip install -r requirements.txt
```

Long Paths bisa diaktifkan dengan perintah berikut di PowerShell yang dibuka sebagai Administrator. Perubahan ini berlaku untuk seluruh sistem.

```powershell
New-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\FileSystem' -Name LongPathsEnabled -Value 1 -PropertyType DWORD -Force
```

Unduhan TensorFlow dan PyTorch berukuran besar, jadi instalasi bisa memakan waktu beberapa menit.

## 4. Verifikasi instalasi

```powershell
venv\Scripts\python.exe -c "import cv2, tensorflow; from fer.fer import FER; print('OK')"
venv\Scripts\python.exe -m pip check
```

Instalasi benar jika perintah pertama mencetak `OK` dan perintah kedua mencetak `No broken requirements found.`

Pesan berikut hanya peringatan dan bisa diabaikan:

- `oneDNN custom operations are on`
- `pkg_resources is deprecated`
- `tf.lite.Interpreter is deprecated`

## 5. Menjalankan program

```powershell
.\run_game.bat
```

Bisa juga dengan klik dua kali `run_game.bat` di File Explorer, atau dengan perintah langsung:

```powershell
venv\Scripts\python.exe DRAFT0_EC.py
```

Pemuatan pertama membutuhkan beberapa detik. Program lalu membuka jendela fullscreen berjudul **EMO RUSH!!!**.

| Tombol | Fungsi |
| --- | --- |
| `F` | Beralih antara fullscreen dan jendela biasa |
| `Q` | Keluar |
| `R` | Main lagi dari layar akhir |

## 6. Pengaturan program

Pengaturan berada di bagian atas `DRAFT0_EC.py`, di bawah komentar `Konfigurasi`.

| Konstanta | Nilai awal | Fungsi |
| --- | --- | --- |
| `CHALLENGE_DURATION` | `5` | Durasi fase permainan per emosi, dalam detik |
| `PREP_DURATION` | `5` | Durasi fase persiapan, dalam detik |
| `COUNTDOWN_DURATION` | `3` | Durasi hitung mundur, dalam detik |
| `RESULT_DURATION` | `3` | Durasi tampilan hasil per emosi, dalam detik |
| `MAX_PLAYERS` | `3` | Jumlah pemain maksimum |
| `DETECTION_WIDTH` | `640` | Lebar frame untuk deteksi; nilai lebih kecil lebih ringan untuk CPU |
| `PHOTO_FOLDER` | `hasil_photo` | Folder penyimpanan hasil |
| `PHOTO_TEMPLATE_PATH` | `assets/photo_strip_template.png` | Template photo strip |

Pengaturan kamera berada di fungsi `main()`:

- Ubah angka pada `cv2.VideoCapture(0)` menjadi `1`, `2`, dan seterusnya untuk memakai kamera lain.
- Resolusi diminta melalui `cap.set(cv2.CAP_PROP_FRAME_WIDTH, 1280)` dan `cap.set(cv2.CAP_PROP_FRAME_HEIGHT, 720)`.

## 7. Hasil foto

Hasil yang disetujui disimpan secara lokal di `hasil_photo/<email>/`, berisi photo strip PNG dan GIF animasi. Program tidak mengirim email. Isi folder ini diabaikan oleh Git supaya foto peserta tidak ikut ter-commit.

## 8. Git

Repositori ini berada di folder proyek dengan branch `main`. Yang diabaikan oleh Git adalah `venv`, cache Python, foto hasil, dan file editor.

Untuk mengirim ke remote, buat repositori kosong di layanan Git, lalu jalankan:

```powershell
git remote add origin <URL-repositori>
git push -u origin main
```

Periksa isi commit sebelum push dengan `git status` dan `git ls-files`, terutama agar tidak ada foto peserta yang ikut terkirim.

## 9. Mengatasi masalah

| Gejala | Penyebab dan solusi |
| --- | --- |
| `py` tidak dikenali | Pasang ulang Python dengan Python launcher, lalu buka terminal baru |
| Python 3.11 tidak ditemukan | Pasang Python 3.11 64-bit |
| `No such file or directory` di dalam `tensorflow\include` | Path terlalu panjang. Gunakan Opsi A atau aktifkan Long Paths |
| `[ERROR] No Python installation with OpenCV, FER, and TensorFlow was found` | `venv` belum terpasang atau rusak. Ulangi langkah 3 |
| `[ERROR] Webcam tidak bisa diakses!` | Buka **Settings → Privacy & security → Camera**, izinkan aplikasi desktop, dan tutup aplikasi lain yang memakai kamera |
| `[ERROR] Webcam stopped returning frames.` | Kamera terputus atau dipakai aplikasi lain. Periksa kamera, lalu jalankan ulang |
| Kamera yang terbuka salah | Ubah indeks pada `cv2.VideoCapture(0)` |
| Deteksi emosi lambat | Tutup aplikasi berat atau kecilkan `DETECTION_WIDTH` |
| `venv` rusak setelah folder `.venvs` dihapus atau dipindah | Junction kehilangan target. Hapus junction, lalu ulangi Opsi A |

### Mengulang instalasi

Jika `venv` adalah junction (Opsi A), hapus junction dengan `rmdir` dari `cmd`. Perintah ini hanya menghapus tautannya:

```powershell
cmd /c rmdir venv
Remove-Item -Recurse -Force "$env:USERPROFILE\.venvs\emorush"
```

Jangan menghapus junction dengan `Remove-Item -Recurse`, karena pada Windows PowerShell 5.1 perintah itu bisa ikut menghapus isi folder tujuannya.

Jika `venv` adalah folder biasa (Opsi B):

```powershell
Remove-Item -Recurse -Force venv
```

Setelah itu, ulangi langkah 3.
