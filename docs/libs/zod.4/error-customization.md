---
description: В Zod ошибки валидации отображаются как экземпляры класса
---

# Настройка ошибок

В Zod ошибки валидации отображаются как экземпляры класса `z.core.$ZodError`.

> Класс `ZodError` в пакете `zod` является подклассом, который реализует некоторые дополнительные удобные методы.

Экземпляры `$ZodError` содержат массив `.issues`. Каждая проблема содержит удобочитаемое `message` и дополнительные структурированные метаданные о проблеме.

```ts
import * as z from 'zod';

const result = z.string().safeParse(12); // { success: false, error: ZodError }
result.error.issues;
// [
//   {
//     expected: 'string',
//     code: 'invalid_type',
//     path: [],
//     message: 'Invalid input: expected string, received number'
//   }
// ]
```

Каждый выпуск содержит свойство `message` с понятным для человека сообщением об ошибке. Сообщения об ошибках можно настраивать различными способами.

## Параметр `error`

Практически каждый API Zod принимает необязательное сообщение об ошибке.

```ts
z.string('Not a string!');
```

Эта настраиваемая ошибка будет отображаться как свойство `message` для всех проблем валидации, возникающих в этой схеме.

```ts
z.string('Not a string!').parse(12);
// ❌ throws ZodError {
//   issues: [
//     {
//       expected: 'string',
//       code: 'invalid_type',
//       path: [],
//       message: 'Not a string!'   <-- 👀 custom error message
//     }
//   ]
// }
```

Все функции `z` и методы схемы принимают пользовательские ошибки.

```ts
z.string('Bad!');
z.string().min(5, 'Too short!');
z.uuid('Bad UUID!');
z.iso.date('Bad date!');
z.array(z.string(), 'Not an array!');
z.array(z.string()).min(5, 'Too few items!');
z.set(z.string(), 'Bad set!');
```

Если хотите, вы можете вместо этого передать объект params с параметром `error`.

```ts
z.string({ error: 'Bad!' });
z.string().min(5, { error: 'Too short!' });
z.uuid({ error: 'Bad UUID!' });
z.iso.date({ error: 'Bad date!' });
z.array(z.string(), { error: 'Bad array!' });
z.array(z.string()).min(5, { error: 'Too few items!' });
z.set(z.string(), { error: 'Bad set!' });
```

Параметр `error` опционально принимает функцию. Функция настройки ошибок в терминологии Zod называется **картой ошибок**. Карта ошибок запускается во время анализа, если происходит ошибка проверки.

```ts
z.string({
    error: () => `[${Date.now()}]: Validation failure.`,
});
```

!!!note ""

    В Zod v3 были отдельные параметры для `message` (строка) и `errorMap` (функция). В Zod 4 они были объединены в `error`.

Карта ошибок получает объект контекста, который можно использовать для настройки сообщения об ошибке в зависимости от проблемы проверки.

```ts
z.string({
    error: (iss) =>
        iss.input === undefined
            ? 'Field is required.'
            : 'Invalid input.',
});
```

В сложных случаях объект `iss` предоставляет дополнительную информацию, которую можно использовать для настройки ошибки.

```ts
z.string({
    error: (iss) => {
        iss.code; // the issue code
        iss.input; // the input data
        iss.inst; // the schema/check that originated this issue
        iss.path; // the path of the error
    },
});
```

В зависимости от используемого API могут быть доступны дополнительные свойства. Воспользуйтесь автозаполнением TypeScript, чтобы ознакомиться с доступными свойствами.

```ts
z.string().min(5, {
    error: (iss) => {
        // ...the same as above
        iss.minimum; // the minimum value
        iss.inclusive; // whether the minimum is inclusive
        return `Password must have ${iss.minimum} characters or more`;
    },
});
```

