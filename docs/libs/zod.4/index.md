---
description: Zod — это библиотека для проверки данных, основанная на TypeScript
---

# Zod

## Введение

**Zod** — это библиотека для проверки данных, основанная на TypeScript. С помощью Zod вы можете определять схемы, которые можно использовать для проверки данных, от простых строк до сложных вложенных объектов.

```ts
import * as z from 'zod';

const User = z.object({
    name: z.string(),
});

// some untrusted data...
const input = {
    /* stuff */
};

// the parsed result is validated and type safe!
const data = User.parse(input);

// so you can use it with confidence :)
console.log(data.name);
```

## Особенности

-   Полное отсутствие внешних зависимостей
-   Работает в Node.js и всех современных браузерах
-   Миниатюрный: 2 КБ основного пакета (сжатый gzip)
-   Неизменяемый API: методы возвращают новый экземпляр
-   Лаконичный интерфейс
-   Работает с TypeScript и простым JS
-   Встроенное преобразование JSON Schema
-   Обширная экосистема

## Установка

```sh
npm install zod       # npm
deno add npm:zod      # deno
yarn add zod          # yarn
bun add zod           # bun
pnpm add zod          # pnpm
```

Zod также публикует тестовую версию при каждом коммите. Чтобы установить тестовую версию:

```sh
npm install zod@canary       # npm
deno add npm:zod@canary      # deno
yarn add zod@canary          # yarn
bun add zod@canary           # bun
pnpm add zod@canary          # pnpm
```

## Требования

Zod тестируется на _TypeScript v5.5_ и более поздних версиях. Более старые версии могут работать, но официально не поддерживаются.

### `"strict"`

Вы должны включить режим `strict` в вашем `tsconfig.json`. Это лучшая практика для всех проектов TypeScript.

```json
// tsconfig.json
{
    // ...
    "compilerOptions": {
        // ...
        "strict": true
    }
}
```

### `"moduleResolution"`

Ваш `"moduleResolution"` должен быть установлен в одно из следующих значений. Устаревшие режимы `"node"` и `"classic"` не поддерживаются, так как они не поддерживают импорт подпутей.

-   `"node16"` (default if `"module"` is set to `"node16"`/`"node18"`)
-   `"nodenext"` (default if `"module"` is set to `"nodenext"`)
-   `"bundler"`
