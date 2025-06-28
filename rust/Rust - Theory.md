# Table of contents

- [Table of contents](#table-of-contents)
- [Introduction](#introduction)
  - [Compare langs](#compare-langs)
    - [System programming](#system-programming)
    - [Пример проблемного C кода](#пример-проблемного-c-кода)
  - [`Hello world!`](#hello-world)
- [Types](#types)
  - [Integers](#integers)
  - [Floats and bools](#floats-and-bools)
  - [Arithmetic](#arithmetic)
    - [Работа с переполнением](#работа-с-переполнением)
  - [Числа с плавающей точкой](#числа-с-плавающей-точкой)
  - [Tuple](#tuple)
  - [Shadowing](#shadowing)
  - [Array](#array)
  - [References](#references)
    - [Сравнение с C++](#сравнение-с-c)
  - [Pointers](#pointers)
    - [Box](#box)
  - [Functions](#functions)
  - [Expressions and Statements – Выражения и операторы](#expressions-and-statements--выражения-и-операторы)
    - [Выражения (Expressions)](#выражения-expressions)
    - [Операторы (Statements)](#операторы-statements)
  - [Structures – структуры](#structures--структуры)
    - [Инициализация структуры](#инициализация-структуры)
  - [Generics](#generics)
  - [Conditions and loops: if, while, for, loop](#conditions-and-loops-if-while-for-loop)
    - [if](#if)
    - [while](#while)
    - [loop](#loop)
    - [Inner \& outer loops](#inner--outer-loops)
    - [for](#for)
  - [Enums](#enums)
    - [Important enums in std](#important-enums-in-std)
  - [Match](#match)
    - [Collect в контейнеры](#collect-в-контейнеры)
    - [Match с несколькими объектами](#match-с-несколькими-объектами)
    - [Match с диапазонами и OR паттернами](#match-с-диапазонами-и-or-паттернами)
    - [Match с guards](#match-с-guards)
    - [Match как expression](#match-как-expression)
    - [Destructuring структур](#destructuring-структур)
    - [Binding values to names](#binding-values-to-names)
    - [Binding values с массивами](#binding-values-с-массивами)
    - [Enumerations](#enumerations)
  - [If let](#if-let)
  - [While let](#while-let)
  - [Enumerations in Rust vs C++](#enumerations-in-rust-vs-c)
    - [Оптимизация размера enum](#оптимизация-размера-enum)
    - [Тесты размера](#тесты-размера)
  - [Vector](#vector)
  - [Slices](#slices)
  - [`Panic!`](#panic)
    - [Полезные макросы которые вызывают `panic!`](#полезные-макросы-которые-вызывают-panic)
  - [`Println!`](#println)
  - [Inhabited и uninhabited types](#inhabited-и-uninhabited-types)
  - [Borrow checker](#borrow-checker)
  - [Examples with vector](#examples-with-vector)
  - [Правила владения](#правила-владения)
  - [Общие правила Borrowing](#общие-правила-borrowing)
  - [Lifetime](#lifetime)
  - [Drop Implementation](#drop-implementation)
  - [Drop Flags](#drop-flags)
  - [Borrowing examples](#borrowing-examples)
- [Standard Library](#standard-library)
  - [Option](#option)
    - [Option API](#option-api)
    - [`.unwrap()` и `.expect()`](#unwrap-и-expect)
    - [Магическая функция `.as_ref()`](#магическая-функция-as_ref)
    - [Отображение `Option<T>` в `Option<U>`](#отображение-optiont-в-optionu)
    - [Различные методы Option](#различные-методы-option)
    - [Option и ownership](#option-и-ownership)
    - [Option API и ownership: take](#option-api-и-ownership-take)
- [Traits - Трейты](#traits---трейты)
  - [Трейты с дефолтной реализацией](#трейты-с-дефолтной-реализацией)
    - [Трейты как параметры](#трейты-как-параметры)
    - [Важные встроенные трейты](#важные-встроенные-трейты)
  - [Result и обработка ошибок](#result-и-обработка-ошибок)
    - [Основы Result](#основы-result)
    - [Методы Result](#методы-result)
    - [Оператор ?](#оператор-)
  - [Closures - Замыкания](#closures---замыкания)
    - [Основы замыканий](#основы-замыканий)
    - [Захват переменных](#захват-переменных)
  - [Iterators - Итераторы](#iterators---итераторы)
    - [Основы итераторов](#основы-итераторов)
    - [Адаптеры итераторов](#адаптеры-итераторов)
    - [Потребляющие адаптеры](#потребляющие-адаптеры)
  - [Modules и Crates](#modules-и-crates)
    - [Модули](#модули)
    - [use ключевое слово](#use-ключевое-слово)
  - [Testing - Тестирование](#testing---тестирование)
    - [Unit тесты](#unit-тесты)
    - [Макросы для тестирования](#макросы-для-тестирования)
  - [Concurrency - Многопоточность](#concurrency---многопоточность)
    - [Threads - Потоки](#threads---потоки)
    - [Channels - Каналы](#channels---каналы)
    - [Shared state - Разделяемое состояние](#shared-state---разделяемое-состояние)
  - [Smart Pointers - Умные указатели](#smart-pointers---умные-указатели)
    - [`Rc<T>` - Reference Counted](#rct---reference-counted)
    - [`RefCell<T>` - Interior Mutability](#refcellt---interior-mutability)
  - [Cargo - Система сборки](#cargo---система-сборки)
    - [Основы Cargo](#основы-cargo)
    - [Cargo команды](#cargo-команды)
  - [Async/Await - Асинхронное программирование](#asyncawait---асинхронное-программирование)
    - [Основы async/await](#основы-asyncawait)
    - [Futures](#futures)
    - [Async блоки и closures](#async-блоки-и-closures)
    - [Работа с несколькими futures](#работа-с-несколькими-futures)
  - [Macros - Макросы](#macros---макросы)
    - [Declarative Macros (macro\_rules!)](#declarative-macros-macro_rules)
    - [Макрос `vec!`](#макрос-vec)
    - [Procedural Macros](#procedural-macros)
  - [Unsafe Rust](#unsafe-rust)
    - [Когда использовать unsafe](#когда-использовать-unsafe)
    - [Создание safe абстракций над unsafe кодом](#создание-safe-абстракций-над-unsafe-кодом)
    - [FFI (Foreign Function Interface)](#ffi-foreign-function-interface)
  - [Advanced Lifetimes](#advanced-lifetimes)
    - [Lifetime annotations](#lifetime-annotations)
    - [Static lifetime](#static-lifetime)
    - [Lifetime bounds](#lifetime-bounds)
  - [Advanced Type System](#advanced-type-system)
    - [Associated Types](#associated-types)
    - [Default Type Parameters](#default-type-parameters)
    - [Newtype Pattern](#newtype-pattern)
    - [Type Aliases](#type-aliases)
  - [Performance и оптимизация](#performance-и-оптимизация)
    - [Профилирование](#профилирование)
    - [Zero-cost abstractions](#zero-cost-abstractions)
    - [Избегание аллокаций](#избегание-аллокаций)
    - [Использование Cow (Clone on Write)](#использование-cow-clone-on-write)

---

# Introduction

## Compare langs

**C, C++**

- быстрые
- нет централизованной системы сборки и управления зависимостями
- undefined behavior, memory unsafety (70% ошибок в ПО)

**Java, C#, Go**

- имеют garbage collector, который иногда вызывается и замедляет работу программы/приложения
- не подходят для системного программирования

### System programming

- программы которые взаимодействуют с другими программами, а не с пользователем в первую очередь
- быстро, безопасно, стабильно (например из-за garbage collector стабильность падает)

**Rust**

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

## `Hello world!`

```rust
fn main() {
    println!("Hello, World!");  
    //   ~~~^ макрос (блокирует std::io чтоб один поток писал)
}
```

```bash
$ rustc main.rs
$ ./main
```

---

# Types

## Integers

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

## Floats and bools

```rust
// Типы с плавающей точкой
let f1: f32 = 0.0;
let f2: f64 = 0.0;

// Булевы значения
let flag: bool = true;
```

`bool` может быть только `true` или `false`

## Arithmetic

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

## Числа с плавающей точкой

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

---

## Tuple

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

---

## Shadowing

```rust
let x = 10;
for i in 0..5 {
    if x == 10 {
        println!("{x}");
        let x = 12; 
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

## Array

Размер массива постоянный и известен на этапе компиляции:

```rust
let xs: [u8; 3] = [1, 2, 3];

assert_eq!(xs[0], 1); // index -- usize
assert_eq!(xs.len(), 3); // len() -- usize

let mut buf = [0u8; 1024];
```

---

## References

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

### Box

Указатель на какие-то данные в куче. Похожи на C++ `std::unique_ptr` но без NULL.

```rust
let x: Box<i32> = Box::new(92);
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

**В Rust нет перегрузки методов**

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

## Borrow checker

## Examples with vector

```rust
let mut v = vec![1, 2, 3];
let x = &v[0]; 
v.push(4); // ссылка может испортиться т.к. при push в вектор, может реалоцироваться память
println!("{}", x);
```

В Rust есть **move** семантика, поэтому `sum` будет забирать владение  
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

## Правила владения

- Каждое value в Rust имеет переменную **owner**
- В один момент времени у value может быть только один **owner**
- Когда **owner** выходит из зоны видимости, value деинициализируется

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

## Общие правила Borrowing

- Может быть множество немутабельных (immutable) ссылок одновременно
- Но mutable ссылка может быть только одна  
  
  **=> Контракт** – в один момент времени есть либо много immutable ссылок, либо одна mutable  
  
- **Интуиция** – либо много reader, либо один writer

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

---

# Standard Library

## Option

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

```rust
let result = Some("string");
match result {
    Some(s) => println!("string inside: {s}"),
    None => println!("NO string inside"),
}
```

### `.unwrap()` и `.expect()`

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

---

# Traits - Трейты

- это основа системы типов Rust, определяющая общее поведение.
- похож на интерфейс в других языках

```rust
// Определение трейта
trait Summary {
    fn summarize(&self) -> String;
}

// Реализация трейта для типа
struct NewsArticle {
    headline: String,
    author: String,
    content: String,
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {}", self.headline, self.author)
    }
}

// Использование
let article = NewsArticle {
    headline: String::from("Breaking News"),
    author: String::from("John Doe"),
    content: String::from("Content here..."),
};

println!("{}", article.summarize());
```

## Трейты с дефолтной реализацией

```rust
trait Summary {
    fn summarize_author(&self) -> String;

    fn summarize(&self) -> String {
        format!("(Read more from {}...)", self.summarize_author())
    }
}

impl Summary for NewsArticle {
    fn summarize_author(&self) -> String {
        format!("@{}", self.author)
    }
    // summarize() использует дефолтную реализацию
}
```

### Трейты как параметры

```rust
// Trait bound syntax
fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}

// impl Trait syntax
fn notify(item: &impl Summary) {
    println!("Breaking news! {}", item.summarize());
}

// Множественные trait bounds
fn notify<T: Summary + Display>(item: &T) {
    // ...
}

// Where clause для сложных bounds
fn some_function<T, U>(t: &T, u: &U) -> i32
where
    T: Display + Clone,
    U: Clone + Debug,
{
    // ...
}
```

### Важные встроенные трейты

```rust
// Clone - создание копии
#[derive(Clone)]
struct Point {
    x: i32,
    y: i32,
}

// Copy - побитовое копирование (автоматически для простых типов)
#[derive(Copy, Clone)]
struct Point {
    x: i32,
    y: i32,
}

// Debug - для отладочного вывода
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

// PartialEq - для сравнения
#[derive(PartialEq)]
struct Person {
    name: String,
    age: u32,
}
```

---

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

### Оператор ?

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

## Closures - Замыкания

### Основы замыканий

```rust
// Простое замыкание
let add_one = |x| x + 1;
println!("{}", add_one(5)); // 6

// Замыкание с типами
let add_one = |x: i32| -> i32 { x + 1 };

// Многострочное замыкание
let expensive_closure = |num| {
    println!("Calculating slowly...");
    std::thread::sleep(std::time::Duration::from_secs(2));
    num
};
```

### Захват переменных

```rust
fn main() {
    let x = 4;

    // Захват по неизменяемой ссылке
    let equal_to_x = |z| z == x;
    println!("{}", equal_to_x(4)); // true

    // Захват по изменяемой ссылке
    let mut y = 5;
    let mut add_to_y = |z| y += z;
    add_to_y(3);
    println!("{}", y); // 8

    // Захват по значению (move)
    let x = vec![1, 2, 3];
    let equal_to_x = move |z| z == x;
    // x больше недоступен в этой области видимости
}
```

---

## Iterators - Итераторы

### Основы итераторов

```rust
let v1 = vec![1, 2, 3];

// Создание итератора
let v1_iter = v1.iter(); // &T
let v1_iter = v1.into_iter(); // T
let mut v1 = vec![1, 2, 3];
let v1_iter = v1.iter_mut(); // &mut T

// Использование итератора
for val in v1.iter() {
    println!("{}", val);
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

---

## Modules и Crates

### Модули

```rust
// src/lib.rs или src/main.rs
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
        
        fn seat_at_table() {} // private
    }
    
    mod serving {
        fn take_order() {}
        fn serve_order() {}
        fn take_payment() {}
    }
}

// Использование
pub fn eat_at_restaurant() {
    // Абсолютный путь
    crate::front_of_house::hosting::add_to_waitlist();
    
    // Относительный путь
    front_of_house::hosting::add_to_waitlist();
}
```

### use ключевое слово

```rust
use crate::front_of_house::hosting;
// или
use self::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}

// Переименование
use std::fmt::Result;
use std::io::Result as IoResult;

// Группировка
use std::{cmp::Ordering, io};
use std::collections::{HashMap, BTreeMap, HashSet};

// Glob operator
use std::collections::*;
```

---

## Testing - Тестирование

### Unit тесты

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }

    #[test]
    fn another() {
        panic!("Make this test fail");
    }
    
    #[test]
    #[should_panic]
    fn this_should_panic() {
        panic!("Expected panic");
    }
    
    #[test]
    #[should_panic(expected = "specific error message")]
    fn panic_with_message() {
        panic!("specific error message occurred");
    }
    
    #[test]
    fn test_result() -> Result<(), String> {
        if 2 + 2 == 4 {
            Ok(())
        } else {
            Err(String::from("two plus two does not equal four"))
        }
    }
}
```

### Макросы для тестирования

```rust
#[test]
fn test_assertions() {
    assert!(true);
    assert_eq!(4, 2 + 2);
    assert_ne!(4, 2 + 3);
    
    // Кастомные сообщения
    assert!(
        true,
        "This should be true but got {}",
        false
    );
}
```

---

## Concurrency - Многопоточность

### Threads - Потоки

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }

    handle.join().unwrap(); // Ждём завершения потока
}
```

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

## Smart Pointers - Умные указатели

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

### `RefCell<T>` - Interior Mutability

```rust
use std::cell::RefCell;
use std::rc::Rc;

#[derive(Debug)]
enum List {
    Cons(Rc<RefCell<i32>>, Rc<List>),
    Nil,
}

use crate::List::{Cons, Nil};

fn main() {
    let value = Rc::new(RefCell::new(5));

    let a = Rc::new(Cons(Rc::clone(&value), Rc::new(Nil)));
    let b = Cons(Rc::new(RefCell::new(3)), Rc::clone(&a));
    let c = Cons(Rc::new(RefCell::new(4)), Rc::clone(&a));

    *value.borrow_mut() += 10;

    println!("a after = {:?}", a);
    println!("b after = {:?}", b);
    println!("c after = {:?}", c);
}
```

---

## Cargo - Система сборки

### Основы Cargo

```toml
# Cargo.toml
[package]
name = "my_project"
version = "0.1.0"
edition = "2021"
authors = ["Your Name <you@example.com>"]
description = "A sample project"

[dependencies]
serde = "1.0"
serde_json = "1.0"
tokio = { version = "1.0", features = ["full"] }

[dev-dependencies]
criterion = "0.3"

[[bin]]
name = "server"
path = "src/bin/server.rs"

[profile.release]
opt-level = 3
debug = false
```

### Cargo команды

```bash
# Создание проекта
cargo new my_project
cargo new my_library --lib

# Сборка и запуск
cargo build
cargo run
cargo build --release

# Тестирование
cargo test
cargo test -- --nocapture  # показать println!
cargo test specific_test    # запуск конкретного теста

# Документация
cargo doc
cargo doc --open

# Проверка кода
cargo check
cargo clippy
cargo fmt

# Работа с зависимостями
cargo update
cargo tree
```

---

## Async/Await - Асинхронное программирование

### Основы async/await

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

---

## Macros - Макросы

### Declarative Macros (macro_rules!)

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

## Unsafe Rust

### Когда использовать unsafe

```rust
// Разыменование сырых указателей
unsafe fn dangerous() {
    let mut num = 5;
    
    let r1 = &num as *const i32;
    let r2 = &mut num as *mut i32;
    
    unsafe {
        println!("r1 is: {}", *r1);
        println!("r2 is: {}", *r2);
    }
}

// Вызов unsafe функций
unsafe fn unsafe_function() {
    println!("This is unsafe");
}

fn main() {
    unsafe {
        unsafe_function();
    }
}
```

### Создание safe абстракций над unsafe кодом

```rust
use std::slice;

fn split_at_mut(slice: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
    let len = slice.len();
    let ptr = slice.as_mut_ptr();
    
    assert!(mid <= len);
    
    unsafe {
        (
            slice::from_raw_parts_mut(ptr, mid),
            slice::from_raw_parts_mut(ptr.add(mid), len - mid),
        )
    }
}
```

### FFI (Foreign Function Interface)

```rust
// Вызов C функций
extern "C" {
    fn abs(input: i32) -> i32;
}

fn main() {
    unsafe {
        println!("Absolute value of -3 according to C: {}", abs(-3));
    }
}

// Экспорт Rust функций для C
#[no_mangle]
pub extern "C" fn call_from_c() {
    println!("Just called a Rust function from C!");
}
```

---

## Advanced Lifetimes

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

---

## Advanced Type System

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

## Performance и оптимизация

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
