# Документация по проекту Parser Service

## Содержание
1. [Общее описание](#общее-описание)
2. [Структура проекта](#структура-проекта)
3. [Компоненты системы](#компоненты-системы)
4. [API Endpoints](#api-endpoints)
5. [Тестирование](#тестирование)
6. [Работа с Docker](#работа-с-docker)

## Общее описание

Проект представляет собой микросервисное приложение для парсинга веб-страниц с использованием FastAPI, Celery, Redis и PostgreSQL. Система позволяет выполнять как синхронный, так и асинхронный парсинг URL-адресов с сохранением результатов в базе данных и кэше.

### Основные возможности:
- Синхронный и асинхронный парсинг веб-страниц
- Кэширование результатов в Redis
- Сохранение истории парсинга в PostgreSQL
- Периодические задачи через Celery Beat
- Микросервисная архитектура
- Docker-контейнеризация всех компонентов

## Структура проекта

```
lr3/
├── app/
│   ├── api/              # API endpoints и маршрутизация
│   │   ├── __init__.py
│   │   └── main.py
│   ├── core/            # Базовая конфигурация
│   │   ├── __init__.py
│   │   ├── database.py  # Настройки базы данных
│   │   └── cache.py     # Настройки Redis
│   ├── models/          # Модели базы данных
│   │   ├── __init__.py
│   │   └── parse_result.py
│   ├── schemas/         # Pydantic модели
│   │   ├── __init__.py
│   │   └── parser.py
│   ├── services/        # Сервисы
│   │   ├── __init__.py
│   │   └── parser_service.py
│   ├── worker/          # Celery воркер
│   │   ├── __init__.py
│   │   └── celery_worker.py
│   └── __init__.py
├── docker/             # Docker конфигурации
│   ├── Dockerfile.web
│   ├── Dockerfile.parser
│   └── Dockerfile.celery
├── docker-compose.yml
└── requirements.txt
```

## Компоненты системы

### 1. Parser Service (app/services/parser_service.py)
- Микросервис для парсинга веб-страниц
- Извлекает заголовок, контент и метаданные
- Сохраняет результаты в БД и кэш
- Предоставляет API для проверки кэша и истории

### 2. База данных (app/core/database.py)
- PostgreSQL для хранения результатов
- Модель данных включает:
  - URL
  - Заголовок
  - Контент
  - Метаданные
  - Время создания

### 3. Кэширование (app/core/cache.py)
- Redis для кэширования результатов
- Время жизни кэша: 1 час
- Автоматическая инвалидация

### 4. Celery Worker (app/worker/celery_worker.py)
- Асинхронная обработка задач
- Периодические задачи
- Интеграция с Redis как брокером

## API Endpoints

### Parser Service (порт 8001)

#### 1. Синхронный парсинг
```http
POST /parse
Content-Type: application/json

{
    "url": "https://example.com"
}
```

Ответ:
```json
{
    "url": "https://example.com",
    "title": "Example Domain",
    "content": "...",
    "metadata": {
        "description": "...",
        "keywords": "...",
        "links": 5,
        "images": 2
    }
}
```

#### 2. Проверка кэша
```http
GET /cache/{url}
```

Ответ:
```json
{
    "cached": true,
    "data": {
        "url": "https://example.com",
        "title": "Example Domain",
        ...
    }
}
```

#### 3. История парсинга
```http
GET /history
```

Ответ:
```json
[
    {
        "id": 1,
        "url": "https://example.com",
        "title": "Example Domain",
        "content": "...",
        "metadata": {...},
        "created_at": "2024-05-25T20:14:05.002802"
    }
]
```

### Web Service (порт 8000)

#### 1. Асинхронный парсинг
```http
POST /parse/async
Content-Type: application/json

{
    "url": "https://example.com"
}
```

Ответ:
```json
{
    "task_id": "3f92726b-4d08-4562-b6ad-38fee1be570f",
    "message": "Parsing task submitted successfully"
}
```

#### 2. Статус задачи
```http
GET /parse/status/{task_id}
```

Ответ:
```json
{
    "task_id": "3f92726b-4d08-4562-b6ad-38fee1be570f",
    "message": "Task completed",
    "result": {...}
}
```

## Тестирование

### Предварительные условия
1. Установленный Docker и Docker Compose
2. Клонированный репозиторий
3. Терминал в директории проекта

### Запуск тестов

1. Запуск всех сервисов:
```bash
docker-compose up --build
```

2. Тест синхронного парсинга:
```bash
curl -X POST "http://localhost:8001/parse" \
     -H "Content-Type: application/json" \
     -d '{"url": "https://www.python.org"}'
```

3. Тест асинхронного парсинга:
```bash
# Создание задачи
curl -X POST "http://localhost:8000/parse/async" \
     -H "Content-Type: application/json" \
     -d '{"url": "https://www.python.org"}'

# Проверка статуса (подставьте полученный task_id)
curl "http://localhost:8000/parse/status/{task_id}"
```

4. Проверка кэша:
```bash
curl "http://localhost:8001/cache/https://www.python.org"
```

5. Проверка истории:
```bash
curl "http://localhost:8001/history"
```

### Ожидаемые результаты

1. Синхронный парсинг должен вернуть результаты немедленно
2. Асинхронный парсинг должен вернуть task_id
3. Повторный запрос того же URL должен вернуться быстрее (из кэша)
4. История должна содержать все выполненные парсинги
5. Celery Beat должен автоматически парсить заданный URL каждый час

## Работа с Docker

### Управление контейнерами

1. Запуск всех сервисов:
```bash
docker-compose up --build
```

2. Остановка сервисов:
```bash
docker-compose down
```

3. Просмотр логов:
```bash
# Все логи
docker-compose logs

# Логи конкретного сервиса
docker-compose logs parser
```

4. Перезапуск отдельного сервиса:
```bash
docker-compose restart parser
```

### Мониторинг

1. Статус контейнеров:
```bash
docker-compose ps
```

2. Использование ресурсов:
```bash
docker stats
```

### Отладка

1. Подключение к контейнеру:
```bash
docker-compose exec parser bash
```

2. Просмотр логов Redis:
```bash
docker-compose exec redis redis-cli monitor
```

3. Подключение к PostgreSQL:
```bash
docker-compose exec db psql -U postgres -d fastapi_db
``` 