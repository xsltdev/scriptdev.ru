---
title: Методы клиента
description: Методы клиента
---

# Методы клиента

`$disconnect` — закрывает соединение с БД, которое было установлено после вызова метода `$connect` (данный метод чаще всего не требуется вызывать явно), и останавливает движок запросов (query engine) Prisma

```js
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

async function seedDb() {
    try {
        await prisma.model.create(data);
    } catch (e) {
        onError(e);
    } finally {
        // !
        await prisma.$disconnect();
    }
}
```

`$use` — добавляет посредника (`middleware`)

```js
prisma.$use(async (params, next) => {
    console.log('Это посредник');
    // работаем с `params`
    return next(params);
});
```

`next` — представляет "следующий уровень" в стеке посредников. Таким уровнем может быть следующий посредник или движок запросов Prisma;

`params` — объект со следующими свойствами:

-   `action` — тип запроса, например, `create` или `findMany`;
-   `args` — аргументы, переданные в запрос, например, `where` или `data`;
-   `model` — модель, например, `User` или `Post`;
-   `runInTransaction` — возвращает `true`, если запрос был запущен в контексте транзакции;

методы `$queryRaw`, `$executeRaw` и `$runCommandRaw` предназначены для работы с SQL. Почитать о них можно [здесь](https://www.prisma.io/docs/concepts/components/prisma-client/raw-database-access);

`$transaction` — выполняет запросы в контексте транзакции.

Подробнее о клиенте можно почитать [здесь](https://www.prisma.io/docs/reference/api-reference/prisma-schema-reference).
