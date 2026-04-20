# 🧭 Omanyzer — Tourism & Guide Booking Platform

> A full-featured tourism management platform built with **Laravel 11**, connecting **travelers** with local **guides** across Oman.
> The system includes a mobile-facing REST API and a fully operational admin dashboard.

---

## ✨ Features

### 👤 Traveler (Mobile API)

- OTP-based registration & login via WhatsApp
- Browse events by city & category
- Select guides and book across multiple dates
- pay with cash and visa
- Leave reviews and manage favorites

### 🧑‍🏫 Guide (Mobile API)

- Apply to become a guide (approval flow)
- Manage bookings and availability per date
- Wallet balance & withdrawal requests

### 🛠️ Admin Dashboard

- Full management: Users, Guides, Events, Cities, Categories, Banners
- Booking & withdrawal oversight
- Firebase push notifications (custom & bulk)
- Role-based permissions (Spatie)
- Multi-language support (Arabic / English)
- Real-time statistics & monthly charts

---
## 👤 My Role

Built end-to-end as the **sole backend developer** — architecture, API design, admin dashboard, payment integration, and deployment.

---
## 🏗️ Tech Stack


| Layer       | Technology                                 |
| ----------- | ------------------------------------------ |
| Backend     | Laravel 11 (PHP 8.2)                       |
| Auth        | Laravel Sanctum + OTP (WhatsApp)           |
| Admin UI    | Blade + Yajra DataTables                   |
| Permissions | Spatie Laravel Permission (admin staff)    |
| Media       | Spatie Laravel MediaLibrary                |
| i18n        | Spatie Translatable + Laravel Localization |
| Payments    | AmwalPay Integration                       |
| Push        | Firebase Cloud Messaging (FCM)             |
| Export      | Maatwebsite Excel + MisterSpelik PDF       |
| Monitoring  | Laravel Telescope                          |


---

## 📐 Architecture

```
├── app/
│   ├── Http/Controllers/     # Thin controllers (Admin + API V1)
│   ├── Core/
│   │   ├── Services/         # Business logic layer
│   │   ├── Datatables/       # Yajra DataTable definitions
│   │   ├── Enums/            # Type-safe constants
│   │   ├── Helpers/          # Firebase
│   │   └── Traits/           # InteractWithResponse, HasDatatableTrait
├── routes/
│   ├── admin.php             # Admin dashboard routes
│   └── apis/                 # Versioned API routes (public, user, guide, auth, notification)
└── resources/views/          # Blade templates
```

---

## 🧩 Technical Challenges & Solutions

### 1. Unified User Table for Travelers & Guides

**Challenge:** The platform has two user types — travelers and guides — that share most of their data (name, phone, wallet, bookings). Creating a separate table and guard for each would mean duplicated migrations, duplicated auth logic, and unnecessary complexity.

**Solution:** Both types live in a single `users` table with a dedicated `**role`** column backed by a PHP enum (`traveler` / `guide`). The same `User` model and Sanctum guard serve both apps; controllers and services branch behavior from that enum.

---

### 2. Three-Phase Traveler Booking Flow

**Challenge:** A booking is not a single “submit form” action. The traveler must discover who is available, understand pricing and rules before paying, and only then lock a reservation — without creating half-valid records or race conditions between steps.

**Solution:** The flow has **three simple steps**: (1) list **guides who are free** for the event and dates; (2) **preview** — check the request again and return **price breakdown** for the screen before payment; (3) **save the booking** only when the traveler confirms. The server **checks availability again** on preview and on save, so an outdated pick from an earlier screen cannot become a real booking by mistake.

---

### 3. Multi-Date Guide Availability Conflict Detection

**Challenge:** When a traveler books a guide, they may select multiple dates at once. The system must guarantee that the chosen guide is fully available across *every* selected date — not just some of them. A guide who is booked on even one of those dates must not appear as available.

**Solution:** The availability check works at the database query level by counting how many of the requested dates conflict with the guide's existing bookings. Only guides with zero conflicts across all selected dates are returned as available. This prevents partial-availability issues before they reach the application layer.

**Performance & scalability:** Composite **database indexes** on **guide** and **date** fields keep those lookups cheap as bookings and calendar rows grow—so the feature stays responsive under heavier load instead of degrading into full table scans.

---

## 📄 License

Private commercial project — source shared for **portfolio purposes only**. Not for redistribution or reuse.
