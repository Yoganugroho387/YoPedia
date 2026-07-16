<div align="center">
  <img src="icon.png" width="100" height="100" alt="YoPedia Logo" style="border-radius: 16px; margin-bottom: 16px;" />
  
  # YoPedia — Self-Hosted Payment Link Gateway
  
  A modern, lightweight, and secure self-hosted payment gateway for developers and online merchants.
  
  <p>
    <a href="#key-features">Key Features</a> • 
    <a href="#tech-stack">Tech Stack</a> • 
    <a href="#installation">Installation</a> • 
    <a href="#api-documentation">API Documentation</a> • 
    <a href="#webhook-integration">Webhooks</a>
  </p>

  <hr />
</div>

## Key Features

<table width="100%">
  <tr>
    <td width="50%" valign="top">
      <strong>Multi-Project Slugs</strong><br />
      Manage payment flows for multiple websites or applications from a single dashboard using custom API keys, slugs, and transaction prefixes.
    </td>
    <td width="50%" valign="top">
      <strong>Dual-Mode Sandbox & Live</strong><br />
      Separate your testing and production data completely. Track virtual sandbox funds and real cleared funds in independent balances.
    </td>
  </tr>
  <tr>
    <td width="50%" valign="top">
      <strong>Sandbox Payment Simulator</strong><br />
      Trigger successful payment callbacks with a single click in sandbox mode to easily verify merchant API integrations and webhooks.
    </td>
    <td width="50%" valign="top">
      <strong>Custom Redirections</strong><br />
      Configure the "Return to Merchant" action per project to either automatically close the tab or redirect back to your website.
    </td>
  </tr>
  <tr>
    <td width="50%" valign="top">
      <strong>Transparent Webhook Logs</strong><br />
      Monitor the exact payload, HTTP response status, and response bodies for all outgoing callbacks to your merchant server.
    </td>
    <td width="50%" valign="top">
      <strong>Responsive & Minimalist UI</strong><br />
      An elegant, green-accented invoice receipt and payment selector design engineered to fit all desktop and mobile screens.
    </td>
  </tr>
</table>

---

## Tech Stack

* **Language**: PHP 8.0+ (Fully compatible with PHP 8.1+)
* **Database**: MySQL 5.7+ / MariaDB 10.3+ (PDO driver)
* **Web Server**: Apache / Nginx (Supports custom URL rewriting)
* **Extensions**: `pdo_mysql`, `curl`, `json`, `openssl`
* **Styling**: Vanilla CSS / Tailwind CSS v3

---

## Installation

1. **Clone the Repository**:
   ```bash
   git clone https://github.com/yourusername/yopedia.git
   ```
2. **Setup Environment Variables**:
   Copy `.env.example` to `.env` and fill in your database and API credentials:
   ```env
   DB_HOST=localhost
   DB_NAME=db_yopedia_cv
   DB_USER=root
   DB_PASS=yourpassword
   ```
3. **Database Migration**:
   Import `schema.sql` into your database.
4. **Point Domain Root**:
   Configure your server virtual host to point to the `public/` directory.

---

## API Documentation

All request payloads should use the `application/json` format.

### 1. Create Invoice (Without Payment Method)

Creates a new invoice and returns a `payment_link`. The customer selects their payment method on the YoPedia payment page.

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

Creates a transaction with a pre-selected payment method (e.g., `qris`, `bni_va`, `shopeepay`).

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
For Virtual Accounts (e.g. `bri_va`), the response includes the VA number (`va_number`) and instructions page (`payment_link`). The `payment_url` is empty since YoPedia renders instructions internally.
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
For E-Wallets (e.g. `shopeepay`), the response includes a `payment_url` to redirect the customer to open the payment app.
```json
{
  "payment": {
    "project": "project-slug",
    "order_id": "INV-100020",
    "amount": 50000,
    "fee": 1500,
    "total_payment": 51500,
    "payment_method": "shopeepay",
    "payment_number": "https://sandbox.yopedia.test/topup/direct?ref=SA26...",
    "va_number": "",
    "payment_url": "https://sandbox.yopedia.test/topup/direct?ref=SA26...",
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

When a transaction is successfully completed, YoPedia sends a secure HTTP `POST` webhook to your configured `callback_url`:

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
