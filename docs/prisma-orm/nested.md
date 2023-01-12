---
title: Вложенные запросы
description: Вложенные запросы
---

# Вложенные запросы

`create: { data } | [{ data1 }, { data2 }, ...{ dataN }]` — добавляет новую связанную запись или набор записей в родительскую запись. `create` доступен при создании (`create`) новой родительской записи или обновлении (`update`) существующей родительской записи

```js
const user = await prisma.user.create({
    data: {
        email,
        profile: {
            // вложенный запрос
            create: {
                first_name,
                last_name,
            },
        },
    },
});
```

`createMany: [{ data1 }, { data2 }, ...{ dataN }]` — добавляет набор новых связанных записей в родительскую запись. `createMany` доступен при создании (`create`) новой родительской записи или обновлении (`update`) существующей родительской записи

```js
const userWithPosts = await prisma.user.create({
    data: {
        email,
        posts: {
            // !
            createMany: {
                data: posts,
            },
        },
    },
});
```

`update: { data } | [{ data1 }, { data2 }, ...{ dataN }]` — обновляет одну или более связанных записей

```js
const user = await prisma.user.update({
    where: { email },
    data: {
        profile: {
            // !
            update: { age },
        },
    },
});
```

`updateMany: { data } | [{ data1 }, { data2 }, ...{ dataN }]` — обновляет массив связанных записей. Поддерживается фильтрация

```js
const result = await prisma.user.update({
    where: { id },
    data: {
        posts: {
            // !
            updateMany: {
                where: {
                    published: false,
                },
                data: {
                    like_count: 0,
                },
            },
        },
    },
});
```

`upsert: { data } | [{ data1 }, { data2 }, ...{ dataN }]` — обновляет существующую связанную запись или создает новую

```js
const user = await prisma.user.update({
    where: { email },
    data: {
        profile: {
            // !
            upsert: {
                create: { age },
                update: { age },
            },
        },
    },
});
```

`delete: boolean | { data } | [{ data1 }, { data2 }, ...{ dataN }]` — удаляет связанную запись. Родительская запись при этом не удаляется

```js
const user = await prisma.user.update({
    where: { email },
    data: {
        profile: {
            delete: true,
        },
    },
});
```

`deleteMany: { data } | [{ data1 }, { data2 }, ...{ dataN }]` — удаляет связанные записи. Поддерживается фильтрация

```js
const user = await prisma.user.update({
    where: { id },
    data: {
        age,
        posts: {
            // !
            deleteMany: {},
        },
    },
});
```

`set: { data } | [{ data1 }, { data2 }, ...{ dataN }]` — перезаписывает значение связанной записи

```js
const userWithPosts = await prisma.user.update({
    where: { email },
    data: {
        posts: {
            // !
            set: newPosts,
        },
    },
});
```

`connect` — подключает запись к существующей связанной записи по идентификатору или уникальному полю

```js
const user = await prisma.post.create({
    data: {
        title,
        content,
        author: {
            connect: { email },
        },
    },
});
```

`connectOrCreate` — подключает запись к существующей связанной записи по идентификатору или уникальному полю либо создает связанную запись при отсутствии таковой;

`disconnect` — отключает родительскую запись от связанной без удаления последней. `disconnect` доступен только если отношение является опциональным.
