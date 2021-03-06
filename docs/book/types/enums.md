# Перечисления

Перечисления - это способ организовать коллекцию связанных значений. Многие другие языки программирования (C/C#/Java) имеют тип данных `enum`, а JavaScript - нет. Тем не менее, TypeScript имеет. Вот пример описания перечисления в TypeScript:

```ts
enum CardSuit {
    Clubs,
    Diamonds,
    Hearts,
    Spades,
}

// Пример использования
var card = CardSuit.Clubs;

// Предохранитель
card = 'not a member of card suit';
// Ошибка : строка не может быть назначена для типа `CardSuit`
```

Эти значения перечислений являются `числами`, поэтому далее я буду называть их числовыми перечислениями.

## Числовые перечисления и числа

TypeScript перечисления основанные на числах. Это означает, что числа могут быть назначены на экземпляр перечисления, а также всё, что совместимо с `числом`.

```ts
enum Color {
    Red,
    Green,
    Blue,
}
var col = Color.Red;
col = 0; // Фактически то же что и Color.Red
```

## Числовые перечисления и строки

Прежде чем мы углубимся в перечисления, давайте рассмотрим генерируемый ими JavaScript, вот пример TypeScript:

```ts
enum Tristate {
    False,
    True,
    Unknown,
}
```

генерирует следующий JavaScript:

```js
var Tristate;
(function (Tristate) {
    Tristate[(Tristate['False'] = 0)] = 'False';
    Tristate[(Tristate['True'] = 1)] = 'True';
    Tristate[(Tristate['Unknown'] = 2)] = 'Unknown';
})(Tristate || (Tristate = {}));
```

давайте сосредоточимся на строке `Tristate[Tristate["False"] = 0] = "False";`. Внутри неё выражение `Tristate["False"] = 0` не требует особых объяснений, по факту оно присваивает свойству `"False"` объекта `Tristate` значение `0`.

Обратите внимание, что в JavaScript оператор присваивания возвращает присвоенное значение (в данном случае `0`). Поэтому следующая операция, выполняемая средой выполнения JavaScript, это `Tristate[0] = "False"`. Это означает, что вы можете использовать переменную `Tristate` для преобразования строковой версии перечисления в число или числовой версии перечисления в строку. Это продемонстрировано ниже:

```ts
enum Tristate {
    False,
    True,
    Unknown,
}
console.log(Tristate[0]); // "False"
console.log(Tristate['False']); // 0
console.log(Tristate[Tristate.False]);
// "False" потому что `Tristate.False == 0`
```

## Изменение числа, в числовом перечислении

По умолчанию перечисления начинаются с `0`, а затем каждое последующее значение автоматически увеличивается на 1. В качестве примера рассмотрим следующее:

```ts
enum Color {
    Red, // 0
    Green, // 1
    Blue, // 2
}
```

Однако вы можете изменить число, связанное с любым элементом перечисления, назначив его конкретным. Это показано ниже, где мы начинаем с 3 и начинаем увеличивать:

```ts
enum Color {
    DarkRed = 3, // 3
    DarkGreen, // 4
    DarkBlue, // 5
}
```

> СОВЕТ: Я обычно инициализирую первое перечисление с `= 1`, чтобы можно было наверняка проверить значение перечисления на достоверность.

## Числовые перечисления в качестве флагов

Одним из отличных вариантов использования перечислений является возможность использовать перечисления в качестве `Флагов`. Флаги позволяют вам проверить, верно ли определенное условие из набора условий. Рассмотрим следующий пример, где у нас есть набор свойств о животных:

```ts
enum AnimalFlags {
    None = 0,
    HasClaws = 1 << 0,
    CanFly = 1 << 1,
    EatsFish = 1 << 2,
    Endangered = 1 << 3,
}
```

Здесь мы используем оператор сдвига влево для перемещения `1` вокруг определенного уровня битов, чтобы получить побитовые непересекающиеся числа `0001`, `0010`, `0100` и `1000` (это десятичные числа `1`,`2`,`4`,`8` если вам интересно). Побитовые операторы `|` (или) / `&` (и) / `~` (не) являются вашими лучшими друзьями при работе с флагами и продемонстрированы ниже:

```ts
enum AnimalFlags {
    None = 0,
    HasClaws = 1 << 0,
    CanFly = 1 << 1,
}
type Animal = {
    flags: AnimalFlags;
};

function printAnimalAbilities(animal: Animal) {
    var animalFlags = animal.flags;
    if (animalFlags & AnimalFlags.HasClaws) {
        console.log('животное имеет когти');
    }
    if (animalFlags & AnimalFlags.CanFly) {
        console.log('животное может летать');
    }
    if (animalFlags == AnimalFlags.None) {
        console.log('ничего');
    }
}

let animal: Animal = { flags: AnimalFlags.None };
printAnimalAbilities(animal); // ничего
animal.flags |= AnimalFlags.HasClaws;
printAnimalAbilities(animal); // животное имеет когти
animal.flags &= ~AnimalFlags.HasClaws;
printAnimalAbilities(animal); // ничего
animal.flags |= AnimalFlags.HasClaws | AnimalFlags.CanFly;
printAnimalAbilities(animal); // животное имеет когти, животное может летать
```

Здесь:

-   мы использовали `|=`, чтобы добавить флаги
-   комбинация `&=` и `~` для очистки флага
-   `|` объединить флаги

> Примечание: вы можете комбинировать флаги для создания удобных сокращений в определении перечисления, например `EndangeredFlyingClawedFishEating` ниже:

```ts
enum AnimalFlags {
    None = 0,
    HasClaws = 1 << 0,
    CanFly = 1 << 1,
    EatsFish = 1 << 2,
    Endangered = 1 << 3,

    EndangeredFlyingClawedFishEating = HasClaws |
        CanFly |
        EatsFish |
        Endangered,
}
```

## Строковые перечисления

Мы смотрели только на перечисления, где значениями являются `числа`. На самом деле вы также можете использовать перечисления со строковыми значениями. Например

```ts
export enum EvidenceTypeEnum {
    UNKNOWN = '',
    PASSPORT_VISA = 'passport_visa',
    PASSPORT = 'passport',
    SIGHTED_STUDENT_CARD = 'sighted_tertiary_edu_id',
    SIGHTED_KEYPASS_CARD = 'sighted_keypass_card',
    SIGHTED_PROOF_OF_AGE_CARD = 'sighted_proof_of_age_card',
}
```

С ними проще работать и проверять, так как они предоставляют выразительные / проверяемые строковые значения.

Вы можете использовать эти значения для простого сравнения строк. Например

```ts
// Где `someStringFromBackend` будет
// '' | 'passport_visa' | 'passport' ... и т. д.
const value = someStringFromBackend as EvidenceTypeEnum;

// Пример использования в коде
if (value === EvidenceTypeEnum.PASSPORT) {
    console.log('Вы предоставили паспорт');
    console.log(value); // `passport`
}
```

## Константные перечисления

Если у вас есть определение перечисления, подобное следующему:

```ts
enum Tristate {
    False,
    True,
    Unknown,
}

var lie = Tristate.False;
```

Строка `var lie = Tristate.False` компилируется в JavaScript как `var lie = Tristate.False` (да, вывод совпадает с вводом). Это означает, что во время выполнения потребуется поиск `Tristate`, а затем `Tristate.False`. Чтобы повысить производительность, вы можете записать `enum` как `const enum`. Это продемонстрировано ниже:

```ts
const enum Tristate {
    False,
    True,
    Unknown,
}

var lie = Tristate.False;
```

генерирует следующий JavaScript:

```js
var lie = 0;
```

то есть компилятор:

1. _Встраивает_ любое использование перечисления (`0` вместо `Tristate.False`).
2. Не генерирует JavaScript для определения перечисления (во время выполнения отсутствует переменная `Tristate`), поскольку его использование встроено.

## Константные перечисления с флагом preserveConstEnums

Встраивание имеет очевидные преимущества в производительности. Тот факт, что во время выполнения отсутствует переменная `Tristate`, просто означает что компилятор помогает вам не генерировать JavaScript, который фактически не используется во время выполнения.

Однако вы, возможно, захотите, чтобы компилятор по-прежнему генерировал JavaScript-версию определения перечисления для перевода _числа в строку_ или _строки в число_, как мы рассматривали выше. В этом случае вы можете использовать флаг компилятора `--preserveConstEnums`, и он все равно сгенерирует определение `var Tristate`, чтобы вы могли использовать `Tristate["False"]` или `Tristate[0]` вручную во время выполнения, если это нужно. Это никак не влияет на _встраивание_.

## Перечисление со статическими функциями

Вы можете использовать объявление `перечисления` + `область имен`, чтобы добавить статические методы в перечисление. Следующий пример демонстрирует как мы добавляем статический элемент `isBusinessDay` в перечисление `Weekday`:

```ts
enum Weekday {
    Monday,
    Tuesday,
    Wednesday,
    Thursday,
    Friday,
    Saturday,
    Sunday,
}
namespace Weekday {
    export function isBusinessDay(day: Weekday) {
        switch (day) {
            case Weekday.Saturday:
            case Weekday.Sunday:
                return false;
            default:
                return true;
        }
    }
}

const mon = Weekday.Monday;
const sun = Weekday.Sunday;
console.log(Weekday.isBusinessDay(mon)); // true
console.log(Weekday.isBusinessDay(sun)); // false
```

## Перечисления расширяемы

> ПРИМЕЧАНИЕ: расширяемые перечисления имеют значение только в том случае, если вы не используете модули. Вы должны использовать модули. Следовательно, этот раздел является последним.

Вот сгенерированный JavaScript для перечисления из известного выше примера:

```js
var Tristate;
(function (Tristate) {
    Tristate[(Tristate['False'] = 0)] = 'False';
    Tristate[(Tristate['True'] = 1)] = 'True';
    Tristate[(Tristate['Unknown'] = 2)] = 'Unknown';
})(Tristate || (Tristate = {}));
```

Мы уже объяснили часть `Tristate[Tristate["False"] = 0] = "False";`. Теперь обратите внимание на оборачивающий код `(function (Tristate) { /*code here */ })(Tristate || (Tristate = {}));` особенно часть `(Tristate || (Tristate = {}));`. Он в основном описывает локальную переменную `Tristate`, которая будет либо указывать на уже определенное значение `Tristate`, либо инициализировать новый пустой объект `{}`.

Это означает, что вы можете разделить (и расширить) определение перечисления на несколько файлов. Например, ниже мы разделили определение для `Color` на два блока

```ts
enum Color {
    Red,
    Green,
    Blue,
}

enum Color {
    DarkRed = 3,
    DarkGreen,
    DarkBlue,
}
```

Обратите внимание, что вы _должны_ повторно инициализировать первый элемент (здесь `DarkRed = 3`) в продолжении перечисления, чтобы получить производный код, а не значения перезаписанные из предыдущего определения (то есть `0`, `1`, ... и так далее). TypeScript предупредит вас, если вы этого не сделаете (сообщение об ошибке `В перечислении с несколькими объявлениями только одно объявление может не использовать инициализатор для своего первого элемента`).
