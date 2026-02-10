# Rust'ta Struct

Bu dosya, Rust dilindeki üç farklı **struct** türünü, **impl** blokları ile metot tanımlamayı ve struct'ların pratik kullanımını ele alır.

---

## 1) Named Field Struct

Named Field struct, her alanına isim verilen en yaygın struct türüdür. Alanlar `key: value` formatında tanımlanır ve erişilir.

```rust
struct Vehicle {
    brand: String,
    year: u16,
    is_electric: bool,
    top_speed: f32,
}

fn main() {
    let car = Vehicle {
        brand: String::from("Rivian"),
        year: 2025,
        is_electric: true,
        top_speed: 201.0,
    };

    println!("{} ({}) - Electric: {}", car.brand, car.year, car.is_electric);
    println!("Top Speed: {} km/h", car.top_speed);
}
```

Burada `Vehicle` struct'ı dört farklı tipte alana sahiptir. `String`, `u16`, `bool` ve `f32` tipleri aynı yapı içinde bir arada tutulabilir. Named field struct, okunabilirliği yüksek olduğu için karmaşık veri modellerinde tercih edilir.

> **Not:** Struct alanlarına atama sırası önemli değildir. Önemli olan her alanın tanımlı olmasıdır.

---

## 2) Tuple Struct

Tuple struct, alanlara isim verilmeyen ancak sıralı tipleri gruplayan bir yapıdır. Elemanlara indeks ile erişilir (`.0`, `.1` gibi).

```rust
struct Color(u8, u8, u8);

fn main() {
    let sunset_orange = Color(255, 99, 71);

    println!("R: {}, G: {}, B: {}", sunset_orange.0, sunset_orange.1, sunset_orange.2);
}
```

Tuple struct, özellikle anlamsal bir tip oluşturmak istediğiniz durumlarda faydalıdır. Örneğin üç adet `u8` değeri tek başına anlamsızken, `Color` olarak sarmalandığında kodun okunabilirliği artar.

### 🔄 Alternatif Yaklaşım

Destructuring ile tuple struct'ın elemanlarını doğrudan değişkenlere atayabilirsiniz:

```rust
fn main() {
    let sky_blue = Color(135, 206, 235);
    let Color(r, g, b) = sky_blue;

    println!("Sky: R={}, G={}, B={}", r, g, b);
}
```

Bu yöntem, elemanları birden fazla yerde kullanacaksanız indeks tekrarından kaçınmanızı sağlar.

---

## 3) Unit Struct

Unit struct, hiçbir alanı olmayan bir yapıdır. Genellikle trait implementasyonları veya Entity Component System (ECS) gibi mimari desenlerde *marker* (işaretleyici) olarak kullanılır.

```rust
struct Marker;

fn main() {
    let _tag = Marker;
    println!("Marker struct oluşturuldu. Veri taşımaz, sadece bir tip olarak var olur.");
}
```

> **Not:** Unit struct, bellekte yer kaplamaz (zero-sized type). Bu özellik, tip düzeyinde kontrol sağlamak istediğiniz senaryolarda oldukça verimlidir.

---

## 4) Factory Fonksiyonu ile Struct Oluşturma

Struct alanlarını dışarıdan bir fonksiyon aracılığıyla doldurmak, başlangıç mantığını merkezileştirmek için kullanışlıdır. Rust'ta `fn` parametresi ile struct alanı aynı isme sahipse kısa yazım (field init shorthand) kullanılabilir.

```rust
struct Sensor {
    id: String,
    temperature: f64,
    is_online: bool,
}

fn create_sensor(id: String, temperature: f64) -> Sensor {
    Sensor {
        id,               // field init shorthand
        temperature,
        is_online: true,
    }
}

fn main() {
    let s1 = create_sensor(String::from("TMP-4021"), 36.6);
    println!("Sensor [{}]: {}°C — Online: {}", s1.id, s1.temperature, s1.is_online);
}
```

