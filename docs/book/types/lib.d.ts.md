# lib.d.ts

Специальный файл объявления `lib.d.ts` добавляется в каждую сборку TypeScript. Этот файл содержит объявления среды для различных распространенных конструкций JavaScript, присутствующих во время выполнения JavaScript и DOM.

-   Этот файл автоматически включается в контекст компиляции проекта TypeScript.
-   Цель этого файла - упростить процесс написания кода JavaScript с _контролем типов_.

Вы можете исключить этот файл из контекста компиляции, указав флаг командной строки компилятора `--noLib` (или `"noLib" : true` в `tsconfig.json`).

## Пример использования

Как всегда, давайте рассмотрим примеры использования этого файла:

```ts
var foo = 123;
var bar = foo.toString();
```

Этот код выполняет проверку типа правильно _поскольку_ функция `toString` определена в `lib.d.ts` для всех объектов JavaScript.

Если вы используете тот же пример кода с опцией `noLib`, вы получите ошибку проверки типа:

```ts
var foo = 123;
var bar = foo.toString(); // ОШИБКА: Свойство 'toString' не существует
// для типа 'number'.
```

Итак, теперь, когда вы понимаете важность `lib.d.ts`, как выглядит его содержимое? Мы рассмотрим это дальше.

## `lib.d.ts` взгляд изнутри

Содержимое `lib.d.ts` - это прежде всего набор _переменных_ объявлений, например. `window`, `document`, `math` и куча похожих объявлений _интерфейсов_, например `Window`, `Document`, `Math`.

Простейший способ прочитать документацию и описания типов глобального содержимого - это ввести код, _который вы знаете как работает_, например `Math.floor` и затем F12 (перейти к определению) с использованием вашей IDE (VSCode имеет отличную поддержку этого).

Давайте рассмотрим пример объявления _переменной_, например `window` определяется как:

```ts
declare var window: Window;
```

Это просто `declare var`, за которым следует имя переменной (здесь `window`) и интерфейс для описания типа (здесь интерфейс `Window`). Эти переменные обычно указывают на некоторый глобальный _интерфейс_, например вот небольшой пример (на самом деле довольно большого) интерфейса `Window`:

```ts
interface Window
    extends EventTarget,
        WindowTimers,
        WindowSessionStorage,
        WindowLocalStorage,
        WindowConsole,
        GlobalEventHandlers,
        IDBEnvironment,
        WindowBase64 {
    animationStartTime: number;
    applicationCache: ApplicationCache;
    clientInformation: Navigator;
    closed: boolean;
    crypto: Crypto;
    // и так далее и тому подобное...
}
```

Вы можете видеть, что в этих интерфейсах есть _много_ информации о типах. В отсутствие TypeScript _вам_ нужно было бы держать это в _своей_ голове. Теперь вы можете перенести эти знания в компилятор и сделать их легкодоступными, используя такие вещи, как `intellisense`.

Есть веская причина использования _интерфейсов_ для этих глобальных переменных. Это позволяет вам _добавлять дополнительные свойства_ к этим глобальным переменным _без_ необходимости изменять `lib.d.ts`. Мы рассмотрим эту концепцию дальше.

## Изменение нативных типов

Поскольку `интерфейс` в TypeScript является расширяемым, это означает, что вы можете просто добавить элементы к интерфейсам, объявленным в `lib.d.ts`, и TypeScript подхватит эти добавления. Обратите внимание, что вам нужно сделать эти изменения в [_глобальном модуле_](../project/modules.md), чтобы эти интерфейсы были связаны с `lib.d.ts`. Для этой цели мы даже рекомендуем создать специальный файл с именем [`global.d.ts`](../project/globals.md).

Вот несколько примеров, в которых мы добавляем элементы в `window`, `Math`, `Date`:

### Пример `window`

Просто добавьте элементы в интерфейс `Window`, например:

```ts
interface Window {
    helloWorld(): void;
}
```

Это позволит вам обезопасить себя проверкой типов:

```ts
// Добавить в среду выполнения
window.helloWorld = () => console.log('hello world');
// Вызвать
window.helloWorld();
// Неправильное использование и вы получите ошибку:
window.helloWorld('gracius'); // Ошибка: Предоставленные параметры
// не соответствуют подписи
```

### Пример `Math`

Глобальная переменная `Math` определена в `lib.d.ts` в виде (опять же, используйте ваши инструменты разработчика для перехода к определению):

```ts
/** Внутренний объект, который обеспечивает базовые математические
 *  функции и константы. */
declare var Math: Math;
```

т.е. переменная `Math` является экземпляром интерфейса `Math`. Интерфейс `Math` определяется как:

```ts
interface Math {
    E: number;
    LN10: number;
    // остальные ...
}
```

Это означает, что если вы хотите добавить элементы в глобальную переменную `Math`, вам просто нужно добавить его в глобальный интерфейс `Math`, например, рассмотрим проект [`seedrandom` project](https://www.npmjs.com/package/seedrandom), который добавляет функцию `seedrandom` к глобальному объекту `Math`. Она может быть объявлена довольно легко:

```ts
interface Math {
    seedrandom(seed?: string);
}
```

И далее вы можете просто использовать её:

```ts
Math.seedrandom();
// или
Math.seedrandom('Любая строка, которую вы хотите!');
```

### Пример `Date`

Если вы посмотрите на определение _переменной_ `Date` в `lib.d.ts`, вы увидите:

```ts
declare var Date: DateConstructor;
```

Интерфейс `DateConstructor` похож на то, что вы видели ранее с `Math` и `Window`, в том, что он содержит элементы, которые вы можете использовать вне глобальной переменной `Date`, например, `Date.now()`. В дополнение к этим элементам он содержит сигнатуры _конструктора_, которые позволяют вам создавать экземпляры `Date` (например `new Date()`). Фрагмент интерфейса `DateConstructor` показан ниже:

```ts
interface DateConstructor {
    new (): Date;
    // ... другие сигнатуры конструктора

    now(): number;
    // ... другие элементы
}
```

Рассмотрим проект [`datejs`](https://github.com/abritinthebay/datejs). DateJS добавляет элементы как в глобальную переменную `Date`, так и в экземпляры `Date`. Поэтому определения TypeScript для этой библиотеки будет выглядеть следующим образом ([Кстати, сообщество уже написало их для вас в этом случае](https://github.com/DefinitelyTyped/DefinitelyTyped/blob/master/types/datejs/index.d.ts)):

```ts
/** DateJS открытые статические методы */
interface DateConstructor {
    /** Получает дату, которая установлена на текущую дату. Время
     * устанавливается на начало дня (00:00 или 12:00 AM) */
    today(): Date;
    // ... и так далее, и тому подобное
}

/** DateJS открытые методы экземпляра */
interface Date {
    /** Добавляет указанное количество миллисекунд к этому экземпляру. */
    addMilliseconds(milliseconds: number): Date;
    // ... и так далее, и тому подобное
}
```

Это позволяет вам делать следующее с проверкой типов:

```ts
var today = Date.today();
var todayAfter1second = today.addMilliseconds(1000);
```

### Пример `string`

Если вы поищите информацию о string в `lib.d.ts`, вы обнаружите нечто похожее на то, что мы видели для `Date` (глобальная переменная `String`, интерфейс `StringConstructor`, интерфейс `String`). Однако следует отметить, что интерфейс `String` также влияет на строковые _литералы_, как показано в следующем примере кода:

```ts
interface String {
    endsWith(suffix: string): boolean;
}

String.prototype.endsWith = function (
    suffix: string
): boolean {
    var str: string = this;
    return (
        str &&
        str.indexOf(suffix, str.length - suffix.length) !==
            -1
    );
};

console.log('foo bar'.endsWith('bas')); // false
console.log('foo bas'.endsWith('bas')); // true
```

Подобные переменные и интерфейсы существуют и для других вещей, которые имеют как статические, так и методы экземпляров, такие как `Number`, `Boolean`, `RegExp` и т.д., и эти интерфейсы также влияют на литеральные экземпляры этих типов.

## Пример global из модуля

Мы рекомендуем создать `global.d.ts` для удобства поддержки. Однако вы можете проникнуть в _глобальное пространство имен_ изнутри _файлового модуля_, если захотите. Это делается с помощью `declare global { /*global namespace here*/ }`. Например. предыдущий пример также можно сделать так:

```ts
// Убедитесь, что это обрабатывается как модуль.
export {};

declare global {
    interface String {
        endsWith(suffix: string): boolean;
    }
}

String.prototype.endsWith = function (
    suffix: string
): boolean {
    var str: string = this;
    return (
        str &&
        str.indexOf(suffix, str.length - suffix.length) !==
            -1
    );
};

console.log('foo bar'.endsWith('bas')); // false
console.log('foo bas'.endsWith('bas')); // true
```

## Использование своего собственного lib.d.ts

Как мы упоминали ранее, использование логического флага компилятора `--noLib` заставляет TypeScript исключить автоматическое добавление `lib.d.ts`. Есть несколько причин, почему это может быть полезной функцией. Вот некоторые из распространенных:

-   Вы работаете в пользовательской среде JavaScript, которая _значительно_ отличается от стандартной среды выполнения на основе браузера.
-   Вам нравится иметь _строгий_ контроль над _глобальными переменными_, доступными в вашем коде. Например lib.d.ts определяет `item` как глобальную переменную, а вы не хотите, чтобы это попадало в ваш код.

После того как вы убрали стандартный lib.d.ts, вы можете добавить файл с аналогичным именем в контекст компиляции, и TypeScript подхватит его для проверки типов.

> Примечание: будьте осторожны с `--noLib`. Как только вы окажетесь на территории noLib, если вы решите поделиться своим проектом с другими, они будут тоже _вынуждены_ попасть на территорию noLib (или, скорее, на территорию _вашей lib_). Еще хуже ситуация, если вы добавите _их_ код в ваш проект, вам может понадобиться портировать его в _код на основе вашей lib_.

## Эффект зависимости `lib.d.ts` от параметров вывода

Настройка компилятора с выводом в `es6` заставляет `lib.d.ts` включать _дополнительные_ объявления среды для более современных (es6) вещей, таких как `Promise`. Этот особый эффект меняет _окружение_ кода и желаем для некоторых разработчиков, а для других он проблематичен, поскольку он связывает _генерацию кода_ со _средой кода_.

Однако, если вы хотите более детальный контроль над своей средой, вы должны использовать параметр `--lib`, который мы обсудим далее.

## lib опция

Иногда (довольно часто) вы хотите разорвать связь между выводом компиляции (сгенерированной версией JavaScript) и поддержкой окружающих библиотек. Типичным примером является `Promise`, например, сегодня (в июне 2016 года) вы, скорее всего, захотите `--target es5`, но по-прежнему будете использовать новейшие функции, такие как `Promise`. Для поддержки этого вы можете получить явный контроль над `lib` с помощью опции компилятора `lib`.

> Примечание: использование `--lib` позволяет отделить любую lib магию от `--target`, обеспечивая вам лучший контроль.

Вы можете использовать эту опцию в командной строке или в `tsconfig.json` (рекомендуется):

**Командная строка**:

```
tsc --target es5 --lib dom,es6
```

**tsconfig.json**:

```json
"compilerOptions": {
    "lib": ["dom", "es6"]
}
```

libs можно классифицировать следующим образом:

-   JavaScript версия:
    -   es5
    -   es6
    -   es2015
    -   es7
    -   es2016
    -   es2017
    -   esnext
-   Среда выполнения
    -   dom
    -   dom.iterable
    -   webworker
    -   scripthost
-   ESNext по отдельным фичам (меньше, чем версия)
    -   es2015.core
    -   es2015.collection
    -   es2015.generator
    -   es2015.iterable
    -   es2015.promise
    -   es2015.proxy
    -   es2015.reflect
    -   es2015.symbol
    -   es2015.symbol.wellknown
    -   es2016.array.include
    -   es2017.object
    -   es2017.sharedmemory
    -   esnext.asynciterable

> ПРИМЕЧАНИЕ: опция `--lib` обеспечивает чрезвычайно гибкий контроль. Так что, скорее всего вы хотите выбрать отдельные фичи из версий + среду выполнения.
> Если --lib не указан, будет добавлена lib по умолчанию:

-   Для --target es5 => es5, dom, scripthost
-   Для --target es6 => es6, dom, dom.iterable, scripthost

Моя личная рекомендация:

```json
"compilerOptions": {
    "target": "es5",
    "lib": ["es6", "dom"]
}
```

Пример для Symbol с ES5:

Symbol API не добавляется, когда вывод в es5. На самом деле мы получаем ошибку вроде: [ts] Не удается найти имя 'Symbol'.
Мы можем использовать "target": "es5" в сочетании с "lib" для предоставления Symbol API в TypeScript:

```json
"compilerOptions": {
    "target": "es5",
    "lib": ["es5", "dom", "scripthost", "es2015.symbol"]
}
```

## Polyfill для старых движков JavaScript

> [Egghead PRO видео по этой теме](https://egghead.io/lessons/typescript-using-es6-and-esnext-with-typescript)

Существует довольно много функций среды выполнения, таких как `Map` / `Set` и даже `Promise` (этот список, конечно, со временем изменится), которые вы можете использовать с современными опциями `lib`. Чтобы использовать их, все, что вам нужно сделать, это использовать `core-js`. Просто установите:

```
npm install core-js --save-dev
```

И добавьте импорт в точку входа вашего приложения:

```js
import 'core-js';
```

И он должен заполифиллить эти функции среды выполнения для вас 🌹.
