---
title: Инициализация проекта
description: Инициализация проекта для Prisma ORM
---

# Инициализация проекта

Создаем директорию, переходим в нее и инициализируем Node.js-проект:

```sh
mkdir prisma-test
cd prisma-test

yarn init -yp
# or
npm init -y
```

Устанавливаем Prisma в качестве зависимости для разработки:

```sh
yarn add -D prisma

# or
npm i -D prisma
```

Инициализируем проект Prisma:

```sh
npx prisma init
```

Это приводит к генерации файлов `prisma/schema.prisma` и `.env`.

В файле `.env` содержится переменная `DATABASE_URL`, значением которой является путь к (адрес) БД. Файл `schema.prisma` мы рассмотрим позже.
