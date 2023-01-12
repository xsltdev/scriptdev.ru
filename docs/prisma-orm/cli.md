---
title: CLI
description: Интерфейс командной строки
---

# CLI

**Интерфейс командной строки** (Command line interface, CLI) Prisma предоставляет следующие основные возможности (команды):

**`init`** — создает шаблон Prisma-проекта:

-   `--datasource-provider` — провайдер для работы с БД: `sqlite`, `postgresql`, `mysql`, `sqlserver` или `mongodb` (перезаписывает datasource из `schema.prisma`);
-   `--url` — адрес БД (перезаписывает `DATABASE_URL`).

```sh
npx prisma init --datasource-provider mysql --url mysql://user:password@localhost:3306/mydb
```

**`generate`** — генерирует клиента Prisma на основе схемы (`schema.prisma`). Клиент Prisma предоставляет программный интерфейс приложения (Application Programming Interface, API) для работы с моделями и типы для TypeScript.

```sh
npx prisma generate
```

**`db pull`** — генерирует модели на основе существующей схемы БД.

```sh
npx prisma db pull
```

**`db push`** — синхронизирует состояние схемы Prisma с БД без выполнения миграций. БД создается при отсутствии. Используется для прототипировании БД и в локальной разработке. Также может быть полезной в случае ограниченного доступа к БД, например, при использовании БД, предоставляемой облачными провайдерами, такими как ElephantSQL или Heroku.

```sh
npx prisma db push
```

**`seed`** — выполняет скрипт для наполнения БД начальными (фиктивными) данными. Путь к соответствующему файлу определяется в `package.json`.

```json
"prisma": {
	"seed": "node prisma/seed.js"
}
```

```sh
npx prisma seed
```

**`migrate`**

-   `dev` — выполняет миграцию для разработки:
-   `--name` — название миграции

```sh
npx prisma migrate dev --name init
```

Это приводит к созданию БД при ее отсутствии, генерации файла `prisma/migrations/migration_name.sql`, выполнению инструкции из этого файла (синхронизации БД со схемой) и генерации (регенерации) клиента (`prisma generate`).

Данная команда должна выполняться после каждого изменения схемы.

**`reset`** — удаляет и заново создает БД или выполняет "мягкий сброс", удаляя все данные, таблицы, индексы и другие артефакты

```sh
npx prisma migrate reset
```

**`deploy`** — выполняет производственную миграцию

```sh
npx prisma migrate deploy
```

**`studio`** — позволяет просматривать и управлять данными, хранящимися в БД, в интерактивном режиме:

-   `--browser`, `-b` — название браузера (по умолчанию используется дефолтный браузер);
-   `--port`, -p — номер порта (по умолчанию — `5555`)

```sh
npx prisma studio
```

без автоматического открытия вкладки браузера

```sh
npx prisma studio -b none
```

Подробнее о CLI можно почитать [здесь](https://www.prisma.io/docs/reference/api-reference/command-reference).
