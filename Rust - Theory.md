# Rust - Theory

# Rust Foundations

## Сравнение с другими языками 

### C, C++

- быстрые
- нет централизованной системы сборки и управления зависимостями
- undefined behavior, memory unsafety (70% ошибок в ПО)

### Java, C#, Go

- имеют garbage collector, который иногда вызывается и замедляет работу программы/приложения
- не подходят для системного программирования

### System programming

- программы которые взаимодействуют с другими программами, а не с пользователем в первую очередь
- быстро, безопасно, стабильно (например из-за garbage collector стабильность падает)

### Rust

- быстрый
- есть много абстракций
- нет undefined behavior, memory unsafety
- общая система сборки и управления зависимостями
- много оптимизаций приводит к необходимости частого использования **unsafe**

### Пример проблемного C кода

```c
#include <assert.h>
#include <stdio.h>
#include <stdlib.h>

typedef struct {
    int *data;
    int length;
    int capacity;
} Vec;

Vec *vec_new() {
    Vec vec;
    vec.data = NULL;
    vec.length = 0;
    vec.capacity = 0;
    return &vec;
    // возврат указателя на стеке - провисший указатель (dangling pointer) !!!
}

void vec_push(Vec *vec, int n) {
    if (vec->length == vec->capacity) {
        int new_capacity = vec->capacity * 2; // +1 !!!
        int *new_data = (int *)malloc(new_capacity);
        assert(new_data != NULL);

        for (int i = 0; i < vec->length; ++i) {
            new_data[i] = vec->data[i];
        }

        vec->data = new_data;
        // не очищенная память!!!
        vec->capacity = new_capacity;
    }

    vec->data[vec->length] = n;
    ++vec->length;
}

void vec_free(Vec *vec) {
    free(vec);
    free(vec->data); // дважды очищенная память!!!
}

void main() {
    Vec *vec = vec_new();
    vec_push(vec, 107);

    int *n = &vec->data[0]; 
    vec_push(vec, 110); // реалокация памяти!!!
    printf("%d\n", *n);

    free(vec->data); 
    vec_free(vec); // дважды очищенная память!!!
}
```

---

## Версии Rust

| Edition           | Год  | Основные изменения       |
| ----------------- | ---- | ------------------------ |
| Rust 2015         | 2015 | Первая стабильная версия |
| Rust 2018         | 2018 | Async/await, модули, NLL |
| Rust 2021 (1.56+) | 2021 | Closures, panic, prelude |

---
## `Hello world!`

```rust
fn main() {
    println!("Hello, World!");  
    //   ~~~^ макрос (блокирует std::io чтоб один поток писал)
}
```

```bash
rustc main.rs
./main
```

---

## Базовые типы (Types)

### Integers

| Bits | Signed | Unsigned |
|------|--------|----------|
| 8    | i8     | u8       |
| 16   | i16    | u16      |
| 32   | i32    | u32      |
| 64   | i64    | u64      |
| 128  | i128   | u128     |
| arch | isize  | usize    |

`isize`/`usize` – как `size_t`

```rust
let idx: usize = 92;

// Литералы
let y = 92_000_000i64;
let hex_octal_bin = 0xffff_ffff + 0o777 + 0b1;
```

Типы в Rust изначально неизменяемые (`let` можно воспринимать как `const`), чтобы сделать изменяемый нужно использовать **mut**:

```rust
let mut idx: usize = 0x1022022;
```

В Rust есть auto, причем он смотрит на то как переменная используется в дальнейшем.

### Floats and bools

```rust
// Типы с плавающей точкой
let f1: f32 = 0.0;
let f2: f64 = 0.0;

// Булевы значения
let flag: bool = true;
```

`bool` может быть только `true` или `false`

### Arithmetic

```rust
// Основные операции: +, -, *, /, %

// / и % округляет до нуля
let (x, y) = (15, -15);
let (a1, b1) = (x / -4, x % 4);
let (a2, b2) = (y / 4, y % 4);

println!("({a1} {b1}) and ({a2} {b2})");
// (-3 3) and (-3 -3)
```

**Нет неявного каста типов:**

```rust
let x: u16 = 1;

// let y: u32 = x; // error: mismatched types

let y: u32 = x as u32; // заполнение лидирующих бит нулями
let z: u16 = y as u16;
let y: u32 = x.into();

let to_usize = 92u64 as usize;
let from_usize = 92usize as u64;
```

Каст не транзитивен:

```rust
e as U1 as U2  
// не то же самое что  
e as U2
```

### Работа с переполнением

```rust
fn main() {
    let x = i32::MAX;
    let y = x + 1;
    println!("{}", y);
}
```

```bash
$ cargo run
thread 'main' panicked at 'attempt to add with overflow', main.rs:3:13

$ cargo run --release  
-2147483648
```

**Методы для работы с переполнением:**

```rust
let x = i32::MAX;

let y = x.wrapping_add(1);
assert_eq!(y, i32::MIN);

let y = x.saturating_add(1);
assert_eq!(y, i32::MAX);

let (y, overflowed) = x.overflowing_add(1);
assert!(overflowed);
assert_eq!(y, i32::MIN);

match x.checked_add(1) {
    Some(y) => unreachable!(),
    None => println!("overflowed"),
}
```

### Числа с плавающей точкой

```rust
let y = 0.0f32; // Литерал f32
let x = 0.0; // default value, будет f64

// Точка обязательна
// let z: f32 = 0; // error: expected f32, found integer
let z = 0.0f32;

let f: f32 = 0.0;
// 0.0 по умолчанию f64, но зная, что f: f32, компилятор автоматически приведет 0.0 к типу f32

let not_a_number = std::f32::NAN;
let inf = std::f32::INFINITY;

// Методы для работы с f32/f64
8.5f32.ceil().sin().round().sqrt(); // и т.д.
```

### Tuple

В памяти расположены последовательно (рядом два элемента).
Не влияют на производительность, на tuple и на первый элемент tuple – один указатель.

```rust
let pair: (f32, i32) = (0.0, 92);
let (x, y) = pair;

// То же самое что и:
// Note the shadowing! - старая переменная скрывается и используется новая
let x = pair.0;
let y = pair.1;

let void_result = println!("hello"); // unit () - пустой тип
assert_eq!(void_result, ());

let trailing_comma = (
    "Archibald",
    "Buttle",
);

// Кортеж из нуля элементов
let x = ();
let y = {};
assert_eq!(x, y); // OK

// Кортеж из одного элемента
let x = (42,);
```

### Array

Размер массива постоянный и известен на этапе компиляции:

```rust
let xs: [u8; 3] = [1, 2, 3];

assert_eq!(xs[0], 1); // index -- usize
assert_eq!(xs.len(), 3); // len() -- usize

let mut buf = [0u8; 1024];
```

### ❓String и &str❓ #Q

**String**
- Владеющий тип: хранит данные в куче (heap) с возможностью изменять, расширять и уменьшать размер.
- Представляет собой структуру с тремя полями: указатель на данные, длина и ёмкость. Он владеет буфером, и при выходе из зоны видимости автоматически освобождает память
```rust
  struct String {
    ptr: *mut u8,  // указатель на heap-данные
    len: usize,    // длина строки (байты)
    cap: usize,    // ёмкость буфера (байты)
}
```

**&str**
- «Срез» строки — неизменяемое представление части UTF-8 строки.
- Это fat-pointer: содержит указатель на начало и длину.
- Не владеет данными — просто заимствует их на время жизни, управляемой borrow checker’ом

| Характеристика | `String`                            | `&str`                             |
| -------------- | ----------------------------------- | ---------------------------------- |
| Владелец       | Да                                  | Нет (borrow)                       |
| Изменяемость   | Да (мутабельный, можно append/push) | Нет (только чтение)                |
| Память         | Аллоцируется в куче                 | Не владеет памятью                 |
| Размер         | Динамический (length + capacity)    | Фиксированный при создании         |
| Использование  | Структуры, возвращаемые функции     | Передача параметров, чтение, срезы |
> `str` — это примитивный тип, представляющий собой последовательность байтов в UTF‑8.
>
>Он **не имеет фиксированного размера** — это _DST_ (Dynamically Sized Type), поэтому **нельзя** его хранить напрямую в переменной или передавать без указателя. Чтобы использовать `str`, нужно оборачивать его в указатель (`&str`, `Box<str>`, `Rc<str>` и т.п.)

---
## Переменные и Shadowing

```rust
let x = 10;
for i in 0..5 {
    if x == 10 {
        println!("{x}");
        let x = 12; // затирается
    }
}
// output: 10 10 10 10 10

let x = 10;
for i in 0..5 {
    if x == 10 {
        let x = 12;
        println!("{i} {x}");
    }
}
// output: 0 12 1 12 2 12 3 12 4 12
```

---

## Functions

```rust
fn func1() {} // return nothing

fn func2() -> () {} // same

fn func3() -> i32 {
    0 // без return
}

fn func4(x: u32) -> u32 {
    return x; // с return
}

fn func5(x: u32, mut y: u64) -> u64 {
    y = x as u64 + 10;
    return y; // return y
}

fn func6(x: u32) -> u32 {
    x + 10 // return x + 10, если добавим ; то будет возвращаться unit
}
```

### ❓Почему `usize` копируется, а `String` перемещается❓ #Q
>  **usize реализует `Copy` трейт.**
> `usize` — это простой целочисленный тип фиксированного размера. Он находится полностью на стеке и имеет фиксированный размер, известный на этапе компиляции.

- Типы вроде `usize`, `u32`, `bool` реализуют трейт **`Copy`**.
- При передаче в функцию они **копируются по значению**, оригинал остаётся валидным.

>**String не реализует `Copy` трейт, реализует `Clone` с глубоким копированием**
>Состоит из (1) указатель на буфер в куче, (2) длины строки, (3) емкости. 
>
>В Rust по умолчанию при присваивании или передаче объекта, который не реализует `Copy`, происходит **move**.

Эти данные не могут быть просто скопированы побайтово, так как это приведёт к **двойному освобождению памяти**.

- Типы вроде `String`, `Vec<T>` не реализуют `Copy`.
- При передаче таких значений во функцию происходит **перемещение владения** (move).

Полную копию создает `Clone` трейт, который выделяет новое место в куче и с новым указателем.

---

## Expressions and Statements – Выражения и операторы

### Выражения (Expressions)

```rust
"Hello, world!"; // литералы  

let x = 5;
x;    // доступ к переменным

3 + 4; // арифметические операции

let y = f(10);   // вызовы функций

let z = { // блоки кода
    let a = 5;
    let b = 10;
    a + b // <- выражение
};

let number = if x > 5 { 10 } else { 20 }; // if выражение
```

### Операторы (Statements)

```rust
let x = 5; // объявления переменных
let x = 5 + 6;  // объявление и присвоение значений переменным

if x > 5 { // if операторы
    println!("x > 5");
} else {
    println!("x <= 5");
}

for i in 0..5 { // циклы
    println!("{i}");
}
```

---
## Conditions and loops: if, while, for, loop

### if

```rust
let mut x = 2;  
if x == 2 {
    x += 2;    
}
```

### while

```rust
while x > 0 {
    x -= 1;
    println!("{x}");
}
```

### loop

```rust
let mut counter = 0;

let result = loop {
    counter += 1;
    if counter == 10 {
        break counter * 2; // default break is break()
    }
};

assert_eq!(result, 20);
```

### Inner & outer loops

```rust
'outer: loop {
    println!("Entered the outer loop");
    
    'inner: for _ in 0..10 {
        println!("Entered the inner loop");
        
        // Break только inner loop
        // break;

        // Break outer loop
        break 'outer;
    }
    
    println!("Will never be reached");
}

println!("Exited the outer loop");
```

### for

```rust
for i in 0..10 {
    println!("{i}");
}

for i in 0..=10 {
    println!("{i}");
}

for i in [1, 2, 3, 4] {
    println!("{i}");
}

let vec = vec![1, 2, 3, 4];
for i in &vec {
    println!("{i}");
}
```

---
## Structures – структуры

```rust
struct Example {
    oper_count: usize,
    data: Vec<i32>, // должна быть запятая
}
```

Rust не дает гарантий о том как он расположит типы в памяти.

```rust
// эти структуры могут быть расположены по-разному в памяти  
struct A {
    x: Example,    
}

struct B {
    y: Example,    
}

impl Example {
    // Associated – ассоциированная (static в C++)
    pub fn new() -> Self { // Self – текущий тип, alias на слово Example
        // new – соглашение в Rust для имени конструктора  
        Self {
            oper_count: 0,
            data: Vec::new(),
        }
    }

    pub fn push(&mut self, x: i32) {  
        self.oper_count += 1;
        self.data.push(x);
    }

    pub fn oper_count(&self) -> usize {
        self.oper_count
    }

    pub fn eat_self(self) {
        println!("later");
    }
}
```

### Инициализация структуры

```rust
let mut x = Example {
    oper_count: 0,
    data: Vec::new(),
};

let y = Example::new();
x.push(10); // pub function
assert_eq!(x.oper_count(), 1);
```

> В Rust нет перегрузки методов

---

## Generics

```rust
struct Example<T> {
    oper_count: usize,
    data: Vec<T>,
}

impl<T> Example<T> {
    pub fn new() -> Self {
        Self {
            oper_count: 0,
            data: Vec::new(),
        }
    }

    pub fn push(&mut self, x: T) {
        self.oper_count += 1;
        self.data.push(x);
    }

    pub fn oper_count(&self) -> usize {
        self.oper_count
    }

    pub fn eat_self(self) {
        println!("later");
    }
}
```

---
## Enums

```rust
enum MyEnum {
    First,
    Second,
    Third, // запятая в конце !!!
}

let x = MyEnum::First;
let y: MyEnum = MyEnum::First;

enum OneMoreEnum<T> {
    Ein(i32),
    Zwei(u64, Example<T>),
}

let z = OneMoreEnum::Zwei(42, Example::<usize>::new());
```

### Important enums in std

```rust
// Используются для обработки ошибок
enum Option<T> {
    Some(T),
    None,
}

enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

---

## Match

```rust
let x = MyEnum::First;

match x {
    MyEnum::First => println!("First"),
    
    MyEnum::Second => {
        for i in 0..5 { 
            println!("{i}"); 
        }
        println!("Second");
    },
    
    _ => println!("Matched something"), // wildcard (_) матчит все
}
```

### Collect в контейнеры

```rust
// Rust может делать collect в контейнеры  
let mut vec: Vec<_> = (0..10).collect();
vec.push(42u64);

let _x = 10; // пометка что переменная дальше не используется
```

### Match с несколькими объектами

```rust
let x = OneMoreEnum::<i32>::Ein(2);
let y = MyEnum::First;

match (x, y) {
    (OneMoreEnum::Ein(x), MyEnum::First) => println!("Hi"),
    
    // Destructuring
    (OneMoreEnum::Zwei(a, _), _) => println!("{a}"),
    
    _ => println!("ooof"), // everything else
}
```

### Match с диапазонами и OR паттернами

```rust
let n = 13;

match n {
    1 => println!("match n"),
    
    1 | 3 | 13 | 22 => println!("or case"),
    // match завершает свою работу, остальное не проверяется
    
    13..=19 => println!("range"), // если n находится в этом диапазоне
    _ => println!("else"), // wildcard – любое значение кроме предыдущих
}
```

### Match с guards

```rust
let pair = (2, -2);

println!("Tell me about {:?}", pair);
match pair {
    (x, y) if x == y => println!("These are twins"),
    (x, y) if x + y == 0 => println!("Antimatter, kaboom!"), // выполнится и выйдет
    (x, _) if x % 2 == 1 => println!("The first one is odd"),
    _ => println!("No correlation..."),
}
```

### Match как expression

```rust
let boolean = 13;

let x = match boolean {
    13 if foo() => 0, // foo - функция которая возвращает bool
    13 => 1,
    _ => 2,
};
```

### Destructuring структур

```rust
struct Foo {
    x: (i32, i32),
    y: i32,
}

match foo {
    Foo { x: (1, b), y } => {
        println!("First of x is 1, b={}, y={}", b, y);
    },
    
    Foo { y: 2, x: i } => {
        println!("y is 2, i = {:?}", i); // {:?} – debug output
    },
    
    Foo { y, .. } => { // ignoring some variables:
        println!("y = {}, we don't care about x", y);
    },
    
    // error: pattern does not mention field `x`
    // Foo { y } => println!("y = {}", y),
}
```

### Binding values to names

```rust
fn age() -> u32 { 25 }

match age() {
    0 => println!("I haven't celebrated my birthday yet"),
    n @ 1..=12 => println!("I'm a child of age {n}"),
    n @ 13..=19 => println!("I'm a teen of age {n}"),
    n => println!("I'm an old person of age {n}"),
}
```

### Binding values с массивами

```rust
let s = [1, 2, 3, 4];

let mut t = &s[..];  
// or s.as_slice() – подмассив, в данном случае это весь массив. по сути это два указателя  

loop {
    match t {
        [head, tail @ ..] => {
            println!("{head}");
            t = tail;
        }
        [] => break,
    }
} 
// outputs: 1\n2\n3\n4\n
```

### Enumerations

```rust
enum Test {
    A(i32),
    B,
}

impl Test {
    pub fn unwrap(self) -> i32 {
        match self {
            Self::A(x) => x,
            Self::B => panic!("error!"),
        }
    } 
}
```

---

## If let

```rust
let optional = Some(7);

match optional {
    Some(i) => {
        println!("It's Some({i})");
    },
    _ => {},
    // ^ Required because `match` is exhaustive
}

// Другой способ
let optional = Some(7);
if let Some(i) = optional {
    println!("It's Some({i})");
}
```

---

## While let

```rust
let mut optional = Some(0);  

while let Some(i) = optional {
    if i > 9 {
        println!("Greater than 9, quit!");
        optional = None;
    } else {
        println!("`i` is `{i}`. Try again.");
        optional = Some(i + 1);
    } 
}
```

---

## Enumerations in Rust vs C++

**Discriminant** – id варианта enum, которые хранятся в битах в полях enum. Эти биты – дискриминант.

Число бит точно такое, какое нужно числу вариантов (30 вариантов enum – 5 бит).

Биты располагаются в неиспользуемых битах enum в других полях (компилятор распределяет их произвольно).

```rust
enum SimpleEnum {
    First,  // Дискриминант равен 0
    Second, // Дискриминант равен 1
    Third,  // Дискриминант равен 2
}

enum ExplicitEnum {
    First = 1,    // Дискриминант равен 1
    Second = 5,   // Дискриминант равен 5
    Third = 10,   // Дискриминант равен 10
}

enum ComplexEnum {
    Integer(i32),       // Дискриминант для этого варианта – ID1
    FloatingPoint(f64), // Дискриминант для этого варианта – ID2
    Text(String),       // Дискриминант для этого варианта – ID3
}
```

### Оптимизация размера enum

```rust
enum Test {
    First(bool), // bool имеет размер 1 байт, но занят только 1 бит
    Second, // не содержит данных
    Third, // не содержит данных
    Fourth, // не содержит данных
}

assert_eq!(
    std::mem::size_of::<Test>(), 
    1
); // компилятор помещает дискриминанты в свободные биты

assert_eq!(
    std::mem::size_of::<Option<Box<i32>>>(), 
    8 // аналог unique_ptr из C++
);
```

### Тесты размера

```rust
enum Test1 {
    First(bool, bool),
    Field0,
    // More fields...
    Field253,
}

enum Test2 {
    First(bool, bool),
    Field0,
    // More fields...
    Field254, // не содержит данных
}

assert_eq!(std::mem::size_of::<Test1>(), 2);
assert_eq!(std::mem::size_of::<Test2>(), 3);
```

---

## Vector

Динамический массив такой же как в C++:

```rust
pub struct Vec<T> {
    ptr: *mut T,
    length: usize,
    capacity: usize,
    // PhantomData<T>?
}

use std::mem::size_of;

assert_eq!(
    size_of::<Vec<i32>>(),
    size_of::<usize>() * 3, // 3 usize: указатель, capacity, length
);

let mut xs = vec![1, 2, 3];

// Чтобы объявить вектор с одинаковыми элементами
// и определенным количеством элементов:
// vec![42; 113];

xs.push(4);
assert_eq!(xs.len(), 4);
assert_eq!(xs[2], 3);
```

---

## Slices

```rust
let a = [1, 2, 3, 4, 5];
let slice1 = &a[1..4];
let slice2 = &slice1[..2];
assert_eq!(slice1, &[2, 3, 4]);
assert_eq!(slice2, &[2, 3]);
```

---

## `Panic!`

Вызывает паническое завершение программы:

```rust
let x = 42;

if x == 42 {
    panic!("The answer");
}
```

### Полезные макросы которые вызывают `panic!`

- `unimplemented!()`
- `unreachable!()`
- `todo!()`
- `assert!()` // проверяет что внутри () true
- `assert_eq!()` // проверяет равенство

---

## `Println!`

```rust
let x = 42;  
println!("{x}");
println!("The value of x is {}, and it's cool!", x);
println!("{:04}", x); // 0042
println!("{value}", value = x); // 42

let vec = vec![1, 2, 3];
println!("{vec:?}"); // [1, 2, 3]
println!("{:?}", vec);

let y = (100, 200);
println!("{:#?}", y);
// (
//     100, 
//     200,
// )
```

---

## Inhabited и uninhabited types

Бывают Inhabited и uninhabited типы. Например Inhabited это `i32`, `bool` и т.д., то есть те которые имеют значение. Uninhabited значений не имеет. Это тип `!`.

```rust
enum InhabitedEnum {
    Variant1,
    Variant2,
}

enum UninhabitedEnum {}
```

Rust всегда требует возвращать что-то корректное. Если функция не возвращает значение, то она возвращает тип `!`, такой тип не имеет значений.

```rust
fn never_returns() -> ! {
    panic!("This function never returns");
}

// error: mismatched types
// expected `i32`, found `()`
// fn func() -> i32 {}

// Работает вот так:
fn func() -> i32 {
    unimplemented!("not ready yet")
}
```

---

# Система владения (Ownership System)

## ❓Ownership❓#Q

**Ключевые правила владения:**

- **Каждое значение (value) имеет единственного владельца** — переменную, которая владеет этим значением.
- **Владелец может уступить (move) владение другому** — после этого предыдущая переменная становится недействительной.
- **Когда владелец выходит из области видимости, значение освобождается** — вызывается деструктор и память очищается.
- **Для копируемых (Copy) типов** (например, чисел) значения копируются, а не перемещаются — старый владелец остаётся действительным.

```rust
fn main() {
    let s = vec![1, 4, 8, 8];
    let u = s; // передача владения u
    println!("{:?}", u);
    
    // This won't compile!
    // println!("{:?}", s); // будет ошибка тк s деинициализорована
}

