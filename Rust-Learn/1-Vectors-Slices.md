# Rust: Vector ve Slice Yapısı

Bu doküman, Rust'ın belleğin Heap bölgesinde tutulan dinamik dizileri (`Vec<T>`) ve bellekteki bir veri bloğuna maliyetsiz erişim sağlayan Slice (`&[T]`) kavramlarını teknik detaylarıyla ele alır.

## Temel Farklar

*   **Array (`[T; N]`):** Boyutu derleme zamanında (compile-time) bilinir, Stack'te tutulur ve boyutu sabittir.
*   **Vector (`Vec<T>`):** Boyutu çalışma zamanında (runtime) değişebilir, Heap'te tutulur ve dinamiktir.

---

## 1. Vector Tanımlama ve Bellek Yönetimi (Memory Allocation)

`vec!` makrosu, arka planda bir `Vec` yapısı oluşturur ve elemanları Heap'e yerleştirir. `push` işlemi, vektörün kapasitesi (capacity) dolduğunda yeni ve daha büyük bir bellek alanı tahsis edip (allocate) verileri oraya taşır (reallocation).

```rust
fn main() {
    // `vec!` makrosu ile ilklendirme. Tür çıkarımı (Type Inference) f64 olarak yapılır.
    let mut metrics = vec![150.50, 42.0, 75.90, 10.0];

    // Heap'te yer ayrılır ve eleman sona eklenir.
    // `mut` anahtar kelimesi, vektörün değiştirilebilir (mutable) olduğunu belirtir.
    metrics.push(200.25);

    println!("Tüm Metrikler: {:?}", metrics);
    
    // Indexing işlemi: Sınırların dışına çıkılırsa (out of bounds) program çalışma zamanında "panic" üretir.
    println!("İlk metrik: {}", metrics[0]);
}
```

> **Not:** Reallocation işlemi maliyetli olabilir. Eğer tahmini eleman sayısı biliniyorsa, `Vec::with_capacity(n)` kullanmak performans artışı sağlar.

---

## 2. Stack Davranışı (LIFO) ve Option Türü

Vector veri yapısı, sonuna ekleme yapıp sonundan eleman çıkarabildiğimiz için bir Stack (LIFO - Last In First Out) gibi kullanılabilir. `pop()` metodu, güvenli bellek yönetimi için `Option<T>` (`Some(T)` veya `None`) döndürür. Bu, boş bir listeden eleman çekilmeye çalışıldığında programın çökmesini engeller.

```rust
fn main() {
    let mut transactions = vec![150.50, 42.0, 75.90, 10.0];
    
    // Son elemanı çıkarır ve döner. Liste boşsa `None` döner.
    let last_tx = transactions.pop(); 
    
    println!("Geri alınan işlem: {:?}", last_tx); // Çıktı: Some(10.0)
    
    // `{:#?}` format belirleyicisi, çıktıyı "pretty print" formatında daha okunabilir hale getirir.
    println!("Güncel stack durumu: {:#?}", transactions);
}
```

---

## 3. `Vec::new()` ve Kapasite Yönetimi

`Vec::new()` ile boş bir vektör oluşturulduğunda genellikle bellek tahsis edilmez. Eleman eklendikçe `len` (eleman sayısı) artar, `capacity` (ayrılan yer) ise performansı korumak adına genellikle katlanarak büyür.

```rust
fn main() {
    let mut users = Vec::new();
    
    // String heap-allocated bir türdür. Vector de verisini Heap'te tutar.
    // Bu durumda Vector, Heap üzerindeki String verilerinin pointer'larını (yine Heap'te) tutar.
    users.push(String::from("admin"));
    users.push(String::from("user_1"));
    users.push(String::from("user_2"));
    users.push(String::from("guest"));

    // In-place (yerinde) ters çevirme işlemi. Yeni bellek tahsisi yapmaz.
    users.reverse();

    println!("Allocated Capacity: {:?}", users.capacity());
    println!("Active Length: {:?}", users.len());
    
    println!("User List: \n {:#?}", users);
}
```

---

## 4. Iterator ve `collect` Kullanımı

Bir `Range` veya `Iterator` üzerindeki veriler `collect()` metodu ile bir veri yapısına dönüştürülebilir. Derleyici, hedef türü (Array, Vector, HashMap vb.) çıkaramayabileceği için açıkça Tip Belirtimi (Type Annotation, örn: `: Vec<u8>`) yapılması gereklidir.

```rust
fn main() {
    // 1..=20 Range'ini u8 türünde bir vektöre dönüştürür.
    // u8 kullanımı bellekte eleman başına sadece 1 byte yer kaplar.
    let ticket_ids: Vec<u8> = (1..=20).collect();
    
    println!("Ticket IDs: {:?}", ticket_ids);
}
```

---

## 5. Slice (Dilim) ve Borrowing

Slice (`&[T]`), bellekteki bir dizi veriye (Array, Vector vb.) yapılan bir referanstır.

*   **Zero-Cost Abstraction:** Veriyi kopyalamaz (cloning yapmaz), sadece mevcut veriye işaret eder.
*   **Fat Pointer:** Slice, bellekteki verinin başlangıç adresi (pointer) ve uzunluk (length) bilgisinden oluşur.
*   **Kullanım Alanı:** Verinin sadece belirli bir kısmına salt okunur (read-only) erişim gerektiğinde idealdir.

```rust
fn main() {
    // 0-50 arası veri seti oluşturuluyor.
    let raw_data: Vec<u8> = (0..50).collect();

    // `&` sembolü Borrowing (ödünç alma) işlemini temsil eder.
    // Sadece 5 ile 15 (dahil) arasındaki veriye referans veriyoruz (slicing).
    // `data_window` değişkeni stack'te duran bir "Fat Pointer"dır.
    let data_window = &raw_data[5..=15]; 

    println!("Analiz Penceresi: {:#?}", data_window);
}
```

> **Best Practice:** Fonksiyon parametresi olarak bir dizi veri alacaksanız ve sahipliği (ownership) devralmanız gerekmiyorsa, `&Vec<T>` yerine `&[T]` (slice) kullanın. Bu sayede fonksiyon hem `Vector` hem de `Array` referansları ile çalışabilir ve daha esnek olur.
