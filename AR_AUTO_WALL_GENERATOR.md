# Unity 6 --- Auto Wall Generator dari Denah yang Sudah Terpasang

## 1. Tujuan dan batas cakupan

Proyek AR kampus sudah memiliki gambar denah dan konfigurasi dasar di
Unity 6. Tugas pada tahap ini **hanya membuat dinding 3D berdasarkan
garis pada denah yang sudah ada**.

Jangan membangun ulang sistem AR, jangan mengganti gambar denah, dan
jangan mengulang tahap setup project.

Tujuan utama: mengekstrak garis dinding/partisi dari gambar denah secara
otomatis, lalu menghasilkan dinding 3D yang mengikuti garis tersebut
sedekat mungkin tanpa pengguna menggambar garis satu per satu.

## 2. Kondisi proyek yang harus dipertahankan

Sebelum mengubah apa pun, periksa scene aktif dan identifikasi: -
GameObject yang menampilkan gambar denah. - Transform gambar denah:
posisi, rotasi, dan skala. - Bidang/plane tempat denah dipasang, jika
ada. - Objek AR yang sudah tersedia, termasuk `AR Session`, `XR Origin`,
`XR Environment`, kamera AR, dan konfigurasi terkait.

Pertahankan semua objek dan pengaturan tersebut. Jangan menghapus,
mengganti nama, memindahkan, atau mengatur ulang objek yang tidak
terkait dengan generator dinding.

Gunakan gambar/texture sumber yang dipakai oleh GameObject denah sebagai
input bila dapat diakses. Jika tidak dapat ditemukan secara pasti,
laporkan nama dan lokasi kandidat file lalu minta konfirmasi. Jangan
diam-diam menggunakan screenshot Unity sebagai pengganti gambar asli.

## 3. Hasil yang diinginkan

Buat dinding 3D yang mengikuti: - Garis batas luar bangunan. - Garis
partisi horizontal dan vertikal antar-ruangan. - Partisi berbentuk tidak
beraturan, termasuk bentuk U pada area Ruang Direktur. - Dinding di
sekitar ruangan kecil, toilet, pantry, ruang server, dan ruang
lainnya. - Garis pembatas ruang dan lorong yang memang merupakan dinding
pada gambar.

Jangan menganggap teks nama ruang, kotak label, simbol kolom, garis
furnitur, garis dimensi, atau busur simbol pintu sebagai dinding.

Bukaan pintu harus dipertahankan jika bisa dideteksi. Jangan membuat
dinding menutup pintu secara sembarangan.

## 4. Metode implementasi

### A. Audit singkat

1.  Periksa project dan scene aktif.
2.  Temukan sumber gambar denah yang digunakan di scene.
3.  Catat transform denah saat ini.
4.  Periksa package dan tool yang sudah tersedia.
5.  Pilih pendekatan computer vision paling praktis untuk lingkungan
    yang ada, misalnya OpenCV/Python atau implementasi pemrosesan gambar
    lain yang kompatibel.

Jangan membuat ulang scene atau setup AR.

### B. Ekstraksi garis otomatis

1.  Baca gambar denah yang benar-benar digunakan sebagai sumber.
2.  Lakukan prapemrosesan gambar, grayscale/threshold, pengurangan
    noise, dan deteksi garis.
3.  Deteksi garis horizontal, vertikal, dan segmen lain yang relevan.
4.  Gabungkan segmen yang terputus akibat noise menggunakan toleransi
    yang dapat diatur.
5.  Kurangi gangguan dari teks, label, simbol kolom, simbol pintu, dan
    elemen non-dinding.
6.  Hasilkan data segmen dinding berisi koordinat awal dan akhir pada
    ruang koordinat gambar.
7.  Buat overlay hasil deteksi di atas gambar denah untuk memeriksa
    apakah garis dinding terdeteksi dengan benar.

Jangan membuat koordinat secara manual sebagai pengganti ekstraksi
otomatis. Jangan menyatakan hasil akurat sebelum pipeline dijalankan
pada gambar sebenarnya.

### C. Generator dinding di Unity

Buat Unity Editor Tool yang dapat: - Mengambil hasil ekstraksi garis. -
Mengubah koordinat gambar menjadi koordinat lokal yang sesuai dengan
transform denah yang sudah ada. - Membuat dinding 3D dengan parameter
tinggi dan ketebalan yang bisa diatur. - Menjaga arah dan skala agar
dinding sejajar dengan denah. - Mengurangi celah pada sambungan dinding
dan menghindari dinding duplikat. - Mempertahankan bukaan pintu yang
terdeteksi. - Menempatkan hasil di bawah GameObject khusus bernama
`GeneratedWalls`. - Menjalankan generate ulang tanpa menggandakan hasil
sebelumnya. - Menghapus hanya objek yang dibuat generator, bukan objek
AR atau objek lain.

