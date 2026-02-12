# Rust'ta Lifetimes (Yaşam Ömürleri)

Bu dosya, Rust'ın bellek güvenliğinin temel taşlarından biri olan **lifetime** kavramını, **dangling reference** sorununu, **lifetime annotation** kullanımını ve **`'static` lifetime** bildirimini ele alır.

---

## 1) Dangling Reference Sorunu

Rust'ta her referansın bir yaşam ömrü (lifetime) vardır. Derleyici, bir referansın işaret ettiği verinin hâlâ geçerli olup olmadığını derleme zamanında kontrol eder. Referans, veriden daha uzun yaşarsa **dangling reference** oluşur ve Rust bunu engeller.

```rust
fn main() {
    // let result;
    // {
    //     let temperature = 36.6_f64;
    //     result = &temperature;
    // } // temperature burada drop edilir
    // println!("{}", result); // HATA: dangling reference

    // Doğru kullanım: referans ile veri aynı scope'da yaşamalıdır
    let temperature = 36.6_f64;
    let result = &temperature;
    println!("Temperature: {}", result);
}
```

Yorum satırlarındaki kod, `temperature` değişkeninin iç scope sonunda drop edilmesine rağmen `result` referansının dış scope'da kullanılmaya çalışılmasını gösterir. Rust derleyicisi bu durumu `E0597: does not live long enough` hatası ile yakalar.

> **Not:** Bu kontrol tamamen derleme zamanında gerçekleşir. Çalışma zamanında herhangi bir maliyet yoktur. Bu, Rust'ın "zero-cost abstractions" felsefesinin bir parçasıdır.

---

## 2) Fonksiyonlarda Lifetime Annotation

Bir fonksiyon birden fazla referans alıp referans döndürdüğünde, derleyici dönen referansın hangi girdiye bağlı olduğunu bilemeyebilir. Bu durumda lifetime annotation (`'a`) ile ilişki açıkça belirtilmelidir.

```rust
fn longer_text<'a>(first: &'a str, second: &'a str) -> &'a str {
    if first.len() >= second.len() {
        first
    } else {
        second
    }
}

fn main() {
    let greeting = String::from("Merhaba Dünya");
    let result;

    {
        let farewell = String::from("Güle güle");
        result = longer_text(&greeting, &farewell);
        println!("Longer: {}", result);
    }
    // `result` burada kullanılamaz çünkü `farewell` drop edilmiştir
    // ve lifetime annotation her iki referansın da en az `'a` kadar yaşamasını gerektirir.
}
```

`'a` annotation'ı, dönen referansın her iki girdi referansının **en kısa yaşam ömrüne** bağlı olduğunu bildirir. Bu sayede derleyici, dönen değerin geçersiz bir bellek bölgesini işaret etmesini engeller.

### 🔄 Alternatif Yaklaşım

Lifetime karmaşıklığından kaçınmak istiyorsanız, referans döndürmek yerine **sahipliği taşıyan** bir değer döndürebilirsiniz:

```rust
fn longer_text_owned(first: &str, second: &str) -> String {
    if first.len() >= second.len() {
        first.to_string()
    } else {
        second.to_string()
    }
}
```

Bu yaklaşımda yeni bir `String` oluşturulduğu için heap allocation maliyeti vardır, ancak lifetime annotation'a gerek kalmaz. Performans kritik değilse bu yöntem daha basittir.

---

## 3) Struct'larda Lifetime Annotation

Struct içinde referans (`&str`, `&T` gibi) barındırılacaksa, lifetime annotation zorunludur. Bu bildirim, struct'ın referans ettiği verinin en az struct kadar yaşamasını garanti eder.

```rust
struct Article<'a> {
    title: &'a str,
    author: &'a str,
    word_count: usize,
}

impl<'a> Article<'a> {
    fn summary(&self) -> String {
        format!("\"{}\" by {} ({} words)", self.title, self.author, self.word_count)
    }
}

fn main() {
    let title = String::from("Rust ve Bellek Güvenliği");
    let author = String::from("Ferris");

    let article = Article {
        title: &title,
        author: &author,
        word_count: 2500,
    };

    println!("{}", article.summary());
}
```