Возвращайте `undefined`, чтобы избежать настройки сообщения об ошибке и вернуться к сообщению по умолчанию. (Точнее говоря, Zod передаст управление следующей карте ошибок в [цепочке приоритетов](#error-precedence).) Это полезно для выборочной настройки определенных сообщений об ошибках, но не других.

```ts
z.int64({
    error: (issue) => {
        // override too_big error message
        if (issue.code === 'too_big') {
            return {
                message: `Value must be <${issue.maximum}`,
            };
        }

        //  defer to default
        return undefined;
    },
});
```

## Настройка ошибок для каждого разбора

Чтобы настроить ошибки для каждого разбора, передайте карту ошибок в метод разбора:

```ts
const schema = z.string();

schema.parse(12, {
    error: (iss) => 'per-parse custom error',
});
```

Это имеет _более низкий приоритет_, чем любые настраиваемые сообщения на уровне схемы.

```ts
const schema = z.string({ error: 'highest priority' });
const result = schema.safeParse(12, {
    error: (iss) => 'lower priority',
});

result.error.issues;
// [{ message: "highest priority", ... }]
```

Объект `iss` представляет собой [дискриминированное объединение](https://www.typescriptlang.org/docs/handbook/2/narrowing.html#discriminated-unions) всех возможных типов проблем. Для их различения используйте свойство `code`.

> Подробное описание всех кодов проблем Zod см. в документации [`zod/v4/core`](https://zod.dev/packages/core#issue-types).

```ts
const result = schema.safeParse(12, {
    error: (iss) => {
        if (iss.code === 'invalid_type') {
            return `invalid type, expected ${iss.expected}`;
        }
        if (iss.code === 'too_small') {
            return `minimum is ${iss.minimum}`;
        }
        // ...
    },
});
```

### Включение ввода в задачи

По умолчанию Zod не включает вводные данные в задачи. Это сделано для предотвращения непреднамеренной регистрации потенциально конфиденциальных вводных данных. Чтобы включить вводные данные в каждую задачу, используйте флаг `reportInput`:

```ts
z.string().parse(12, {
    reportInput: true,
});

// ZodError: [
//   {
//     "expected": "string",
//     "code": "invalid_type",
//     "input": 12, // 👀
//     "path": [],
//     "message": "Invalid input: expected string, received number"
//   }
// ]
```

## Настройка глобальных ошибок

Чтобы указать глобальную карту ошибок, используйте `z.config()` для установки параметра конфигурации Zod `customError`:

```ts
z.config({
    customError: (iss) => {
        return 'globally modified error';
    },
});
```

Глобальные сообщения об ошибках имеют _более низкий приоритет_, чем сообщения об ошибках на уровне схемы или при разборе.

Объект `iss` представляет собой [дискриминированное объединение](https://www.typescriptlang.org/docs/handbook/2/narrowing.html#discriminated-unions) всех возможных типов проблем. Для их различения используйте свойство `code`.

> Подробное описание всех кодов проблем Zod см. в документации [`zod/v4/core`](https://zod.dev/packages/core#issue-types).

```ts
const result = schema.safeParse(12, {
    error: (iss) => {
        if (iss.code === 'invalid_type') {
            return `invalid type, expected ${iss.expected}`;
        }
        if (iss.code === 'too_small') {
            return `minimum is ${iss.minimum}`;
        }
        // ...
    },
});
```

## Интернационализация

Для поддержки интернационализации сообщений об ошибках Zod предоставляет несколько встроенных **локалей**. Они экспортируются из пакета `zod/v4/core`.

!!!note ""

    Обычная библиотека `zod` автоматически загружает локаль `en`. Zod Mini по умолчанию не загружает никаких локалей; вместо этого все сообщения об ошибках по умолчанию имеют вид `Invalid input`.

```ts
import * as z from 'zod';
import { en } from 'zod/locales';

z.config(en());
```

Чтобы загружать локаль в режиме отложенной загрузки, рассмотрите возможность использования динамического импорта:

```ts
import * as z from 'zod';

async function loadLocale(locale: string) {
    const { default: locale } = await import(
        `zod/v4/locales/${locale}.js`
    );
    z.config(locale());
}

await loadLocale('fr');
```

Для удобства все локализации экспортируются как `z.locales` из `"zod"`. В некоторых пакетах это может быть невозможно.

```ts
import * as z from 'zod';

z.config(z.locales.en());
```

### Локальные настройки

Доступны следующие локальные настройки:

-   `ar` — арабский
-   `az` — азербайджанский
-   `be` — белорусский
-   `ca` — каталонский
-   `cs` — чешский
-   `de` — немецкий
-   `en` — английский
-   `eo` — эсперанто
-   `es` — испанский
-   `fa` — фарси
-   `fi` — финский
-   `fr` — французский
-   `frCA` — канадский французский
-   `he` — иврит
-   `hu` — венгерский
-   `id` — индонезийский
-   `it` — итальянский
-   `ja` — японский
-   `kh` — кхмерский
-   `ko` — корейский
-   `mk` — македонский
-   `ms` — малайский
-   `nl` — нидерландский
-   `no` — норвежский
-   `ota` — тюркский
-   `ps` — пушту
-   `pl` — польский
-   `pt` — португальский
-   `ru` — русский
-   `sl` — словенский
-   `sv` — шведский
-   `ta` — тамильский
-   `th` — тайский
-   `tr` — турецкий
-   `ua` — украинский
-   `ur` — урду
-   `vi` — вьетнамский
-   `zhCN` — упрощенный китайский
-   `zhTW` — традиционный китайский

## Приоритет ошибок {#error-precedence}

Ниже приводится краткая справка по определению приоритета ошибок: если определено несколько настроек ошибок, какая из них имеет приоритет? От _наивысшего к наименьшему_ приоритету:

1.  **Ошибка на уровне схемы** — любое сообщение об ошибке, «жестко запрограммированное» в определении схемы.

    ```ts
    z.string('Not a string!');
    ```

2.  **Ошибка при разборе** — пользовательская карта ошибок, передаваемая в метод `.parse()`.

    ```ts
    z.string().parse(12, {
        error: (iss) => 'My custom error',
    });
    ```

3.  **Глобальная карта ошибок** — настраиваемая карта ошибок, передаваемая в `z.config()`.

    ```ts
    z.config({
        customError: (iss) => 'My custom error',
    });
    ```

4.  **Карта локальных ошибок** — настраиваемая карта ошибок, передаваемая в `z.config()`.

    ```ts
    z.config(z.locales.en());
    ```
