# Database Schema - RH Store

## Overview
Database untuk aplikasi toko roti "Rendah Hati Store" dengan 7 tabel utama.

---

## 1. users
Tabel untuk menyimpan data pengguna (customer, admin, super_admin)

| Column | Type | Length | Nullable | Default | Description |
|--------|------|--------|----------|---------|-------------|
| **user_id** | INT | - | NO | AUTO_INCREMENT | Primary Key |
| user_name | VARCHAR | 100 | NO | - | Username (unique) |
| email | VARCHAR | 100 | NO | - | Email (unique) |
| password | VARCHAR | 255 | NO | - | Password (hashed) |
| phone | VARCHAR | 20 | YES | NULL | Nomor telepon |
| role | ENUM | - | NO | 'customer' | customer, admin, super_admin |
| is_active | BOOLEAN | - | NO | TRUE | Status aktif user |

**Indexes:**
- PRIMARY KEY: user_id
- UNIQUE: user_name
- UNIQUE: email

**Notes:**
- Tidak ada timestamps (created_at, updated_at)
- Password harus di-hash menggunakan bcrypt
- Role default adalah 'customer'

---

## 2. addresses
Tabel untuk menyimpan alamat pengiriman user

| Column | Type | Length | Nullable | Default | Description |
|--------|------|--------|----------|---------|-------------|
| **address_id** | INT | - | NO | AUTO_INCREMENT | Primary Key |
| user_id | INT | - | NO | - | Foreign Key ke users |
| address_label | VARCHAR | 50 | NO | - | Label alamat (Rumah, Kantor) |
| recipient_name | VARCHAR | 100 | NO | - | Nama penerima |
| phone | VARCHAR | 20 | NO | - | Nomor telepon penerima |
| street_address | TEXT | - | NO | - | Alamat lengkap |
| city | VARCHAR | 50 | NO | - | Kota |
| province | VARCHAR | 50 | NO | - | Provinsi |
| postal_code | VARCHAR | 10 | NO | - | Kode pos |
| is_default | BOOLEAN | - | NO | FALSE | Alamat default |

**Indexes:**
- PRIMARY KEY: address_id
- FOREIGN KEY: user_id → users(user_id) ON DELETE CASCADE

**Notes:**
- Satu user bisa memiliki banyak alamat
- Hanya satu alamat yang bisa menjadi default per user

---

## 3. products
Tabel untuk menyimpan data produk dengan varian ukuran dan rasa

| Column | Type | Length | Nullable | Default | Description |
|--------|------|--------|----------|---------|-------------|
| **product_id** | INT | - | NO | AUTO_INCREMENT | Primary Key |
| product_name | VARCHAR | 100 | NO | - | Nama produk |
| category | VARCHAR | 50 | NO | - | Kategori produk |
| description | TEXT | - | YES | NULL | Deskripsi produk |
| base_price | INT | - | NO | - | Harga dasar (Rupiah) |
| stock | INT | - | NO | 0 | Stok dasar |
| image_url | VARCHAR | 255 | YES | NULL | URL gambar produk |
| is_active | BOOLEAN | - | NO | TRUE | Status aktif produk |
| **size_small** | VARCHAR | 20 | YES | NULL | Nama ukuran kecil |
| **price_small** | INT | - | YES | NULL | Harga ukuran kecil |
| **stock_small** | INT | - | YES | NULL | Stok ukuran kecil |
| **size_medium** | VARCHAR | 20 | YES | NULL | Nama ukuran sedang |
| **price_medium** | INT | - | YES | NULL | Harga ukuran sedang |
| **stock_medium** | INT | - | YES | NULL | Stok ukuran sedang |
| **size_large** | VARCHAR | 20 | YES | NULL | Nama ukuran besar |
| **price_large** | INT | - | YES | NULL | Harga ukuran besar |
| **stock_large** | INT | - | YES | NULL | Stok ukuran besar |
| **available_variants** | TEXT | - | YES | NULL | JSON array varian rasa |

**Indexes:**
- PRIMARY KEY: product_id

**Notes:**
- Varian ukuran (size) mempengaruhi harga
- Varian rasa (available_variants) tidak mempengaruhi harga
- available_variants disimpan sebagai JSON: `["Coklat", "Keju", "Original"]`
- Jika tidak ada varian ukuran, gunakan base_price dan stock

