---
title: Клиент
description: Импортируем и создаем экземпляр клиента Prisma
---

# Клиент

Импортируем и создаем экземпляр клиента Prisma:

```js
import { PrismaClient } from '@prisma/client';
const prisma = new PrismaClient();
export default prisma;
```

Иногда может потребоваться делать так:

```js
const Prisma = require('prisma');
const prisma = new Prisma.PrismaClient();
module.exports = prisma;
```

## Запросы

### findUnique

`findUnique` позволяет извлекать единичные записи по идентификатору или уникальному полю.

**Сигнатура**

```
findUnique({
	where: condition,
	select?: fields,
	include?: relations,
	rejectOnNotFound?: boolean
})
```

Модификатор `?` означает, что поле является опциональным.

-   `condition` — условие для выборки;
-   `fields` — поля для выборки;
-   `relations` — отношения (связанные поля) для выборки;
-   `rejectOnNotFound` — если имеет значение `true`, при отсутствии записи выбрасывается исключение `NotFoundError`. Если имеет значение `false`, при отсутствии записи возвращается `null`.

**Пример**

```js
async function getUserById(id) {
    try {
        const user = await prisma.user.findUnique({
            where: {
                id,
            },
        });
        return user;
    } catch (e) {
        onError(e);
    }
}
```

### findFirst

`findFirst` возвращает первую запись, соответствующую заданному критерию.

**Сигнатура**

```
findFirst({
	where?: condition,
	select?: fields,
	include?: relations,
	rejectOnNotFound?: boolean,
	distinct?: field,
	orderBy?: order,
	cursor?: position,
	skip?: number,
	take?: number
})
```

