# Как скачать и установить выгрузку из sql-ex.ru
Скрипты учебных баз данных, к которым адресуются запросы в упражнениях по SQL, можно скачать по ссылке с официального сайта:
[https://www.sql-ex.ru/db_script_download.php](https://www.sql-ex.ru/db_script_download.php) (выберите все базы одним файлом для PostgreSQL).

Используя скачанные скрипты, можно воссоздать у себя базу данных для выполнения упражнений. Для удобства делать это будем при помощи докера.

## Запуск контейнера с помощью docker-compose
Создадим файл `docker-compose-use-dump.yaml` и заполним его следующим содержимым:
```
version: '3.8'

services:
  postgres:
    image: postgres:14-alpine
    container_name: sql-ex
    ports:
      - 5432:5432
    volumes:
      - postgres_data:/var/lib/postgresql/data/
      - ./sql-ex-pg.sql:/docker-entrypoint-initdb.d/sql-ex-pg.sql
    env_file:
      - ./.env

volumes:
  postgres_data:
```

В папке с docker-compose файлом должны находиться следующие файлы:
1. Скачанный дамп базы данных `sql-ex-pg.sql`
2. `.env` с переменными окружения.

Содержание `.env`:
```
POSTGRES_PASSWORD=<password>
```

Запустим контейнер:
```
docker compose -f docker-compose-use-dump.yaml up -d --build
```

## Подключение к базе данных
Подключиться к базе можно по адресу `localhost:5432`, используя любой совместимый клиент, в том числе и сторонние плагины для вашей IDE.

Для проверки используем `psql`, запустить который можно следующей командой:

```
docker compose -f docker-compose-use-dump.yaml exec postgres psql -U postgres
```
Запустив `psql`, мы увидим приглашение вида:
```
psql (14.2)
Type "help" for help.

postgres=#
```

Проверим, создались ли таблицы из дампа:
```
postgres-# \dt
            List of relations
 Schema |     Name     | Type  |  Owner
--------+--------------+-------+----------
 public | battles      | table | postgres
 public | classes      | table | postgres
 public | company      | table | postgres
 public | income       | table | postgres
 public | income_o     | table | postgres
 public | laptop       | table | postgres
 public | outcome      | table | postgres
 public | outcome_o    | table | postgres
 public | outcomes     | table | postgres
 public | pass_in_trip | table | postgres
 public | passenger    | table | postgres
 public | pc           | table | postgres
 public | printer      | table | postgres
 public | product      | table | postgres
 public | ships        | table | postgres
 public | trip         | table | postgres
 public | utb          | table | postgres
 public | utq          | table | postgres
 public | utv          | table | postgres
(19 rows)
```



## Узнать больше
- [PostgreSQL on Docker hub](https://hub.docker.com/_/postgres)
- [Dockerize PostgreSQL](https://docs.docker.com/samples/postgresql_service/)
- [Установка и настройка PostgreSQL в Docker](https://selectel.ru/blog/postgresql-docker-setup/)