**Contoh Data:**
```json
{
  "product_id": 1,
  "product_name": "Roti Tumpuk",
  "category": "Aneka Roti & Sayur",
  "base_price": 15000,
  "stock": 50,
  "size_small": "Kecil",
  "price_small": 12000,
  "stock_small": 20,
  "size_medium": "Sedang",
  "price_medium": 15000,
  "stock_medium": 20,
  "size_large": "Besar",
  "price_large": 20000,
  "stock_large": 10,
  "available_variants": ["Coklat", "Keju", "Original", "Strawberry"]
}
```

---

## 4. flyers
Tabel untuk menyimpan data flyer/promo

| Column | Type | Length | Nullable | Default | Description |
|--------|------|--------|----------|---------|-------------|
| **flyer_id** | INT | - | NO | AUTO_INCREMENT | Primary Key |
| flyer_title | VARCHAR | 100 | NO | - | Judul flyer |
| flyer_description | TEXT | - | YES | NULL | Deskripsi flyer |
| flyer_image | VARCHAR | 255 | NO | - | URL gambar flyer |
| valid_from | DATE | - | NO | - | Tanggal mulai berlaku |
| valid_until | DATE | - | NO | - | Tanggal berakhir |
| is_active | BOOLEAN | - | NO | TRUE | Status aktif flyer |

**Indexes:**
- PRIMARY KEY: flyer_id

**Notes:**
- Flyer hanya ditampilkan jika is_active = TRUE dan tanggal saat ini berada di antara valid_from dan valid_until

---

## 5. cart_items
Tabel untuk menyimpan item di keranjang belanja

| Column | Type | Length | Nullable | Default | Description |
|--------|------|--------|----------|---------|-------------|
| **cart_id** | INT | - | NO | AUTO_INCREMENT | Primary Key |
| user_id | INT | - | NO | - | Foreign Key ke users |
| product_id | INT | - | NO | - | Foreign Key ke products |
| quantity | INT | - | NO | 1 | Jumlah item |
| size | VARCHAR | 20 | YES | NULL | Ukuran yang dipilih |
| variant | VARCHAR | 50 | YES | NULL | Varian rasa yang dipilih |

**Indexes:**
- PRIMARY KEY: cart_id
- FOREIGN KEY: user_id → users(user_id) ON DELETE CASCADE
- FOREIGN KEY: product_id → products(product_id) ON DELETE CASCADE

**Notes:**
- Satu user bisa memiliki banyak item di cart
- Jika size NULL, gunakan base_price dari products
- variant hanya untuk informasi, tidak mempengaruhi harga

---

## 6. orders
Tabel untuk menyimpan data pesanan

| Column | Type | Length | Nullable | Default | Description |
|--------|------|--------|----------|---------|-------------|
| **order_id** | INT | - | NO | AUTO_INCREMENT | Primary Key |
| user_id | INT | - | NO | - | Foreign Key ke users |
| order_number | VARCHAR | 50 | NO | - | Nomor pesanan (unique) |
| total_amount | INT | - | NO | - | Total harga (Rupiah) |
| status | ENUM | - | NO | 'pending' | pending, processing, shipped, completed, cancelled |
| payment_method | ENUM | - | NO | 'cod' | cod, transfer, ewallet |
| payment_status | ENUM | - | NO | 'unpaid' | unpaid, paid, refunded |
| shipping_name | VARCHAR | 100 | NO | - | Nama penerima |
| shipping_phone | VARCHAR | 20 | NO | - | Telepon penerima |
| shipping_address | TEXT | - | NO | - | Alamat pengiriman |
| shipping_city | VARCHAR | 50 | NO | - | Kota pengiriman |
| shipping_province | VARCHAR | 50 | NO | - | Provinsi pengiriman |
| shipping_postal | VARCHAR | 10 | NO | - | Kode pos pengiriman |
| notes | TEXT | - | YES | NULL | Catatan pesanan |
| order_date | TIMESTAMP | - | NO | CURRENT_TIMESTAMP | Tanggal pesanan |

**Indexes:**
- PRIMARY KEY: order_id
- UNIQUE: order_number
- FOREIGN KEY: user_id → users(user_id) ON DELETE CASCADE

