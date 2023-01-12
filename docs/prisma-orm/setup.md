---
title: Настройки
description: Настройки
---

# Настройки

## select

`select` определяет, какие поля включаются в возвращаемый объект.

```js
const user = await prisma.user.findUnique({
    where: { email },
    select: {
        id: true,
        email: true,
        first_name: true,
        last_name: true,
        age: true,
    },
});

// or
const usersWithPosts = await prisma.user.findMany({
    select: {
        id: true,
        email: true,
        posts: {
            select: {
                id: true,
                title: true,
                content: true,
                author_id: true,
                created_at: true,
            },
        },
    },
});

// or
const usersWithPostsAndComments = await prisma.user.findMany(
    {
        select: {
            id: true,
            email: true,
            posts: {
                include: {
                    comments: true,
                },
            },
        },
    }
);
```

## include

`include` определяет, какие отношения (связанные записи) включаются в возвращаемый объект.

```js
const userWithPostsAndComments = await prisma.user.findUnique(
    {
        where: { email },
        include: {
            posts: true,
            comments: true,
        },
    }
);
```

## where

`where` определяет один или более фильтр (о фильтрах мы поговорим отдельно), применяемый к свойствам записи или связанных записей:

```js
const admins = await prisma.user.findMany({
    where: {
        email: {
            contains: 'admin',
        },
    },
});
```

## orderBy

`orderBy` определяет поля и порядок сортировки. Возможными значениями `orderBy` являются `asc` и `desc`.

```js
const usersByPostCount = await prisma.user.findMany({
    orderBy: {
        posts: {
            count: 'desc',
        },
    },
});
```

## distinct

`distinct` определяет поля, которые должны быть уникальными в возвращаемом объекте.

```js
const distinctCities = await prisma.user.findMany({
    select: {
        city: true,
        country: true,
    },
    distinct: ['city'],
});
```
