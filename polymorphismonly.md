# Polymorphism Analysis - E-Commerce System

## Kategori Polymorphism

| Kategori | Polymorphism |
|----------|--------------|
| **Definisi** | Kemampuan objek dari kelas yang berbeda untuk merespons pemanggilan metode yang sama secara berbeda. |
| **Tujuan** | Fleksibilitas dan kemampuan untuk memperlakukan objek secara bergantian. |
| **Hubungan** | Objek bisa bersifat polimorfik di dalam kelasnya sendiri atau kelas leluhurnya. |
| **Konsep** | Perilaku sebenarnya ditentukan berdasarkan objek spesifik yang direferensikan. |
| **Implementasi** | Dicapai melalui penggantian metode dan kelebihan metode. |
| **Method Overriding** | Subkelas yang berbeda dapat memiliki implementasi unik dari metode yang sama. |
| **Method Overloading** | Dapat digunakan untuk menyediakan beberapa metode dengan nama yang sama tetapi parameternya berbeda. |
| **Use Cases** | Berurusan dengan kumpulan objek, merancang kode fleksibel, dan meningkatkan modularitas dan ekstensibilitas. |

---

## Polymorphism yang Sudah Diterapkan di Code

### 1. Method Overloading (Parameter Berbeda)

| Class | Method | Parameter Variants | Behavior | Kategori |
|-------|--------|-------------------|----------|----------|
| **Product** | `getPriceBySize()` | `'small'` | Return `price_small` | Method Overloading |
| | | `'medium'` | Return `price_medium` | Method Overloading |
| | | `'large'` | Return `price_large` | Method Overloading |
| | | `null` atau default | Return `base_price` | Method Overloading |
| **Product** | `getStockBySize()` | `'small'` | Return `stock_small` | Method Overloading |
| | | `'medium'` | Return `stock_medium` | Method Overloading |
| | | `'large'` | Return `stock_large` | Method Overloading |
| | | `null` atau default | Return `stock` | Method Overloading |

**Konsep:** Perilaku sebenarnya ditentukan berdasarkan objek spesifik yang direferensikan (parameter size).

---

### 2. Accessor Polymorphism (Attribute Casting)

| Model | Accessor Method | Return Type | Behavior | Kategori |
|-------|----------------|-------------|----------|----------|
| **Order** | `getStatusLabelAttribute()` | `string` | Konversi status code ke label Indonesia | Implementasi |
| **Order** | `getPaymentMethodLabelAttribute()` | `string` | Konversi payment method ke label Indonesia | Implementasi |
| **UserAddress** | `getFullAddressAttribute()` | `string` | Gabungkan semua field alamat menjadi satu string | Implementasi |
| **Product** | `hasSizeVariants()` | `boolean` | Cek apakah produk punya varian ukuran | Implementasi |
| **Product** | `hasFlavorVariants()` | `boolean` | Cek apakah produk punya varian rasa | Implementasi |

**Konsep:** Dicapai melalui penggantian metode - setiap model memiliki implementasi berbeda untuk accessor.

---

### 3. Relational Polymorphism (Eloquent Relationships)

| Model | Relationship Type | Target Model | Method | Behavior | Kategori |
|-------|------------------|--------------|--------|----------|----------|
| **User** | `hasMany` | UserAddress | `addresses()` | User memiliki banyak alamat | Hubungan |
| **User** | `hasMany` | Order | `orders()` | User memiliki banyak pesanan | Hubungan |
| **User** | `hasMany` | CartItem | `cartItems()` | User memiliki banyak item di keranjang | Hubungan |
| **Product** | `hasMany` | OrderItem | `orderItems()` | Product ada di banyak order | Hubungan |
| **Product** | `hasMany` | CartItem | `cartItems()` | Product ada di banyak keranjang | Hubungan |
| **Order** | `belongsTo` | User | `user()` | Order dimiliki oleh satu user | Hubungan |
| **Order** | `hasMany` | OrderItem | `items()` | Order memiliki banyak item | Hubungan |
| **CartItem** | `belongsTo` | User | `user()` | CartItem dimiliki oleh satu user | Hubungan |
| **CartItem** | `belongsTo` | Product | `product()` | CartItem merujuk ke satu product | Hubungan |
| **UserAddress** | `belongsTo` | User | `user()` | Address dimiliki oleh satu user | Hubungan |

