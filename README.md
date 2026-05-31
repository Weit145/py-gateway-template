# Py Gateway Template

Это шаблон и каркас HTTP gateway-сервиса на Python. Репозиторий можно брать за основу для новых gateway-разработок, чтобы не собирать каждый раз с нуля базовую структуру проекта, запуск, конфигурацию, логирование, HTTP API, работу с PostgreSQL и миграции.

## Для чего нужен шаблон

- Быстро стартовать новый gateway-сервис на FastAPI.
- Использовать готовую структуру `app`, `core`, `domain`, `usecase`, `transport`, `repositories`.
- Публиковать HTTP API поверх прикладного слоя.
- Работать с PostgreSQL через асинхронный SQLAlchemy.
- Держать единый подход к Docker, env-конфигурации, логированию и миграциям.
- Использовать встроенную OpenAPI-документацию FastAPI.

## Что уже есть

- HTTP-сервер на `FastAPI` и `uvicorn`.
- Роутинг через `APIRouter`.
- Конфигурация через переменные окружения и `.env`.
- Настройка логирования через `logging`.
- Асинхронное подключение к PostgreSQL через `SQLAlchemy` и `asyncpg`.
- Репозиторий для работы с пользователями.
- Usecase-слой для бизнес-логики.
- Endpoint для создания пользователя.
- Служебный endpoint для проверки доступности приложения.
- OpenAPI, Swagger UI и ReDoc из коробки.
- Каркас Alembic для миграций.
- Dockerfile и docker-compose для локального запуска.
- MakeFile с базовыми командами.

## Структура проекта

```text
app/
  main.py                                      # точка входа приложения
  core/
    config.py                                 # чтение конфигурации
    logging.py                                # настройка логирования
  domain/                                     # доменные сущности
  usecase/
    service.py                                # прикладная логика
  transport/
    api/v1/
      handler/                                # HTTP handlers
      schemas/                                # Pydantic-схемы запросов и ответов
  repositories/
    storage/
      models/                                 # SQLAlchemy-модели
      postgres/                               # подключение к БД и репозитории
migrations/                                   # Alembic-миграции
alembic.ini
Dockerfile
docker-compose.yml
MakeFile
.env.example
pyproject.toml
poetry.lock
```

## Быстрый старт

Скопируйте пример переменных окружения:

```bash
cp .env.example .env
```

Запустите сервис через Docker Compose:

```bash
make up
```

Или пересоберите контейнер перед запуском:

```bash
make build
```

После запуска API будет доступно на:

```text
http://localhost:8000
```

Swagger UI:

```text
http://localhost:8000/docs
```

ReDoc:

```text
http://localhost:8000/redoc
```

OpenAPI JSON:

```text
http://localhost:8000/openapi.json
```

## Локальный запуск без контейнера приложения

Установите зависимости:

```bash
poetry install
```

Поднимите PostgreSQL:

```bash
docker compose up db
```

Для запуска приложения локально укажите адрес базы через `localhost`, например:

```bash
DATABASE_URL=postgresql+asyncpg://postgres:postgres@localhost:5432/postgres
```

Примените миграции:

```bash
poetry run alembic upgrade head
```

Запустите приложение:

```bash
poetry run uvicorn app.main:app --reload
```

## Доступные endpoints

```text
GET /_info
```

Проверяет доступность приложения. Возвращает HTTP-статус `200`.

```text
POST /users
```

Создаёт пользователя.

Пример запроса:

```json
{
  "name": "Ivan"
}
```

Пример ответа:

```json
{
  "id": "7c179b02-6e77-4d5d-9cb1-8a4dc7b3fb4f",
  "name": "Ivan",
  "created_at": "2026-05-31T12:00:00Z"
}
```

Если пользователь с таким `name` уже существует, сервис возвращает `409 Conflict`.

## Переменные окружения

| Переменная | Значение по умолчанию | Описание |
| --- | --- | --- |
| `DATABASE_URL` | `postgresql+asyncpg://postgres:postgres@db:5432/postgres` | DSN для подключения к PostgreSQL |
| `LOG_LEVEL` | `INFO` | Уровень логирования |

Пример конфигурации находится в `.env.example`.

## Команды MakeFile

```bash
make up
```

Запустить сервис через Docker Compose без пересборки образа.

```bash
make build
```

Пересобрать образ и запустить сервис.

```bash
make down
```

Остановить контейнеры.

```bash
make delete
```

Остановить контейнеры и удалить volumes/orphans.

## Миграции

В проекте подключен Alembic. Docker Compose перед стартом приложения выполняет:

```bash
alembic upgrade head
```

Создать новую миграцию можно командой:

```bash
poetry run alembic revision --autogenerate -m "add users table"
```

Применить миграции локально:

```bash
poetry run alembic upgrade head
```

Файлы миграций хранятся в `migrations/versions`.

## Как использовать для нового gateway

1. Скопировать репозиторий как основу нового сервиса.
2. Переименовать проект в `pyproject.toml`.
3. Настроить `.env.example` под нужные сервисы и базу данных.
4. Добавить доменные сущности в `app/domain`.
5. Добавить SQLAlchemy-модели в `app/repositories/storage/models`.
6. Добавить репозитории в `app/repositories/storage/postgres`.
7. Добавить usecase-логику в `app/usecase`.
8. Добавить HTTP handlers и схемы в `app/transport/api/v1`.
9. Создать и применить Alembic-миграции.
10. Проверить запуск через `make build`.

## Архитектурная идея

Gateway принимает HTTP-запросы, валидирует их через Pydantic-схемы, передаёт управление в usecase-слой, работает с данными через репозитории и возвращает наружу HTTP-ответы. В шаблоне уже разведены основные слои:

- `transport/api` отвечает за HTTP-слой;
- `usecase` содержит прикладную логику;
- `repositories` изолирует доступ к хранилищу;
- `domain` хранит доменные сущности;
- `core` содержит общую инфраструктуру приложения.

Такой каркас удобно расширять под новые домены, не смешивая HTTP handlers, бизнес-логику, модели хранения и конфигурацию в одном месте.

## Лицензия

Проект распространяется под лицензией MIT. Подробнее: [`LICENSE`](LICENSE).
