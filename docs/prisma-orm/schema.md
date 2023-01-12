---
title: Схема
description: Схема
---

# Схема

В файле `schema.prisma` мы видим такие строки:

```
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}
```

`datasource` — источник данных:

-   `provider` — название провайдера для доступа к БД: `sqlite`, `postgresql`, `mysql`, `sqlserver` или `mongodb` (по умолчанию — `postgresql`);
-   `url` — адрес БД (по умолчанию — значение переменной `DATABASE_URL`);
-   `shadowDatabaseUrl` — адрес "теневой" БД (для БД, предоставляемых облачными провайдерами): используется для миграций для разработки (`prisma migrate dev`);

`generator` — генератор клиента на основе схемы:

-   `provider` — провайдер генератора (единственным доступным на сегодняшний день провайдером является `prisma-client-js`);
-   `binaryTargets` — определяет операционную систему для клиента Prisma. Значением по умолчанию является `native`, но иногда это приходится указывать явно, например, при использовании клиента в Docker-контейнере (в этом случае также приходится явно выполнять `prisma generate`)

```
generator client {
  provider      = "prisma-client-js"
  binaryTargets = ["native"]
}

datasource db {
  provider          = "postgresql"
  url               = env("DATABASE_URL")
  shadowDatabaseUrl = env("SHADOW_DATABASE_URL")
}
```