fn om_nom_nom(s: Vec<i32>) {
    println!("I have consumed {s:?}");
}

fn main() {
    let s = vec![1, 4, 8, 8];
    om_nom_nom(s); // передали владение в om_nom_nom
    // println!("{s:?}"); // не получится тк вектор деинициализирован после om_nom_nom 
}
```

**Однако:**

```rust
fn om_nom_nom(n: u32) {
    println!("{n}");
}

fn main() {
    let n: u32 = 110;
    let m = n; // тут все ок, есть Copy trait
    om_nom_nom(n); // Copy n в om_nom_nom
    om_nom_nom(m); // Copy m в om_nom_nom
    println!("{}", m + n);    
}
```

---

## Borrowing

### Правила Borrowing

1. **Много неизменяемых ссылок (`&T`)** на одно значение разрешены.
2. **Только одна изменяемая ссылка (`&mut T`)** разрешена на одно значение за раз.
3. **Нельзя одновременно иметь неизменяемые и изменяемые ссылки**.

```rust
let mut v = vec![1, 2, 3];
let x = &v[0]; 
v.push(4); // ссылка может испортиться т.к. при push в вектор, может реалоцироваться память
println!("{}", x);
```

В Rust есть **move** семантика, поэтому функция `sum` будет забирать владение  
(в C++ copy семантика, там в функции будет копия вектора):

```rust
fn sum(v: Vec<i32>) -> i32 {
    let mut result = 0;
    for i in v {
        result += i; 
    }
    result
} // выход из области видимости, v уничтожается

fn main() {
    let mut v = vec![1, 2, 3];
    println!("first sum: {}", sum(v));  
    // переменная v перемещается во владение sum
    
    v.push(4); // после вызова sum v больше не доступна в main
    println!("second sum: {}", sum(v));
}
```

**Трейт (trait)** в Rust — это механизм для определения общих интерфейсов (или контрактов), которые типы могут реализовывать.

```rust
trait Greet {
    fn greet(&self); // Метод, который должен быть реализован типами, поддерживающими этот трейт
}
```

Тип `Vec<T>` не реализует `Copy` trait, вместо этого `Vec<T>` реализует `Clone` trait.

`Copy` предназначается для типов которые можно побитово скопировать, например примитивные типы и массивы фиксированной длины.

---

## Lifetime

- Начинается когда value создан и заканчивается когда последний раз использован
- Ссылки которые живут дольше чем lifetime в Rust не допускаются
- Rust рассчитывает lifetime используя статический анализ кода во время компиляции
- Когда lifetime заканчивается, Rust вызывает Drop функции

---

## Drop Implementation

```rust
let vec = vec![];
drop(vec);

pub fn drop<T>(_x: T) {} // передача владения, дальше автоматически вызывается drop реализованный для типа _x
```

---

## Drop Flags

```rust
fn main() {
    let x = Box::new(92);
    let y;
    let z;

    if cond {
        y = x;
        // z не инициализируется, но это безопасно, так как z не используется
        // `x` больше не может использоваться после этого присваивания
    } else {
        z = x;
        // y не инициализируется, но это безопасно, так как y не используется
        // `x` больше не может использоваться после этого присваивания
    }    
    
    // Кто-то из y или z освободит память, когда выйдет из области видимости
}
```

---

## Borrowing examples

```rust
let mut x;
// assert_eq!(x, 42); // нельзя получать доступ к неинициализированной памяти
x = 42;  

let y = &x; // создается неизменяемая ссылка, указывает на значение 42 в x
x = 43; // x поменялся, но ссылка все еще указывает на 42  
assert_eq!(*y, 42); // это отработает

let x1 = 42;
let y1 = Box::new(84);
{ // starts a new scope
    let z = (x1, y1); // x1 копируется (реализует трейт Copy) в z, т.к. i32,
                       // y1 перемещается в z, т.к. Box<T> реализует Move
} // выход из области видимости, z дропается и освобождает значение y1, память выделенную в куче для Box освобождается  

// x1's value is **Copy**, so it was **not** moved into z
let x2 = x1;  

// y1's value is **not Copy**, so it was moved into z
// let y2 = y1; // ошибка
```

## References

>**ссылка `&`** (например, `&u32`) представляет собой **простой указатель** (тонкий, _thin pointer_):

- просто адрес в памяти
- размер ссылки == размер указателя 8 байт на 64 битных системах

- Реально существующий указатель в скомпилированной программе
- Не может быть NULL
- Гарантирует что объект жив
- Неизменяемые `&` и изменяемые `&mut`

```rust
let mut x: i32 = 92;

// let r: &mut i32 = &mut 92;
// изменяемая ссылка на литерал - ошибка компиляции

// let r: &i32 = &92;  
// так делать можно  

let r: &mut i32 = &mut x; // ссылка создана явно
*r += 1; // разыменование явно, x увеличится на 1  

// r = r + 1; // ошибка компиляции, в Rust нельзя так менять ссылки
```

### Сравнение с C++

```cpp
// C++
// чтобы хранить ссылки в векторе нужен std::reference_wrapper
int x = 10;
std::vector<std::reference_wrapper<int>> v;
v.push_back(x);  
// x автоматически преобразуется в std::reference_wrapper<int>, и ссылка на x добавляется в вектор v. Теперь v содержит ссылку на x, а не копию значения x
```

```rust
// Rust
let x = 10;
let mut v = Vec::new();
v.push(&x);
```

### Fat pointer

>ссылка, которая кроме самого указателя на данные хранит дополнительную информацию:

Например, типы вроде:

- **срезов** (`&[T]`, `&str`).
- **trait objects** (`&dyn Trait`).

хранят два элемента:

- указатель на данные,
- дополнительную информацию (длина или vtable).

```rust
let slice: &[u32] = &[1, 2, 3];
// slice содержит:
// - адрес первого элемента
// - длину (3 элемента)
```

Размер обычно 16 байт (указатель + доп. данные)

### ❓На какую область памяти может указывать срез `&str`❓ #Q

Неизменяемая ссылка (срез) на данные типа `str`, которые могут находиться в:

- **В сегменте данных программы** (для строковых литералов):

```rust
let s: &str = "hello"; // "hello" хранится в бинарном сегменте программы (read-only память).
```

- **На куче (heap)** (если `&str` — это срез строки, которой владеет объект типа `String` или другая структура на куче):

```rust
let string = String::from("hello");
let slice: &str = &string[0..3]; // slice указывает на данные, хранящиеся на heap-е.
```

- **На стеке (stack)** (если срез ссылается на временные данные, выделенные на стеке):

```rust
let array = ['a', 'b', 'c'];
let slice: &str = std::str::from_utf8(&array).unwrap(); // slice указывает на стековый массив.
```

---

## Pointers

- Бесполезны без `unsafe`
- Могут быть NULL
- Нет гарантии что объект жив
- Редко используются

```rust
// Rust
let x: *const i32 = std::ptr::null(); // константный указатель на константный i32  
let mut y: *const i32 = std::ptr::null(); // изменяемый указатель на константный i32  
let z: *mut i32 = std::ptr::null(); // константный указатель на изменяемый i32  
let mut t: *mut i32 = std::ptr::null(); // изменяемый указатель на изменяемый i32
```

```cpp
// C++
uint32_t const * const x = nullptr; // константный указатель на константный uint32_t
uint32_t const * y = nullptr; // указатель на константный uint32_t
uint32_t* const z = nullptr; // константный указатель на изменяемый uint32_t
uint32_t* t = nullptr; // изменяемый указатель на изменяемый uint32_t
```

## Умные указатели

### Box

Указатель на какие-то данные в куче. Похожи на C++ `std::unique_ptr` но без NULL.

```rust
let x: Box<i32> = Box::new(92);
```

### ❓`Box::new([0u64; 1000000000])`❓#Q

1. **Тип данных**

- [0u64; 1_000_000_000] — массив из миллиарда элементов типа u64.
- Размер каждого u64 равен 8 байт.
- Общий объем памяти: 8 ГБ.

1. **Размещение памяти:**
    - Box::new выделит непрерывный участок памяти на куче (heap) размером **8 ГБ**.

2. **Детали процесса аллокации**
    - Вызовется аллокатор памяти (по умолчанию системный, обычно через malloc или аналогичный).
    - Попытка выделения 8 ГБ непрерывного блока памяти.
      - _ОС выделяет **виртуальное адресное пространство** (системные вызовы mmap или brk), а не физическую память напрямую._
      - _Виртуальное пространство — это просто набор адресов, которые процесс видит и использует. Оно не обязательно подкреплено реальной физической памятью сразу._
    - Если у системы недостаточно свободной оперативной памяти и виртуальной памяти (swap), то:
        - Либо ОС завершит процесс с ошибкой out-of-memory.
        - Либо аллокатор сразу вернёт ошибку и произойдёт panic (alloc::alloc::handle_alloc_error).
3. **Последствия:**
    - Успешная аллокация требует, чтобы у системы были доступны минимум 8 ГБ оперативной памяти или swap.
    - Если система не может удовлетворить такой запрос, программа упадёт с ошибкой аллокации.

### `Rc<T>` - Reference Counted

```rust
use std::rc::Rc;

enum List {
    Cons(i32, Rc<List>),
    Nil,
}

use crate::List::{Cons, Nil};

fn main() {
    let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil)))));
    let b = Cons(3, Rc::clone(&a));
    let c = Cons(4, Rc::clone(&a));
    
    println!("Reference count: {}", Rc::strong_count(&a)); // 3
}
```

### `Arc<T>` - Atomic Reference Counted

`Arc<T>` - Atomic Reference Counted. Похож на `Rc<T>`, но потокобезопасный.

```rust
use std::sync::Arc;

let a = Arc::new(Cons(5, Arc::new(Cons(10, Arc::new(Nil)))));
let b = Cons(3, Arc::clone(&a));
let c = Cons(4, Arc::clone(&a));

println!("Reference count: {}", Arc::strong_count(&a)); // 3
```

`Arc<T>` используется для многопоточных приложений, когда нужно использовать `Rc<T>`, но нужно гарантировать, что счетчик ссылок будет атомарным.

### ❓Для чего использовать умные указатели Box, Rc, Arc❓ #Q

- **`Box<T>`** – владение данными на куче (**heap**), единичное владение
  - Рекурсивные структуры данных
  - Динамический полиморфизм (трейты-объекты)
- **`Rc<T>`** – разделяемое владение (**shared ownership**) для однопоточных сценариев (не реализует `Send`) 
  - Графы, деревья или структуры данных с разделяемыми узлами.
  - Ситуации, когда трудно выделить единственного владельца
- **`Arc<T>`** – разделяемое владение для многопоточных сценариев (**thread-safe**). Под капотом атомарные операции.
  - Совместное владение данными между потоками.
  - Многопоточные структуры данных.

### ❓Как переместить значение из стека в кучу❓ #Q

- `Box`

```rust
let x = 5; // на стеке
let boxed_x = Box::new(x); // перемещаем x на кучу
```

- `Vec`

```rust
let array = [1, 2, 3]; // массив на стеке
let vec = array.to_vec(); // перемещаем данные на кучу в Vec
```

- `String`

```rust
let s = "hello"; // строковый литерал на стеке (ссылка на статические данные)
let string = s.to_string(); // копируем данные на кучу
```

---

# Standard Library

## Option & Result

```rust
enum Option<T> {
    Some(T),
    None,
}

enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

### Option API

Используется в Rust вместо null.

```rust
let result = Some("string");
match result {
    Some(s) => println!("string inside: {s}"),
    None => println!("NO string inside"),
}
```

### `.unwrap()` и `.expect()`

**Unwrap**
- Метод unwrap извлекает значение из Option или Result, если оно есть.
- Eсли значения нет (None) или имеется ошибка (Err), unwrap вызывает **panic** (аварийное завершение программы).

**Expect**
- Работает аналогично unwrap, но позволяет указать **собственное сообщение об ошибке** для паники.
- Полезен для улучшения читаемости кода и отладки.

```rust
fn unwrap(self) -> T;
fn expect(self, msg: &str) -> T;

let opt = Some(22022022);
assert!(opt.is_some());  
assert!(!opt.is_none());  

assert_eq!(opt.unwrap(), 22022022);
let x = opt.unwrap(); // **Copy**

let newest_opt: Option<i32> = None;
// newest_opt.expect("I'll panic!");

let new_opt = Some(Vec::<i32>::new());
assert_eq!(new_opt.unwrap(), Vec::<i32>::new());
// ~~~~~~^^^^^^^^ тут передалось владение

// let x = new_opt.unwrap(); // уже не сработает т.к. владение было передано и значение не может быть снова использовано
```

В настоящем коде – `unwrap_or` или `unwrap_or_else`

```rust
let value = result.unwrap_or(0); // Вернёт значение или 0 по умолчанию
```
### Магическая функция `.as_ref()`

```rust
fn as_ref(&self) -> Option<&T>; // &self is &Option<T>

let new_opt = Some(Vec::<i32>::new());
assert_eq!(new_opt.as_ref().unwrap(), &Vec::<i32>::new()); // мы тут не передаем владение и значит сохраняем его  

let x = new_opt.unwrap(); // можно, т.к. мы сохранили значение
// There's also .as_mut() function
```

### Отображение `Option<T>` в `Option<U>`

```rust
fn map<U, F>(self, f: F) -> Option<U>;

let maybe_some_string = Some(String::from("Hello, World!"));
// `Option::map` takes self *by value*,
// consuming `maybe_some_string`
let maybe_some_len = maybe_some_string.map(|s| s.len());
assert_eq!(maybe_some_len, Some(13));
```

### Различные методы Option

```rust
// Применяет функцию f к значению внутри Option, если оно существует.
// Если Option равно None, возвращает значение default.
fn map_or<U, F>(self, default: U, f: F) -> U;

// Похож на map_or, но позволяет лениво вычислять значение default с помощью функции default, только если Option равно None.
fn map_or_else<U, D, F>(self, default: D, f: F) -> U;

// Возвращает значение внутри Option, если оно существует. В противном случае возвращает default.
fn unwrap_or(self, default: T) -> T;

// Похож на unwrap_or, но позволяет лениво вычислять значение по умолчанию с помощью функции f, только если Option равно None.
fn unwrap_or_else<F>(self, f: F) -> T;

// Если self равно Some, возвращает optb. В противном случае возвращает None.
fn and<U>(self, optb: Option<U>) -> Option<U>;

// Если self равно Some, применяет функцию f к значению внутри Option и возвращает результат. В противном случае возвращает None.
fn and_then<U, F>(self, f: F) -> Option<U>;

// Если self равно Some, возвращает self. В противном случае возвращает optb.
fn or(self, optb: Option<T>) -> Option<T>;

// Похож на or, но позволяет лениво вычислять вторую опцию с помощью функции f, только если Option равно None.
fn or_else<F>(self, f: F) -> Option<T>;

// Возвращает Some только если одна из опций равна Some, но не обе.
fn xor(self, optb: Option<T>) -> Option<T>;

// Если обе опции равны Some, возвращает Some с кортежем значений внутри опций. В противном случае возвращает None.
fn zip<U>(self, other: Option<U>) -> Option<(T, U)>;
```

### Option и ownership

```rust
fn take(&mut self) -> Option<T>;
// извлечь значение из Option

fn replace(&mut self, value: T) -> Option<T>;  
// заменить текущее значение Option новым значением, возвращает старое, если оно существовало (Some -> Some, None -> None)

fn insert(&mut self, value: T) -> &mut T;
// вставить значение в Option и вернуть изменяемую ссылку на новое значение
```

### Option API и ownership: take

```rust
struct Node<T> {
    elem: T,
    next: Option<Box<Node<T>>>,
}

pub struct List<T> {
    head: Option<Box<Node<T>>>,
}

impl<T> List<T> {
    pub fn pop(&mut self) -> Option<T> {
        self.head.take().map(|node| {  
            self.head = node.next;
            node.elem        
        })
    }
}
```

## Result и обработка ошибок

### Основы Result

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}

// Пример функции возвращающей Result
fn divide(x: f64, y: f64) -> Result<f64, String> {
    if y == 0.0 {
        Err(String::from("Division by zero"))
    } else {
        Ok(x / y)
    }
}

// Использование
match divide(10.0, 2.0) {
    Ok(result) => println!("Result: {}", result),
    Err(e) => println!("Error: {}", e),
}
```

### Методы Result

```rust
let result: Result<i32, &str> = Ok(42);

// unwrap() - паникует при ошибке
let value = result.unwrap();

// expect() - паникует с сообщением
let value = result.expect("Failed to get value");

// unwrap_or() - возвращает дефолтное значение при ошибке
let value = result.unwrap_or(0);

// map() - преобразует Ok значение
let doubled = result.map(|x| x * 2);

// and_then() - цепочка операций
let result = divide(10.0, 2.0)
    .and_then(|x| divide(x, 2.0))
    .and_then(|x| divide(x, 1.0));
```

### Оператор `?`

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_file_contents(filename: &str) -> Result<String, io::Error> {
    let mut file = File::open(filename)?; // ? автоматически возвращает ошибку
    let mut contents = String::new();
    file.read_to_string(&mut contents)?;
    Ok(contents)
}
```

---

# Traits - Трейты

- это основа системы типов Rust, определяющая общее поведение.
- похож на интерфейс в других языках

```rust
// Определение трейта
pub struct Sheep {
    name: String,
}

impl Animal for Sheep {
    fn name(&self) -> String {
        self.name.clone()
    }

    fn noise(&self) -> String {
        String::from("baaah")
    }
}
```

```rust
trait Animal {
    // No pub keyword
    fn name(&self) -> String;
    fn noise(&self) -> String;
    fn talk(&self) {
        println!("{} says {}", self.name(), self.noise());
    }
}
```

```rust
let sheep = Sheep { name: String::from("Dolly") };
assert_eq!(sheep.name(), "Dolly");
sheep.talk(); // Dolly says baaah
```

## Трейты с дефолтной реализацией

```rust
pub struct Dog {
    name: String,
}

impl Animal for Dog {
    fn name(&self) -> String {
        self.name.clone()
    }

    fn noise(&self) -> String {
        String::from("woof")
    }

    // Дефолтные методы можно переопределить
    // Default trait methods can be overridden
    fn talk(&self) { 
        println!("{} says {}", self.name(), self.noise());
    }
}
```

## Traits: where keyword

```rust
#[derive(Clone)] // Clone trait is implemented for all types that implement Copy
pub struct Human {
    name: String,
}

// impl – конкретная реализация трейта для типа Human
impl Animal for Human {
    fn name(&self) -> String {
        self.name.clone()
    }

    fn noise(&self) -> String {
        // self: &Human - конкретный тип!
        // self.name: String - тоже конкретный тип!
        let cloned = self.name.clone();  // ✅ ОК
        cloned.name()

        // Тоже работает, тк Human иммеет #[derive(Clone)]
        let cloned_human = self.clone(); // Human::clone()
    }

    fn talk(&self) {
        println!("My name is {}", self.name());
    }
}
```

```rust
// pub - дефолтная реализация трейта для типа Animal
pub trait Animal {
    fn name(&self) -> String;
    fn noise(&self) -> String;
    
    fn talk(&self) {
        // This clones &self, not self - клонирование ссылки, а не самого объекта
        // let cloned = self.clone();

        // error: no method named `clone` found for type parameter `Self` in the current scope

        // let cloned = self.clone();  // ❌ ОШИБКА
        // let cloned = (*self).clone(); // ❌ ОШИБКА

        println!("{} says {}", self.name(), self.noise());
    }
}
```

```rust
pub trait Animal {
    where
        Self: Clone, // For Self we require that it implements Clone
    {
        fn name(&self) -> String;
        fn noise(&self) -> String;
    }

    fn talk(&self) {
        // Compiles fine
        // This clones self, not &self - клонирование самого объекта

        let cloned = self.clone();
        println!("{} says {}", cloned.name(), cloned.noise());
    }
}
```

**По умолчанию Rust ничего не требует от типов. Если мы хотим чтобы тип реализовывал трейт, мы должны явно указать это.**

## Generics & trait bounds

```rust
trait Strange1<T: Clone + Hash + Iterator> {
    where
        T::Item: Clone
    {
        fn new() -> Self;
    }

    trait Strange2<T>
    where
        T: Clone + Hash + Iterator,
        T::Item: Clone
    {
        fn new() -> Self;
    }
}
```

**Требования можно указать только для generic с `where`**

## Супертрейты - Supertraits

В rust нет наследования, но можно определить трейты, которые являются множествами других трейтов.

```rust
trait Person {
    fn name(&self) -> String;
}

trait Student: Person {
    fn university(&self) -> String;
}

trait Programmer {
    fn fav_language(&self) -> String;
}

trait CompSciStudent: Programmer + Student {
    fn git_username(&self) -> String;
}
```

## Fully Qualified Syntax

Несколько методов с одинаковым именем, но разными параметрами.

```rust
struct Form {
    username: String,
    age: u8,
}

trait UsernameWidget {
    fn get(&self) -> String;
}

trait AgeWidget {
    fn get(&self) -> u8;
}

let form = Form {
    username: String::from("John"),
    age: 25,
};

println!("{}", form.get()); // error[E0034]: multiple applicable items in scope
```

Указать что метод принадлежит конкретному трейту

```rust
// ✅ Работает, если нет конфликтов имен трейтов
let username = UsernameWidget::get(&form);
assert_eq!(username, "John");

// ✅ Всегда работает, максимально явно
let age = <Form as AgeWidget>::get(&form); // Fully Qualified Syntax
assert_eq!(age, 25);
```

## `impl` keyword

Принимается любой тип, который реализует трейты.

```rust
fn func<T: MyTrait + Clone>(input:T) {
    // ...
}

// Тоже самое через специальный синтаксис для указания трейтов
fn func(input: impl MyTrait + Clone) {
    // ...
}
```

`impl` - это синтаксический сахар для указания трейтов.

## Miltiple `impl`

```rust
pub enum Option<T> {
    // ...
}

// Первый impl блок - методы доступны для ВСЕХ Option<T>
impl<T> Option<T> {
    // ...
    
}

// Второй impl блок - методы доступны только для Option<T>, где T: Default
impl<T> Option<T> 
where
    T: Default,
{
    // ...
}
```

## where and selection

```rust
pub enum Option<T> {
    // ...
}

impl<T> Option<T> {
    pub fn unwrap_or_default(self) -> T {
    where
        T: Default,
    {
        // ...
    }
}
```

## Exotically Sized Types

В основном размер типов известен на этапе компиляции. И положтельны.

Олнако в Rust есть типы:

