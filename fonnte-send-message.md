# fonnte-send-message.md

### Ringkasan
Dokumen ini merangkum penggunaan endpoint POST https://api.fonnte.com/send untuk mengirim pesan WhatsApp melalui API Fonnte, meliputi parameter tersedia, contoh PHP, respons sukses/gagal, dan rekomendasi pengembangan serta troubleshooting.

---

## Prasyarat
- Token API dari menu device di dashboard Fonnte (isi header Authorization dengan token, tidak perlu kata "Bearer").  
- PHP dengan ekstensi curl aktif untuk contoh kode yang disediakan.  

---

## Endpoint dan header
- Endpoint: POST https://api.fonnte.com/send.  
- Header wajib: Authorization: TOKEN (TOKEN langsung, tanpa Bearer).

---

## Parameter utama (ringkas)
- target (required, string): nomor tujuan, dipisah koma; mendukung group id dan rotator id; nilai harus string.  
- message (optional, string): teks pesan, mendukung variable dan emoji; batas 60.000 karakter.  
- url (optional, string): public url file (image/file/audio/video); hanya tersedia untuk paket super/advanced/ultra; url harus langsung file (bukan halaman).  
- file (optional, binary): upload file langsung dari localhost/form menggunakan CURLFile; tersedia pada paket super/advanced/ultra.  
- filename (optional, string): nama berkas yang diterima; hanya berlaku untuk non-image/non-video.  
- schedule (optional, int): unix timestamp untuk penjadwalan pengiriman; perhatikan konversi timezone saat membuat timestamp.  
- delay (optional, string): string atau range seperti "5" atau "1-10" untuk delay antar target pada multi-target; harus string.  
- countryCode (optional, string): menggantikan atau menambahkan kode negara pada nomor; default "62"; set "0" untuk menonaktifkan filter penggantian otomatis.  
- location (optional, string): format "latitude,longitude" untuk mengirim lokasi.  
- typing (optional, bool): tampilkan indikator mengetik; default false.  
- choices / select / pollname (optional): parameter untuk membuat polling (choices min 2, max 12; select "single" atau "multiple").  
- connectOnly (optional, bool): bila true (default) permintaan ditolak saat device terputus; jika false request disimpan dan diproses saat device kembali online.  
- data (optional, string): JSON-encoded array (string) untuk menggabungkan banyak request dalam satu panggilan; memudahkan sequencing, delay antar pesan, dan pengiriman berurutan; tidak dapat digunakan bersama parameter file.  
- sequence (optional, bool): memaksa eksekusi berurutan; menonaktifkan delay/schedule/followup pada grup tersebut; default false.  
- preview (optional, bool): tentukan apakah link diparsing untuk preview; default true; set false untuk menonaktifkan preview parsing oleh Fonnte.

---

## Contoh respons
- Contoh sukses: detail "success! message in queue", id array, process "pending", requestid, status true.  
- Contoh kegagalan: status false dengan field reason (contoh: "token invalid", "input invalid", "url invalid", "file format not supported", "file size must under 4MB", "target invalid", "JSON format invalid", "insufficient quota").

---

## Contoh kode PHP minimal
```php
<?php
$curl = curl_init();
curl_setopt_array($curl, array(
  CURLOPT_URL => 'https://api.fonnte.com/send',
  CURLOPT_RETURNTRANSFER => true,
  CURLOPT_POST => true,
  CURLOPT_POSTFIELDS => array(
    'target' => '08123456789|Fonnte|Admin,08123456789|Lili|User',
    'message' => 'test message to {name} as {var1}',
    'url' => 'https://md.fonnte.com/images/wa-logo.png',
    'filename' => 'filename',
    'schedule' => 0,
    'typing' => false,
    'delay' => '2',
    'countryCode' => '62',
    // 'file' => new CURLFile('localfile.jpg'),
    'location' => '-7.983908, 112.621391',
    'followup' => 0,
  ),
  CURLOPT_HTTPHEADER => array('Authorization: TOKEN'),
));
$response = curl_exec($curl);
if (curl_errno($curl)) {
  $error_msg = curl_error($curl);
}
curl_close($curl);
if (isset($error_msg)) { echo $error_msg; }
echo $response;
```
Kode contoh ini berasal dari dokumentasi resmi dan mencerminkan pola penggunaan parameter yang umum.

---

## Praktik terbaik pengembangan
- Gunakan parameter data (JSON string) untuk menggabungkan banyak permintaan jadi satu panggilan ketika memerlukan kontrol sequencing dan delay antar pesan; ingat bahwa file tidak didukung di mode data.  
- Untuk bulk sederhana tanpa data, gunakan field target berisi banyak nomor yang dipisah koma dan atur delay sebagai string untuk menghindari masalah throttling.  
- Simpan token di .env atau secret manager; jangan commit token ke repository.  
- Validasi format nomor sebelum dikirim dan gunakan countryCode dengan benar untuk menghindari target invalid.  
- Perhatikan batas ukuran file (maks 4MB) dan format file yang didukung ketika mengirim lampiran melalui url atau file.  

---

## Troubleshooting cepat
- "token invalid": periksa token di header Authorization dan pastikan token valid untuk account/device yang sama.  
- "url invalid" / "url unreachable": pastikan url menunjuk langsung ke file publik dan dapat diakses dari internet.  
- "file size must under 4MB": kompres file agar di bawah batas 4MB sebelum upload.  
- "JSON format invalid": ketika menggunakan parameter data pastikan json_encode string dilakukan dengan benar dan dikirim sebagai string, bukan array langsung.  
- Jika device terputus dan Anda ingin queue tetap tersimpan, set connectOnly ke false; default connectOnly true akan menolak request saat device offline.

---

## Contoh pola penggunaan data (ringkas)
- data harus dikirim sebagai string JSON ter-encode, contoh:
  - '[{"target":"08123456789","message":"1"},{"target":"08123456789","message":"2","delay":"2"},{"target":"08123456789","message":"3","delay":"1"}]'  
- data memungkinkan urutan pengiriman bertingkat: pesan 1 -> tunggu -> pesan 2 -> tunggu -> pesan 3.

---

## Catatan akhir
Dokumen ini menyederhanakan ringkasan parameter dan contoh dari halaman dokumentasi Fonnte untuk keperluan pembelajaran dan pengembangan; selengkapnya dan referensi parameter lanjutan dapat dilihat pada dokumentasi resmi Fonnte.
