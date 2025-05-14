# Документация проекта Hackathon System

## Описание проекта

Hackathon System - это система управления хакатонами, которая позволяет организовывать и проводить хакатоны, управлять командами участников, заданиями и оценивать результаты.

## Технический стек

- Python 3.x
- FastAPI (веб-фреймворк)
- SQLModel (ORM)
- PostgreSQL (база данных)
- Alembic (система миграций)
- JWT (аутентификация)

## Структура проекта

```
lr1/
├── alembic.ini            # Конфигурация Alembic
├── connection.py          # Настройки подключения к БД
├── main.py               # Основной файл приложения
├── controllers/          # Контроллеры API
├── models/              # Модели данных
├── migrations/          # Миграции базы данных
└── util/               # Вспомогательные утилиты
```

## Модели данных

### User (Пользователь)

```python
class User:
    id: int
    username: str        # Уникальное имя пользователя
    hashed_password: str # Хэшированный пароль
    email: str          # Уникальный email
    bio: str           # Описание пользователя (опционально)
```

### Team (Команда)

```python
class Team:
    id: int
    team_name: str      # Уникальное название команды
    description: str    # Описание команды (опционально)
    creator_id: int     # ID создателя команды
```

### TeamMembership (Участие в команде)

```python
class TeamMembership:
    id: int
    team_id: int       # ID команды
    user_id: int       # ID пользователя
    role: str         # Роль в команде
    join_date: str    # Дата присоединения
```

### Hackathon (Хакатон)

```python
class Hackathon:
    id: int
    event_name: str    # Название мероприятия
    description: str   # Описание хакатона
    start_date: datetime # Дата начала
    end_date: datetime  # Дата окончания
```

### Task (Задание)

```python
class Task:
    id: int
    title: str        # Название задания
    description: str  # Описание задания
    requirements: str # Требования к выполнению
    criteria: str    # Критерии оценки
    deadline: datetime # Срок выполнения
    hackathon_id: int # ID хакатона
```

### Submission (Решение)

```python
class Submission:
    id: int
    description: str  # Описание решения
    file_url: str    # Ссылка на файл решения
    submitted_at: datetime # Время отправки
    evaluation: float # Оценка решения
    task_id: int     # ID задания
    user_id: int     # ID пользователя
    team_id: int     # ID команды (опционально)
```

## API Endpoints

### Аутентификация

```http
POST /users/register - Регистрация нового пользователя
POST /users/login - Вход в систему
```

### Управление пользователями

```http
GET /users/{user_id} - Получение информации о пользователе
PUT /users/{user_id}/password - Обновление пароля пользователя
```

### Управление командами

```http
POST /teams - Создание новой команды
GET /teams - Получение списка команд
GET /teams/{team_id} - Получение информации о команде
```

### Управление участием в командах

```http
POST /memberships - Присоединение к команде
GET /memberships - Получение списка участий в командах
```

### Управление хакатонами

```http
POST /hackathons - Создание хакатона
GET /hackathons - Получение списка хакатонов
GET /hackathons/{hackathon_id} - Получение информации о хакатоне
```

### Управление заданиями

```http
POST /tasks - Создание задания
GET /tasks - Получение списка заданий
GET /tasks/{task_id} - Получение информации о задании
```

### Управление решениями

```http
POST /submissions - Отправка решения
GET /submissions - Получение списка решений
GET /submissions/{submission_id} - Получение информации о решении
PUT /submissions/{submission_id}/evaluate - Оценка решения
```

## Безопасность

Система использует JWT (JSON Web Tokens) для аутентификации пользователей. Все конфиденциальные данные, такие как пароли, хэшируются перед сохранением в базе данных. API защищен с помощью механизма Bearer-токенов.

## База данных

Проект использует PostgreSQL в качестве основной базы данных. Миграции управляются с помощью Alembic, что обеспечивает версионирование схемы базы данных.

### Подключение к базе данных

```python
DATABASE_URL = "postgresql://postgres:password@localhost:5432/hackathon_db"
```

## Разработка

### Установка зависимостей

```bash
pip install fastapi sqlmodel alembic psycopg2-binary python-jose[cryptography] python-multipart
```

### Запуск приложения

```bash
uvicorn main:app --reload
```

### Управление миграциями

```bash
# Создание новой миграции
alembic revision --autogenerate -m "description"

# Применение миграций
alembic upgrade head
```
