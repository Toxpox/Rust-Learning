# Generic Türler

Generic türler, aynı fonksiyonu veya veri yapısını farklı tipler için tekrar tekrar yazmak yerine **tek bir tanımla birden fazla tiple** çalışabilmeyi sağlar. Rust'ın tip güvenliğinden ödün vermeden kod tekrarını azaltan temel mekanizmalardan biridir. C# ve Go gibi dillerde de yaygın olarak karşılaşılan bu kavram, Rust'ta **trait bound**'lar ile birleşerek güçlü derleme zamanı garantileri sunar.

---

## 1) Generic Fonksiyon

Aşağıdaki `log_any` fonksiyonu, **herhangi bir türden** parametre alarak onu konsola yazdırır.

```rust
use std::fmt::Debug;

fn log_any<T: Debug>(object: T) {
    println!("Logged value is '{:?}'", object);
}
```

```rust
// Kullanım
log_any(3.14f32);
log_any("TCP connection established");
log_any(false);
```

### Açıklama

- `<T: Debug>` söz dizimi bir **trait bound** (kısıtlama) tanımlar. `T` herhangi bir tür olabilir, ancak `Debug` trait'ini uygulamış olmalıdır.
- `{:?}` format belirteci (debug format), `Debug` trait'ini gerektirdiği için bu kısıtlama zorunludur. Kısıtlama olmadan derleme hatası alınır.
- `T` burada **ownership** alır. Fonksiyon çağrıldıktan sonra gönderilen değer artık çağıran tarafta kullanılamaz (Copy trait'ini uygulamayan türler için).

### Trait Bound Nedir?

Trait bound, generic bir tipe hangi davranışları (trait'leri) uygulamış olması gerektiğini belirtir. Bu sayede compiler, `T` üzerinde hangi metotların çağrılabileceğini derleme zamanında doğrular.

### 🔄 Alternatif Yaklaşım: `where` Söz Dizimi

Özellikle birden fazla trait bound olduğunda `where` söz dizimi okunabilirliği artırır:

```rust
fn log_any<T>(object: T)
where
    T: Debug,
{
    println!("Logged value is '{:?}'", object);
}
```

---

## 2) Generic Enum ile Kullanım

Kendi enum türlerimizi de generic fonksiyonlarla birlikte kullanabiliriz. Aşağıdaki `State` enum'u `Debug` trait'ini derive ederek `log_any` fonksiyonu ile uyumlu hale gelir.

```rust
#[derive(Debug)]
enum State {
    InProgress,
    Done,
    Error,
}
```

```rust
// Kullanım
log_any(State::InProgress);
```

> **Not:** `#[derive(Debug)]` özniteliği, Rust compiler'ına `Debug` trait'ini otomatik olarak uygulamasını söyler. Özel bir formatlama gerekmedikçe `derive` kullanımı standart yaklaşımdır.

---

## 3) Generic Struct — `Point<T>`

Yaygın kullanılan generic yapılardan biri koordinat sistemidir. Aşağıda 3 boyutlu bir `Point` veri yapısı tanımlanmıştır.

```rust
use std::fmt::Debug;
use std::ops::Add;

struct Point<T: Copy + Debug + Add<Output = T>> {
    x: T,
    y: T,
    z: T,
}
```

### Trait Bound'lar

| Trait | Neden Gerekli |
|-------|---------------|
| `Copy` | Değerlerin kopyalanabilmesi için. Aksi halde `self.x + other.x` gibi ifadelerde ownership sorunu oluşur. |
| `Debug` | `{:?}` format belirteci ile yazdırılabilmesi için. |
| `Add<Output = T>` | `+` operatörünün kullanılabilmesi için. `Output = T`, toplama sonucunun da aynı tür olacağını garanti eder. |

Bu üç kısıtlama birlikte, `Point<T>` yapısının yalnızca sayısal türlerle (`i32`, `f32`, `u8` vb.) çalışmasını sağlar.

---

## 4) Generic Struct için `impl` Bloğu

`Point<T>` yapısına metotlar eklemek için `impl` bloğu aynı trait bound'ları tekrar eder:

```rust
impl<T: Copy + Debug + Add<Output = T>> Point<T> {
    fn new(x: T, y: T, z: T) -> Self {
        Point { x, y, z }
    }

    fn info(&self) -> String {
        format!("({:?}, {:?}, {:?})", self.x, self.y, self.z)
    }

    fn add(self, other: Point<T>) -> Point<T> {
        Point {
            x: self.x + other.x,
            y: self.y + other.y,
            z: self.z + other.z,
        }
    }
}
```

```rust
// Kullanım
let game_point = Point::<i32>::new(10, 20, 10);
println!("{}", game_point.info());

let vehicle_position: Point<f32> = Point::new(5.5, 3.14, 2.20);
println!("Vehicle Position {}", vehicle_position.info());

let vehicle_new_position: Point<f32> = vehicle_position.add(Point::new(1.0, 1.0, 2.0));
println!("New position after move {}", vehicle_new_position.info());
```

### Açıklama

- `Point::<i32>::new(...)` söz diziminde **turbofish** (`::<>`) kullanılarak tür açıkça belirtilmiştir.
- `let vehicle_position: Point<f32>` satırında ise tür değişken üzerinde annotasyon ile belirtilmiştir. Her iki yöntem de geçerlidir; Rust compiler'ı çoğu durumda türü otomatik çıkarabilir (**type inference**).
- `add` metodu `self`'i move alır (`&self` değil). Bu nedenle `vehicle_position`, `add` çağrısından sonra artık kullanılamaz. `Copy` trait'i yalnızca alanlar için geçerlidir; struct'ın kendisi için `derive(Copy)` yapılmadığı sürece move semantiği geçerlidir.

### ✨ İdiomatic Rust: `Add` Trait'ini `std::ops::Add` Olarak Uygulamak

`add` metodunu doğrudan struct üzerinde tanımlamak yerine, `std::ops::Add` trait'ini implemente etmek `+` operatörünün kullanılabilmesini sağlar:

```rust
impl<T: Copy + Debug + Add<Output = T>> std::ops::Add for Point<T> {
    type Output = Point<T>;

    fn add(self, other: Point<T>) -> Point<T> {
        Point {
            x: self.x + other.x,
            y: self.y + other.y,
            z: self.z + other.z,
        }
    }
}

// Artık + operatörü kullanılabilir
let new_position = vehicle_position + Point::new(1.0, 1.0, 2.0);
```

Bu yaklaşım, Rust'ın operatör aşırı yükleme (operator overloading) kurallarına uygundur ve kodu daha doğal bir şekilde okunabilir kılar.

### ⚡ Performans Notu

Generic türler Rust'ta **monomorphization** ile derlenir. Yani `Point<i32>` ve `Point<f32>` için compiler ayrı ayrı kod üretir. Bu, çalışma zamanında (runtime) herhangi bir performans maliyeti olmadığı anlamına gelir; soyutlama tamamen derleme zamanında çözülür.

---
