# Документация по проекту Warriors API (PR2)

## Технический стек

- Python 3.x
- FastAPI
- SQLModel (ORM)
- PostgreSQL
- Docker

## Модели данных

### Warrior (Воин)

```python
class Warrior:
    id: int
    race: RaceType
    name: str
    level: int
    profession_id: Optional[int]
    profession: Optional[Profession]
    skills: Optional[List[Skill]]
```

### Profession (Профессия)

```python
class Profession:
    id: int
    title: str
    description: str
    warriors_prof: List[Warrior]
```

### Skill (Навык)

```python
class Skill:
    id: int
    name: str
    description: str
    warriors: Optional[List[Warrior]]
```

### RaceType (Тип расы)

```python
class RaceType(Enum):
    director = "director"
    worker = "worker"
    junior = "junior"
```

## API Endpoints

### Управление воинами

```http
POST /warrior - Создать нового воина
GET /warriors_list - Получить список всех воинов
GET /warrior/{warrior_id} - Получить информацию о воине
PATCH /warrior/{warrior_id} - Обновить информацию о воине
DELETE /warrior/delete/{warrior_id} - Удалить воина
POST /warrior/{warrior_id}/add_skill/{skill_id} - Добавить навык воину
```

### Управление профессиями

```http
GET /professions_list - Получить список всех профессий
GET /profession/{profession_id} - Получить информацию о профессии
POST /profession - Создать новую профессию
```

### Управление навыками

```http
GET /skills_list - Получить список всех навыков
GET /skill/{skill_id} - Получить информацию о навыке
POST /skill - Создать новый навык
PATCH /skill/{skill_id} - Обновить информацию о навыке
DELETE /skill/delete/{skill_id} - Удалить навык
```