- "Regular" (нет формального названия)
- Dynamic Sized Types (DST)
- Zero-Sized Types (ZST)
- Empty Types

## Dynamic Sized Types (DST)

Типы у которых размер не известен на этапе компиляции.

- Slice, даже обычный `[u8]` и `str`
- Trait Object, такие как `dyn Trait`

Такие типы не имплементируют `Sized` маркер трейт.

**По-умолчанию Rust требует от типов, чтобы они были `Sized`.**

```rust
pub trait Sized {}
```

- Тип `T` и `&T` - это разные типы
- `str` unsized
- `&str` - это указатель на начало слайса и его длину, sized
- `[u8]` и  `[i64]` unsized

## Dynamic Sized Types (DST): Trait Object

```rust
trait Hello {
    fn hello(&self);
}

fn func(arr: &[Hello]) {
    for i in arr {
        i.hello();
    }
}
```

Этот код не скомпилируется, т.к. `Hello` - это трейт, а не тип.

```rust
fn func<T: Hello>(arr: &[T]) {
    for i in arr {
        i.hello();
    }
}
```

Скомпилируется, тк мы неявно требуем от `T` `Sized`.

`dyn` - это ключевое слово для создания trait object.

```rust
fn func(arr: &[&dyn Hello]) {
    for i in arr {
        i.hello();
    } // ✅ Скомпилируется
}
```

`dyn Hello` - это `Unsized`

`&dyn Hello` - это `trait object`, который в памяти представляет собой `fat pointer` (толстый указатель) размером 16 байт, состоящий из двух указателей:

```rust
// Упрощенное представление в памяти
struct TraitObject {
    data: *mut (),      // 8 байт - указатель на данные объекта
    vtable: *const (),  // 8 байт - указатель на таблицу виртуальных методов
}
```

```rust
impl Hello for str {
    fn hello(&self) {
        println!("Hello, {}!", self);
    }
}

let x = "Hello"; // x: &str (строковый литерал)

let r1: &dyn Hello = &x;  // trait object через ссылку
// r1 - это fat pointer (16 байт):
// [ptr to data: 8 байт][ptr to vtable: 8 байт]
//        ↓                    ↓
//    "Hello" в памяти    таблица методов Hello для str

// Единоличное владение
let r2: Box<dyn Hello> = Box::new(x.clone()); // trait object в куче
// Box выделяет память в куче и хранит там строку
// r2 также fat pointer, но указывает на данные в куче

// Множественное владение
let r3: Rc<dyn Hello> = Rc::new(x.clone()); // trait object в куче с подсчетом ссылокъ
// Rc (Reference Counted) позволяет множественное владение
// Внутри Rc хранится счетчик ссылок + данные
```

С `dyn` нельзя требовать больше чем один не автотрейт от trait object, нужно вместо этого использовать supertrait.

```rust
let x = "hello world";
// let r: &dyn Hello + Clone = &x; // ❌ ОШИБКА

trait HelloWorld: Hello + World {} // supertrait

impl HelloWorld for str {
    // ...
}

let r: &dyn HelloWorld = &x; // ✅ Работает
```

Можно требовать только автотрейты от trait object.

```rust
trait X {}

fn test(x: Box<dyn X + Send>) {}
```

## Trait Objects: Object Safety

```rust
fn test(x: Box<dyn Clone + Send>) {}
// ~~~~~~~~~~~~^^^^^^^^^ ❌ ОШИБКА
// Clone не может стать объектом
```

1. Методы не должны возвращать `Self`
2. Методы не могут быть `generic`
3. Трейт не может иметь статические методы (тк невозможно определить какой инстанс использовать)

```rust
trait Clone {
    fn clone(&self) -> Self; // ❌ Возвращает Self!
}

// При создании trait object мы теряем информацию о конкретном типе
let obj: Box<dyn Clone> = Box::new(String::from("hello"));
// Какой тип должен вернуть obj.clone()? 
// String? i32? Мы не знаем!
```

## Impl dyn

```rust
impl dyn Example {
    fn is_dyn(&self) -> bool {
        true
    }
}

strust Test {}
impl Example for Test {}

let x = Test {};
let y: Box<dyn Example> = Box::new(Test {});
// Ошибка: `dyn Example` не реализует `Example`
// x.is_dyn(); // ❌ ОШИБКА

y.is_dyn(); // ✅ Работает
```

## Trait Object vs Generics

### Когда использовать Trait Object, а когда Generics? #Q 

| Критерий               | Trait Objects (dyn Trait)                                                                                                                                         | Generics (<T: Trait>)                                                                                                     |
| ---------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| Производительность     | Медленнее (vtable)                                                                                                                                                | Быстрее (инлайнинг)                                                                                                       |
| Размер бинарника       | Меньше                                                                                                                                                            | Больше (дублирование кода)                                                                                                |
| Гетерогенные коллекции | ✅ Можно                                                                                                                                                           | ❌ Нельзя                                                                                                                  |
| Object safety          | Только object-safe трейты                                                                                                                                         | Любые трейты                                                                                                              |
| Время компиляции       | Быстрее                                                                                                                                                           | Медленнее                                                                                                                 |
| Use cases              | 🎯 Гетерогенные коллекции<br>📦 Размер бинарника <br>🔌 Создаете plugin систему<br>⚡ Время компиляции важнее производительности<br>🎭 Типы определяются в runtime | 🚀 Производительность<br>🎯 Один тип за раз<br>🔧 Non-object-safe трейты<br>📚 Библиотекa<br>🎨 Associated types или Self |

> **Гетерогенные коллекции** - это коллекции, которые могут содержать элементы разных типов в одном контейнере.

```rust
// Heterogenous collections:
// shapes[0]: [ptr to Circle data | ptr to Circle's Draw vtable]
// shapes[1]: [ptr to Rectangle data | ptr to Rectangle's Draw vtable]
//             ↑ разные типы данных!   ↑ разные vtables!

// Homogenous collections:
// circles[0]: Circle { radius: 5.0 }      ← прямо в векторе
// circles[1]: Circle { radius: 3.0 }      ← тот же тип
//             ↑ одинаковые типы, одинаковый размер
```

- Generics - все элементы должны быть одного типа.
- Trait Object - можно создавать коллекции из разных типов, которые реализуют один трейт.

```rust
// Разные типы, но реализующие один трейт
trait Draw {
    fn draw(&self);
}

struct Circle { radius: f32 }
struct Rectangle { width: f32, height: f32 }
struct Triangle { base: f32, height: f32 }

impl Draw for Circle {
    fn draw(&self) { println!("Drawing circle with radius {}", self.radius); }
}

impl Draw for Rectangle {
    fn draw(&self) { println!("Drawing rectangle {}x{}", self.width, self.height); }
}

impl Draw for Triangle {
    fn draw(&self) { println!("Drawing triangle with base {}", self.base); }
}

// ✅ Гетерогенная коллекция с trait objects
let shapes: Vec<Box<dyn Draw>> = vec![
    Box::new(Circle { radius: 5.0 }),      // Тип: Circle
    Box::new(Rectangle { width: 10.0, height: 20.0 }), // Тип: Rectangle
    Box::new(Triangle { base: 8.0, height: 12.0 }),    // Тип: Triangle
    Box::new(Circle { radius: 3.0 }),      // Снова Circle
];

// Вызываем методы для всех элементов
for shape in &shapes {
    shape.draw(); // Динамическая диспетчеризация
}
```

## Standard Library Traits

Трейты из стандартной библиотеки:

- `Default`
- `Clone`
- `Copy`
- `PartialEq`
- `Eq`
- Oredering:
  - `PartialOrd`
  - `Ord`
  - `Reverese`
- `Hasher`
- `Hash`
- TBD

- Процедурные макросы генерируют код, например `derive`
- Декларативные макросы (такие как `println!`, `macro_rules!`) работают по принципу pattern matching (сопоставления с образцом)Ё

```rust
macro_rules! create_function {
    ($func_name:ident) => {
        fn $func_name() {
            println!("You called {:?}()", stringify!($func_name));
        }
    };
}

create_function!(foo);
create_function!(bar);

// Сгенерирует:
// fn foo() { println!("You called \"foo\"()"); }
// fn bar() { println!("You called \"bar\"()"); }
```

### Default

- Всего один метод default(), который возвращает экземпляр типа Self с разумными значениями по умолчанию.
- Используется для создания экземпляров типа с разумными значениями по умолчанию. Также чтобы не повторять один и тот же код для создания экземпляров типа с разумными значениями по умолчанию.
- Без default Rust не компилируется без прописанных всех полей типа.

```rust
// Создает дефолтную имплементацию T
#[derive(Default)]

pub trait Default {
    fn default() -> Self;
}
```

#### ❓Почему `Default` не наследуется по умолчанию❓ #Q

- Философия явности (Explicit over implicit). Rust предпочитает явное поведение неявному. Автоматическое наследование трейтов может привести к неожиданным последствиям.

```rust
struct DatabaseConnection {
    host: String,
    password: String,
    socket: TcpStream, // Нет Default!
}
```

- Не все типы должны иметь "разумные" Default значения

```rust
struct BankAccount {
    balance: f64,        // 0.0 может быть НЕ разумным
    account_number: u64, // 0 - невалидный номер счета!
    owner_id: UserId,    // Нет Default у UserId
}
```

- Проблемы с вложенными типами

```rust
struct Config {
    database: DatabaseConfig,
    redis: RedisConfig,
    logging: LoggingConfig,
}

struct DatabaseConfig {
    host: String,
    port: u16,
    credentials: Credentials, // ❌ У Credentials может не быть Default
}
```

- Ресурсы и RAII: Многие типы в Rust управляют ресурсами. Автоматический Default может нарушить RAII
- Принцип "opt-in" против "opt-out": Разработчик сознательно выбирает Default и явно его использует.

### Clone

- Трейт Clone позволяет явно создавать копии экземпляров типа.
- `#[derive(Clone)]` автоматически генерирует код для создания копий экземпляров типа.

```rust
pub trait Clone {
    fn clone(&self) -> Self;

    // The default implementation
    fn clone_from(&mut self, source: &Self) {
        *self = source.clone();
    }
}
```

#### ❓Почему `Clone` не наследуется по умолчанию❓ #Q

- Производительность - Clone может быть очень дорогим
- Управление уникальными ресурсами: некоторые ресурсы НЕ должны клонироваться
- Типы с чувствительными данными
### Copy

- `#[derive(Copy)]`
- Copy - это маркер трейт (marker trait) который:
  - Не имеет методов
  - Наследует от Clone
  - Указывает компилятору: "этот тип можно скопировать побитово"

```rust
pub trait Copy : Clone {}
```

#### Copy типы (Copy семантика) #Q

- Все примитивные типы:

```rust
// Числовые типы
let a: i32 = 42;
let b: u64 = 100;
let c: f64 = 3.14;
let d: usize = 8;

// Булевы и символы
let flag: bool = true;
let letter: char = 'A';

// Все они Copy!
let a2 = a; // Copy, не move
let b2 = b; // Copy, не move
// a, b все еще доступны
```

- Массивы и кортежи (если элементы Copy):

```rust
let arr: [i32; 3] = [1, 2, 3];
let arr2 = arr; // ✅ Copy

let tuple: (i32, bool) = (42, true);
let tuple2 = tuple; // ✅ Copy

// Но НЕ Copy, если содержат не-Copy типы:
let mixed: (i32, String) = (42, "hello".to_string());
let mixed2 = mixed; // ❌ Move, потому что String не Copy
```

- Ссылки:

```rust
let value = 42;
let ref1: &i32 = &value;
let ref2 = ref1; // ✅ Copy - копируется указатель, не данные

// Обе ссылки доступны
println!("{}", ref1); // ✅ ОК
println!("{}", ref2); // ✅ ОК
```

#### Типы которые не могут быть Copy #Q

- Типы с кучей;
- Типы с Drop;
- Типы с не Copy полями.

### PartialEq

- `#[derive(PartialEq)]`
- Трейт PartialEq позволяет сравнивать экземпляры типа на равенство.

```rust
// The generic and default value!
pub trait PartialEq<Rhs: ?Sized = Self>
    where
        Rhs: ?Sized,
    {
        fn eq(&self, other: &Rhs) -> bool;
        fn ne(&self, other: &Rhs) -> bool {
            !self.eq(other)
        }
    }
```

**Таже этот трейт перегружает операторы == и !=.**

Иногда мы хотим от трейта чтобы он работал по разному для разных типов. В случае `PartialEq` мы хотим чтобы было сравнение двух элементов разных типов.

```rust
struct A {
    x: i32,
}

// The same as `#[derive(PartialEq)]`
// Allows us to compare two A's
impl PartialEq for A {
    fn eq(&self, other: &Self) -> bool {
        self.x.eq(&other.x)
    }
}
```

### Trairs and generics

```rust
#[derive(PartialEq)]
struct B {
    x: i32,
}

// Allow us to compare B with A when A is on the right side
impl PartialEq<A> for B {
    fn eq(&self, other: &A) -> bool {
        self.x.eq(&other.x) // Same as self.x == other.x
    }
}
```

```rust
// Define structs and traits

let a1 = A { x: 42 };
let a2 = A { x: 43 };

assert!(a1 == a2); // ✅ OK

let b = B { x: 42 };
assert!(b == a2); // ❌ Error: no method named `eq` found for struct `B` in the current scope
```

Имплементация `PartialEq` должна удовлетворять:

- `a != b` тогда и только тогда, когда `!(a == b)`
- Симметричности: если A: `PartialEq<B>` и B: `PartialEq<A>`, тогда `a == b` тогда и только тогда, когда `b == a`
- Транзитивности: если A: `PartialEq<B>` и B: `PartialEq<C>`, и A: `PartialEq<C>`, тогда `a == c` и `a == b` и `b == c`, тогда `a == c`

#### ❓Почему нужен PartialEq❓ #Q

В математике существует разница между:

- Частичное отношение эквивалентности - не все элементы можно сравнить
- Полное отношение эквивалентности - все элементы сравнимы

**В Rust мы используем PartialEq для частичного отношения эквивалентности.**

| Свойство                                                                      | PartialEq                         | Eq                         |
|---------------------------------------------------|------------------------|------------------|
| Симметричность: a == b ⟺ b == a                       | ✅                                    | ✅                         |
| Транзитивность: если a == b и b == c, то a == c  | ✅                                    | ✅                         |
| Рефлексивность: a == a                                            | ❌ (может быть false!) | ✅ (всегда true) |

- Числа с плавающей точкой (NaN не равен ничему, даже самому себе!)
- Частичные структуры данных
- Сравнение разных типов
- Сортировка чисел с плавающей точкой

### Eq

- `#[derive(Eq)]`
- Трейт говорит компилятору, что `PartialEq` имплементация рефлексивна.

```rust
pub trait Eq: PartialEq<Self> {}

// Reflexive: a == a is always true
```

### Ordering

```rust
pub enum Ordering {
    Less,
    Equal,
    Greater,
}
```

Functions

```rust
fn is_eq(self, other: Self) -> bool;
fn is_ne(self, other: Self) -> bool;
fn is_lt(self, other: Self) -> bool; // And some similar to this there
fn reverse(self) -> Ordering;
fn then(self, other: Ordering) -> Ordering;
fn then_with<F>(self, f: F) -> Ordering; // Applies F to Ordering
```

#### PartialOrd

`#[derive(PartialOrd)]`

Трейт для срвавнваемых элементов.

```rust
pub trait PartialOrd<Rhs: ?Sized = Self>: PartialEq<Rhs>
    where
        Rhs: ?Sized,
    {
        fn partial_cmp(&self, other: &Rhs) -> Option<Ordering>;

        fn lt(&self, other: &Rhs) -> bool {..}
        fn le(&self, other: &Rhs) -> bool {..}
        fn gt(&self, other: &Rhs) -> bool {..}
        fn ge(&self, other: &Rhs) -> bool {..}
    }
```

**Также перегружает операторы `<` и `>`.**

Методы этого трейта должны быть консистентными друг с другом и с `PartialEq`.

- `a == b` <=> `partial_cmp(a, b) == Some(Equal)`
- `a < b` <=> `partial_cmp(a, b) == Some(Less)`
- `a > b` <=> `partial_cmp(a, b) == Some(Greater)`
- `a <= b` <=> `partial_cmp(a, b) == Some(Less) || partial_cmp(a, b) == Some(Equal)`
- `a >= b` <=> `partial_cmp(a, b) == Some(Greater) || partial_cmp(a, b) == Some(Equal)`

#### Зачем нужен PartialOrd? #Q

- 🔢 NaN проблема - не все числа сравнимы
- 🎯 Частичные порядки в математике
- 📊 Многомерные данные - точки могут быть несравнимы
- 🛡️ Типобезопасность - компилятор не даст сломать логику
- 🎨 Гибкость - кастомная логика сравнения

#### Ord

`#[derive(Ord)]`

Трейт для полных порядков.

```rust
pub trait Ord: Eq + PartialEq<Self> {
    fn cmp(&self, other: &Self) -> Ordering;

    fn max(self, other: Self) -> Self {...};
    fn min(self, other: Self) -> Self {...};
    fn clamp(self, min: Self, max: Self) -> Self {...};
    // clamp - это функция ограничения (зажимания) значения в определенном диапазоне.
}
```

Имплементация должна быть консистентной с `PartialOrd` и обеспечивать `max`, `min` и `clamp`.

- `partial_cmp(a, b) == Some(Equal)` <=> `a == b`
- `max(a, b) == max_by(a, b, cmp)`
- `min(a, b) == min_by(a, b, cmp)`
- `a.clamp(min, max)` возвращает `min` если `self < min`, `max` если `self > max`, иначе `self`

#### Зачем нужен Ord? #Q

- 🔢 NaN проблема - не все числа сравнимы
- 🎯 Частичные порядки в математике
- 📊 Многомерные данные - точки могут быть несравнимы
- 🛡️ Типобезопасность - компилятор не даст сломать логику
- 🎨 Гибкость - кастомная логика сравнения

#### Reverse

```rust
// helper struct for reverse ordering
pub trait Reverse<T> (put T);

// Example
let mut v = vec![1, 2, 3, 4, 5, 6];
v.sort_by_key(|&k| (k > 3, Reverse(k)));
assert_eq!(v, vec![3, 2, 1, 6, 5, 4]);
```

### New Type Idiom

- Фундаментальная идиома в Rust для создания новых типов на основе существующих.
- Мы создаем новый тип, обертывая существующий тип в tuple struct.

```rust
pub struct Years(i64);
pub struct Days(i64);

impl Years {
    pub fn to_days(&self) -> Days {
        Days(self.0 * 365) // New type is a tuple
    }
}

impl Days {
    pub fn to_years(&self) -> Years {
        Years(self.0 / 365)
    }
}

pub struct Example<T> (i32, i64, T); 
```

### Hasher

- Generic trait для любой структуры конорая может хеширрвать объкты из их байтовой репрезентации.
- Можно хешировать произвольные наборы байт => можно работать с любыми типами.

```rust
pub trait Hasher {
    fn finish(&self) -> u64;
    fn write(&mut self, bytes: &[u8]);

    fn write_u8(&mut self, i: u8) {...}
    fn write_u16(&mut self, i: u16) {...}
    fn write_u32(&mut self, i: u32) {...}
    fn write_u64(&mut self, i: u64) {...}
    fn write_usize(&mut self, i: usize) {...}
    fn write_i8(&mut self, i: i8) {...}
    // ...
}
```

**Пример использования:**

```rust
use std::collections::hash_map::DefaultHasher;
use std::hash::Hasher;

let mut hasher = DefaultHasher::new();
hasher.write_u32(1989);
hasher.write_u8(11);
hasher.write_u8(9);

hasher.write_u8(b"Huh?");
// `b` перед строковым литералом &str – пометка компилятору, что это байтовый срез &[u8].

println!("Hash: {:x}", hasher.finish());
// Hash: 24r3g2r3gr2gr2rgrw
```

### Hash

- `#[derive(Hash)]`
- Трейт означает что тип может быть хеширован.

```rust
pub trait Hash {
    fn hash<H: Hasher>(&self, state: &mut H)
    where
        H: Hasher;
    
    fn hash_slice<H: Hasher>(data: &[Self], state: &mut H)
    where
        H: Hasher;
    {...}
}
```

```rust
struct Person {
    id: u32,
    name: String,
    phone: u64,
}

impl Hash for Person {
    fn hash<H: Hasher>(&self, state: &mut H) {
        self.id.hash(state);
        // name is ignored
        self.phone.hash(state);
    }
}
```

Имплементирвоание `Hash` должно быть консистентно с `Eq`.

`k1 == k2 => hash(k1) == hash(k2)`

`HashMap` и `HashSet` полагаются на этот принцип.

### Drop

Запуск кастомного кода с деструктором.

```rust
pub trait Drop {
    fn drop(&mut self); // Принимает &mut self тк 
}
```

Имплементация ручного деструктора:

```rust
struct HasDrop;

impl Drop for HasDrop {
    fn drop(&mut self) {
        println!("Dropping HasDrop");
    }
}

struct HasTwoDrops {
    a: HasDrop,
    b: HasDrop,
}

impl Drop for HasTwoDrops {
    fn drop(&mut self) {
        println!("Dropping HasTwoDrops");
    }
}

let _x = HasTwoDrops { a: HasDrop, b: HasDrop };

// Dropping HasTwoDrops
// Dropping HasDrop
// Dropping HasDrop
```

- Обратный порядок создания

```rust
let _first = Resource { name: "First".to_string() };
let _second = Resource { name: "Second".to_string() };
let _third = Resource { name: "Third".to_string() };

// Dropping Third ← Последний созданный
// Dropping Second
// Dropping First ← Первый созданный
```

- Поля структур

```rust
let _outer = Outer {
        inner1: Inner { name: "Inner1".to_string() },
        inner2: Inner { name: "Inner2".to_string() },
    };

// Dropping outer   ← Сначала сама структура
// Dropping Inner2  ← Потом поля в обратном порядке
// Dropping Inner1
```

### ManuallyDrop

`ManuallyDrop` - это обертка над типом, которая позволяет избежать вызова деструктора для этого типа.

```rust
pub struct ManuallyDrop<T>
    where
        T: ?Sized,
    {
        pub value: T,
    }
```

Методы:

```rust
fn new(value: T) -> ManuallyDrop<T>;
fn into_inner(val: ManuallyDrop<T>) -> T;
unsafe fn take(val: &mut ManuallyDrop<T>) -> T;
unsafe fn drop(val: &mut ManuallyDrop<T>);
```


### ❓Для чего используется трейt `Drop`?❓#Q


### Add

Имплементация оператора `+` для типов.

