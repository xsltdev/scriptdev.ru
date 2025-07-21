---
description: В этом разделе описывается настройка цепочки инструментов для компиляции программ на языке Rust в WebAssembly и их интеграции в JavaScript
---

# Настройка

В этом разделе описывается настройка цепочки инструментов для компиляции программ на языке Rust в WebAssembly и их интеграции в JavaScript.

## Инструментарий Rust

Вам понадобится стандартный набор инструментов Rust, включая `rustup`, `rustc` и `cargo`.

[Следуйте этим инструкциям, чтобы установить инструментарий Rust][rust-install].

Опыт работы с Rust и WebAssembly оседлал поезда с релизом Rust и перешел на стабильный уровень! Это означает, что нам не требуются флаги экспериментальных функций. Тем не менее, нам требуется Rust 1.30 или новее.

## `wasm-pack`

`wasm-pack` - это ваш универсальный магазин для создания, тестирования и публикации WebAssembly, сгенерированного на Rust.

[Получить `wasm-pack` можно здесь!][wasm-pack-install]

## `cargo-generate`

[`cargo-generate` поможет вам быстро начать работу над новым проектом Rust, используя уже существующий git-репозиторий в качестве шаблона][cargo-generate].

Установите `cargo-generate` с помощью этой команды:

```
cargo install cargo-generate
```

## `npm`

`npm` - это менеджер пакетов для JavaScript. Мы будем использовать его для установки и запуска JavaScript-бандлера и сервера разработки. В конце урока мы опубликуем наш скомпилированный `.wasm` в реестре `npm`.

[Следуйте этим инструкциям для установки `npm`][npm-install].

Если у вас уже установлен `npm`, убедитесь в его актуальности с помощью этой команды:

```
npm install npm@latest -g
```

[rust-install]: https://www.rust-lang.org/tools/install
[npm-install]: https://www.npmjs.com/get-npm
[wasm-pack]: https://github.com/rustwasm/wasm-pack
[cargo-generate]: https://github.com/ashleygwilliams/cargo-generate
[wasm-pack-install]: https://rustwasm.github.io/wasm-pack/installer/

<small>:material-information-outline: Источник &mdash; <https://rustwasm.github.io/docs/book/game-of-life/setup.html></small>
