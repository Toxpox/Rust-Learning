'''Rust
fn main() {
    println!("System Programming with Rust.");
}
'''
### Mutable/Immutable
**Mutable** sonradan değiştirilebilir iken **Immutable** sonradan değiştirilemez değişkenlerdir. Default olarak Immutable olarak gelir. Değişkenden önce **mut** yazıldığında mutable hale getirebiliriz.

'''Rust
fn main() {
    let points = 90; // Immutable
    println!("Point: {}", points);

    let mut points = 80; // Mutable
    println!("Point: {}", points);
    last_score = 60;
    println!("Point: {}", points);
}
'''

