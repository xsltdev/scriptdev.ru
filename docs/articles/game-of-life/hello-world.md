---
description: В этом разделе вы узнаете, как создать и запустить свою первую программу на Rust и WebAssembly - веб-страницу, оповещающую Hello, World!
---

# Hello, World!

В этом разделе вы узнаете, как создать и запустить свою первую программу на Rust и WebAssembly: веб-страницу, оповещающую "Hello, World!".

Перед началом работы убедитесь, что вы выполнили инструкции [setup instructions](setup.html).

## Клонирование шаблона проекта

Шаблон проекта поставляется предварительно сконфигурированным с разумными настройками по умолчанию, чтобы вы могли быстро собрать, интегрировать и упаковать свой код для Web.

Клонируйте шаблон проекта с помощью этой команды:

```text
cargo generate --git https://github.com/rustwasm/wasm-pack-template
```

Вам будет предложено ввести название нового проекта. Мы будем использовать **"wasm-game-of-life"**.

```text
wasm-game-of-life
```

## Что внутри

Заходите в новый проект `wasm-game-of-life`

```
cd wasm-game-of-life
```

и давайте посмотрим на его содержимое:

```text
wasm-game-of-life/
├── Cargo.toml
├── LICENSE_APACHE
├── LICENSE_MIT
├── README.md
└── src
    ├── lib.rs
    └── utils.rs
```

Давайте рассмотрим несколько таких файлов подробнее.

**`wasm-game-of-life/Cargo.toml`**

Файл `Cargo.toml` определяет зависимости и метаданные для `cargo`, менеджера пакетов и инструмента сборки Rust. Этот файл поставляется с предустановленной зависимостью `wasm-bindgen`, несколькими дополнительными зависимостями, которые мы рассмотрим позже, и правильно инициализированным `crate-type` для генерации библиотек `.wasm`.

**`wasm-game-of-life/src/lib.rs`**

Файл `src/lib.rs` является корнем Rust crate, который мы компилируем в WebAssembly. Он использует `wasm-bindgen` для взаимодействия с JavaScript. Он импортирует JavaScript-функцию `window.alert` и экспортирует Rust-функцию `greet`, которая выдает приветствие.

```rust
mod utils;

use wasm_bindgen::prelude::*;

// When the `wee_alloc` feature is enabled, use `wee_alloc` as the global
// allocator.
#[cfg(feature = "wee_alloc")]
#[global_allocator]
static ALLOC: wee_alloc::WeeAlloc = wee_alloc::WeeAlloc::INIT;

#[wasm_bindgen]
extern {
    fn alert(s: &str);
}

#[wasm_bindgen]
pub fn greet() {
    alert("Hello, wasm-game-of-life!");
}
```

**`wasm-game-of-life/src/utils.rs`**

Модуль `src/utils.rs` предоставляет общие утилиты для облегчения работы с Rust, скомпилированным в WebAssembly. Мы рассмотрим некоторые из этих утилит более подробно позже в учебнике, например, когда будем рассматривать [отладку нашего wasm-кода](debugging.html), но пока мы можем игнорировать этот файл.

## Сборка проекта

Мы используем `wasm-pack` для организации следующих шагов сборки:

-   Убедитесь, что у нас установлен Rust 1.30 или новее и цель `wasm32-unknown-unknown` через `rustup`,
-   Скомпилируйте наши исходные тексты Rust в бинарник WebAssembly `.wasm` с помощью `cargo`,
-   Используйте `wasm-bindgen` для генерации JavaScript API для использования нашей Rust-генерированной WebAssembly.

Чтобы сделать все это, запустите эту команду в каталоге проекта:

```
wasm-pack build
```

После завершения сборки мы можем найти ее артефакты в директории `pkg`, и она должна иметь следующее содержимое:

```
pkg/
├── package.json
├── README.md
├── wasm_game_of_life_bg.wasm
├── wasm_game_of_life.d.ts
└── wasm_game_of_life.js
```

Файл `README.md` скопирован из основного проекта, но все остальные - совершенно новые.