Utamakan geometri yang mudah diedit dan efisien untuk perangkat Android.
Jangan membuat satu objek untuk setiap piksel; gunakan segmen dinding
yang telah diekstrak.

### D. Koordinat dan skala

-   Jadikan transform gambar denah yang sudah ada sebagai acuan
    penempatan.
-   Jangan mengubah posisi, rotasi, atau skala gambar denah untuk
    menyesuaikan dinding.
-   Konversi koordinat piksel ke bidang lokal secara konsisten;
    dokumentasikan orientasi sumbu.
-   Sediakan pengaturan tinggi dan ketebalan dinding.
-   Jika ukuran dunia nyata tidak diketahui, jangan mengarang ukuran.
    Sediakan faktor skala yang dapat dikalibrasi.
-   Pastikan hasil dilihat dari tampak atas agar dapat dibandingkan
    langsung dengan garis denah.

## 5. UI Editor yang diharapkan

Sediakan panel sederhana dengan: 1. Referensi gambar denah yang
terdeteksi. 2. Tombol `Extract Wall Lines`. 3. Tombol atau tampilan
`Preview Overlay`. 4. Pengaturan tinggi dinding, ketebalan, dan
toleransi penggabungan garis. 5. Tombol `Generate Walls`. 6. Tombol
`Regenerate` yang mengganti hasil `GeneratedWalls` lama tanpa menyentuh
objek lain. 7. Ringkasan jumlah segmen terdeteksi dan peringatan bagian
ambigu.

Jangan wajibkan pengguna menggambar garis dinding satu per satu. Jika
beberapa garis tidak dapat dikenali, tampilkan overlay dan jelaskan
bagian yang bermasalah; perbaiki preprocessing/algoritma terlebih dahulu
sebelum meminta koreksi manual.

## 6. Batasan dan penanganan ambiguitas

Denah adalah gambar raster, bukan CAD vektor. Garis dapat menyatu dengan
simbol kolom, teks, atau simbol pintu. Karena itu: - Utamakan
otomatisasi, tetapi jangan menjanjikan kesempurnaan tanpa validasi. -
Buat overlay agar kesalahan dapat diketahui sebelum dinding dibuat. -
Jangan memperlakukan semua garis gelap sebagai dinding. - Jika pintu
atau garis tertentu ambigu, laporkan secara spesifik. - Jangan mengklaim
akurasi penuh tanpa membandingkan hasil dengan denah.

## 7. Batas perubahan dan keamanan

-   Jangan membuat ulang project.
-   Jangan mengubah konfigurasi AR.
-   Jangan menghapus atau mengganti `AR Session`, `XR Origin`, kamera
    AR, `XR Environment`, atau komponen yang sudah ada.
-   Jangan mengubah gambar denah dan transform-nya.
-   Jangan memodifikasi file yang tidak diperlukan.
-   Sebelum implementasi, laporkan file yang akan ditambahkan/diubah.
-   Jangan menambah dependency baru tanpa menjelaskan alasannya.
-   Periksa error kompilasi setelah membuat file C#.
-   Jika butuh Python/OpenCV, periksa apakah sudah tersedia sebelum
    memasang apa pun.

## 8. Kriteria penerimaan

Implementasi dianggap memenuhi target jika: 1. Garis dinding diekstrak
otomatis dari sumber gambar denah yang ada. 2. Sistem membuat overlay
hasil deteksi. 3. Unity menghasilkan dinding 3D dari data hasil
ekstraksi, bukan dari koordinat contoh. 4. Dinding mengikuti batas luar
dan partisi ruangan sedekat mungkin dengan gambar. 5. Dinding
ditempatkan sejajar dengan denah berdasarkan transform yang sudah ada.
6. Bukaan pintu tidak ditutup secara sembarangan. 7. Generate ulang
tidak menghasilkan duplikasi. 8. Setup dan objek AR yang sudah ada tetap
utuh. 9. Hasil dan keterbatasan dilaporkan secara jujur.

## 9. Instruksi pertama untuk OpenCode

Baca dokumen ini, lalu audit project dan scene aktif. Temukan GameObject
denah beserta sumber gambar dan transform-nya. Laporkan: - Nama
GameObject denah dan sumber asset-nya. - Posisi, rotasi, dan skala
denah. - Pendekatan ekstraksi garis yang paling sesuai. - Daftar file
yang akan ditambahkan atau diubah.

Pada tahap audit, jangan mengubah scene atau file project. Setelah
rencana disetujui, implementasikan generator dinding sesuai spesifikasi
ini.