-   `distinct` — фильтрация по определенному полю;
-   `orderBy` — сортировка по определенному полю и в определенном порядке;
-   `cursor` — позиция начала списка (как правило, `id` или другое уникальное значение);
-   `skip` — количество пропускаемых записей;
-   `take` — количество возвращаемых записей (в данном случае может иметь значение `1` или `-1`: во втором случае возвращается последняя запись.

**Пример**

```js
async function getLastPostByAuthorId(author_id) {
    try {
        const post = await prisma.post.findFirst({
            where: {
                author_id,
            },
            orderBy: {
                created_at: 'asc',
            },
            take: -1,
        });
        return post;
    } catch (e) {
        onError(e);
    }
}
```

### findMany

`findMany` возвращает все записи, соответствующие заданному критерию.

**Сигнатура**

```
findMany({
	where?: condition,
	select?: fields,
	include?: relations,
	rejectOnNotFound?: boolean,
	distinct?: field,
	orderBy?: order,
	cursor?: position,
	skip?: number,
	take?: number
})
```

**Пример**

```js
async function getAllPostsByAuthorId(author_id) {
    try {
        const posts = await prisma.post.findMany({
            where: {
                author_id,
            },
            orderBy: {
                updated_at: 'desc',
            },
        });
        return posts;
    } catch (e) {
        onError(e);
    }
}
```

### create

`create` создает новую запись.

**Сигнатура**

```
create({
	data: _data,
	select?: fields,
	include?: relations
})
```

-   `_data` — данные создаваемой записи.

**Пример**

```js
async function createUserWithProfile(data) {
    const {
        email,
        password,
        firstName,
        lastName,
        age,
    } = data;
    try {
        const hash = await argon2.hash(password);
        const user = await prisma.user.create({
            data: {
                email,
                hash,
                profile: {
                    create: {
                        first_name: firstName,
                        last_name: lastName,
                        age,
                    },
                },
            },
            select: {
                email: true,
            },
            include: {
                profile: true,
            },
        });
        return user;
    } catch (e) {
        onError(e);
    }
}
```

### update

`update` обновляет существующую запись.

**Сигнатура**

```
update({
	data: _data,
	where: condition,
	select?: fields,
	include?: relations
})
```

**Пример**

```js
async function updateUserById(id, changes) {
    const { email, age } = changes;
    try {
        const user = await prisma.user.update({
            where: {
                id,
            },
            data: {
                email,
                profile: {
                    update: {
                        age,
                    },
                },
            },
            select: {
                email: true,
            },
            include: {
                profile: true,
            },
        });
        return user;
    } catch (e) {
        onError(e);
    }
}
```

### upsert

`upsert` обновляет существующую или создает новую запись.

**Сигнатура**

```
upsert({
	create: _data,
	update: _data,
	where: condition,
	select?: fields,
	include?: relations
})
```

**Пример**

```js
async function updateOrCreateUser(data) {
    const { userName, email, password } = data;
    try {
        const hash = await argon2.hash(password);
        const user = await prisma.user.create({
            where: { user_name: userName },
            update: {
                email,
                hash,
            },
            create: {
                email,
                hash,
                user_name: userName,
            },
            select: { user_name: true, email: true },
        });
        return user;
    } catch (e) {
        onError(e);
    }
}
```

### delete

`delete` удаляет существующую запись по идентификатору или уникальному полю.

**Сигнатура**

```
delete({
	where: condition,
	select?: fields,
	include?: relations
})
```

**Пример**

```js
async function removeUserById(id) {
    try {
        await prisma.user.delete({
            where: {
                id,
            },
        });
    } catch (e) {
        onError(e);
    }
}
```

### createMany

`createMany` создает несколько записей с помощью одной транзакции (о транзакциях мы поговорим отдельно).

**Сигнатура**

```
createMany({
	data: _data[],
	skipDuplicates?: boolean
})
```

-   `_data[]` — данные для создаваемых записей в виде массива;
-   `skipDuplicates` — при значении `true` создаются только уникальные записи.

**Пример**

```js
// предположим, что `users` - это массив объектов
async function createUsers(users) {
    try {
        const users = await prisma.user.createMany({
            data: users,
        });
        return users;
    } catch (e) {
        onError(e);
    }
}
```

### updateMany

`updateMany` обновляет несколько существующих записей за один раз и возвращает количество (sic) обновленных записей.

**Сигнатура**

```
updateMany({
	data: _data[],
	where?: condition
})
```

**Пример**

```js
async function updateProductsByCategory(
    category,
    newDiscount
) {
    try {
        const count = await prisma.product.updateMany({
            where: {
                category,
            },
            data: {
                discount: newDiscount,
            },
        });
        return count;
    } catch (e) {
        onError(e);
    }
}
```

### deleteMany

`deleteMany` удаляет несколько записей с помощью одной транзакции и возвращает количество удаленных записей.

**Сигнатура**

```
deleteMany({
	where?: condition
})
```

**Пример**

```js
async function removeAllPostsByUserId(author_id) {
    try {
        const count = await prisma.post.deleteMany({
            where: {
                author_id,
            },
        });
        return count;
    } catch (e) {
        onError(e);
    }
}
```

### count

`count` возвращает количество записей, соответствующих заданному критерию.

**Сигнатура**

```
count({
	where?: condition,
	select?: fields,
	cursor?: position,
	orderBy?: order,
	skip?: number,
	take?: number
})
```

**Пример**

```js
async function countUsersWithPublishedPosts() {
    try {
        const count = await prisma.user.count({
            where: {
                post: {
                    some: {
                        published: true,
                    },
                },
            },
        });
        return count;
    } catch (e) {
        onError(e);
    }
}
```

### aggregate

`aggregate` выполняет агрегирование полей.

**Сигнатура**

```
aggregate({
	where?: condition,
	select?: fields,
	cursor?: position,
	orderBy?: order,
	skip?: number,
	take?: number,

	_count: count,
	_avg: avg,
	_sum: sum,
	_min: min,
	_max: max
})
```

-   `_count` — возвращает количество совпадающих записей или не null-полей;
-   `_avg` — возвращает среднее значение определенного поля;
-   `_sum` — возвращает сумму значений определенного поля;
-   `_min` — возвращает наименьшее значение определенного поля;
-   `_max` — возвращает наибольшее значение определенного поля.

**Пример**

```js
async function getAllUsersCountAndMinMaxProfileViews() {
    try {
        const result = await prisma.user.aggregate({
            _count: {
                _all: true,
            },
            _max: {
                profileViews: true,
            },
            _min: {
                profileViews: true,
            },
        });
        return result;
    } catch (e) {
        onError(e);
    }
}
```

### groupBy

`groupBy` выполняет группировку полей.

**Сигнатура**

```
groupBy({
	by?: by,
	having?: having,

	where?: condition,
	orderBy?: order,
	skip?: number,
	take?: number,

	_count: count,
	_avg: avg,
	_sum: sum,
	_min: min,
	_max: max
})
```

-   `by` — определяет поле или комбинацию полей для группировки записей;
-   `having` — позволяет фильтровать группы по агрегируемому значению.

**Пример**

В следующем примере мы выполняем группировку по `country` / `city`, где среднее значение `profileViews` превышает `100`, и возвращаем общее количество (`_sum`) `profileViews` для каждой группы. Запрос также возвращает количество всех (`_all`) записей в каждой группе и все записи с не `null` значениями поля `city` в каждой группе:

```js
async function getUsers() {
    try {
        const result = await prisma.user.groupBy({
            by: ['country', 'city'],
            _count: {
                _all: true,
                city: true,
            },
            _sum: {
                profileViews: true,
            },
            orderBy: {
                country: 'desc',
            },
            having: {
                profileViews: {
                    _avg: {
                        gt: 100,
                    },
                },
            },
        });
        return result;
    } catch (e) {
        onError(e);
    }
}
```