```rust
pub trait Add<Rhs = Self> {
    type Output;
    fn add(self, rhs: Rhs) -> Self::Output;
}
```

```rust
struct Point {
    x: T,
    y: T,
}

impl<T: Add<Output = T>> Add for Point<T> {
    type Output = Self;

    fn add(self, rhs: Self) -> Self::Output {
        Self {
            x: self.x + rhs.x,
            y: self.y + rhs.y,
        }
    }
}
```

### AddAssign

Имплементация оператора `+=` для типов.

```rust
pub trait AddAssign<Rhs = Self> {
    fn add_assign(&mut self, rhs: Rhs);
}

```rust
struct Point {
    x: i32,
    y: i32,
}

impl<T: AddAssign> AddAssign for Point<T> {
    fn add_assign(&mut self, rhs: Self) {
        *self = Self { // в `*self` мы записываем новый экземпляр структуры
            x: self.x + rhs.x,
            y: self.y + rhs.y,
        };
    }
}
```

### Operator overloading

- Add, Sub, Mul, Div, Rem
- Shl, Shr
- BitAnd, BitOr, BitXor
- Not, Neg (Не могут иметь assign)

### Index, IndexMut

#### IMMUTABLE contexts - `container[index]`

```rust
pub trait Index<Idx>
where
    Idx: ?Sized,
{
    type Output: ?Sized;
    fn index(&self, index: Idx) -> &Self::Output;
}
```

`index` разрешает паниковать если выход за границы.

```rust
enum Nucleotide {
    A,
    C,
    G,
    T,
}

struct NucleotideCount {
    a: usize,
    c: usize,
    g: usize,
    t: usize,
}

impl Index<usize> for NucleotideCount {
    type Output = usize;

    fn index(&self, nucleotide: Nucleotide) -> &Self::Output {
        match nucleotide {
            Nucleotide::A => &self.a,
            Nucleotide::C => &self.c,
            Nucleotide::G => &self.g,
            Nucleotide::T => &self.t,
        }
    }
}
```

#### MUTABLE contexts - `container[index]`

```rust
pub trait IndexMut<Idx>: Index<Idx>
where
    Idx: ?Sized,
{
    fn index_mut(&mut self, index: Idx) -> &mut Self::Output;
}
```

`index_mut` разрешает паниковать если выход за границы.

```rust
struct Test {
    x: usize,
}

impl Index<usize> for Test {
    type Output = usize;

    fn index(&self, index: usize) -> &Self::Output {
        println!("Indexing");
        &self.x
    }
}
```

### Read, Write

```rust
pub trait Read {
    fn read(&mut self, buf: &mut [u8]) -> Result<usize>;

    fn read_to_end(&mut self, buf: &mut Vec<u8>) -> Result<usize> {}
    fn read_to_string(&mut self, buf: &mut String) -> Result<usize> {}
    fn read_exact(&mut self, buf: &mut [u8]) -> Result<()> {}
}

pub trait Write {
    fn write(&mut self, buf: &[u8]) -> Result<usize>;
    fn flush(&mut self) -> Result<()> {}

    fn write_all(&mut self, buf: &[u8]) -> Result<()> {}
    fn write_fmt(&mut self, fmt: fmt::Arguments<'_>) -> Result<()> {}
}
```

We can read from File, TcpStream, Stdin, &[u8] and more.

### BufRead, BufReader, BufWriter

**BufRead** - трейт для буферизованного чтения с методами `read_line()`, `lines()`, `read_until()`.

**BufReader** - обертка для буферизованного чтения из любого `Read`:

```rust
use std::io::{BufRead, BufReader};
use std::fs::File;

let file = File::open("file.txt")?;
let reader = BufReader::new(file);
for line in reader.lines() {
    println!("{}", line?);
}
```

**BufWriter** - обертка для буферизованной записи в любой `Write`:

```rust
use std::io::{BufWriter, Write};
use std::fs::File;

let file = File::create("file.txt")?;
let mut writer = BufWriter::new(file);
writer.write_all(b"Hello")?;
writer.flush()?; // Принудительная запись буфера
```

### Display, Debug

**Display** - для пользовательского отображения (`{}` в println!):

```rust
impl fmt::Display for Person {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "{} ({})", self.name, self.age)
    }
}
```

**Debug** - для отладочного вывода (`{:?}` в println!):

```rust
#[derive(Debug)]
struct Person { name: String, age: u32 }

// Или вручную:
impl fmt::Debug for Person {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        f.debug_struct("Person")
            .field("name", &self.name)
            .field("age", &self.age)
            .finish()
    }
}
```

### ToString

**ToString** - трейт для преобразования в `String`. Автоматически реализуется для типов с `Display`:

```rust
let num = 42;
let s = num.to_string(); // "42"

// Любой тип с Display получает ToString бесплатно
struct Point(i32, i32);
impl fmt::Display for Point {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "({}, {})", self.0, self.1)
    }
}
// Point теперь имеет метод to_string()
```

### Defer, DeferMut

**Deref** - позволяет использовать тип как ссылку на целевой тип (`*` и автоматическое разыменование):

```rust
impl Deref for MyWrapper {
    type Target = String;
    fn deref(&self) -> &String { &self.0 }
}
let wrapper = MyWrapper("hello".to_string());
println!("{}", wrapper.len()); // Вызывает String::len()
```

**DerefMut** - для изменяемых ссылок:

```rust
impl DerefMut for MyWrapper {
    fn deref_mut(&mut self) -> &mut String { &mut self.0 }
}
let mut wrapper = MyWrapper("hello".to_string());
wrapper.push_str(" world"); // Вызывает String::push_str()
```

### Operator `.`

**Dot operator** - автоматическое разыменование и поиск методов:

1. Сначала ищет метод для `T`
2. Потом для `&T`
3. Потом для `&mut T`
4. Потом применяет `Deref` и повторяет поиск
5. Автоматически добавляет `&` или `&mut` если нужно

```rust
let s = String::from("hello");
let len = s.len();        // Вызывает String::len(&self)
let len = (&s).len();     // То же самое
let len = (*&s).len();    // И это тоже

// Через умные указатели
let boxed = Box::new(String::from("hello"));
let len = boxed.len();    // Автоматически разыменовывается
```

---

# Interior mutability

Memory safety правило: для объекта T возможно только одно из:

- Иметь несколько immutable references (&T) – aliasing
- Иметь один mutable reference (&mut T) – mutability

Иногда нужно мутировать объект имея несколько alias'ов `&T`.

Примеры:

- Modifying `Rc` (reference counting)
- `Atomic`
- `Mutex`
- `RwLock`

Любое изменение через `&` это interior mutability.

Далее мы рассматривает однопоточное изменение.

### Cell

`Cell<T> - это тип для interior mutability (внутренней изменяемости) в Rust, который позволяет изменять данные через неизменяемую ссылку &T

Safe abstracton over unsafe modification

```rust
pub struct Cell<T: ?Sized> {}

impl<T: Copy> Cell<T> {}
```

`Сell` копирует значение внутри.

```rust
fn new(value: T) -> Cell<T>;
fn set(&self, value: T);

// Только когда `T: Copy`
fn get(&self) -> T;
```

#### Почему `Cell` требует `Copy`, а не `Clone`? #Q

- Copy гарантирует побитовое копирование
- Предотвращение циклических ссылок (например `&Cell<Option<&Cell<Option<...>>>>`)
- Атомарность операций
- Copy типы НИКОГДА не паникуют при копировании

Здесь используется `Copy` trait, а не `Clone`. Т.к. `Clone` позволяет записать любую логику внутри, есть шанс создать memory unsafety и undefined behavior.

```rust
stuct BadClone<'a> {
    data: i32,
    pointer: &'a Cell<Option<BadClone<'a>>>
}

//'a - это lifetime parameter для `BadClone`
//ссылка живет столько же сколько и `BadClone`

impl<'a> Clone for BadClone<'a> {
    fn clone(&self) -> BadClone<'a> {
        // Получаем ссылку на наши внутренние данные
        let data: &i32 = &self.data;
        println!("before: {}", *data);
        
        // Очищаем ячейку, на которую указывает `pointer`
        self.pointer.set(None);

        // Выводим значение после очистки – не изменится
        println!("after: {}", *data);

        // Возвращаем новый `BadClone` с теми же данными
        BadClone { data: *data, pointer: self.pointer }
    }
}
```

```rust
let cell = Cell::new(None);
cell.set(Some(BadClone {
    data: 123,
    pointer: &cell,
}));

let bad_clone = cell.get();

// before: 123
// after: 0 // UB
```

Cell с Clone **unsound**. **Unsound** - абстракция не является safe.

Sound = безопасная абстракция, которая правильно поддерживает инварианты Rust.

Баг называется **reentrancy**. Изменяем данные во время их клонирования.

### RefCell

`RefCell` отличается от `Cell` тем, что он возвращает ссылки. Динамическая проверка безопасности.

Самые важные методы:

```rust
fn new(value: T) -> RefCell<T>;
fn get_mut(&mut self) -> &mut T;

fn borrow(&self) -> Ref<T>; // Паникует при ошибке
fn borrow_mut(&self) -> RefMut<T>; // Паникует при ошибке

fn try_borrow(&self) -> Result<Ref<T>, BorrowError>; // НЕ паникует
fn try_borrow_mut(&self) -> Result<RefMut<T>, BorrowMutError>; // НЕ паникует
```

Структуры `Ref` и `RefMut` прикрепляются к `RefCell` и изменяют внутренние счетчики ссылок когда объекты удаляются.

Default и `try_` варианты отличаются тем, что `try_` варианты возвращают `Result`, а Default паникуют.

### Cell, RefCell, Rc

Частый паттерн - использовать `Cell` и `RefCell` вместе с `Rc`.

```rust
pub sruct List<T> {
    head: Link<T>,
    tail: Link<T>,
}

type Link<T> = Option<Rc<RefCell<Node<T>>>>;
```

#### Как внутри работает Cell и RefCell? – UnsafeCell #Q

Unsafe code:

- Unsafe структуры которая отдает ссылки без проверки счетчиков ссылок
- Оборачивает эту структуру в safe абстракцию

```rust
pub struct UnsafeCell<T: ?Sized> {
    value: T,
}
```

```rust
fn new(value: T) -> UnsafeCell<T>;

// Невозможно dereference pointer без unsafe
fn get(&self) -> *mut T;
fn get_mut(&mut self) -> &mut T;
```

#### Definition of `Cell` and `RefCell`

```rust
pub struct Cell<T: ?Sized> {
    value: UnsafeCell<T>,
}

pub struct RefCell<T: ?Sized> {
    borrow: Cell<BorrowFlag>,
    value: UnsafeCell<T>,
}
```

#### `Cell` and `RefCell` summary

- Невозможно проверить инварианты в compile time при изменении из нескольких мест, они сдвинуты в runtime.
- `Cell` только копирует значение, не может использоваться `Clone`.
- `RefCell` возвращает ссылки на внутренние данные и считает их в runtime.
- В C++ нужно поддерживать владение вручную, аккуратно работать с ссылками.
- Плюсы быстрее и используют меньше памяти.
- Unsafe дает силу и ответственность C и C++.

---

# Cargo и Crates

## Cargo

Cargo - менеджер пакетов для Rust.

- Загружает и управляет зависимостями
- Собирает и компилирует проекты
- Запускает тесты и линтинг
- Создает документацию
- Создает пакеты
- Создает бинарные файлы

## Crate

Единица компиляции.

`cargo new --bin example`

```shell
example
├── Cargo.lock
├── Cargo.toml
└── src
    └── main.rs
```

Crates.io - репозиторий пакетов.

Cargo.toml

```toml
[package]
name = "example"
version = "0.1.0"
edition = "2021"

[dependencies]
clap = "4.5.28"
```

### Типы Crates

- bin - бинарный файл
- lib - "compiler recommended"
- dylib - динамическая библиотека
- staticlib - статическая библиотека
- cdylib - C dynamic library
- rlib - Rust library file
- proc-macro - procedural macro crate

bin и lib достаточно для большинства проектов.

### Crate: versions

Semantic versioning (semver): `MAJOR.MINOR.PATCH`

- MAJOR - breaking changes, incompatible API changes
- MINOR - new features, backwards compatible
- PATCH - backwards compatible bug fixes

TBD: Lecture 4

## Modules

TBD: Lecture 4

### Как организована модульная система в Rust? #Q

- **Модули (`mod`)**
	- *Модули позволяют группировать связанные функции, структуры и типы.*
	- *По умолчанию элементы внутри модуля **приватные** (видны только в пределах текущего модуля).*
- **Крейты (`crate`)**
	- Единица компиляции. Может быть либо бинарной программой, либо библиотекой.
- **Пакеты (`package`)**
	- Набор файлов и папок, управляемый Cargo (`Cargo.toml`).
- **Пути (`use`, `pub`)**
	- pub делает элемент модуля доступны вне модуля
	- use позволяет удобно импортировать элементы из модулей
```rust
my_project/
├── Cargo.toml
└── src/
    ├── main.rs        (бинарный крейт)
    └── lib.rs         (библиотечный крейт)
```
# Iterators and Closures - Итераторы и Замыкания

## Iterator Trait

В Rust итераторы - это трейт.

```rust
pub trait Iterator {
    type Item;
    fn next(&mut self) -> Option<Self::Item>;
}
```

Если возвращается `None`, то итератор закончился.

More Methods:

```rust
pub trait Iterator {
    type Item;
    fn next(&mut self) -> Option<Self::Item>;

    fn size_hint(&self) -> (usize, Option<usize>);
    fn last(&mut self) -> Option<Self::Item>;
    fn enumerate(&mut self) -> Enumerate<Self>;
    fn peekable(&mut self) -> Peekable<Self>;
    fn map<B, F>(self, f: F) -> Map<Self, F>
    fn lt<I>(self, other: I) -> bool;

    // Total 71 methods
}
```

Возвращается сам Item (не ссылка).

### &self methods

`size_hint` - возвращает диапазон итераций. Используется для оптимизации.

### self methods

- `last` - возвращает последний элемент.
- `enumerate` - возвращает индекс и элемент, `(index, value)`.
- `peekable` - возвращает итератор, который может посмотреть на следующий элемент без его извлечения. Имеет `peek()` метод. Возвращает `Option<&Self::Item>`.
- `count` - возвращает количество элементов.
- `map` - применяет функцию к каждому элементу.
- `lt` - сравнивает два итератора.
- `and more...`

### &mut self methods

- `map` - возвращает новый итератор, который применяет функцию к каждому элементу. Lazy operation.
- `by_ref` - возвращает ссылку `&mut Self` на итератор, позволяет вызывать `next` без ownership.
- `nth` - возвращает n-ый элемент.
- `nth_back` - возвращает n-ый элемент с конца.
- `all` - проверяет, все ли элементы удовлетворяют условию.

### `IntoIterator` trait

Абстракции над объектами могут быть преобразованы в итераторы.

```rust
pub trait IntoIterator {
    type Item;
    type IntoIter: Iterator<Item = Self::Item>;
    fn into_iter(self) -> Self::IntoIter;
}
```

Берем на вход self и возвращаем итератор.

```rust
let vec = vec![1, 2, 3];
let into_iter = vec.into_iter();

// let first = vec[0]; // ❌
let first = into_iter.next(); // ✅
```

`IntoIterator` используется для `for` цикла.

```rust
// Синтаксический сахар
for v in vec{
    // do something
}

// В явном виде это:
let mut __into_iter = vec.into_iter();
while let Some(v) = __into_iter.next() {
    // do something
}
```

#### Итерирование по ссылкам и по mutable ссылкам #Q

```rust
let vec = vec![1, 2, 3];
for v in &vec { // v имеет тип &i32
    println!("{}", v);  // Читаем через ссылку
}
// vec остается доступным после цикла

for v in &mut vec { // v имеет тип &mut i32
    *v *= 2;  // Изменяем через ссылку
}
// vec = [2, 4, 6]
```

`IntoIterator` имплементирован для `&Vec`, `&mut Vec`. Возвращает тоже самое что `.iter()` и `.iter_mut()`.

Ключевое отличие:

- `&vec` - не забирает владение, `vec` остается доступным
- `vec` - забирает владение, `vec` больше нельзя использовать

```rust
let vec = vec![1, 2, 3];
for v in &vec { /* */ }  // ✅ vec еще можно использовать
for v in vec { /* */ }   // ❌ vec перемещен, больше недоступен
```

### Range

`a..b` синтаксис - это синтаксический сахар для `Range`, `RangeFrom`, `RangeTo`, `RangeFull`.

- `Range` и `RangeFrom` - итераторы.
- `RangeTo` и `RangeFill` - используются для `matching`

```rust
for i in 0..10 {
    
}

for i in 0.. {

}
```

### `FromIterator` trait

Можно конвертирваоть из итератора в объект.

```rust
pub trait FromIterator<A> {
    fn from_iter<T>(iter: T) -> Self
    where 
        T: IntoIterator<Item = A>
}
```

Это создаст новую коллекцию `A` из итератора `T`.

```rust
let vec = vec![(0,1), (1, 2)];
let map: HashMap<i32, i32> = HashMap::from_iter(vec);
```

`.collect()` - это метод ожидает коллекцию, которая может быть собрана из итератора с таким `Item`.

```rust
fn collect<B: FromIterator<Self::Item>>(self) -> B
where
    Self: Sized,
{
   // ...
}
```

`FromIterator` имплементирован для `Result`, `Option` и `()` (unit).

### FromIterator trait: Result

```rust
impl<A, E, V> FromIterator<Result<A, E>> for Result<V, E>
where
    V: FromIterator<A>,
{
    // ...
}

// example
let integers: Vec<&str> = vec!["0", "17", "2", "42"];
let res: Result<Vec<u32>, ParseIntError> = integers
    .into_iter()
    .map(|s| s.parse::<u32>())
    // ^- impl Iterator<Item = Result<u32, ParseIntError>>
    .collect();

assert_eq!(res, Ok(vec![0, 17, 2, 42]));
```

### FromIterator trait: Option

```rust
impl<A, V> FromIterator<Option<A>> for Option<V>
where
    V: FromIterator<A>,
{
    // ...
}

// example
let v: Vec<u32> = vec![1, 2, 11, 12];
let res: Option<Vec<u32>> = v.into_iter()
    .map(|x| x.checked_sub(1)).collect();
assert_eq!(res, Some(vec![0, 1, 10, 11]));

let v: Vec<u32> = vec![1, 2, 0, 12];
let res: Option<Vec<u32>> = v.iter()
    .map(|x| x.checked_sub(1))
    .collect();

assert_eq!(res, None);
```

### FromIterator trait: ()

```rust
impl FromIterator<()> for () {/*...*/}

// example
let data = vec![1, 2, 3, 4, 5];
let res: io::Result<()> = data
    .into_iter()
    .map(|x| writeln!(io::stdout(), "{}", x))
    .collect(); // итератор по Result<()> преобразуется в Result<()>

assert_eq!(res.is_ok());
```

### `ExactSizeIterator` trait 

Итератор точно знает свою длину.

```rust
pub trait ExactSizeIterator: Iterator {
    fn len(&self) -> usize {/*...*/}
    fn is_empty(&self) -> bool {/*...*/}
}
```

Этот трейт safe, но не может гарантирвоать что возвращаемся длина верна. Unsafe код не может использовать этот трейт.

### `DoubleEndedIterator` trait

Итератор, который может быть проитерирован с обоих концов.

```rust
pub trait DoubleEndedIterator: Iterator {
    fn next_back(&mut self) -> Option<Self::Item>;

    fn advance_back_by(&mut self, n: usize) -> Result<(), usize>;  
    fn nth_back(&mut self, n: usize) -> Option<Self::Item>;
    fn try_rfold<B, F, R>(&mut self, init: B, f: F) -> R;
    fn rfold<B, F>(self, init: B, f: F) -> B;
    fn rfind<P>(&mut self, predicate: P) -> Option<Self::Item>;

    // example
    let data = vec![1, 2, 3, 4, 5];
    let mut iter = data.iter();

    assert_eq!(iter.next(), Some(&1));
    assert_eq!(iter.next_back(), Some(&5));

    assert_eq!(iter.next(), Some(&2));
    assert_eq!(iter.next_back(), Some(&4));

    assert_eq!(iter.next(), Some(&3)); // ✅
    assert_eq!(iter.next_back(), None); // ❌ iter закончился
}
```

### Адаптеры итераторов

```rust
let v1: Vec<i32> = vec![1, 2, 3];

// map - преобразование каждого элемента
let v2: Vec<_> = v1.iter().map(|x| x + 1).collect();

// filter - фильтрация элементов
let v3: Vec<_> = v1.iter().filter(|&x| *x > 1).collect();

// enumerate - добавление индекса
for (i, v) in v1.iter().enumerate() {
    println!("{}: {}", i, v);
}

// zip - объединение двух итераторов
let names = vec!["Alice", "Bob", "Carol"];
let ages = vec![25, 30, 35];
for (name, age) in names.iter().zip(ages.iter()) {
    println!("{} is {} years old", name, age);
}
```

### Потребляющие адаптеры

```rust
let v1 = vec![1, 2, 3];

// collect - собирает в коллекцию
let v2: Vec<i32> = v1.iter().cloned().collect();

// fold - аккумулирует значения
let sum = v1.iter().fold(0, |acc, x| acc + x);

// reduce - аналог fold но без начального значения
let sum = v1.iter().cloned().reduce(|acc, x| acc + x);

// find - находит первый подходящий элемент
let found = v1.iter().find(|&&x| x > 1);

// any/all - проверки
let has_even = v1.iter().any(|&x| x % 2 == 0);
let all_positive = v1.iter().all(|&x| x > 0);
```

### Iterators Perfomance

- `size_of` комплексного итератора - это `size_of` его часей.
- Все стейты - это флаги в стеке.
- Компилятор знает какие функции вызываются, он инлайнит вызовы и код работает быстрее.
- Код итераторов векторизуется, например `flatten` оптимизирован под векторизацию.

Векторизация = выполнение одной операции над несколькими элементами одновременно с помощью SIMD инструкций процессора.

```rust
// Высокоуровневый код
let data: Vec<i32> = (0..1000).collect();
let doubled: Vec<i32> = data.iter().map(|x| x * 2).collect();

// Компилятор превращает это в SIMD инструкции!
```

## Closures - Замыкания

```rust
let x = 4;
let equal_to_x = |z| z == x;

let y = 4;
assert!(equal_to_x(y));
```

### ❓Отличия closures от функций❓ #Q

Может использовать переменные из окружения.

