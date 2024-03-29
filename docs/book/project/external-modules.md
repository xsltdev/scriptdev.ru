# Внешние модули

Внешний модуль TypeScript предоставляет много возможностей и удобства. Здесь мы обсудим его возможности и некоторые паттерны, используемые на практике.

## Прояснение: модули commonjs, amd, es modules, и другие

Прежде всего нам нужно прояснить (ужасную) несогласованность систем модулей. Я просто дам вам свою _текущую_ рекомендацию и уберу шум, т.е. скрою некоторые _возможные_ варианты работы.

Из _одного и того же TypeScript_ вы можете генерировать разный код _JavaScript_ в зависимости от опции `module`. Вот вещи, которые вы можете игнорировать (я не заинтересован в объяснении устаревших технологий):

-   AMD: Не используйте. Использовались только в браузерах.
-   SystemJS: Был неплохим вариантом. Заменен ES модулями.
-   ES Модули: Еще не готовы.

Теперь это всего лишь варианты _генерации JavaScript_. Вместо этих опций используйте `module:commonjs`

То, как вы _пишете_ модули TypeScript, также немного запутанно. Опять же, вот как лучше не делать _сегодня_:

-   `import foo = require('foo')`. т.е. `import/require`. Вместо этого используйте синтаксис модуля ES.

Окей, вместе с тем давайте посмотрим на синтаксис модуля ES.

> Резюме: используйте `module:commonjs` и используйте синтаксис модуля ES для импорта / экспорта / создания модулей.

## Синтаксис ES модулей

-   Экспортировать переменную (или тип) можно просто добавив ключевое слово `export`, например

```js
// файл `foo.ts`
export let someVar = 123;
export type SomeType = {
    foo: string,
};
```

-   Экспорт переменной или типа в отдельной `export` инструкции, например

```js
// файл `foo.ts`
let someVar = 123;
type SomeType = {
    foo: string,
};
export { someVar, SomeType };
```

-   Экспорт переменной или типа в отдельной `export` инструкции _с переименованием_, например

```js
// файл `foo.ts`
let someVar = 123;
export { someVar as aDifferentName };
```

-   Импортируйте переменную или тип, используя `import`, например

```js
// файл `bar.ts`
import { someVar, SomeType } from './foo';
```

-   Импортировать переменную или тип, используя `import` _с переименованием_, например

```js
// файл `bar.ts`
import { someVar as aDifferentName } from './foo';
```

-   Импортируйте все из модуля под новым именем с помощью `import * as`, например

```js
// файл `bar.ts`
import * as foo from './foo';
// вы можете использовать `foo.someVar` и` foo.SomeType`
// и все остальное, что foo может экспортировать
```

-   Импорт файла для общего доступа с помощью одной _только_ инструкции импорта:

```js
import 'core-js'; // библиотека основных полифиллов
```

-   Реэкспорт всех переменных из другого модуля

```js
export * from './foo';
```

-   Реэкспорт только некоторых переменных из другого модуля

```js
export { someVar } from './foo';
```

-   Реэкспорт только некоторых переменных из другого модуля _с переименованием_

```js
export { someVar as aDifferentName } from './foo';
```

## exports/imports по умолчанию

Как вы узнаете позже, я не фанат экспорта по умолчанию. Тем не менее есть синтаксис для экспорта и использования экспорта по умолчанию

-   Экспорт с использованием `export по умолчанию`
    -   перед переменной (не требуется `let / const / var`)
    -   перед функцией
    -   перед классом

```js
// какая-то переменная
export default someVar = 123;
// ИЛИ какая-то функция
export default function someFunction() {}
// ИЛИ какой-то класс
export default class SomeClass {}
```

-   Импорт с использованием `import someName from "someModule"` (вы можете назвать импорт как угодно), например

```js
import someLocalNameForThisFile from '../foo';
```

## Пути к модулям

> Я собираюсь предложить использовать `moduleResolution: commonjs`. Эта опция должна быть в вашей конфигурации TypeScript. А эта настройка `module:commonjs` подразумевается автоматически.

Есть два разных вида модулей. Различие обусловлено секцией пути в операторе импорта (например, `import foo from 'ЭТО_СЕКЦИЯ_ПУТИ'`).

-   Модули с относительным путем (где путь начинается с `.`, например, `./someFile` или `../../someFolder/someFile` и т.д.)
-   Другие модули с динамической подстановкой (например, `'core-js'` или `'typestyle'` или `'react'` или даже `'react/core'` и т.д.)

Основное отличие состоит в том, _как модуль находится в файловой системе_.

> Я буду использовать концептуальный термин _место_, который я объясню после упоминания шаблона поиска.

### Относительные пути к модулям

Это легко, просто следуй по пути :), например

-   если файл `bar.ts` выполняет `import * as foo from './foo';`, то файл `foo` должен существовать в той же папке.
-   если файл `bar.ts` выполняет `import * as foo from '../foo';`, то файл `foo` должен существовать в папке выше.
-   если файл `bar.ts` выполняет `import * as foo from '../someFolder/foo';`, то в папке выше должна быть папка `someFolder` с местом `foo`

Или любой другой относительный путь, который вы можете придумать :)

### Динамическая подстановка

