# Rust'ta Kontrol Akışı (Control Flow)

Bu dosya, Rust'ta **if/else if**, **loop**, **while**, **for** döngülerini ve fonksiyonlarla birlikte kontrol akışının nasıl yönetildiğini gösterir.

---

## 1) Rastgele Sayı Üretimi ve `if / else if`

Rust'ta koşul ifadeleri `if`, `else if` ve `else` blokları ile oluşturulur. Koşullar **mutlaka `bool` tipinde** olmalıdır; C/C++'taki gibi tam sayılar örtük olarak `bool`'a dönüştürülmez.

Aşağıdaki örnekte `rand` crate'i kullanılarak 1 ile 999 arasında rastgele bir sayı üretilir ve bu sayının çift mi yoksa 3'e bölünebilir mi olduğu kontrol edilir.

```rust
use rand::Rng;

fn main() {
    let mut rnd = rand::thread_rng();
    let some_number = rnd.gen_range(1..1_000);

    println!("{}", some_number);

    if some_number % 2 == 0 {
        println!("{} is even.", some_number);
    } else if some_number % 3 == 0 {
        println!("{} is odd", some_number);
    }
}
```

- `rand::thread_rng()` mevcut thread'e bağlı bir rastgele sayı üreteci (RNG) döndürür. `mut` olarak tanımlanmalıdır çünkü her `gen_range` çağrısı üretecin iç durumunu değiştirir.
- `gen_range(1..1_000)` ifadesinde `1..1_000` bir **Range** literalidir. Üst sınır dahil değildir, yani 1 ile 999 arasında değer üretir.
- `1_000` yazımı, büyük sayılarda okunabilirliği artırmak için kullanılan sayısal ayırıcıdır (underscore separator). Derleyici bunu `1000` olarak işler.

> **Uyarı:** Bu koşul yapısında bir mantık boşluğu vardır. Sayı tek olup 3'e bölünemiyorsa (örneğin 7) hiçbir dalga girmez ve herhangi bir çıktı üretilmez. Ayrıca `else if` dalındaki mesaj "is odd" yazsa da aslında kontrol edilen şey 3'e bölünebilirliktir, teklik değildir.

### ✨ İdiomatic Rust: `match` ile Yeniden Yazım

`if/else if` zincirleri uzadığında `match` ifadesi daha okunabilir bir alternatif sunar. Ayrıca `else` dalı eklenerek tüm durumlar kapsanabilir.

```rust
match (some_number % 2, some_number % 3) {
    (0, _) => println!("{} is even.", some_number),
    (_, 0) => println!("{} is divisible by 3 and odd.", some_number),
    _      => println!("{} is odd and not divisible by 3.", some_number),
}
```

Bu yaklaşım, tüm olası durumları kapsadığı için `match`'in **exhaustiveness** (kapsamlılık) garantisinden yararlanır.

---

## 2) `loop` ile Sonsuz Döngü

`loop` bloğu koşulsuz bir sonsuz döngü oluşturur. Döngüden çıkmak için `break`, mevcut iterasyonu atlayıp bir sonrakine geçmek için `continue` kullanılır.

```rust
loop {
    let number = rnd.gen_range(1..101);
    println!("{}", number);
    if number % 23 == 0 {
        println!("I have got you {}", number);
        break;
    } else {
        continue;
    }
}
```

- Döngü her turda 1–100 arasında rastgele bir sayı üretir.
- Sayı 23'e tam bölünüyorsa `break` ile döngüden çıkılır.
- Bölünmüyorsa `continue` ile bir sonraki iterasyona geçilir.

> **Not:** Buradaki `else { continue; }` bloğu aslında gereksizdir. `break` çalışmadığı sürece döngü zaten bir sonraki iterasyona geçer. Açıkça `continue` yazmak okunabilirlik tercihidir, ancak idiomatic Rust'ta bu durumda genellikle yazılmaz.

### ⚡ Performans Notu

`loop` + `break` kombinasyonu, Rust'ta en düşük seviyeli döngü yapısıdır. Derleyici bu kalıbı çok verimli biçimde optimize eder. Koşulun döngü gövdesinin ortasında veya sonunda değerlendirilmesi gereken durumlarda `loop`, `while`'a göre daha doğal bir tercihtir.