| Аспект             | Функции              | Closures                  |
| ------------------ | -------------------- | ------------------------- |
| Синтаксис          | `fn name() {}`       | `                         |
| Захват окружения   | ❌ Нет                | ✅ Да                      |
| Размер             | 0 байт               | Размер захваченных данных |
| Производительность | Zero-cost            | Может быть overhead       |
| Использование      | Независимые операции | Работа с контекстом       |
| Типы               | `fn`                 | `Fn`, `FnMut`, `FnOnce`   |

Можно выводить типы.

```rust
let option = Some(2);

let x = 3;
//explicit types:
let new: Option<i32> = option.map(|val: i32| -> i32 {
    val * x
});

println!("{:?}", new); // Some(6)

let y = 10;
// inferred
let new2 = option.map(|val| val * y);
println!("{:?}", new2); // Some(20)
```

### Closures and traits

Продублируем функциональность `Optiom::map`

```rust
fn map<X, Y>(option: Option<X>, transform: ...) -> Option<Y> {
    match option {
        Some(x) => Some(transform(x)),
        None => None,
    }
}
```

```rust
fn map<X, Y>(option: Option<X>, transform: ...)
//~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~^^^ какой тут тип?

fn map<X, Y>(option: Option<X>, transform: T) -> Option<Y>
where T: /* the trait */ { ... }
```

- Компилятор генерирует структуру, которая имплементирует какой-то трейт.
- У трейта будет только одна фукция
- Входной тип tuple `(X, Y)`

```rust
trait Transform<Input> {
    type Output;
    fn transform(/* self */, input: Input) -> Self::Output;
}

#### Нужно ли `self`, `&mut self` или `&self` в трейтах для Closures? #Q

Self сохраняет текущее окружение и передает его в функцию. Без `self` нет окружения и теряется контекст.

```rust
// Определения трейтов
trait FnOnce<Args> {
    type Output;
    fn call_once(self, args: Args) -> Self::Output;  // ← self
}

trait FnMut<Args>: FnOnce<Args> {
    fn call_mut(&mut self, args: Args) -> Self::Output;  // ← &mut self  
}

trait Fn<Args>: FnMut<Args> {
    fn call(&self, args: Args) -> Self::Output;  // ← &self
}
```

| Self Type | Trait | Description |
|-----------|-------|-------------|
| `&self` | `Fn` | Можно вызывать много раз |
| `&mut self` | `FnMut` | Можно вызывать много раз, но мутабельно |
| `self` | `FnOnce` | Можно вызывать только один раз, можно изменять переменные |

```rust
// имплементируем Map
trait Transform<Input> {
    type Output;
    fn transform(self, input: Input) -> Self::Output;
}

fn map<X, Y, T>(option: Option<X>, transform: T) -> Option<Y>
where T: Transform<X, Output = Y> 
{
    match option {
        Some(x) => Some(transform.transform(x)),
        None => None,
    }
}
```

### Fn, FnMut, FnOnce

```rust
// Определения трейтов
trait FnOnce<Args> {
    type Output;
    fn call_once(self, args: Args) -> Self::Output;  // ← self
}

trait FnMut<Args>: FnOnce<Args> {
    fn call_mut(&mut self, args: Args) -> Self::Output;  // ← &mut self  
}

trait Fn<Args>: FnMut<Args> {
    fn call(&self, args: Args) -> Self::Output;  // ← &self
}
```

| Self Type | Trait | Description |
|-----------|-------|-------------|
| `&self` | `Fn` | Можно вызывать много раз |
| `&mut self` | `FnMut` | Можно вызывать много раз, но мутабельно |
| `self` | `FnOnce` | Можно вызывать только один раз, можно изменять переменные |

Имплементация `map`

```rust
impl<T> Option<T> {
    pub fn map<U, F>(self, f: F) -> Option<U>
    where F: FnOnce(T) -> U
    {
        match self {
            Some(x) => Some(f(x)),
            None => None,
        }
    }
}
```

`FnOnce(T) -> U` - это другое имя для `Transform<X, Output = Y>` ограничения и `f(x)` для transform.transform(x).

### Function pointer `fn`

`fn` это конкретный тип который ссылается на код, не данные. `fn` не захватывает переменные из окружения.

### Closures: Захват переменных

```rust
struct T {...}

fn by_value(_: T) {}
fn by_ref(t: &T) {}
fn by_mut(t: &mut T) {}
```

```rust
let x: T = ...;
let mut y: T = ...;
let mut z: T = ...;

let closure = || {
    by_ref(&x);
    by_ref(&y);
    by_ref(&z);

    // Заставим y и z быть захваченными хотя бы по ссылке &mut 
    by_mut(&mut y);
    by_mut(&mut z);

    // Захватим y и z по значению
    by_value(z);
};
```

---

# System Safety

## Как Rust обеспечивает memory safety без garbage collector? #Q

- Ownership
	- Каждое value имеет единственного владельца
	- Когда владелец выходит из скоупа, память очищается
	- Ownership может быть перемещено, но не может дублироваться неявно
- Borrowing & References – одновременно может быть либо
	- Несколько неизменяемых ссылок `&T` 
	- Одна мутабельная ссылка `&mut T`
- Lifetimes
	- Компилятор анализирует сколько живет value
	- Компилятор гарантирует что ссылка никогда не outlive данные на которые ссылается
	  
Принципы Rust:
- Performance: Fast and memory efficient
- Reliability (Memory Safety, Thread Safety, Type Safety)
- Productivity: Overall speed of development

## Aliasing

- Aliasing = когда несколько указателей/ссылок ссылаются на одну и ту же область памяти
- Rust не позволяет создать aliasing для mutable ссылок

```rust
fn compute(input: &u32, output: &mut u32) {
    if *input > 10 {
        *output = 1;
    } 
    if *input > 5 {
        *output *= 2;
    }
}
```

Оптимизируем

```rust
fn compute(input: &u32, output: &mut u32) {
    // keep `*input` in a register
    let cached_input = *input;

    if cached_input > 10 {
        *output = 2;
    } else if cached_input > 5 {
        *output *= 2;
    }
}
```

Для других языков этот код небезопасен в случае передачи двух одинаковых ссылок.

В Rust это безопасно, потому что lifetimes гарантируют что две ссылки не могут указывать на одно и то же место в памяти.

`&mut` не может быть aliased, но `&` может. Правило **aliasing XOR mutability - AXM**.

Примеры оптимизаций

- Созранение значений в регистре, при этом нет указателей на эти значения в памяти
- Безопасное чтение из-за гарантий того что в память не было записей с момента чтения
- Безопасная запись из-за гарантий того что в памяти не было чтения с момента записи
- Перемещение или переупорядочивание чтения и записи тк они на зависят друг от друга

## Lifetimes

Lifetimes - именованый регион кода. Может быть с промежутками времени.

Объект живет внутри lifetime, ссылки не могут жить дольше объекта.

```rust
pub struct Person<'a, T> { // 'a - lifetime, структура зависит от lifetime
    pub first_name: &'a str,
    pub last_name: &'a str,
    pub age: usize,
    pub private_property: T,
}
```

Lifetime не является типом, но является частью типа.

В локальных скоупах обычно не нужно указывать lifetimes.

Без синтаксического сахара

```rust
let x = 0;
let y = &x;
let z = &y;

// `'a: {` и `&'b x` это не валидный синтаксис

'a: {
    let x: i32 = 0; // 'x' имеет lifetime 'a
    'b: {
        let y: &'b i32 = &'b x; // 'y' имеет lifetime 'b
        'c: {
            let z: &'c &'b i32 = &'c y; // 'z' имеет lifetime 'c
            //     ~~~~~~~~~~~~
            //     │   │   │
            //     │   │   └── Конечный тип: i32
            //     │   └────── Внутренняя ссылка с lifetime 'b
            //     └────────── Внешняя ссылка с lifetime 'c
        }
    }
}
```

```rust
let x = 0;
let z;
let y = &x;
z = y;

// Desugared:
'a: {
    let x: i32 = 0;
    'b: {
        let z: &'b i32;
        'c: {
            // Нужно использовать 'b здесь, эта ссылка передана
            // в 'c
            let y: &'c &'b i32 = &'c x;
            z = y; // Присваивание: 'c должно быть ≥ 'b
//~~~~~~~~~~^~~~^ lifetime 'c
//~~~~~~~~~~^ lifetime 'b
        }
    }
}
```

Пример с ошибкой

```rust
fn as_str(data: &u32) -> &str {
    let s = format!("{}", data);
    &s // Ошибка – возвращаемая ссылка в локальном скоупе
}

// Desugared:
fn as_str<'a>(data: &'a u32) -> &'a str {
    'b: {
        let s: String = format!("{}", data);
        'c: {
            let s: &'c String = &'c s;
            &'c s
        }
    }
}
```

```rust
let mut data = vec![1, 2, 3];
let x = &data[0]; // &T
data.push(4); // &mut T
// либо один &mut T, либо много &T, здесь мы создаем оба
// нарушение AXM(aliasing XOR mutability)
println!("{}", x);

// Desugared:
'a: {
    let mut data: Vec<i32> = vec![1, 2, 3];
    'b: {
        let x: &'b i32 = Index::index::<'b>(&'b data, 0); // &T
        'c: {
            Vec::push(&'c mut data, 4); // создается &mut data – ОК
        }
        println!("{}", x); // Ошибка – x ссылается на data[0] который уже не существует
    }
}
```

```rust
let mut data = vec![1, 2, 3];
let x = &data[0]; // &T - валидный lifetime
if some_condition() {
    println!("{}", x); // последнее использование x, здесь заканчивается lifetime x
    data.push(4); // &mut data – новая область lifetime – ✅ Безопасно - x "умерла"
} else {
    // нет использования x
    data.push(5); // &mut data – ✅ Безопасно - x не мешает
}
```

### Дырки в Lifetime

```rust
let mut data = vec![1, 2, 3];
let mut x = &data[0]; // x: &i32 ───┐
                      //            │ Lifetime #1
println!("{}", x);    // ───────────┘ (заканчивается)
                      //
                      // ← Дырка в lifetime
                      //
data.push(4);         // Безопасная мутация data
                      //
x = &data[3];         // x: &i32 ───┐ 
                      //            │ Lifetime #2
println!("{}", x);    // ───────────┘ (заканчивается)
```

### Lifetime elision (3 правила)

Rust автоматически выводить lifetimes в простых случаях, чтобы не заставлять программиста писать их явно.

1) Каждый параметр получает свой lifetime

```rust
// Что мы пишем:
fn foo(x: &str, y: &str) -> &str

// Что компилятор понимает:
fn foo<'a, 'b>(x: &'a str, y: &'b str) -> &str
```

2)Если есть только один input lifetime, он используется для всех output lifetimes

```rust
// Что мы пишем:
fn first_word(s: &str) -> &str

// Что компилятор понимает:
fn first_word<'a>(s: &'a str) -> &'a str
```

3) Если есть &self или &mut self, его lifetime используется для всех output lifetimes

```rust
// Что мы пишем:
impl MyStruct {
    fn get_name(&self) -> &str
}

// Что компилятор понимает:
impl MyStruct {
    fn get_name<'a>(&'a self) -> &'a str
}
```

### Возможные ошибки с lifetime

Компилируется но семантически некорректный код

```rust
struct ByteIter<'remainder> {
    remainder: &'remainder [u8]
}
impl<'remainder> ByteIter<'remainder> {
    fn next<'rself>(&'rself mut self) -> Option<&'rself u8> {
        if self.remainder.is_empty() {
            None
        } else {
            let byte = &self.remainder[0];
            self.remainder = &self.remainder[1..];
            Some(byte)
        }
    }
}
```

Код может работать:

```rust
let mut bytes = ByteIter { remainder: b"1123" };
let byte_1 = bytes.next(); // lifetime 'remainder
let byte_2 = bytes.next(); // lifetime 'remainder – ошибка

if byte_1 == byte_2 { ... }

// Ошибка компиляции:
// borrow bytes more than once
// когда дважды вызван next вернулось два next с одинаковыми lifetime
```

### Lifetime annotations

```rust
// Простая аннотация lifetime
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

// Структуры с lifetime
struct ImportantExcerpt<'a> {
    part: &'a str,
}

impl<'a> ImportantExcerpt<'a> {
    fn level(&self) -> i32 {
        3
    }
    
    fn announce_and_return_part(&self, announcement: &str) -> &str {
        println!("Attention please: {}", announcement);
        self.part
    }
}
```

### Static lifetime

```rust
// 'static lifetime - живёт всё время программы
let s: &'static str = "I have a static lifetime.";

// Функции с static lifetime
fn get_static_str() -> &'static str {
    "static string"
}
```
### Lifetime bounds

```rust
use std::fmt::Display;

// Lifetime bounds с generic типами
fn longest_with_an_announcement<'a, T>(
    x: &'a str,
    y: &'a str,
    ann: T,
) -> &'a str
where
    T: Display,
{
    println!("Announcement! {}", ann);
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

### ❓Что такое время жизни `'static` и `&'static` в Rust❓ #Q



## Higher-Rank Trait Bounds (HRTBs)

Это способ выразить, что трейт должен быть реализован для ВСЕХ возможных lifetimes, а не для какого-то конкретного.

```rust
fn example<'a, F>(f: F) 
where 
    F: Fn(&'a str) -> &'a str  // F работает с КОНКРЕТНЫМ lifetime 'a
{
    // код
}
```

```rust
fn example<F>(f: F) 
where 
    F: for<'a> Fn(&'a str) -> &'a str  // F работает с ЛЮБЫМ lifetime
{
    // код
}
```

### `T`, `&T`, `&mut T`

Распространенная ошибка считать что `T` это владеющий тип.

```rust
impl<T> Trait for T {}
impl<T> Trait for &T {}
impl<T> Trait for &mut T {}
```

Эти типы сматчатся на следующих impl'ах:

| T                    | &T          | &mut T              |
| -------------------- | ----------- | ------------------- |
| i32, &i32, &mut i32  | &i32, &&i32 | &mut i32, &mut &i32 |
| &&i32, &mut &mut i32 | &&mut i32   | &mut &mut i32       |

- T это супермножество для &T и &mut T
- &T и &mut T это непересекающиеся множества

### Unbounded Lifetime

Лайфтайм который валиден на протяжении всей программы.

```rust
let static_str: &'static str = "Hello, world!";
```

### `T: a` и `&'a T`

Лайфтаймы для типов и ссылок на типы.

```rust
impl<'a> Trait for MyType 
where
    Self: 'a,
{
    /* ... */
}
```

Это означает что тип должен жить лайфтайм `'a`, то есть все лайфтаймы типа `T` переживают `'a`.

### ❓Что значит `T: 'static` ❓ #Q

- T не имеет лайфтайм ограничений
- Например владеющие типы как `i32`, `Vec<String>`, `static VALUE: str = "hello"` не имеют ограничений на лайфтайм
- Можно дропать в любой момент времени

`T: 'a` - множество всех типов который переживают `'a`. `'a` включает ссылки `&'a T`

## Subtyping

В Rust есть наследование интерфейсов(поведения)

**Subtyping** - это концепт означает что один объект как минимум на столько же полезен как и другой.

```rust
trait Animal {
    fn snuggle(&self);
}

trait Cat: Animal {
    fn meow(&self);
}

trait Dog: Animal {
    fn bark(&self);
}
```

```rust
fn love(pet: &dyn Animal) {
    pet.snuggle();
}

struct SomeCat;
impl Animal for SomeCat {}
impl Cat for SomeCat {}

let cat = SomeCat;
love(&cat);
```

### Subtyping и лайфтайм.

```rust
fn shortest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() < y.len() {x} else {y}
}

let a = "hello";
let b = String::from("students");
println!("{}", shortest(&a, &b));
```

Лайфтаймы ведут себя как типы и относятся к подтипам. Подтипы - главня причина существования лайфтаймов.

`'static` явлфется подтипом `'a`

### `'a: b`

Мы можем потребоать чтобы лайвтайм не строко переживал другой лайвтайм.

```rust
fn foo<'a, 'b: 'a>(one: &'b str, two: &'a str) -> &'a str {
    one
}

fn main() {
    let string_one = "foo".to_string(); // ← 'b начинается здесь
    {
        let string_two = "bar".to_string(); // ← 'a начинается здесь
        println!("{}", foo(&string_one, &string_two));
        //                     ↑'b          ↑'a
    } // ← 'a заканчивается здесь
} // ← 'b заканчивается здесь

// 'b: |========================|  (string_one живет дольше)
// 'a:      |==========|           (string_two живет меньше)
```

## Variance

- `&'static T` - это подтип `&'a T` – ДА
- `&'a T` - это подтип `&'a U`, где `T` это подтип `U` – ДА
- `&'static mut T` - это подтип `&'a mut T` - ДА
- `&'a mut T` - это подтип `&'a mut U`, где `T` это подтип `U` - НЕТ!!!

Не интуитивно но!

Возьмем `&mut Vec<&'a str>` и подадим как вектор `&mut Vec<&'static str>`.

В таком случае можно положить ссылку на вектор которая живет меньше. Так нельзя.

```rust
fn invariant<'a>(vec: &mut Vec<&'a str>, s: &'a str) {
    vec.push(s);
}

// Нормально работает
let s = String::from("some &'a str");
let mut vec = vec![];
invariant(&mut vec, s.as_str());

// Не скомпилируется
let mut vec = vec!["some &'static str"];
{
    let s = String::from("some &'a str"); // 'a - короткий lifetime
    invariant(&mut vec, s.as_str()); // Пытаемся передать Vec<&'static str> как &mut Vec<&'a str>
}
// строка уже очищена
println!("{vec:?}");
```

`&mut T` нельзя приводить к подтипу.

**Variance** - это концепт который описывает как типы ведут себя в отношении подтипов.
- Covariance - если `T` это подтип `U`, то `&T` это подтип `&U`
- Contravariance - если `T` это подтип `U`, то `&mut T` это подтип `&mut U`
- Invariance - если `T` это подтип `U`, то `&T` и `&mut T` не являются подтипами `&U` и `&mut U`

### Covariance
- Можно использовать подтип вместо супертипа.
- `&'a T` это ковариант в `'a`. `&'a T` так же ковариант в `T`.
- Можно подать `&Vec<&'static str>` в функцию которая принимает `&Vec<&'a str>`.

```rust
fn covariant(x: &dyn Cat) { ... }

covariant(&domestic_cat);
covariant(&cat);

// не скомпилируется
// covariant(&animal);
```

### Invariance
- Обязательно продоставить точный тип.
- `&'a mut T` - пример инвариантного типа.

```rust
fn invariant<'a>(vec: &mut Vec<&'a str>, s: &'a str) {
    vec.push(s);
}

// Не скомпилируется
let mut vec = vec!["some &'static str"];
{
    let s = String::from("some &'a str");
    invariant(&mut vec, s.as_str());
}
println!("{vec:?}");
```

### Contravariance

- Можно использовать супертип вместо подтипа.
- `fn(T) -> U` - это контравариант в `T`. Можно использовать любой супертип вместо T и ковариант в U.
- Единственный источник контравариантности в Rust - это аргументы входные функций.

```rust
fn contravariant<'a>(f: fn(&'static str)) {}
    
fn f1(s1: &'static str) {}
fn f2<'a>(s1: &'a str) {}

contravariant(f1);
contravariant(f2);

// Заметим что
// &'static str <: &'a str
// fn(&'a str) <: &'static str
```

### Таблица вариантностей

| Type                          | Variance in `'a` | Variance in `T` |
| ----------------------------- | ---------------- | --------------- |
| `&'a T`                       | covariant        | covariant       |
| `&'a mut T`                   | covariant        | invariant       |
| `*const T`                    |                  | covariant       |
| `*mut T`                      |                  | invariant       |
| `[T]` and `[T; n]`            |                  | covariant       |
| `fn() -> T`                   |                  | covariant       |
| `fn(T) -> ()`                 |                  | contravariant   |
| `std::cell::UnsafeCell<T>`    |                  | invariant       |
| `std::marker::PhantomData<T>` |                  | covariant       |
| `dyn Trait<T> + 'a`           | covariant        | invariant       |

Какая вариантность типа `&'a (dyn Trait<'b> + 'c)`?

`&'a (dyn Trait<'b> + 'c)`

`'a` - covariant
`'b` - invariant
`'c` - covariant

### `dyn Trait<T> + 'a`

`dyn Trait` это очистка типа. Поэтому `dyn Trait<T> + 'a` имеет ассоциированный лайфтайм.

```rust
let s = my_string.as_str();
let b: Box<dyn Trait> = Box::new(s);
return b; // OOPS
```

### Почему код не компилируется? #Q

```rust
fn evil_feeder<T>(input: &mut T, val: T) {
    *input = val;
}

fn main() {
    let mut mr_snuggles: &'static str = "Mr. Snuggles";
    {
        let spike = String::from("Spike");
        let spike_str: &str = spike.as_str();
        evil_feeder(&mut mr_snuggles, spike_str);
        //          ^^^^^^^^^^^^^^^^ это &mut &'static str
        //                           ^^^^^^^^^^^ но spike_str имеет тип &str
        // &mut T инвариантен по T, поэтому типы должны точно совпадать
    }
}
```

### ❓Почему ссылку типа `&'static str` можно передать в функцию, ожидающую `&str`❓ #Q 

- Тип `&'a str` является **ковариантным по `'a`** – супертип можно использовать вместо подтипа
- `&'static str`имеет больше лайфтайм чем `&str`
- Rust допускает сокращение лайфтайм при передаче в функции
## Variance и unsafe

Variance существенно при написании unsafe кода.

```rust
struct MyCell<T> {
    value: T,
}

impl<T: Copy> MyCell<T> {
    fn set(&self, new_value: T) {
        // pub usafe fn write<T>(dst: *mut T, src: T
        unsafe {
            std::ptr::write(
                &self.value as *const T as *mut T,
                new_value
            );
        }
    }
}

fn foo(rcell: &MyCell<i32>) {
    let val: i32 = 13;
    rcell.set(&val);
    println!("{}", rcell.value);
} // val очищается и внутри Cell невалидный регион памяти

fn main() {
    static X: i32 = 10;
    let cell = MyCell { value: &X };
    foo(&cell);
    println!("end value: {}", cell.value);
}

// foo set value: 13
// end value: 32766 // OOPS

// MyCell это &MyCell<&'static i32>
```

- Мы хранили `&'a i32` ссылку на cell которая может хранить `&'static i32` ссылки
- По умолчанию компиятор такой код не пропускает, но в нашем случае `MyCell<T>` это ковариант в `T`
- Как результат `MyCell` получает ссылку с слишком коротким лайфтаймом внутри
## Drop checker

Drop checker - это механизм который проверяет чтобы все ссылки на объекты были валидны.

