---
description: Часто бывает полезно связать схему с некоторыми дополнительными метаданными для документирования, генерации кода, структурированных выводов ИИ, проверки форм и других целей
---

# Метаданные и реестры

Часто бывает полезно связать схему с некоторыми дополнительными метаданными для документирования, генерации кода, структурированных выводов ИИ, проверки форм и других целей.

## Реестры {#registries}

Метаданные в Zod обрабатываются с помощью реестров. Реестры — это наборы схем, каждая из которых связана с некоторыми строго типизированными метаданными. Чтобы создать простой реестр:

```ts
import * as z from 'zod';

const myRegistry = z.registry<{ description: string }>();
```

Чтобы зарегистрировать, найти и удалить схемы из этого реестра:

```ts
const mySchema = z.string();

myRegistry.add(mySchema, { description: 'A cool schema!' });
myRegistry.has(mySchema); // => true
myRegistry.get(mySchema); // => { description: "A cool schema!" }
myRegistry.remove(mySchema);
myRegistry.clear(); // wipe registry
```

TypeScript требует, чтобы метаданные для каждой схемы соответствовали **типу метаданных** реестра.

```ts
myRegistry.add(mySchema, { description: 'A cool schema!' }); // ✅
myRegistry.add(mySchema, { description: 123 }); // ❌
```

> **Особый подход к `id`** — реестры Zod обрабатывают свойство `id` особым образом. Если несколько схем зарегистрированы с одинаковым значением `id`, будет сгенерирована ошибка `Error`. Это правило действует для всех реестров, включая глобальный реестр.

### `.register()`

> **Примечание** — Этот метод отличается тем, что он не возвращает новую схему, а возвращает исходную схему. Ни один другой метод Zod не делает этого! Это включает в себя `.meta()` и `.describe()` (описанные ниже), которые возвращают новый экземпляр.

Схемы предоставляют метод `.register()` для более удобного добавления в реестр.

```ts
const mySchema = z.string();

mySchema.register(myRegistry, {
    description: 'A cool schema!',
});
// => mySchema
```

Это позволяет вам определять метаданные «встроенными» в ваших схемах.

```ts
const mySchema = z.object({
    name: z.string().register(myRegistry, {
        description: "The user's name",
    }),
    age: z.number().register(myRegistry, {
        description: "The user's age",
    }),
});
```

!!!note ""

    Если реестр определен без типа метаданных, его можно использовать как общую «коллекцию», метаданные не требуются.

    ```ts
    const myRegistry = z.registry();

    myRegistry.add(z.string());
    myRegistry.add(z.number());
    ```

## Metadata

### `z.globalRegistry`

Для удобства Zod предоставляет глобальный реестр (`z.globalRegistry`), который можно использовать для хранения метаданных для генерации JSON Schema или других целей. Он принимает следующие метаданные:

```ts
export interface GlobalMeta {
    id?: string;
    title?: string;
    description?: string;
    example?: unknown;
    examples?:
        | unknown[] // array-style (JSON Schema)
        | Record<
              string,
              { value: unknown; [k: string]: unknown }
          >; // map-style (OpenAPI)
    deprecated?: boolean;
    [k: string]: unknown;
}
```

Чтобы зарегистрировать некоторые метаданные в `z.globalRegistry` для схемы:

```ts
import * as z from 'zod';

const emailSchema = z.email().register(z.globalRegistry, {
    id: 'email_address',
    title: 'Email address',
    description: 'Your email address',
    examples: ['first.last@example.com'],
});
```

### `.meta()`

Для более удобного подхода используйте метод `.meta()`, чтобы зарегистрировать схему в `z.globalRegistry`.

```ts
const emailSchema = z.email().meta({
    id: 'email_address',
    title: 'Email address',
    description: 'Please enter a valid email address',
});
```

Вызов `.meta()` без аргумента _извлекает_ метаданные для схемы.

```ts
emailSchema.meta();
// => { id: "email_address", title: "Email address", ... }
```

Метаданные связаны с _конкретным экземпляром схемы_. Это важно иметь в виду, особенно учитывая, что методы Zod являются неизменяемыми — они всегда возвращают новый экземпляр.

```ts
const A = z.string().meta({ description: 'A cool string' });
A.meta(); // => { hello: "true" }

const B = A.refine((_) => true);
B.meta(); // => undefined
```

### `.describe()`

!!!note ""

    Метод `.describe()` по-прежнему существует для обеспечения совместимости с Zod 3, но теперь рекомендуется использовать метод `.meta()`.

Метод `.describe()` является сокращением для регистрации схемы в `z.globalRegistry` только с полем `description`.

```ts
const emailSchema = z.email();
emailSchema.describe('An email address');

// equivalent to
emailSchema.meta({ description: 'An email address' });
```

## Пользовательские реестры

Вы уже видели простой пример пользовательского реестра:

```ts
import * as z from "zod";

const myRegistry = z.registry<{ description: string };>();
```

Давайте рассмотрим несколько более сложных шаблонов.

### Ссылки на выведенные типы

Часто бывает полезно, чтобы тип метаданных ссылался на _выведенный тип_ схемы. Например, вы можете захотеть, чтобы поле `examples` содержало примеры вывода схемы.

```ts
import * as z from 'zod';

type MyMeta = { examples: z.$output[] };
const myRegistry = z.registry<MyMeta>();

myRegistry.add(z.string(), {
    examples: ['hello', 'world'],
});
myRegistry.add(z.number(), { examples: [1, 2, 3] });
```

Специальный символ `z.$output` является ссылкой на тип вывода, выведенный из схемы (`z.infer<typeof schema>`). Аналогично, вы можете использовать `z.$input` для ссылки на тип ввода.

### Ограничение типов схем

Передайте второй общий параметр в `z.registry()`, чтобы ограничить типы схем, которые могут быть добавлены в реестр. Этот реестр принимает только строковые схемы.

```ts
import * as z from 'zod';

const myRegistry = z.registry<
    { description: string },
    z.ZodString
>();

myRegistry.add(z.string(), { description: 'A number' }); // ✅
myRegistry.add(z.number(), { description: 'A number' }); // ❌
//             ^ 'ZodNumber' is not assignable to parameter of type 'ZodString'
```
