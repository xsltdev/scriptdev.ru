---
title: Транзакции
description: Транзакция — это последовательность операций чтения/записи, которые обрабатываются как единое целое, т. е. либо все операции завершаются успешно, либо все операции отклоняются с ошибкой
---

# Транзакции

**Транзакция** — это последовательность операций чтения/записи, которые обрабатываются как единое целое, т. е. либо все операции завершаются успешно, либо все операции отклоняются с ошибкой.

Prisma позволяет использовать транзакции тремя способами:

_вложенные запросы_ (см. выше): операции с родительскими и связанными записями выполняются в контексте одной транзакции

```js
const newUserWithProfile = await prisma.user.create({
    data: {
        email,
        profile: {
            // !
            create: {
                first_name,
                last_name,
            },
        },
    },
});
```

_пакетированные/массовые_ (batch/bulk) транзакции: выполнение нескольких операций за один раз с помощью таких запросов, как `createMany`, `updateMany` и `deleteMany`

```js
const removedUser = await prisma.user.delete({
    where: {
        email,
    },
});

// !
await prisma.post.deleteMany({
    where: {
        author_id: removedUser.id,
    },
});
```

_вызов метода \$transaction_.

## \$transaction

Интерфейс `$transaction` может быть использован в двух формах:

-   `$transaction([ query1, query2, ...queryN ])` — принимает массив последовательно выполняемых запросов;
-   `$transaction(fn)` — принимает функцию, которая может включать запросы и другой код.

Пример транзакции, возвращающей посты, в заголовке которых встречается слово TypeScript и общее количество постов:

```js
const [
    postsAboutTypeScript,
    totalPostCount,
] = await prisma.$transaction([
    prisma.post.findMany({
        where: { title: { contains: 'TypeScript' } },
    }),
    prisma.post.count(),
]);
```

В `$transaction` допускается использование SQL:

```js
const [userNames, updatedUser] = await prisma.$transaction([
    prisma.$queryRaw`SELECT 'user_name' FROM users`,
    prisma.$executeRaw`UPDATE users SET user_name = 'Harry' WHERE id = 42`,
]);
```

## Интерактивные транзакции

Интерактивные транзакции предоставляют разработчикам больший контроль над выполняемыми в контексте транзакции операциями. В данный момент они имеют статус экспериментальной возможности, которую можно включить следующим образом:

```
generator client {
  provider        = "prisma-client-js"
  previewFeatures = ["interactiveTransactions"]
}
```

Рассмотрим пример совершения платежа.

Предположим, что у Alice и Bob имеется по 100$ на счетах (account), и Alice хочет отправить Bob свои 100$.

```js
import { PrismaClient } from '@prisma/client';
const prisma = new PrismaClient();

async function transfer(from, to, amount) {
    try {
        await prisma.$transaction(async (prisma) => {
            // 1. Уменьшаем баланс отправителя
            const sender = await prisma.account.update({
                data: {
                    balance: {
                        decrement: amount,
                    },
                },
                where: {
                    email: from,
                },
            });

            // 2. Проверяем, что баланс отправителя после уменьшения >= 0
            if (sender.balance < 0) {
                throw new Error(
                    `${from} имеет недостаточно средств для отправки ${amount}`
                );
            }

            // 3. Увеличиваем баланс получателя
            const recipient = await prisma.account.update({
                data: {
                    balance: {
                        increment: amount,
                    },
                },
                where: {
                    email: to,
                },
            });

            return recipient;
        });
    } catch (e) {
        // обрабатываем ошибку
    }
}

async function main() {
    // эта транзакция разрешится
    await transfer('alice@mail.com', 'bob@mail.com', 100);
    // а эта провалится
    await transfer('alice@mail.com', 'bob@mail.com', 100);
}

main().finally(() => {
    prisma.$disconnect();
});
```

Подробнее о транзакциях можно почитать [здесь](https://www.prisma.io/docs/concepts/components/prisma-client/transactions).
