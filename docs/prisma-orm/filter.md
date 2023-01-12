---
title: Фильтры и операторы
description: Фильтры и операторы
---

# Фильтры и операторы

## Фильтры

`equals` — значение равняется `n`

```js
const usersWithNameHarry = await prisma.user.findMany({
    where: {
        name: {
            equals: 'Harry',
        },
    },
});

// `equals` может быть опущено
const usersWithNameHarry = await prisma.user.findMany({
    where: {
        name: 'Harry',
    },
});
```

`not` — значение не равняется `n`;

`in` — значение `n` содержится в списке (массиве)

```js
const usersWithNameAliceOrBob = await prisma.user.findMany({
    where: {
        user_name: {
            // !
            in: ['Alice', 'Bob'],
        },
    },
});
```

`notIn` — `n` не содержится в списке;

`lt` — `n` меньше `x`

```js
const notPopularPosts = await prisma.post.findMany({
    where: {
        likeCount: {
            lt: 100,
        },
    },
});
```

`lte` — `n` меньше или равно `x`;

`gt` — `n` больше `x`;

`gte` — `n` больше или равно `x`;

`contains` — `n` содержит `x`

```js
const admins = await prisma.user.findMany({
    where: {
        email: {
            contains: 'admin',
        },
    },
});
```

`startsWith` — `n` начинается с `x`

```js
const usersWithNameStartsWithA = await prisma.user.findMany(
    {
        where: {
            user_name: {
                startsWith: 'A',
            },
        },
    }
);
```

`endsWith` — `n` заканчивается `x`.

## Операторы

`AND` — все условия должны возвращать `true`

```js
const notPublishedPostsAboutTypeScript = await prisma.post.findMany(
    {
        where: {
            AND: [
                {
                    title: {
                        contains: 'TypeScript',
                    },
                },
                {
                    published: false,
                },
            ],
        },
    }
);
```

Обратите внимание: оператор указывается до названия поля (снаружи поля), а фильтр после (внутри).

`OR` — хотя бы одно условие должно возвращать `true`;

`NOT` — все условия должны возвращать `false`.

## Фильтры для связанных записей

`some` — возвращает все связанные записи, соответствующие одному или более критерию фильтрации

```js
const usersWithPostsAboutTypeScript = await prisma.user.findMany(
    {
        where: {
            posts: {
                some: {
                    title: {
                        contains: 'TypeScript',
                    },
                },
            },
        },
    }
);
```

`every` — возвращает все связанные записи, соответствующие всем критериям;

`none` — возвращает все связанные записи, не соответствующие ни одному критерию;

`is` — возвращает все связанные записи, соответствующие критерию;

`notIs` — возвращает все связанные записи, не соответствующие критерию.
