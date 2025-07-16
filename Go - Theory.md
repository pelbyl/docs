# Главная ссылка `https://github.com/golang/go`

Актуальная версия Go 1.24

Императивный язык.
## Плюсы
- Поддержка параллелизма через горутины
- Каналы для синхронизации
- Встроенные инструменты для разработки
  - go fmt - форматирование кода
  - go vet - статический анализ кода
  - go test - встроенный тестовый фреймворк с поддержкой unit тестов
  - go mod - управление зависимостями
  - runtime/pprof
  - runtime/trace
  - expvar – мониторинг в реальном времени через HTTP
  - delve – дебаггер
- Garbage Collection
- Статическая компиляция и независимые бинарные файлы
## Минусы
- Медленный Garbage Collection
- Нет явного управления:
  - памятью
  - потоками
- Нет наследования
- Каждый инстанс Go тащит свой собственный runtime за собой
- Отсутствие ковариантности и контрвариантности

```go
type Animal interface {
  Speak() string
}

type Dog struct {}

func (d Dog) Speak() string {
  return "Woof"
}

func main() {
  animals := []Animal{}  // Массив животных
  dogs := []Dog{Dog{}}

  // Нельзя присвоить массив Dog в массив Animal
  // animals = dogs  // Ошибка компиляции
}
```
## Средства обобщенного программирования
- Generic
- Кодогенерация
- Интерфейсы
- Рефлексия
---
# Data Types / Типы данных
1) Базовые: числа, строки, booleans
2) Аггрегированные: массивы и структуры
3) Ссылочные: указатели, слайсы, мапы, функции и каналы
4) Интерфейсы

## String / Строка

Последовательность байтов, интерпретируемых как текст в UTF-8

```go
// StringHeader описывает внутреннее представление строки.
// Используется только для отражения (reflection)
// и не предназначен для прямого использования в приложениях.

type StringHeader struct {
	Data uintptr // указатель на данные строки (массив байтов)
	Len  int     // длина строки

}
```

- Immutable - нельзя изменять после создания. Любая операция изменения строки на самом деле вернет новую строку
- Внутри
  Это структура содержащая указатель на массив байтов и длину
  Емкости нет, поэтому данные строки не могут быть изменены
- Кодировка
  UTF-8 хранит и ASCII и многоязычный текст.
  Индексация возвращает байты, а не символы  
### Эффективная конкатенация большого количества строк #Q #String
- strings.Builder
- `Make` слайс рун нужного размера, затем `concat = append(concat, []rune(str1), []rune(str2), []rune(str3))`
### Сравнение строк #Q #String
- `strings.Compare`
- по длине нельзя, тк длина строки - это число байт на кодировку UTF-8 символа
### Подстрока из строки #Q #String
- strings.Substring
- преобразовать строку в слайс рун и с помощью слайса получить подстроку
### Преобразование строки в число и обратно #Q #String 
- `strconv.Atoi(s)`
- `strconv.ParseFloat(s, 64)` -> float64
- `strconv.Itoa(n)`

## Rune
Тип rune это синоним `int32`, используется для представления Unicode символов.

Rune[] - это слайс, который представляет строку в виде набора отдельных символов. Используется для посимвольной итерации.

Преобразование строки в rune гарантирует что каждый элемент будет представлять Unicode символ.

## Побитовые операции

- Применимы к целочисленным типам: int, uint, int32, uint32, int64, uint64
- Нельзя применять к строкам или срезам
- Операторы сдвига (<< и >>) имеют более высокий приоритет, чем побитовое И (&), но ниже, чем арифметические операторы умножения и деления

```
& (AND)
| (OR)
^ (XOR)
<< (left shift)
>> (right shift)
```

---
# Data Structures (Standard Library)

## Map

<https://go.dev/src/runtime/map.go>

- Вставка по ключу
- Удаление по ключу
- Добавление по ключу
- Операции выполняются за O(1) время

**Buckets** группы в которых хранятся записи map, дает константное время поиска.

Для равномерного распределения записей по buckets используется хеш-функция `bucket = hash(key)`.
- Равномерность
- Детерминированность
- Быстродействие
- Криптографическая стойкость (не позволяет подобрать ключи так, что бы значения были в одном bucket)

```go
value, ok := m[key]
```

- Операции выполняются с **unsafe.Pointer** для ускорения работы с картой.
- Мета-информация хранится в type descriptor
- **type descriptor** хранит информацию о типе ключа и значения, а также о функциях сравнения и хеширования, предоставляет операции hash, equal и copy

```go
// дескриптор типа
type _type struct{
    size       uintptr
    ptrdata    uintptr
    hash       func(unsafe.Pointer, uintptr) uintptr
    eq         func(unsafe.Pointer, unsafe.Pointer) bool
    gcdata     *byte
    str        int32
    ptrToThis  int32
}

// дескриптор Map
type maptype struct {
    typ    _type
    key    *_type
    elem   *_type
    bucket *_type
    hasher func(unsafe.Pointer, uintptr) uintptr
    keysize uint8
    elemsize uint8
    bucketsize uint16
    flags uint32
}
```

#### Поиск значения по ключу

---

```go
value := m[key]

pk := unsafe.Pointer(&key)
func lookup(t *maptype, m *mapHeader, key unsafe.Pointer) (unsafe.Pointer, bool)

pv := runtime.lookup(typeOf(m), m, pk)

v = *(*V)pv
```

#### Полная раскладка

---

```go
v:= m["k"]
func mapaccessit *maptype, h *hmap, k unsafe.Pointer) unsafe.Pointer

v, ok:= m["k"]
func mapaccess2(t *maptype, h *hmap, k unsafe.Pointer) (unsafe.Pointer, bool)

m["k"] = 9001
func mapassignt *maptype, h *hmap, k unsafe. Pointer) unsafe. Pointer

delete(m, "k")
func mapdelete(t *maptype, h *hmap, k unsafe.Pointer)
```

#### Заголовок Map (hmap struct)

---
<https://go.dev/src/runtime/map.go>

```go
type hmap struct {
    // число элементов, должно быть первым в структуре тк используется функцей len()
    count     int       
    flags     uint8     
    // iterator - есть активный итератор)
	 // olditerator - промежуточное состояние, данные из старых бакетов переносятся в новые
	 // growing - карта в процессе расширения, в это время Go поддерживает два массива бакетов: старый и новый

    B         uint8     // log_2 от числа бакетов (can hold up to loadFactor * 2^B items)
    noverflow uint16    // approximate number of overflow buckets; see incrnoverflow for details
    hash0     uint32    // hash seed, для безопасности
    
    buckets    unsafe.Pointer // array of 2^B Buckets. may be nil if count==0.
                              // указатель на список бакетов
    oldbuckets unsafe.Pointer // previous bucket array of half the size, non-nil only when growing
    nevacuate  uintptr        // progress counter for evacuation (buckets less than this have been evacuated)
   
    extra *mapextra // optional fields
}
```

#### Low oreder bits of hash

---

1) Получаем хеш ключа `Hash(key) = 54234`
2) Получаем промежуточный номер побитовой операцией `Bucket = 54234 & (2^B - 1)`
3) Далее берется B младших битов от хеша `Hash(key) >> B`

### Структура Bucket

---
*8 слотов* для старших битов хеша, для того чтобы быстро проверить есть ли в бакете нужный ключ пройдясь только по ним.

Если да, то сравнивается каждый ключ в бакете с ключом который мы ищем

Не более 8 элементов в бакете

```go
// Заголовок: HOB (High Order Bits Hash), K – ключ, V – значение

// Хэш-ячейки (8 ячеек):
0           1           2           3           4           5           6           7   
+--------+  +--------+  +--------+  +--------+  +--------+  +--------+  +--------+
|  HOB   |  |  HOB   |  |  HOB   |  |  HOB   |  |  HOB   |  |  HOB   |  |  Empty |  |  Empty |
+--------+  +--------+  +--------+  +--------+  +--------+  +--------+  +--------+  +--------+

// Соответствующее хранение ключей и значений (каждый блок – 16 ячеек):
// Индексы: 0..7 для ключей, 8..15 для значений

0     1     2     3     4     5     6     7      8     9     10    11    12    13    14    15
+---+ +---+ +---+ +---+ +---+ +---+ +---+ +---+  +---+ +---+ +---+ +---+ +---+ +---+ +---+ +---+
| K | | K | | K | | K | | K | |   | |   | |   |  | V | | V | | V | | V | | V | |   | |   | |   |
+---+ +---+ +---+ +---+ +---+ +---+ +---+ +---+  +---+ +---+ +---+ +---+ +---+ +---+ +---+ +---+
```

Ключи и значения хранятся в виде списка. Сначала ключи, затем значения.

Такой порядок обусловлен **выравниванием типов**.

#### Переполнение бакета (попытка положить 9 элемент)

В таком случае создается новый бакет.

#### Load factor(таблица переполнения бакетов для мап)

---

```go
// Picking loadFactor: too large and we have lots of overflow
// buckets, too small and we waste a lot of space. I wrote
// a simple program to check some stats for different loads:
// (64-bit, 8 byte keys and elems)
//  loadFactor    %overflow  bytes/entry     hitprobe    missprobe
//        4.00         2.13        20.77         3.00         4.00
//        4.50         4.05        17.30         3.25         4.50
//        5.00         6.85        14.77         3.50         5.00
//        5.50        10.55        12.94         3.75         5.50
//        6.00        15.27        11.67         4.00         6.00
//        6.50        20.90        10.79         4.25         6.50
//        7.00        27.14        10.15         4.50         7.00
//        7.50        34.03         9.73         4.75         7.50
//        8.00        41.10         9.40         5.00         8.00
//
// %overflow   = percentage of buckets which have an overflow bucket
// bytes/entry = overhead bytes used per key/elem pair
// hitprobe    = # of entries to check when looking up a present key
// missprobe   = # of entries to check when looking up an absent key
//
// Keep in mind this data is for maximally loaded tables, i.e. just
// before the table grows. Typical tables will be somewhat less loaded.
```

#### Эвакуация данных из бакета

---
Создание новых бакетов и перенос данных из старых в новые.

- Происходит во время расширения Map, данные равномерно раскидываются по новым бакетам при новых вставках

Переполнение Map значительно увеличивает время поиска элемента. Поэтому при инициализации Map рекомендуется указывать размер.

Указатель на элемент Map нельзя взять т.к. при эвакуации Map может произойти перенос данных и указатель станет невалидным.

Но `mapaccess1` использует указатели, но сразу после использования удаляет их.
#### Случайный порядок следования элементов при итерировании по map

---
В инициализации итератора `mapiterinit` стартовое место для обхода Map выбирается рандомно.

Однако `fmt.Println()` выведет отсортированный список ключей, т.к. Map сортируется внутри Println().

---
## Вопросы Map #Q #Map 
### МОЖЕТ быть ключом в Map? #Q #Map 

**Любой тип, который можно сравнивать оператором `==`**
- Числовые типы(int, float, uint, и т.д.)
- Строки
- Булевы значения
- Указатели
- Интерфейсы, если значения в интерфейсе можно сравнивать
- Структуры, если все поля можно сравнивать
### НЕ МОЖЕТ быть ключом в Map? #Q #Map 
- Слайсы
- Мапы
- Функции
- Массивы с некомпарабельными элементами

