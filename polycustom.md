# Polymorphism Analysis - E-Commerce System

## Polymorphism yang Sudah Diterapkan di Code

### 1. Method Overloading (Parameter Berbeda)

| Class | Method | Parameter Variants | Behavior |
|-------|--------|-------------------|----------|
| **Product** | `getPriceBySize()` | `'small'` | Return `price_small` |
| | | `'medium'` | Return `price_medium` |
| | | `'large'` | Return `price_large` |
| | | `null` atau default | Return `base_price` |
| **Product** | `getStockBySize()` | `'small'` | Return `stock_small` |
| | | `'medium'` | Return `stock_medium` |
| | | `'large'` | Return `stock_large` |
| | | `null` atau default | Return `stock` |

---

### 2. Accessor Polymorphism (Attribute Casting)

| Model | Accessor Method | Return Type | Behavior |
|-------|----------------|-------------|----------|
| **Order** | `getStatusLabelAttribute()` | `string` | Konversi status code ke label Indonesia |
| **Order** | `getPaymentMethodLabelAttribute()` | `string` | Konversi payment method ke label Indonesia |
| **UserAddress** | `getFullAddressAttribute()` | `string` | Gabungkan semua field alamat menjadi satu string |
| **Product** | `hasSizeVariants()` | `boolean` | Cek apakah produk punya varian ukuran |
| **Product** | `hasFlavorVariants()` | `boolean` | Cek apakah produk punya varian rasa |

### 3. Relational Polymorphism (Eloquent Relationships)

| Model | Relationship Type | Target Model | Method | Behavior |
|-------|------------------|--------------|--------|----------|
| **User** | `hasMany` | UserAddress | `addresses()` | User memiliki banyak alamat |
| **User** | `hasMany` | Order | `orders()` | User memiliki banyak pesanan |
| **User** | `hasMany` | CartItem | `cartItems()` | User memiliki banyak item di keranjang |
| **Product** | `hasMany` | OrderItem | `orderItems()` | Product ada di banyak order |
| **Product** | `hasMany` | CartItem | `cartItems()` | Product ada di banyak keranjang |
| **Order** | `belongsTo` | User | `user()` | Order dimiliki oleh satu user |
| **Order** | `hasMany` | OrderItem | `items()` | Order memiliki banyak item |
| **CartItem** | `belongsTo` | User | `user()` | CartItem dimiliki oleh satu user |
| **CartItem** | `belongsTo` | Product | `product()` | CartItem merujuk ke satu product |
| **UserAddress** | `belongsTo` | User | `user()` | Address dimiliki oleh satu user |

### 4. Trait Polymorphism (Reusable Behaviors)

| Trait | Implemented By | Method | Behavior |
|-------|----------------|--------|----------|
| **HasApiTokens** | User | `createToken()`, `tokens()` | Manajemen API token untuk autentikasi |
| **Notifiable** | User | `notify()`, `notifications()` | Sistem notifikasi Laravel |
| **HasFactory** | Semua Model | `factory()` | Generate data dummy untuk testing |

---

## Polymorphism yang Bisa Ditambahkan

### 1. Interface Activatable

**Tujuan:** Standardisasi method untuk entitas yang bisa diaktifkan/nonaktifkan

| Method | User | Product | Flyer | Behavior |
|--------|------|---------|-------|----------|
| `activate()` | Set `is_active = true` | Set `is_active = true` | Set `is_active = true` | Aktifkan entitas |
| `deactivate()` | Set `is_active = false` | Set `is_active = false` | Set `is_active = false` | Nonaktifkan entitas |
| `isActive()` | Check `is_active` | Check `is_active` | Check `is_active` + validasi tanggal | Cek status aktif |

**Implementasi:**
```php
interface Activatable {
    public function activate(): bool;
    public function deactivate(): bool;
    public function isActive(): bool;
}
```

---

### 2. Interface Priceable

**Tujuan:** Standardisasi method untuk entitas yang memiliki harga

| Method | Product | CartItem | OrderItem | Behavior |
|--------|---------|----------|-----------|----------|
| `getPrice()` | Return harga berdasarkan size | Return harga dari product | Return snapshot price | Ambil harga |
| `calculateSubtotal()` | `price * 1` | `price * quantity` | `price * quantity` | Hitung subtotal |
| `getPriceAttribute()` | Accessor untuk harga dinamis | Accessor dari relasi | Accessor dari field | Format harga |

**Implementasi:**
```php
interface Priceable {
    public function getPrice(): int;
    public function calculateSubtotal(int $quantity = 1): int;
}
```

---

### 3. Interface Displayable

**Tujuan:** Standardisasi method untuk display data

| Method | User | Product | Order | Flyer | Behavior |
|--------|------|---------|-------|-------|----------|
| `getDisplayName()` | Return `user_name` | Return `product_name` | Return `order_number` | Return `flyer_title` | Nama untuk display |
| `getIdentifier()` | Return `user_id` | Return `product_id` | Return `order_id` | Return `flyer_id` | Primary key |
| `toSearchableArray()` | Array untuk search | Array untuk search | Array untuk search | Array untuk search | Data untuk indexing |

**Implementasi:**
```php
interface Displayable {
    public function getDisplayName(): string;
    public function getIdentifier(): int;
    public function toSearchableArray(): array;
}
```

---

### 4. Trait CustomTimestamps

**Tujuan:** Handle custom timestamp columns

| Model | Created At Column | Updated At Column | Behavior |
|-------|------------------|-------------------|----------|
| **Order** | `order_date` | `null` | Hanya track tanggal pembuatan |
| **User** | `null` | `null` | Tidak pakai timestamp |
| **Product** | `null` | `null` | Tidak pakai timestamp |

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

---

## Summary Polymorphism Implementation

### ✅ Yang Sudah Ada di Code

| No | Jenis Polymorphism | Lokasi | Contoh | Kategori |
|----|-------------------|--------|--------|----------|
| 1 | **Method Overloading** | Product | `getPriceBySize($size)`, `getStockBySize($size)` | Implementasi |
| 2 | **Accessor Polymorphism** | Order, UserAddress, Product | `getStatusLabelAttribute()`, `getFullAddressAttribute()` | Implementasi |
| 3 | **Relational Polymorphism** | Semua Model | `hasMany()`, `belongsTo()` | Hubungan |
| 4 | **Trait Polymorphism** | User, Semua Model | `HasApiTokens`, `Notifiable`, `HasFactory` | Method Overriding |

### 🔵 Yang Bisa Ditambahkan

| No | Jenis Polymorphism | Target Models | Benefit | Kategori |
|----|-------------------|---------------|---------|----------|
| 1 | **Interface Activatable** | User, Product, Flyer | Standardisasi aktivasi/deaktivasi | Implementasi |
| 2 | **Interface Priceable** | Product, CartItem, OrderItem | Standardisasi perhitungan harga | Implementasi |
| 3 | **Interface Displayable** | Semua Model | Standardisasi method display | Implementasi |
| 4 | **Trait CustomTimestamps** | Order, Model lain | Flexible timestamp handling | Method Overriding |

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
