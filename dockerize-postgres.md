# PostgreSQL в Docker-контейнере

PostgreSQL имеет официальный образ на [Docker Hub](https://hub.docker.com/_/postgres), доступный в различных вариантах. Представленные по ссылке тэги позволяют выбрать основные весрии PostgreSQL от 10 до 14 версии, а также выбрать операционную систему, на которой основан соответствующий образ: Alpine, Debian Stretch и Debian Bullseye.

В данном туториале мы будем использовать сборку на Alpine с тэгом `postgres:14-alpine`, но вы можете выбрать любой другой, наиболее подходящий вашим потребностям.

## Запуск контейнера с помощью `docker run`
Создать и запустить наш PostgreSQL контейнер можно с помощью команды `docker run`:

```bash
docker run -d --name postgres -p 5432:5432 -e POSTGRES_PASSWORD=<password> -v postgres:/var/lib/postgresql/data postgres:14-alpine
```

Образ PostgreSQL использует несколько переменных окружения, большинство из которых можно опустить. Единственной **обязательной** переменной является `POSTGRES_PASSWORD`. Эта переменная будет определять пароль для дефолтного суперпользователя с именем `postgres`. Имя пользователя также может быть задано через переменную `POSTGRES_USER`. Все эти переменные мы передаем с использованием флага `-e`.

Флаг `-v` используется для монтирования Docker-тома к папке с данными контейнера. Docker создаст или перемонтирует том, если он уже создан. Использование тома необходимо для хранения данных Postgres вне контейнера, в противном случае данные будут утеряны при остановке / перезапуске контейнера.

По умолчанию PostgreSQL слушает порт 5432. С помощью флага `-p` мы пробрасываем порт 5432 изнутри контейнера в наш Docker хост.

Флаг `-d` используется для запуска контейнера в detached режиме, что запускает его в фоновом режиме до тех пор, пока он не будет остановлен командой `docker stop`.

Имя создаваемого нами контейнера, очевидно, указывается флагом `--name`.

## Запуск контейнера с помощью `docker-compose`
Использвание однострочной комманды `docker run` для запуска PostgreSQL может показаться неудобным, особенно когда мы будем указывать новые параметры запуска.
Создадим файл `docker-compose.yaml` и заполним его следующим содержимым:
```
version: '3.9'

services:
  postgres:
    image: postgres:14-alpine
    container_name: postgres
    ports:
      - 5432:5432
    volumes:
      - postgres_data:/var/lib/postgresql/data/
    env_file:
      - ./.env

volumes:
  postgres_data:
```
В данном случае переменные окружения мы передаем из `.env` файла, расположенного в той же директории, что и создаваемый нами `docker-compose.yaml` файл. Напомню, что единственной **обязательной** переменной является `POSTGRES_PASSWORD`. Про остальные переменные вы можете почитать на [Docker hub](https://hub.docker.com/_/postgres) в разделе Environment Variables.

Содержание `.env`:
```
POSTGRES_PASSWORD=<password>
```

Запустим контейнер с помощью созданного файла:
```
docker compose -f docker-compose.yaml up -d --build
```


## Подключение к базе данных
Учитывая то, что при запуске контейнера мы пробрасывали порт 5432, подключиться к базе можно по адресу `localhost:5432`, используя любой совместимый клиент и данные учетной записи пользователя с которыми был инициализирован контейнер (пароль суперпользователя, его имя и имя базы данных, если указывались явно).

Docker образ, использованный нами, включает в себя бинарник `psql`, запустить который можно используя [docker exec](https://docs.docker.com/engine/reference/commandline/exec/):
```
docker exec -it postgres psql -U postgres
```

или [docker compose exec](https://docs.docker.com/compose/reference/exec/):
```
docker compose -f docker-compose.yaml exec postgres psql -U postgres
```
Запустив `psql`, проверим версию PostgreSQL:
```
SELECT version();
```

## Узнать больше
- [PostgreSQL on Docker hub](https://hub.docker.com/_/postgres)
- [Dockerize PostgreSQL](https://docs.docker.com/samples/postgresql_service/)
- [Установка и настройка PostgreSQL в Docker](https://selectel.ru/blog/postgresql-docker-setup/)