### Nil map #Q #Map 
```go
var m map[string]int
m["key"] = 42 // panic, разименование nil указателя

// fixed
m := make(map[string] int)
```
### Ссылочная природа Map #Q #Map 
```go
func modifyMap(m map[int]string) {
    m[2] = "changed"
}

func main() {
    myMap := map[int]string{1: "one", 2: "two", 3: "three"}
    modifyMap(myMap)
    fmt.Println(myMap) // {1: "one", 2: "changed", 3: "three"}
}
```
### Удаление во время итерации #Q #Map 
```go
m := map[int]bool{1: true, 2: true, 3: true}
for k := range m {
    if k == 2 {
        delete(m, k)
    }
}

// все хорошо, в Go гарантирована безопасноть
```
### Отсутствие ключа #Q #Map 
```go
m := map[string]int{ "a": 1, "b": 2,}
value, ok := m["c"]
if !ok {
	 // ключа нет
}
```
### Порядок итерации - рандом #Q #Map 
### Получить указатель на элемент мапы - НЕЛЬЗЯ #Q #Map 
### Мапа не потокобезопасна  #Q #Map 
Если несколько горутин читают или пишут в мапу то без синхронизации может быть гонка данных и неопредленное поведение

## Массив
- фиксированная длина
- редко используется в Go для хранения фиксированных данных

В контексте блокчейна
- Можно хранить фиксированное число блоков, массив с элементами типо Block
- Фиксированное число транзакций
- Адреса и ключи

## Slice
### Внутренняя структура Slice
<https://go.dev/src/runtime/slice.go>
```go
type slice struct {
	array unsafe.Pointer // ОДИН УКАЗАТЕЛЬ НА ПОДЛЕЖАЩИЙ МАССИВ
	len   int // длина
	cap   int // емкость
}
```
### Append
```go
len = 3
cap = 6

array = {"f", "f", "@", "x", "#", "#"}
[]string{"f", "f", "@"}

append("Q")
array = {"f", "f", "@", "Q", "#", "#"}
[]string{"f", "f", "@", "Q"}
len = 4

append("w")
array = {"f", "f", "@", "Q", "w", "#"}
[]string = {"f", "f", "@", "Q", "w"}
len = 5

append("e")
array = {"f", "f", "@", "Q", "w", "e"}
[]string = {"f", "f", "@", "Q", "w", "e"}
len = 6 // слайс заполнен, в базовый массив не влезут элементы

append("n")
new_array = {"f", "f", "@", "Q", "w", "e", "n", "", "", "", "", ""}
[]string = {"f", "f", "@", "Q", "w", "e", "n"}
len = 7 // слайс заполнен, в базовый массив не влезут элементы
cap = 12
```
### Пустой Slice
- Нулевое значение Slice - nil
- Slice можно инициализировать пустым слайсом, тогда слайс будет не nil
- Проверку на пустоту нужно делать с len()
### Аллокация памяти
```go
func main() {
    list := make([]int, 5, 10)
    // len 5, cap 10
}

func double(nms []int) []int {
    // var newSlice []int
    // ПЛОХО
    // append создает новый слайс и копирует в него все элементы и каждый раз удваивает размер
    res := make([]int, 0, len(nms))
    for _, v := range nms {
        newSlice = append(newSlice, v*2)
    }
    // либо
    res2 := make([]int, len(nms))
    for i, v := range nms {
        res2[i] = v * 2
    }
}
```

### Передача по значению и по ссылке
```go
func main() {
    list := []int{1, 2, 3, 4}
    fmt.Println(list)

    handle(list)
    fmt.Println(list) // [100 2 3 4] значение одного из элементов изменилось

    handle2(list)
    fmt.Println(list) // [100 2 3 4] ничего не изменилось тк мы смотрим исходный слайс а не тот что вернула функция append

    handle3(list[:1]) // передаем кусок слайса
    fmt.Println(list) // [100 5 3 4] значение изменилось, базовый массив остается тем же, увеличилась его используемая часть [100 2] -> [100 5]

    handle4(list[:1])
    fmt.Println(list) // [100 10 3 4], внешний слайс не меняется
}

// передача по значению слайса, но базовый массив передается по ссылке все равно
func handle(nms []int) {
    nms[0] = 100
}

func handle2(list []int) {
    _ = append(list, 5)
}

func handle3(list []int) {
   list = append(list, 5)
   fmt.Println(list) // [100 5]
}

func handle4(list []int) {
    newList := make([]int, len(list))
    copy(newList, list)
    newList = append(newList, 10)
    fmt.Println(newList) // [100 10]
}
```
- длина и емкость базового массива передается по значению всегда
- если хотим новый массив, делаем его с make

### Правильное использование append

```go
func main() {
    list := make([]int, 4, 4)
    list2 := append(list, 1)

    list[0] = 5
    list2[0] = 9

    fmt.Println(list) // [5 0 0 0]
    fmt.Println(list2) // [9 0 0 0 1]

    list3 := make([]int, 4, 5)
    list4 := append(list, 1)

    list3[0] = 5
    list4[0] = 9

    fmt.Println(list3) // [9 0 0 0] тк базовый массив один и append не создает новый массив
    fmt.Println(list4) // [9 0 0 0 1]
}
```

## Вопросы Slices #Q #Slice
### len(), cap() #Q #Slice 
```go
// Q1
a := [5]int{1, 2, 3, 4, 5} // массив len(a) = 5, cap(a) = 5
s := a[1:4] // слайс len(s) = 3, cap(s) = 4

// Q2
base := []int{10, 20, 30, 40}
newSlice := base[1:3] // {20, 30, 40}
newSlice[1] = 50 // {20, 50, 40}
// base = {10, 20, 50, 40}

// Q3
original := make([]int, 3, 5) // {0, 0, 0, _, _}
original = append(original, 1, 2, 3) // {0, 0, 0, 1, 2, 3, _, _, _, _}
// len(original) = 6, cap(original) = 10

```
### Отличие nil slice и empty slice #Q #Slice 
```go
var nilSlice []int // значение nil, не имеет выделенной памяти
emptySlice := make([]int, 0) // значение 0, выделена памать, len 0, cap 0

nilSlice == nil // true
emptySlice == nil // false
```
## Значение после выполнения кода #Q #Slice 
```go
// Q1
slices := [][]int{
	{1, 2},
	{3, 4},
}
slices[0] = append(slices[0], 3)

// {1, 2, 3} {3, 4}

//Q2
taskList := []string{ "first", "second", "third",}

wakeup := taskList[0:2] // len 2, cap 3 {"first", "second",}
work := taskList[2:3]  // len 1, cap 1 {"third",}

wakeup = append(wakeup, "fourth") // {"first", "second", "fourth"}
// work {"fourth"}
```
## Обрезание слайса, память #Q #Slice 

- Пока емкости достаточно – память не реалоцируется;
- Пока на массив есть указатели garbage collector не очистит; 
- Реалокация памяти и перенос значений в новый слайс -> garbage collector очистит память старого массива;
- Можно скопировать данные в новый слайс с помощью `copy`.

## List #LinkedList 
Source code:
<https://cs.opensource.google/go/go/+/master:src/container/list/list.go>
### Element

```go
// Element is an element of a linked list.
type Element struct {
    // Next and previous pointers in the doubly-linked list of elements.
    // To simplify the implementation, internally a list l is implemented
    // as a ring, such that &l.root is both the next element of the last
    // list element (l.Back()) and the previous element of the first list
    // element (l.Front()).
    next, prev *Element

    // The list to which this element belongs.
    list *List

    // The value stored with this element.
    Value any
}

// Next returns the next list element or nil.
func (e *Element) Next() *Element {
    if p := e.next; e.list != nil && p != &e.list.root {
        return p
    }
    return nil
}

// Prev returns the previous list element or nil.
func (e *Element) Prev() *Element {
    if p := e.prev; e.list != nil && p != &e.list.root {
        return p
    }
    return nil
}
```

### List
```go
// List represents a doubly linked list.
// The zero value for List is an empty list ready to use.
type List struct {
    root Element // sentinel list element, only &root, root.prev, and root.next are used
    len  int     // current list length excluding (this) sentinel element
}
```

### List Methods
- `Len()` returns the number of elements of list l;
- `Front()` returns the first element of list l or nil if the list is empty;
- `Init()` initializes or clears list l;
- `New()` returns an initialized list;
- `Len()` returns the number of elements of list l;
```
// Front returns the first element of list l or nil if the list is empty.
// Back returns the last element of list l or nil if the list is empty.

// lazyInit lazily initializes a zero List value.

// insert inserts e after at, increments l.len, and returns e.
// func (l *List) insert(e, at *Element) *Element

// insertValue is a convenience wrapper for insert(&Element{Value: v}, at).
// func (l *List) insertValue(v any, at *Element) *Element

// remove removes e from its list, decrements l.len

// move moves e to next to at.

// Remove removes e from l if e is an element of list l.
// It returns the element value e.Value.
// The element must not be nil.

// PushFront inserts a new element e with value v at the front of list l and returns e.

// PushBack inserts a new element e with value v at the back of list l and returns e.

// InsertBefore inserts a new element e with value v immediately before mark and returns e.
// If mark is not an element of l, the list is not modified.
// The mark must not be nil.

// InsertAfter inserts a new element e with value v immediately after mark and returns e.
// If mark is not an element of l, the list is not modified.
// The mark must not be nil.

// MoveToFront moves element e to the front of list l.
// If e is not an element of l, the list is not modified.
// The element must not be nil.

// MoveToBack moves element e to the back of list l.
// If e is not an element of l, the list is not modified.
// The element must not be nil.

// MoveBefore moves element e to its new position before mark.
// If e or mark is not an element of l, or e == mark, the list is not modified.
// The element and mark must not be nil.

// MoveAfter moves element e to its new position after mark.
// If e or mark is not an element of l, or e == mark, the list is not modified.
// The element and mark must not be nil.

// PushBackList inserts a copy of another list at the back of list l.
// The lists l and other may be the same. They must not be nil.

// PushFrontList inserts a copy of another list at the front of list l.
// The lists l and other may be the same. They must not be nil.
```
---
# Runtime GO
- Allocator
- Garbage Collector
- Scheduler
- Pprof (профилирование) и trace (трассировка)
## Allocator
Основан на TCMalloc(thread caching allocator). TCMalloc внутри работает с памятью потоков и с кучей.
Выделяется сразу большой кусок памяти Arena.
runtime/malloc.go

```go
//       Platform  Addr bits  Arena size  L1 entries   L2 entries
// --------------  ---------  ----------  ----------  -----------
//       */64-bit         48        64MB           1    4M (32MB)
// windows/64-bit         48         4MB          64    1M  (8MB)
//      ios/arm64         40         4MB           1  256K  (2MB)
//       */32-bit         32         4MB           1  1024  (4KB)
//     */mips(le)         31         4MB           1   512  (2KB)
```

Arena поделена на страницы по 8кб. Страницами управляет структура mheap.
Фрагментация arenа
### `mspan`

![go-mspan](pics/go-mspan.png)

(runtime/mheap.go)

