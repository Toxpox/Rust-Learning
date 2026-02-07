# Rust ile Fonksiyonlar

Bu dosya, Rust'ta fonksiyon tanımlama, parametre geçirme, değer döndürme ve yaygın kullanım kalıplarını gösterir.

---

## 1) Basit Fonksiyon Tanımlama

Rust'ta çalıştırılabilir bir programın giriş noktası `fn main()` fonksiyonudur. Dönüş tipi belirtilmezse `()` (unit type) döndürüldüğü varsayılır.

```rust
fn main() {
    print_greetings();
}

fn print_greetings() {
    println!("Welcome to my new world!");
}
```

---

## 2) Parametre ile Fonksiyon

Parametre türü zorunlu olarak belirtilmelidir. `&str` kullanımı, fonksiyonun hem `String` hem de `&str` kabul etmesini sağlar.

```rust
fn main() {
    let name = "Can Cloud Van Dam";
    greet(name);
}

fn greet(name: &str) {
    println!("Hello {}! How are you?", name);
}
```

---

## 3) Değer Döndüren Fonksiyon

`->` işareti dönüş türünü belirtir. Son ifade (noktalı virgül olmadan) otomatik olarak döndürülür.

```rust
fn main() {
    println!("4 + 5 = {}", sum(4, 5));
}

fn sum(value_1: i32, value_2: i32) -> i32 {
    value_1 + value_2
}

fn double_it(value: i32) -> i32 {
    value * 2
}
```

> Not: `return` anahtar kelimesi yalnızca erken dönüş gerektiğinde kullanılır. Son satırda `return` kullanmak idiomatik değildir.

---

## 4) Mutable Parametreler ve Tuple Dönüşü

`mut` parametreler fonksiyon içinde değiştirilebilir. Orijinal değer kopyalanır (f32 `Copy` trait'ini implemente eder).

```rust
fn main() {
    let (x, y) = move_position(10.0, 5.0, 1.0);
    println!("{:?}:{:?}", x, y);
}

fn move_position(mut x: f32, mut y: f32, acceleration: f32) -> (f32, f32) {
    x += acceleration;
    y += acceleration;
    (x, y)
}
```

Tuple dönüşü, birden fazla değer döndürmenin pratik yoludur.

### Struct ile alternatif yaklaşım

Daha karmaşık senaryolarda struct kullanımı okunabilirliği artırır:

```rust
struct Position { x: f32, y: f32 }

fn move_position(pos: Position, acceleration: f32) -> Position {
    Position {
        x: pos.x + acceleration,
        y: pos.y + acceleration,
    }
}
```

---

## 5) Referans ile Vector Geçirme

`&Vec<i32>` ile vector'ün sahipliği alınmaz, sadece ödünç alınır (borrowing).

```rust
fn main() {
    let numbers = vec![1, 4, 6, 2, 8, 10, 3, 7, 27, 9, 144, 1024, 5];
    let evens = get_evens(&numbers);
    println!("{:?}", evens);
}

fn get_evens(numbers: &Vec<i32>) -> Vec<i32> {
    let mut evens: Vec<i32> = Vec::new();
    for number in numbers {
        if *number % 2 == 0 {
            evens.push(*number);
        }
    }
    evens
}
```

### Slice ile daha esnek yaklaşım

`&Vec<T>` yerine `&[T]` (slice) kullanmak daha idiomatiktir:

```rust
fn get_evens(numbers: &[i32]) -> Vec<i32> {
    numbers.iter().filter(|&&n| n % 2 == 0).cloned().collect()
}
```

Bu şekilde hem `Vec` hem de array referansları kabul edilir.

---

## 6) Iterator Chain Kullanımı

Fonksiyonel yaklaşım ile veri işleme:

```rust
fn main() {
    let numbers = vec![1, 4, 6, 2, 8, 10, 3, 7, 27, 9, 144, 1024, 5];
    let odds = get_odds(&numbers);
    println!("{:?}", odds);
}

fn get_odds(numbers: &[i32]) -> Vec<i32> {
    numbers
        .iter()
        .filter(|&&i| i % 2 != 0)
        .cloned()
        .collect()
}
```

- `.iter()`: Referans iterator'ı oluşturur.
- `.filter()`: Koşula uyan elemanları seçer.
- `.cloned()`: `&i32`'yi `i32`'ye dönüştürür.
- `.collect()`: Sonucu koleksiyona toplar.

> Not: `Copy` türleri için `.cloned()` yerine `.copied()` kullanmak daha açık ve idiomatiktir.

---

## 7) Ownership Alan Fonksiyon

Bu fonksiyon `Vec`'in sahipliğini alır. Çağrıldıktan sonra orijinal vector kullanılamaz.

```rust
fn main() {
    let names = vec!["caN", "clOUD", "vAn", "DaM"];
    print_all_to_upper(names);
    // names artık kullanılamaz
}

fn print_all_to_upper(names: Vec<&str>) {
    for name in names {
        println!("{}", name.to_uppercase());
    }
}
```

### Borrowing ile alternatif

Sahipliği almadan çalışmak için slice kullanılabilir:

```rust
fn print_all_to_upper(names: &[&str]) {
    for name in names {
        println!("{}", name.to_uppercase());
    }
}

// Çağırırken:
print_all_to_upper(&names); // names hala kullanılabilir
```

---

## Özet

| Kavram | Açıklama |
|--------|----------|
| `fn` | Fonksiyon tanımlama anahtar kelimesi |
| `->` | Dönüş türünü belirtir |
| `mut` parametre | Kopyalanan değer fonksiyon içinde değiştirilebilir |
| Tuple dönüş | Birden fazla değer döndürmenin basit yolu |
| `&[T]` | Slice parametresi, daha esnek |
| Iterator chain | Fonksiyonel ve verimli veri işleme |
