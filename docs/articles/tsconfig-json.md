---
description: Чтобы чувствовать себя более уверенно в своем tsconfig.json, я решил просмотреть документацию по tsconfig.json, собрать все часто используемые опции и описать их ниже
---

# Контрольный список для tsconfig.json

Версия: TypeScript 5.7

Чтобы чувствовать себя более уверенно в своем `tsconfig.json`, я решил просмотреть [документацию по `tsconfig.json`](https://www.typescriptlang.org/tsconfig/), собрать все часто используемые опции и описать их ниже:

-   Вы можете присоединиться ко мне и пройтись по опциям вместе со мной. После этого вы должны быть в состоянии понять весь ваш `tsconfig`.
-   Или вы можете перейти к краткому описанию в конце.
-   Я также привожу рекомендации по `tsconfig.json` от нескольких других людей.

Мне интересно, каков ваш опыт работы с `tsconfig.json`: Согласны ли вы с моими рекомендациями? Может быть, я что-то упустил?

## Функции, не вошедшие в эту статью

В этой статье описывается только создание проектов, локальные модули которых состоят из ESM. Однако в ней даются советы по импорту CommonJS.

Здесь не объясняется:

-   Импорт и проверка типа обычного JavaScript в вашей кодовой базе, а именно опции [`allowJs`](https://www.typescriptlang.org/tsconfig/#allowJs) и [`checkJs`](https://www.typescriptlang.org/tsconfig/#checkJs).
-   Как настроить JSX. См. раздел [«JSX»](https://www.typescriptlang.org/docs/handbook/jsx.html) в TypeScript Handbook.
-   «Проекты» (полезно для монорепов): опция `composite` и т. д. Дополнительную информацию по этой теме см:
    -   Глава [«Ссылки на проекты»](https://www.typescriptlang.org/docs/handbook/project-references.html) в TypeScript Handbook.
    -   Заметка в моем блоге [«Простые монорепозитории с помощью рабочих пространств npm и ссылок на проекты TypeScript»](https://2ality.com/2021/07/simple-monorepos.html)

## Нотация

Для отображения выводимых типов в исходном коде я использую пакет npm `ts-expect` - например:

```ts
// Check that the inferred type of `someVariable` is `boolean`
expectType<boolean>(someVariable);
```

Я часто использую запятые в конце JSON, потому что это поддерживается в `tsconfig.json` и потому что это помогает при перестановке, копировании и т. д.

## Расширение базовых файлов с помощью `extends`

Эта опция позволяет нам ссылаться на существующий `tsconfig.json` через спецификатор модуля (как если бы мы импортировали JSON-файл). Этот файл становится базой, которую расширяет наш `tsconfig`. Это означает, что наш `tsconfig` имеет все опции базы, но может переопределять любые из них и добавлять опции, не упомянутые в базе.

В репозитории GitHub [`tsconfig/bases`](https://github.com/tsconfig/bases) перечислены базы, которые доступны в пространстве имен npm `@tsconfig` и могут быть использованы подобным образом (после их локальной установки через npm):

```json
{
    "extends": "@tsconfig/node-lts/tsconfig.json"
}
```

Увы, ни один из этих файлов не подходит для моих нужд. Но они могут послужить вдохновением для вашего `tsconfig`.

## Где находятся входные файлы?

```json
{
    "include": ["src/**/*", "test/**/*"]
}
```

С одной стороны, мы должны указать TypeScript, какие файлы будут использоваться для ввода. Вот доступные варианты:

-   `files`: исчерпывающий массив всех входных файлов
-   `include`: Указывает входные файлы через массив шаблонов с подстановочными знаками, которые интерпретируются как относительные к `tsconfig.json`.
-   `exlude`: Указывает, какие файлы должны быть исключены из набора включаемых файлов - с помощью массива шаблонов.

## Каков результат?

### Куда записываются выходные файлы?

```json
{
    "compilerOptions": {
        "rootDir": ".",
        "outDir": "dist"
    }
}
```

Как TypeScript определяет, куда записать выходной файл:

-   Берется входной путь (относительно `tsconfig.json`),
-   удаляет префикс, указанный в `rootDir`, и
-   «добавляет» результат в `outDir`.

По умолчанию значение `rootDir` - это самый длинный общий префикс относительных путей входных файлов.

В качестве примера рассмотрим следующий `tsconfig.json`:

```json
{
    "include": ["src/**/*", "test/**/*"],
    "compilerOptions": {
        "rootDir": ".",
        "outDir": "dist"
    }
}
```

Это файловая структура проекта:

```
/tmp/my-proj/
  tsconfig.json
  src/
    main.ts
  test/
    test.ts
```

Компилятор TypeScript выдает такой результат:

```
/tmp/my-proj/
  dist/
    src/
      main.js
    test/
      test.js
```

Если мы удалим `rootDir` из `tsconfig.json`, то результат будет таким же, потому что его значение по умолчанию - «`.`».

Однако результат будет другим, если мы также изменим `include`:

```json
{
    "include": ["src/**/*"],
    "compilerOptions": {
        "outDir": "dist"
    }
}
```

Теперь значение по умолчанию для `rootDir` равно `src`, и вывод будет таким:

```
/tmp/my-proj/
  dist/
    main.js
```

Поскольку значение `rootDir` по умолчанию меняется в зависимости от `include`, я предпочитаю указывать его явно в `tsconfig.json`. Но вы можете не указывать его, если вас устраивает то, как работает значение по умолчанию.

### Карты исходного кода

```json
{
    "compilerOptions": {
        "sourceMap": true
    }
}
```

`sourceMap` создает файлы карты исходных текстов, которые указывают на исходный TypeScript из транспилированного JavaScript. Это помогает при отладке и обычно является хорошей идеей.

### Создание файлов `.d.ts` (например, для библиотек)

Если мы хотим, чтобы код TypeScript потреблял наш транспилированный код TypeScript, нам обычно нужно включать файлы `.d.ts`:

```json
{
    "compilerOptions": {
        "declaration": true,
        "declarationMap": true // enables importers to jump to source
    }
}
```

В качестве опции мы можем включить исходный код TypeScript в наш пакет npm и активировать `declarationMap`. Импортеры могут, например, щелкнуть на типах или перейти к определению значения, а их редактор отправит их к исходному коду.

#### Опция `declarationDir`

По умолчанию каждый файл `.d.ts` помещается рядом с файлом `.js`. Если вы хотите изменить это, вы можете использовать опцию [`declarationDir`](https://www.typescriptlang.org/tsconfig/#declarationDir).

### Тонкая настройка выдаваемых файлов

```json
{
    "compilerOptions": {
        "newLine": "lf",
        "removeComments": false
    }
}
```

Значения, указанные выше, являются значениями по умолчанию.

-   `newLine` настраивает окончания строк для выдаваемых файлов. Допустимыми значениями являются:
    -   `"lf": "\n"` (Unix)
    -   `"crlf": "\r\n"` (Windows)
-   `removeComments`: Если активна, все комментарии в файлах TypeScript будут опущены в транслируемых файлах JavaScript. Я слабо склоняюсь к тому, чтобы оставить значение по умолчанию и не удалять комментарии:
    -   Это помогает при чтении транслированного JavaScript - особенно если исходный код TypeScript не включен.
    -   Сборщики удаляют комментарии.
    -   На Node.js дополнительное бремя не имеет большого значения.

## Особенности языка и платформы

```json
{
    "compilerOptions": {
        "target": "ES2024",
        // Omit if you want to use the DOM
        "lib": ["ES2024"]
    }
}
```

### `target`

`target` определяет, какой новый синтаксис JavaScript будет транспонирован в старый синтаксис. Например, если target - `"ES5"`, то стрелочная функция `() => {}` будет транспонирована в функцию выражения `function () {}`.

-   Рекомендуемые настройки для различных платформ можно посмотреть в репозитории GitHub [`tsconfig/bases`](https://github.com/tsconfig/bases).
-   Значение `"ESNext"` означает «наивысшая версия, поддерживаемая установленным TypeScript». Поскольку это значение меняется между версиями TypeScript, оно может вызвать проблемы при обновлении.

Я задаюсь вопросом, должна ли быть опция, позволяющая никогда не транспонировать JavaScript-функции. С другой стороны, возможность писать современный JavaScript на потенциально старых браузерах весьма удобна.

#### Как выбрать хорошую целевую платформу

Мы должны выбрать версию ECMAScript, которая подходит для наших целевых платформ. Есть две таблицы, которые дают хороший обзор:

-   Для браузеров: [compat-table.github.io](https://compat-table.github.io/compat-table/es2016plus/)
-   Для Node.js: [node.green](https://node.green/)

Мы также можем посмотреть [официальные базы tsconfig](https://github.com/tsconfig/bases), которые предоставляют значения для `target`.

### `lib`

`lib` определяет, какие типы для встроенных API доступны - например, `Math` или методы встроенных типов:

-   В [документации TypeScript](https://www.typescriptlang.org/tsconfig/#lib) описано, какие значения можно добавлять в массив. Полный их список можно найти в [репозитории TypeScript](https://github.com/microsoft/TypeScript/tree/main/src/lib). Если вы ищете тип, то ищите его в этих файлах!
-   Там есть такие категории, как `"ES2024"` и `"DOM"`, и такие подкатегории, как `"DOM.Iterable"` и `"ES2024.Promise"`.
-   Значения не зависят от регистра: Предложения автозаполнения Visual Studio Code содержат много заглавных букв, а имена файлов - ни одной. Значения `lib` могут быть записаны в любом случае.

Когда TypeScript поддерживает тот или иной API? Он должен быть «доступен без префиксов/флагов как минимум в 2 браузерных движках (то есть не только в 2 браузерах chromium)» ([источник](https://github.com/microsoft/TypeScript/tree/main/src/lib)).

#### Настройка `lib` через `target`

`target` определяет значение `lib` по умолчанию: Если последний параметр опущен, а `target` - `"ES20YY"`, то будет использоваться `"ES20YY.Full"`. Однако это не то значение, которое мы можем использовать сами. Если мы хотим воспроизвести то, что делает удаление `lib`, мы должны сами перечислить содержимое (например, `es2024.full.d.ts`):

```ts
/// <reference lib="es2024" />
/// <reference lib="dom" />
/// <reference lib="webworker.importscripts" />
/// <reference lib="scripthost" />
/// <reference lib="dom.iterable" />
/// <reference lib="dom.asynciterable" />
```

В этом файле мы можем наблюдать интересное явление:

-   Категория `"ES20YY"` обычно включает все свои подкатегории.
-   Категория `"DOM"` не включает - например, подкатегория `"DOM.Iterable"` еще не является ее частью.

Помимо прочего, `"DOM.Iterable"` позволяет выполнять итерацию над списками NodeLists - например:

```ts
for (const x of document.querySelectorAll('div')) {
}
```

### Типы для встроенных API Node.js

Типы для API Node.js должны быть установлены с помощью [пакета npm](https://www.npmjs.com/package/@types/node):

```bash
npm install @types/node
```

## Система модулей

### `module`: Как TypeScript будет искать импортированные модули?

Эти опции влияют на то, как TypeScript ищет импортированные модули:

```json
{
    "compilerOptions": {
        "module": "NodeNext",
        "noUncheckedSideEffectImports": true
    }
}
```

#### Опция `module`

С помощью этой опции мы определяем системы для работы с модулями. Если мы правильно настроим ее, то позаботимся и о связанной с ней опции `moduleResolution`, для которой она предоставляет хорошие значения по умолчанию. Документация TypeScript рекомендует использовать одно из двух следующих значений:

-   Node.js: `"NodeNext"` поддерживает как CommonJS, так и последние возможности ESM.
    -   Подразумевает `"moduleResolution": "NodeNext"`.
-   Bundlers: `"Preserve"` поддерживает как CommonJS, так и последние возможности ESM. Это поведение соответствует тому, что делает большинство бандлеров.
    -   Подразумевает `"moduleResolution": "bundler"`.

Учитывая, что бандлеры в основном имитируют работу Node.js, я всегда использую `"NodeNext"` и не сталкивался с какими-либо проблемами.

Обратите внимание, что в обоих случаях TypeScript заставляет нас указывать полные имена локальных модулей, которые мы импортируем. Мы не можем опускать расширения имен файлов, как это часто делалось, когда Node.js компилировался только в CommonJS. Новый подход отражает то, как работает ESM на чистом JavaScript.

#### Опция `noUncheckedSideEffectImports`

По умолчанию TypeScript не жалуется, если пустой импорт не существует. Причина такого поведения заключается в том, что это шаблон, поддерживаемый некоторыми бандлерами для связывания артефактов, не относящихся к TypeScript, с модулями. А TypeScript видит только файлы TypeScript. Вот как выглядит такой импорт:

```ts
import './component-styles.css';
```

Интересно, что TypeScript нормально воспринимает несуществующие файлы TypeScript, импортированные в пустоту. Он жалуется только в том случае, если мы импортируем что-то из несуществующего файла.

```ts
import './does-not-exist.js'; // no error!
```

Установка значения `noUncheckedSideEffectImports` в `true` изменит это. Позже я расскажу об альтернативе для импорта артефактов, не относящихся к TypeScript.

### Запуск TypeScript напрямую (без генерации JS-файлов)

Большинство небраузерных JavaScript-платформ теперь могут запускать код TypeScript напрямую, без его транспонирования.

```json
{
    "compilerOptions": {
        "allowImportingTsExtensions": true,
        // Only needed if compiling to JavaScript:
        "rewriteRelativeImportExtensions": true
    }
}
```

-   `allowImportingTsExtensions`: Эта опция позволяет нам при импорте ссылаться на TypeScript-версию модуля, а не на его транслируемую версию.
-   `rewriteRelativeImportExtensions`: С помощью этой опции мы также можем транслировать код TypeScript, предназначенный для непосредственного выполнения. По умолчанию TypeScript не изменяет спецификаторы модулей при импорте. Эта опция имеет несколько оговорок:
    -   Переписываются только относительные пути.
    -   Они переписываются «наивно» - без учета опций `baseUrl` и `paths` (которые выходят за рамки этой статьи).
    -   Пути, проложенные через свойства `"exports"` и `"imports"`, не выглядят как относительные пути и поэтому также не переписываются.

Если вы хотите использовать `tsc` для проверки типов (только), то обратите внимание на раздел, посвященный опции `noEmit`.

Для получения дополнительной информации о встроенной поддержке TypeScript в Node вы можете прочитать [мою статью](https://2ality.com/2025/01/nodejs-strip-type.html) в блоге.

### Импортирование JSON

```json
{
    "compilerOptions": {
        "resolveJsonModule": true
    }
}
```

Опция `resolveJsonModule` позволяет импортировать файлы JSON:

```ts
import config from './config.json' with {type: 'json'};
console.log(config.hello);
```

### Импорт других артефактов, не относящихся к TypeScript

Всякий раз, когда мы импортируем файл `basename.ext`, расширение которого `ext` TypeScript не знает, он ищет файл `basename.d.ext.ts`. Если он его не находит, то выдает ошибку. В документации по TypeScript есть [хороший пример](https://www.typescriptlang.org/tsconfig/#allowArbitraryExtensions) того, как может выглядеть такой файл.

Есть два способа предотвратить появление в TypeScript ошибок, связанных с неизвестным импортом.

Во-первых, мы можем использовать опцию `allowArbitraryExtensions`, чтобы предотвратить любое сообщение об ошибке в этом случае.

Во-вторых, мы можем создать объявление окружающего модуля со спецификатором wildcard - файл `.d.ts`, который должен находиться где-то среди файлов, о которых знает TypeScript. Следующий пример подавляет ошибки для всех импортов с расширением имени файла `.css`:

```ts
// ./src/globals.d.ts
declare module '*.css' {}
```

## Проверка типов

```json
{
    "compilerOptions": {
        "strict": true,
        "exactOptionalPropertyTypes": true,
        "noFallthroughCasesInSwitch": true,
        "noImplicitOverride": true,
        "noImplicitReturns": true,
        "noPropertyAccessFromIndexSignature": true,
        "noUncheckedIndexedAccess": true
    }
}
```

`strict` является обязательным, на мой взгляд. С остальными настройками вы должны сами решить, нужна ли вам дополнительная строгость для вашего кода. Вы можете начать с добавления всех из них и посмотреть, какие из них вызывают слишком много проблем на ваш вкус. В этом разделе мы будем игнорировать настройки, которые покрываются `strict` (например, `noImplicitAny`).

-   `noFallthroughCasesInSwitch`: Если `true`, то непустые случаи `switch` должны заканчиваться `break`, `return` или `throw`.
-   `noImplicitOverride`: Если `true`, то методы, переопределяющие методы суперкласса, должны иметь модификатор `override`.
-   `noImplicitReturns`: Если `true`, то «неявный возврат» (завершение функции или метода) разрешен только в том случае, если тип возврата - `void`.

### `exactOptionalPropertyTypes`

Если `true`, то `.colorTheme` может быть только опущен, а не установлен в значение `undefined`, как в следующем примере:

```ts
interface Settings {
    // Absent property means “system”
    colorTheme?: 'dark' | 'light';
}
const obj1: Settings = {}; // allowed
const obj2: Settings = { colorTheme: undefined }; // not allowed
```

### `noPropertyAccessFromIndexSignature`

Если `true`, то для типов, подобных следующему, мы не можем использовать точечную нотацию для неизвестных свойств, только для известных:

```ts
interface ObjectWithId {
    id: string;
    [key: string]: string;
}
declare const obj: ObjectWithId;

const value1 = obj.id; // allowed
const value2 = obj['unknownProp']; // allowed
const value3 = obj.unknownProp; // not allowed
```

### `noUncheckedIndexedAccess`

Если `true`, то тип неизвестного свойства является объединением `undefined` и типа индексной подписи:

```ts
interface ObjectWithId {
    id: string;
    [key: string]: string;
}
declare const obj: ObjectWithId;
expectType<string>(obj.id);
expectType<undefined | string>(obj.unknownProp);
```

### `noUncheckedIndexedAccess` и массивы

Опция `noUncheckedIndexedAccess` также влияет на работу с массивами:

```ts
const arr = ['a', 'b'];
const elem = arr[0];
expectType<undefined | string>(elem);
```

Если этот параметр равен `false`, то `elem` имеет тип `string`.

Одним из распространенных шаблонов для массивов является проверка длины перед обращением к элементу. Однако эта схема становится неудобной при использовании `noUncheckedIndexedAccess`:

```ts
function logElemAt0(arr: Array<string>) {
    if (0 < arr.length) {
        const elem = arr[0];
        expectType<undefined | string>(elem);
        console.log(elem);
    }
}
```

Поэтому разумнее использовать другой шаблон:

```ts
function logElemAt0(arr: Array<string>) {
    if (0 in arr) {
        const elem = arr[0];
        expectType<string>(elem);
        console.log(elem);
    }
}
```

Я размышляю над этим вариантом: С одной стороны, новый шаблон отражает, что массивы могут содержать дыры. С другой стороны, дыры встречаются редко, и, начиная с ES6, JavaScript делает вид, что это элементы, имеющие значение `undefined`:

```
> Array.from([,,,])
[ undefined, undefined, undefined ]
```

### Параметры проверки типов, которые имеют хорошие значения по умолчанию

По умолчанию следующие опции выдают предупреждения в редакторах, но мы также можем выбрать выдачу ошибок компилятора или игнорирование проблем:

-   `allowUnreachableCode`
-   `allowUnusedLabels`
-   `noUnusedLocals`
-   `noUnusedParameters`

## Совместимость: помощь внешним инструментам в компиляции TypeScript в JavaScript и декларации

```json
{
    "compilerOptions": {
        "verbatimModuleSyntax": true,
        "isolatedDeclarations": true
    }
}
```

Компилятор TypeScript выполняет три задачи:

-   Проверка типов
-   Эмиссия файлов JavaScript
-   Эмиссия файлов деклараций

В настоящее время стали популярны внешние инструменты, которые выполняют задачи #2 и #3 гораздо быстрее. У них есть два требования:

-   Выдача выходного файла не должна требовать поиска информации в файлах, импортированных входным файлом.
-   Он также не должен требовать семантического анализа; только синтаксический анализ.

Есть две настройки, которые обеспечивают статическое соблюдение этих ограничений - они вызывают ошибки компилятора, но не изменяют способ вывода JavaScript и деклараций:

-   `verbatimModuleSyntax` помогает при компиляции TypeScript в JavaScript.
-   `isolatedDeclarations` помогает компилировать TypeScript в декларации.

### `verbatimModuleSyntax`: компиляция TypeScript в JavaScript

Большинство частей файла TypeScript, не относящихся к JavaScript, легко обнаружить. Исключением являются импорты: Без (относительно простого) семантического анализа мы не знаем, является ли импорт типом (TypeScript) или значением (JavaScript).

Если активен `verbatimModuleSyntax`, мы вынуждены добавлять ключевое слово `type` к импортам, относящимся только к типам - например:

```ts
// Input
import {type SomeInterface, SomeClass} from './my-module.js';

// Output
import {SomeClass} from './my-module.js';
```

Обратите внимание, что класс - это и значение, и тип. В этом случае ключевое слово `type` не нужно, потому что эта часть синтаксиса может оставаться в обычном JavaScript.

Нам также нужно добавить `type`, если мы упоминаем тип в предложении экспорта:

```ts
interface MyInterface {}
export {type MyInterface};

// Alternative:
export interface MyInterface {}
```

#### `isolatedModules`

Активация `verbatimModuleSyntax` также активирует `isolatedModules`, поэтому нам нужна только первая настройка. Вторая не позволяет нам использовать некоторые более непонятные функции, которые также являются проблематичными.

Кроме того, эта опция позволяет esbuild компилировать TypeScript в JavaScript ([исходный текст](https://esbuild.github.io/content-types/#isolated-modules)).

#### `isolatedDeclarations`: компиляция TypeScript в декларации

`isolatedDeclarations` в основном заставляет нас добавлять аннотации возвращаемого типа к экспортируемым функциям и методам. Это означает, что внешним инструментам не придется выводить возвращаемые типы.

**Дополнительное чтение**: В примечаниях к выпуску TypeScript 5.5 есть [обширный раздел](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-5.html#isolated-declarations), посвященный изолированным объявлениям.

#### `noEmit`: использование tsc только для проверки типов

Иногда мы хотим использовать tsc только для проверки типов - например, если мы запускаем TypeScript напрямую или используем внешние инструменты для компиляции файлов TypeScript (в файлы JavaScript, файлы деклараций и т. д.):

```json
{
    "compilerOptions": {
        "noEmit": true
    }
}
```

-   `noEmit`: Если значение равно `true`, мы можем запустить `tsc`, и он будет проверять только код TypeScript, но не будет выдавать никаких файлов.

Нужно ли дополнительно удалять опции, связанные с выводом, зависит от того, какие из них используются вашими внешними инструментами.

## Импорт CommonJS из ESM

Одна из ключевых проблем связана с импортом модуля CommonJS из модуля ESM:

-   В ESM экспорт по умолчанию - это свойство `.default` объекта пространства имен модуля.
-   В CommonJS экспортом по умолчанию является объект модуля - например, многие модули CommonJS устанавливают `module.exports` в функцию.

Давайте рассмотрим две опции, которые помогут вам в этом.

### `allowSyntheticDefaultImports`: проверка типов импорта по умолчанию модулей CommonJS

Эта опция влияет только на проверку типов, но не на JavaScript-код, создаваемый TypeScript: Если она активна, импорт по умолчанию модуля CommonJS будет ссылаться на `module.exports` (а не на `module.exports.default`) - но только если нет `module.exports.default`.

Это отражает то, как Node.js обрабатывает импорт по умолчанию модулей CommonJS ([источник](https://nodejs.org/api/esm.html#interoperability-with-commonjs)): «При импорте модулей CommonJS объект `module.exports` предоставляется в качестве экспорта по умолчанию. Могут быть доступны именованные экспорты, предоставляемые статическим анализом в качестве удобства для лучшей совместимости с экосистемой.»

**Нужна ли нам эта опция?** Да, но она автоматически активируется, если `moduleResolution` имеет значение `"bundler"` или если `module` имеет значение `"NodeNext"` (что активирует `esModuleInterop`, который активирует `allowSyntheticDefaultImports`).

### `esModuleInterop`: улучшенная компиляция TypeScript в код CommonJS

Эта опция влияет на эмулируемый код CommonJS:

-   Если `false`:
    -   `import * as m from 'm'` компилируется в `const m = require('m')`.
    -   `import m from 'm'` (приблизительно) компилируется в `const m = require('m')`, а каждое обращение к `m` компилируется в `m.default`.
-   Если `true`:
    -   `import * as m from 'm'` присваивает `m` новый объект, который имеет те же свойства, что и `module.exports`, плюс свойство `.default`, которое ссылается на `module.exports`.
    -   `import m from 'm'` присваивает новый объект `m`, который имеет единственное свойство `.default`, ссылающееся на `module.exports`. Каждое обращение к `m` компилируется в `m.default`.
-   Если модуль CommonJS имеет маркерное свойство `.__esModule`, то он всегда импортируется так, как если бы `esModuleInterop` был выключен.

**Нужна ли нам эта опция?** Нет, так как мы создаем только модули ESM.

## Другие опции с хорошими значениями по умолчанию

Обычно мы можем игнорировать эти опции:

-   `moduleDetection`: Эта опция определяет, как TypeScript определяет, является ли файл скриптом или модулем. Обычно его можно не указывать, потому что его значение по умолчанию `"auto"` хорошо работает в большинстве случаев. Вам нужно явно установить значение `"force"`, только если в вашей кодовой базе есть модуль, у которого нет ни импорта, ни экспорта. Если `module` имеет значение `"NodeNext"` и в `package.json` есть `"type": "module"`, то даже эти файлы будут интерпретироваться как модули.
-   `skipLibCheck`: Если вы не делаете ничего сложного с файлами деклараций, вы можете проигнорировать эту опцию и просто использовать настройку по умолчанию. Согласно [обсуждению на GitHub](https://github.com/sindresorhus/tsconfig/issues/15), установка этого параметра в `true` (по умолчанию `false`) имеет много минусов.

## Параметры `package.json`, учитываемые TypeScript

TypeScript учитывает несколько свойств `package.json`:

-   `type`: Это важный параметр. Если вы компилируете в модули ESM, ваш `package.json` всегда должен содержать:

    ```json
    {
        "type": "module"
    }
    ```

-   `exports` определяет, какие файлы пакета будут публично видны, и ремапирует пути (чтобы то, что видят импортеры, отличалось от реальных внутренних путей). Все эти настройки могут применяться условно - в зависимости от среды импорта (браузеры, Node.js и т. д.). Более подробную информацию можно найти в моем блоге в статье [«TypeScript и нативный ESM на Node.js»](https://2ality.com/2021/06/typescript-esm-nodejs.html). Одна из интересных особенностей экспорта пакетов заключается в том, что мы можем ссылаться на свой собственный пакет через «голый» импорт, и при этом будут применяться правила экспорта пакетов. Это полезно для модульных тестов.

-   `imports` позволяет нам определять сокращения, такие как `#util`, для внутренних модулей и внешних пакетов. Более подробную информацию можно найти в главе [«Пакеты: Единицы JavaScript для распространения программного обеспечения»](https://exploringjs.com/nodejs-shell-scripting/ch_packages.html#package-imports) моей книги „Shell scripting with Node.js“.

## Visual Studio Code

Если вас не устраивают спецификаторы модулей для локального импорта в автоматически создаваемых импортах, то вы можете воспользоваться следующими двумя настройками:

```
javascript.preferences.importModuleSpecifierEnding
typescript.preferences.importModuleSpecifierEnding
```

По умолчанию VSC должен быть достаточно умным, чтобы добавлять расширения имен файлов там, где это необходимо.

## Резюме

После проведения исследований для этой статьи в блоге, это «база», которую я использую в настоящее время. В следующих подразделах объясняется, как адаптировать ее для различных случаев использования.

```json
{
    "include": ["src/**/*", "test/**/*"],
    "compilerOptions": {
        // Specify explicitly (don’t derive from source file paths):
        "rootDir": ".",
        "outDir": "dist",

        //===== Output: JavaScript =====
        "target": "ES2024",
        "module": "NodeNext", // sets up "moduleResolution"
        // Emptily imported modules must exist
        "noUncheckedSideEffectImports": true,
        //
        "sourceMap": true, // .js.map files

        //===== Interoperability: help external tools =====
        // Helps tools that compile .ts to .js by enforcing
        // `type` modifiers for type-only imports etc.
        "verbatimModuleSyntax": true,

        //===== Type checking =====
        "strict": true, // activates several useful options
        "exactOptionalPropertyTypes": true,
        "noFallthroughCasesInSwitch": true,
        "noImplicitOverride": true,
        "noImplicitReturns": true,
        "noPropertyAccessFromIndexSignature": true,
        "noUncheckedIndexedAccess": true,

        //===== Other options =====
        // Lets us import JSON files:
        "resolveJsonModule": true
    }
}
```

Примечания:

-   Для получения дополнительной информации о выборе хорошего `target` см. раздел, посвященный этой теме, ранее в этом блоге.
-   `verbatimModuleSyntax`: Мне нравятся ограничения, которые он накладывает на мой код: tsc работает без них, но они необходимы для многих инструментов, которые компилируют TypeScript в JavaScript.

### npm package (библиотеки и т. д.)

```json
{
    "compilerOptions": {
        // ···
        //===== Output: declarations =====
        "declaration": true, // .d.ts files
        // “Go to definition” jumps to TS source etc.:
        "declarationMap": true, // .d.ts.map files

        //===== Interoperability: help external tools =====
        // Helps tools that compile .ts to .d.ts by enforcing
        // return type annotations for exported functions, etc.
        "isolatedDeclarations": true,

        //===== Misc =====
        "lib": ["ES2024"] // don’t provide types for DOM
    }
}
```

Примечание: Если ваша библиотека использует DOM, вам следует убрать `"lib"`.

Я бы с удовольствием всегда использовал `isolatedDeclarations`, но TypeScript позволяет это делать только в том случае, если активна опция `declaration` или опция `composite`. Джейк Бейли [объясняет](https://github.com/microsoft/TypeScript/issues/58262#issuecomment-2597014286), почему это так:

На уровне реализации диагностика `isolatedDeclarations` - это дополнительная диагностика декларации, производимая трансформатором декларации, которую мы запускаем только тогда, когда `declaration` включена.

Теоретически можно реализовать так, что `isolatedDeclarations` включает эти проверки (диагностика на самом деле происходит от того, что мы запускаем трансформатор и затем отбрасываем полученный AST), но это изменение по сравнению с первоначальным дизайном.

### Приложение Node.js

```json
{
    "compilerOptions": {
        // ···
        //===== Misc =====
        "lib": ["ES2024"] // don’t provide types for DOM
    }
}
```

### Веб-приложение

`"module":"NodeNext"` должен хорошо работать и для бандлеров. Но вы можете перейти на более специфичный для бандлеров `"module":"preserve"`.

### Запуск TypeScript напрямую

```json
{
    "compilerOptions": {
        "allowImportingTsExtensions": true,
        // Only needed if compiling to JavaScript:
        "rewriteRelativeImportExtensions": true
    }
}
```

Дополнительную информацию см. в разделе «Выполнение TypeScript напрямую».

### Использование tsc только для проверки типов

```json
{
    "compilerOptions": {
        "noEmit": true
    }
}
```

Дополнительные сведения см. в разделе «Использование tsc только для проверки типов».

## Рекомендации по использованию `tsconfig.json` от других людей

-   Мэтт Покок [«Шпаргалка по TSConfig»](https://www.totaltypescript.com/tsconfig-cheat-sheet)
-   [`base.json`](https://github.com/voxpelli/tsconfig/blob/main/base.json) Пелле Вессмана
-   [`tsconfig.json`](https://github.com/sindresorhus/tsconfig/blob/main/tsconfig.json) Синдре Сорхуса

## Источники этой статьи в блоге

Некоторые источники уже были упомянуты ранее. Это дополнительные источники, которые я использовал:

-   [Официальная документация TSConfig](https://www.typescriptlang.org/tsconfig/)
-   Раздел [«Path Rewriting for Relative Paths»](https://devblogs.microsoft.com/typescript/announcing-typescript-5-7/#path-rewriting-for-relative-paths) в анонсе TypeScript 5.7.
-   В документации по esbuild содержатся [интересные наблюдения](https://esbuild.github.io/content-types/#typescript) о компиляции TypeScript.

<small>:material-information-outline: Источник &mdash; <https://2ality.com/2025/01/tsconfig-json.html></small>