```rust
let x;
let y;

// Desugaring to
{
    let x; // drop second
    {
        let y; // drop first
    }
}
```

Переменные дропаются в обратном порядке от их создания.

```rust
let tuple = (vec![], vec![])
```

Для borrow checker оба вектора имеют одинаковый lifetime.

### Structs drop

В структурах дроп происходит в порядке создания полей.

```rust
struct Inspactor<'a>(&'a u8);
struct World<'a>{
    inspector: Option<Inspector<'a>>,
    days: Box<u8>,
}

fn main() {
    let world = World {
        inspector: None,
        days: Box::new(10),
    };
    world.inspector = Some(Inspector(&world.days));
    // все ОК
}
```

```rust
impl<'a> Drop for Inspector<'a> {
    fn drop(&mut self) {
        println!("Inspector dropped after {} days", self.0);
    }
}
```

Код не скомпилируется. Когда Inspector дропается, он пытается взять ссылку на days, но days уже очищенная память.

- sound generic drop

Все аргументы дженерик типов должны переживать свои зависимости. `'a: 'b` означает что `'a` должен переживать `'b`.

## Phantom Data

```rust
struct Vec<T> {
    data: *const T,
    len: usize,
    cap: usize,
    _marker: marker::PhantomData<T>,
}
```

Из-за Drop Checker мы считаем что вектор не владеет никакими значениями. PhantomData притровяется что содержит тип.

PhantomData это ZST (Zero-sized type) чтобы притвориться как T.

---

# Unsafe

Надмножество языка которое позволяет сделать ошибки memory safety и undefined behavior.

Safe rust не позволяет написать весь код который мы хотим.

- `Vec` не может быть реализован полностьюв safe rust. Первая половина вектора инициализирована а вторая нет.
- Оптимизации такие как имплементация `linked list` без runtime оверхеда.
- `split_at_mut` - принимает мутабельную ссылку на слайс и возвращает две мутабельные ссылки.
- Прямое взаимодействие с памятью и железом.

## Функционал unsafe rust

- Разименование указателей
- Вызов unsafe функций(С функции, compiler intrinsics и аллокатор)
- Имплементировать unsafe traits
- Мутрирование `static` переменных
- Доступ к полям `union`

Это все.

## `unsafe` keyword

`get_unchecked` функция в `Vec`:

```rust
pub unsafe fn get_unchecked<I>(&self, index: I) -> &Self::Output
where
    I: SliceIndex<Self>,
{
    unsafe { &*index.get_unchecked(self) }
}
```

Каждая `unsafe` функция имеет контракт который она поддерживает. В данном примере нужно убедиться что `index` находится в границах вектора.

Пример использования `unsafe` функции:

```rust
let v = vec![1];
unsafe {
    let x = v.get_unchecked(0);
}
```

- `unsafe` блок позволяет использовать `unsafe` функции и разименовать указатели.
- каждая `unsafe` функция внутри неявно имеет `unsafe` блок (могут убрать в будущих версиях)

```rust
unsafe trait TrustedOrd: Ord {}
```

`unsafe trait` означает что имплементация данного трейта несет какой-то контракт. `Ord` должен быть корректно имплементированным.

`unsafe` объявляет контракт для пользователя.

## Soundness и Unsoundness

- Sound - код который не может привести к undefined behavior и memory unsafety.
- Unsound - противоположное.

`Unsafe` код unsound по умолчанию.

Рассмотрим пример `split_at_mut` - функция слайса которая принимает мутабельную ссылку на слайс и возвращает две мутабельные ссылки. Нарушает AXM (Aliaing XOR Mutability) rule.

```rust
pub fn split_at_mut(&mut self, mid: usize) -> (&mut [T], &mut [T]) {
    let len = self.len();
    assert!(mid <= len);

    // SAFETY: `[ptr; mid]` and `[mid; len]` are inside `self`, which
    // fulfills the requirements of `split_at_unchecked`.
    unsafe { self.split_at_mut_unchecked(mid) }
}
```

# Pointers - Указатели

`std::ptr` и `std::mem` содержат функции для работы с памятью.

```rust
use std::ptr;

let x: *mut i32 = ptr::null_mut();
if x.is_null() {
    let y: *const i32 = ptr::null();
}
let z: *mut &str = Box::into_raw(Box::new("Hello, world!")); // ссылка на слайс (строку)
unsafe {
    println!("{}", &(*z)[0..2]); // "He"
}
```

## `&*` 

Срывание мутабельность

```rust
let s: *mut & =str = Box::into_raw(Box::new("abc"));
let r: &&str = unsafe { &*s };
```

Разыменовывать указатель и затем создать ссылку на внутреннее значение. Это нарушает AXM rule тк создается две мутабельные ссылки на одно и то же значение – UB.

## Pointers and references

Различия:

- Указатели не указывают на валидные данные все время.
- Ссылки зависят от лафйтайма чтобы отслеживать что они переживают своих родителей.

Указатели используются когда невозможно проверить лайфтайм объекта.

## Pointer arithmetic

Можно делать `.add()` и `.sub()` на указателях для движения указателей в пределах аллокации.

```rust
let arr: [u8; 4] = [0, 1, 2, 3];
let ptr: *const u8 = arr.as_ptr();

unsafe {
    println!("{}", *ptr.add(1)); // 1
    println!("{}", *ptr.sub(2)); // 2
}
```

Что значит в пределах аллокации? #Q

Указатели невалидны если они выходят за пределы аллокации – UB. Компилятор может yдалить код как "мертвый".

```rust
// No optimization
char p[1], q[1] = {0};
uintptr_t ip = (uintptr_t)(p+1);
uintptr_t iq = (uintptr_t)q;
if (ip == iq) {
    *(char*)ip = 10;
    print(q[0]);
}
// OK
```

```rust
// 1 optimization
char p[1], q[1] = {0};
uintptr_t ip = (uintptr_t)(p+1);
uintptr_t iq = (uintptr_t)q;
if (ip == iq) {
    *(char*)(uintptr_t)(p+1) = 10; // <- This line changed - ip заменен на p+1
    print(q[0]);
}
// Not OK - UB
```

```rust
// 2 optimization
char p[1], q[1] = {0};
uintptr_t ip = (uintptr_t)(p+1);
uintptr_t iq = (uintptr_t)q;
if (ip == iq) {
    *(p+1) = 10; // <- This line changed
    print(q[0]);
}
// Not OK - UB
```

```rust
// 3 optimization
char p[1], q[1] = {0};
uintptr_t ip = (uintptr_t)(p+1);
uintptr_t iq = (uintptr_t)q;
if (ip == iq) {
    *(p+1) = 10; // <- This line changed
    print(0);
}
// Этот код никогда не даст такой же результат как и изначальный - UB
```

Какая из оптимизаций некорректна? #Q

Все оптимизации корректны.

Это ошибка разработчика.

`p+1`это one-past-the-end указатель, он может иметь такой же аддрес как `q[0]`

LLVM IR не позволяет доступ к памяти через one-past-the-end указатель.

## `mem::transmute`

Берет данные типа `T` и преобразует их в данные типа `U`. В C можно кастануть тип. В Rust мы знаем как конвертировать один тип в другой.

```rust
fn foo() -> i32{ 42 }
let pointer = foo as *const();
let function = unsafe{
    std::mem::transmute::<*const(), fn() -> i32>(pointer)
    };
assert_eq!(function(), 42);
```

- `T` и `U` должны быть одинакового размера. Иначе UB.
- `3u8` нельзя перевести в `bool`
- Transmutting `&` в `&mut` всегда UB.

## `mem::transmute_copy`

Более небезопасный чем `mem::transmute`.

- Не проверяет что `T` и `U` одинакового размера.
- Копирует `size_of<U>` байт из `&T` и интерпретирует их как `U`.
- Если `U` > `T` то UB.

# Uninitialized memory

Нужно сохранять значения которые не валидны для текущего цикла. Для этого используется кусок памяти для типа `T`.

`std::mem::MaybeUninit<T>` структура.

Основные методы `MaybeUninit`:

- `uninit()` - создает `MaybeUninit` или проще говоря кусок памяти для типа `T`. Нельзя полагаться на содержимое тк оно не инициализировано.
- `new(val: T)` - создает `MaybeUninit` инициалзтруемый содержимым `T`. Компилятор все еще не знает о содержании `MaybeUninit`.
- `assume_init(self) -> T` - предполагает текущий `MaybeUninit` инициализированным и возвращает его как `T`. Unsafe.

```rust
let array = unsafe{
    // Type inference gives
    // MaybeUninit<[MaybeUninit<MyType>; 256]>::uninit()
    
    let mut array: [MaybeUninit<MyType>; 256] = MaybeUninit::uninit().assume_init();

    for (i, elem) in array.iter_mut().enumerate() {
        *elem = MaybeUninit::new(calculate_elem(i));
    }

    std::mem::transmute::<_, [MyType; 256]>(array)
};
```

При работе с unitialized memory и указателями надо помнить что при попытке изменить содержимое по указателю через разыменование происхоит Drop.

Niche optimization

# Когда и зачем использовать `unsafe`? #Q

---

# Parallel Computing

Для того чтобы писать параллельный код надо использовать `std::thread::spawn`.

```rust
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
where
    F: FnOnce() -> T,
    F: Send + 'static, // ← 'static здесь!
    T: Send + 'static,
```

`'static` lifetime на случай если поток может пережить функцию.

```rust
const THREAD_NUM: usize = 8;
let result: Vec<usize> = (0..THREAD_NUM)
    .map(|_| thread::spawn(move || simulate()))
    .map(|handle| handle.join().expect("Thread panicked"))
    .collect();
```

## `expect()`

`JoinHandle` даст `Result` со значеним или ошибку со значением паники.

`expect()` позволяет обработать эту ситуацию в главном потоке.

```rust
// Проблема lifetimes: vec переживает поток
fn example(){
    let vec = vec![1, 2, 3];
    let handle = thread::spawn(|| {
        // Замыкание заимствует &vec
        for i in vec.iter() {
            println!("{i}");
        }
    }); // Поток может жить дольше функции!
    handle.join();
} // vec уничтожается здесь
```

## `'static` lifetime in `thread::spawn`

Замыкание может пережить фунцию, оно заимствует `vec` которым владеет функция.

- Rust не знает ничего о `join`.
- Также нет гарантий что не будет паники до `join`.
- Ничего не останавливает нас от утечки `JoinHandle`

```rust
// Пример утечки `JoinHandle`
fn example(){
    let vec = vec![1, 2, 3];
    let handle = thread::spawn(move || {
        for i in vec.iter() {
            println!("{i}");
        }
    });
    std::mem::forget(handle); // "Забываем" handle
    // Поток становится detached и может жить вечно
}
```

=> В `thread::spawn` нужно использовать `'static` lifetime.

Мы используем `move` чтобы передать владение `vec` в замыкание как и советует компилятор.

```rust
fn example(){
    let vec = vec![1, 2, 3];
    let handle = thread::spawn(move || { // ✅ move передает владение
        for i in vec.iter() { // vec теперь принадлежит замыканию
            println!("{i}");
        }
    }); // Поток может жить дольше функции!
    handle.join();
} // println!("{:?}", vec); // ❌ Error: vec больше не доступен
```

```shell
Функция example():  |-----------|
vec (original):     |-----|      ← уничтожается при move
                          ↓
vec (в замыкании):        |-------------|
Поток (thread):     |----------------------|
                                    ↑ vec безопасно живет в потоке
```

```rust
// Не хватает lifetime - ошибка компиляции
fn count_foo_bar(data: &str) -> usize {
    let t1 = thread::spawn(move || data.mathces("foo").count());
    // ‼️ Error: lifetime 'static required
    let t2 = thread::spawn(move || data.mathces("bar").count());
    t1.join().unwrap() + t2.join().unwrap()
}
```

```rust
// Data race на счетчике внутри Rc<str>
fn count_foo_bar(data: Rc<str>) -> usize {
    let data_2 = data.clone();
    let t1 = thread::spawn(move || data.mathces("foo").count());
    let t2 = thread::spawn(move || data_2.mathces("bar").count());
    t1.join().unwrap() + t2.join().unwrap()
}
```

## `Send` и `Sync` traits

`Send` означает что тип может быть безопасно передан в другой поток.

Если мы делим `Rc` между двумя возможно что два потока будут изменять счетчик одновременно - data race.

Data race:

- Два или больше потока конкуретно получают доступ к одному участку памяти.
- Один или больше из них изменяют значение.
- Один или больше из них не синхронизированны.

**Interior mutability** делает более сложным memory safety. Поэтому мы используем `Send` и `Sync` traits.

`Send` и `Sync` traits `unsafe`:

- Тип `Send` если его безопасно передавать в другой поток.
- Тип `Sync` если его безопасно делить между потоками - иметь ссылку на объект в нескольких потоках. 
- `T` это `Sync` <=> `&T` это `Send`.

## Типы `Send` и `Sync`

- `i32` - `Send` и `Sync` ✅✅
- `Vec<i32>` - `Send` и `Sync` ✅✅
- `&str` - `Send` и `Sync` (тут подразумевается что `static str`) ✅✅
- `Rc<T>` - НЕ `Send` и НЕ `Sync` ❌❌
- `Cell<T>` - `Send` и НЕ `Sync` ✅❌
- `MutexGuard<'static, ()>` - НЕ `Send` и `Sync` ❌✅
- `*mut T` - НЕ `Send` и НЕ `Sync` ❌❌

`Send` и `Sync` - это auto traits.

```rust
pub unsafe trait Send {}
pub unsafe trait Sync {}
```

## `std::sync`

`std::sync` содержит типы для синхронизации потоков.

`Arc` - Atomic reference counting pointer.

```rust
use std::sync::Arc;

fn count_foo_bar(data: Arc<str>) -> usize {
    let data_2 = data.clone();
    let t1 = thread::spawn(move || data.mathces("foo").count());
    let t2 = thread::spawn(move || data_2.mathces("bar").count());
    t1.join().unwrap() + t2.join().unwrap()
}
// ✅ тут все окей тк `Arc` это `Send` и `Sync` из-за атомарности
```

Rust использует модель памяти C++20.

## `std::sync::atomic::*`

```rust
use std::sync::atomic::{AtomicUsize, Ordering};

struct RequestHandler {
    counter: Arc<AtomicUsize>,
}

impl RequestHandler {
    fn handle_request(&self) {
        self.counter.fetch_add(1, Ordering::SeqCst);
        /* ... */
    }
}
```

`AtomicUsize` - это примитив Interior mutability.

Из-за того что мы не знаем сколько проживут handler'ы мы используем `Arc` для того чтобы сделать счетчик атомарным.

### Могут ли некоректный Memory ordering привести к memory unsafety? #Q

Нет, все будет ок, можно использовать например `Relaxed` ordering.

### Могут ли случаться Race condition #Q

Да могут, программа может получить **deadlock** или **data race**. Но race condition не может привести к memory unsafety если использовать Safe Rust.

Это называется **Fearless concurrency**.

```rust
let data = vec![1, 2, 3];
let idx = Arc::new(AtomicUsize::new(0));
let other_idx = idx.clone();

thread::spawn(move || {
    other_idx.fetch_add(10, Ordering::SeqCst);
});

// Race condition!
// May panic but not memory unsafety
println!("{}", data[idx.load(Ordering::SeqCst)]);

// А вот тут будет memory unsafety из-за Unsafe code
if idx.load(Ordering::SeqCst) < data.len() {
    unsafe {
        println!("{}", data.get_unchecked(idx.load(Ordering::SeqCst)));
    }
}
```

Атомик - это многопоточный `Cell`
### Что такое многопоточный `RefCell`? #Q

Mutex - это примитив синхронизации для защиты `T`.

`RefCell` - это многопоточный Mutex.

## Список примитивов синхронизации в Rust

- `Mutex`
- `Barrier`
- `Condvar`
- `Once` - один раз выполнить код, в основном используется для инициализации.
- `Semaphore`
- `SpinLock`
- `SpinWait`
- `WaitGroup`
- `RwLock` - read write lock.
- `mpsc` - multiple producer, single consumer.

### Channels - Каналы

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
    });

    let received = rx.recv().unwrap();
    println!("Got: {}", received);
}

// Множественные производители
fn multiple_producers() {
    let (tx, rx) = mpsc::channel();
    let tx1 = tx.clone();

    thread::spawn(move || {
        tx.send(String::from("hi from thread 1")).unwrap();
    });

    thread::spawn(move || {
        tx1.send(String::from("hi from thread 2")).unwrap();
    });

    for received in rx {
        println!("Got: {}", received);
    }
}
```

### Shared state - Разделяемое состояние

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();
            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```

---

# Poisoning

Ситуация в которой тред паникует во время `MutexGuard`. Данные были частично модифицированны и произошла паника - потеряли консистентность.

`Mutex` даст `LockResult<MutexGuard<'_, T>>` который дает **lock** или **PoisonError**, означающий что тред запаниковал.

# Crossbeam

Crate для cuncurrent computing. Расширяе возможности `std::sync`.

## `crossbeam::scope`

Используется для создания scoped потоков.

```rust
pub fn scope<'env, F, R>(
    f: F
) -> Result<R, Box<dyn Any + 'static + Send, Global>>
where
    F: FnOnce(&Scope<'env>) -> R;
```

Создали `Scope`, область в которой мы буде создавать потоки.

```rust
let people = vec![
    "Alice".to_owned(),
    "Bob".to_owned(),
    "Charlie".to_owned()
];

thread::scope(|s| {
    for person in &people {
        s.spawn(move |_| {
            println!("{} is eating", person);
        });
    }
}).unwrap();
```

`unwrap()` - это метод для извлечения значения из `Option<T>` и `Result<T, E>`, который паникует если значения нет.

```rust
// Panic example
thread::scope(|s| {
    s.spawn(move |_| {
        panic!("Panic in thread");
    });
}).map_err(|e| eprintln!("Error: {}", e));
```

## `crossbeam::channel`

Альтернатива `std::sync::mpsc`.

- Мультиконсьюмеры и мультипроизводители.
- Канал может быть ограничен по размеру буфера или неограничен.

```rust
let (send_end, recv_end) = channel::bounded(5);
for i in 0...5 {
    send_end.send(i).unwrap();
}

// Will block 
send_end.send(5).unwrap();

let (send_end, recv_end) = channel::unbounded();

for i in 0...100 {
    send_end.send(i).unwrap();
}
```

- `channel::bounded` - канал с ограниченным буфером.
- `channel::unbounded` - канал без ограничения на размер буфера.
- `channel::tick` - канал с таймаутом

Методы:

- `send` - отправляет значение в канал.
- `recv` - получает значение из канала.
- `try_send` - отправляет значение в канал, если канал заполнен, то возвращает `Err`.
- `try_recv` - получает значение из канала, если канал пуст, то возвращает `Err`.
- `send_timeout`

- если все хендлеры одного из концов дропнуты канал войдет в **disconnected** состояние.
- сообщения не могут быть записаны в канал если он а **closed** состоянии, читать из него можно.
- операции над **disconnected** каналом не блокируются.
- можно использовать итераторы `iter()` для блокировки, `try_iter()` для неблокировки.

## `crossbeam::select`

```rust
let(s1, r1) = channel::unbounded();
let(s2, r2) = channel::unbounded();

thread::spawn(move || assert_eq!(s1.send(10), Ok(())));
thread::spawn(move || assert_eq!(r2.recv(), Ok(20)));
select! {
    recv(r1) -> msg => assert_eq!(msg, Ok(10)),
    send(s2, 20) => assert_eq!(r2.recv(), Ok(())),
    default(Duration::from_secs(1)) => println!("Timeout"),
}
```

## `crossbeam::utils`

- `CachePadded` - padding для данных по кешу для предотвращения false sharing.
- `ShardedLock` - шардированный RwLock из крейта crossbeam, которая разделяет блокировку на несколько частей (shards) для улучшения производительности чтения. Захватывает read lock только в одной части (shard). Захватывает write lock во всех частях одновременно.
- `Backoff` - exponential backoff для алгоритмов конкурентного программирования.
- `AtomicCell` - атомарный `Cell`.
- `SegQueue` - сегментный очередь.
- `WaitGroup` - wait group.

## `crossbeam::deque`

Конкурентная duble ended queue. Поддерживает work stealing.

- Включает в себя глобальную очередь и локальную очередь для каждого потока.
- Есть Steal - операция которая позволяет забрать элемент из очереди другого потока.
- Тред пытается забрать элемент из своей локальной очереди, если она пуста, то пытается забрать из глобальной очереди, если она пуста, то пытается украсть.

## `crossbeam::epoch`

Garbage collection для lock-free программ.

- Решает проблему удаления элементов в lock-free программах.
- Когда удаляем элемент, он добавляется в корзину привязанную к эпохе.
- Когда взаимодейтсвуем с `lock-free` структурой, `epoch` инкрементируется.

# Rayon

Крейт который предназначен для параллелизации данных. Похож на `OpenMP`.

Создает **work-stealing thread-pool** и отправляет туда задачи на выполнение. Крейт оптимизирован для CPU-bound задач.

```rust
use rayon::prelude::*;

fn sum_of_squares(input: &[i32]) -> i32 {
    input.par_iter().map(|&i| i * i).sum()
}
```

Нет проблем с momory safety.

## `rayon::join`

Используется чтобы запустить несколько closure(замыканий) параллельно.

```rust
pub fn join<A, B, RA, RB>(a: A, b: B) -> (RA, RB)
where
    A: FnOnce() -> RA + Send,
    B: FnOnce() -> RB + Send,
    RA: Send,
    RB: Send; 
```

- Все происходит на стеке.
- Если работает внутри thread pool, то текущий поток берет одну closure и выполняет ее, а другая идет в thread pool deque.
- Если работает вне thread pool, то замыкания выполняются последовательно.
- Предполагается что все closure CPU-bound. Если closure заблокируется то это заблокирует тред в thread pool и помешает полностью использовать ядера процессора. Не стоит использовать при работе с сетью.

## `rayon::scope`

Используется чтобы создать scoped thread

```rust
pub fn scope<'scope, OP, R>(op: OP) -> R
where
    R: Send,
    OP: FnOnce(&Scope<'scope>) -> R + Send;
```

- Поход на `crossbeam::scope`, но `crossbeam` запускает треды в созданном скоупе, а `rayon` создает задания которые будует выполняться в тред пула в созданном скоупе.
- Происходят аллокации в куче. Поэтому лучше использовать join
- Блокирующая, ждет пока все задачи завершатся.
- Более универсальная чем `rayon::join` но использует кучу в то время как `join` использует стек без аллокаций.

---
## ❓Parallel vs Async❓#Q

