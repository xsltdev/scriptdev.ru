---
description: Zod 4 представляет новую функцию - встроенное преобразование JSON Schema
---

# JSON схема

!!!note ""

    Zod 4 представляет новую функцию: встроенное преобразование [JSON Schema](https://json-schema.org/). JSON Schema — это стандарт для описания структуры JSON (с помощью JSON). Он широко используется в определениях [OpenAPI](https://www.openapis.org/) и определении [структурированных выводов](https://platform.openai.com/docs/guides/structured-outputs?api-mode=chat) для ИИ.

Чтобы преобразовать схему Zod в схему JSON, используйте функцию `z.toJSONSchema()`.

```ts
import * as z from 'zod';

const schema = z.object({
    name: z.string(),
    age: z.number(),
});

z.toJSONSchema(schema);
// => {
//   type: 'object',
//   properties: { name: { type: 'string' }, age: { type: 'number' } },
//   required: [ 'name', 'age' ],
//   additionalProperties: false,
// }
```

Все схемы и проверки преобразуются в наиболее близкий эквивалент JSON Schema. Некоторые типы не имеют аналогов и не могут быть адекватно представлены. Дополнительные сведения об обработке таких случаев см. в разделе [`unrepresentable`](#unrepresentable) ниже.

```ts
z.bigint(); // ❌
z.int64(); // ❌
z.symbol(); // ❌
z.void(); // ❌
z.date(); // ❌
z.map(); // ❌
z.set(); // ❌
z.transform(); // ❌
z.nan(); // ❌
z.custom(); // ❌
```

## Форматы строк

Zod преобразует следующие типы схем в эквивалентный формат JSON Schema `format`:

```ts
// Supported via `format`
z.email(); // => { type: "string", format: "email" }
z.iso.datetime(); // => { type: "string", format: "date-time" }
z.iso.date(); // => { type: "string", format: "date" }
z.iso.time(); // => { type: "string", format: "time" }
z.iso.duration(); // => { type: "string", format: "duration" }
z.ipv4(); // => { type: "string", format: "ipv4" }
z.ipv6(); // => { type: "string", format: "ipv6" }
z.uuid(); // => { type: "string", format: "uuid" }
z.guid(); // => { type: "string", format: "uuid" }
z.url(); // => { type: "string", format: "uri" }
```

Эти схемы поддерживаются через `contentEncoding`:

```ts
z.base64(); // => { type: "string", contentEncoding: "base64" }
```

Все другие форматы строк поддерживаются через `pattern`:

```ts
z.base64url();
z.cuid();
z.regex();
z.emoji();
z.nanoid();
z.cuid2();
z.ulid();
z.cidrv4();
z.cidrv6();
```

## Числовые типы

Zod преобразует следующие числовые типы в JSON Schema:

```ts
// number
z.number(); // => { type: "number" }
z.float32(); // => { type: "number", exclusiveMinimum: ..., exclusiveMaximum: ... }
z.float64(); // => { type: "number", exclusiveMinimum: ..., exclusiveMaximum: ... }

// integer
z.int(); // => { type: "integer" }
z.int32(); // => { type: "integer", exclusiveMinimum: ..., exclusiveMaximum: ... }
```

## Схемы объектов

По умолчанию схемы `z.object()` содержат `additionalProperties: "false"`. Это точно отражает поведение Zod по умолчанию, поскольку простая схема `z.object()` удаляет дополнительные свойства.

```ts
import * as z from 'zod';

const schema = z.object({
    name: z.string(),
    age: z.number(),
});

z.toJSONSchema(schema);
// => {
//   type: 'object',
//   properties: { name: { type: 'string' }, age: { type: 'number' } },
//   required: [ 'name', 'age' ],
//   additionalProperties: false,
// }
```

При преобразовании в JSON Schema в режиме `"input"`, `additionalProperties` не устанавливается. Дополнительную информацию см. в [документации по io](#io).

```ts
import * as z from 'zod';

const schema = z.object({
    name: z.string(),
    age: z.number(),
});

z.toJSONSchema(schema, { io: 'input' });
// => {
//   type: 'object',
//   properties: { name: { type: 'string' }, age: { type: 'number' } },
//   required: [ 'name', 'age' ],
// }
```

Напротив:

-   `z.looseObject()` никогда не будет устанавливать `additionalProperties: false`
-   `z.strictObject()` всегда будет устанавливать `additionalProperties: false`

## Схемы файлов

Zod преобразует `z.file()` в следующую схему, совместимую с OpenAPI:

```ts
z.file();
// => { type: "string", format: "binary", contentEncoding: "binary" }
```

Также представлены проверки размера и MIME:

```ts
z.file()
    .min(1)
    .max(1024 * 1024)
    .mime('image/png');
// => {
//   type: "string",
//   format: "binary",
//   contentEncoding: "binary",
//   contentMediaType: "image/png",
//   minLength: 1,
//   maxLength: 1048576,
// }
```

## Nullability

Zod преобразует как `undefined`/`null`, так и `{ type: "null" }` в JSON Schema.

```ts
z.null();
// => { type: "null" }

z.undefined();
// => { type: "null" }
```

Аналогично, `nullable` представляется с помощью объединения с `null`:

```ts
z.nullable(z.string());
// => { oneOf: [{ type: "string" }, { type: "null" }] }
```

Необязательные схемы представлены в том виде, в котором они есть, хотя они украшены аннотацией `optional`.

```ts
z.optional(z.string());
// => { type: "string" }
```

## Конфигурация

Второй аргумент можно использовать для настройки логики преобразования.

```ts
z.toJSONSchema(schema, {
    // ...params
});
```

Ниже приводится краткая справка по каждому поддерживаемому параметру. Каждый из них более подробно описан ниже.

```ts
interface ToJSONSchemaParams {
    /** The JSON Schema version to target.
     * - `"draft-2020-12"` — Default. JSON Schema Draft 2020-12
     * - `"draft-7"` — JSON Schema Draft 7 */
    target?: 'draft-7' | 'draft-2020-12';

    /** A registry used to look up metadata for each schema.
     * Any schema with an `id` property will be extracted as a $def. */
    metadata?: $ZodRegistry<Record<string, any>>;

    /** How to handle unrepresentable types.
     * - `"throw"` — Default. Unrepresentable types throw an error
     * - `"any"` — Unrepresentable types become `{}` */
    unrepresentable?: 'throw' | 'any';

    /** How to handle cycles.
     * - `"ref"` — Default. Cycles will be broken using $defs
     * - `"throw"` — Cycles will throw an error if encountered */
    cycles?: 'ref' | 'throw';

    /* How to handle reused schemas.
     * - `"inline"` — Default. Reused schemas will be inlined
     * - `"ref"` — Reused schemas will be extracted as $defs */
    reused?: 'ref' | 'inline';

    /** A function used to convert `id` values to URIs to be used in *external* $refs.
     *
     * Default is `(id) => id`.
     */
    uri?: (id: string) => string;
}
```

### `target`

Чтобы установить целевую версию JSON Schema, используйте параметр `target`. По умолчанию Zod будет ориентироваться на Draft 2020-12.

```ts
z.toJSONSchema(schema, { target: 'draft-7' });
z.toJSONSchema(schema, { target: 'draft-2020-12' });
```

### `metadata`

> Если вы еще не сделали этого, прочтите страницу [Метаданные и реестры](./metadata.md), чтобы узнать о хранении метаданных в Zod.

В Zod метаданные хранятся в реестрах. Zod экспортирует глобальный реестр `z.globalRegistry`, который можно использовать для хранения общих полей метаданных, таких как `id`, `title`, `description` и `examples`.

```ts
import * as z from 'zod';

// `.meta()` is a convenience method for registering a schema in `z.globalRegistry`
const emailSchema = z.string().meta({
    title: 'Email address',
    description: 'Your email address',
});

z.toJSONSchema(emailSchema);
// => { type: "string", title: "Email address", description: "Your email address", ... }
```

Все поля метаданных копируются в результирующую схему JSON.

```ts
const schema = z.string().meta({
    whatever: 1234,
});

z.toJSONSchema(schema);
// => { type: "string", whatever: 1234 }
```

### `unrepresentable` {#unrepresentable}

Следующие API не могут быть представлены в JSON Schema. По умолчанию Zod выдаст ошибку, если они будут обнаружены. Попытка преобразования в JSON Schema нецелесообразна; вам следует изменить свои схемы, поскольку они не имеют эквивалента в JSON. При обнаружении любого из них будет выведена ошибка.

```ts
z.bigint(); // ❌
z.int64(); // ❌
z.symbol(); // ❌
z.void(); // ❌
z.date(); // ❌
z.map(); // ❌
z.set(); // ❌
z.transform(); // ❌
z.nan(); // ❌
z.custom(); // ❌
```

По умолчанию Zod выдаст ошибку, если встретит любое из этих событий.

```ts
z.toJSONSchema(z.bigint());
// => throws Error
```

Вы можете изменить это поведение, установив для параметра `unrepresentable` значение `"any"`. Это приведет к преобразованию всех непредставимых типов в `{}` (эквивалент `unknown` в JSON Schema).

```ts
z.toJSONSchema(z.bigint(), { unrepresentable: 'any' });
// => {}
```

### `cycles`

Как обрабатывать циклы. Если при прохождении схемы функцией `z.toJSONSchema()` встречается цикл, он будет представлен с помощью `$ref`.

```ts
const User = z.object({
    name: z.string(),
    get friend() {
        return User;
    },
});

z.toJSONSchema(User);
// => {
//   type: 'object',
//   properties: { name: { type: 'string' }, friend: { '$ref': '#' } },
//   required: [ 'name', 'friend' ],
//   additionalProperties: false,
// }
```

Если же вы хотите выдать ошибку, установите для параметра `cycles` значение `"throw"`.

```ts
z.toJSONSchema(User, { cycles: 'throw' });
// => throws Error
```

### `reused`

Как обрабатывать схемы, которые повторяются несколько раз в одной схеме. По умолчанию Zod будет вставлять эти схемы в текст.

```ts
const name = z.string();
const User = z.object({
    firstName: name,
    lastName: name,
});

z.toJSONSchema(User);
// => {
//   type: 'object',
//   properties: {
//     firstName: { type: 'string' },
//     lastName: { type: 'string' }
//   },
//   required: [ 'firstName', 'lastName' ],
//   additionalProperties: false,
// }
```

Вместо этого вы можете установить опцию `reused` в значение `"ref"`, чтобы извлечь эти схемы в `$defs`.

```ts
z.toJSONSchema(User, { reused: 'ref' });
// => {
//   type: 'object',
//   properties: {
//     firstName: { '$ref': '#/$defs/__schema0' },
//     lastName: { '$ref': '#/$defs/__schema0' }
//   },
//   required: [ 'firstName', 'lastName' ],
//   additionalProperties: false,
//   '$defs': { __schema0: { type: 'string' } }
// }
```

### `override`

Чтобы определить некоторую пользовательскую логику переопределения, используйте `override`. Предоставленный обратный вызов имеет доступ к исходной схеме Zod и схеме JSON по умолчанию. _Эта функция должна напрямую изменять `ctx.jsonSchema`._

```ts
const mySchema =
    /* ... */
    z.toJSONSchema(mySchema, {
        override: (ctx) => {
            ctx.zodSchema; // the original Zod schema
            ctx.jsonSchema; // the default JSON Schema

            // directly modify
            ctx.jsonSchema.whatever = 'sup';
        },
    });
```

Обратите внимание, что непредставимые типы будут вызывать ошибку `Error` перед вызовом этой функции. Если вы пытаетесь определить пользовательское поведение для непредставимого типа, вам необходимо использовать настройку `unrepresentable: "any"` вместе с `override`.

```ts
// support z.date() as ISO datetime strings
const result = z.toJSONSchema(z.date(), {
    unrepresentable: 'any',
    override: (ctx) => {
        const def = ctx.zodSchema._zod.def;
        if (def.type === 'date') {
            ctx.jsonSchema.type = 'string';
            ctx.jsonSchema.format = 'date-time';
        }
    },
});
```

### `io` {#io}

Некоторые типы схем имеют разные типы ввода и вывода, например `ZodPipe`, `ZodDefault` и принудительные примитивы. По умолчанию результат `z.toJSONSchema` представляет тип вывода; чтобы извлечь тип ввода, используйте `"io": "input"`.

```ts
const mySchema = z
    .string()
    .transform((val) => val.length)
    .pipe(z.number());
// ZodPipe

const jsonSchema = z.toJSONSchema(mySchema);
// => { type: "number" }

const jsonSchema = z.toJSONSchema(mySchema, {
    io: 'input',
});
// => { type: "string" }
```

## Реестры

Передача схемы в `z.toJSONSchema()` вернет _самостоятельную_ схему JSON.

В других случаях у вас может быть набор схем Zod, которые вы хотите представить с помощью нескольких взаимосвязанных схем JSON, например, для записи в файлы `.json` и обслуживания с веб-сервера.

```ts
import * as z from 'zod';

const User = z.object({
    name: z.string(),
    get posts() {
        return z.array(Post);
    },
});

const Post = z.object({
    title: z.string(),
    content: z.string(),
    get author() {
        return User;
    },
});

z.globalRegistry.add(User, { id: 'User' });
z.globalRegistry.add(Post, { id: 'Post' });
```

Для этого можно передать [реестр](./metadata.md#registries) в `z.toJSONSchema()`.

> **Важно** — Все схемы должны иметь зарегистрированное свойство `id` в реестре! Любые схемы без `id` будут игнорироваться.

```ts
z.toJSONSchema(z.globalRegistry);
// => {
//   schemas: {
//     User: {
//       id: 'User',
//       type: 'object',
//       properties: {
//         name: { type: 'string' },
//         posts: { type: 'array', items: { '$ref': 'Post' } }
//       },
//       required: [ 'name', 'posts' ],
//       additionalProperties: false,
//     },
//     Post: {
//       id: 'Post',
//       type: 'object',
//       properties: {
//         title: { type: 'string' },
//         content: { type: 'string' },
//         author: { '$ref': 'User' }
//       },
//       required: [ 'title', 'content', 'author' ],
//       additionalProperties: false,
//     }
//   }
// }
```

По умолчанию URI `$ref` представляют собой простые относительные пути, такие как `"User"`. Чтобы сделать эти URI абсолютными, используйте опцию `uri`. Для этого требуется функция, которая преобразует `id` в полный URI.

```ts
z.toJSONSchema(z.globalRegistry, {
    uri: (id) => `https://example.com/${id}.json`,
});
// => {
//   schemas: {
//     User: {
//       id: 'User',
//       type: 'object',
//       properties: {
//         name: { type: 'string' },
//         posts: {
//           type: 'array',
//           items: { '$ref': 'https://example.com/Post.json' }
//         }
//       },
//       required: [ 'name', 'posts' ],
//       additionalProperties: false,
//     },
//     Post: {
//       id: 'Post',
//       type: 'object',
//       properties: {
//         title: { type: 'string' },
//         content: { type: 'string' },
//         author: { '$ref': 'https://example.com/User.json' }
//       },
//       required: [ 'title', 'content', 'author' ],
//       additionalProperties: false,
//     }
//   }
// }
```
