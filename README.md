# Limus
A modern langage and optimizing compiler.

The name means mud, slime or muck in Latin and refer to [Physarum polycephalum](https://fr.wikipedia.org/wiki/Physarum_polycephalum) a slime mold able to solve optimization problems.

## Langage and Compiler
### Ideas
- The assume keyword allow the programmer to express it's own knowledge
```rust
fn main(Vec<str> args) {
    assume!(args.len() != 0); // The first argument is always the program name
    println!("{}", args[0]); // This will not be bound checked !
}
```
- Access the parity, carry and overflow flags without assembly
```rust
use std::rand;

fn main() {
    let rng: Rng<u64> = Rng::new();
    let sum = rng.gen();
    assume!(propagate_carry(sum));
    sum += rng.gen();
    sum += rng.gen();
    sum += rng.gen(); // sum is now an u256
    println!("{}{}", carry(sum) ? "c" : "", sum);
}
```
- Native support for fixed-point
```rust
fn main() {
    let num: q0d8 = 0.5;
    println!("{:#b}", num); // "0b00001111"
}
```
- What you use is what you get (WYUIWYG)
```rust
fn main(Vec<str> args) {
    println!("{:?}", args);
}

// This code doesn't emit the GetCommandLineW function
fn main() {
    println!("Hello, world!");
}
```
```rust
fn main() {
    let array = Vec::new();
    array.push(0);
    array.push(1);
    array.pop();
    array.append(1); // array is now a ring buffer
    array.remove();
    array[10] = 2; // skip a range in the ring buffer
    array[10000] = 3; // array now contains a ring buffer and a linked list
}
```