**`wasm-game-of-life/pkg/wasm_game_of_life_bg.wasm`**

Файл `.wasm` - это двоичный файл WebAssembly, который генерируется компилятором Rust из наших исходных текстов Rust. Он содержит скомпилированные в wasm версии всех наших функций и данных Rust. Например, в нем есть экспортируемая функция "greet".

**`wasm-game-of-life/pkg/wasm_game_of_life.js`**

Файл `.js` генерируется `wasm-bindgen` и содержит JavaScript-клеи для импорта DOM и JavaScript-функций в Rust и раскрытия красивого API для функций WebAssembly в JavaScript. Например, есть функция JavaScript `greet`, которая оборачивает функцию `greet`, экспортированную из модуля WebAssembly. Сейчас этот клей не делает многого, но когда мы начнем передавать более интересные значения туда и обратно между wasm и JavaScript, он поможет переправить эти значения через границу.

```js
import * as wasm from './wasm_game_of_life_bg';

// ...

export function greet() {
    return wasm.greet();
}
```

**`wasm-game-of-life/pkg/wasm_game_of_life.d.ts`**

Файл `.d.ts` содержит объявления типов [TypeScript][typescript] для JavaScript-клея. Если вы используете TypeScript, вы можете проверять типы ваших вызовов функций WebAssembly, а ваша IDE может предоставить автозавершение и предложения! Если вы не используете TypeScript, можете смело игнорировать этот файл.

```typescript
export function greet(): void;
```

[typescript]: http://www.typescriptlang.org/

**`wasm-game-of-life/pkg/package.json`**

[Файл `package.json` содержит метаданные о сгенерированном пакете JavaScript и WebAssembly.][package.json] Он используется npm и JavaScript bundlers для определения зависимостей между пакетами, имен пакетов, версий и множества других вещей. Он помогает нам интегрироваться с инструментами JavaScript и позволяет опубликовать наш пакет в npm.

```json
{
    "name": "wasm-game-of-life",
    "collaborators": ["Your Name <your.email@example.com>"],
    "description": null,
    "version": "0.1.0",
    "license": null,
    "repository": null,
    "files": [
        "wasm_game_of_life_bg.wasm",
        "wasm_game_of_life.d.ts"
    ],
    "main": "wasm_game_of_life.js",
    "types": "wasm_game_of_life.d.ts"
}
```

[package.json]: https://docs.npmjs.com/files/package.json

## Вставка в веб-страницу

Чтобы взять наш пакет `wasm-game-of-life` и использовать его в веб-странице, мы используем [шаблон проекта JavaScript `create-wasm-app`][create-wasm-app].

[create-wasm-app]: https://github.com/rustwasm/create-wasm-app

Выполните эту команду в директории `wasm-game-of-life`:

```
npm init wasm-app www
```

Вот что содержит наш новый подкаталог `wasm-game-of-life/www`:

```
wasm-game-of-life/www/
├── bootstrap.js
├── index.html
├── index.js
├── LICENSE-APACHE
├── LICENSE-MIT
├── package.json
├── README.md
└── webpack.config.js
```

Давайте еще раз внимательно рассмотрим некоторые из этих файлов.

**`wasm-game-of-life/www/package.json`**

Этот `package.json` поставляется с предустановленными зависимостями `webpack` и `webpack-dev-server`, а также зависимостью от `hello-wasm-pack`, которая представляет собой версию начального пакета `wasm-pack-template`, опубликованного на npm.

**`wasm-game-of-life/www/webpack.config.js`**

Этот файл настраивает webpack и его локальный сервер разработки. Он поставляется уже настроенным, и вам не придется вносить в него какие-либо изменения, чтобы заставить webpack и его локальный сервер разработки работать.

**`wasm-game-of-life/www/index.html`**

Это корневой HTML-файл для веб-страницы. Он не делает ничего особенного, кроме загрузки `bootstrap.js`, который является очень тонкой оберткой вокруг `index.js`.

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8" />
        <title>Hello wasm-pack!</title>
    </head>
    <body>
        <script src="./bootstrap.js"></script>
    </body>
