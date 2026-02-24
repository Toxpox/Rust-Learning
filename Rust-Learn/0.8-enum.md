# Rust'ta Enum Yapıları

Bu dosya, Rust dilindeki **enum** veri yapısını, enum'lara metot eklemeyi, **pattern matching** kullanımını, **Option** enum'ını ve enum'ların struct'larla birlikte kullanımını ele alır.

---

## 1) Temel Enum Tanımı

Enum, bir değişkenin alabileceği olası durumları (variant) tanımlamak için kullanılır. Her variant farklı veri taşıyabilir veya hiç veri taşımayabilir.

```rust
enum HttpStatus {
    Ok,
    NotFound,
    ServerError(String),
}

fn main() {
    let status = HttpStatus::Ok;
    let error = HttpStatus::ServerError("Database connection timed out".to_string());

    print_status(&status);
    print_status(&error);
}

fn print_status(status: &HttpStatus) {
    match status {
        HttpStatus::Ok => println!("✅ 200 - Request succeeded"),
        HttpStatus::NotFound => println!("❌ 404 - Resource not found"),
        HttpStatus::ServerError(msg) => println!("🔥 500 - {}", msg),
    }
}
```

`HttpStatus` enum'ı üç variant barındırır. `Ok` ve `NotFound` herhangi bir veri taşımazken, `ServerError` bir `String` değer içerir. `match` bloğu ile tüm olası durumlar ele alınır.

> **Uyarı:** `match` ifadesinde enum'ın tüm variant'ları kapsanmalıdır (exhaustive matching). Eksik bırakılan bir variant derleme hatasına neden olur. Tüm durumları tek tek ele almak istemiyorsanız `_ => {}` ile geri kalanları yakalayabilirsiniz.

---

## 2) Enum'lara Metot Ekleme (`impl`)

Struct'larda olduğu gibi, enum'lara da `impl` bloğu ile metot tanımlanabilir.

```rust
enum TrafficLight {
    Red,
    Yellow,
    Green,
}

impl TrafficLight {
    fn duration_seconds(&self) -> u32 {
        match self {
            TrafficLight::Red => 60,
            TrafficLight::Yellow => 5,
            TrafficLight::Green => 45,
        }
    }

    fn is_safe_to_go(&self) -> bool {
        matches!(self, TrafficLight::Green)
    }
}

fn main() {
    let light = TrafficLight::Red;
    println!("Duration: {} seconds", light.duration_seconds());
    println!("Safe to go: {}", light.is_safe_to_go());

    let green = TrafficLight::Green;
    println!("Green duration: {} seconds", green.duration_seconds());
    println!("Safe to go: {}", green.is_safe_to_go());
}
```

### ✨ Idiomatic Rust

`matches!` makrosu, bir değerin belirli bir pattern ile eşleşip eşleşmediğini tek satırda kontrol eder. `match` bloğu ile `true`/`false` döndürmek yerine `matches!` kullanmak daha idiomatik ve okunabilirdir.

---

## 3) Veri Taşıyan Enum Variant'ları

Enum variant'ları farklı türde ve miktarda veri barındırabilir. Bu, Rust'ın en güçlü özelliklerinden biridir ve klasik OOP'deki sınıf hiyerarşilerinin yerine geçebilir.

```rust
enum Shape {
    Circle(f64),                    // Sadece yarıçap
    Rectangle { width: f64, height: f64 }, // Named fields
    Triangle(f64, f64, f64),        // Üç kenar uzunluğu
}

impl Shape {
    fn area(&self) -> f64 {
        match self {
            Shape::Circle(radius) => std::f64::consts::PI * radius * radius,
            Shape::Rectangle { width, height } => width * height,
            Shape::Triangle(a, b, c) => {
                // Heron formülü
                let s = (a + b + c) / 2.0;
                (s * (s - a) * (s - b) * (s - c)).sqrt()
            }
        }
    }
}

fn main() {
    let shapes: Vec<Shape> = vec![
        Shape::Circle(7.5),
        Shape::Rectangle { width: 12.0, height: 8.0 },
        Shape::Triangle(3.0, 4.0, 5.0),
    ];

    for shape in &shapes {
        println!("Area: {:.2}", shape.area());
    }
}
```

Bu örnekte `Shape` enum'ı üç farklı geometrik şekli temsil eder. Her variant farklı yapıda veri taşır: tuple-like, struct-like ve çoklu değer. Tüm şekiller `Vec<Shape>` içinde homojen bir koleksiyonda tutulabilir.

### 🔄 Alternatif Yaklaşım

Struct-like variant (`Rectangle { width, height }`) kullanmak, tuple-like variant'a göre daha okunabilirdir. Özellikle iki veya daha fazla aynı tipte parametre varsa, alanları isimlendirmek hata riskini azaltır.