**Notes:**
- order_number format: ORD-YYYYMMDD-XXXXX
- Alamat pengiriman disimpan langsung di order (tidak reference ke addresses)
- order_date adalah satu-satunya timestamp di tabel ini

---

## 7. order_items
Tabel untuk menyimpan detail item pesanan

| Column | Type | Length | Nullable | Default | Description |
|--------|------|--------|----------|---------|-------------|
| **order_item_id** | INT | - | NO | AUTO_INCREMENT | Primary Key |
| order_id | INT | - | NO | - | Foreign Key ke orders |
| product_id | INT | - | NO | - | Foreign Key ke products |
| product_name | VARCHAR | 100 | NO | - | Nama produk (snapshot) |
| size | VARCHAR | 20 | YES | NULL | Ukuran yang dipesan |
| variant | VARCHAR | 50 | YES | NULL | Varian rasa yang dipesan |
| quantity | INT | - | NO | - | Jumlah item |
| price | INT | - | NO | - | Harga per item (snapshot) |
| subtotal | INT | - | NO | - | Total harga (price × quantity) |

**Indexes:**
- PRIMARY KEY: order_item_id
- FOREIGN KEY: order_id → orders(order_id) ON DELETE CASCADE
- FOREIGN KEY: product_id → products(product_id) ON DELETE CASCADE

**Notes:**
- product_name dan price disimpan sebagai snapshot (tidak berubah jika produk diupdate)
- subtotal = price × quantity

---

## Relationships

```
users (1) ──────< (N) addresses
users (1) ──────< (N) cart_items
users (1) ──────< (N) orders

products (1) ───< (N) cart_items
products (1) ───< (N) order_items

orders (1) ─────< (N) order_items
```

---

## Data Types Summary

- **INT**: Integer untuk ID dan harga (dalam Rupiah, tanpa desimal)
- **VARCHAR**: String dengan panjang maksimal
- **TEXT**: String panjang tanpa batas
- **BOOLEAN**: TRUE/FALSE (disimpan sebagai TINYINT(1))
- **ENUM**: Pilihan terbatas
- **DATE**: Format YYYY-MM-DD
- **TIMESTAMP**: Format YYYY-MM-DD HH:MM:SS

---

## Validation Rules

### Input Validation (Regex)
- **product_name, user_name**: `^[A-Za-z0-9\s]+$` (huruf, angka, spasi)
- **category**: `^[A-Za-z0-9\s&]+$` (huruf, angka, spasi, &)
- **size**: `^[A-Za-z0-9\s]+$` (huruf, angka, spasi)
- **available_variants**: `^[A-Za-z0-9\s,]+$` (huruf, angka, spasi, koma)

### Not Allowed
- Simbol: `!@#$%^*()_+-=[]{}|;:'",.<>?/\`
- Emoji
- Special characters

---

## Migration Files

1. `2024_01_01_000001_create_users_table.php`
2. `2024_01_01_000002_create_addresses_table.php`
3. `2024_01_01_000003_create_products_table.php`
4. `2024_01_01_000004_create_flyers_table.php`
5. `2024_01_01_000005_create_cart_items_table.php`
6. `2024_01_01_000006_create_orders_table.php`
7. `2024_01_01_000007_create_order_items_table.php`

---

## Sample Queries

### Get Product with Variants
```sql
SELECT * FROM products WHERE product_id = 1;
```

### Get User's Cart
```sql
SELECT c.*, p.product_name, p.image_url 
FROM cart_items c
JOIN products p ON c.product_id = p.product_id
WHERE c.user_id = 1;
```

### Get Order with Items
```sql
SELECT o.*, oi.product_name, oi.quantity, oi.price, oi.subtotal
FROM orders o
JOIN order_items oi ON o.order_id = oi.order_id
WHERE o.order_id = 1;
```

### Get Active Flyers
```sql
SELECT * FROM flyers 
WHERE is_active = TRUE 
AND valid_from <= CURDATE() 
AND valid_until >= CURDATE();
```

---

## Notes

- Semua tabel menggunakan **integer** sebagai primary key (bukan bigInteger)
- Tidak ada **created_at** dan **updated_at** di semua tabel (kecuali order_date di orders)
- Semua harga disimpan dalam **Rupiah** (integer, tanpa desimal)
- **Varian ukuran** mempengaruhi harga, **varian rasa** tidak
- Foreign key menggunakan **ON DELETE CASCADE**
