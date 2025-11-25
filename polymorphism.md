# Polymorphism Analysis - E-Commerce System

## Inheritance vs Polymorphism

| Kategori | Inheritance | Polymorphism |
|----------|-------------|--------------|
| **Definisi** | Mekanisme dimana suatu kelas mewarisi properti dan perilaku dari kelas lain (superkelas). | Kemampuan objek dari kelas yang berbeda untuk merespons pemanggilan metode yang sama secara berbeda. |
| **Tujuan** | Penggunaan kembali kode dan membangun hubungan hierarki antar kelas | Fleksibilitas dan kemampuan untuk memperlakukan objek secara bergantian. |
| **Hubungan** | Membangun hubungan antar kelas. Dicapai melalui warisan tetapi tidak terbatas pada itu. | Objek bisa bersifat polimorfik di dalam kelasnya sendiri atau kelas leluhurnya. |
| **Konsep** | Konsep waktu kompilasi. Hubungan ditentukan selama kompilasi. Konsep waktu proses. | Perilaku sebenarnya ditentukan berdasarkan objek spesifik yang direferensikan. |
| **Implementasi** | Kelas baru (subkelas) akan mewarisi properti dan perilaku dari kelas dasar (superkelas). | Dicapai melalui penggantian metode dan kelebihan metode. |
| **Method Overriding** | Mengizinkan subkelas menyediakan implementasi berbeda dari metode yang diwarisi dari superkelas. | Subkelas yang berbeda dapat memiliki implementasi unik dari metode yang sama. |
| **Method Overloading** | Dapat digunakan dalam kelas atau hierarki yang sama. | Dapat digunakan untuk menyediakan beberapa metode dengan nama yang sama tetapi parameternya berbeda. |
| **Use Cases** | Memodelkan hierarki dunia nyata dan reuse kode. | Berurusan dengan kumpulan objek, merancang kode fleksibel, dan meningkatkan modularitas dan ekstensibilitas. |

---

## Polymorphism yang Sudah Diterapkan

### 1. Inheritance (Pewarisan)

| Model/Class | Parent Class | Method yang Diwarisi | Method yang Di-override |
|-------------|--------------|---------------------|------------------------|
| **User** | `Authenticatable` | `getAuthIdentifier()`, `getAuthPassword()`, `getRememberToken()` | Custom: `isAdmin()`, `isSuperAdmin()`, `getDefaultAddress()` |
| **Product** | `Model` | `save()`, `delete()`, `find()`, `all()` | Custom: `getPriceBySize()`, `getStockBySize()`, `hasSizeVariants()` |
| **Order** | `Model` | `save()`, `delete()`, `find()`, `all()` | Custom: `getStatusLabelAttribute()`, `getPaymentMethodLabelAttribute()` |
| **CartItem** | `Model` | `save()`, `delete()`, `find()`, `all()` | Custom: `getSubtotal()` |
| **UserAddress** | `Model` | `save()`, `delete()`, `find()`, `all()` | Custom: `getFullAddressAttribute()` |
| **Flyer** | `Model` | `save()`, `delete()`, `find()`, `all()` | Custom: `isActive()` |

---

### 2. Method Overloading (Parameter Berbeda)

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

### 3. Interface Polymorphism (Implicit via Eloquent)

| Interface/Trait | Implemented By | Method | Behavior |
|-----------------|----------------|--------|----------|
| **HasMany** | User, Product, Order | `hasMany()` | Relasi one-to-many dengan model lain |
| **BelongsTo** | CartItem, UserAddress, Order | `belongsTo()` | Relasi many-to-one dengan model parent |
| **HasApiTokens** | User | `createToken()`, `tokens()` | Manajemen API token untuk autentikasi |
| **Notifiable** | User | `notify()`, `notifications()` | Sistem notifikasi Laravel |

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

### 3. Abstract Class BaseModel

**Tujuan:** Standardisasi method umum untuk semua model

| Method | User | Product | Order | Flyer | Behavior |
|--------|------|---------|-------|-------|----------|
| `getDisplayName()` | Return `user_name` | Return `product_name` | Return `order_number` | Return `flyer_title` | Nama untuk display |
| `getIdentifier()` | Return `user_id` | Return `product_id` | Return `order_id` | Return `flyer_id` | Primary key |
| `toSearchableArray()` | Array untuk search | Array untuk search | Array untuk search | Array untuk search | Data untuk indexing |

**Implementasi:**
```php
abstract class BaseModel extends Model {
    abstract public function getDisplayName(): string;
    abstract public function getIdentifier(): int;
    abstract public function toSearchableArray(): array;
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

| No | Jenis Polymorphism | Lokasi | Contoh |
|----|-------------------|--------|--------|
| 1 | **Inheritance** | Semua Model | `User extends Authenticatable`, `Product extends Model` |
| 2 | **Method Overloading** | Product | `getPriceBySize($size)`, `getStockBySize($size)` |
| 3 | **Interface (Implicit)** | Eloquent Relations | `hasMany()`, `belongsTo()`, `HasApiTokens` |
| 4 | **Accessor/Mutator** | Order, UserAddress | `getStatusLabelAttribute()`, `getFullAddressAttribute()` |

### 🔵 Yang Bisa Ditambahkan

| No | Jenis Polymorphism | Target Models | Benefit |
|----|-------------------|---------------|---------|
| 1 | **Interface Activatable** | User, Product, Flyer | Standardisasi aktivasi/deaktivasi |
| 2 | **Interface Priceable** | Product, CartItem, OrderItem | Standardisasi perhitungan harga |
| 3 | **Abstract BaseModel** | Semua Model | Standardisasi method umum |
| 4 | **Trait CustomTimestamps** | Order, Model lain | Flexible timestamp handling |

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

**Dengan Polymorphism (Abstract BaseModel):**
```php
$name = $model->getDisplayName(); // Works for all models
```

---

## Kesimpulan

1. **Inheritance** sudah diterapkan dengan baik melalui Eloquent Model
2. **Method Overloading** sudah ada di Product untuk handle size variants
3. **Interface Polymorphism** bisa ditambahkan untuk standardisasi behavior
4. **Abstract Class** bisa digunakan untuk method umum semua model
5. **Trait** berguna untuk reusable functionality seperti timestamps

Polymorphism meningkatkan:
- **Maintainability** - Code lebih mudah di-maintain
- **Flexibility** - Mudah menambah model baru
- **Reusability** - Mengurangi duplikasi code
- **Testability** - Lebih mudah untuk testing