**Parallel ⏸️**
- CPU-bound задачи
- Большая вычислительная мощность
- **Цель:** Ускорение тяжёлых вычислений
- Физическое использование нескольких вычислительных ресурсов(ядер, потоков)

*Перемножение матриц, обработка чисел -> Генерация изображений, обработка видео -> Алгоритмы машинного обучения и анализа данных.*
 
 📦 Пакеты
	- **Rayon** (рекомендуется для data-parallel задач).
	- **std::thread** (низкоуровневый, ручной контроль потоков).

**Async ⏯️**
- Input/Output тяжелые задачи, задачи с ожиданием Input/Output
- **Цель:** Эффективное ожидание и обработка множества задач, каждая из которых большую часть времени простаивает, ожидая внешних данных.

*Веб-серверы (HTTP, TCP, WebSockets). Асинхронные клиенты баз данных. Event-driven архитектуры (управление событиями, сообщениями).*

📦 Пакеты:
	- **Tokio** (самый популярный async runtime).
	- **async-std** (альтернативный async runtime).
	- Стандартный механизм языка (async/await).

---
# Asynchronous computing - Асинхронность

Асинхронный код (async/await) в Rust создан для неблокирующих операций (сеть, диск, ожидание событий).

Async/Await - Асинхронное программирование

## Основы async/await

```rust
use std::time::Duration;
use tokio::time::sleep;

// Асинхронная функция
async fn say_hello() {
    println!("Hello");
    sleep(Duration::from_secs(1)).await;
    println!("World");
}

#[tokio::main]
async fn main() {
    say_hello().await;
}
```

### Futures

**Future** в Rust — это тип, который представляет собой **значение, которое станет доступным позже (асинхронно)**. Другими словами, это отложенное вычисление, результат которого будет получен в будущем.

```rust
pub trait Future {
    type Output;

    // Метод poll вызывается для продвижения future к завершению
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}

pub enum Poll<T> {
    Ready(T),   // Значение готово
    Pending,    // Значение ещё не готово
}
```

- **Future** может быть запущена, приостановлена и возобновлена позже

```rust
use std::future::Future;
use std::pin::Pin;
use std::task::{Context, Poll};

// Кастомный Future
struct TimerFuture {
    shared_state: Arc<Mutex<SharedState>>,
}

struct SharedState {
    completed: bool,
    waker: Option<Waker>,
}

impl Future for TimerFuture {
    type Output = ();
    
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output> {
        let mut shared_state = self.shared_state.lock().unwrap();
        if shared_state.completed {
            Poll::Ready(())
        } else {
            shared_state.waker = Some(cx.waker().clone());
            Poll::Pending
        }
    }
}
```

### Async блоки и closures

```rust
async fn example() {
    // async блок
    let future = async {
        println!("Inside async block");
        42
    };
    
    let result = future.await;
    println!("Result: {}", result);
    
    // async closure (unstable)
    // let closure = async |x| {
    //     x * 2
    // };
}
```

### Работа с несколькими futures

```rust
use tokio::join;
use tokio::select;

async fn fetch_data(id: u32) -> String {
    format!("Data for id: {}", id)
}

async fn example() {
    // Параллельное выполнение
    let (data1, data2, data3) = join!(
        fetch_data(1),
        fetch_data(2),
        fetch_data(3)
    );
    
    // Выбор первого завершившегося
    select! {
        result = fetch_data(1) => {
            println!("Got: {}", result);
        }
        result = fetch_data(2) => {
            println!("Got: {}", result);
        }
    }
}
```


## Pinning

API **Pin / Box::pin** используется в Rust для того, чтобы гарантировать **неперемещаемость** объектов в памяти после того, как они были закреплены (pinned)

Тип `Pin<P>` — это тип-обёртка вокруг указателей (`Box`, `&mut`, `&`), гарантирующая, что **данные за указателем никогда не будут перемещены после закрепления**.

## Tokio

### Асинхронный Runtime
Асинхронный runtime отвечает за:

- Планирование выполнения (`scheduler`).
- Опрос и управление задачами (futures).
- Реализацию механизма `async/await`.

Запуск Tokio runtime бывает двух видов:

- **Многопоточный (multi-threaded runtime)**  
  - Создаёт пул потоков (по умолчанию количество равно числу CPU-ядер).
  - Автоматическое распределение задач между потоками.

```rust
#[tokio::main]
async fn main() {
    // код
}
```

- **Однопоточный (single-threaded runtime)**
  - Выполняет задачи в одном потоке (эффективно для lightweight-приложений).

```rust
#[tokio::main(flavor = "current_thread")]
async fn main() {
    // код
}
```

### Фьючерсы (Futures)

- В основе асинхронности лежат futures:

```rust
async fn fetch() -> String {"hello".into()}  let fut = fetch(); // Future, который пока не запущен let result = fut.await; // запуск и ожидание результата
```

- Tokio сам управляет состоянием и опросом (polling) фьючерсов.

### Асинхронные задачи и управление ими

- Tokio запускает асинхронные задачи через `tokio::spawn`:

```rust
async fn fetch() -> String {     "hello".into() }  let fut = fetch();
// Future, который пока не запущен let result = fut.await; // запуск и ожидание результата
```

- Задачи исполняются параллельно внутри runtime.
- Задачи возвращают JoinHandle:

```rust
let handle = tokio::spawn(async { 42 });
let result = handle.await.unwrap(); // результат = 42
```

### Работа с сетевым вводом-выводом

Tokio активно используется для сетевых приложений:

- TCP-клиент:

```rust
use tokio::net::TcpStream;
let stream = TcpStream::connect("127.0.0.1:8080").await?;
```

- TCP-сервер:

```rust
use tokio::net::TcpListener;
let listener = TcpListener::bind("127.0.0.1:8080").await?;
loop {     
    let (socket, addr) = listener.accept().await?;
    tokio::spawn(async move {         
        handle_connection(socket).await;     
        }); 
    }
```

- UDP:

```rust
use tokio::net::UdpSocket;  let sock = UdpSocket::bind("127.0.0.1:8080").await?;
```

### Асинхронные каналы и синхронизация

Tokio предоставляет асинхронные примитивы для коммуникаций:

- **Каналы (Channels)**:
```rust
use tokio::sync::mpsc;
let (tx, mut rx) = mpsc::channel(32);
tokio::spawn(async move {     
    tx.send("сообщение").await.unwrap(); 
    });
while let Some(msg) = rx.recv().await { println!("Получено: {}", msg); }
```
    
- **Mutex/RwLock (асинхронные локи)**:
```rust
use tokio::sync::Mutex;
let data = Mutex::new(vec![1, 2, 3]);
let mut locked = data.lock().await;
locked.push(4);
```

### Работа с файловой системой

Tokio предоставляет асинхронные операции с файлами:

```rust
use tokio::fs;
let data = fs::read_to_string("file.txt").await?;
fs::write("out.txt", "content").await?;
```

### Таймеры и планирование задач

- Tokio имеет удобные таймеры:
```rust
use tokio::time::{sleep, Duration};  sleep(Duration::from_secs(5)).await;
```

- Интервалы для периодических задач:
```rust
use tokio::time;
use std::time::Duration;

let mut interval = time::interval(Duration::from_secs(1));
loop {
    interval.tick().await;
    println!("тик каждую секунду");
}
```

### Блокирующие (CPU-bound) операции

Если нужно выполнить CPU-bound операцию, можно использовать:

```rust
let result = tokio::task::spawn_blocking(|| {     // медленная CPU-bound задача     expensive_computation() }).await.unwrap();
```

- Tokio выделяет отдельный пул потоков для таких задач, избегая блокировок runtime.

### Полезные дополнительные библиотеки экосистемы Tokio

- **Hyper** — HTTP сервер/клиент.
- **Tonic** — gRPC фреймворк.
- **Axum**, **Warp** — HTTP-фреймворки.
- **Reqwest** — HTTP-клиент.
- **Tokio-stream** — потоки данных (streams).

---

## Как Tokio работает внутри (кратко)

- Tokio использует механизм неблокирующего ввода-вывода (`epoll` на Linux, `kqueue` на macOS), позволяя выполнять множество операций параллельно.
- Tokio самостоятельно управляет пулом потоков, балансирует задачи, избегая блокировок.
- Внутри Tokio:
  - Асинхронные задачи представлены futures.
  - Runtime вызывает метод `poll()` на future при наступлении событий (например, доступность данных в сокете).

## Ограничения

- Не предназначен для ускорения CPU-bound задач (требует дополнительной интеграции с `rayon` или `spawn_blocking`).

## Когда использовать Tokio

- Сетевые приложения (веб-сервисы, прокси).
- Асинхронная работа с диском (если уместно).
- Нагруженные серверы, требующие эффективного асинхронного I/O.

---

# Macros - Макросы

## Declarative Macros (macro_rules!)

```rust
// Простой макрос
macro_rules! say_hello {
    () => {
        println!("Hello!");
    };
}

// Макрос с параметрами
macro_rules! create_function {
    ($func_name:ident) => {
        fn $func_name() {
            println!("You called {:?}()", stringify!($func_name));
        }
    };
}

create_function!(foo);

// Макрос с множественными паттернами
macro_rules! test {
    ($left:expr; and $right:expr) => {
        println!("{:?} and {:?} is {:?}",
                 stringify!($left),
                 stringify!($right),
                 $left && $right)
    };
    ($left:expr; or $right:expr) => {
        println!("{:?} or {:?} is {:?}",
                 stringify!($left),
                 stringify!($right),
                 $left || $right)
    };
}

test!(1i32 + 1 == 2i32; and 2i32 * 2 == 4i32);
test!(true; or false);
```

### Макрос `vec!`

```rust
// Упрощённая реализация vec!
macro_rules! vec {
    ( $( $x:expr ),* ) => {
        {
            let mut temp_vec = Vec::new();
            $(
                temp_vec.push($x);
            )*
            temp_vec
        }
    };
}
```

### Procedural Macros

```rust
// В Cargo.toml
// [lib]
// proc-macro = true

use proc_macro::TokenStream;
use quote::quote;
use syn;

#[proc_macro_derive(HelloMacro)]
pub fn hello_macro_derive(input: TokenStream) -> TokenStream {
    let ast = syn::parse(input).unwrap();
    impl_hello_macro(&ast)
}

fn impl_hello_macro(ast: &syn::DeriveInput) -> TokenStream {
    let name = &ast.ident;
    let gen = quote! {
        impl HelloMacro for #name {
            fn hello_macro() {
                println!("Hello, Macro! My name is {}!", stringify!(#name));
            }
        }
    };
    gen.into()
}
```

---

# Advanced Type System

### Associated Types

```rust
pub trait Iterator {
    type Item; // ассоциированный тип
    
    fn next(&mut self) -> Option<Self::Item>;
}

struct Counter {
    current: usize,
    max: usize,
}

impl Iterator for Counter {
    type Item = usize;
    
    fn next(&mut self) -> Option<Self::Item> {
        if self.current < self.max {
            let current = self.current;
            self.current += 1;
            Some(current)
        } else {
            None
        }
    }
}
```

### Default Type Parameters

```rust
use std::ops::Add;

#[derive(Debug, Copy, Clone, PartialEq)]
struct Point {
    x: i32,
    y: i32,
}

impl Add for Point {
    type Output = Point;
    
    fn add(self, other: Point) -> Point {
        Point {
            x: self.x + other.x,
            y: self.y + other.y,
        }
    }
}

// Кастомный тип для сложения
struct Millimeters(u32);
struct Meters(u32);

impl Add<Meters> for Millimeters {
    type Output = Millimeters;
    
    fn add(self, other: Meters) -> Millimeters {
        Millimeters(self.0 + (other.0 * 1000))
    }
}
```

### Newtype Pattern

```rust
// Newtype pattern для создания новых типов
struct Wrapper(Vec<String>);

impl fmt::Display for Wrapper {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "[{}]", self.0.join(", "))
    }
}
```

### Type Aliases

```rust
// Псевдонимы типов
type Kilometers = i32;
type Result<T> = std::result::Result<T, std::io::Error>;

// Псевдонимы для сложных типов
type Thunk = Box<dyn Fn() + Send + 'static>;

let f: Thunk = Box::new(|| println!("hi"));
```

---

# Performance и оптимизация

### Профилирование

```rust
// Бенчмарки с criterion
use criterion::{black_box, criterion_group, criterion_main, Criterion};

fn fibonacci(n: u64) -> u64 {
    match n {
        0 => 1,
        1 => 1,
        n => fibonacci(n-1) + fibonacci(n-2),
    }
}

fn criterion_benchmark(c: &mut Criterion) {
    c.bench_function("fib 20", |b| b.iter(|| fibonacci(black_box(20))));
}

criterion_group!(benches, criterion_benchmark);
criterion_main!(benches);
```

### Zero-cost abstractions

Компилятор (особенно Rust и C++) производит агрессивные оптимизации, которые полностью устраняют накладные расходы абстракций:

- **Inlining** (встраивание функций).    
- Удаление промежуточных объектов.
- Исключение ненужного кода (dead-code elimination).
- Мономорфизация (создание специализированного кода под конкретные типы при generics).

```rust
// Итераторы компилируются в эффективный код
let v: Vec<i32> = (0..1_000_000).collect();

// Это...
let sum: i32 = v.iter().map(|x| x * x).filter(|&x| x % 2 == 0).sum();

// ...компилируется примерно в это
let mut sum = 0;
for i in 0..1_000_000 {
    let squared = i * i;
    if squared % 2 == 0 {
        sum += squared;
    }
}
```

### Избегание аллокаций

```rust
// Плохо: множественные аллокации
fn format_numbers(numbers: &[i32]) -> String {
    let mut result = String::new();
    for number in numbers {
        result.push_str(&format!("{}, ", number));
    }
    result
}

// Хорошо: одна аллокация с предварительным резервированием
fn format_numbers_optimized(numbers: &[i32]) -> String {
    let mut result = String::with_capacity(numbers.len() * 4); // примерная оценка
    for (i, number) in numbers.iter().enumerate() {
        if i > 0 {
            result.push_str(", ");
        }
        result.push_str(&number.to_string());
    }
    result
}
```

### Использование Cow (Clone on Write)

```rust
use std::borrow::Cow;

fn process_text(input: &str) -> Cow<str> {
    if input.contains("bad_word") {
        Cow::Owned(input.replace("bad_word", "***"))
    } else {
        Cow::Borrowed(input)
    }
}
```

# ⚙️ Rust-решения для Ethereum-валидаторов

## 🧱 Рамка “awesome-ethereum-rust”

- **Lighthouse** – реализация consensus-клиента (валидатора) на Rust
- **grandine** – эфемерный консенсус-клиент, ориентированный на высокую производительность 
- **reth** – Rust-исполнение EVM. В нём есть модуль `EthTransactionValidator` для валидации транзакций 
- **Ream** – consensus-клиент с Rust-эндпоинтами (подготовка proposer’ов) 

Эти компоненты вместе формируют стек:

🔥 **Consensus**: _Lighthouse_, _grandine_, _Ream_  
💻 **Execution**: _reth_  
🔐 **Validator layer**: отдельно (например, SafeStake + Lighthouse/Ream) для управления подписанием блоков/операций

---

## 🔐 Архитектура: как Rust решает задачи Ethereum-валидатора

1. **Consensus-клиент** (Beacon node): реализует PoS, раунды, слешинг, рекламирует себя в сети. Rust-клиенты: _Lighthouse_, _grandine_, _Ream_.
2. **Execution-клиент**: исполняет EVM-транзакции, ведёт стейт, предоставляет API. Rust-вариант – _reth_.
3. **Validator layer**: управляет приватными ключами, готовит блоки, подписывает block proposal. Rust-решения: _SafeStakeOperator_ и плагины к consensus-клиентам (Lighthouse/Ream).
4. **DVT**: распределение ключей (мультиподписи / фрагментация ключевых частей). Реализовано в SafeStakeOperator.

---

# Questions pool

## Общие вопросы по Rust (базовые)

