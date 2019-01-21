# Limus
A modern langage and optimizing compiler.

The name means mud, slime or muck in Latin and refer to [Physarum polycephalum](https://fr.wikipedia.org/wiki/Physarum_polycephalum) a slime mold able to solve optimization problems.

## Langage
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

# Compiler
## Lex, Parse and Syntax Tree
### Parsing Expression Grammar
#### Introduction
Parsing expression grammar, or PEG, is a type of analytic formal grammar syntactically similar to [Extended Backus-Naur form](https://en.wikipedia.org/wiki/Extended_Backus%E2%80%93Naur_form) and [Regular Expression](https://en.wikipedia.org/wiki/Regular_expression). It was introduced by Bryan Ford in 2004 and is closely related to the family of [top-down parsing](https://en.wikipedia.org/wiki/Top-down_parsing) languages introduced in the early 1970s.

#### Syntax
|Operators|Syntax|Semantic|
|---|---|---|
|Any|&epsilon;|Match anything|
|Sequence|e<sub>1</sub> e<sub>2</sub>|Match e<sub>1</sub> then e<sub>2</sub>|
|Ordered choice|e<sub>1</sub> / e<sub>2</sub>|Match e<sub>1</sub> first or e<sub>2</sub> then|
|Zero-or-more|e* &hArr; N &larr; e N / &epsilon;|Match e zero or more time(s)|
|One-or-more|e+ &hArr; ee*|Match e once or more time(s)|
|Optional|e? &hArr; e / &epsilon;|Match e if possible|
|And-predicate|&e|Look ahead if e match|
|Not-predicate|!e|Look ahead if e doesn't match|
|If-then-else<sup>1</sup>|e ? e<sub>1</sub> : e<sub>2</sub> &hArr; (e e<sub>1</sub>) / e<sub>2</sub>|Match e<sub>1</sub> if e succeed else e<sub>2</sub>|
|Redirect|e > on_e|Call on_e when e match|

1. The paper below describe a more general operator (also defined on recursion) that help to reduce the backtracking activity but is harder to write with.  
Kota Mizushima, Atusi Maeda, and Yoshinori Yamaguchi. 2010. Packrat parsers can handle practical grammars in mostly constant space. In Proceedings of the 9th ACM SIGPLAN-SIGSOFT workshop on Program analysis for software tools and engineering (PASTE '10). ACM, New York, NY, USA, 29-36. https://doi.org/10.1145/1806672.1806679

#### Example
Grammar that recognize the basic four operations to non-negative integers with precedence handling.  
Expr &larr; Product (('+' / '-') Product)* > expr  
Product &larr; Value (('\*' / '/') Value)* > product  
Value &larr; Num / '(' Expr ')'  
Num &larr; [0-9]+ > num

```rust
fn expr(left: u64, right: Option<Vec<(char, u64)>>) -> u64 {
    if let Some(right_inner) = right {
        for tuple in right_inner {
            let value = tuple.1;
            match tuple.0 {
                '+' => left += value,
                '-' => left -= value,
                _ => unreachable!(),
            }
        }
    }
    left
}

fn product(left: u64, right: Option<Vec<(char, u64)>>) -> u64 {
    if let Some(right_inner) = right {
        for tuple in right_inner {
            let value = tuple.1;
            match tuple.0 {
                '*' => left *= value,
                '/' => left /= value,
                _ => unreachable!(),
            }
        }
    }
    left
}

fn num(Vec<char> arg) -> u64 {
    arg.into_iter().collect::<String>().parse().unwrap()
}
```

#### Advantages