Для работы со схемой удобно пользоваться расширением [Prisma для VSCode](https://marketplace.visualstudio.com/items?itemName=Prisma.prisma). Соответствующий раздел в файле `settings.json` должен выглядеть так:

```json
"[prisma]": {
	"editor.defaultFormatter": "Prisma.prisma"
}
```

Определим в схеме модели для пользователя (`User`) и поста (`Post`):

```
model User {
	id String @id @default(uuid()) @db.Uuid
	email String @unique
	hash String @map("password_hash")
	first_name String?
	last_name String?
	age Int?
	role Role @default(USER)
	posts Post[]
	created_at DateTime @default(now())
	updated_at DateTime @updatedAt

	@@map("users")
}

model Post {
	id String @id @default(uuid())
	title String
	content String
	published Boolean
	author_id String
	author User @relation(fields: [author_id], references: [id])
	created_at DateTime @default(now())
	updated_at DateTime @updatedAt

	@@map("posts")
}

enum Role {
	USER
	ADMIN
}
```

Вот что мы здесь видим:

-   `id`, `email`, `hash` etc. — названия полей (колонок таблицы);
-   `@map` привязывает поле схемы (`hash`) к указанной колонке таблицы (`password_hash`). `@map` не меняет название колонки в БД и поля в генерируемом клиенте. Для MongoDB использование `@map` для `@id` является обязательным: `id String @default(auto()) @map("_id") @db.ObjectId`;
-   `String`, `Int`, `DateTime` etc. — типы данных (см. ниже);
-   `@db.Uuid` — тип данных, специфичный для одной или нескольких БД (в данном случае PostgreSQL);
-   модификатор `?` после названия типа означает, что данное поле является опциональным (необязательным, может иметь значение `NULL`);
-   модификатор `[]` после названия типа означает, что значением данного поля является список (массив). Такое поле не может быть опциональным;
-   префикс `@` означает атрибут поля, а префикс `@@` — атрибут блока (модели, таблицы). Некоторые атрибуты принимают параметры;
-   атрибут `@id` означает, что данное поле является первичным (основным) ключом таблицы (`PRIMARY KEY`) (идентификатор модели). Такое поле не может быть опциональным;
-   атрибут `@default` присваивает полю указанное значение по умолчанию (при отсутствии значения поля) (`DEFAULT`). Дефолтными могут быть статические значения (`42`, `hi`) или значения, генерируемые функциями `autoincrement`, `dbgenerated`, `cuid`, `uuid` и `now` (функции атрибутов; см. ниже);
-   атрибут `@unique` означает, что значение поля должно быть уникальным в пределах таблицы (`UNIQUE`). Таблица должна иметь хотя бы одно поле `@id` или `@unique`;
-   атрибут `@relation` указывает на существование отношений между таблицами. В данном случае между таблицами `users` и `posts` существуют отношения один-ко-многим (one-to-many, 1-n) — у одного пользователя может быть несколько постов (`FOREIGN KEY` / `REFERENCES`) (об отношениях мы поговорим отдельно);
-   атрибут `@updatedAt` обновляет поле текущими датой и временем при любой модификации записи;
    у нас имеется перечисление (`enum`), значения которого используются в качестве значений поля `role` модели `User` (значением по умолчанию является `USER`);
-   атрибут `@@map` привязывает название модели к названию таблицы в БД. `@@map` не меняет название таблицы в БД и модели в генерируемом клиенте.

## Типы данных

Допустимыми в названиях полей являются следующие символы: `[A-Za-z][a-za-z0-9_]*`.

-   `String` — строка переменной длины (для PostgreSQL — это тип text);
-   `Boolean` — логическое значение: `true` или `false` (boolean);
-   `Int` — целое число (integer);
-   `BigInt` — BigInt (integer);
-   `Float` — число с плавающей точкой (запятой) (double precision);
-   `Decimal` (decimal);
-   `DateTime` — дата и время в формате ISO 8601;
-   `Json` — объект в формате JSON (jsonb);
-   `Bytes` (bytea).

Атрибут `@db` позволяет использовать типы данных, специфичные для одной или нескольких БД.

## Атрибуты

Кроме упомянутых выше, в схеме можно использовать следующие атрибуты:

-   `@@id` — определяет составной (composite) первичный ключ таблицы, например, `@@id[title, author]` (в данном случае соответствующее поле будет называться `title_author` — это можно изменить);
-   `@@unique` — определяет составное ограничение уникальности (unique constraint) для указанных полей (такие поля не могут быть опциональными), например, `@@unique([title, author])`;
-   `@@index` — определяет индекс в БД (`INDEX`), например, `@@index([title, author])`;
-   `@ignore`, `@@ignore` — используется для обозначения невалидных полей и моделей, соответственно.

## Функции атрибутов

-   `auto` — представляет дефолтные значения, генерируемые БД (только для MongoDB);
-   `autoincrement` — генерирует последовательные целые числа (`SERIAL` в PostgreSQL, не поддерживается MongoDB);
-   `cuid` — генерирует глобальный уникальный идентификатор на основе спецификации `cuid`;
-   `uuid` — генерирует глобальный уникальный идентификатор на основе спецификации `UUID`;
-   `now` — возвращает текущую отметку времени (`timestamp`) (`CURRENT_TIMESTAMP` в PostgreSQL);
-   `dbgenerated` — представляет дефолтные значения, которые не могут быть выражены в схеме (например, `random()`).

Подробнее о схеме можно почитать [здесь](https://www.prisma.io/docs/reference/api-reference/prisma-schema-reference).

## Отношения

Атрибут `@relation` указывает на существование отношений между моделями (таблицами). Он принимает следующие параметры:

-   `name?: string` — название отношения;
-   `fields?: [field1, field2, ...fieldN]` — список полей текущей модели (в нашем случае это `[author_id]` модели `Post`); обратите внимание: само поле определяется отдельно);
-   `references: [field1, field2, ...fieldN]` — список полей другой модели (стороны отношений) (в нашем случае это `[id]` модели `User`).

В приведенной выше схеме полями, указывающими на существование отношений между моделями `User` и `Post`, являются поля `posts` и `author`. Эти поля существуют только на уровне Prisma, в БД они не создаются. Скалярное поле `author_id` также существует только на уровне Prisma — это внешний ключ (`FOREIGN KEY`), соединяющий `Post` с `User`.

Как известно, существует 3 вида отношений:

-   один-к-одному (one-to-one, 1-1);
-   один-ко-многим (one-to-many, 1-n);
-   многие-ко-многим (many-to-many, m-n).

Атрибут `@relation` является обязательным только для отношений 1-1 и 1-n.

Предположим, что в нашей схеме имеются такие модели:

```
model User {
	id Int @id @default(autoincrement())
	posts Post[]
	profile Profile?
}

model Profile {
	id Int @id @default(autoincrement())
	user User @relation(fields: [userId], references: [id])
	userId Int
}

model Post {
	id Int @id @default(autoincrement())
	author User @relation(fields: [authorId], references: [id])
	authorId Int
	categories Category[]
}

model Category {
	id Int @id @default(autoincrement())
	posts Post[]
}
```

Вот что мы здесь видим:

-   между моделями `User` и `Profile` существуют отношения 1-1 — у одного пользователя может быть только один профиль;
-   между моделями `User` и `Post` существуют отношения 1-n — у одного пользователя может быть несколько постов;
-   между моделями `Post` и `Category` существуют отношения m-n — один пост может принадлежать к нескольким категориям, в одну категорию может входить несколько постов.

Подробнее об отношениях можно почитать [здесь](https://www.prisma.io/docs/concepts/components/prisma-schema/relations).