Когда путь импорта _не_ относительный, поиск определяется [_node style resolution_](https://nodejs.org/api/modules.html#modules_all_together). Здесь я приведу только простой пример:

-   У вас есть `import * as foo from 'foo'`, ниже перечислены места, которые проверяются _по порядку_

    -   `./node_modules/foo`
    -   `../node_modules/foo`
    -   `../../node_modules/foo`
    -   И далее до корня файловой системы

-   У вас есть `import * as foo from 'something/foo'`, ниже перечислены места, которые проверяются _по порядку_
    -   `./node_modules/something/foo`
    -   `../node_modules/something/foo`
    -   `../../node_modules/something/foo`
    -   И далее до корня файловой системы

## Что такое _место_

Когда я говорю _проверенные места_, я имею в виду, что в этом месте проверяются следующие вещи. Например, для места `foo`:

-   Если место представляет собой файл, например, `foo.ts`, завершено!
-   иначе, если место - это папка, а есть файл `foo/index.ts`, завершено!
-   в противном случае, если местом является папка и существует `foo/package.json` с указанным и существующим в ключе `types` файлом, тогда завершено!
-   в противном случае, если местом является папка и существует package.json с указанным и существующим в ключе `main` файлом, то завершено!

Под файлом я имею в виду `.ts` / `.d.ts` и `.js`.

И это все. Теперь вы являетесь экспертами по поиску модулей (немаленький подвиг!).

## Отмена динамического поиска _только для типов_

Вы можете объявить модуль _глобально_ для вашего проекта, используя `declare module 'somePath'`, и тогда импорт исполнит _волшебным образом_ этот путь,

например,

```ts
// global.d.ts
declare module 'foo' {
    // Некоторые объявления переменных
    export var bar: number; /*пример*/
}
```

и потом:

```ts
// anyOtherTsFileInYourProject.ts
import * as foo from 'foo';
// TypeScript предполагает (без каких-либо подстановок), что
// foo это {bar:number}
```

## `import/require` только для импорта типа

Следующая инструкция:

```ts
import foo = require('foo');
```

на самом деле делает _две_ вещи:

-   Импортирует информацию о типах модуля foo.
-   Определяет необходимые зависимости в среде выполнения для модуля foo.

Вы можете выбрать, чтобы загружалась только _информация о типе_ и избежать загрузки зависимостей. Прежде чем продолжить, вы можете вспомнить раздел [_области объявлений_](../project/declarationspaces.md) этой книги.

Если вы не используете импортированное имя в области объявления переменных, то импорт полностью удаляется из сгенерированного JavaScript. Это лучше всего объяснить на примерах. Как только вы поймете это, мы представим вам варианты использования.

### Пример 1

```ts
import foo = require('foo');
```

сгенерирует следующий JavaScript:

```js

```

Всё в порядке. _Пустой_ файл, так как `foo` не используется.

### Пример 2

```ts
import foo = require('foo');
var bar: foo;
```

сгенерирует следующий JavaScript:

```js
var bar;
```

Это потому, что `foo` (или любое из его свойств, например, `foo.bas`) никогда не используется в качестве переменной.

### Пример 3

```ts
import foo = require('foo');
var bar = foo;
```

сгенерирует следующий JavaScript (при условии использования `commonjs` параметра конфига):

```js
var foo = require('foo');
var bar = foo;
```

Это потому что `foo` используется как переменная.

## Вариант использования: Отложенная загрузка

Вывод типа должен быть сделан _предварительно_. Это означает, что если вы хотите использовать какой-либо тип из файла `foo` в файле `bar`, вам придется сделать следующее:

```ts
import foo = require('foo');
var bar: foo.SomeType;
```

Однако вы можете захотеть загружать файл `foo` только во время выполнения при определенных условиях. В таких случаях вы должны использовать `импортированное` значение только в _описаниях типа_, но **не** в качестве _переменной_. Это удаляет любой _предварительный_ код зависимостей в среде выполнения, внедренный TypeScript. Затем _вручную импортируйте_ реальный модуль, используя код, необходимый для вашего загрузчика модулей.

В качестве примера рассмотрим следующий код, основанный на `commonjs`, где мы загружаем модуль `'foo'` только при вызове определенной функции:

```ts
import foo = require('foo');

export function loadFoo() {
    // Это отложенная загрузка `foo` и использование исходного модуля
    // *только* в качестве описания типа
    var _foo: typeof foo = require('foo');
    // Теперь используем `_foo` в качестве переменной вместо `foo`.
}
```

Подобный пример в `amd` (используя requirejs) будет выглядеть так:

```ts
import foo = require('foo');

export function loadFoo() {
    // Это отложенная загрузка `foo` и использование исходного модуля
    // *только* в качестве описания типа
    require(['foo'], (_foo: typeof foo) => {
        // Теперь используем `_foo` в качестве переменной вместо `foo`.
    });
}
```

Этот шаблон обычно используется:

-   в веб-приложениях, где вы загружаете определенный JavaScript на определенных роутах,
-   в nodejs приложениях, где вы загружаете только определенные модули, если это необходимо для ускорения загрузки приложений.

## Вариант использования: размыкание круговых зависимостей

Как и в случае с отложенной загрузкой, некоторые загрузчики модулей (commonjs/node и amd/requirejs) плохо работают с циклическими зависимостями. В таких случаях полезно иметь _отложенную загрузку_ кода в одном направлении и загружать модули предварительно в противоположном направлении.

## Вариант использования: безопасный импорт

Иногда вы хотите загрузить файл только для побочного эффекта (например, для регистрации в какой-то библиотеке [CodeMirror addons](https://codemirror.net/doc/manual.html#addons) и т. д.). Тем не менее, если вы просто выполните `import/require`, транспиленный JavaScript не будет содержать зависимости от модуля, и ваш загрузчик модулей (например, webpack) может полностью игнорировать импорт. В таких случаях вы можете использовать переменную `ensureImport`, чтобы гарантировать, что скомпилированный JavaScript принимает зависимость от модуля, например:

```ts
import foo = require('./foo');
import bar = require('./bar');
import bas = require('./bas');
const ensureImport: any = foo && bar && bas;
```