### 🔄 Alternatif Yaklaşım: `loop` ile Değer Döndürme

Rust'ta `loop` bir ifade (expression) olduğundan, `break` ile bir değer döndürebilir:

```rust
let found = loop {
    let number = rnd.gen_range(1..101);
    if number % 23 == 0 {
        break number; // Değeri döndür
    }
};
println!("I have got you {}", found);
```

Bu yaklaşım, bulunan değeri döngü dışında kullanmanız gerektiğinde hem daha temiz hem de daha güvenlidir.

---

## 3) `while` Döngüsü

`while` döngüsü, verilen koşul `true` olduğu sürece çalışır. Aşağıdaki örnekte en fazla 100 deneme içinde 23'e bölünebilen bir sayı aranır.

```rust
let mut try_count = 0;
while try_count < 100 {
    let number = rnd.gen_range(1..101);
    if number % 23 == 0 {
        println!("I found the number {} in {} try", number, try_count);
        break;
    }
    try_count += 1;
}
```

- `try_count` mutable bir sayaç olarak tanımlanır ve her turda artırılır.
- 23'e bölünebilen bir sayı bulunduğunda `break` ile döngüden erken çıkılır.
- Koşul sağlanmadan 100 denemeye ulaşılırsa döngü doğal olarak sona erer.

> **Uyarı:** `break` çalışırsa `try_count` o turda artırılmaz, bu nedenle yazdırılan deneme sayısı gerçek deneme sayısından 1 eksik olabilir. İlk turda (try_count = 0) bulunursa "0 try" yazdırılır. Sayacı `break`'ten önce artırmak veya 1'den başlatmak bu durumu düzeltir.

### ✨ İdiomatic Rust: Iterator Kullanımı

Aynı mantık, fonksiyonel stil ile de ifade edilebilir:

```rust
let result = (0..100).find(|_| {
    let number = rnd.gen_range(1..101);
    number % 23 == 0
});

match result {
    Some(try_index) => println!("Found in {} tries", try_index + 1),
    None => println!("Not found in 100 tries"),
}
```

`find` metodu ilk eşleşen elemanı `Option<T>` olarak döndürür. Bu yaklaşım, hem başarılı hem de başarısız durumu açıkça ele alır.

---

## 4) `for` Döngüsü ve `enumerate`

`for` döngüsü, Rust'ta bir iterator üzerinde gezinmenin en yaygın yoludur. Aşağıdaki örnekte bir fonksiyondan dönen `Vec<i32>` üzerinde hem indeks hem de değer ile iterasyon yapılır.

```rust
let data = get_random_numbers(10);
for (index, value) in data.iter().enumerate() {
    println!("{index}\t {value}");
}
```

- `data.iter()` vektörün elemanlarına **referans** (`&i32`) veren bir iterator oluşturur. Sahiplik (ownership) vektörde kalır.
- `.enumerate()` her elemana bir indeks ekler ve `(usize, &i32)` tuple'ları üretir.
- `(index, value)` destructuring ile tuple, bileşenlerine ayrılır.

> **Not:** `println!` içindeki `{index}` ve `{value}` formatı, Rust 1.58+ ile gelen **captured identifier** özelliğidir. Daha eski sürümlerde `println!("{}\t {}", index, value)` kullanılması gerekir.

---

## 5) `Vec<i32>` Döndüren Fonksiyon

`get_random_numbers` fonksiyonu, belirtilen üst limite kadar tekrarsız rastgele sayılar üreterek bir `Vec<i32>` döndürür.

```rust
fn get_random_numbers(max_limit: u8) -> Vec<i32> {
    let mut rnd = rand::thread_rng();
    let mut numbers = Vec::new();
    for _ in 0..max_limit {
        let n = rnd.gen_range(1..101);
        if numbers.contains(&n) {
            continue;
        }
        numbers.push(n);
    }
    numbers
}
```

