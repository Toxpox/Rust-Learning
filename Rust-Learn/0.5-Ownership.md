# Rust'ta Ownership

Bu dosya, Rust'ın en temel ve ayırt edici özelliği olan **Ownership** (Sahiplik) sistemini, **Move**, **Clone** ve **Borrowing** kavramlarıyla birlikte ele alır.

---

## 1) Ownership Nedir?

Rust'ta her değerin tek bir **sahibi (owner)** vardır. Sahip scope'dan çıktığında değer otomatik olarak bellekten temizlenir. Bu, garbage collector olmadan bellek güvenliğini sağlar.

```rust
fn main() {
    let kullanici = String::from("Ahmet");
    println!("Kullanıcı: {}", kullanici);
} // kullanici burada scope'dan çıkar ve bellek serbest bırakılır
```

> **Not:** `String` tipi heap üzerinde yer kaplar. Scope sonunda Rust otomatik olarak `drop` fonksiyonunu çağırır ve belleği temizler.

---

## 2) Move (Taşıma)

Heap üzerindeki veriler atama sırasında **taşınır** (move). Orijinal değişken artık geçersiz olur.

```rust
fn main() {
    let sehir = String::from("İstanbul");
    let yeni_sehir = sehir; // Ownership taşındı
    
    println!("Şehir: {}", yeni_sehir);
    
    // println!("{}", sehir); // HATA: value borrowed here after move
}
```

### 🔄 Alternatif Yaklaşım: Stack Verilerinde Copy

Primitive tipler (i32, f64, bool, char) stack üzerinde tutulur ve **copy** semantiği kullanır:

```rust
fn main() {
    let puan = 85;
    let yedek_puan = puan; // Copy yapılır, move değil
    
    println!("Orijinal: {}", puan);      // Geçerli
    println!("Yedek: {}", yedek_puan);   // Geçerli
}
```

> **Önemli:** Stack üzerindeki sabit boyutlu tipler `Copy` trait'ine sahiptir. Heap verileri (String, Vec, vb.) ise `Move` semantiği kullanır.

---

## 3) Clone (Derin Kopyalama)

Heap verisinin tam bir kopyasını almak için `clone()` kullanılır. Bu işlem maliyetlidir çünkü tüm veri yeniden oluşturulur.

```rust
fn main() {
    let mesaj = String::from("Merhaba Dünya");
    let mesaj_kopya = mesaj.clone(); // Derin kopya
    
    println!("Orijinal: {}", mesaj);
    println!("Kopya: {}", mesaj_kopya);
}
```

### ⚡ Performans Notu

`clone()` heap üzerinde yeni alan ayırır ve veriyi byte byte kopyalar. Büyük verilerde performans etkisi göz önünde bulundurulmalıdır.

---

## 4) Borrowing (Ödünç Alma)

Ownership'i taşımadan veriye erişmek için **referans** kullanılır. İki türü vardır:

### Immutable Reference (&T)

Veriyi okumak için kullanılır. Birden fazla immutable referans aynı anda olabilir.

```rust
fn main() {
    let kitap = String::from("Rust Programlama");
    let uzunluk = hesapla_uzunluk(&kitap);
    
    println!("'{}' kitabı {} karakter içeriyor.", kitap, uzunluk);
}

fn hesapla_uzunluk(metin: &String) -> usize {
    metin.len()
}
```

### Mutable Reference (&mut T)

Veriyi değiştirmek için kullanılır. Aynı anda sadece **bir tane** mutable referans olabilir.

```rust
fn main() {
    let mut not = String::from("Bugün hava");
    ekle_metin(&mut not);
    
    println!("{}", not);
}

fn ekle_metin(metin: &mut String) {
    metin.push_str(" güneşli.");
}
```

### ✨ Idiomatic Rust: Borrowing Kuralları

1. Aynı anda ya birden fazla `&T` ya da tek bir `&mut T` olabilir
2. Referanslar her zaman geçerli veriye işaret etmelidir (dangling pointer yok)

```rust
fn main() {
    let mut sayilar = vec![10, 20, 30];
    
    // Birden fazla immutable referans - OK
    let ilk = &sayilar[0];
    let son = &sayilar[sayilar.len() - 1];
    println!("İlk: {}, Son: {}", ilk, son);
    
    // Mutable referans - immutable'lar kullanıldıktan sonra
    sayilar.push(40);
    println!("Güncel: {:?}", sayilar);
}
```

---

## 5) Fonksiyonlarda Ownership

Fonksiyona değer geçirme de ownership'i etkiler:

```rust
fn main() {
    let veri = String::from("Kritik Veri");
    islem_yap(veri); // Ownership fonksiyona taşındı
    
    // println!("{}", veri); // HATA: veri artık geçersiz
}

fn islem_yap(girdi: String) {
    println!("İşleniyor: {}", girdi);
} // girdi burada drop edilir
```

### 🔄 Alternatif: Ownership'i Geri Almak

```rust
fn main() {
    let veri = String::from("Önemli Bilgi");
    let veri = isle_ve_don(veri); // Ownership geri alındı
    
    println!("Sonuç: {}", veri);
}

fn isle_ve_don(mut girdi: String) -> String {
    girdi.push_str(" [İşlendi]");
    girdi // Ownership geri veriliyor
}
```

### ⚡ Daha İyi Yaklaşım: Referans Kullanımı

```rust
fn main() {
    let mut rapor = String::from("2024 Yıllık Rapor");
    guncelle(&mut rapor);
    
    println!("{}", rapor); // Ownership hâlâ main'de
}

fn guncelle(metin: &mut String) {
    metin.push_str(" - Revize Edildi");
}
```

---

## Özet Tablo

| Kavram | Açıklama | Kullanım |
|--------|----------|----------|
| **Ownership** | Her değerin tek sahibi var | `let x = value;` |
| **Move** | Heap verisi atamada taşınır | `let y = x;` |
| **Clone** | Derin kopya oluşturur | `let y = x.clone();` |
| **& (Borrow)** | Okuma referansı | `&x` |
| **&mut (Mutable Borrow)** | Yazma referansı | `&mut x` |

---
