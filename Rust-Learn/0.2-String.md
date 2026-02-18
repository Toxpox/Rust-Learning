# Rust ile String İşlemleri

Bu dosya, Rust'ta **String** ve **&str** (string slice) türlerini, aralarındaki farkları ve yaygın kullanım senaryolarını gösterir.

---

## 1) String Oluşturma Yöntemleri

Rust'ta `String` türü heap üzerinde dinamik olarak ayrılmış, değiştirilebilir ve sahipliği olan bir veri türüdür.

```rust
fn main() {
    let hero_name = "Can Cloud Van Dam".to_string();
    println!("{}", hero_name);
    let position = String::from("Quarter back");
    println!("{}", position);
}
```

- `.to_string()`: `Display` trait'ini implemente eden herhangi bir tür üzerinde kullanılabilir.
- `String::from()`: Doğrudan `&str`'den `String` oluşturur.

### Alternatif yaklaşım

`.into()` metodu, `From` trait'i sayesinde tür dönüşümü yapar:

```rust
let hero_name: String = "Can Cloud Van Dam".into();
```

---

## 2) String Değiştirme

`replace()` metodu orijinal string'i değiştirmez, yeni bir `String` döndürür.

```rust
fn main() {
    let hero_name = "Can Cloud Van Dam".to_string();
    let short_name = hero_name.replace("Can Cloud Van Dam", "CCVD");
    println!("{}", &short_name);
}
```

> Not: Birden fazla değiştirme yapılacaksa `replacen()` veya regex tabanlı çözümler düşünülebilir.

---

## 3) &str (String Slice) Kullanımı

`&str`, bir string'in belli bir bölümüne referans verir. Heap üzerinde yeni alan ayırmaz.

```rust
fn main() {
    let greetings = "Greetings dear young and crazy brains".to_string();
    let first_word = greetings.get(0..10).unwrap();
    println!("{}", first_word);

    let first_words = "hello there"; // doğrudan &str
    println!("{}", first_words);
}
```

- `get(0..10)`: Güvenli dilim alma yöntemi, `Option<&str>` döner.
- `.unwrap()`: `Some` değerini çıkarır; `None` durumunda panik yapar.

### Daha güvenli yaklaşım

```rust
let first_word = greetings.get(0..10).unwrap_or("Varsayılan");
```

> Uyarı: Byte indeksleri UTF-8 karakter sınırlarına denk gelmezse program panik yapar. Türkçe karakterlerde dikkatli olunmalıdır.

---

## 4) Literal &str ve String Dönüşümü

String literal'ler (`"..."`) varsayılan olarak `&'static str` türündedir.

```rust
fn main() {
    let word_aloha = "Aloha!";
    let word = word_aloha.to_string();
    let word_ref = &word;
    
    println!("{}", word_aloha);
    println!("{}", word);
    println!("{}", word_ref);
}
```

`&word`: Bir `String`'e referans alındığında Rust bunu otomatik olarak `&str`'e dönüştürebilir (deref coercion).

---

## 5) String Karşılaştırma

```rust
fn main() {
    let word_aloha = "Aloha!";
    println!("Words is equal, {}", word_aloha.to_lowercase() == "aloha!");
}
```

`to_lowercase()` yeni bir `String` döndürür. Rust, `String` ile `&str` karşılaştırmasını destekler.

---

## 6) Unicode Desteği

Rust, Unicode karakterleri `\u{XXXX}` formatında destekler.

```rust
fn main() {
    let konnichiwa = "\u{3053}\u{3093}\u{306B}\u{3061}\u{306F}";
    println!("{}", konnichiwa); // こんにちは
}
```

---

## Özet

| Tür | Özellik |
|-----|---------|
| `String` | Heap'te saklanır, sahipliği vardır, değiştirilebilir |
| `&str` | String slice, mevcut veriye referans, boyutu sabit |

> Best Practice: Fonksiyon parametrelerinde `String` yerine `&str` kullanmak, hem `String` hem de `&str` değerlerin kabul edilmesini sağlar.
