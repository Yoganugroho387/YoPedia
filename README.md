<div align="center">

<img src="icon.png" width="80" height="80" alt="YoPedia Logo" style="border-radius: 12px; margin-bottom: 10px;" />

# YoPedia — Self-Hosted Payment Link Gateway

[![PHP Version](https://img.shields.io/badge/PHP-%3E%3D%208.0-777bb4?style=flat-square&logo=php&logoColor=white)](https://www.php.net/)
[![Database](https://img.shields.io/badge/MySQL-Database-4479a1?style=flat-square&logo=mysql&logoColor=white)](https://www.mysql.com/)
[![Gateway](https://img.shields.io/badge/Gateway-Duitku-059669?style=flat-square&logo=emerald&logoColor=white)](https://duitku.com/)
[![Status](https://img.shields.io/badge/Status-Stable-blue?style=flat-square)](#)

YoPedia is a modern, self-hosted payment link gateway built with PHP. It allows businesses and developers to manage multiple payment projects, issue dynamic checkout URLs, and accept payments through QRIS, Virtual Accounts, and E-Wallets via Duitku integration.

[Key Features](#key-features) • [Installation](#installation) • [API Documentation](#api-documentation) • [Webhook Integration](#webhook-integration)

</div>

---

## Key Features

* **Multi-Project Management (Merchant Slugs)**: Manage multiple websites or apps using unique API keys, custom transaction prefixes, dedicated logos, and callback URLs.
* **Dual Environments**: Fully separated Sandbox and Production environments with distinct user balance storage (`sandbox_balance` vs `pending_balance`).
* **Sandbox Payment Simulator**: Trigger successful payment flows directly from the checkout detail page for seamless sandbox webhook testing.
* **Custom Back Button Behavior**: Configure projects to either close the window/tab or redirect users back to the merchant's checkout/callback page upon payment completion.
* **Webhook Logger**: Transparent dashboard tracking all sent webhooks, HTTP response codes, payloads, and response bodies.
* **Elegant Checkout Interface**: A responsive, clean, green-themed payment selection and invoice receipt page tailored for desktop and mobile viewports.

---

## Tech Stack & Prerequisites

* **Backend**: PHP 8.0 or higher (Tested and fully compatible with PHP 8.1.10)
* **Web Server**: Apache / Nginx
* **Database**: MySQL 5.7+ / MariaDB 10.3+ (PDO driver enabled)
* **PHP Extensions**: `pdo_mysql`, `curl`, `json`, `openssl`
* **Styling**: Vanilla CSS / Tailwind CSS v3

---

## Installation

1. **Clone the Repository**:
   ```bash
   git clone https://github.com/yourusername/yopedia.git
   ```
2. **Configure Environment Variables**:
   Copy `.env.example` to `.env` and fill in your database and Duitku API credentials:
   ```env
   DB_HOST=localhost
   DB_NAME=db_yopedia_cv
   DB_USER=root
   DB_PASS=yourpassword
   
   DUITKU_MERCHANT_CODE=your_merchant_code
   DUITKU_API_KEY=your_api_key
   ```
3. **Database Migration**:
   Import `schema.sql` into your database.
4. **Configure Web Server**:
   Point your domain or local vhost document root to the `public/` directory.

---

## API Documentation

All requests should be sent as JSON payloads with the header `Content-Type: application/json`.

### 1. Create Invoice (Without Payment Method)

Creates a new transaction invoice and returns a `payment_link`. The customer selects their payment method on the YoPedia checkout page.

* **Method**: `POST`
* **Endpoint**: `/api/transactioncreate`
* **Request Body**:
  ```json
  {
    "project": "project-slug",
    "order_id": "INV-100020",
    "amount": 50000,
    "api_key": "YOUR_PROJECT_API_KEY",
    "redirect_url": "https://yourwebsite.com/success"
  }
  ```
* **Response**:
  ```json
  {
    "invoice_id": "INV09dcb0c9b",
    "amount": 50000,
    "fee": 0,
    "total": 50000,
    "qris_image": "",
    "payment_link": "http://yopedia.test/payment/INV09dcb0c9b",
    "expired_at": "2026-07-16T22:30:00+07:00",
    "order_id": "INV-100020"
  }
  ```

### 2. Create Transaction V2 (With Specific Payment Method)

Directly initiates a transaction with a pre-selected payment method (e.g. `qris`, `bni_va`, `shopeepay`). 

* **Method**: `POST`
* **Endpoint**: `/api/transactioncreate/{method_code}`

#### QRIS Response Example
For QRIS (`qris`), the response includes the raw QRIS payload (`qr_string`), an image link (`qris_image`), and the checkout details page (`payment_link`).
```json
{
  "payment": {
    "project": "project-slug",
    "order_id": "INV-100020",
    "amount": 50000,
    "fee": 1000,
    "total_payment": 51000,
    "payment_method": "qris",
    "payment_number": "00020101021226610016ID.CO.SHOPEE...",
    "va_number": "",
    "payment_url": "",
    "qr_string": "00020101021226610016ID.CO.SHOPEE...",
    "qris_image": "http://yopedia.test/public/uploads/qris/INV-100020.png",
    "payment_link": "http://yopedia.test/payment/detail/INV-100020",
    "expired_at": "2026-07-16T22:30:00+07:00"
  }
}
```

#### Virtual Account Response Example
For VA (e.g. `bri_va`), the response includes the Virtual Account number (`va_number`) and instructions link (`payment_link`). The Duitku `payment_url` is bypassed/empty for VAs as YoPedia handles VA displays internally.
```json
{
  "payment": {
    "project": "project-slug",
    "order_id": "INV-100020",
    "amount": 50000,
    "fee": 4000,
    "total_payment": 54000,
    "payment_method": "bri_va",
    "payment_number": "1308200985000957",
    "va_number": "1308200985000957",
    "payment_url": "",
    "qr_string": "",
    "qris_image": "",
    "payment_link": "http://yopedia.test/payment/detail/INV-100020",
    "expired_at": "2026-07-16T22:30:00+07:00"
  }
}
```

#### E-Wallet Response Example
For E-Wallets (e.g. `shopeepay`), the response includes a Duitku `payment_url` to redirect the customer to the app or payment session.
```json
{
  "payment": {
    "project": "project-slug",
    "order_id": "INV-100020",
    "amount": 50000,
    "fee": 1500,
    "total_payment": 51500,
    "payment_method": "shopeepay",
    "payment_number": "https://sandbox.com/topup/topupdirectv2.aspx?ref=SA26...",
    "va_number": "",
    "payment_url": "https://sandbox.com/topup/topupdirectv2.aspx?ref=SA26...",
    "qr_string": "",
    "qris_image": "",
    "payment_link": "http://yopedia.test/payment/detail/INV-100020",
    "expired_at": "2026-07-16T22:30:00+07:00"
  }
}
```

### 3. Check Transaction Status

* **Method**: `GET`
* **Endpoint**: `/api/transactiondetail?project={slug}&amount={amount}&order_id={order_id}&api_key={api_key}`
* **Response**:
  ```json
  {
    "transaction": {
      "amount": 50000,
      "order_id": "INV-100020",
      "project": "project-slug",
      "status": "completed",
      "payment_method": "qris",
      "completed_at": "2026-07-16T21:40:02+07:00"
    }
  }
  ```

### 4. Get Payment Methods & Fees

Retrieves a list of all active payment methods and their dynamic calculated fees for a given transaction amount.

* **Method**: `GET`
* **Endpoint**: `/api/payment/methods?project={slug}&amount={amount}&api_key={api_key}`
* **Response**:
  ```json
  {
    "success": true,
    "project": "project-slug",
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

### 5. Simulate Payment (Sandbox Only)

* **Method**: `POST`
* **Endpoint**: `/api/paymentsimulation`
* **Request Body**:
  ```json
  {
    "project": "project-slug",
    "order_id": "INV-100020",
    "api_key": "YOUR_PROJECT_API_KEY"
  }
  ```
* **Response**:
  ```json
  {
    "success": true,
    "message": "Simulasi pembayaran sukses berhasil dikirim."
  }
  ```

### 6. Cancel Transaction

* **Method**: `POST`
* **Endpoint**: `/api/transactioncancel`
* **Request Body**:
  ```json
  {
    "project": "project-slug",
    "order_id": "INV-100020",
    "api_key": "YOUR_PROJECT_API_KEY"
  }
  ```
* **Response**:
  ```json
  {
    "success": true,
    "message": "Transaksi berhasil dibatalkan."
  }
  ```

---

## Webhook Integration

When a transaction is successfully completed, YoPedia will send a secure HTTP `POST` webhook to your project's configured `callback_url` with the following JSON payload:

```json
{
  "amount": 50000,
  "order_id": "INV-100020",
  "project": "project-slug",
  "status": "completed",
  "payment_method": "qris",
  "completed_at": "2026-07-16T21:40:02+07:00"
}
```

---

## License
Copyright © 2026 YoPedia. All rights reserved.