**Konsep:** Objek bisa bersifat polimorfik di dalam kelasnya sendiri atau kelas leluhurnya (relasi antar model).

---

### 4. Trait Polymorphism (Reusable Behaviors)

| Trait | Implemented By | Method | Behavior | Kategori |
|-------|----------------|--------|----------|----------|
| **HasApiTokens** | User | `createToken()`, `tokens()` | Manajemen API token untuk autentikasi | Method Overriding |
| **Notifiable** | User | `notify()`, `notifications()` | Sistem notifikasi Laravel | Method Overriding |
| **HasFactory** | Semua Model | `factory()` | Generate data dummy untuk testing | Method Overriding |

**Konsep:** Subkelas yang berbeda dapat memiliki implementasi unik dari metode yang sama (via trait).

---

## Polymorphism yang Bisa Ditambahkan

### 1. Interface Activatable

**Tujuan:** Standardisasi method untuk entitas yang bisa diaktifkan/nonaktifkan

| Method | User | Product | Flyer | Behavior | Kategori |
|--------|------|---------|-------|----------|----------|
| `activate()` | Set `is_active = true` | Set `is_active = true` | Set `is_active = true` | Aktifkan entitas | Implementasi |
| `deactivate()` | Set `is_active = false` | Set `is_active = false` | Set `is_active = false` | Nonaktifkan entitas | Implementasi |
| `isActive()` | Check `is_active` | Check `is_active` | Check `is_active` + validasi tanggal | Cek status aktif | Implementasi |

**Implementasi:**

```php
interface Activatable {
    public function activate(): bool;
    public function deactivate(): bool;
    public function isActive(): bool;
}
```

**Konsep:** Dicapai melalui penggantian metode - setiap model implement interface dengan cara berbeda.

---

### 2. Interface Priceable

**Tujuan:** Standardisasi method untuk entitas yang memiliki harga

| Method | Product | CartItem | OrderItem | Behavior | Kategori |
|--------|---------|----------|-----------|----------|----------|
| `getPrice()` | Return harga berdasarkan size | Return harga dari product | Return snapshot price | Ambil harga | Implementasi |
| `calculateSubtotal()` | `price * 1` | `price * quantity` | `price * quantity` | Hitung subtotal | Implementasi |
| `getPriceAttribute()` | Accessor untuk harga dinamis | Accessor dari relasi | Accessor dari field | Format harga | Implementasi |

**Implementasi:**

```php
interface Priceable {
    public function getPrice(): int;
    public function calculateSubtotal(int $quantity = 1): int;
}
```

**Konsep:** Dicapai melalui penggantian metode dan kelebihan metode - method sama, parameter berbeda.

---

### 3. Interface Displayable

**Tujuan:** Standardisasi method untuk display data

| Method | User | Product | Order | Flyer | Behavior | Kategori |
|--------|------|---------|-------|-------|----------|----------|
| `getDisplayName()` | Return `user_name` | Return `product_name` | Return `order_number` | Return `flyer_title` | Nama untuk display | Implementasi |
| `getIdentifier()` | Return `user_id` | Return `product_id` | Return `order_id` | Return `flyer_id` | Primary key | Implementasi |
| `toSearchableArray()` | Array untuk search | Array untuk search | Array untuk search | Array untuk search | Data untuk indexing | Implementasi |

**Implementasi:**

```php
interface Displayable {
    public function getDisplayName(): string;
    public function getIdentifier(): int;
    public function toSearchableArray(): array;
}
```

**Konsep:** Kemampuan objek dari kelas yang berbeda untuk merespons pemanggilan metode yang sama secara berbeda.

---

### 4. Trait CustomTimestamps

**Tujuan:** Handle custom timestamp columns

| Model | Created At Column | Updated At Column | Behavior | Kategori |
|-------|------------------|-------------------|----------|----------|
| **Order** | `order_date` | `null` | Hanya track tanggal pembuatan | Method Overriding |
| **User** | `null` | `null` | Tidak pakai timestamp | Method Overriding |
| **Product** | `null` | `null` | Tidak pakai timestamp | Method Overriding |

**Implementasi:**

```php
trait CustomTimestamps {
    public function getCreatedAtColumn() {
        return static::CREATED_AT ?? 'created_at';
    }

    public function getUpdatedAtColumn() {
        return static::UPDATED_AT ?? 'updated_at';
    }
}
```

**Konsep:** Subkelas yang berbeda dapat memiliki implementasi unik dari metode yang sama.