</html>
```

**`wasm-game-of-life/www/index.js`**

Сайт `index.js` - это основная точка входа для JavaScript нашей веб-страницы. Он импортирует пакет npm `hello-wasm-pack`, который содержит скомпилированный WebAssembly и JavaScript-клея по умолчанию `wasm-pack-template`, а затем вызывает функцию `hello-wasm-pack` `greet`.

```js
import * as wasm from 'hello-wasm-pack';

wasm.greet();
```

### Установите зависимости

Сначала убедитесь, что локальный сервер разработки и его зависимости установлены, запустив `npm install` в подкаталоге `wasm-game-of-life/www`:

```text
npm install
```

Эту команду нужно выполнить только один раз, и она установит пакет JavaScript `webpack` и его сервер разработки.

> Обратите внимание, что `webpack` не является обязательным для работы с Rust и WebAssembly, это просто бандлер и сервер разработки, который мы выбрали для удобства. Parcel и Rollup также должны поддерживать импорт WebAssembly как модулей ECMAScript. Вы также можете использовать Rust и WebAssembly [без бандлера](https://rustwasm.github.io/docs/wasm-bindgen/examples/without-a-bundler.html), если хотите!

### Использование нашего локального пакета `wasm-game-of-life` в `www`

Вместо того чтобы использовать пакет `hello-wasm-pack` из npm, мы хотим использовать наш локальный пакет `wasm-game-of-life`. Это позволит нам постепенно развивать нашу программу Game of Life.

Откройте `wasm-game-of-life/www/package.json` и рядом с `"devDependencies"` добавьте поле `"dependencies"`, включая `"wasm-game-of-life": "file:../pkg"`:

```js
{
  // ...
  "dependencies": {                     // Add this three lines block!
    "wasm-game-of-life": "file:../pkg"
  },
  "devDependencies": {
    //...
  }
}
```

Затем измените `wasm-game-of-life/www/index.js`, чтобы импортировать `wasm-game-of-life` вместо пакета `hello-wasm-pack`:

```js
import * as wasm from 'wasm-game-of-life';

wasm.greet();
```

Поскольку мы объявили новую зависимость, нам нужно ее установить:

```text
npm install
```

Наша веб-страница теперь готова к местному обслуживанию!

## Обслуживание локально

Далее откройте новый терминал для сервера разработки. Запуск сервера в новом терминале позволяет нам оставить его работать в фоновом режиме и не мешает нам выполнять другие команды в это время. В новом терминале выполните эту команду из директории `wasm-game-of-life/www`:

```
npm run start
```

Перейдите в веб-браузере по адресу [http://localhost:8080/](http://localhost:8080/), и вас должно встретить сообщение с предупреждением:

![Скриншот оповещения веб-страницы "Привет, wasm-game-of-life!"](hello-world.png)

Каждый раз, когда вы вносите изменения и хотите, чтобы они отразились на [http://localhost:8080/](http://localhost:8080/), просто повторно выполните команду `wasm-pack build` в директории `wasm-game-of-life`.

## Упражнения

Измените функцию `greet` в `wasm-game-of-life/src/lib.rs`, чтобы она принимала параметр `name: &str`, который настраивает сообщение с предупреждением, и передайте свое имя в функцию `greet` из каталога `wasm-game-of-life/www/index.js`. Пересоберите бинарный файл `.wasm` с помощью `wasm-pack build`, затем обновите [http://localhost:8080/](http://localhost:8080/) в веб-браузере, и вы увидите настроенное приветствие!

!!!note "Answer"

    New version of the `greet` function in `wasm-game-of-life/src/lib.rs`:

    ```rust
    #[wasm_bindgen]
    pub fn greet(name: &str) {
    	alert(&format!("Hello, {}!", name));
    }
    ```

    New invocation of `greet` in `wasm-game-of-life/www/index.js`:

    ```js
    wasm.greet('Your Name');
    ```

<small>:material-information-outline: Источник &mdash; <https://rustwasm.github.io/docs/book/game-of-life/hello-world.html></small>
