# <img src="http://yopedia.test/icon.png" width="40" height="40" alt="YoPedia Logo" align="center" style="border-radius: 4px;" /> YoPedia — Self-Hosted Payment Link Gateway

[![PHP Version](https://img.shields.io/badge/PHP-%3E%3D%208.0-777bb4?style=flat&logo=php&logoColor=white)](https://www.php.net/)
[![Database](https://img.shields.io/badge/MySQL-Database-4479a1?style=flat&logo=mysql&logoColor=white)](https://www.mysql.com/)
[![Payment Gateway](https://img.shields.io/badge/Gateway-Duitku-059669?style=flat&logo=emerald&logoColor=white)](https://duitku.com/)
[![License](https://img.shields.io/badge/License-Proprietary-gray?style=flat)](#)

YoPedia adalah platform *self-hosted payment link* & *payment gateway* modern, dinamis, dan responsif yang dirancang untuk memudahkan integrasi pembayaran online (QRIS, E-Wallet, & Virtual Account) ke berbagai proyek aplikasi atau website Anda melalui satu dasbor pusat.

---

## 🚀 Fitur Utama

- **Multi-Proyek (Merchant Slugs)**: Kelola banyak proyek/aplikasi dengan *API Key*, *prefix transaksi*, logo khusus, dan *callback URL* masing-masing.
- **Pemisahan Lingkungan**: Mendukung mode **Sandbox (Uji Coba)** dan **Live (Production)** dengan pencatatan saldo terpisah (`sandbox_balance` vs `pending_balance`).
- **Simulasi Pembayaran (Sandbox)**: Tombol simulasi instan langsung dari halaman pembayaran uji coba untuk memudahkan developer melakukan testing webhook.
- **Konfigurasi Aksi Kembali**: Pengaturan fleksibel per proyek untuk menutup jendela (*close tab*) atau dialihkan kembali ke website merchant setelah pembayaran selesai.
- **Log Webhook Transparan**: Riwayat detail pengiriman callback webhook ke server merchant lengkap dengan request payload, kode HTTP, dan response body.
- **UI Pembayaran Clean & Hijau**: Tampilan halaman pembayaran bersih, ramah seluler, minimalis, dan elegan dengan warna hijau sebagai aksen utama.

---

## 🛠️ Tech Stack & Persyaratan

- **Bahasa Pemrograman**: PHP `>= 8.0` (Diuji dan berjalan dengan baik pada **PHP 8.1.10**)
- **Web Server**: Apache / Nginx
- **Database**: MySQL `>= 5.7` atau MariaDB `>= 10.3` (Menggunakan driver **PDO**)
- **Ekstensi PHP Wajib**: `pdo_mysql`, `curl`, `json`, `openssl`
- **Styling**: Vanilla CSS / Tailwind CSS v3 (Clean Flat & Responsive Design)
- **Integrasi PG**: API Duitku (Virtual Account, QRIS, E-Wallet)

---

## 📚 Panduan Dokumentasi API

Seluruh request API wajib menyertakan tipe data `application/json` pada header request.

### 1. Membuat Invoice / Payment Link (Tanpa Menentukan Metode)
Membuat invoice pembayaran baru dan mendapatkan tautan pembayaran (`payment_link`) di mana pelanggan akan memilih sendiri metode pembayarannya di halaman selektor hijau YoPedia.

* **Method**: `POST`
* **Endpoint**: `/api/transactioncreate`
* **Request Body**:
  ```json
  {
    "project": "slug-proyek-anda",
    "order_id": "INV-123456",
    "amount": 50000,
    "api_key": "API_KEY_PROYEK_ANDA",
    "redirect_url": "https://website-anda.com/finish" (Opsional)
  }
  ```
* **Contoh Response**:
  ```json
  {
    "invoice_id": "INV1fa7e023b",
    "amount": 50000,
    "fee": 0,
    "total": 50000,
    "qris_image": "",
    "payment_link": "http://yopedia.test/payment/INV1fa7e023b",
    "expired_at": "2026-07-16T22:30:00+07:00",
    "order_id": "INV-123456"
  }
  ```

### 2. Membuat Transaksi V2 (Dengan Metode Pembayaran Spesifik)
Membuat transaksi baru dengan langsung menentukan metode pembayarannya dari API (misal `qris`, `bni_va`, dll.).
- **QRIS**: Menghasilkan string payload QRIS (`payment_number`) dan tautan ke halaman detail rincian pembayaran YoPedia (`payment_link`).
- **Virtual Account**: Menghasilkan nomor VA (`payment_number`) dan tautan rincian pembayaran YoPedia (`payment_link`). *Catatan:* Tautan `paymentUrl` bawaan Duitku diabaikan (tidak digunakan) karena YoPedia memproses dan menampilkan VA & panduan bayarnya secara internal.
- **E-Wallet**: Khusus untuk E-wallet, respons API tetap menyertakan tautan `payment_url` bawaan Duitku guna mengarahkan/membuka aplikasi E-wallet terkait.

* **Method**: `POST`
* **Endpoint**: `/api/transactioncreate/{method_code}`
* **Request Body**:
  ```json
  {
    "project": "slug-proyek-anda",
    "order_id": "INV-123456",
    "amount": 50000,
    "api_key": "API_KEY_PROYEK_ANDA",
    "redirect_url": "https://website-anda.com/finish" (Opsional)
  }
  ```
* **Contoh Response (QRIS)**:
  ```json
  {
    "payment": {
      "project": "slug-proyek-anda",
      "order_id": "INV-123456",
      "amount": 50000,
      "fee": 1000,
      "total_payment": 51000,
      "payment_method": "qris",
      "payment_number": "00020101021226610016ID.CO.SHOPEE...",
      "va_number": "",
      "payment_url": "",
      "qr_string": "00020101021226610016ID.CO.SHOPEE...",
      "qris_image": "http://yopedia.test/public/uploads/qris/INV-123456.png",
      "payment_link": "http://yopedia.test/payment/detail/INV-123456",
      "expired_at": "2026-07-16T22:30:00+07:00"
    }
  }
  ```

* **Contoh Response (Virtual Account - VA)**:
  ```json
  {
    "payment": {
      "project": "slug-proyek-anda",
      "order_id": "INV-123456",
      "amount": 50000,
      "fee": 4000,
      "total_payment": 54000,
      "payment_method": "bri_va",
      "payment_number": "1308200985000957",
      "va_number": "1308200985000957",
      "payment_url": "",
      "qr_string": "",
      "qris_image": "",
      "payment_link": "http://yopedia.test/payment/detail/INV-123456",
      "expired_at": "2026-07-16T22:30:00+07:00"
    }
  }
  ```

* **Contoh Response (E-Wallet)**:
  ```json
  {
    "payment": {
      "project": "slug-proyek-anda",
      "order_id": "INV-123456",
      "amount": 50000,
      "fee": 1500,
      "total_payment": 51500,
      "payment_method": "shopeepay",
      "payment_number": "https://sandbox.com/topup/topupdirectv2.aspx?ref=SA26...",
      "va_number": "",
      "payment_url": "https://sandbox..com/topup/topupdirectv2.aspx?ref=SA26...",
      "qr_string": "",
      "qris_image": "",
      "payment_link": "http://yopedia.test/payment/detail/INV-123456",
      "expired_at": "2026-07-16T22:30:00+07:00"
    }
  }
  ```


### 3. Cek Detail Transaksi
Mendapatkan informasi detail dan status terbaru dari transaksi tertentu.

* **Method**: `GET`
* **Endpoint**: `/api/transactiondetail?project={slug}&amount={amount}&order_id={order_id}&api_key={api_key}`
* **Contoh Response**:
  ```json
  {
    "transaction": {
      "amount": 50000,
      "order_id": "INV-123456",
      "project": "slug-proyek-anda",
      "status": "completed",
      "payment_method": "qris",
      "completed_at": "2026-07-16T21:40:02+07:00"
    }
  }
  ```
### 4. Dapatkan Metode Pembayaran & Biaya Admin (Fee)
Mendapatkan daftar metode pembayaran yang aktif beserta biaya admin (fee) dinamis yang disesuaikan dengan nominal transaksi.

* **Method**: `GET`
* **Endpoint**: `/api/payment/methods?project={slug}&amount={amount}&api_key={api_key}`
* **Contoh Response**:
  ```json
  {
    "success": true,
    "project": "slug-proyek-anda",
    "amount": 50000,
    "payment_methods": [
      {
        "code": "qris",
        "name": "QRIS",
        "fee": 1000,
        "total_payment": 51000,
        "min_amount": 1000
      },
      {
        "code": "bni_va",
        "name": "BNI Virtual Account",
        "fee": 4000,
        "total_payment": 54000,
        "min_amount": 10000
      }
    ]
  }
  ```


### 5. Simulasi Pembayaran Sukses (Sandbox Only)
Melakukan simulasi pembayaran sukses tanpa perlu melakukan transfer nyata. Hanya berlaku untuk transaksi berstatus `sandbox`.

* **Method**: `POST`
* **Endpoint**: `/api/paymentsimulation`
* **Request Body**:
  ```json
  {
    "project": "slug-proyek-anda",
    "order_id": "INV-123456",
    "api_key": "API_KEY_PROYEK_ANDA"
  }
  ```
* **Contoh Response**:
  ```json
  {
    "success": true,
    "message": "Simulasi pembayaran sukses berhasil dikirim."
  }
  ```

### 6. Membatalkan Transaksi
Membatalkan transaksi tertunda agar status berubah menjadi `expired` / `failed` secara instan.

* **Method**: `POST`
* **Endpoint**: `/api/transactioncancel`
* **Request Body**:
  ```json
  {
    "project": "slug-proyek-anda",
    "order_id": "INV-123456",
    "api_key": "API_KEY_PROYEK_ANDA"
  }
  ```
* **Contoh Response**:
  ```json
  {
    "success": true,
    "message": "Transaksi berhasil dibatalkan."
  }
  ```

---

## 🔔 Webhook Callback (Merchant HTTP POST)

Ketika pelanggan menyelesaikan pembayaran, sistem YoPedia akan mengirimkan data callback secara otomatis menggunakan HTTP **`POST`** ke `callback_url` proyek Anda dengan format body JSON berikut:

```json
{
  "amount": 50000,
  "order_id": "INV-123456",
  "project": "slug-proyek-anda",
  "status": "completed",
  "payment_method": "qris",
  "completed_at": "2026-07-16T21:40:02+07:00"
}
```

---

## 🔒 Lisensi & Hak Cipta
Hak Cipta © 2026 **YoPedia**. Dikembangkan dengan dedikasi penuh untuk efisiensi transaksi Anda.
