# Rust ile Hello World

Bu dosya, Rust’ta temel bir **Hello World** örneğini, **mutable/immutable** değişken mantığını ve **tuple** kullanımını gösterir.
---

## 1) Hello World

Rust’ta çalıştırılabilir bir programın giriş noktası `fn main()` fonksiyonudur. `println!` bir *macro*’dur ve konsola yazdırma yapar.

```rust
fn main() {
    println!("System Programming with Rust.");
}
```

---

## 2) Mutable / Immutable

Rust’ta değişkenler **varsayılan olarak immutable** (değiştirilemez) gelir.
Bir değişkeni sonradan değiştirmek istiyorsanız `mut` kullanmanız gerekir.

### Immutable örneği

Bu örnekte `points` immutable olduğu için sonradan değer ataması yapılamaz.

```rust
fn main() {
    let points = 90; // Immutable
    println!("Points: {}", points);

    // points = 80; // HATA: cannot assign twice to immutable variable
}
```

### Mutable örneği

`mut` ile tanımlanan değişkenin değeri güncellenebilir.

```rust
fn main() {
    let mut points = 90; // Mutable
    println!("Points: {}", points);

    points = 80;
    println!("Points: {}", points);

    points = 60;
    println!("Points: {}", points);
}
```

> Not: Aynı isimle tekrar `let points = ...;` yazmak **shadowing** (gölgeleme) yapar. Bu, mevcut değişkeni güncellemek değil, yeni bir değişken oluşturmak demektir.

---

## 3) Tuple Kullanımı

Tuple, farklı tipleri tek bir değişkende tutmaya yarar. Öğelere:

- indeksle (`config.0` gibi) erişilebilir,
- ya da tuple parçalama (destructuring) ile tek seferde ayrıştırılabilir.

```rust
fn main() {
    println!("Hello, world!");

    // (width, height, host, debug)
    let config = (680, 420, String::from("localhost"), false);
    println!("Config: {:?}", config);

    // İndeks ile erişim
    let width = config.0;
    let height = config.1;
    let host = &config.2; // String'i taşımamak için referans
    let debug = config.3;

    println!("Width: {}", width);
    println!("Height: {}", height);
    println!("Host: {}", host);
    println!("Debug mode: {}", debug);

    // Destructuring (parçalama)
    let (w, h, _, _) = config;
    println!("Screen resolution: {}x{}", w, h);
}
```

### `String` ve ownership notu

`String` heap üzerinde tutulur. `config.2` ile doğrudan bir değişkene atamak sahipliği (ownership) taşıyabilir.
Sadece okumak istiyorsanız `&config.2` gibi referans kullanmak daha uygundur.

---