Структура mspan представляет собой двусвязный список, объект, который содержит начальный адрес страницы, сведения о размере страницы и количество страниц, входящих в него.

Хранит информацию о пуллах.

```go
// go/src/runtime/mheap.go
type mspan struct {
    // ...
    next *mspan     // next span in list, or nil if none
    prev *mspan     // previous span in list, or nil if none
    // ...
    spanclass             spanClass     // size class and noscan (uint8)
}
```

### `mcentral`

![go-mcentral](pics/go-mcentral.png)

- Всего 67 mcentral, по одному для кадого класса;
- Каждый mcentral содержит nonempty и empty указатели.

>Много горутин пытаются получить доступ к Arena. Mutex сильно замедляет работу.

![go-allocator-concurrency](pics/go-allocator-concurrency.png)

Каждая горутина работает со своим собственным mcache.

### `mcache`
При выделении памяти горутина обращается к своему mcache без блокировки.
![go-mcache](pics/go-mcache.png)

Каждый класс размера может быть представлен одним из следующих объектов:
- Объект `scan` — это объект, который содержит указатель.
- Объект `noscan` — это объект, в котором нет указателя.
Сделано для оптимизации работы Garbage Collector. `noscan` объекты обходить не нужно, так как они не содержат объектов, под которые выделена память.
### Процесс выделения памяти под объекты

- `Tiny`(< 16Kb) - могут быть помещены в уже существующий `mspan`;
- `Small` (16Kb ~ 32Kb) - выясняется, какой класс размера (sizeClass) нужно использовать, затем в mcache выделяется подходящий блок. Выделяются в `mcache` с использованием `mspan`;
- `Large` (> 32Kb) – память выделяется в `mhep` через блокировку помещается в `mspan` в `Arena`.

- Если в `sizeClass`, соответствующем `mcache`, нет доступных блоков, производится обращение к `mcentral`.
- Если у `mcentral` нет свободных блоков, тогда обращаются к `mheap` и выполняется поиск наиболее подходящего `mspan`. Если размер памяти, необходимый приложению, оказывается большим, чем можно выделить, запрошенный размер памяти будет обработан так, чтобы можно было бы возвратить столько страниц, сколько нужно программе, сформировав новую структуру `mspan`.
## Garbage Collector

Запускается автоматически во время выполнения программы, поведение настраивается через **GODEBUG** и **runtime/debug** пакет

Используется концепция "GC pacer" - определяется оптимальное время для запуска GC основываясь на текущем потреблении памяти и на том как быстро растет потребление памяти
### Триггеры Запуска GC

- Выделение Памяти:
	Основной триггер для GC — это выделение памяти. Когда программа продолжает выделять память, и общий объем выделенной памяти достигает определенного порога, GC будет запущен.

- Ручной Запуск:
	Можно явно запустить GC, через `runtime.GC()` из стандартной библиотеки
	Но лучше полагаться на автоматический запуск GC рантаймом Go

### Конфигурация GC

- GODEBUG - включение режимов отладки
  `export GODEBUG=gctrace=1,schedtrace=1000` - трейсить GC при каждом сборе мусора и Планировщик каждые 1000 мс (каждую секунду)

- GOGC - агрессивность сборщика мусора
	- **GOGC=200** - запуск при увеличении объема памяти в 3 раза по сравнению с последним освобождением.
	- **GOGC=50** - запуск при увеличении объема памяти на 50 процентов по сравнению с последним освобождением.

### Mark and Sweep

1. **Mark (маркировка):**
	На этом этапе GC (сборщик мусора) проходит по всем объектам, доступным из "корневых" ссылок (например, глобальные переменные, стековые переменные, объекты, на которые ссылаются другие активные объекты), и отмечает их как "живые" (reachable). Это делается, чтобы понять, какие объекты все еще используются приложением.

2. **Sweep (очистка):**
	После этапа маркировки GC проходит по всей куче и освобождает память для объектов, которые не были отмечены как живые. Эти объекты считаются "мусором", поскольку к ним больше не существует доступа, и их память можно вернуть системе.

**Недостатки:**
-  Может приводить к длительным "stop-the-world" паузам, когда приложение приостанавливается для проведения GC.
-  Не оптимизирован для ситуаций, когда в куче много объектов, особенно если они часто создаются и уничтожаются.
## Scheduler

### Переключение контекста в потоках ОС

- ОС сохраняет регистры процессора, указатель на текущую инструкцию (program counter), стек и другие аппаратные ресурсы текущего потока в context structure
- Контекстное переключение происходит в ядре ОС:
  - Time slot потока истек;
  - Поток в состоянии ожидания, например, из-за ввода/вывода;
- ОС также может перемещать контекст между процессорами (CPU migration)
(дизайн планировщика общий для Rust, Kotlin, Java)

```go
// Структура g которая представляет горутину

type g struct {
    stack       stack    // диапазон стека
    stackguard0 uintptr  // граница для проверки роста стека
    atomicstatus uint32  // текущее состояние горутины
    goid        int64    // уникальный идентификатор
    sched       gobuf    // контекст выполнения
    m           *m       // указатель на поток m
    waitreason  string   // причина ожидания
    preempt     bool     // флаг прерывания
    defer       *defer   // список отложенных вызовов
    panic       *panic   // информация о панике
}
```

Планировщик управляет тремя типами ресурсов – логические процессоры, горутины, машины(поток операционной системы)
### GMP модель
- G - горутина
- M - OS thread
- P - логический процессор (виртуальное представление ядра или части ядра для ОС)

### User and Kernel Space

![go-user-kernel-space](pics/go-user-kernel-space.png)

![go-components-of-scheduler](pics/go-components-of-scheduler.png)

```go
							                         Глобальная очередь
+--------+     +--------+     +--------+      +--------------+
|   P1   |     |   P2   |     |   P3   |      |  Scheduler   |
|--------|     |--------|     |--------|      |--------------|
| G1, G2 |     | G3, G4 |     | G5, G6 |      |   G7, G8     |
+--------+     +--------+     +--------+      +--------------+
     ▲             ▲              ▲
     │             │              │
     └───── Парковка горутин ──────┘
           (выбор горутины, например, G выбрана)
                    │     через Mutex
                    ▼
         
             /      |      \
            /       |       \
      +--------+ +--------+ +--------+ Машины, выполняющие горутины 
      |   M1   | |   M2   | |   M3   |
      | Thread | | Thread | | Thread |
      +--------+ +--------+ +--------+
             ▲        ▲        ▲
             │        │        │
             └────────┴────────┘
                Исполняемая горутина (G)
```

### Добавление в очередь

1. Создается объект G который представляет горутину;
2. G помещается в локальную очередь того P который инициировал ее создание;
3. Если локальная очередь переполнена, то горутина помещается в глобальную очередь, откуда ее могут достать другие P если их очереди пустые.

### Распределение и выполнение

1. M привязанный к P достает горутину из локальной очереди для P
    – Если локальная очередь пустая, то P переходит к **ВОРОВСТВУ(WORK STEALING)** задач из локальной очереди другого P, если 4 попытки воровства пустые, то пытается взять задачи из глобальной очереди, если и она пуста P, то poll из сети.

    >**ВОРОВСТВО (WORK STEALING)** – Доступ к глобально очереди происходит через mutex. При воровстве задач планировщик используются lock-free алгоритмы.

2. Выбранную горутину M запускает для выполнения – Если горутина блокируется (например ожидает I/O), то M может временно приостановить ее выполнение и переключиться на другую горутину;

3. Парковка и завершение
– Горутины которые завершены или приостановлены возвращаются в соответствующие очереди ИЛИ помечаются как завершенные
– Объект завершенной горутины может быть возвращен в пулл свободных объектов

Основной механизм балансировки в Go – воровство задач, но также есть и WORK SHARING

Когда поток создает новую задачу (или горутину), он сразу пытается распределить её на другие потоки, если они доступны.

Попытка украсть происходит – 4 раза
Крадется половина задач из локальной очереди другого P

```go
                        +-----------+
                        |           |
                        | netpoller |
                        |  Thread   |
                        +-----------+
                               │
                               │  Отработавшие открепленные потоки
                               ▼
        +--------+    +--------+    +--------+    +--------+
        | sysmon |    |   P1   |    |   P2   |    |   P3   |
        | Thread |    |--------|    |--------|    |--------|
        +--------+    | G1, G2 |    | G3, G4 |    | G5, G6 |
                      +--------+    +--------+    +--------+
                           ▲             ▲             ▲
       Проверяет           │             │             │
   неоткрепленные          └─────────────┴─────────────┘
 заблокированными потоками       (заблокированные горутины, ожидающие syscall)
                            │
                            ▼
                    +----------------+
                    |  Thread pool   |
                    |    MY MZ       |
                    +----------------+
                            │
                            ▼
                        +--------+
                        |  MX    |
                        | Thread |
                        +--------+
                            │
                            ▼
          G (горутина, заблокированная syscall)                
```

### Sheduler Syscalls

- Когда происходит syscall поток M переходит в режим системного вызова. Может быть заблокирован, если системный вызов блокирующий (read/write)
- При syscall поток, в котором исполняется зависшая горутина, открепляется от своей локальной очереди и ее начинает обслуживать другой поток.
- После завершения операции поток складывается в зарезервированный thread pool.

### Handoff

Открепление потока:

- Если syscall выполняется долго, то планировщик открепляет поток.
- Если syscall короткий, то поток не открепляется от процессора.
- Однако если ожидаемо короткий syscall превышает лимит ожидания, то это определяется sysmon и поток открепляется.

### Netpoller(Thread)

Обрабатывает асинхронные сетевые операции ввода / вывода. Использует epoll на Linux и kqueue на macOS.

Когда дескриптор доступен для чтения / записи то приходит соответсвующий event. В одном из потоков работает event loop Затем Netpoller уведомляет планировщика Go. Планировщик перемещает горутину из состояния ожидания обратно в очередь выполнения.

### Очереди

```go
   Локальная очередь
   FIFO                                                  LIFO
   +------------------------------------------------+    +--------+
   | +----+    +----+    +----+    +----+    +----+ |    | +----+ |
<- | | G1 | <- | G2 | <- | G3 | <- | G4 | <- | G5 | | <- | | G6 | | <->
   | +----+    +----+    +----+    +----+    +----+ |    | +----+ |
   +------------------------------------------------+    +--------+
   Глобальная очередь – FIFO
```

Однокомпонентная очередь LIFO нужна для ситуации когда несколько горутин общаются между собой.

Новые горутины добавляются и извлекаются по принципу LIFO. Это позволяет максимизировать локальность кэша, так как вновь созданные горутины, вероятно, еще находятся в кэше процессора.

FIFO для задач которые ждут какого-то события.

Когда две горутины постоянно гоняют друг друга в LIFO считается время работы, в случае превышения лимита – они вытесняются из LIFO.
Синхронизация
Блокировки находятся не на уровне системы, а блокируется горутина

### Состояния горутины

- Running;
- Runnable готова к выполнению
- Waiting

Waiting горутины помещаются в wait queue.

Starvation mode (режим голодания)

![deque](pics/starvation-mode.jpg)

### Циклы

Как только обнаруживается горутина, которая работает более 10мс, в поток передается сигнал для ее вытеснения (SIGURG). Этим занимается sysmon.