---

## Summary Polymorphism Implementation

### ✅ Yang Sudah Ada di Code

| No  | Jenis Polymorphism          | Lokasi                       | Contoh                                                       | Kategori          |
| --- | --------------------------- | ---------------------------- | ------------------------------------------------------------ | ----------------- |
| 1   | **Method Overloading**      | Product                      | `getPriceBySize($size)`, `getStockBySize($size)`             | Method Overloading |
| 2   | **Accessor Polymorphism**   | Order, UserAddress, Product  | `getStatusLabelAttribute()`, `getFullAddressAttribute()`     | Implementasi      |
| 3   | **Relational Polymorphism** | Semua Model                  | `hasMany()`, `belongsTo()`                                   | Hubungan          |
| 4   | **Trait Polymorphism**      | User, Semua Model            | `HasApiTokens`, `Notifiable`, `HasFactory`                   | Method Overriding |

### 🔵 Yang Bisa Ditambahkan

| No  | Jenis Polymorphism          | Target Models                | Benefit                           | Kategori          |
| --- | --------------------------- | ---------------------------- | --------------------------------- | ----------------- |
| 1   | **Interface Activatable**   | User, Product, Flyer         | Standardisasi aktivasi/deaktivasi | Implementasi      |
| 2   | **Interface Priceable**     | Product, CartItem, OrderItem | Standardisasi perhitungan harga   | Implementasi      |
| 3   | **Interface Displayable**   | Semua Model                  | Standardisasi method display      | Implementasi      |
| 4   | **Trait CustomTimestamps**  | Order, Model lain            | Flexible timestamp handling       | Method Overriding |

---

## Contoh Use Case Polymorphism

### Use Case 1: Aktivasi Massal

**Tanpa Polymorphism:**

```php
$user->is_active = true;
$user->save();

$product->is_active = true;
$product->save();

$flyer->is_active = true;
$flyer->save();
```

**Dengan Polymorphism (Interface Activatable):**

```php
foreach ([$user, $product, $flyer] as $entity) {
    $entity->activate(); // Polymorphic call
}
```

**Benefit:** Berurusan dengan kumpulan objek, merancang kode fleksibel, dan meningkatkan modularitas.

---

### Use Case 2: Perhitungan Total Harga

**Tanpa Polymorphism:**

```php
$total = 0;
foreach ($cartItems as $item) {
    $total += $item->product->getPriceBySize($item->size) * $item->quantity;
}
```

**Dengan Polymorphism (Interface Priceable):**

```php
$total = $cartItems->sum(fn($item) => $item->calculateSubtotal());
```

**Benefit:** Fleksibilitas dan kemampuan untuk memperlakukan objek secara bergantian.

---

### Use Case 3: Search/Display Name

**Tanpa Polymorphism:**

```php
if ($model instanceof User) {
    $name = $model->user_name;
} elseif ($model instanceof Product) {
    $name = $model->product_name;
} elseif ($model instanceof Order) {
    $name = $model->order_number;
}
```

**Dengan Polymorphism (Interface Displayable):**

```php
$name = $model->getDisplayName(); // Works for all models
```

**Benefit:** Kemampuan objek dari kelas yang berbeda untuk merespons pemanggilan metode yang sama secara berbeda.

---

## Kesimpulan

### Polymorphism yang Sudah Diterapkan:

1. ✅ **Method Overloading** - Product untuk handle size variants
2. ✅ **Accessor Polymorphism** - Order, UserAddress, Product untuk format data
3. ✅ **Relational Polymorphism** - Semua model dengan Eloquent relationships
4. ✅ **Trait Polymorphism** - HasApiTokens, Notifiable, HasFactory

### Polymorphism yang Bisa Ditambahkan:

1. 🔵 **Interface Activatable** - Standardisasi aktivasi/deaktivasi
2. 🔵 **Interface Priceable** - Standardisasi perhitungan harga
3. 🔵 **Interface Displayable** - Standardisasi method display
4. 🔵 **Trait CustomTimestamps** - Flexible timestamp handling

### Benefit Polymorphism:

- **Maintainability** - Code lebih mudah di-maintain
- **Flexibility** - Mudah menambah model baru
- **Reusability** - Mengurangi duplikasi code
- **Testability** - Lebih mudah untuk testing
- **Consistency** - Behavior konsisten antar model
- **Modularity** - Meningkatkan modularitas dan ekstensibilitas