- **What are lifetimes in Rust?** (Что такое _времена жизни_ ссылок в Rust?)[linkedin.com](https://www.linkedin.com/posts/ganesh-mishra-a49221195_rustlang-interviewjourney-learning-activity-7330133113349603328-jpTK#:~:text=data%20races,5%EF%B8%8F%E2%83%A3%20How%20does%20Rust)

- **How does Rust manage errors?** (Как в Rust осуществляется обработка ошибок, например с помощью `Result`и `panic!`?)[linkedin.com](https://www.linkedin.com/posts/ganesh-mishra-a49221195_rustlang-interviewjourney-learning-activity-7330133113349603328-jpTK#:~:text=slice%2C%20a%20reference%20to%20a,enum%20for%20recoverable%20errors%20and)

- **How does Rust handle “null” values?** (Что используется в Rust вместо null?)[usebraintrust.com](https://www.usebraintrust.com/hire/interview-questions/rust-developers#:~:text=ensures%20that%20each%20value%20has,Lifetimes%20specify%20how%20long%20a)

- **Describe Rust's ownership system.** (Опишите систему владения данными в Rust)[usebraintrust.com](https://www.usebraintrust.com/hire/interview-questions/rust-developers#:~:text=,immutable%2C%20enforcing%20a%20strict%20set)

- **Explain Rust's borrowing and referencing rules.** (Объясните правила заимствования и ссылок в Rust)[usebraintrust.com](https://www.usebraintrust.com/hire/interview-questions/rust-developers#:~:text=,that%20might%20be%20cleaned%20up)

- **What is the difference between using `panic!` and `unwrap()` for error handling?** (Чем отличается использование `panic!` и `unwrap()`?)[usebraintrust.com](https://www.usebraintrust.com/hire/interview-questions/rust-developers#:~:text=,Answer%3A%20Cargo%20is%20Rust%27s%20package)

- **What is Cargo in the context of Rust?** (Какова роль Cargo в экосистеме Rust?)[usebraintrust.com](https://www.usebraintrust.com/hire/interview-questions/rust-developers#:~:text=checks%20at%20compile%20time%2C%20ensuring,35)
    
- **What are Rust macros (`macro_rules!`)?** (Зачем нужны макросы в Rust?)[usebraintrust.com](https://www.usebraintrust.com/hire/interview-questions/rust-developers#:~:text=%2A%20Explain%20,Rust%3F%20Answer%3A%20Both%20are%20smart)

- **How does Rust enforce immutability and mutability?** (Как в Rust обеспечивается неизменяемость и объявляется изменяемость переменных?)[usebraintrust.com](https://www.usebraintrust.com/hire/interview-questions/rust-developers#:~:text=writes%20other%20code%2C%20a%20metaprogramming,Answer%3A%20Rust%27s%20module%20system%20allows)

- **Describe Rust's module system.** (Опишите, как организована модульная система в Rust)[usebraintrust.com](https://www.usebraintrust.com/hire/interview-questions/rust-developers#:~:text=,Answer%3A%20The%20Drop%20trait%20allows)

- **How does Rust handle error handling (exceptions vs `Result`)?** (Как в Rust обрабатываются ошибки, учитывая отсутствие исключений?)[linkedin.com](https://www.linkedin.com/posts/ganesh-mishra-a49221195_rustlang-interviewjourney-learning-activity-7330133113349603328-jpTK#:~:text=slice%2C%20a%20reference%20to%20a,enum%20for%20recoverable%20errors%20and)[usebraintrust.com](https://www.usebraintrust.com/hire/interview-questions/rust-developers#:~:text=splitting%20code%20into%20reusable%20chunks,level)

- **What is the `Drop` trait in Rust used for?** (Для чего используется трейt `Drop`?)[usebraintrust.com](https://www.usebraintrust.com/hire/interview-questions/rust-developers#:~:text=,level%20code)

- **Explain the term “zero-cost abstractions” in Rust.** (Что означает принцип «абстракций без лишних затрат» в Rust?)[usebraintrust.com](https://www.usebraintrust.com/hire/interview-questions/rust-developers#:~:text=cleanup%20tasks.%20%20,level%20code)

- **How are object-oriented principles implemented in Rust?** (Как в Rust реализованы принципы ООП?)[glassdoor.co.in](https://www.glassdoor.co.in/Interview/rust-developer-interview-questions-SRCH_KO0,14_IP2.htm#:~:text=26%20Sept%202024)[usebraintrust.com](https://www.usebraintrust.com/hire/interview-questions/rust-developers#:~:text=means%20that%20abstractions%20don%27t%20impose,The%20unsafe%20keyword%20allows%20operations)

- **What does the `unsafe` keyword allow in Rust?** (Что позволяет делать ключевое слово `unsafe` в Rust?)[usebraintrust.com](https://www.usebraintrust.com/hire/interview-questions/rust-developers#:~:text=,The%20unsafe%20keyword%20allows%20operations)

- **How do generics work in Rust?** (Как Rust поддерживает обобщенное программирование с помощью обобщений?)[usebraintrust.com](https://www.usebraintrust.com/hire/interview-questions/rust-developers#:~:text=through%20structs%2C%20enums%2C%20and%20traits,fearless)

- **What is a `Box<T>` in Rust and when would you use it?** (Что такое `Box<T>` и для чего он нужен?)[usebraintrust.com](https://www.usebraintrust.com/hire/interview-questions/rust-developers#:~:text=different%20data%20types.%20%20,Answer%3A%20Rust%27s%20strong%2C%20static%20type)

- **How does Rust achieve thread safety (“fearless concurrency”)?** (Как Rust обеспечивает потокобезопасность, избегая состязаний данных?)[usebraintrust.com](https://www.usebraintrust.com/hire/interview-questions/rust-developers#:~:text=,matching%20is%20a%20way%20to)[usebraintrust.com](https://www.usebraintrust.com/hire/interview-questions/rust-developers#:~:text=different%20data%20types.%20%20,related%20errors)   

## Вопросы по Rust (средние) - ответы от LLM (не проверял)

- Что означает ограничение `'static` в параметрах трейта или типа?
  - Ограничение `'static` на типах или трейтах означает, что тип не должен содержать ссылок с меньшим временем жизни. Он должен быть либо владением, либо ссылкой с бесконечным lifetime `'static`.

- Как работает подтипизация (subtyping) времён жизни в Rust?
  - В Rust подтипизация lifetime'ов работает по принципу: больший lifetime (`'static`) может быть неявно преобразован (coerced) в меньший. Обратное неверно.

- Как переместить значение из стека в кучу в Rust?
  - Использовать типы, размещающие данные на куче (`Box`, `Vec`, `String`):`

- Почему можно передать `&Box<T>` в функцию, которая ожидает `&T`?
  - В Rust реализован автоматический deref (авторазыменование через трейт `Deref`).

- Как добиться изменяемости данных без передачи `&mut` ссылки?
  - Использовать внутреннюю изменяемость (interior mutability):
    - `Cell` и `RefCell` для однопоточных сценариев.
    - `Mutex`, `RwLock`, `Atomic` — для многопоточных.

- Как организовать нескольких владельцев одного значения в Rust (однопоточно и многопоточно)?
  - **Однопоточно**: `Rc` (Reference Counted)
  - **Многопоточно**: `Arc` (Atomic Reference Counted)

- Разница между мономорфизацией (generics) и динамической диспетчеризацией (trait objects)?
  - **Generics (мономорфизация)**:
    - Генерируют отдельный код для каждого типа (статически).
    - Быстро, без накладных расходов вызова.
    - Больше размер бинарника.

  - **Trait Objects (динамическая диспетчеризация)**:
    - Вызовы виртуальны (через vtable).
    - Один код для всех типов с оверхедом вызова.
    - Компактный бинарник, больше накладных расходов при вызове.

- Что такое RAII и как применяется в Rust?
  - RAII (Resource Acquisition Is Initialization): ресурсы освобождаются автоматически, когда владеющая ими переменная выходит из области видимости (drop).
  - Rust реализует RAII через трейт `Drop`.

- Как Rust предотвращает ошибки памяти типа buffer overflow?

  - Жёсткий контроль доступа и размера массивов.
  - Проверки границ при каждом обращении к срезу.
  - Владение и borrowing предотвращают использование освобождённой памяти.
  - Примеры таких ошибок (Heartbleed) практически невозможны благодаря встроенным проверкам.

- Для чего нужны трейты `Send` и `Sync` в Rust?
  - **`Send`**: Тип безопасно передавать между потоками.
  - **`Sync`**: Тип безопасно использовать из нескольких потоков одновременно (через ссылки).
  - Оба являются ключевыми для thread safety.

- Что такое функции высшего порядка в Rust?
  - Функции, принимающие другие функции как аргументы или возвращающие их как результат.

- Что такое алгебраические типы данных (ADT)?
  - Типы данных, которые формируются через комбинацию sum-типа (`enum`) и product-типа (`struct`, `tuple`).

- Чем отличается `Copy` от `Clone`?
  - **`Copy`**: простое побитовое копирование, дешёвая операция.
  - **`Clone`**: явное, возможно дорогое копирование.

- Для чего используется API `Pin` / `Box::pin` в Rust и как оно работает?
  - `Pin` используется для фиксации значения в памяти, предотвращая его перемещение.
  - `Box::pin` используется для фиксации значения в памяти, предотвращая его перемещение.

- **Why can a variable of type `usize` be passed to a function without losing ownership, but not a `String`?**(Почему при передаче `usize` во функцию значение не теряет владельца, а при передаче `String` – теряет?)[reddit.com](https://www.reddit.com/r/rust/comments/1f0qbpe/if_you_were_the_interviewer_what_rust_questions/#:~:text=I%20don%E2%80%99t%20believe%20in%20asking,ask%20to%20gauge%20Rust%20expertise)[reddit.com](https://www.reddit.com/r/rust/comments/1f0qbpe/if_you_were_the_interviewer_what_rust_questions/#:~:text=,a%20variable%20of%20type%20String)

- **`&str` is a reference to data allocated where?** (На какую область памяти может указывать срез `&str`?)[reddit.com](https://www.reddit.com/r/rust/comments/1f0qbpe/if_you_were_the_interviewer_what_rust_questions/#:~:text=ownership%20but%20not%20a%20variable,of%20type%20String)[reddit.com](https://www.reddit.com/r/rust/comments/1f0qbpe/if_you_were_the_interviewer_what_rust_questions/#:~:text=Types%20that%20implement%20the%20,leaving%20the%20original%20value%20intact)

- **What is a `'static` lifetime and a static reference in Rust?** (Что такое время жизни `'static` и статическая ссылка в Rust?)[reddit.com](https://www.reddit.com/r/rust/comments/1f0qbpe/if_you_were_the_interviewer_what_rust_questions/#:~:text=)[reddit.com](https://www.reddit.com/r/rust/comments/1f0qbpe/if_you_were_the_interviewer_what_rust_questions/#:~:text=A%20lifetime%20is%20the%20,value%20can%20be%20valid%20for)

- **What does the `'static` lifetime bound mean for a type or trait?** (Что означает ограничение `'static` в параметрах трейта или типа?)[reddit.com](https://www.reddit.com/r/rust/comments/1f0qbpe/if_you_were_the_interviewer_what_rust_questions/#:~:text=)[reddit.com](https://www.reddit.com/r/rust/comments/1f0qbpe/if_you_were_the_interviewer_what_rust_questions/#:~:text=leaked%20values)

- **Why can an `&'static str` be used where an `&str` is expected?** (Почему ссылку типа `&'static str` можно передать в функцию, ожидающую `&str`?)[reddit.com](https://www.reddit.com/r/rust/comments/1f0qbpe/if_you_were_the_interviewer_what_rust_questions/#:~:text=,as%20a%20trait%20bound)[reddit.com](https://www.reddit.com/r/rust/comments/1f0qbpe/if_you_were_the_interviewer_what_rust_questions/#:~:text=,the%20alias%20be%20a%20%26str)

- **How does subtyping/coercion work with lifetimes in Rust?** (Как работает подтипизация времён жизни в Rust?)[reddit.com](https://www.reddit.com/r/rust/comments/1f0qbpe/if_you_were_the_interviewer_what_rust_questions/#:~:text=,the%20alias%20be%20a%20%26str)[reddit.com](https://www.reddit.com/r/rust/comments/1f0qbpe/if_you_were_the_interviewer_what_rust_questions/#:~:text=It%27s%20implicitly%20downgraded%20to%20a,static%20outlives%20every%20other%20lifetime)

- **How can you move a value from the stack to the heap in Rust?** (Как переместить значение из стека в кучу в Rust?)[reddit.com](https://www.reddit.com/r/rust/comments/1f0qbpe/if_you_were_the_interviewer_what_rust_questions/#:~:text=,the%20stack%20onto%20the%20heap)[reddit.com](https://www.reddit.com/r/rust/comments/1f0qbpe/if_you_were_the_interviewer_what_rust_questions/#:~:text=,the%20stack%20onto%20the%20heap)

- **Why can you pass an `&Box<T>` to a function expecting an `&T`?** (Почему можно передать `&Box<T>` в функцию, которая ожидает `&T`?)[reddit.com](https://www.reddit.com/r/rust/comments/1f0qbpe/if_you_were_the_interviewer_what_rust_questions/#:~:text=,functions%20that%20expect%20an%20%26T)[reddit.com](https://www.reddit.com/r/rust/comments/1f0qbpe/if_you_were_the_interviewer_what_rust_questions/#:~:text=,functions%20that%20expect%20an%20%26T)

- **How can you obtain mutability without using a mutable reference in Rust?** (Как добиться изменяемости данных без передачи `&mut` ссылки?)[reddit.com](https://www.reddit.com/r/rust/comments/1f0qbpe/if_you_were_the_interviewer_what_rust_questions/#:~:text=,passing%20around%20a%20mutable%20reference)[reddit.com](https://www.reddit.com/r/rust/comments/1f0qbpe/if_you_were_the_interviewer_what_rust_questions/#:~:text=,passing%20around%20a%20mutable%20reference)

- **How can you have multiple owners of a value in Rust (single-threaded vs multi-threaded)?** (Как организовать нескольких владельцев одного значения в Rust в однопоточном и многопоточном контексте?)[reddit.com](https://www.reddit.com/r/rust/comments/1f0qbpe/if_you_were_the_interviewer_what_rust_questions/#:~:text=mutable%20reference%3F)[reddit.com](https://www.reddit.com/r/rust/comments/1f0qbpe/if_you_were_the_interviewer_what_rust_questions/#:~:text=,threaded%20context)

- **What is the difference between using generics (monomorphization) and trait objects (dynamic dispatch) in Rust?** (В чем разница между обобщенными функциями (мономорфизацией) и объектами трейтов (динамическая диспетчеризация) в Rust?)[users.rust-lang.org](https://users.rust-lang.org/t/testing-rust-skills-of-candidates/92309#:~:text=,style%20pattern%20matching%3B%20interviewers%20understanding)

- **Explain RAII (Resource Acquisition Is Initialization) in the context of Rust.** (Что такое RAII и как этот принцип применяется в Rust?)[users.rust-lang.org](https://users.rust-lang.org/t/testing-rust-skills-of-candidates/92309#:~:text=pointers%20%28eg.%20trees%29%20,based%20explanation%20of%20a)

- **What are common causes of memory bugs (e.g. buffer overflows) and how does Rust prevent them?** (Каковы распространенные причины ошибок управления памятью, например _buffer overflow_, и как Rust помогает их избежать? Назовите примеры типа Heartbleed)[users.rust-lang.org](https://users.rust-lang.org/t/testing-rust-skills-of-candidates/92309#:~:text=dispatch%20,based%20explanation%20of%20a%20candidate)

- **What are `Send` and `Sync` traits in Rust and why are they important?** (Для чего нужны трейты `Send` и `Sync` и как они связаны с многопоточностью?)[glassdoor.co.in](https://www.glassdoor.co.in/Interview/rust-developer-interview-questions-SRCH_KO0,14_IP4.htm#:~:text=10%20Oct%202023)    
- **What are higher-order functions in Rust?** (Что такое функции высшего порядка в Rust?)[glassdoor.co.in](https://www.glassdoor.co.in/Interview/rust-developer-interview-questions-SRCH_KO0,14_IP4.htm#:~:text=10%20Oct%202023)

- **What are algebraic data types (ADTs) in Rust?** (Что такое алгебраические типы данных в Rust, и как они выражаются в языке?)[glassdoor.co.in](https://www.glassdoor.co.in/Interview/rust-developer-interview-questions-SRCH_KO0,14_IP4.htm#:~:text=10%20Oct%202023)

- **What is the difference between the `Copy` trait and the `Clone` trait in Rust?** (Чем отличается трейт `Copy` от трейта `Clone` в Rust?)[glassdoor.co.in](https://www.glassdoor.co.in/Interview/rust-developer-interview-questions-SRCH_KO0,14_IP4.htm#:~:text=Rust%3A%20multithreading%20,Rust%2C%20onion%20architecture%2C%20Copy%2FClone%20traits)

- **How does the Pin/`Box::pin` API work in Rust and when would you use it?** (Для чего используется API `Pin` / `Box::pin` в Rust и как оно работает?)[glassdoor.co.in](https://www.glassdoor.co.in/Interview/rust-developer-interview-questions-SRCH_KO0,14_IP2.htm#:~:text=20%20Sept%202023)

## Вопросы по Substrate и ink! (Polkadot)

- **What do you think are the key advantages of using Rust for developing blockchain applications?** (В чем вы видите основные преимущества использования Rust для разработки блокчейн-приложений?)[aspect-hq.com](https://aspect-hq.com/interview-questions-2/Substrate-Rust-Developer-Interview-Questions#:~:text=experience%20do%20you%20have%20with,client%3F%20Do%20you%20have%20any)
    
- **Why is Substrate (Rust) a particularly good choice for developing blockchain applications?** (Почему платформа Substrate на Rust считается удачным выбором для разработки блокчейнов?)[aspect-hq.com](https://aspect-hq.com/interview-questions-2/Substrate-Rust-Developer-Interview-Questions#:~:text=experience%20do%20you%20have%20with,client%3F%20Do%20you%20have%20any)

- **What is the Substrate framework and how is it used in blockchain development?** (Что такое Substrate и как он применяется для создания блокчейн-систем?)

- **What do you find most challenging about developing on Substrate?** (Что было самым сложным в разработке на Substrate, по вашему опыту?)[aspect-hq.com](https://aspect-hq.com/interview-questions-2/Substrate-Rust-Developer-Interview-Questions#:~:text=think%20makes%20Substrate%20Rust%20a,did%20you%20solve%20them%3F%20What)

- **Have you encountered any problems while working with Substrate, and how did you solve them?** (С какими проблемами вы сталкивались при работе с Substrate и как вы их решали?)[aspect-hq.com](https://aspect-hq.com/interview-questions-2/Substrate-Rust-Developer-Interview-Questions#:~:text=Substrate%20Rust%3F%20What%20do%20you,client%3F%20Do%20you%20have%20any)

- **How do you test your code in a Substrate-based project?** (Как вы тестируете свой код в проектах на Substrate?)

- **Что такое ink! и какую роль он играет в экосистеме Polkadot/Substrate?** (Объясните, для чего предназначен фреймворк ink! при написании смарт-контрактов на Rust в экосистеме Polkadot)[credmark.ai](https://credmark.ai/practice/top-rust-smart-contracts-interview-questions-and-answers#:~:text=,contracts%20for%20the%20Polkadot%20ecosystem)

- **Which framework is primarily used for writing Rust smart contracts in the Polkadot ecosystem?** (Какой фреймворк используется для разработки смарт-контрактов на Rust в экосистеме Polkadot?)[credmark.ai](https://credmark.ai/practice/top-rust-smart-contracts-interview-questions-and-answers#:~:text=12,contracts%20on%20the%20Polkadot%20ecosystem)

## Вопросы по CosmWasm (Cosmos)

- **What is CosmWasm and what is its primary purpose?** (Что такое CosmWasm и для чего он предназначен?)[credmark.ai](https://credmark.ai/practice/top-cosmwasm-interview-questions-and-answers#:~:text=1,purpose%20of%20Cosmwasm)[credmark.ai](https://credmark.ai/practice/top-cosmwasm-interview-questions-and-answers#:~:text=4,designed%20for)

- **How do you develop a smart contract using CosmWasm?** (Как создается смарт-контракт с использованием CosmWasm?)

- **How is state managed and stored in CosmWasm smart contracts?** (Как управляется и хранится состояние в смарт-контрактах CosmWasm?)

- **How do you perform contract upgrades or migrations in CosmWasm?** (Как осуществляется миграция/обновление смарт-контракта в CosmWasm?)[credmark.ai](https://credmark.ai/practice/top-cosmwasm-interview-questions-and-answers#:~:text=Implementing%20comprehensive%20access%20controls%20and,based%20permissions)

- **How can CosmWasm contracts on different blockchains communicate with each other?** (Как смарт-контракты CosmWasm взаимодействуют между разными блокчейнами?)[credmark.ai](https://credmark.ai/practice/top-cosmwasm-interview-questions-and-answers#:~:text=21,communicate%20and%20share%20state%20effectively)

- **What are some security considerations when writing CosmWasm contracts (e.g. reentrancy)?** (Какие меры безопасности нужно учитывать при разработке CosmWasm-контрактов, например защита от реентранси-атак?)[credmark.ai](https://credmark.ai/practice/top-cosmwasm-interview-questions-and-answers#:~:text=10,developing%20smart%20contracts%20in%20Cosmwasm)[credmark.ai](https://credmark.ai/practice/top-cosmwasm-interview-questions-and-answers#:~:text=23,attacks%20in%20Cosmwasm%20smart%20contracts)

## Вопросы по Solana

- **What is Solana and how does it differ from other blockchain platforms?** (Что такое Solana и чем она отличается от других блокчейн-платформ?)[diptendud.medium.com](https://diptendud.medium.com/solana-blockchain-common-interview-questions-2d2fa0c01947#:~:text=What%20is%20Solana%2C%20and%20how,differ%20from%20other%20blockchain%20platforms)

- **Explain the concept of Proof of History (PoH) in Solana.** (Объясните суть механизма Proof of History в Solana)[usebraintrust.com](https://www.usebraintrust.com/hire/interview-questions/solana-developers#:~:text=,token%20in%20various%20decentralized%20applications)[diptendud.medium.com](https://diptendud.medium.com/solana-blockchain-common-interview-questions-2d2fa0c01947#:~:text=Explain%20the%20Proof%20of%20History,concept%20in%20Solana)

- **How does Solana achieve high scalability and throughput?** (Как Solana достигает высокой масштабируемости и пропускной способности?)[usebraintrust.com](https://www.usebraintrust.com/hire/interview-questions/solana-developers#:~:text=,and%20can%20store%20arbitrary%20data)

- **How are Solana Programs (smart contracts) different from those on Ethereum?** (Чем смарт-контракты Solana (Solana Programs) отличаются от, например, Ethereum-смарт-контрактов?)[usebraintrust.com](https://www.usebraintrust.com/hire/interview-questions/solana-developers#:~:text=,Solana%3F%20Answer%3A%20Solana%20uses%20accounts)

- **What is an account in Solana and how is data stored in accounts?** (Что такое аккаунт в Solana и как данные хранятся в аккаунтах?)[usebraintrust.com](https://www.usebraintrust.com/hire/interview-questions/solana-developers#:~:text=smart%20contracts%3F%20Answer%3A%20Solana%20Programs,and%20can%20store%20arbitrary%20data)

- **How is transaction processing parallelized in Solana?** (Как параллелизуется обработка транзакций в Solana?)[usebraintrust.com](https://www.usebraintrust.com/hire/interview-questions/solana-developers#:~:text=,Sealevel%2C%20which%20parallelizes%20transaction%20processing)

- **What is the Solana Runtime (Sealevel) and how does it enable parallel execution?** (Что такое Runtime Solana (Sealevel) и как он обеспечивает параллельное исполнение транзакций?)[usebraintrust.com](https://www.usebraintrust.com/hire/interview-questions/solana-developers#:~:text=and%20other%20blockchains.%20%20,specific%20hardware%20features)

- **What role does the SOL token play in the Solana ecosystem?** (Какова роль токена SOL в экосистеме Solana?)[usebraintrust.com](https://www.usebraintrust.com/hire/interview-questions/solana-developers#:~:text=contracts%20written%20in%20popular%20programming,token%20in%20various%20decentralized%20applications)

- **What developer tools and libraries does Solana provide?** (Какие инструменты предоставляет Solana разработчикам смарт-контрактов?)[usebraintrust.com](https://www.usebraintrust.com/hire/interview-questions/solana-developers#:~:text=,between%20Solana%20and%20other%20blockchains)[usebraintrust.com](https://www.usebraintrust.com/hire/interview-questions/solana-developers#:~:text=earn%20rewards.%20%20,js%20utilities)

- **How do cross-chain bridges (e.g. Wormhole) allow interoperability with Solana?** (Как кросс-чейн мосты вроде Wormhole обеспечивают совместимость Solana с другими сетями?)[usebraintrust.com](https://www.usebraintrust.com/hire/interview-questions/solana-developers#:~:text=is%20a%20set%20of%20tools,can%20be%20transferred%20between%20Solana)

- **Have you used Anchor for Solana development?** (Приходилось ли вам пользоваться фреймворком Anchor при разработке на Solana?)

## Прочие технические и доменные вопросы

- **What is your experience with public-key cryptography?** (Есть ли у вас опыт работы с криптографией с открытым ключом?)[glassdoor.co.in](https://www.glassdoor.co.in/Interview/rust-developer-interview-questions-SRCH_KO0,14_IP2.htm#:~:text=20%20Sept%202023)

- **How would you optimize a database query that handles millions of records?** (Как бы вы оптимизировали запрос к базе данных с миллионами записей?)[glassdoor.co.in](https://www.glassdoor.co.in/Interview/rust-developer-interview-questions-SRCH_KO0,14_IP2.htm#:~:text=20%20Sept%202023)

- **Do you have experience with real-time data streaming?** (Есть ли у вас опыт работы с потоковой обработкой данных?)[glassdoor.co.in](https://www.glassdoor.co.in/Interview/rust-developer-interview-questions-SRCH_KO0,14_IP4.htm#:~:text=28%20Jun%202023)

- **Have you worked with high-frequency or high-speed trading systems?** (Приходилось ли вам разрабатывать системы высокочастотной торговли?)[glassdoor.co.in](https://www.glassdoor.co.in/Interview/remote-rust-developer-interview-questions-SRCH_IL.0,6_KO7,21.htm#:~:text=19%20Mar%202022)

- **What is FFI (Foreign Function Interface) and have you used it in Rust?** (Что такое Foreign Function Interface (FFI) и приходилось ли вам его использовать в Rust?)

- **How do you debug Rust programs?** (Как вы отлаживаете программы на Rust?)

- **How do you ensure code you write is portable across different platforms?** (Как вы обеспечиваете переносимость вашего кода Rust между разными платформами?)

- **Please describe your Rust experience.** (Расскажите о своем опыте работы с Rust)[glassdoor.co.in](https://www.glassdoor.co.in/Interview/remote-rust-developer-interview-questions-SRCH_IL.0,6_KO7,21.htm#:~:text=16%20Oct%202023)

- **What is your skill level with Rust?** (Как бы вы оценили свой уровень владения Rust?)[glassdoor.co.in](https://www.glassdoor.co.in/Interview/rust-developer-interview-questions-SRCH_KO0,14_IP4.htm#:~:text=3)

- **Could you show or discuss an original Rust project you have worked on?** (Можете ли вы показать оригинальный проект на Rust, который вы разработали?)[glassdoor.co.in](https://www.glassdoor.co.in/Interview/rust-developer-interview-questions-SRCH_KO0,14_IP4.htm#:~:text=5%20Feb%202024)

- **What blockchain or crypto projects have you worked on, and what was your role?** (Над какими блокчейн/крипто-проектами вы работали и какова была ваша роль?)[glassdoor.co.in](https://www.glassdoor.co.in/Interview/rust-developer-interview-questions-SRCH_KO0,14_IP4.htm#:~:text=15%20Feb%202021)

- **Implement a simple key-value store in Rust.** (Реализуйте простое хранилище ключ-значение на Rust)[glassdoor.co.in](https://www.glassdoor.co.in/Interview/remote-rust-developer-interview-questions-SRCH_IL.0,6_KO7,21.htm#:~:text=14%20May%202025)

- **Implement an allocator that allocates memory at addresses that are multiples of 10.** (Реализуйте аллокатор, который выделяет память по адресам, кратным 10)[glassdoor.co.in](https://www.glassdoor.co.in/Interview/rust-developer-interview-questions-SRCH_KO0,14_IP2.htm#:~:text=14%20Feb%202021)

- **Write a program that reads numbers from stdin and then writes them out, using as many advanced Rust concepts as possible.** (Напишите программу, которая считывает числа из stdin и выводит их, задействовав по возможности все сложные возможности Rust)[glassdoor.co.in](https://www.glassdoor.co.in/Interview/rust-developer-interview-questions-SRCH_KO0,14_IP4.htm#:~:text=1%20Nov%202022)

- **Pair programming task: implement a Rust struct (and its constructor) with generic traits to accept different types of strings.** (Задание в парном программировании: реализуйте структуру в Rust с конструктором, используя обобщенные типы/трейт-границы, чтобы принимать разные виды строк)