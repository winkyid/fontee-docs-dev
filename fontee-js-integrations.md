# Mengirim Pesan API (Fonnte) – JS/Next.js/React Modern

### Contoh kode dasar untuk mengirim pesan via API:

```javascript
// contoh di Next.js API Route atau Node.js
import FormData from 'form-data';
import fs from 'fs';
import fetch from 'node-fetch';

const form = new FormData();

form.append('target', '08123456789|Fonnte|Admin,08123456789|Lili|User');
form.append('message', 'test message to {name} as {var1}');
form.append('url', 'https://md.fonnte.com/images/wa-logo.png');
form.append('filename', 'filename');
form.append('schedule', 0);
form.append('typing', false);
form.append('delay', '2');
form.append('countryCode', '62');
form.append('file', fs.createReadStream('localfile.jpg'));
form.append('location', '-7.983908, 112.621391');
form.append('followup', 0);

const response = await fetch('https://api.fonnte.com/send', {
  method: 'POST',
  headers: {
    Authorization: 'TOKEN', // ganti dengan token kamu
  },
  body: form,
});

const result = await response.json();
console.log(result);
```

> Catatan: Jika di browser/React client-side, **tidak bisa langsung akses file lokal**. Gunakan `<input type="file">` dan kirim file melalui `FormData`.

---

### Parameter yang Tersedia:

* **target** (required) (string) – nomor tujuan, dipisahkan koma, mendukung variable. [baca selengkapnya]
* **message** (optional) (string) – teks pesan, mendukung variable. [baca selengkapnya]
* **url** (optional) (string) – mendukung image, file, audio, video. [baca selengkapnya]
* **filename** (optional) (string) – custom filename. [baca selengkapnya]
* **schedule** (optional) (int) – unix timestamp untuk pengiriman terjadwal. [baca selengkapnya]
* **delay** (optional) (string) – delay pengiriman pesan, bisa rentang misal `1-10`. [baca selengkapnya]
* **countryCode** (string) – ganti angka 0 awal dengan kode negara, default 62. [baca selengkapnya]
* **location** (optional) (string) – format latitude,longitude. [baca selengkapnya]
* **typing** (optional) (bool) – indikator mengetik, default false. [baca selengkapnya]
* **choices** (optional) (string) – opsi polling, minimal 2, maksimal 12, dipisahkan koma. [baca selengkapnya]
* **select** (optional) (string) – limit polling: single/multiple. [baca selengkapnya]
* **pollname** (optional) (string) – nama polling. [baca selengkapnya]
* **file** (optional) (binary) – upload file dari server/user upload, max 4MB. [baca selengkapnya]
* **connectOnly** (optional) (bool) – hanya kirim jika device connected. [baca selengkapnya]
* **followup** (optional) (int) – delay pengiriman pesan. [baca selengkapnya]
* **data** (optional) (string) – gabungkan semua request menjadi satu. [baca selengkapnya]
* **sequence** (optional) (bool) – kirim pesan berurutan, default false. [baca selengkapnya]
* **preview** (optional) (bool) – link di pesan akan ada preview, default true. [baca selengkapnya]

---

### Catatan:

* **TOKEN** wajib diisi, bisa lebih dari satu dipisahkan koma (`xxxxx,yyyyy`) untuk rotator.
* Mengirim beberapa nomor bisa dipisah koma (`6282xxxx,6285xxxx,6283xxxx`).
* Variabel didukung (`{name}`, `{var1}`).
* Filename hanya berlaku untuk file/audio.
* Perhatikan batasan file attachment.

---

### Penjelasan Parameter

**target (string)**

* Bisa nomor WA, group ID, rotator ID.
* Harus string, dipisah koma.

**message (string)**

* Bisa emoji, max 60.000 karakter.
* Menjadi deskripsi file jika `url/file` didefinisikan.

**url (string)**

* Hanya untuk super/advanced/ultra plan.
* Harus public URL, bukan localhost atau IP private.

**filename (string)**

* Hanya untuk file/audio, bukan image/video.

**schedule (int)**

* Unix timestamp.
* Contoh PHP: `strtotime('2025-01-09T20:46:00+0700')`.

**delay (string)**

* Delay per target. Bisa fixed `'5'` atau range `'5-10'`.

**countryCode (string)**

* Default `62`.
* Bisa set `'0'` untuk bypass filter.

**location (string)**

* Format latitude,longitude.

**typing (bool)**

* Tampilkan indikator mengetik.

**choices, select, pollname**

* Parameter polling.

**file (binary)**

* Upload file dari server atau user.

**connectOnly (bool)**

* Default `true`, jika false pesan tetap disimpan meski device offline.

**data (string)**

* Gabungkan beberapa request jadi satu, gunakan `JSON.stringify`.

**sequence (bool)**

* Pastikan urutan pengiriman pesan.

**preview (bool)**

* Default `true`, bisa dimatikan untuk link tertentu.

---

### Contoh Response Sukses

```json
{
  "detail": "success! message in queue",
  "id": ["80367170"],
  "process": "pending",
  "requestid": 2937124,
  "status": true,
  "target": ["6282227097005"]
}
```

**Detail status:**

1. Success! message in queue → langsung diproses
2. Success! message will be sent on scheduled time → diproses sesuai schedule
3. Success! message pending due to server issue → disimpan, dikirim saat server siap

---

### Contoh Response Gagal

```json
{
  "Status": false,
  "reason": "token invalid",
  "requestid": 2937124
}
```

* Devices must belong to an account → token beda akun
* Input invalid → nilai parameter salah
* URL/file invalid → URL/file tidak bisa diakses
* File format / size → format tidak didukung atau >4MB
* Target invalid → nomor tidak valid
* JSON format invalid → format JSON salah
* Insufficient quota → kuota habis

---

### Knowledge Related / Deprecated

* **API Status Pesan Pemeriksaan** (deprecated)
* **API Nomor Validasi** → cek WA terdaftar
* **API Dapatkan QR** → scan perangkat di luar Fonnte

---

Made with ♥ in Indonesia

---