Burada `id` ve `temperature` parametreleri ile struct alanları aynı isme sahip olduğu için `id: id` yazmaya gerek yoktur. Bu, Rust'ın sunduğu **field init shorthand** özelliğidir.

---

## 5) `impl` Bloğu ile Metotlar

`impl` bloğu, bir struct'a metot ve ilişkili fonksiyon (associated function) eklemek için kullanılır.

- `&self` alan metotlar → instance üzerinde çağrılır.
- `&mut self` alan metotlar → instance'ı değiştirebilir.
- `self` parametresi olmayan fonksiyonlar → `Struct::fonksiyon()` şeklinde çağrılır (constructor pattern).

```rust
struct Circle {
    radius: f64,
}

impl Circle {
    // Associated function (constructor)
    fn new(radius: f64) -> Self {
        Circle { radius }
    }

    // Immutable borrow: sadece okuma
    fn area(&self) -> f64 {
        std::f64::consts::PI * self.radius * self.radius
    }

    // Immutable borrow: sadece okuma
    fn circumference(&self) -> f64 {
        2.0 * std::f64::consts::PI * self.radius
    }

    // Mutable borrow: değiştirme
    fn scale(&mut self, factor: f64) {
        self.radius *= factor;
    }
}

fn main() {
    let mut circle = Circle::new(5.0);

    println!("Radius: {:.2}", circle.radius);
    println!("Area: {:.2}", circle.area());
    println!("Circumference: {:.2}", circle.circumference());

    circle.scale(2.0);
    println!("\n--- After scaling x2 ---");
    println!("Radius: {:.2}", circle.radius);
    println!("Area: {:.2}", circle.area());
    println!("Circumference: {:.2}", circle.circumference());
}
```

### ⚡ Performans Notu

`&self` (immutable borrow) kullanan metotlar struct'ın sahipliğini almaz, sadece okuma erişimi sağlar. Bu sayede struct birden fazla yerde referans olarak kullanılabilir. Sahipliği taşımak gerekmedikçe her zaman referans tercih edilmelidir.

### ✨ Idiomatic Rust

`Self` anahtar kelimesi, `impl` bloğu içinde struct'ın kendi tipini temsil eder. `Circle { radius }` yazmak yerine `Self { radius }` yazmak da geçerlidir ve özellikle struct adı uzun olduğunda okunabilirliği artırır:

```rust
impl Circle {
    fn new(radius: f64) -> Self {
        Self { radius }
    }
}
```

---

## 6) Struct'ları Bir Arada Kullanma

Farklı struct türleri iç içe (nested) kullanılarak daha karmaşık veri modelleri oluşturulabilir.

```rust
struct Coordinate(f64, f64);

struct Satellite {
    name: String,
    orbit_km: u32,
    position: Coordinate,
    is_active: bool,
}

fn launch_satellite(name: String, orbit_km: u32) -> Satellite {
    Satellite {
        name,
        orbit_km,
        position: Coordinate(0.0, 0.0),
        is_active: true,
    }
}

fn main() {
    let sat = launch_satellite(String::from("Stargazer-7"), 550);

    println!(
        "{} — Orbit: {} km — Active: {} — Pos: ({}, {})",
        sat.name, sat.orbit_km, sat.is_active, sat.position.0, sat.position.1
    );
}
```

Bu örnekte `Coordinate` bir tuple struct, `Satellite` ise named field struct'tır. `position` alanı `Coordinate` tipinde olduğu için struct'lar kompozisyon yoluyla bir araya getirilmiştir. Rust'ta kalıtım (inheritance) yoktur; bunun yerine **kompozisyon** (composition) tercih edilir.

> **Uyarı:** `String` tipindeki alanlar heap üzerinde tutulur. `name` alanını bir fonksiyona taşıdığınızda (move), orijinal struct artık kullanılamaz hale gelir. Sadece okuma erişimi gerekiyorsa `&sat.name` şeklinde referans kullanılmalıdır.

---
