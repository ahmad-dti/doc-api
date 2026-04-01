# 📘 Dishub Integrated Ticketing API Documentation

---

# 📑 Table of Contents

* [Overview](#overview)
* [Base URL](#base-url)
* [Ticket API](#ticket-api)
* [Token API](#token-api)
* [Validation API](#validation-api)
* [Journey API](#journey-api)
* [Settlement API](#settlement-api)
* [Fraud Detection API](#fraud-detection-api)
* [Business Rules](#business-rules)
* [Error Response](#error-response)

---

# 📌 Overview

API ini digunakan untuk sistem tiket terusan lintas kota berbasis:

* time-based validity (3 hari + 3 jam)
* city-based usage (1 kota = 1 perjalanan)
* centralized validation oleh Dishub
* payment diproses oleh Payment Gateway

---

# 🌐 Base URL

```
https://api.example.com
```

---

# 🎟️ Ticket API

## Create Ticket

**POST** `/tickets`

### Request

```json
{
  "channel": "mobile_app",
  "user_id": "user-001"
}
```

### Response

```json
{
  "ticket_id": "TCK123456",
  "price": 15000,
  "status": "CREATED",
  "payment_expiry_seconds": 900
}
```

---

## Confirm Payment

**POST** `/tickets/{ticket_id}/pay`

### Request

```json
{
  "payment_method": "GOPAY",
  "payment_ref": "PAY123456",
  "amount": 15000,
  "paid_at": "2026-03-31T10:00:00Z"
}
```

### Response

```json
{
  "ticket_id": "TCK123456",
  "status": "PAID",
  "activated": false,
  "inactive_expiry": "2026-04-03T10:00:00Z"
}
```

---

## Get Ticket Detail

**GET** `/tickets/{ticket_id}`

### Response

```json
{
  "ticket_id": "TCK123456",
  "status": "ACTIVE",
  "activated": true,
  "start_time": "2026-03-31T11:00:00Z",
  "expire_time": "2026-03-31T14:00:00Z",
  "inactive_expiry": "2026-04-03T10:00:00Z",
  "city_usage": {
    "A": {
      "tap_in": true,
      "tap_out": true
    },
    "B": {
      "tap_in": true,
      "tap_out": false
    }
  }
}
```

---

# 🔐 Token API

## Generate QR Token

**GET** `/tickets/{ticket_id}/token`

### Response

```json
{
  "token": "abc123xyz456",
  "expires_in_seconds": 30
}
```

---

# 🚪 Validation API

## Validate Tap

**POST** `/validate`

### Request

```json
{
  "token": "abc123xyz456",
  "gate_id": "GATE_A_01",
  "city": "A",
  "mode": "BUS",
  "tap_type": "IN",
  "timestamp": "2026-03-31T11:00:00Z"
}
```

### Response (Success)

```json
{
  "status": "VALID",
  "action": "OPEN_GATE",
  "ticket_status": "ACTIVE",
  "activated": true,
  "remaining_time_seconds": 7200,
  "message": "Access granted"
}
```

### Response (Rejected)

```json
{
  "status": "REJECTED",
  "reason": "INVALID_TOKEN",
  "message": "Access denied"
}
```

---

# 📊 Journey API

## Save Journey

**POST** `/journeys`

### Request

```json
{
  "ticket_id": "TCK123456",
  "city": "A",
  "tap_type": "IN",
  "gate_id": "GATE_A_01",
  "timestamp": "2026-03-31T11:00:00Z"
}
```

### Response

```json
{
  "status": "RECORDED"
}
```

---

# 💰 Settlement API

## Calculate Settlement

**POST** `/settlement/calculate`

### Request

```json
{
  "ticket_id": "TCK123456"
}
```

### Response

```json
{
  "ticket_id": "TCK123456",
  "total_amount": 15000,
  "split": [
    {
      "operator": "TRANS_A",
      "amount": 7500
    },
    {
      "operator": "TRANS_B",
      "amount": 7500
    }
  ]
}
```

---

## Distribute Settlement

**POST** `/settlement/distribute`

### Request

```json
{
  "ticket_id": "TCK123456",
  "settlement_ref": "SETT123456"
}
```

### Response

```json
{
  "status": "SUCCESS",
  "distributed_at": "2026-03-31T18:00:00Z"
}
```

---

# 🛡️ Fraud Detection API

## Fraud Check

**POST** `/fraud/check`

### Request

```json
{
  "ticket_id": "TCK123456",
  "event": "SUSPICIOUS_TAP",
  "timestamp": "2026-03-31T11:05:00Z"
}
```

### Response

```json
{
  "risk_level": "LOW",
  "action": "ALLOW"
}
```

---

# 🧠 Business Rules

* Tiket berlaku maksimal **3 hari sebelum digunakan**
* Tiket aktif saat **tap-in pertama**
* Masa aktif setelah digunakan: **3 jam**
* Setiap kota hanya boleh:

  * 1x tap-in
  * 1x tap-out

---

# ⚠️ Error Response

```json
{
  "status": "ERROR",
  "code": "INVALID_REQUEST",
  "message": "Invalid payload",
  "request_id": "REQ123456"
}
```

---

# 💡 Notes

* Semua validasi dilakukan oleh sistem Dishub
* Gate hanya berfungsi sebagai trigger
* Payment Gateway menangani seluruh transaksi dan distribusi dana