---

## 4) Enum ve Struct Birlikte Kullanımı

Enum'lar struct alanlarında kullanılarak zengin veri modelleri oluşturulabilir. `#[derive(Debug)]` trait'i ile struct ve enum'ların `{:?}` formatında yazdırılması sağlanır.

```rust
#[derive(Debug)]
enum Priority {
    Low,
    Medium,
    High,
    Critical(String),
}

#[derive(Debug)]
struct Ticket {
    id: u32,
    summary: String,
    priority: Priority,
    resolved: bool,
}

impl Ticket {
    fn new(id: u32, summary: String, priority: Priority) -> Self {
        Ticket {
            id,
            summary,
            priority,
            resolved: false,
        }
    }

    fn resolve(&mut self) {
        self.resolved = true;
    }

    fn is_urgent(&self) -> bool {
        matches!(self.priority, Priority::High | Priority::Critical(_))
    }
}

fn main() {
    let mut ticket = Ticket::new(
        1042,
        String::from("Authentication service unresponsive"),
        Priority::Critical("Affects all users".to_string()),
    );

    println!("{:#?}", ticket);
    println!("Urgent: {}", ticket.is_urgent());

    ticket.resolve();
    println!("Resolved: {}", ticket.resolved);
}
```

> **Not:** `#[derive(Debug)]` direktifi bir struct'a uygulandığında, o struct'ın içerdiği tüm tiplerin de `Debug` trait'ini implemente etmesi gerekir. `Ticket` içinde `Priority` kullanıldığı için her ikisine de `#[derive(Debug)]` eklenmelidir.

---

## 5) Enum ile Durum Modelleme (State Pattern)

Enum'lar, bir nesnenin farklı durumlarını ve her duruma özgü veriyi modellemek için idealdir. Bu yaklaşım, klasik OOP'deki State Pattern'in Rust karşılığıdır.

```rust
use std::time::SystemTime;

#[derive(Debug)]
enum Session {
    Anonymous,
    LoggedIn {
        username: String,
        login_time: SystemTime,
    },
    Suspended {
        username: String,
        reason: String,
    },
}

impl Session {
    fn login(username: String) -> Session {
        Session::LoggedIn {
            username,
            login_time: SystemTime::now(),
        }
    }

    fn suspend(&self, reason: String) -> Option<Session> {
        match self {
            Session::LoggedIn { username, .. } => Some(Session::Suspended {
                username: username.clone(),
                reason,
            }),
            _ => None,
        }
    }

    fn display_info(&self) {
        match self {
            Session::Anonymous => println!("Guest session — no user data"),
            Session::LoggedIn { username, .. } => {
                println!("{} is currently logged in", username);
            }
            Session::Suspended { username, reason } => {
                println!("{} suspended: {}", username, reason);
            }
        }
    }
}

fn main() {
    let session = Session::login(String::from("neo"));
    session.display_info();

    if let Some(suspended) = session.suspend("Policy violation".to_string()) {
        suspended.display_info();
    }

    let guest = Session::Anonymous;
    guest.display_info();
}
```

Burada `Session` enum'ı bir kullanıcının oturum durumunu modeller. `suspend` metodu `Option<Session>` döner: Yalnızca `LoggedIn` durumundayken askıya alınabilir, diğer durumlarda `None` döner.

---

## 6) Built-in `Option` Enum'ı

Rust'ta `null` kavramı yoktur. Bir değerin var olup olmadığını ifade etmek için standart kütüphanedeki `Option<T>` enum'ı kullanılır.

```rust
fn find_element(data: &[i32], target: i32) -> Option<usize> {
    for (index, &value) in data.iter().enumerate() {
        if value == target {
            return Some(index);
        }
    }
    None
}

fn main() {
    let numbers = [10, 25, 37, 42, 58];

    match find_element(&numbers, 42) {
        Some(idx) => println!("Found at index {}", idx),
        None => println!("Element not found"),
    }

    // if let ile kısa kullanım
    if let Some(idx) = find_element(&numbers, 99) {
        println!("Found at index {}", idx);
    } else {
        println!("99 is not in the array");
    }
}
```

`Option<T>`, `Some(T)` ve `None` olmak üzere iki variant'a sahip generic bir enum'dır. Bazı dillerdeki `null`/`nil` kontrollerinin derleme zamanında güvenli bir şekilde yapılmasını sağlar.

### ⚡ Performans Notu

`Option<T>`, `T` referans tipi olduğunda (örneğin `Option<&T>`), Rust derleyicisi **null pointer optimization** uygular. Bu durumda `Option` sarmalayıcısı bellekte ekstra yer kaplamaz; `None` değeri doğrudan null pointer olarak temsil edilir.

---
