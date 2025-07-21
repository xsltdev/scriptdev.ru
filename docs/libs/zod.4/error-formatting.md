---
title: Formatting errors
---

# Ошибки форматирования

Zod уделяет особое внимание _полноте_ и _правильности_ отчетов об ошибках. Во многих случаях полезно преобразовать `$ZodError` в более удобный формат. Zod предоставляет для этого несколько утилит.

Рассмотрим эту простую схему объекта.

```ts
import * as z from 'zod';

const schema = z.strictObject({
    username: z.string(),
    favoriteNumbers: z.array(z.number()),
});
```

Попытка анализа этих недействительных данных приводит к ошибке, содержащей две проблемы.

```ts
const result = schema.safeParse({
    username: 1234,
    favoriteNumbers: [1234, '4567'],
    extraKey: 1234,
});

result.error!.issues;
[
    {
        expected: 'string',
        code: 'invalid_type',
        path: ['username'],
        message:
            'Invalid input: expected string, received number',
    },
    {
        expected: 'number',
        code: 'invalid_type',
        path: ['favoriteNumbers', 1],
        message:
            'Invalid input: expected number, received string',
    },
    {
        code: 'unrecognized_keys',
        keys: ['extraKey'],
        path: [],
        message: 'Unrecognized key: "extraKey"',
    },
];
```

## `z.treeifyError()`

Чтобы преобразовать ("treeify") эту ошибку в вложенный объект, используйте `z.treeifyError()`.

```ts
const tree = z.treeifyError(result.error);

// =>
// {
//   errors: [ 'Unrecognized key: "extraKey"' ],
//   properties: {
//     username: { errors: [ 'Invalid input: expected string, received number' ] },
//     favoriteNumbers: {
//       errors: [],
//       items: [
//         undefined,
//         {
//           errors: [ 'Invalid input: expected number, received string' ],
//         }
//       ]
//     }
//   }
// }
```

В результате получается вложенная структура, которая отражает саму схему. Вы можете легко получить доступ к ошибкам, которые произошли на определенном пути. Поле `errors` содержит сообщения об ошибках на данном пути, а специальные свойства `properties` и `items` позволяют вам продвигаться глубже в дерево.

```ts
tree.properties?.username?.errors;
// => ["Invalid input: expected string, received number"]

tree.properties?.favoriteNumbers?.items?.[1]?.errors;
// => ["Invalid input: expected number, received string"];
```

> Обязательно используйте опциональное цепочечное соединение (`?.`) для предотвращения ошибок при доступе к вложенным свойствам.

## `z.prettifyError()`

Функция `z.prettifyError()` предоставляет удобочитаемое строковое представление ошибки.

```ts
const pretty = z.prettifyError(result.error);
```

Это возвращает следующую строку:

```
✖ Unrecognized key: "extraKey"
✖ Invalid input: expected string, received number
  → at username
✖ Invalid input: expected number, received string
  → at favoriteNumbers[1]
```

## `z.formatError()`

!!!warn ""

    Это было признано устаревшим в пользу `z.treeifyError()`.

Чтобы преобразовать ошибку в вложенный объект:

```ts
const formatted = z.formatError(result.error);

// returns:
// {
// 	_errors: [ 'Unrecognized key: "extraKey"' ],
// 	username: { _errors: [ 'Invalid input: expected string, received number' ] },
// 	favoriteNumbers: {
// 		'1': { _errors: [ 'Invalid input: expected number, received string' ] },
// 		_errors: []
// 	}
// }
```

В результате получается вложенная структура, которая отражает саму схему. Вы можете легко получить доступ к ошибкам, которые произошли на определенном пути.

```ts
formatted?.username?._errors;
// => ["Invalid input: expected string, received number"]

formatted?.favoriteNumbers?.[1]?._errors;
// => ["Invalid input: expected number, received string"]
```

> Обязательно используйте опциональное цепочечное соединение (`?.`) для предотвращения ошибок при доступе к вложенным свойствам.

## `z.flattenError()`

Хотя `z.treeifyError()` полезен для обхода потенциально сложной вложенной структуры, большинство схем являются _плоскими_ — всего одного уровня глубины. В этом случае используйте `z.flattenError()` для получения чистого, неглубокого объекта ошибки.

```ts
const flattened = z.flattenError(result.error);
// { errors: string[], properties: { [key: string]: string[] } }

// {
//   formErrors: [ 'Unrecognized key: "extraKey"' ],
//   fieldErrors: {
//     username: [ 'Invalid input: expected string, received number' ],
//     favoriteNumbers: [ 'Invalid input: expected number, received string' ]
//   }
// }
```

Массив `formErrors` содержит все ошибки верхнего уровня (где `path` равен `[]`). Объект `fieldErrors` предоставляет массив ошибок для каждого поля в схеме.

```ts
flattened.fieldErrors.username; // => [ 'Invalid input: expected string, received number' ]
flattened.fieldErrors.favoriteNumbers; // => [ 'Invalid input: expected number, received string' ]
```
