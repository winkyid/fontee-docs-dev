# fonnte-whatsapp-integration.md

### Ringkasan
Dokumen ini merangkum cara menggunakan API Fonnte untuk mengirim pesan WhatsApp dengan PHP, menyertakan prasyarat, pola request utama (single, bulk, scheduled, dynamic, media), praktik terbaik untuk development, checklist repo demo, contoh skrip minimal, dan tip troubleshooting.

---

## Prasyarat
- **Akun Fonnte**: daftar, tambahkan device, salin **API token** dari menu device.  
- **PHP**: versi >= 7.1 dengan ekstensi **curl** aktif.  
- **Lingkungan dev**: XAMPP, LAMP, atau server yang mendukung multipart/form-data upload.  
- **Paket / Lisensi Fonnte**: periksa apakah paket mendukung attachment/media (mis. super/advanced/ultra).

---

## Struktur proyek (saran)
- .env (TOKEN, BASE_URL)  
- src/
  - send_single.php
  - send_bulk.php
  - send_schedule.php
  - send_media.php
- examples/
  - example_data.json
- README.md

---

## Endpoint dan pola request
- **URL**: POST https://api.fonnte.com/send  
- **Headers**: Authorization: TOKEN  
- **Content-Type**: multipart/form-data (umumnya)  
- **Field umum**:
  - **target**: nomor tujuan atau comma-separated untuk list sederhana  
  - **message**: teks pesan (bisa menyertakan placeholder dynamic seperti {name})  
  - **countryCode**: kode negara (contoh: 62)  
  - **delay**: jeda antar pesan dalam detik (untuk bulk)  
  - **schedule**: unix timestamp (detik) untuk penjadwalan  
  - **followup**: delay dalam detik untuk follow-up setelah request  
  - **data**: JSON array berisi objek pesan (untuk bulk lebih efisien)  
  - **url**: URL file publik untuk media  
  - **file**: file local diupload via CURLFile  
  - **filename**: nama file yang dikirim (opsional)  
  - **location**: "latitude,longitude" untuk mengirim lokasi  
  - **choices/select/pollname**: untuk membuat polling

---

## Use cases dan contoh payload singkat
#### Single send (form-data)
- Fields: **target**, **message**, **countryCode**  
- Gunakan untuk notifikasi satu-ke-satu.

#### Bulk send (rekomendasi: field data)
- Field: **data** = JSON-encoded array contoh:
  - [{"target":"62812...","message":"Halo {name}","countryCode":"62","data":{"name":"Budi"}} , ...]
- Lebih sedikit request HTTP, lebih efisien.

#### Schedule
- Tambahkan **schedule** = unix timestamp (detik) di objek pesan untuk penjadwalan.

#### Follow up
- Gunakan **followup** = X (detik) untuk mengirim pesan tindak lanjut otomatis setelah X detik.

#### Dynamic / personalisasi
- Pada **target** atau di **data** sertakan variable seperti: nomor|Nama|Var  
- Di **message** gunakan placeholder {name}, {var1} untuk diganti oleh server.

#### Media
- Media via **url**: field **url** menunjuk file publik.  
- Media via upload: gunakan field **file** dengan CURLFile dan optional **filename**.  
- Pastikan paket Fonnte mendukung attachment.

#### Polling
- Sertakan **choices**, **select**, dan **pollname** sesuai kebutuhan polling.

---

## Contoh skrip PHP minimal
##### Contoh: kirim single message (send_single.php)
```php
<?php
// .env atau config sederhana
$TOKEN = getenv('FONNTE_TOKEN') ?: 'YOUR_TOKEN';
$URL = 'https://api.fonnte.com/send';

$post = [
  'target' => '62812xxxxxxx',
  'message' => 'Halo, ini pesan percobaan dari Fonnte API',
  'countryCode' => '62'
];

$ch = curl_init();
curl_setopt($ch, CURLOPT_URL, $URL);
curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_HTTPHEADER, ["Authorization: $TOKEN"]);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_POSTFIELDS, $post);

$response = curl_exec($ch);
if ($response === false) {
  error_log('cURL error: ' . curl_error($ch));
} else {
  // log response untuk debugging
  file_put_contents('logs/send_single.log', date('c') . " " . $response . PHP_EOL, FILE_APPEND);
}
curl_close($ch);
echo $response;
```

##### Contoh: kirim media via URL (send_media.php)
```php
<?php
$TOKEN = getenv('FONNTE_TOKEN') ?: 'YOUR_TOKEN';
$URL = 'https://api.fonnte.com/send';

$post = [
  'target' => '62812xxxxxxx',
  'message' => 'File terlampir, cek link.',
  'countryCode' => '62',
  'url' => 'https://example.com/image.jpg',
  'filename' => 'promo.jpg'
];

$ch = curl_init();
curl_setopt($ch, CURLOPT_URL, $URL);
curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_HTTPHEADER, ["Authorization: $TOKEN"]);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_POSTFIELDS, $post);

$response = curl_exec($ch);
if ($response === false) {
  error_log('cURL error: ' . curl_error($ch));
} else {
  file_put_contents('logs/send_media.log', date('c') . " " . $response . PHP_EOL, FILE_APPEND);
}
curl_close($ch);
echo $response;
```

---

## Praktik terbaik pengembangan
- **Gunakan field data untuk bulk** daripada loop melakukan banyak request HTTP.  
- **Tambahkan delay** antar pengiriman besar untuk menghindari throttling.  
- **Log lengkap** request/response dan curl_error untuk debugging.  
- **Simpan token di .env** atau secret manager; jangan commit token ke repo.  
- **Validasi nomor** sebelum mengirim (format internasional + countryCode).  
- **Periksa paket** yang aktif untuk memastikan fitur attachment/media tersedia.  
- **Tangani error HTTP dan response API**: retry terbatas untuk transient errors, catat permanent failures.

---

## Checklist dokumentasi pembelajaran (untuk repo)
- [ ] .env.example dengan variabel yang jelas (FONNTE_TOKEN, BASE_URL)  
- [ ] Contoh skrip: single, bulk (data), schedule, followup, media  
- [ ] README yang menjelaskan prasyarat dan cara jalankan (php -S atau CLI)  
- [ ] scripts/ atau makefile untuk menjalankan demo lokal  
- [ ] contoh data JSON untuk testing broadcast dan dynamic variables  
- [ ] file logs/ dan instruksi rotasi log sederhana

---

## Troubleshooting singkat
- Jika curl_exec mengembalikan false: periksa curl_error untuk detail.  
- Response error berkaitan dengan token: verifikasi token di .env dan device Fonnte.  
- Gagal upload file: pastikan PHP mengizinkan upload dan batas ukuran upload (upload_max_filesize, post_max_size).  
- Pesan tidak terkirim ke banyak nomor: gunakan field **data** atau tambahkan **delay** untuk mengurangi penolakan.

---
