---
description: На этой странице вы найдете основную информацию о создании схем, анализе данных и использовании выведенных типов
---

# Основные принципы использования

На этой странице вы найдете основные сведения о создании схем, анализе данных и использовании выведенных типов. Полную документацию по API схем Zod см. в разделе [Определение схем](./api.md).

## Определение схемы

Прежде чем приступить к каким-либо действиям, необходимо определить схему. В целях данного руководства мы будем использовать простую схему объекта.

```ts
import * as z from 'zod';

const Player = z.object({
    username: z.string(),
    xp: z.number(),
});
```

## Анализ данных

Для любой схемы Zod используйте `.parse` для проверки входных данных. Если они действительны, Zod возвращает строго типизированный _глубокий клон_ входных данных.

```ts
Player.parse({ username: 'billie', xp: 100 });
// => returns { username: "billie", xp: 100 }
```

!!!note ""

    Если ваша схема использует определенные асинхронные API, такие как `async` [уточнения](./api.md/#refine) или [трансформации](./api.md#transform), вам необходимо использовать метод `.parseAsync()`.

    ```ts
    await Player.parseAsync({ username: 'billie', xp: 100 });
    ```

## Обработка ошибок

При сбое валидации метод `.parse()` выдает экземпляр `ZodError` с подробной информацией о проблемах валидации.

```ts
try {
    Player.parse({ username: 42, xp: '100' });
} catch (error) {
    if (error instanceof z.ZodError) {
        error.issues;
        /*
		[
			{
				expected: 'string',
				code: 'invalid_type',
				path: [ 'username' ],
				message: 'Invalid input: expected string'
			},
			{
				expected: 'number',
				code: 'invalid_type',
				path: [ 'xp' ],
				message: 'Invalid input: expected number'
			}
		]
		*/
    }
}
```

Чтобы избежать использования блока `try/catch`, можно использовать метод `.safeParse()`, который возвращает простой объект результата, содержащий либо успешно проанализированные данные, либо `ZodError`. Тип результата является [дискриминированным объединением](https://www.typescriptlang.org/docs/handbook/2/narrowing.html#discriminated-unions), поэтому вы можете удобно обрабатывать оба случая.

```ts
const result = Player.safeParse({
    username: 42,
    xp: '100',
});
if (!result.success) {
    result.error; // ZodError instance
} else {
    result.data; // { username: string; xp: number }
}
```

!!!note ""

    Если ваша схема использует определенные асинхронные API, такие как `async` [уточнения](./api.md#refine) или [преобразования](./api.md#transform), вам необходимо использовать метод `.safeParseAsync()`.

    ```ts
    await schema.safeParseAsync('hello');
    ```

## Вывод типов

Zod выводит статический тип из определений вашей схемы. Вы можете извлечь этот тип с помощью утилиты `z.infer<>` и использовать его по своему усмотрению.

```ts
const Player = z.object({
    username: z.string(),
    xp: z.number(),
});

// extract the inferred type
type Player = z.infer<typeof Player>;

// use it in your code
const player: Player = { username: 'billie', xp: 100 };
```

В некоторых случаях типы ввода и вывода схемы могут различаться. Например, API `.transform()` может преобразовывать ввод из одного типа в другой. В таких случаях вы можете извлекать типы ввода и вывода независимо друг от друга:

```ts
const mySchema = z.string().transform((val) => val.length);

type MySchemaIn = z.input<typeof mySchema>;
// => string

type MySchemaOut = z.output<typeof mySchema>; // equivalent to z.infer<typeof mySchema>
// number
```

Теперь, когда мы рассмотрели основы, давайте перейдем к Schema API.