## Syscalls

### Общее про системные вызовы
- Вызов функций ядра. 
- Syscalls предоставляют стандартный API для выполнения таких операций, как чтение и запись файлов, создание процессов, управление памятью, работа с сетью и т.д.
- Примеры:
	- **open, read, write, close:** для работы с файлами.
	- **fork, exec, wait:** для управления процессами.
	- **mmap:** для отображения файлов в память.
	- **socket, connect, send, recv, epoll, ewait:** для работы с сетевыми соединениями. (`kqueue` на маке)
### Syscalls в Go

- **syscall** - пакет для системных  вызовов из стандартной библиотеки Go. Считается устаревшим.
- [golang.org/x/sys/unix](https://pkg.go.dev/golang.org/x/sys/unix) - более хорший пакет для работы с системными вызовами, поддерживается сообществом
## Pprof (профилирование) и trace (трассировка)

### **pprof (runtime/pprof)**

- pprof используется для сбора и анализа профилей, таких как CPU, память (heap), goroutine, блокировки

- `go tool pprof http://localhost:6060/debug/pprof/profile?seconds=30` профиль CPU за 30 секунд, который затем можно анализировать

- pprof можно запускать в проде - оверхед небольшой, и запускается в реальном времени по http запросу

### trace
- **Сбор данных:**
	Трассировку можно собрать с помощью команды `go test -trace trace.out` или программно через пакет `runtime/trace`.

- **Анализ:**
	Полученный файл трассировки можно просмотреть с помощью команды `go tool trace trace.out`, которая откроет интерактивный веб-интерфейс. Там можно увидеть временную шкалу событий, взаимодействие горутин, задержки и прочие детали, помогающие понять динамику работы приложения.

- **Когда использовать:**
	Анализ работы приложения в реальном времени, взаимодействие горутин, диагностировать проблемы синхронизации или обнаружить узкие места, связанные с задержками в работе планировщика.

 - Трассировка собирает большой объем данных – запускать постоянно не стоит

---
# Асинхронность

- Кооперативная многозадачность
- В аварийном режиме можно вытеснить горутину

## Горутины

- Легковесные потоки, управляемые планировщиком Go
- Конкурентно исполняемые функции
- Свой динамически расширяемый стек от 2кб (благодаря этому их легче переключать чем потоки, у потоков стек фиксированный)
- Планировщик Go распределяет горутины по доступным логическим процессорам, работает в User Space
- Runtime Go предоставляет примитивы синхронизации между горутинами
- GOMAXPROC
- `runtime.Gosched()` - передает управление планировщику

## Каналы

- gorutine-safe
- Хранение элементов, семантика FIFO
- Передача данных между горутинами
- Блокировка горутин
- Синхронизирует исполнение в разных горутинах
- Thread safe очередь
- Можно определять каналы только для записи или только для чтения

**буферизированные** - горутина приходит и записывает, если буфер переполнен, то записывающая горутина будет ждать читающую

**не буферизированные** - одна горутина будет висеть на записи в канал до тех пор, пока другая не будет читать из канала

### Каналы в памяти

```go
c : = make(chan int, 5)
c <- 42
a:= <- c
close(c)
```

Каналы хрянятся в heap как структура hchan

### Структура канала

<https://go.dev/src/runtime/chan.go>

```go
type hchan struct {
    qcount   uint           // total data in the queue // число элементов в буфере
    dataqsiz uint           // size of the circular queue // размерность буфера
    buf      unsafe.Pointer // points to an array of dataqsiz elements // ссылка на буфер
    elemsize uint16
    closed   uint32         // 0 - open, 1 - closed // флаг закрытия канала
                            //  он сделан как uint32 для атомарности, atomic не может работать с bool

    timer    *timer // timer feeding this chan
    elemtype *_type // element type
    sendx    uint   // send index // номер ячейки буфера для записи
    recvx    uint   // receive index // номер ячейки буфера для чтения
    recvq    waitq  // list of recv waiters
    sendq    waitq  // list of send waiters
  
    // lock protects all fields in hchan, as well as several
    // fields in sudogs blocked on this channel.
    //
    // Do not change another G's status while holding this lock
    // (in particular, do not ready a G), as this can deadlock
    // with stack shrinking.
    lock mutex // мьютекс для потокобезопасности
}

type waitq struct {
    first *sudog
    last  *sudog
}

type sudog struct {
    g          *g        // ВАЖНО Указатель на горутину, которая заблокирована

    isSelect   bool             // Эта горутина участвует в select
    next       *sudog           // Следующая sudog в очереди ожидания
    prev       *sudog           // Предыдущая sudog (если используется двусвязный список)

    elem       unsafe.Pointer   // ВАЖНО Элемент, который используется в канале или других синхронизирующих примитивах

    waitlink   *sudog           // Связь для горутины в списке ожидания
    acquiretime int64     // Время захвата (для мониторинга выполнения)
    releasetime int64     // Время освобождения (для мониторинга)
    parent     *mutex     // Мьютекс, к которому привязана goroutine (для каналов и других механизмов)
    ticket     uint64     // Для специфичных алгоритмов блокировки/ожидания
}
```

Буфер представляет собой кольцевой буфер, который хранит элементы канала.

Данные в буфер **копируются**. Значит когда данные будут прочитаны из канала, на оригинальные данные они не повлияют.

### Buffer overflow

Если буфер переполнен, то горутина будет на паузе пока другая горутина не прочитает данные из канала.

### Очередь спящих горутин

`ch <- data` вызывает функцию `gopark` которая обращается к планировщику, он изменяет состояние горутины из `running` в `waiting`. Далее разрывает связь горутины с ОС потоком. Затем планировщик переключает контекст на другую горутину.

### Чтение из переполненного канала

**Подготовка:**
Горутина попадает в очередь `sendq`. Ждущая горутина попадает в `g`, ее данные в `elem` затем структура `sudog` добавляется в `sendq`.

Когда в канал приходит читатель, он берет первую горутину из очереди, на который указывает `recvx`, далее `смещается`.

Затем спящая горутина пробуждается и может положить данные в освободившуюся ячейку буфера.

Пробуждением спящей горутины занимается RX. Он берет первую горутину из очереди `sendq` и помещает ее данные в буфер. Затем `sendx` смещается.

### Пробуждение спящей горутины

Далее RX пробуждает спящую горутину. `goready()` меняет состояние горутины на `runnable` и добавляет ее в очередь готовых к выполнению `runq`.

### Чтение из пустого канала

RX хочет прочитать из канала, вызывает функцию `gopark()` и попадает в `reciveq`.

Далее TX приносит свои данные и происходит оптимизация. С помощью `sendDirect()` стек горутины TX переносится в стек горутины RX.

### Summary

- gorutine-safe >> hchan mutex
- Хранение элементов, семантика FIFO >> hchan buf
- Передача данных между горутинами >> sendDirect, operations with buf
- Блокировка горутин >> sendq / recvq, sudog, calls to scheduler: gopark(), goready()

### Закрытие канала

1) Перед close(ch), проверка инициализирован канал или нет. Если нет, то попытка закрытия вызывает `panic`.
2) Защелкивание lock mutex
3) Проверка закрыт канал или нет >> если уже закрыт, то panic
4) Ставится `ch.closed = true`
5) Освободить всех читателей
6) Освободить всех писателей – они будут паниковать

 **Зачем закрывать канал?**
- Писатели так сообщают, что закончили запись, тогда читатели знают что данных больше не будет
- Для избежания блокировки читателей канала

---
## Вопросы #Q #Channel 
### Чтение из двух каналов #Q #Channel 
```go
in1 := make(chan int)
in2 := make(chan int)
out := make(chan int)

// solution
select {
case v := <-in1:
    fmt.Println("in1:", v)
case v := <-in2:
    fmt.Println("in2:", v)
}
```

### Блокировка канала #Q #Channel 
```go
ch := make(chan int)
ch <- 1 // блокировка

// fixed
ch := make(chan int, 1)
ch <- 1

// лтбо в горутине
```

### Блокировка ждущей горутины #Q #Channel 

```go
ch := make(chan int)
go func() {
    <-ch // горутина заблокируется ожидая данных из канала
}()
ch <- 1 // ждущая горутина проснется и получит данные
```

### Паника при записи в закрытый канал #Q #Channel 

```go
ch := make(chan int)
close(ch)
ch <- 1 // panic
```

### Что в переменной val (два канала)? #Q #Channel 

```go
ch1 := make(chan chan int)
ch2 := make(chan int)
go func() {
    ch2 <- 1 // послали 1 и канал заблокировался
}()
ch1 <- ch2 // послали из канала ch2 и ch1 заблокировался
val := <-<-ch1
// val = 2
```

### Как узнать закрыт ли канал? #Q #Channel 
```go
ch = make(chan int, 1)

val, ok := <-ch

if !ok {
	 // канал закрыт
}
```
### Сколько элементов в канале? #Q #Channel 
```go
ch := make(chan int, 3)
ch <- 1
ch <- 2

n := len(ch) // но в этот момент может другая горутина читать
```

### Исправить (добавить буфер в канал, закрыть канал и передать контекст) #Q #Channel 
```go
func (s *Service) ProcessData(timeoutCtx context.Context, r io.Reader) error {
  errCh := make(chan error)
  errCh := make(chan error, 1) // FIXED

  go func() {
	defer close(errCh) // FIXED
    errCh <- s.processDataInternal(r)
	errCh <- s.processDataInternal(timeoutCtx, r) // FIXED
  }()


  select {
  case err := <-errCh:
	return err
  case <-timeoutCtx.Done():
    return timeoutCtx.Err()
  }
}

// FIXED add ctx context.Context
func (s *Service) processDataInternal(ctx context.Context, r io.Reader) error {
    // Включите проверки ctx.Done() в долгих операциях, чтобы прервать выполнение при тайм-ауте.
    return nil
}
```
## sync.Mutex, sync.RWMutex

API

```go
type Mutex struct {...}

func (m *Mutex) Lock()          // захватить мьютекс
func (m *Mutex) Unlock()        // освободить мьютекс
func (m *Mutex) TryLock() bool  // попытка захвата мьютекса
```

```go
var mu sync.Mutex
mu.Lock()
// критическая секция
mu.Unlock()
```

### sync.Once

## sync.WaitGroup

Используется для ожидания завершения горутин.

```go
var wg sync.WaitGroup

func worker(id int) {
    defer wg.Done()
    fmt.Printf("Worker %d starting\n", id)
    // Выполнение работы...
    fmt.Printf("Worker %d done\n", id)
}

func main() {
    for i := 1; i <= 5; i++ {
        wg.Add(1)
        go worker(i)
    }
    wg.Wait()
    fmt.Println("All workers completed")
}
```