`Article<'a>` tanımındaki `'a`, `title` ve `author` referanslarının `Article` instance'ından daha uzun yaşaması gerektiğini ifade eder. Lifetime annotation olmadan bu struct tanımı derlenmez: `error[E0106]: missing lifetime specifier`.

### ⚡ Performans Notu

Referans (`&str`) kullanmak, `String` yerine tercih edildiğinde heap allocation yapılmaz. Struct yalnızca mevcut veriye bir pointer tutar. Bu, özellikle çok sayıda struct oluşturulduğu senaryolarda bellek ve performans açısından avantaj sağlar.

---

## 4) `'static` Lifetime

`'static` lifetime, referansın programın tüm çalışma süresi boyunca geçerli olacağını bildirir. String literal'ler (`"hello"` gibi) otomatik olarak `'static` lifetime'a sahiptir çünkü binary'nin içinde gömülü olarak tutulurlar.

```rust
struct Config<'a> {
    app_name: &'a str,
    api_endpoint: &'a str,
    max_retries: u32,
}

fn default_config() -> Config<'static> {
    Config {
        app_name: "WeatherStation",
        api_endpoint: "https://api.weather.local/v2",
        max_retries: 3,
    }
}

fn main() {
    let config = default_config();

    println!("App: {}", config.app_name);
    println!("Endpoint: {}", config.api_endpoint);
    println!("Max Retries: {}", config.max_retries);
}
```

`default_config` fonksiyonu `Config<'static>` döner çünkü tüm referanslar string literal'dir ve `'static` yaşam ömrüne sahiptir.

> **Uyarı:** `'static` annotation'ını gereksiz yere kullanmaktan kaçınılmalıdır. Bir referansın programın sonuna kadar yaşamasını garanti etmek çoğu durumda gereksizdir ve esnekliği kısıtlar. String literal gibi doğal olarak `'static` olan durumlar dışında, mümkün mertebe nesnelerin ömürlerini gerektiği kadar kısa tutmak bellek optimizasyonu açısından daha doğrudur.

---

## 5) Lifetime Elision Kuralları

Rust derleyicisi, belirli kalıplarda lifetime annotation'ı otomatik olarak çıkarır. Buna **lifetime elision** denir. Üç temel kural uygulanır:

1. Her referans parametresine ayrı bir lifetime atanır.
2. Tek bir input lifetime varsa, output lifetime da aynı kabul edilir.
3. Metotlarda `&self` veya `&mut self` varsa, output lifetime `self` ile aynı kabul edilir.

```rust
struct Document {
    content: String,
}

impl Document {
    // Lifetime elision sayesinde annotation gerekmez
    // Derleyici bunu fn title<'a>(&'a self) -> &'a str olarak yorumlar
    fn title(&self) -> &str {
        let end = self.content.find('\n').unwrap_or(self.content.len());
        &self.content[..end]
    }
}

fn first_word(text: &str) -> &str {
    // Tek input referans olduğu için output lifetime otomatik çıkarılır
    let end = text.find(' ').unwrap_or(text.len());
    &text[..end]
}

fn main() {
    let doc = Document {
        content: String::from("Rust Programlama\nGüvenli ve hızlı sistem programlama dili"),
    };

    println!("Title: {}", doc.title());
    println!("First word: {}", first_word("Merhaba Rust"));
}
```

### ✨ Idiomatic Rust

Elision kuralları yeterli olduğunda lifetime annotation yazmamak idiomatik Rust'tır. Gereksiz annotation eklemek kodu karmaşıklaştırır. Derleyici annotation gerektiğinde `missing lifetime specifier` hatası vererek sizi bilgilendirir; bu noktaya kadar manuel annotation gerekmez.

---
