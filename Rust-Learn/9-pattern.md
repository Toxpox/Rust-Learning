# Pattern Matching

Rust'ın en güçlü kontrol mekanizmalarından biri olan **pattern matching**, `match` ifadesi ile çalışır. `Option`, `Result` gibi yerleşik enum yapılarında veya kendi tasarladığımız enum'larda karar yapıları inşa etmek için temiz ve idiomatik bir yol sunar.

---

## 1) Temel `match` Kullanımı — Enum ile Eşleştirme

Aşağıdaki örnekte, bir servise yapılan HTTP isteğinin sonucunu temsil eden `HttpStatus` enum'u tanımlanmıştır. `ping` fonksiyonu rastgele bir durum kodu döner ve `match` ifadesi ile her durum ayrı ayrı ele alınır.

```rust
use rand::Rng;

fn main() {
    let call_response = ping("http://localhost:3456/ping");
    match call_response {
        HttpStatus::Ok => {
            println!("Http Status is OK(200)");
        }
        HttpStatus::Accepted => {
            println!("Http Status is ACCEPTED(201)");
        }
        HttpStatus::NotFound => {
            println!("Http Status is NOT FOUND(404)");
        }
        HttpStatus::BadRequest => {
            println!("Http Status is BAD REQUEST(400)");
        }
        HttpStatus::InternalServerError => {
            println!("Http Status INTERNAL ERROR(500)");
        }
    }
}

enum HttpStatus {
    Ok,
    Accepted,
    NotFound,
    BadRequest,
    InternalServerError,
}

fn ping(service_address: &str) -> HttpStatus {
    let mut generator = rand::thread_rng();
    let lucy_number = generator.gen_range(0..=10);
    println!("Pinging the {service_address}");
    match lucy_number {
        1 => HttpStatus::Ok,                  // Tam eşleşme
        2..=4 => HttpStatus::Accepted,        // Aralık: 2, 3, 4 dahil
        5 => HttpStatus::BadRequest,          // Tam eşleşme
        8 | 10 => HttpStatus::NotFound,       // Çoklu eşleşme: 8 veya 10
        _ => HttpStatus::InternalServerError, // Wildcard: geri kalan tüm durumlar
    }
}
```

### Önemli noktalar

- **Exhaustive matching:** Rust, `match` ifadesinde tüm olasılıkların kapsanmasını zorunlu kılar. Eksik bir dal varsa derleme hatası verir.
- **Wildcard (`_`):** Kapsanmayan tüm durumlar için kullanılır. Dikkatli kullanılmalıdır; yeni bir enum varyantı eklendiğinde sessizce `_` dalına düşebilir.
- **`|` operatörü:** Birden fazla değeri aynı dal altında toplamak için kullanılır.
- **`..=` aralık deseni:** Belirli bir sayısal aralıktaki değerleri eşleştirmek için kullanılır.

---

## 2) Değer Aralıkları ile Pattern Matching

`match` ifadesinde sayısal aralıklar kullanarak öğrenci not değerlendirmesi yapılabilir.

```rust
struct Student {
    id: u32,
    full_name: String,
    grade: u8,
}

fn check_exam(student: Student) {
    match student.grade {
        0..=49 => println!("[{}]{} failed.", student.id, student.full_name),
        50..=79 => println!("[{}]{} passed.", student.id, student.full_name),
        80..=100 => println!(
            "[{}]{} passed with congrats.",
            student.id, student.full_name
        ),
        _ => println!("Invalid grade score"),
    }
}
```

```rust
// Kullanım
check_exam(Student {
    id: 1,
    full_name: String::from("Burak De La Fuante Dos Selimos"),
    grade: 44,
});
```

> **Not:** `u8` tipi 0–255 arası değer alabilir. 101–255 arası değerler `_` dalına düşer. Eğer `grade` alanı iş kuralı gereği 0–100 arasında olmalıysa, bir `new` fonksiyonu ile bu kısıtlama yapı oluşturulurken denetlenmelidir.

---

## 3) Enum ile Veri Taşıma ve Destructuring

Rust enum'ları varyantlarında veri taşıyabilir. `match` ifadesi bu veriyi destructure ederek kullanıma sunar.

```rust
enum CustomerTransaction {
    Deposit(f64),
    Withdraw(f64),
}

fn process_transaction(balance: &mut f64, transaction: CustomerTransaction) {
    match transaction {
        CustomerTransaction::Deposit(amount) => {
            *balance += amount;
            println!("Deposited ${}\nNew balance: ${}.", amount, balance);
        }
        CustomerTransaction::Withdraw(amount) => {
            if *balance >= amount {
                *balance -= amount;
                println!("Withdraw ${}\nNew balance: ${}.", amount, balance);
            } else {
                println!("Insufficient funds.");
            }
        }
    }
}
```