### Исправить(`wg.Add(1)` внутри горутины) #Q #WaitGroup 
```go
func printText(data []string) {
    wg := sync.WaitGroup{}
    for _, v := range data {
		wg.Add(1) // ADD TO FIX
        go func(v string ) {
			defer wg.Done()  // ADD TO FIX
            wg.Add(1) // REMOVE
            fmt.Println(v)
            wg.Done() // REMOVE
        }() // REMOVE
	    }(v) // pass copy to the function
    }
	wg.Wait() // ADD TO FIX
    fmt.Println("done!")
}

data := []string{"one", "two", "three"}
printText(data)
```
### Data race #Q #Mutex #WaitGroup
```go
var callCounter uint
var sync.Mutex mu // ADD TO FIX

func main() {
	var wg sync.WaitGroup // ADD TO FIX
    
	for i := 0; i < 10000; i++ {
		wg.Add(1) // ADD TO FIX
        go func() {
		    defer wg.Done() // ADD TO FIX
            time.Sleep(time.Second) // Делаем долгую работу
            mu.Lock() // ADD TO FIX
            callCounter++
            mu.Unlock() // ADD TO FIX
        }()
    }

	wg.Wait()
    fmt.Println("Call counter value = ", callCounter)
}
```
### ErrorGroup (не в стандартном пакете sync, но расширяет WaitGroup)

- Не находится в стандартном пакете sync, находится во внешнем пакете golang.org/x/sync/errgroup.
- Позволяет собирать ошибки из горутин и отменять оставшиеся горутины при ошибке.
- Удобен для параллельного выполнения задач, где важна обработка ошибок.

```go
import (
    "fmt"
    "net/http"

    "golang.org/x/sync/errgroup"
)

func main() {
    var g errgroup.Group
    urls := []string{
        "https://www.google.com",
        "https://www.example.com",
        "https://nonexistent.url", // Эта ссылка вызовет ошибку
    }

    for _, url := range urls {
        url := url // Захват переменной
        g.Go(func() error {
            resp, err := http.Get(url)
            if err != nil {
                return fmt.Errorf("ошибка запроса %s: %v", url, err)
            }
            defer resp.Body.Close()
            fmt.Printf("Получен ответ от %s, статус: %s\n", url, resp.Status)
            return nil
        })
    }

    if err := g.Wait(); err != nil {
        fmt.Printf("Произошла ошибка: %v\n", err)
    } else {
        fmt.Println("Все запросы успешно выполнены")
    }
}
```

## sync.Pool

### Worker Pool 

Pool из пакета sync предоставляет пул объектов для повторного использования, что помогает снизить нагрузку на сборщик мусора и повысить производительность в приложениях, где часто создаются и уничтожаются объекты.

- `New func() interface{}`: Функция, которая создает новый объект, если пул пуст.
- `Get() interface{}`: Получает объект из пула или создает новый, если пул пуст.
- `Put(x interface{})`: Возвращает объект в пул для повторного использования.

```go
package main

import (
    "bytes"
    "fmt"
    "sync"
)

var bufferPool = sync.Pool{
    New: func() interface{} {
        fmt.Println("Создание нового буфера")
        return new(bytes.Buffer)
    },
}

func process(data string) {
    buf := bufferPool.Get().(*bytes.Buffer)
    defer bufferPool.Put(buf) // отложенный вызов - буфер возвращается в пул

    buf.Reset() // объект полностью очищен
    buf.WriteString(data) // запись данных в буфер

    // Используем буфер
    fmt.Println(buf.String())
}

func main() {
    process("Первый вызов") // пул пуст, поэтому вызывается New, создается новый bytes.Buffer
    process("Второй вызов")
}

// Вывод:
// Создание нового буфера
// Первый вызов
// Второй вызов
```

### Объекты в `sync.Pool`

- Буферы и временные данные
  - `bytes.Buffer`
  - `strings.Builder`
  - Временные слайсы и мапы
- Объекты сериализации и десериализации
  - JSON/XML/YAML энкодеры и декодеры: Например, `json.Encoder`, `json.Decoder`.
  - Кастомные структуры для парсинга данных.
- Сетевые объекты
  - Структура для работы с HTTP-запросами и ответами.
  - Буфер для чтения и записи сетевых данных.
- Пользовательские типы данных

## sync.Map

- Потокобезопасная реализация map
- Предназначен для конкурентного доступа из нескольких горутин без необходимости явной синхронизации
- Оптимизирован для большого количества операций чтения и небольшого количества операций записи
- Более сложные механизмы синхронизации для повышения производительности

**Почему создан `sync.Map`?**

`sync.Mutex` или `sync.RWMutex` для обычной Map могут быть неэффективными при большом количестве операций чтения и записи

### Методы

- Store(key, value interface{}): Сохраняет значение по заданному ключу.
- Load(key interface{}) (value interface{}, ok bool): Загружает значение по ключу. Возвращает значение и признак успешности.
- LoadOrStore(key, value interface{}) (actual interface{}, loaded bool): Если ключ уже существует, возвращает существующее значение и loaded = true. Иначе сохраняет новое значение и возвращает loaded = false.
- Delete(key interface{}): Удаляет ключ и его значение из Map.
- Range(f func(key, value interface{}) bool): Итерирует по всем парам ключ-значение в Map. Функция f вызывается для каждой пары. Если f возвращает false, итерация прекращается.

### Особенности `sync.Map`

- Типы ключей и значений – interface{}, что позволяет использовать любые типы данных с методами Equal и Hash.
- При интенсивных операциях записи может быть эффективнее использовать обычный map с RWMutex

### Внутреннее устройство `sync.Map`

- Компоненты
  - `read` - неизменяемая хеш-таблица(read-only), хранит наиболее актуальные данные и обеспечивает быстрый доступ к ним без блокировки
  - `dirty` - изменяемая хеш-таблица, хранит обновления, не обработанные `read`
  - `mutex` - мьютекс для синхронизации доступа к `dirty`

- Misses
  - Счётчик, который увеличивается при неудачной попытке чтения из read. Когда количество промахов превышает определённый порог (обычно 2), read обновляется из dirty.
- Amended
- Флаг, который указывает, что dirty содержит обновления, которые не были применены к read.

### Алгоритм работы `sync.Map`

3) Load (Чтение)
   - Попытка чтения из read без блокировки
   - Если ключ найден, возвращается значение
   - Если ключ не найден, увеличивается счётчик misses, при превышении порога
   - read обновляется из dirty:
     - Захватывается мьютекс (sync.Mutex).
     - read обновляется, объединяя данные из dirty.
     - Флаг amended устанавливается в false, так как read теперь актуален.
     - После обновления read из dirty функция повторно пытается прочитать ключ из обновлённого read, но уже под блокировкой.
   - Проверка dirty с блокировкой
   - Если ключ найден, возвращается значение
   - Если ключ не найден, возвращается nil и false

4) Store (Запись)
5) LoadOrStore
6) Delete
7) Range

## Пакет atomic

API

```go
type Bool struct {...}

func (x *Bool) CompareAndSwap(old, new bool) (swapped bool)
func (x *Bool) Load() bool          // атомарно загрузить значение в память
func (x *Bool) Store(val bool)
func (x *Bool) Swap(new bool) (old bool)
```

- AddInt64
- atomic.Int64
- atomic.Value

---
# Архитектура кода

- Time to market
- Расширяемость и тестируемость

## Структура пакета

– Не должно быть циклической зависимости(пакеты не используют зависимости своих подпакетов)
– Каждый домен инкапсулировал в свой пакет

## Стандарт(рекомендация) архитектуры проекта на Go

<https://github.com/golang-standards/project-layout>

```go
|-api           # REST, POST-like, GRPC
|-assets
|-build
|---ci
|---package
|-cmd           # main файлы приложений, конфиги, internal приложения
|---_your_app_
|-----internal  # внутренние пакеты отдельного приложения
|-------pack_a
|-------pack_b
|-configs       # дефолтные конфиги для локального разворачивания сервиса
|-deployments   # конфиги для локального деплоя и разработки (docker-compose и другие CI файлы)
|-docs          # документация
|-examples
|-githooks
|-init
|-internal      # внутренние пакеты утилит и сервисов
|---app
|-----_your_app_
|---pkg
|-----_your_private_lib_
|-pkg           # пакеты которые могут импортироваться другими проектами
|---_your_public_lib_
|-scripts
|-test          # интеграционные тесты, E2E тесты, снапшот тесты и др используемые в проектах, кроме Unit тестов
|-third_party
|-tools
|-vendor
|-web
|---app
|---static
|---template
|-website
|go.mod         # зависимости
|LICENSE.md
|Makefile       # инструкции сборки
|README.md
```

## Конфигурация проекта

Пример хорошей конфигурации

```go
package postgresql

const defaultPort    = 5432

// Config содержит настройки подключения к PostgreSQL.
type Config struct {
    Host     string `json:"host" yaml:"host"`
    Port     int    `json:"port" yaml:"port"`
    User     string `json:"user" yaml:"user"`
    Password string `json:"password" yaml:"password"`
    SSLMode  string `json:"ssl_mode" yaml:"ssl_mode"`
}

// withDefaults применяет значения по умолчанию для пустых полей конфигурации.
func (c *Config) withDefaults() Config {
    conf := *c

    if conf.Port == 0 {
        conf.Port = defaultPort
    }
    if conf.SSLMode == "" {
        conf.SSLMode = defaultSSLMode
    }

    return
}
```

## Dependency injection(DI)

DI - паттерн проектирования, который позволяет управлять зависимостями между объектами в приложении. Пакеты не создают свои зависимости самостоятельно, а получают их из вне.

DI container управляет зависимостями между частями программы

## ООП

Утиная типизация – тип объекта определяется наличием методов и свойств

## Интерфейсы

- Интерфейс в Go — это набор методов, которые тип должен реализовать. В интерфейсе не указывается, какие типы его реализуют. Это обеспечивает неявную реализацию интерфейсов.
- Любой тип, который реализует все методы интерфейса, автоматически удовлетворяет этому интерфейсу. В Go нет необходимости явно заявлять, что тип реализует интерфейс.
- Чтобы метод был частью типа, он должен принимать указатель на этот тип (или значение) в качестве первого аргумента. Это связывает метод с типом и делает его доступным для использования в интерфейсе.

```go
// Определяем структуру CSVProcessor
type CSVProcessor struct{}

// Определяем интерфейс FileProcessor с методом Process
type FileProcessor interface {
    Process(data []byte) error
}

// Метод Process для типа CSVProcessor
func (p *CSVProcessor) Process(data []byte) error {
    fmt.Println("Processing CSV data:", string(data))
    return nil
}
// (p *CSVProcessor) передается как аргумент в метод интерфейса FileProcessor чтобы тип CSVProcessor удовлетворял интерфейсу, причем указатель на CSVProcessor

func main() {
    // Создаем экземпляр CSVProcessor и присваиваем его переменной типа FileProcessor.
    var processor FileProcessor = &CSVProcessor{}

    // Пример данных, которые нужно обработать (CSV данные в виде среза байт).
    data := []byte("id,name,age\n1,Alice,30\n2,Bob,25")

    // Вызываем метод Process через интерфейс FileProcessor.
    err := processor.Process(data)
    if err != nil {
        fmt.Println("Error processing CSV:", err)
    }
}
```

Таким образом CSVProcessor это реализация интерфейса FileProcessor

Еще пример определения интерфейсов

