---
description: Для проверки данных необходимо сначала определить схему. Схемы представляют типы, от простых примитивных значений до сложных вложенных объектов и массивов
---

# Определение схем

Для проверки данных необходимо сначала определить _схему_. Схемы представляют _типы_, от простых примитивных значений до сложных вложенных объектов и массивов.

## Примитивы

```ts
import * as z from 'zod';

// primitive types
z.string();
z.number();
z.bigint();
z.boolean();
z.symbol();
z.undefined();
z.null();
```

### Принудительное преобразование

Чтобы принудительно преобразовать входные данные в соответствующий тип, используйте вместо этого `z.coerce`:

```ts
z.coerce.string(); // String(input)
z.coerce.number(); // Number(input)
z.coerce.boolean(); // Boolean(input)
z.coerce.bigint(); // BigInt(input)
```

Принудительный вариант этих схем пытается преобразовать входное значение в соответствующий тип.

```ts
const schema = z.coerce.string();

schema.parse('tuna'); // => "tuna"
schema.parse(42); // => "42"
schema.parse(true); // => "true"
schema.parse(null); // => "null"
```

!!!note "Как работает принуждение в Zod"

    Zod принудительно обрабатывает все входные данные с помощью встроенных конструкторов.

    | Zod API              | Приведение        |
    | -------------------- | ----------------- |
    | `z.coerce.string()`  | `String(value)`   |
    | `z.coerce.number()`  | `Number(value)`   |
    | `z.coerce.boolean()` | `Boolean(value)`  |
    | `z.coerce.bigint()`  | `BigInt(value)`   |
    | `z.coerce.date()`    | `new Date(value)` |

    Булевое принудительное преобразование с помощью `z.coerce.boolean()` может работать не так, как вы ожидаете. Любое [истинное](https://developer.mozilla.org/en-US/docs/Glossary/Truthy) значение принудительно преобразуется в `true`, а любое [ложное](https://developer.mozilla.org/en-US/docs/Glossary/Falsy) значение принудительно преобразуется в `false`.

    ```ts
    const schema = z.coerce.boolean(); // Boolean(input)

    schema.parse('tuna'); // => true
    schema.parse('true'); // => true
    schema.parse('false'); // => true
    schema.parse(1); // => true
    schema.parse([]); // => true

    schema.parse(0); // => false
    schema.parse(''); // => false
    schema.parse(undefined); // => false
    schema.parse(null); // => false
    ```

    Для полного контроля над логикой принуждения рассмотрите возможность использования [`z.transform()`](#transforms) или [`z.pipe()`](#pipes).

## Литералы

Литеральные схемы представляют собой [литеральный тип](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#literal-types), например `"hello world"` или `"5"`.

```ts
const tuna = z.literal('tuna');
const twelve = z.literal(12);
const twobig = z.literal(2n);
const tru = z.literal(true);
```

Для представления литералов JavaScript `null` и `undefined`:

```ts
z.null();
z.undefined();
z.void(); // equivalent to z.undefined()
```

Чтобы разрешить несколько литеральных значений:

```ts
const colors = z.literal(['red', 'green', 'blue']);

colors.parse('green'); // ✅
colors.parse('yellow'); // ❌
```

Чтобы извлечь набор допустимых значений из литеральной схемы:

```ts
colors.values; // => Set<"red" | "green" | "blue">
```

## Строки

Zod предоставляет несколько встроенных API для проверки и преобразования строк. Для выполнения некоторых распространенных проверок строк:

```ts
z.string().max(5);
z.string().min(5);
z.string().length(5);
z.string().regex(/^[a-z]+$/);
z.string().startsWith('aaa');
z.string().endsWith('zzz');
z.string().includes('---');
z.string().uppercase();
z.string().lowercase();
```

Чтобы выполнить несколько простых преобразований строк:

```ts
z.string().trim(); // trim whitespace
z.string().toLowerCase(); // toLowerCase
z.string().toUpperCase(); // toUpperCase
```

## Форматы строк

Для проверки на соответствие некоторым распространенным форматам строк:

```ts
z.email();
z.uuid();
z.url();
z.emoji(); // проверяет один символ эмодзи
z.base64();
z.base64url();
z.nanoid();
z.cuid();
z.cuid2();
z.ulid();
z.ipv4();
z.ipv6();
z.cidrv4(); // ipv4 CIDR block
z.cidrv6(); // ipv6 CIDR block
z.iso.date();
z.iso.time();
z.iso.datetime();
z.iso.duration();
```

### Электронные адреса

Для проверки адресов электронной почты:

```ts
z.email();
```

По умолчанию Zod использует сравнительно строгое регулярное выражение для проверки нормальных адресов электронной почты, содержащих общие символы. Оно примерно соответствует правилам, применяемым Gmail. Чтобы узнать больше об этом регулярном выражении, обратитесь к [этой публикации](https://colinhacks.com/essays/reasonable-email-regex).

```ts
/^(?!\.)(?!.*\.\.)([a-z0-9_'+\-\.]*)[a-z0-9_+-]@([a-z0-9][a-z0-9\-]*\.)+[a-z]{2,}$/i;
```

Чтобы настроить поведение проверки электронной почты, вы можете передать пользовательское регулярное выражение в параметр `pattern`.

```ts
z.email({ pattern: /your regex here/ });
```

Zod экспортирует несколько полезных регулярных выражений, которые вы можете использовать.

```ts
// Zod's default email regex
z.email();
z.email({ pattern: z.regexes.email }); // equivalent

// the regex used by browsers to validate input[type=email] fields
// https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/email
z.email({ pattern: z.regexes.html5Email });

// the classic emailregex.com regex (RFC 5322)
z.email({ pattern: z.regexes.rfc5322Email });

// a loose regex that allows Unicode (good for intl emails)
z.email({ pattern: z.regexes.unicodeEmail });
```

### UUIDs

Для проверки UUID:

```ts
z.uuid();
```

Чтобы указать конкретную версию UUID:

```ts
// supports "v1", "v2", "v3", "v4", "v5", "v6", "v7", "v8"
z.uuid({ version: 'v4' });

// for convenience
z.uuidv4();
z.uuidv6();
z.uuidv7();
```

Спецификация UUID RFC 9562/4122 требует, чтобы первые два бита байта 8 были равны `10`. Другие идентификаторы, подобные UUID, не налагают этого ограничения. Для проверки любого идентификатора, подобного UUID:

```ts
z.guid();
```

### URLs

Для проверки любого URL-адреса, совместимого с WHATWG:

```ts
const schema = z.url();

schema.parse('https://example.com'); // ✅
schema.parse('http://localhost'); // ✅
schema.parse('mailto:noreply@zod.dev'); // ✅
schema.parse('sup'); // ✅
```

Как видите, это довольно допустимо. Внутренне для проверки входных данных используется конструктор `new URL()`; это поведение может отличаться на разных платформах и в разных средах выполнения, но это наиболее строгий способ проверки URI/URL в любой среде выполнения/движке JS.

Чтобы проверить имя хоста по определенному регулярному выражению:

```ts
const schema = z.url({ hostname: /^example\.com$/ });

schema.parse('https://example.com'); // ✅
schema.parse('https://zombo.com'); // ❌
```

Чтобы проверить протокол на соответствие определенному регулярному выражению, используйте параметр `protocol`.

```ts
const schema = z.url({ protocol: /^https$/ });

schema.parse('https://example.com'); // ✅
schema.parse('http://example.com'); // ❌
```

!!!note ""

    **Веб-URL** — во многих случаях вам потребуется проверить именно веб-URL. Вот рекомендуемая схема для этого:

    ```ts
    const httpUrl = z.url({
    	protocol: /^https?$/,
    	hostname: z.regexes.domain,
    });
    ```

    Это ограничивает протокол `http`/`https` и гарантирует, что имя хоста является действительным доменным именем с помощью регулярного выражения `z.regexes.domain`:

    ```ts
    /^([a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?\.)+[a-zA-Z]{2,}$/;
    ```

### Даты и время ISO

Как вы, возможно, заметили, строка Zod включает в себя несколько проверок, связанных с датой и временем. Эти проверки основаны на регулярных выражениях, поэтому они не так строги, как полная библиотека даты и времени. Однако они очень удобны для проверки ввода пользователя.

Метод `z.iso.datetime()` обеспечивает соблюдение стандарта ISO 8601; по умолчанию смещения часовых поясов не допускаются:

```ts
const datetime = z.iso.datetime();

datetime.parse('2020-01-01T06:15:00Z'); // ✅
datetime.parse('2020-01-01T06:15:00.123Z'); // ✅
datetime.parse('2020-01-01T06:15:00.123456Z'); // ✅ (arbitrary precision)
datetime.parse('2020-01-01T06:15:00+02:00'); // ❌ (offsets not allowed)
datetime.parse('2020-01-01T06:15:00'); // ❌ (local not allowed)
```

Чтобы использовать смещение часовых поясов:

```ts
const datetime = z.iso.datetime({ offset: true });

// allows timezone offsets
datetime.parse('2020-01-01T06:15:00+02:00'); // ✅

// basic offsets not allowed
datetime.parse('2020-01-01T06:15:00+02'); // ❌
datetime.parse('2020-01-01T06:15:00+0200'); // ❌

// Z is still supported
datetime.parse('2020-01-01T06:15:00Z'); // ✅
```

Чтобы позволить использование неквалифицированных (без указания часового пояса) дат и времени:

```ts
const schema = z.iso.datetime({ local: true });
schema.parse('2020-01-01T06:15:01'); // ✅
schema.parse('2020-01-01T06:15'); // ✅ seconds optional
```

Ограничить допустимое время `precision`. По умолчанию секунды не являются обязательными, и допускается произвольная точность до долей секунды.

```ts
const a = z.iso.datetime();
a.parse('2020-01-01T06:15Z'); // ✅
a.parse('2020-01-01T06:15:00Z'); // ✅
a.parse('2020-01-01T06:15:00.123Z'); // ✅

const b = z.iso.datetime({ precision: -1 }); // minute precision (no seconds)
b.parse('2020-01-01T06:15Z'); // ✅
b.parse('2020-01-01T06:15:00Z'); // ❌
b.parse('2020-01-01T06:15:00.123Z'); // ❌

const c = z.iso.datetime({ precision: 0 }); // second precision only
c.parse('2020-01-01T06:15Z'); // ❌
c.parse('2020-01-01T06:15:00Z'); // ✅
c.parse('2020-01-01T06:15:00.123Z'); // ❌

const d = z.iso.datetime({ precision: 3 }); // millisecond precision only
d.parse('2020-01-01T06:15Z'); // ❌
d.parse('2020-01-01T06:15:00Z'); // ❌
d.parse('2020-01-01T06:15:00.123Z'); // ✅
```

### Даты ISO

Метод `z.iso.date()` проверяет строки в формате `YYYY-MM-DD`.

```ts
const date = z.iso.date();

date.parse('2020-01-01'); // ✅
date.parse('2020-1-1'); // ❌
date.parse('2020-01-32'); // ❌
```

### Время ISO

Метод `z.iso.time()` проверяет строки в формате `HH:MM[:SS[.s+]]`. По умолчанию секунды являются необязательными, как и десятичные доли секунды.

```ts
const time = z.iso.time();

time.parse('03:15'); // ✅
time.parse('03:15:00'); // ✅
time.parse('03:15:00.9999999'); // ✅ (arbitrary precision)
```

Никакие смещения не допускаются.

```ts
time.parse('03:15:00Z'); // ❌ (no `Z` allowed)
time.parse('03:15:00+02:00'); // ❌ (no offsets allowed)
```

Используйте параметр `precision` для ограничения допустимой десятичной точности.

```ts
z.iso.time({ precision: -1 }); // HH:MM (minute precision)
z.iso.time({ precision: 0 }); // HH:MM:SS (second precision)
z.iso.time({ precision: 1 }); // HH:MM:SS.s (decisecond precision)
z.iso.time({ precision: 2 }); // HH:MM:SS.ss (centisecond precision)
z.iso.time({ precision: 3 }); // HH:MM:SS.sss (millisecond precision)
```

### IP адреса

```ts
const ipv4 = z.ipv4();
ipv4.parse('192.168.0.0'); // ✅

const ipv6 = z.ipv6();
ipv6.parse('2001:db8:85a3::8a2e:370:7334'); // ✅
```

### IP блоки (CIDR)

Проверьте диапазоны IP-адресов, указанные с помощью [нотации CIDR](https://ru.wikipedia.org/wiki/Бесклассовая_адресация).

```ts
const cidrv4 = z.string().cidrv4();
cidrv4.parse('192.168.0.0/24'); // ✅

const cidrv6 = z.string().cidrv6();
cidrv6.parse('2001:db8::/32'); // ✅
```

## Шаблонные литералы

**Новое в Zod 4**

Чтобы определить схему шаблонного литерала:

```ts
const schema = z.templateLiteral([
    'hello, ',
    z.string(),
    '!',
]);
// `hello, ${string}!`
```

API `z.templateLiteral` может обрабатывать любое количество строковых литералов (например, `"hello"`) и схем. Можно передавать любую схему с выведенным типом, который можно присвоить `string | number | bigint | boolean | null | undefined`.

```ts
z.templateLiteral(['hi there']);
// `hi there`

z.templateLiteral(['email: ', z.string()]);
// `email: ${string}`

z.templateLiteral(['high', z.literal(5)]);
// `high5`

z.templateLiteral([z.nullable(z.literal('grassy'))]);
// `grassy` | `null`

z.templateLiteral([
    z.number(),
    z.enum(['px', 'em', 'rem']),
]);
// `${number}px` | `${number}em` | `${number}rem`
```

## Числа

Используйте `z.number()` для проверки чисел. Допускаются любые конечные числа.

```ts
const schema = z.number();

schema.parse(3.14); // ✅
schema.parse(NaN); // ❌
schema.parse(Infinity); // ❌
```

Zod реализует несколько проверок, связанных с числами:

```ts
z.number().gt(5);
z.number().gte(5); // alias .min(5)
z.number().lt(5);
z.number().lte(5); // alias .max(5)
z.number().positive();
z.number().nonnegative();
z.number().negative();
z.number().nonpositive();
z.number().multipleOf(5); // alias .step(5)
```

Если (по какой-то причине) вы хотите проверить `NaN`, используйте `z.nan()`.

```ts
z.nan().parse(NaN); // ✅
z.nan().parse('anything else'); // ❌
```

## Целые числа

Для проверки целых чисел:

```ts
z.int(); // restricts to safe integer range
z.int32(); // restrict to int32 range
```

## BigInt

Для проверки BigInt:

```ts
z.bigint();
```

Zod включает в себя несколько проверок, специфичных для BigInt.

```ts
z.bigint().gt(5n);
z.bigint().gte(5n); // alias `.min(5n)`
z.bigint().lt(5n);
z.bigint().lte(5n); // alias `.max(5n)`
z.bigint().positive();
z.bigint().nonnegative();
z.bigint().negative();
z.bigint().nonpositive();
z.bigint().multipleOf(5n); // alias `.step(5n)`
```

## Логические значения

Для проверки логических значений:

```ts
z.boolean().parse(true); // => true
z.boolean().parse(false); // => false
```

## Даты

Используйте `z.date()` для проверки экземпляров `Date`.

```ts
z.date().safeParse(new Date()); // success: true
z.date().safeParse('2022-01-12T06:15:00.000Z'); // success: false
```

Чтобы настроить сообщение об ошибке:

```ts
z.date({
    error: (issue) =>
        issue.input === undefined
            ? 'Required'
            : 'Invalid date',
});
```

Zod предоставляет несколько проверок, связанных с конкретными датами.

```ts
z.date().min(new Date('1900-01-01'), { error: 'Too old!' });
z.date().max(new Date(), { error: 'Too young!' });
```

## Enums {#zod-enums}

Используйте `z.enum` для проверки вводимых данных на соответствие фиксированному набору допустимых _строковых_ значений.

```ts
const FishEnum = z.enum(['Salmon', 'Tuna', 'Trout']);

FishEnum.parse('Salmon'); // => "Salmon"
FishEnum.parse('Swordfish'); // => ❌
```

!!!note "Внимание"

    Если вы объявите массив строк как переменную, Zod не сможет правильно определить точные значения каждого элемента.

    ```ts
    const fish = ['Salmon', 'Tuna', 'Trout'];

    const FishEnum = z.enum(fish);
    type FishEnum = z.infer<typeof FishEnum>; // string
    ```

    Чтобы исправить это, всегда передавайте массив непосредственно в функцию `z.enum()` или используйте [`as const`](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-4.html#const-assertions).

    ```ts
    const fish = ['Salmon', 'Tuna', 'Trout'] as const;

    const FishEnum = z.enum(fish);
    type FishEnum = z.infer<typeof FishEnum>; // "Salmon" | "Tuna" | "Trout"
    ```

Вы также можете передать внешне объявленный TypeScript enum.

!!!note "Zod 4"

    Заменяет API `z.nativeEnum()` в Zod 3.

    Обратите внимание, что использование ключевого слова `enum` в TypeScript [не рекомендуется](https://www.totaltypescript.com/why-i-dont-like-typescript-enums).

```ts
enum Fish {
    Salmon = 'Salmon',
    Tuna = 'Tuna',
    Trout = 'Trout',
}

const FishEnum = z.enum(Fish);
```

### `.enum`

Чтобы извлечь значения схемы в виде объекта, похожего на перечисление:

```ts
const FishEnum = z.enum(['Salmon', 'Tuna', 'Trout']);

FishEnum.enum;
// => { Salmon: "Salmon", Tuna: "Tuna", Trout: "Trout" }
```

### `.exclude()`

Чтобы создать новую схему перечисления, исключив определенные значения:

```ts
const FishEnum = z.enum(['Salmon', 'Tuna', 'Trout']);
const TunaOnly = FishEnum.exclude(['Salmon', 'Trout']);
```

### `.extract()`

Чтобы создать новую схему перечисления, извлекая определенные значения:

```ts
const FishEnum = z.enum(['Salmon', 'Tuna', 'Trout']);
const SalmonAndTroutOnly = FishEnum.extract([
    'Salmon',
    'Trout',
]);
```

## Stringbools {#stringbool}

**💎 Новое в Zod 4**

В некоторых случаях (например, при разборе переменных окружения) полезно преобразовать определенные строковые «булевы» значения в простые значения `boolean`. Для этого в Zod 4 введена функция `z.stringbool()`:

```ts
const strbool = z.stringbool();

strbool.parse('true'); // => true
strbool.parse('1'); // => true
strbool.parse('yes'); // => true
strbool.parse('on'); // => true
strbool.parse('y'); // => true
strbool.parse('enabled'); // => true

strbool.parse('false'); // => false
strbool.parse('0'); // => false
strbool.parse('no'); // => false
strbool.parse('off'); // => false
strbool.parse('n'); // => false
strbool.parse('disabled'); // => false

strbool.parse(/* anything else */); // ZodError<[{ code: "invalid_value" }]>
```

Чтобы настроить значения `truthy` и `falsy`:

```ts
// these are the defaults
z.stringbool({
    truthy: ['true', '1', 'yes', 'on', 'y', 'enabled'],
    falsy: ['false', '0', 'no', 'off', 'n', 'disabled'],
});
```

По умолчанию схема _нечувствительна к регистру_; все вводимые данные преобразуются в нижний регистр перед сравнением со значениями `truthy`/`falsy`. Чтобы сделать схему чувствительной к регистру:

```ts
z.stringbool({
    case: 'sensitive',
});
```

## Опциональные

Чтобы сделать схему _опциональной_ (то есть разрешить ввод `undefined`).

```ts
z.optional(z.literal('yoda')); // or z.literal("yoda").optional()
```

Это возвращает экземпляр `ZodOptional`, который оборачивает исходную схему. Чтобы извлечь внутреннюю схему:

```ts
optionalYoda.unwrap(); // ZodLiteral<"yoda">
```

## Nullables

Чтобы сделать схему _nullable_ (то есть разрешить ввод значений `null`).

```ts
z.nullable(z.literal('yoda')); // or z.literal("yoda").nullable()
```

Это возвращает экземпляр `ZodNullable`, который оборачивает исходную схему. Чтобы извлечь внутреннюю схему:

```ts
nullableYoda.unwrap(); // ZodLiteral<"yoda">
```

## Nullish

Чтобы сделать схему _nullish_ (как опциональной, так и допускающей нулевые значения):

```ts
const nullishYoda = z.nullish(z.literal('yoda'));
```

Более подробную информацию о концепции [nullish](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-7.html#nullish-coalescing) см. в руководстве по TypeScript.

## Unknown

Zod стремится полностью повторить систему типов TypeScript. Таким образом, Zod предоставляет API для представления следующих специальных типов:

```ts
// allows any values
z.any(); // inferred type: `any`
z.unknown(); // inferred type: `unknown`
```

## Never

Ни одно значение не пройдет проверку.

```ts
z.never(); // inferred type: `never`
```

## Объекты

Чтобы определить тип объекта:

```ts
// all properties are required by default
const Person = z.object({
    name: z.string(),
    age: z.number(),
});

type Person = z.infer<typeof Person>;
// => { name: string; age: number; }
```

По умолчанию все свойства являются обязательными. Чтобы сделать некоторые свойства необязательными:

```ts
const Dog = z.object({
    name: z.string(),
    age: z.number().optional(),
});

Dog.parse({ name: 'Yeller' }); // ✅
```

По умолчанию, нераспознанные ключи _удаляются_ из результата анализа:

```ts
Dog.parse({ name: 'Yeller', extraKey: true });
// => { name: "Yeller" }
```

### `z.strictObject`

Чтобы определить _строгую_ схему, которая выдает ошибку при обнаружении неизвестных ключей:

```ts
const StrictDog = z.strictObject({
    name: z.string(),
});

StrictDog.parse({ name: 'Yeller', extraKey: true });
// ❌ throws
```

### `z.looseObject`

Чтобы определить _гибкую_ схему, которая пропускает неизвестные ключи:

```ts
const LooseDog = z.looseObject({
    name: z.string(),
});

Dog.parse({ name: 'Yeller', extraKey: true });
// => { name: "Yeller", extraKey: true }
```

### `.catchall()`

Чтобы определить _универсальную схему_, которая будет использоваться для проверки любых нераспознанных ключей:

```ts
const DogWithStrings = z
    .object({
        name: z.string(),
        age: z.number().optional(),
    })
    .catchall(z.string());

DogWithStrings.parse({
    name: 'Yeller',
    extraKey: 'extraValue',
}); // ✅
DogWithStrings.parse({ name: 'Yeller', extraKey: 42 }); // ❌
```

### `.shape`

Для доступа к внутренним схемам:

```ts
Dog.shape.name; // => string schema
Dog.shape.age; // => number schema
```

### `.keyof()`

Чтобы создать схему `ZodEnum` из ключей схемы объекта:

```ts
const keySchema = Dog.keyof();
// => ZodEnum<["name", "age"]>
```

### `.extend()` {#extend}

Чтобы добавить дополнительные поля в схему объекта:

```ts
const DogWithBreed = Dog.extend({
    breed: z.string(),
});
```

Этот API можно использовать для перезаписи существующих полей! Будьте осторожны с этой функцией! Если две схемы имеют общие ключи, B перезапишет A.

!!!note "Альтернатива: деструктуризация"

    Вы также можете полностью избежать использования `.extend()`, создав совершенно новую схему объекта. Это сделает уровень строгости результирующей схемы визуально очевидным.

    ```ts
    const DogWithBreed = z.object({
    	// or z.strictObject() or z.looseObject()...
    	...Dog.shape,
    	breed: z.string(),
    });
    ```

    Вы также можете использовать эту функцию для объединения нескольких объектов за один раз.

    ```ts
    const DogWithBreed = z.object({
    	...Animal.shape,
    	...Pet.shape,
    	breed: z.string(),
    });
    ```

    Этот подход имеет несколько преимуществ:

    1.  Он использует функции на уровне языка ([синтаксис деструктуризации](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring)) вместо библиотечных API.
    2.  Один и тот же синтаксис работает в Zod и Zod Mini
    3.  Он более эффективен для `tsc` — метод `.extend()` может быть дорогостоящим для больших схем, а из-за [ограничения TypeScript](https://github.com/microsoft/TypeScript/pull/61505) его стоимость возрастает в квадратичной зависимости при цепочке вызовов
    4.  При желании вы можете изменить уровень строгости результирующей схемы с помощью `z.strictObject()` или `z.looseObject()`

### `.pick()`

Вдохновленный встроенными типами утилит `Pick` и `Omit` в TypeScript, Zod предоставляет специальные API для выбора и исключения определенных ключей из схемы объекта.

Начнем с этой начальной схемы:

```ts
const Recipe = z.object({
    title: z.string(),
    description: z.string().optional(),
    ingredients: z.array(z.string()),
});
// { title: string; description?: string | undefined; ingredients: string[] }
```

Чтобы выбрать определенные ключи:

```ts
const JustTheTitle = Recipe.pick({ title: true });
```

### `.omit()`

Чтобы пропустить определенные ключи:

```ts
const RecipeNoId = Recipe.omit({ id: true });
```

### `.partial()`

Для удобства Zod предоставляет специальный API для того, чтобы сделать некоторые или все свойства опциональными, вдохновленный встроенным типом утилиты TypeScript [`Partial`](https://www.typescriptlang.org/docs/handbook/utility-types.html#partialtype).

Чтобы сделать все поля опциональными:

```ts
const PartialRecipe = Recipe.partial();
// { title?: string | undefined; description?: string | undefined; ingredients?: string[] | undefined }
```

Чтобы сделать определенные свойства необязательными:

```ts
const RecipeOptionalIngredients = Recipe.partial({
    ingredients: true,
});
// { title: string; description?: string | undefined; ingredients?: string[] | undefined }
```

### `.required()`

Zod предоставляет API для того, чтобы сделать некоторые или все свойства _обязательными_, вдохновленный типом утилиты TypeScript [`Required`](https://www.typescriptlang.org/docs/handbook/utility-types.html#requiredtype).

Чтобы сделать все свойства обязательными:

```ts
const RequiredRecipe = Recipe.required();
// { title: string; description: string; ingredients: string[] }
```

Чтобы сделать определенные свойства обязательными:

```ts
const RecipeRequiredDescription = Recipe.required({
    description: true,
});
// { title: string; description: string; ingredients: string[] }
```

## Рекурсивные объекты

Чтобы определить самореференциальный тип, используйте [геттер](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/get) для ключа. Это позволяет JavaScript разрешить циклическую схему во время выполнения.

```ts
const Category = z.object({
    name: z.string(),
    get subcategories() {
        return z.array(Category);
    },
});

type Category = z.infer<typeof Category>;
// { name: string; subcategories: Category[] }
```

!!!warn ""

    Хотя рекурсивные схемы поддерживаются, передача циклических данных в Zod приведет к бесконечному циклу.

Вы также можете представлять _взаимно рекурсивные типы_:

```ts
const User = z.object({
    email: z.email(),
    get posts() {
        return z.array(Post);
    },
});

const Post = z.object({
    title: z.string(),
    get author() {
        return User;
    },
});
```

Все API объектов (`.pick()`, `.omit()`, `.required()`, `.partial()` и т. д.) работают так, как и ожидается.

### Ошибки цикличности

Из-за ограничений TypeScript рекурсивный вывод типов может быть привередливым и работает только в определенных сценариях. Некоторые более сложные типы могут вызывать рекурсивные ошибки типов, подобные этой:

```ts
const Activity = z.object({
    name: z.string(),
    get subactivities() {
        // ^ ❌ 'subactivities' implicitly has return type 'any' because it does not
        // have a return type annotation and is referenced directly or indirectly
        // in one of its return expressions.ts(7023)

        return z.nullable(z.array(Activity));
    },
});
```

В таких случаях ошибку можно устранить с помощью аннотации типа для проблемного геттера:

```ts
const Activity = z.object({
    name: z.string(),
    get subactivities(): z.ZodNullable<
        z.ZodArray<typeof Activity>
    > {
        return z.nullable(z.array(Activity));
    },
});
```

## Массивы

Чтобы определить схему массива:

```ts
const stringArray = z.array(z.string()); // or z.string().array()
```

Для доступа к внутренней схеме элемента массива.

```ts
stringArray.unwrap(); // => string schema
```

Zod реализует ряд проверок, специфичных для массивов:

```ts
z.array(z.string()).min(5); // must contain 5 or more items
z.array(z.string()).max(5); // must contain 5 or fewer items
z.array(z.string()).length(5); // must contain 5 items exactly
```

## Кортежи

В отличие от массивов, кортежи обычно представляют собой массивы фиксированной длины, которые задают разные схемы для каждого индекса.

```ts
const MyTuple = z.tuple([
    z.string(),
    z.number(),
    z.boolean(),
]);

type MyTuple = z.infer<typeof MyTuple>;
// [string, number, boolean]
```

Чтобы добавить вариативный («остаточный») аргумент:

```ts
const variadicTuple = z.tuple([z.string()], z.number());
// => [string, ...number[]];
```

## Объединения

Типы объединений (`A | B`) представляют собой логическое «ИЛИ». Схемы объединений Zod проверяют входные данные по порядку для каждого варианта. Возвращается первое значение, которое прошло проверку.

```ts
const stringOrNumber = z.union([z.string(), z.number()]);
// string | number

stringOrNumber.parse('foo'); // passes
stringOrNumber.parse(14); // passes
```

Чтобы извлечь внутренние варианты схем:

```ts
stringOrNumber.options; // [ZodString, ZodNumber]
```

## Дискриминированные объединения

[Дискриминированное объединение](https://www.typescriptlang.org/docs/handbook/2/narrowing.html#discriminated-unions) — это особый вид объединения, в котором а) все варианты являются схемами объектов, которые б) имеют общий ключ («дискриминатор»). На основе значения ключа дискриминатора TypeScript может «сузить» сигнатуру типа, как и ожидается.

```ts
type MyResult =
    | { status: 'success'; data: string }
    | { status: 'failed'; error: string };

function handleResult(result: MyResult) {
    if (result.status === 'success') {
        result.data; // string
    } else {
        result.error; // string
    }
}
```

Вы можете представить это с помощью обычного `z.union()`. Но обычные объединения являются _наивными_ — они проверяют входные данные по порядку для каждого варианта и возвращают первый, который проходит проверку. Это может быть медленно для больших объединений.

Поэтому Zod предоставляет API `z.discriminatedUnion()`, который использует _ключ дискриминатора_ для более эффективного разбора.

```ts
const MyResult = z.discriminatedUnion('status', [
    z.object({
        status: z.literal('success'),
        data: z.string(),
    }),
    z.object({
        status: z.literal('failed'),
        error: z.string(),
    }),
]);
```

!!!note "Вложенные дискриминированные объединения"

    Для сложных случаев использования дискриминируемые объединения могут быть вложенными. Zod определит оптимальную стратегию разбора, чтобы использовать дискриминаторы на каждом уровне.

    ```ts
    const BaseError = {
    	status: z.literal('failed'),
    	message: z.string(),
    };
    const MyErrors = z.discriminatedUnion('code', [
    	z.object({ ...BaseError, code: z.literal(400) }),
    	z.object({ ...BaseError, code: z.literal(401) }),
    	z.object({ ...BaseError, code: z.literal(500) }),
    ]);

    const MyResult = z.discriminatedUnion('status', [
    	z.object({
    		status: z.literal('success'),
    		data: z.string(),
    	}),
    	MyErrors,
    ]);
    ```

## Пересечения

Типы пересечений (`A & B`) представляют собой логическое «И».

```ts
const a = z.union([z.number(), z.string()]);
const b = z.union([z.number(), z.boolean()]);
const c = z.intersection(a, b);

type c = z.infer<typeof c>; // => number
```

Это может быть полезно для пересечения двух типов объектов.

```ts
const Person = z.object({ name: z.string() });
type Person = z.infer<typeof Person>;

const Employee = z.object({ role: z.string() });
type Employee = z.infer<typeof Employee>;

const EmployedPerson = z.intersection(Person, Employee);
type EmployedPerson = z.infer<typeof EmployedPerson>;
// Person & Employee
```

!!!warn ""

    При объединении схем объектов предпочтительнее использовать [`A.extend(B)`](#extend), а не пересечения. Использование `.extend()` даст вам новую схему объекта, тогда как `z.intersection(A, B)` возвращает экземпляр `ZodIntersection`, в котором отсутствуют общие методы объектов, такие как `pick` и `omit`.

## Record

Схемы записей используются для проверки типов, таких как `Record<string, number>`.

```ts
const IdCache = z.record(z.string(), z.string());
type IdCache = z.infer<typeof IdCache>; // Record<string, string>

IdCache.parse({
    carlotta: '77d2586b-9e8e-4ecf-8b21-ea7e0530eadd',
    jimmie: '77d2586b-9e8e-4ecf-8b21-ea7e0530eadd',
});
```

Ключевая схема может быть любой схемой Zod, которая может быть присвоена `string | number | symbol`.

```ts
const Keys = z.union([z.string(), z.number(), z.symbol()]);
const AnyObject = z.record(Keys, z.unknown());
// Record<string | number | symbol, unknown>
```

Чтобы создать схемы объектов, содержащие ключи, определенные перечислением:

```ts
const Keys = z.enum(['id', 'name', 'email']);
const Person = z.record(Keys, z.string());
// { id: string; name: string; email: string }
```

!!!info "Zod 4"

    В Zod 4, если вы передадите `z.enum` в качестве первого аргумента в `z.record()`, Zod тщательно проверит, что все значения перечисления присутствуют во входных данных в качестве ключей. Это поведение соответствует TypeScript:

    ```ts
    type MyRecord = Record<'a' | 'b', string>;
    const myRecord: MyRecord = { a: 'foo', b: 'bar' }; // ✅
    const myRecord: MyRecord = { a: 'foo' }; // ❌ missing required key `b`
    ```

    В Zod 3 исчерпываемость не проверялась. Чтобы воспроизвести прежнее поведение, используйте `z.partialRecord()`.

Если вам нужен тип записи _partial_, используйте `z.partialRecord()`. Это позволяет пропустить специальные проверки полноты, которые Zod обычно выполняет с ключевыми схемами `z.enum()` и `z.literal()`.

```ts
const Keys = z.enum(['id', 'name', 'email']).or(z.never());
const Person = z.partialRecord(Keys, z.string());
// { id?: string; name?: string; email?: string }
```

!!!note "Примечание о числовых ключах"

    Хотя TypeScript позволяет определять типы `Record` с ключами `number` (например, `Record<number, unknown>`), числовые ключи на самом деле не существуют в JavaScript, который преобразует все ключи в строки.

    ```ts
    const myObject = { 1: 'one' };

    Object.keys(myObject);
    // => ["1"]
    ```

    Как видите, JavaScript автоматически преобразует все числовые ключи в строки. Таким образом, использование `z.number()` в качестве схемы ключа внутри `z.record()` всегда будет вызывать ошибку при разборе, но Zod допускает это для обеспечения совместимости с системой типов TypeScript.

## Map

```ts
const StringNumberMap = z.map(z.string(), z.number());
type StringNumberMap = z.infer<typeof StringNumberMap>; // Map<string, number>

const myMap: StringNumberMap = new Map();
myMap.set('one', 1);
myMap.set('two', 2);

StringNumberMap.parse(myMap);
```

## Set

```ts
const NumberSet = z.set(z.number());
type NumberSet = z.infer<typeof NumberSet>; // Set<number>

const mySet: NumberSet = new Set();
mySet.add(1);
mySet.add(2);
NumberSet.parse(mySet);
```

Схемы наборов могут быть дополнительно ограничены с помощью следующих служебных методов.

```ts
z.set(z.string()).min(5); // must contain 5 or more items
z.set(z.string()).max(5); // must contain 5 or fewer items
z.set(z.string()).size(5); // must contain 5 items exactly
```

## Файлы

Для проверки экземпляров `File`:

```ts
const fileSchema = z.file();

fileSchema.min(10_000); // minimum .size (bytes)
fileSchema.max(1_000_000); // maximum .size (bytes)
fileSchema.mime('image/png'); // MIME type
fileSchema.mime(['image/png', 'image/jpeg']); // multiple MIME types
```

## Промисы

!!!warn "Устарело"

    `z.promise()` устарело в Zod 4. Существует крайне мало действительных случаев использования схемы `Promise`. Если вы подозреваете, что значение может быть `Promise`, просто `await` его перед разбором с помощью Zod.

!!!note "См. документацию по z.promise()"

    ```ts
    const numberPromise = z.promise(z.number());
    ```

    «Парсинг» работает немного по-другому с промисами схем. Проверка происходит в два этапа:

    1.  Zod синхронно проверяет, что входные данные являются экземпляром Promise (т. е. объектом с методами `.then` и `.catch`).
    2.  Zod использует `.then`, чтобы добавить дополнительный шаг проверки к существующему Promise. Вам нужно будет использовать `.catch` на возвращенном Promise, чтобы обработать ошибки проверки.

    ```ts
    numberPromise.parse('tuna');
    // ZodError: Non-Promise type: string

    numberPromise.parse(Promise.resolve('tuna'));
    // => Promise<number>

    const test = async () => {
    	await numberPromise.parse(Promise.resolve('tuna'));
    	// ZodError: Non-number type: string

    	await numberPromise.parse(Promise.resolve(3.14));
    	// => 3.14
    };
    ```

## Instanceof

Вы можете использовать `z.instanceof`, чтобы проверить, является ли ввод экземпляром класса. Это полезно для проверки ввода по отношению к классам, экспортированным из сторонних библиотек.

```ts
class Test {
    name: string;
}

const TestSchema = z.instanceof(Test);

TestSchema.parse(new Test()); // ✅
TestSchema.parse('whatever'); // ❌
```

## Уточнения

Каждая схема Zod хранит массив _уточнений_. Уточнения — это способ выполнения настраиваемой проверки, для которой Zod не предоставляет собственного API.

### `.refine()`

```ts
const myString = z
    .string()
    .refine((val) => val.length <= 255);
```

!!!warn ""

    Функции усовершенствования никогда не должны вызывать исключения. Вместо этого они должны возвращать ложное значение, чтобы сигнализировать об ошибке. Вызванные ошибки не перехватываются Zod.

#### `error`

Чтобы настроить сообщение об ошибке:

```ts
const myString = z
    .string()
    .refine((val) => val.length > 8, {
        error: 'Too short!',
    });
```

#### `abort`

По умолчанию проблемы валидации, выявленные в ходе проверок, считаются _продолжаемыми_; то есть Zod выполнит _все_ проверки по порядку, даже если одна из них вызовет ошибку валидации. Обычно это желательно, поскольку Zod может выявить как можно больше ошибок за один раз.

```ts
const myString = z
    .string()
    .refine((val) => val.length > 8, {
        error: 'Too short!',
    })
    .refine((val) => val === val.toLowerCase(), {
        error: 'Must be lowercase',
    });

const result = myString.safeParse('OH NO');
result.error.issues;
/* [
  { "code": "custom", "message": "Too short!" },
  { "code": "custom", "message": "Must be lowercase" }
] */
```

Чтобы пометить конкретное уточнение как _непродолжаемое_, используйте параметр `abort`. Проверка будет прервана, если проверка не пройдет.

```ts
const myString = z
    .string()
    .refine((val) => val.length > 8, {
        error: 'Too short!',
        abort: true,
    })
    .refine((val) => val === val.toLowerCase(), {
        error: 'Must be lowercase',
        abort: true,
    });

const result = myString.safeParse('OH NO');
result.error!.issues;
// => [{ "code": "custom", "message": "Too short!" }]
```

#### `path`

Чтобы настроить путь ошибки, используйте параметр `path`. Обычно это полезно только в контексте схем объектов.

```ts
const passwordForm = z
    .object({
        password: z.string(),
        confirm: z.string(),
    })
    .refine((data) => data.password === data.confirm, {
        message: "Passwords don't match",
        path: ['confirm'], // path of error
    });
```

Это установит параметр `path` в связанной проблеме:

```ts
const result = passwordForm.safeParse({
    password: 'asdf',
    confirm: 'qwer',
});
result.error.issues;
/* [{
  "code": "custom",
  "path": [ "confirm" ],
  "message": "Passwords don't match"
}] */
```

Чтобы определить асинхронное уточнение, просто передайте функцию `async`:

```ts
const userId = z.string().refine(async (id) => {
    // verify that ID exists in database
    return true;
});
```

!!!note ""

    Если вы используете асинхронные уточнения, вы должны использовать метод `.parseAsync` для анализа данных! В противном случае Zod выдаст ошибку.

    ```ts
    const result = await userId.parseAsync('abc123');
    ```

#### `when`

!!!note "Примечание"

    Это функция для опытных пользователей, которая может быть использована не по назначению, что увеличит вероятность возникновения невыявленных ошибок, происходящих изнутри ваших уточнений.

По умолчанию уточнения не запускаются, если уже были обнаружены какие-либо _непрерывные_ проблемы. Zod тщательно проверяет правильность сигнатуры типа значения, прежде чем передавать его в какие-либо функции уточнения.

```ts
const schema = z.string().refine((val) => {
    return val.length > 8;
});

schema.parse(1234); // invalid_type: refinement won't be executed
```

В некоторых случаях требуется более точный контроль над временем выполнения уточнений. Рассмотрим, например, проверку «подтверждение пароля»:

```ts
const schema = z
    .object({
        password: z.string().min(8),
        confirmPassword: z.string(),
        anotherField: z.string(),
    })
    .refine(
        (data) => data.password === data.confirmPassword,
        {
            message: 'Passwords do not match',
            path: ['confirmPassword'],
        }
    );

schema.parse({
    password: 'asdf',
    confirmPassword: 'asdf',
    anotherField: 1234, // ❌ this error will prevent the password check from running
});
```

Ошибка в `anotherField` помешает выполнению проверки подтверждения пароля, даже если эта проверка не зависит от `anotherField`. Чтобы контролировать, когда будет выполняться уточнение, используйте параметр `when`:

```ts
const schema = z
    .object({
        password: z.string().min(8),
        confirmPassword: z.string(),
        anotherField: z.string(),
    })
    .refine(
        (data) => data.password === data.confirmPassword,
        {
            message: 'Passwords do not match',
            path: ['confirmPassword'],

            // run if password & confirmPassword are valid
            when(payload) {
                // [!code ++]
                return schema // [!code ++]
                    .pick({
                        password: true,
                        confirmPassword: true,
                    }) // [!code ++]
                    .safeParse(payload.value).success; // [!code ++]
            }, // [!code ++]
        }
    );

schema.parse({
    password: 'asdf',
    confirmPassword: 'asdf',
    anotherField: 1234, // ❌ this error will prevent the password check from running
});
```

### `.superRefine()`

В Zod 4 функция `.superRefine()` была признана устаревшей и заменена на `.check()`.

```ts
const UniqueStringArray = z
    .array(z.string())
    .superRefine((val, ctx) => {
        if (val.length > 3) {
            ctx.addIssue({
                code: 'too_big',
                maximum: 3,
                origin: 'array',
                inclusive: true,
                message: 'Too many items 😡',
                input: val,
            });
        }

        if (val.length !== new Set(val).size) {
            ctx.addIssue({
                code: 'custom',
                message: `No duplicates allowed.`,
                input: val,
            });
        }
    });
```

### `.check()`

API `.refine()` является синтаксическим сахаром поверх более универсального (и многословного) API под названием `.check()`. Вы можете использовать этот API для создания нескольких проблем в одном уточнении или для полного контроля над сгенерированными объектами проблем.

```ts
const UniqueStringArray = z
    .array(z.string())
    .check((ctx) => {
        if (ctx.value.length > 3) {
            // полный контроль над объектами выпуска
            ctx.issues.push({
                code: 'too_big',
                maximum: 3,
                origin: 'array',
                inclusive: true,
                message: 'Too many items 😡',
                input: ctx.value,
            });
        }

        // создать несколько задач в одном уточнении
        if (ctx.value.length !== new Set(ctx.value).size) {
            ctx.issues.push({
                code: 'custom',
                message: `No duplicates allowed.`,
                input: ctx.value,
                continue: true, // сделать этот вопрос продолжительным
                // (по умолчанию: false)
            });
        }
    });
```

Обычный API `.refine` генерирует только проблемы с кодом ошибки `"custom"`, но `.check()` позволяет выбрасывать другие типы проблем. Для получения дополнительной информации о внутренних типах проблем Zod, прочтите документацию [Настройка ошибок](./error-customization.md).

## Пайпы {#pipes}

Схемы можно объединять в «пайпы». Пайпы в основном полезны при использовании в сочетании с [трансформациями](#transforms).

```ts
const stringToLength = z
    .string()
    .pipe(z.transform((val) => val.length));

stringToLength.parse('hello'); // => 5
```

## Трансформации {#transforms}

**Трансформации** — это особый вид схем. Вместо проверки вводимых данных они принимают любые данные и выполняют над ними определенные преобразования. Чтобы определить трансформацию:

```ts
const castToString = z.transform((val) => String(val));

castToString.parse('asdf'); // => "asdf"
castToString.parse(123); // => "123"
castToString.parse(true); // => "true"
```

Чтобы выполнить логику валидации внутри преобразования, используйте `ctx`. Чтобы сообщить о проблеме валидации, добавьте новую проблему в `ctx.issues` (аналогично API [`.check()`](#check)).

```ts
const coercedInt = z.transform((val, ctx) => {
    try {
        const parsed = Number.parseInt(String(val));
        return parsed;
    } catch (e) {
        ctx.issues.push({
            code: 'custom',
            message: 'Not a number',
            input: val,
        });

        // это специальная константа типа `never`
        // ее возвращение позволяет выйти из преобразования без влияния
        // на выведенный тип возвращаемого значения
        return z.NEVER;
    }
});
```

Чаще всего преобразования используются в сочетании с [пайпами](#pipes). Такое сочетание полезно для выполнения некоторой первоначальной проверки, а затем преобразования проанализированных данных в другую форму.

```ts
const stringToLength = z
    .string()
    .pipe(z.transform((val) => val.length));

stringToLength.parse('hello'); // => 5
```

### `.transform()`

Передача схемы в трансформацию является распространенным паттерном, поэтому Zod предоставляет удобный метод `.transform()`.

```ts
const stringToLength = z
    .string()
    .transform((val) => val.length);
```

Трансформации также могут быть асинхронными:

```ts
const idToUser = z.string().transform(async (id) => {
    // fetch user from database
    return db.getUserById(id);
});

const user = await idToUser.parseAsync('abc123');
```

!!!note ""

    Если вы используете асинхронные трансформации, при разборе данных необходимо использовать `.parseAsync` или `.safeParseAsync`! В противном случае Zod выдаст ошибку.

### `.preprocess()`

Передача преобразования в другую схему — еще один распространенный паттерн, поэтому Zod предоставляет удобную функцию `z.preprocess()`.

```ts
const coercedInt = z.preprocess((val) => {
    if (typeof val === 'string') {
        return Number.parseInt(val);
    }
    return val;
}, z.int());
```

## Настройки по умолчанию

Чтобы установить значение по умолчанию для схемы:

```ts
const defaultTuna = z.string().default('tuna');

defaultTuna.parse(undefined); // => "tuna"
```

В качестве альтернативы можно передать функцию, которая будет повторно выполняться всякий раз, когда необходимо сгенерировать значение по умолчанию:

```ts
const randomDefault = z.number().default(Math.random);

randomDefault.parse(undefined); // => 0.4413456736055323
randomDefault.parse(undefined); // => 0.1871840107401901
randomDefault.parse(undefined); // => 0.7223408162401552
```

## Предустановленные значения

В Zod установка _предустановленного_ значения приводит к прерыванию процесса разбора. Если входные данные имеют значение `undefined`, то предустановленное значение возвращается немедленно. Таким образом, предустановленное значение должно быть присвоимым _типу вывода_ схемы.

```ts
const schema = z
    .string()
    .transform((val) => val.length)
    .default(0);
schema.parse(undefined); // => 0
```

Иногда полезно определить значение _prefault_ («предустановленное значение по умолчанию»). Если входные данные имеют значение `undefined`, вместо них будет проанализировано значение `prefault`. Процесс анализа _не_ прерывается. Таким образом, значение prefault должно быть присвоимым _типу входных данных_ схемы.

```ts
z.string()
    .transform((val) => val.length)
    .prefault('tuna');
schema.parse(undefined); // => 4
```

Это также полезно, если вы хотите передать некоторое входное значение через некоторые изменяющиеся уточнения.

```ts
const a = z
    .string()
    .trim()
    .toUpperCase()
    .prefault('  tuna  ');
a.parse(undefined); // => "TUNA"

const b = z
    .string()
    .trim()
    .toUpperCase()
    .default('  tuna  ');
b.parse(undefined); // => "  tuna  "
```

## Catch

Используйте `.catch()`, чтобы определить запасное значение, которое будет возвращено в случае ошибки проверки:

```ts
const numberWithCatch = z.number().catch(42);

numberWithCatch.parse(5); // => 5
numberWithCatch.parse('tuna'); // => 42
```

В качестве альтернативы можно передать функцию, которая будет повторно выполняться всякий раз, когда необходимо сгенерировать значение catch.

```ts
const numberWithRandomCatch = z.number().catch((ctx) => {
    ctx.error; // the caught ZodError
    return Math.random();
});

numberWithRandomCatch.parse('sup'); // => 0.4413456736055323
numberWithRandomCatch.parse('sup'); // => 0.1871840107401901
numberWithRandomCatch.parse('sup'); // => 0.7223408162401552
```

## Брендированные типы

Система типов TypeScript является [структурной](https://www.typescriptlang.org/docs/handbook/type-compatibility.html), что означает, что два типа, которые являются структурно эквивалентными, считаются одинаковыми.

```ts
type Cat = { name: string };
type Dog = { name: string };

const pluto: Dog = { name: 'pluto' };
const simba: Cat = pluto; // works fine
```

В некоторых случаях может быть желательным имитировать [номинальное типирование](https://en.wikipedia.org/wiki/Nominal_type_system) внутри TypeScript. Это можно сделать с помощью _брендированных типов_ (также известных как «непрозрачные типы»).

```ts
const Cat = z.object({ name: z.string() }).brand<'Cat'>();
const Dog = z.object({ name: z.string() }).brand<'Dog'>();

type Cat = z.infer<typeof Cat>; // { name: string } & z.$brand<"Cat">
type Dog = z.infer<typeof Dog>; // { name: string } & z.$brand<"Dog">

const pluto = Dog.parse({ name: 'pluto' });
const simba: Cat = pluto; // ❌ not allowed
```

Под капотом это работает путем привязки «бренда» к выведенному типу схемы.

```ts
const Cat = z.object({ name: z.string() }).brand<'Cat'>();
type Cat = z.infer<typeof Cat>; // { name: string } & z.$brand<"Cat">
```

С этим брендом любые простые (без бренда) структуры данных больше не могут быть присвоены выведенному типу. Чтобы получить брендированные данные, необходимо проанализировать некоторые данные с помощью схемы.

> Обратите внимание, что брендированные типы не влияют на результат выполнения `.parse`. Это только статическая конструкция.

## Readonly

Чтобы пометить схему как доступную только для чтения:

```ts
const ReadonlyUser = z
    .object({ name: z.string() })
    .readonly();
type ReadonlyUser = z.infer<typeof ReadonlyUser>;
// Readonly<{ name: string }>
```

Выведенный тип новых схем будет помечен как `readonly`. Обратите внимание, что в TypeScript это влияет только на объекты, массивы, кортежи, `Set` и `Map`:

```ts
z.object({ name: z.string() }).readonly(); // { readonly name: string }
z.array(z.string()).readonly(); // readonly string[]
z.tuple([z.string(), z.number()]).readonly(); // readonly [string, number]
z.map(z.string(), z.date()).readonly(); // ReadonlyMap<string, Date>
z.set(z.string()).readonly(); // ReadonlySet<string>
```

Входные данные будут анализироваться как обычно, затем результат будет заморожен с помощью [`Object.freeze()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze), чтобы предотвратить изменения.

```ts
const result = ReadonlyUser.parse({ name: 'fido' });
result.name = 'simba'; // throws TypeError
```

## JSON

Для проверки любого значения, которое можно закодировать в формате JSON:

```ts
const jsonSchema = z.json();
```

Это удобный API, который возвращает следующую объединенную схему:

```ts
const jsonSchema = z.lazy(() => {
    return z.union([
        z.string(params),
        z.number(),
        z.boolean(),
        z.null(),
        z.array(jsonSchema),
        z.record(z.string(), jsonSchema),
    ]);
});
```

## Пользовательский

Вы можете создать схему Zod для любого типа TypeScript с помощью `z.custom()`. Это полезно для создания схем для типов, которые не поддерживаются Zod из коробки, таких как шаблонные строковые литералы.

```ts
const px =
    z.custom <
    `${number}px` >
    ((val) => {
        return typeof val === 'string'
            ? /^\d+px$/.test(val)
            : false;
    });

type px = z.infer<typeof px>; // `${number}px`

px.parse('42px'); // "42px"
px.parse('42vw'); // throws;
```

Если вы не предоставите функцию проверки, Zod будет допускать любые значения. Это может быть опасно!

```ts
z.custom<{ arg: string }>(); // performs no validation
```

Вы можете настроить сообщение об ошибке и другие параметры, передавая второй аргумент. Этот параметр работает так же, как параметр params в [`.refine`](#refine).

```ts
z.custom<...>((val) => ..., "custom error message");
```

## Функции

!!!warn ""

    В Zod 4 функция `z.function()` больше не возвращает схему Zod.

Zod предоставляет утилиту `z.function()` для определения функций, проверенных Zod. Таким образом, вы можете избежать смешивания кода проверки с вашей бизнес-логикой.

```ts
const MyFunction = z.function({
    input: [z.string()], // parameters (must be an array or a ZodTuple)
    output: z.number(), // return type
});
```

Схемы функций имеют метод `.implement()`, который принимает функцию и возвращает новую функцию, которая автоматически проверяет ее входы и выходы.

```ts
const computeTrimmedLength = MyFunction.implement(
    (input) => {
        // TypeScript knows input is a string!
        return input.trim().length;
    }
);

computeTrimmedLength('sandwich'); // => 8
computeTrimmedLength(' asdf '); // => 4
```

Эта функция вызовет ошибку `ZodError`, если входные данные недействительны:

```ts
computeTrimmedLength(42); // throws ZodError
```

Если вас интересует только проверка входных данных, поле `output` можно опустить.

```ts
const MyFunction = z.function({
    input: [z.string()], // parameters (must be an array or a ZodTuple)
});

const computeTrimmedLength = MyFunction.implement(
    (input) => input.trim.length
);
```