- Parametre tipi `u8` olduğundan, en fazla 255 adet sayı istenebilir.
- `Vec::new()` boş bir vektör oluşturur. Rust, `push` çağrılarından tip çıkarımı (type inference) ile `Vec<i32>` olduğunu anlar.
- `numbers.contains(&n)` ile mevcut elemanlar arasında arama yapılır. Eleman zaten varsa `continue` ile o iterasyon atlanır.
- Son satırda `numbers` noktalı virgül olmadan yazılmıştır; bu Rust'ta **expression return** demektir. `return numbers;` ile eşdeğerdir.

> **Uyarı:** Bu fonksiyonda bir tasarım sorunu mevcuttur. Döngü tam olarak `max_limit` kez çalışır, ancak `continue` nedeniyle tekrar eden sayılar atlandığında vektöre eleman eklenmez. Sonuçta döndürülen vektör, istenen sayıdan **daha az** elemana sahip olabilir. Örneğin `max_limit = 10` verildiğinde, tekrarlar nedeniyle 8 veya 9 eleman dönebilir.

### 🔄 Alternatif Yaklaşım: `HashSet` ile Tekrarsız Sayı Üretimi

`Vec::contains()` her çağrıda O(n) doğrusal arama yapar. `HashSet` kullanmak hem doğruluk hem de performans açısından daha uygundur:

```rust
use std::collections::HashSet;

fn get_random_numbers(count: u8) -> Vec<i32> {
    let mut rnd = rand::thread_rng();
    let mut set = HashSet::new();
    while set.len() < count as usize {
        set.insert(rnd.gen_range(1..101));
    }
    set.into_iter().collect()
}
```

- `while` döngüsü, istenilen sayıya ulaşılana kadar çalıştığı için her zaman tam sayıda eleman döndürür.
- `HashSet::insert` zaten mevcut olan elemanları otomatik olarak reddeder; ayrıca `contains` kontrolü gerekmez.
- `HashSet` üzerinde arama O(1) olduğundan, büyük veri kümelerinde belirgin performans farkı yaratır.

---

## 6) Çoklu Koşul Dalları: Sınav Puanı Kontrolü

`check_exam_score` fonksiyonu, verilen puana göre farklı mesajlar yazdırır.

```rust
fn check_exam_score(point: i32) {
    if point == 0 {
        println!("Blank paper! Fails");
    } else if point > 70 {
        println!("{point} is enough for pass.");
    } else if point < 50 {
        println!("{point} is not enough for pass.");
    } else if point > 50 && point < 70 {
        println!("{point} is greater than 50 but less than 70. Come in September!");
    }
}
```

- Fonksiyon `i32` tipinde bir parametre alır, ancak negatif değerler için herhangi bir kontrol yoktur.
- `rnd.gen_range(0..=100)` ile çağrılır; burada `..=` **inclusive range** operatörüdür, yani 0 ve 100 dahildir.

> **Uyarı:** Bu koşul yapısında **iki kenar değer** ele alınmamıştır:
> - `point == 50` → Hiçbir dala girmez (çıktı üretilmez).
> - `point == 70` → Hiçbir dala girmez (çıktı üretilmez).
>
> Bu tür boşluklar, `match` ile **range pattern** kullanılarak önlenebilir.

### ✨ İdiomatic Rust: `match` ile Range Pattern

```rust
fn check_exam_score(point: i32) {
    match point {
        0         => println!("Blank paper! Fails"),
        1..=49    => println!("{point} is not enough for pass."),
        50..=70   => println!("{point} is greater than 50 but less than 70. Come in September!"),
        71..=100  => println!("{point} is enough for pass."),
        _         => println!("{point} is out of valid range."),
    }
}
```

- `match` ifadesi tüm olası değerleri kapsamak zorundadır (**exhaustiveness**). `_` joker deseni bu garantiyi sağlar.
- Range pattern'lar (`1..=49`) sayesinde kenar değerler (50, 70) bir dala mutlaka girer.
- Negatif veya 100'den büyük değerler `_` dalı ile yakalanır; böylece geçersiz girdi sessizce görmezden gelinmez.

Bu yaklaşım hem daha güvenli hem de daha okunabilirdir.

---