```rust
// Kullanım
let mut balance = 1000.0;
process_transaction(&mut balance, CustomerTransaction::Deposit(400.0));
process_transaction(&mut balance, CustomerTransaction::Withdraw(50.0));
```

### Açıklama

- `Deposit(f64)` ve `Withdraw(f64)` varyantları, işlem tutarını kendi içlerinde taşır.
- `match` ifadesinde `amount` adıyla bu değer çıkarılır (destructure edilir) ve fonksiyon gövdesinde kullanılır.
- `balance` parametresi `&mut f64` olarak alınır; böylece fonksiyon, çağrıldığı yerdeki orijinal bakiye değerini doğrudan değiştirebilir.

---

## 4) `Option<T>` ile Pattern Matching

`Option<T>`, Rust'ta bir değerin var olup olmadığını ifade eden yerleşik enum'dur. `Some(T)` veya `None` değerlerinden birini alır. Null pointer hatalarını derleme zamanında engellemek için tasarlanmıştır.

```rust
struct User {
    id: u32,
    title: String,
    email: Option<String>, // Email alanı zorunlu değil
}

impl User {
    fn new(id: u32, title: String, email: Option<String>) -> Self {
        User { id, title, email }
    }

    fn info(&self) -> String {
        match self.email {
            Some(ref em) => format!("{}-{} ({})", self.id, self.title, em),
            None => format!("{} ({})", self.id, self.title),
        }
    }
}
```

```rust
// Kullanım
let klyde = User::new(19, String::from("Jhony Klyde"), None);
println!("{}", klyde.info());

let zee = User::new(
    23,
    String::from("Zee"),
    Some(String::from("zee@somewhere.abc")),
);
println!("{}", zee.info());
```

> **Uyarı:** `match self.email` ifadesinde `Some(ref em)` kullanılmıştır. `ref` anahtar kelimesi, `String` değerinin sahipliğini (ownership) almak yerine referans olarak ödünç alır. Bu olmadan `self.email` move edilir ve struct artık kullanılamaz hale gelir.

### ✨ İdiomatic Rust: `as_ref()` Kullanımı

`ref` anahtar kelimesi yerine `as_ref()` metodu kullanılarak daha okunabilir bir versiyon yazılabilir:

```rust
fn info(&self) -> String {
    match self.email.as_ref() {
        Some(em) => format!("{}-{} ({})", self.id, self.title, em),
        None => format!("{} ({})", self.id, self.title),
    }
}
```

`as_ref()`, `Option<String>` değerini `Option<&String>` formuna dönüştürür. Böylece `ref` anahtar kelimesine ihtiyaç kalmaz.

---

## 5) `Option<&T>` Döndüren Fonksiyonlar

Bir koleksiyon içinde arama yapan fonksiyonlar genellikle `Option<&T>` döndürür. Sonuç bulunamazsa `None`, bulunursa `Some(&T)` gelir.

```rust
struct Account {
    id: u32,
    holder_name: String,
    balance: f64,
}

fn find_account(accounts: &Vec<Account>, id: u32) -> Option<&Account> {
    accounts.iter().find(|acc| acc.id == id)
}

fn load_accounts() -> Vec<Account> {
    vec![
        Account { id: 1001, holder_name: "Nora Min".to_string(), balance: 1000.0 },
        Account { id: 1002, holder_name: "Agnis Yang".to_string(), balance: 750.0 },
        Account { id: 1003, holder_name: "Valeri Mora".to_string(), balance: 850.0 },
        Account { id: 1004, holder_name: "Monti Konti".to_string(), balance: 275.0 },
    ]
}
```

### `match` ile Kullanım

```rust
let accounts = load_accounts();
let result = find_account(&accounts, 1003);

match result {
    Some(account) => println!(
        "Account for '{}' found: {} with balance ${}",
        account.id, account.holder_name, account.balance
    ),
    None => println!("Account not found."),
}
```

### `if let` ile Kullanım

Yalnızca `Some` durumunu ele almak yeterliyse, `if let` daha kısa ve okunabilir bir alternatif sunar:

```rust
if let Some(account) = find_account(&accounts, 1002) {
    println!(
        "Account for '{}' found: {} with balance ${}",
        account.id, account.holder_name, account.balance
    );
}
```

> **Not:** `if let`, sadece tek bir dal (genellikle `Some`) ile ilgilenildiğinde `match` ifadesine kıyasla daha az şablon kod (boilerplate) üretir. Ancak birden fazla dalın ele alınması gerektiğinde `match` tercih edilmelidir.

### 🔄 Alternatif Yaklaşım: `find_account` için Slice Parametresi

```rust
fn find_account(accounts: &[Account], id: u32) -> Option<&Account> {
    accounts.iter().find(|acc| acc.id == id)
}
```

`&Vec<Account>` yerine `&[Account]` (slice) kullanmak daha idiomatiktir. Slice, hem `Vec` hem de dizi (array) ile çalışabildiğinden fonksiyon daha esnek hale gelir.

---
