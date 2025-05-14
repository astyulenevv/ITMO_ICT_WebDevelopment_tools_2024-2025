# Документация по проекту Warriors API (PR3)

## Технический стек

- Python 3.x
- FastAPI
- SQLModel (ORM)
- PostgreSQL
- Docker
- Alembic (миграции)

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

### SkillWarriorLink (Связь навыков с воинами)

```python
class SkillWarriorLink:
    skill_id: Optional[int]
    warrior_id: Optional[int]
    level: int | None
```

### RaceType (Тип расы)

```python
class RaceType(Enum):
    director = "director"
    worker = "worker"
    junior = "junior"
```

## База данных

### Конфигурация

База данных управляется через переменные окружения:

```python
DB_ADMIN = 'postgresql://warrior:12345678@localhost:8432/warriors_db'
```

### Docker-конфигурация

```yaml
services:
  warriors-pr3-db:
    image: postgres:15
    environment:
      POSTGRES_USER: "warrior"
      POSTGRES_PASSWORD: "12345678"
      POSTGRES_DB: "warriors_db"
    ports:
      - "8432:5432"
```