```go
type transport struct{}

type Connector interface {
    Connect(ctx context.Context) error
}

type Sender interface {
    Send(ctx context.Context, data []byte) error
}

// интерфейс Transport объединяет интерфейсы Connector и Sender
type Transport interface {
    Connector
    Sender
}

var _ Transport = &transport{}
// проверка что структура transport действительно реализует интерфейс Transport
// (полезная техника для проверки соответствия контракту интерфейса)

func NewTransport() Transport{
    return &trancport{}
}

func newTransport() *transport{
    return *transport{}
}

// реализация методов интерфейсов
func (t *transport) Connect(ctx context.Context) error {
    fmt.Println("Соединение установлено")
    return nil
}

func (t *transport) Send(ctx context.Context, data []byte) error {
    fmt.Printf("Отправка данных: %s\n", string(data))
    return nil
}


```

## Пустой интерфейс #Q #Interface

```go
interface{}
```

- Не содержит методов => любой тип удовлетворяет интерфейсу;
- Полиморфизм: позволяет работать с объектами любого типа;
- Используется для обобщенных функций.

```go
func printValue(val interface{}) {
    // Определяем конкретный тип с помощью type switch
    switch v := val.(type) {
    case int:
        fmt.Printf("Это целое число: %d\n", v)
    case string:
        fmt.Printf("Это строка: %s\n", v)
    default:
        fmt.Printf("Неизвестный тип: %T\n", v)
    }
}

func main() {
    printValue(42)
    printValue("hello")
    printValue(3.14)
}
```

## Конфликты

```go
type jsonWriter struct{}

func (w *jsonWriter) Write(data []byte) error {}

type xmlWriter struct{}

func (w *xmlWriter) w(data []byte) error {}

type writer struct {
    jsonWriter
    xmlWriter
}

func write() {
    w := writer{}
    w.Write()   // Ошибка компиляции. Конфликт имен.
                // Структура writer содержит поля jsonWriter и xmlWriter. В Go нет перегрузки методов. Не определено метод какого поля берется.
}
```

## Паттерны и идиомы

Паттерн – описания повторяющихся решений типичных проблем при разработке ПО
Идиомы – общепринятые приемы и конструкции которые используются для решения задач в конкретном языке программирования

## Репозиторий

Это популярный подход к организации доступа к данным в приложении. Позволяет абстрагироваться от деталей хранения данных и предоставляет единый интерфейс для работы с данными.

```go
// repository.go
type User struct {
    ID   int
    Name string
    Emai string
    Password    string
}

type UserRepository interface {
    GetByID(id int) (*User, error)
    GetAll() ([]*User, error)
    Create(u *User) error
    Update(u *User) error
    Delete(id int) error
}
```

---
# Errors

По соглашению, ошибки имеют тип error, который является простым встроенным интерфейсом.

```go
type error interface {
    Error() string
}
```

Этот интерфейс можно реализовать для создания пользовательских ошибок.

Пример кастомной ошибки:

```go
// PathError записывает ошибку, а также операцию и путь файла, которые её вызвали.
type PathError struct {
    Op   string // "open", "unlink", и т.д.
    Path string // Связанный файл.
    Err  error  // Возвращено системным вызовом.
}

func (e *PathError) Error() string {
    return e.Op + " " + e.Path + ": " + e.Err.Error()
}
```

**Что происходит с исполнением когда происходит `error`?**

- Когда происходит ошибка, функция возвращает ненулевое значение типа `error`
- Исполнение программы продолжается
- При ошибках поток выполнения не прерывается автоматически
- Обработка ошибок должна быть реализована разработчиком

## Panic

Если ошибка неустранима, программа может вызвать встроенную функцию `panic`. `panic` создаёт ошибку времени выполнения.

**Что происходит с исполнением когда происходит `panic`?**

8) Вызов `panic`
    - Немедленное прекращение выполнения текущей функции
    - Операции после вызова `panic` не выполняются в этой функции не выполняются

9) Запуск раскрутки стека (stack unwinding)
   - Переход к функции вызвавшей текущую функцию и т.д.
   - На каждом уровне стека вызываются `defer` функции

10) Выполнение `defer` функций
   - `defer` функции выполняются в обратном порядке
   - `defer` функции выполняются даже если произошел `panic`, позволяет очистить ресурсы, сделать логирование и др. завершающие операции

11) Возможность восстановления с помощью `recover`
   - `recover`
     - Если `recover` вызывается **внутри** `defer` и в данной функции произошел `panic`, то:
       - `recover` возвращает значение переданное в `panic(value)`
       - Раскрутка стека останавливается и выполнение продолжается после отложенной функции, в которой был вызван `recover`
     - Если `recover` вызван **вне** `defer` функции или вне `panic`, то он возвращает `nil` и не влияет на исполнение программы
   - Продолжение выполнения
     - После успешного восстановления с помощью `recover`, выполнение продолжается с места вызова `panic`, но все операции после `panic` в данной функции не выполняются
     - Обычно требуется завершить текущую функцию и добавить обработку для продолжения выполнения программы

12) Продолжение раскрутки стека при отсутствии `recover`
   - Если `recover` не вызывается или не сработал, то раскрутка стека продолжается
     - Отложенные функции продолжают выполняться на каждом уровне
   - Если паника достигает верхнего уровня стека без восстановления, то горутина аварийно завершается

13) Аварийное завершение программы
   - Если главная горутина завершается аварийно и не была восстановлена, то завершается вся программа
   - Другие горутины продолжают работу, если главная горутина не завершилась и если они не зависят от завершившейся горутины

## Recover

Когда вызывается функция panic, выполнение текущей функции останавливается, и начинается разматывание стека: все функции, вызванные до этой точки, прекращают свою работу, и выполняются отложенные функции (defer).

Если стек разматывается до конца и никакая отложенная функция не вызвала recover, программа завершает свою работу с ошибкой.

Если внутри одной из отложенных функций вызывается recover, это прекращает разматывание стека, и программа может продолжить работу. Recover возвращает аргумент, переданный в panic. Если паники не было, recover вернет nil.

**Важно**
- `recover` полезен только внутри отложенных функций. Если вы вызовете его вне отложенной функции, он всегда будет возвращать `nil`

- **использование** `panic` и `recover` должно быть ограничено локальными задачами (например, внутри пакетов). Не следует распространять панику наружу, передавая её клиентскому коду, так как это усложняет отладку и работу с ошибками

## Вопросы Panic, Recover #Q #Panic #Recover
### Печать после panic #Q #Panic #Recover 
```go
func main() {
    defer func() {
        recover()
    }()
    panic("test panic") // размотка стека / stack unwinding
						// вызывается defer
						// recover() перехватывает панику и возвращает ее значение
    fmt.Println("ok") // напечатан НЕ БУДЕТ
}
```

---
# Unsafe

## Указатели

- Указатели - это число.
- Указатель это смещение начала значения относительно начала памяти.
- Указатель не имее типа.
- Значения по указателю обрабатываются согласно инструкциии процессора, а не типа данных, которые лежат по указателю.
- Адресная арифметика - указатели можно сравнивать, складывать, вычитать из указателей смещения.
- Любое значение в памяти, соотетствующее нативному типу CPU, может использоваться как указатель.

В Go строгая типизация, отсюда могут быть оганичения.

## Неоптимальный код - Слайсы и массивы
- Каждое обращение к элементу слайса/массива/строке дополняется кодом проверки выхода за границы слайса/массива/строки
```go
if index < 0 || index >= len(slice) {
    panic("index out of range")
}
```

- Такие проверки может исключить оптимизатор в ходе Bounds Check Elimination (BCE) и Dead Code Elimination (DCE), но не всегда
- В "горячих" циклах может давать просадку производительности

## Избыточное копирование - Иммутабельность и копирование

Go строки - UTF8, они иммутабельны
- Создание строки из слайса - копирование
- Соединение слайса из строк - копирование

## Работа с указателями cgo - C-функции и передача указателей
- Некоторые C-функции принимают указатели
- Иногда в качестве конца данных используется указатель за ними, например на следующий байт после объекта
```go
// void write(void *start, void *stop) {} // end <- первый байт после объекта
import "C"
import "unsafe"

func write(a []byte) {
    start := unsafe.Pointer(&a[0])
    stop := unsafe.Pointer(&a[len(a)]) // Boom! Проверка на выход за границы слайса
    C.write(start, stop)
}
```

Нет адресной арифметики, сравнить, вычесть Pointer - нельзя без unsafe
```go
// void parse(void **start, void **stop) {} // end <- первый байт после объекта
import "C"
import "unsafe"

func parse(a []byte) {
    var start unsafe.Pointer
    var stop unsafe.Pointer
    C.parse(&start, &stop)
    fmt.Printf("Parsed %d bytes\n", stop - start) // <- не компилируется
}
```

## Копирование и cgo - Go стек и C-функции

Некоторые C-функции работают через указатели
```go
// #include <stdint.h>
// void f1(uint32_t *p) {}
import "C"
import "unsafe"

func escape(a []C.uint) {
    p1 := &a[0]
    C.f1(p1)
}

func main() {
    buf := []C.uint{1, 2, 3, 4, 5}
    escape(buf)
}
```

Unsafe.Pointer позволяет получить C-совместимый указатель
```go
// #include <stdint.h>
// void f1(uint32_t *p) {}
// void f2(void *p) {}
import "C"
import "unsafe"

func escape(a []C.uint) {
    p1 := &a[0]
    C.f1(p1) // тут будет копирование в кучу
    
    p2 := unsafe.Pointer(p1)
    C.f2(p2) // это тоже должно оказаться в куче
}

func main() {
    buf := []C.uint{1, 2, 3, 4, 5}
    escape(buf)
}
```

- Объекты на стеке при передаче в C - переносятся в кучу
- <https://github.com/golang/go/issues/24450>

## Отображение бинарных данных в Go - Работа с бинарными данными

- Бинарные данные обычно представляются как `[]byte`

```go
func read_data(conn net.Conn) {
    buf := make([]byte, 0, 4096)
    for {
        n, err := conn.Read(buf)
        if err != nil {
            // обработка ошибки
            break
        }
        // buf[:n] <- тут слайс прочитанных байт
    }
}
```

- Для разбора данных используются пакет encoding/binary

```go
func read_data(conn net.Conn) {
    buf := make([]byte, 4096)  // создаем буфер с длиной и ёмкостью 4096 байт
    for {
        n, err := conn.Read(buf)
        if err != nil {...}
        // Создаём bytes.Reader для работы с прочитанными байтами
        r := bytes.NewReader(buf[:n])
        // Чтение данных в структуру
        err = binary.Read(r, binary.LittleEndian, &some_struct)
    }
}
```

- Пакет encoding/binary использует reflectin, это ограничивает скорость разбора

## Пакет unsafe

### func Sizeof(x ArbitraryType) uintptr

- Возвращает количество байт в памяти для размещения значения типа аргумента

```go
func main() {
    fmt.Printf("Sizeof(uint32(1234)) %d\n", unsafe.Sizeof(uint32(1234))) // Sizeof(uint32(1234)) 4
    
    var arr [200]int
    fmt.Printf("Sizeof([200]int) %d\n", unsafe.Sizeof(arr)) // Sizeof([200]int) 1600
}
```

- Если переданное значение содержит указатели на другие объекты или данные, то их размер не включается

```go
func main() {
    fmt.Printf("Sizeof(\"test\") %d\n", unsafe.Sizeof("test"))  // Sizeof("test") 16
    fmt.Printf("Sizeof(\"test-test-test-test-test\") %d\n", unsafe.Sizeof("test-test-test-test-test"))  // Sizeof ("test-test-test-test-test") 16
    fmt.Printf("Sizeof([]int) %d\n", unsafe.Sizeof(make([]int, 200)))   // Sizeof([]int) 24
}
```

- Включает заполнения между элементами структуры

```go
type s1 struct {
    u8 uint8 // 1 байт
             // 7 байт заполнения
    u64 uint64 // 8 байт
}

func main() {
    fmt.Printf("Sizeof(s1) %d\n", unsafe.Sizeof(s1{})) // Sizeof(s1) 16
}
```

### func Alignof(x ArbitraryType) uintptr

- Возвращает выравнивание стартового адреса переменной если бы она имела тип x.
- Аналогично reflect.TypeOf(x).Align()

```go
package main

import (
    "fmt"
    "unsafe"
)

type s1 struct {
    u8  uint8
    u64 uint64
}

func main() {
    fmt.Printf("Alignof(uint8) %d\n", unsafe.Alignof(uint8(12)))
	// Alignof(uint8)      1
    fmt.Printf("Alignof(uint32) %d\n", unsafe.Alignof(uint32(1234)))
	// Alignof(uint32)     4
    fmt.Printf("Alignof(\"test\") %d\n", unsafe.Alignof("test"))        
	// Alignof("test")     8
    fmt.Printf("Alignof(struct) %d\n", unsafe.Alignof(s1{}))            
	// Alignof(struct)     8

    var arr [200]int
    fmt.Printf("Alignof([200]int) %d\n", unsafe.Alignof(arr))           
	// Alignof([200]int)   8
}
```

- Выравнивание для типа внутри структуры может отличаться для компилятора gccgo т.к. используется сишный ABI (Application Binary Interface)

```go
type s1 struct {
    u8  uint8
    u64 uint64
}

func main() {
    fmt.Printf("Alignof(s1.u8) %d\n", unsafe.Alignof(s1{}.u8))      // Alignof(s1.u8) 1
    fmt.Printf("Alignof(s1.u64) %d\n", unsafe.Alignof(s1{}.u64))    // Alignof(s1.u64) 8
}
```

### func Offsetof(x ArbitraryType) uintptr

- Показывает смещение адреcа относительно начала структуры

```go
type s1 struct {
    u8  uint8
    u64 uint64
}

func main() {
    fmt.Printf("Offsetof(s1.u8) %d\n", unsafe.Offsetof(s1{}.u8)) // Offsetof(s1.u8) 0
    fmt.Printf("Offsetof(s1.u64) %d\n", unsafe.Offsetof(s1{}.u64)) // Offsetof(s1.u64) 8
}
```

- Стандарт языка Go не ограничивает компилятор в том как располагать поля в структуре

### func SliceData(slice []ArbitraryType) *ArbitraryType

- Возвращает указатель на данные содержащиеся в слайсе
- Указатель показывает на начало данных, не определяет сколько фактических данных содержится в слайсе
- До Go 1.20 использовалась структура SliceHeader

```go
func main() {
    byte_str := []byte{73, 32, 97, 109, 32, 97, 32, 115, 116, 114, 105, 110, 103}
    fmt.Printf("Slice data %v\n", unsafe.SliceData(byte_str))   // Slice data 0x00001a160

    sh := (*reflect.SliceHeader)(unsafe.Pointer(&byte_str))
    fmt. Printf("Slice data %vin", unsafe.Pointer(sh.Data))     // Slice data 0x00001a160
}
```

### func StringData(str string) *byte

- Возвращает указатель на данные содержащиеся в строке
- До Go 1.20 использовалась структура StringHeader
- Данные под строкой немутабельны! (В Go строка хранится в области памяти, которая помечена как только для чтения)

```go
func main () {
    str := "Test string" // строковый литерал, Go может поместить его в область памяти только для чтения
    fmt.Printf ("String data %v\n", unsafe.StringData(str))    // String data 0x49c613

    stringh := (*reflect.StringHeader) (unsafe.Pointer(&str))
    fmt.Printf("String data%v\n", unsafe.Pointer(stringh.Data)) // String data 0x49c613

    fmt. Printf("String data %v\n", unsafe.Pointer(stringh.Data))
    *unsafe.StringData(str) = 70    // BOOM! 
                                    // panic
                                    // unexpected fault address 0x49c61
                                    // fatal error: fault
}
```

### func String(ptr *byte, len IntegerType) string

- Создает строку на участке памяти byte длиной len
- Без копирования
- Строки в Go - иммутабельны, пока строка существует - нельзя менять содержимое [ptr, ptr+len]

```go
func main () {
    byte_str:= []byte{73, 32, 97, 109, 32, 97, 32, 115, 116, 114, 105, 110, 103}
    z str := unsafe.String(unsafe.SliceData(byte_str), len(byte_str))
    fmt.Printf ("Zero copy string: %s\n", z_str) // Zero copy string: I am a string
    fmt.Printf("Compare pointers: %v == %v\n",
        unsafe.StringData(z_str), unsafe.SliceData(byte_str))   // Compare pointers: 0x00001a160 == 0x00001a160
}
```

### func Slice(ptr *ArbitraryType, len IntegerType) []ArbitraryType

- Cоздает слайс на участке памяти byte длиной len с типом переданного значения
- Без копирования
- Если данные использованные для создания слайса принадлежат строке - то в слайс нельзя писать

```go
func main () {
    str := "Test string" // Строковый литерал, Go может поместить его в область памяти только для чтения
    z_slice:= unsafe.Slice(unsafe.StringData(str), len(str))
    fmt. Printf ("Zero copy slice: %v\n", z_slice)  // Zero copy slice: [84 101 115 116 32 115 116 114 105 110 103]
    fmt.Printf ("Compare pointers: %v = %v\n",
        unsafe.SliceData(z_slice), unsafe.StringData(str))  // Compare pointers: 0x49c613 = 0x49c613
    
    z_slice[0] = 4  // unexpected fault address 0x49c613
                    // fatal error: fault
}
```

### type Pointer *ArbitraryType

- Нетипизированный указатель
- Типизированные указатели приводятся к Pointer
- Может быть приведен к типизированному указателю
- Отслеживается GC и рантаймом Go
  GC понимает когда данные слайса используются через Pointer и не удаляет их

```go
func main() {
    var ptr unsafe.Pointer
    byte_slice := []byte{01, 02, 03, 04}
    byte_ptr := unsafe.SliceData(byte_slice) // указатель на первый байт слайса
    ptr = unsafe.Pointer(byte_ptr)  // приведение указателя к НЕтипизированному указателю
    int_ptr := (*uint32)(ptr)  // приведение указателя к типизированному указателю

    // тут на процессоре Little endian
    fmt.Printf("Original slice %v\n", byte_slice)   // Original slice [1 2 3 4]
    fmt.Printf("Combined int: %d, hex: %x\n", *int_ptr, *int_ptr)   // Combined int: 67305985, hex: 4050607

    *int_ptr = 0x05060708   // изменение значения по указателю
    fmt.Printf("Touched slice %v\n", byte_slice)    // Touched slice [8 7 6 5]
}
```

- Может быть приведен к типу uintptr
- Uintptr позволяет использовать адресную арифметику
- Тип uintptr приводится к Pointer
- Тип uintptr не отслеживается GC и рантаймом Go!
- Можно получить провисший указатель (dangling pointer)

```go
func main() {
    byte_slice := []byte{01, 02, 03, 04}
    byte_ptr := unsafe.SliceData(byte_slice)
    raw_ptr := unsafe.Pointer(byte_ptr)
    uptr_0 := uintptr(raw_ptr)
    uptr_3 := uptr_0 + 3 // <- Указатель на последний элемент среза
    fmt.Printf("First ptr %d Fourth ptr %d Delta %d Less %v\n",
        uptr_0, uptr_3, uptr_3 - uptr_0, uptr_0 < uptr_3)
    // First ptr 824634510660 Fourth ptr 824634510663 Delta 3 Less true

    fmt. Printf("Fourth val %d", *(*byte)unsafe.Pointer(uptr_3)) // Fourth val 4

    bad_ptr:= unsafe. Pointer (uptr_3 + 1) // <- dangling pointer
    bad_byte := (*byte)bat_ptr
    *bad_byte = 5 // Порча чужой памяти
}
```

## Опасность unsafe

### Модель памяти и uintptr

- Доступ не валидируется
  
  Например при создании двух слайсов нет гарантий какого-то расположения в памяти, единственное – они не разделяют одну память

- Можно легко попасть на другой объект
- В другой сегмент памяти, например read-only(но туда не срашно, тк сразу пролучим ошибку)
- В собственный стек
- В сегмент кода (можно менять код в рантайме, пропатчить например)

```go
func main() {
    slice1 := []byte{0, 1, 2}
    p1 := uintptr(unsafe.Pointer(unsafe.SliceData(slice1)))
    slice2 := []byte{3, 4, 5}
    p2 := uintptr(unsafe.Pointer(unsafe.SliceData(slice2)))
    
    fmt.Printf("%x %x\n", p1, p2) // c00001a138 c00001a13b

    bad_ptr := (*byte)(unsafe.Pointer(p1 + 5)) // вышли за границу
    *bad_ptr = 10  // записали во второй слайс

    dangling_ptr := (*byte)(unsafe.Pointer(p2 + 5)) // вышли за границу
    *dangling_ptr = 100 // записали неизвестно куда
    
    fmt.Printf("First slice %v Second slice %v\n", slice1, slice2) // First slice [0 1 2] Second slice [3 4 10]
}
```

- uintptr это просто число, и GC не отслеживает объекты по соответствующему адресу
- значение uintptr не меняется при перемещении объекта

```go
func get_ptr(fill byte) uintptr {
    dyn_slice := []byte{fill, fill, fill, fill}
    return uintptr(unsafe.Pointer(unsafe.SliceData(dyn_slice)))
}

/*
dyn_slice - временный объект, который удаляется после завершения функции
при выходе из функции GC может освободить эту память
второй вызов перезаписывает это место памяти
*/

func main() {
    ptrs := make([]uintptr, 0)
    for i := 0; i < 2; i++ {
        ptr := get_ptr(byte(i))
        ptrs = append(ptrs, ptr)
    }

    firstSlice := unsafe.Pointer(ptrs[0])
    fmt.Printf("uintptr %v, first value %d != 0\n", ptrs, *(*byte)(firstSlice)) 
    // uintptr [824634511168 824634511168], first value 1 = 0
    // два объекта на одном адресе
}
```

- приведение Pointer к uintptr и обратно должно быть выполнено в одно выражение
- Между приведениями к uintptr и обратно допускаются только арифметические операции

```go
func f1(buf []byte) *byte {
    start := unsafe.Pointer(&buf[0])

    val_ptr := uintptr(start) + buf[0]
    // Тут слайс может быть удален GC или перемещён
    val := unsafe.Pointer(val_ptr)

    val = unsafe.Pointer(uintptr(start) + buf[0]) // <- Так правильно, компилятор понимает что трогать объекты не нужно

    return (*byte)(val)
}
```

- При вызове функций проведение к uintptr должно происходить непосредственно в аргументах вызова

```go
func main() {
    // ...
    syscall.Syscall(syscall.SYS_READ, uintptr(fd), uintptr(unsafe.Pointer(&p[0])), uintptr(n))
    // ...
}
```

## Польза unsafe

### Строки без копирования

- Можно создавать строку на существующих данных
- Обрабатывать данные строки как набор байт

```go
func main() {
    byte_str := []byte{73, 32, 97, 109, 32, 97, 32, 115, 116, 114, 105, 110, 103}
    z_str := unsafe.String(unsafe.Pointer(&byte_str[0]), len(byte_str))
    // ...
}
```

### Доступ к элементам слайса или строки

- В "горячих циклах" проверка границ может быть замедляющим фактором
- Можно убрать проверки с помощью адресной арифметики

```go
func get_item(slice []uint32, idx int) uint32 {
    start := uintptr(unsafe.Pointer(&slice[0])) // <- Первый байт
    step := unsafe.Sizeof(slice[0])             // <- Размер каждого элемента
    ptr := unsafe.Pointer(start + uintptr(idx)*step) // <- Указатель на элемент
    return *(*uint32)(ptr)
}
```

### Взаимодействие с cgo
#TODO

- Передаем uintptr вместо указателя

```go
//#include <stdint.h>
//void read(void *ptr) {};
//void wrap_read(uintptr_t ptr) { read((void *)ptr); }
import "C"
import "unsafe"

func f2(a []byte) {
    buf := make([]byte, 1024)
    start := unsafe.Pointer(&buf[0])
    C.wrap_read(C.uintptr_t(uintptr(start))) // <- преобразование в вызове
}
```

## Примеры использования unsafe

### Чтение структур

- Пусть есть структура, описывающая метрику

```go
type TimeVal struct {
    TStamp uint32
    Value  uint32
}

type Metric struct {
    Source uint32
    Tag    uint32
    Value  TimeVal
}
```

- Пусть есть сокет, откуда можно читать поток метрик

```go
func Read(conn net.Conn, data []byte) (int, error) {
    n, err := conn.Read(data)
    if err != nil {
        n = -1
    }
    return n, err
}
```

- Требуется получить слайс метрик для передачи на обработку
- Можно использовать `encoding.Binary`

```go
func Parse(data []byte) ([]Metric, error) {
    result := make([]Metric, len(data) / binary.Size(Metric{}))
    if err := binary.Read(bytes.NewReader(data), binary.LittleEndian, &result); err != nil {
        return nil, err
    }
    return result, nil
}
```

- Можно использовать `unsafe`

```go
func ParseUnsafe(data []byte) ([]Metric, error) {
    count := len(data) / int(unsafe.Sizeof(Metric{})) // <- считаем сколько элементов можно прочитать
    startPtr := unsafe.Pointer(&data[0]) // <- Нетипизированный указатель
    startMetric := (*Metric)(startPtr)   // <- Указатель нужного типа
    return unsafe.Slice(startMetric, count), nil // <- Создаем слайс
}
```

- Пока что полученные значения еще нужны – исходные данные нельзя менять!!!

- Бенчмарк для 256 Мб данных:

    > **Unsafe**: Converting 268435456 bytes takes 100ns
    >
    > **Safe**: Converting 268435456 bytes takes 1.49132454s

- Фактически вариант unsafe не копирует данные и даже не читает их, а просто приводит один тип к другому

- Пусть исходные данные имеют `BigEndian` кодирование
- Вариант(безопасный) с encoding/binary:

```go
func Parse(data []byte) ([]Metric, error) {
    result := make([]Metric, len(data) / binary.Size(Metric{})) // <- Создаем слайс
    if err := binary.Read(bytes.NewReader(data), binary.BigEndian, &result); err != nil {
        return nil, err
    }
    return result, nil
}
```

- Вариант(unsafe) добавим перекодирование в `BigEndian`:

```go
func Uint32Swap(value uint32) uint32 {
    valuePtr := unsafe.Pointer(&value)
    slice := unsafe.Slice((*byte)(valuePtr), unsafe.Sizeof(value))
    return binary.BigEndian.Uint32(slice) // binary.BigEndian хорошо оптимизируется компилятором
}
```

- Сконвертируем поля структуры, при этом конвертация работает inplace

```go
func ParseUnsafe(data []byte) ([]Metric, error) {
    count := len(data) / int(unsafe.Sizeof(Metric{}))
    startPtr := unsafe.Pointer(&data[0]) // <- Нетипизированный указатель
    startMetric := (*Metric)(startPtr)   // <- Указатель нужного типа
    result := unsafe.Slice(startMetric, count)
    for idx := range result {
        result[idx] = Metric{
            Source: Uint32Swap(result[idx].Source),
            Tag:    Uint32Swap(result[idx].Tag),
            Value:  TimeVal{
                TStamp: Uint32Swap(result[idx].Value.TStamp),
                Value:  Uint32Swap(result[idx].Value.Value),
            },
        }
    }
    return result, nil
}
```

- Бенчмарк для 256 Мб данных:

    > **Unsafe**: Converting 268435456 bytes takes 36.485552ms
    >
    > **Safe**: Converting 268435456 bytes takes 1.507235589s

- Разница в 40 раз
- Еще немного магии с указателями

```go
func ParseUnsafe(data []byte) ([]Metric, error) {
    count := len(data) / int(unsafe.Sizeof(Metric{}))
    startPtr := unsafe.Pointer(&data[0])
    curPtr := startPtr
    for idx := 0; idx < count; idx++ {
        metricPtr := (*Metric)(curPtr)
        *metricPtr = Metric{
            Source: Uint32Swap(&metricPtr.Source),
            Tag:    Uint32Swap(&metricPtr.Tag),
            Value:  TimeVal{
                TStamp: Uint32Swap(&metricPtr.Value.TStamp),
                Value:  Uint32Swap(&metricPtr.Value.Value),
            },
        }
        curPtr = unsafe.Pointer(uintptr(unsafe.Pointer(metricPtr)) + unsafe.Sizeof(Metric{}))
    }
    startMetric := (*Metric)(startPtr)
    result := unsafe.Slice(startMetric, count)
    return result, nil
}
```

- Бенчмарк для 256 Мб данных:
    > **Pointer iteration**: Converting 268435456 bytes takes 31.084443ms
- Сократили время еще на 15%
- ОДНАКО МЕНЯТЬ СТОРАДЖ НЕЛЬЗЯ!!!

### Чтение структур и memory layout

- Пример выше работает только если расположение в памяти совпадает с разметкой структуры
- Надо проверять Sizeof и Offsetof
- Структуры в Go имеют выравнивание полей и могут иметь пропуски между полями
- Можно использовать cgo и объявить тип там

```go
//#include <stdint.h>
// struct pack {
//    uint8_t type;
//    uint32_t value;
// } __attribute__((packed));

import "C"
import "unsafe"
import "fmt"

type s1 struct {
    u8  uint8
    u32 uint32
}

func main() {
    fmt.Printf("Sizeof(s1) %d\n", unsafe.Sizeof(s1{}))  // Sizeof(s1) 8
    fmt.Printf("Sizeof(C.struct_pack) %d\n", unsafe.Sizeof(C.struct_pack{})) // Sizeof(C.struct_pack) 5
}
```

### Чтение NULL-терминированных массивов с строк

- Есть С функция, которая возвращает массив строк

```go
// char **read(){...}
import "C"
```

- Массив строк неизвестной длины
- Последний элемент массива - NULL
- Строки NULL-терминированны
- Пройтись по строкам
- Прочитать каждую строку

- `unsafe.Pointer` могут быть указателем на что угодно, в том числе `unsafe.Pointer`
- Получаем указатель на указатели
- Строки в C и C++ это указатель на первый символ строки
- Используем адресную арифметику, чтобы перейти на следующий указатель
- Также используем unsafe.Slice
- Длина строки неизвестна, посчитаем ее используя адресную арифметику

```go
func main() {
    CStrings := C.read()
    Ptr := (*unsafe.Pointer)(unsafe.Pointer(CStrings))

    for *Ptr != nil {
        CharPtr := *Ptr // <- указатель на символ
        for *(*byte)(CharPtr) != 0 {
            CharPtr = unsafe.Pointer(uintptr(CharPtr) + 1)
        }
        // разница указателей покажет нам длину строки
        count := uintptr(CharPtr) - uintptr(*Ptr)

        Str := unsafe.String((*byte)(*Ptr), count)
        sum = sum + tsum(Str)
        PtrPtr := unsafe.Pointer(Ptr) // <- приводим к нетипизированному
        PtrPtr = unsafe.Pointer(uintptr(PtrPtr) + unsafe.Sizeof(Ptr))
        Ptr = (*unsafe.Pointer)(PtrPtr)
    }
}
```

- Распарсим 1е6 коротких строк - превосходство более чем в 2 раза
- Бенчмарк:
    > **Unsafe**: processing takes 427.370386ms
    >
    > **C.GoString**: Processing takes 1.058987s
## Задачи
### Reuse of variable / Переиспользование переменных #Q 

```go
package main

import "fmt"

func main() {
    nums := []int{1, 2, 3}
    var pointers []*int

    // ONLY BEFORE VERSION 1.22
    // v created once and then just reused
    for _, v := range nums {

        pointers = append(pointers, &v)
        // [ADDR, ADDR, ADDR] - all pointers point to the same memory location
    }

    // ADDR holds the value of the last iteration – 3

    for _, p := range pointers {
        fmt.Println(*p) // This prints 3 three times.
    }

    // ONLY BEFORE VERSION 1.22
    // [3, 3, 3]
}
```

---
#TODO C GO расшарить

- КАК РАБОТАЕТ КОД ИЗ БИНАРНИКА
- ПОЧЕМУ В ГО БИНАРНИК МНОГО ВЕСИТ 
- МНОГОПОТОЧКА ПОСТГРЕС
- РЭБИТ ДЛЯ ГАРАНТИЙ ДОСТАВКИ ТОЛЬКО ОДИН РАЗ
- ЕСТЬ ОЧЕРЕДЬ ЗАДАЧ (СЕТЕВЫЕ И ВЫЧИСЛИТЕЛЬНЫЕ)
- ЧТО ПРОИСХОДИТ ПРИ ОТСТРЕЛЕ ГОРУТИНЫ
```

## **Marshaling** / Сериализация

### encoding/

- json
- xml
- gob
- ...

```go
package main

import (
	"encoding/json"
	"fmt"
)

type Person struct {
	Name string `json:"name"`
	Age  int    `json:"age"`
}

func main() {
	p := Person{Name: "Alice", Age: 30}

	// Преобразование структуры в JSON
	jsonBytes, err := json.Marshal(p)
	if err != nil {
		fmt.Println("Error marshaling:", err)
		return
	}
	fmt.Println("JSON:", string(jsonBytes))
}

func main() {
	data := `{"name": "Alice", "age": 30}`
	var p Person

	// Преобразование JSON в структуру
	err := json.Unmarshal([]byte(data), &p)
	if err != nil {
		fmt.Println("Error unmarshaling:", err)
		return
	}
	fmt.Printf("Person: %+v\n", p)
}
```
---