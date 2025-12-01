 # Hotel Admin System

Панель администратора для гостиницы: бэкенд на **Django REST Framework** (со Swagger), фронтенд на **React + Vite + MUI**. Система хранит номера, клиентов, сотрудников, проживание и строит квартальные отчёты. Здесь собрана шпаргалка по запуску и структуре проекта.

---

## Быстрый запуск (tl;dr)

```bash
# Backend
cd /Users/vladimirsobol/Documents/web/labLexa
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python manage.py migrate
python manage.py seed_hotel   # опционально: заполняем демо-данные
python manage.py runserver    # http://127.0.0.1:8000/, Swagger: /api/docs/

# Frontend (отдельное окно терминала)
nvm use 20.19.0               # или любая Node >= 20.19
cd /Users/vladimirsobol/Documents/web/labLexa/frontend
cp .env.example .env          # при необходимости поправьте VITE_API_BASE_URL
npm install
npm run dev                   # http://localhost:5173/ (можно --host)
```

После этого UI сразу увидит демо-данные (seed) и будет стучаться на backend через CORS.

---

## Структура репозитория

```
labLexa/
├─ config/                 # Django-проект (настройки, urls, wsgi/asgi)
├─ rooms/                  # Модели и API номеров
├─ clients/                # Клиенты и их проживание
├─ stays/                  # Связь клиент ↔ номер (check-in/out, отчёты)
├─ employees/              # Персонал и графики уборки
├─ reports/                # Квартальные отчёты + seed-команда
├─ common/                 # Вспомогательные утилиты (парсинг дат)
├─ frontend/               # React + Vite + MUI панель администратора
├─ requirements.txt        # Python-зависимости
├─ manage.py               # Django CLI
└─ README.md               # этот файл
```

### Что интересного во `frontend/src`?

```
src/
├─ app/           # Redux store + MUI тема
├─ components/    # Layout и прочие общие блоки
├─ pages/         # Dashboard, Rooms, Clients, Stays, Employees, Reports
├─ services/      # RTK Query API (src/services/api.ts)
├─ types.ts       # Общие DTO (синхронизированы со Swagger)
└─ main.tsx       # Точка входа (Redux, Router, Theme)
```

---

## Настройка окружения и переменные

### Python / Django

`.env` в корне (пример уже есть). Ключевые параметры:
```
DJANGO_SECRET_KEY=dev-secret-key
DEBUG=True
USE_SQLITE=True                 # по умолчанию SQLite
ALLOWED_HOSTS=*
POSTGRES_*                      # заполнить если нужен Postgres
CORS_ALLOWED_ORIGINS=http://localhost:5173,http://127.0.0.1:5173
```

Если переключаемся на Postgres — ставим `USE_SQLITE=False` и прописываем креды.

### Node / Vite

- Требуется Node **20.19.0** или новее (иначе Vite 7 ругается).
- В `frontend/.env.example`:
  ```
  VITE_API_BASE_URL=http://127.0.0.1:8000/api
  ```
  Скопируйте в `.env`, если нужно переопределить адрес API.

---

## Основные команды

### Backend

| Команда | Описание |
|---------|----------|
| `python manage.py migrate` | Применить миграции |
| `python manage.py seed_hotel` | Заполнить базу демо-данными |
| `python manage.py runserver` | Запуск API (`http://127.0.0.1:8000/`) |
| `python manage.py createsuperuser` | Создать администратора |

**Swagger / API документация**
- JSON-схема: `http://127.0.0.1:8000/api/schema/`
- Swagger UI: `http://127.0.0.1:8000/api/docs/`
- Redoc: `http://127.0.0.1:8000/api/redoc/`

### Frontend

```bash
cd frontend
npm install
npm run dev         # dev-сервер
npm run build       # production сборка (dist/)
```

---

## Seed-команда (`seed_hotel`)

- Файл: `reports/management/commands/seed_hotel.py`
- Создаёт 5 номеров, 4 клиентов, 2 сотрудников с расписанием и несколько проживаний (часть активны).
- Завязана на `get_or_create`, так что переиспользовать можно безопасно.

Пересоздать базу с нуля:
```bash
rm db.sqlite3
python manage.py migrate
python manage.py seed_hotel
```

---

## Кратко про API и UI

| Модуль | Описание | Эндпоинты |
|--------|----------|-----------|
| Rooms | список номеров, свободные места, история | `GET /rooms/`, `GET /rooms/free-count/`, `GET /rooms/{id}/clients?start&end` |
| Clients | реестр клиентов, заселение/выселение, «соседи» | `POST /clients/`, `GET /clients/{id}/stays/`, `POST /stays/`, `POST /stays/{id}/checkout/`, `GET /clients/{id}/overlaps/` |
| Employees | персонал и расписания уборок | `POST /employees/`, `POST /employees/{id}/fire/`, `PUT /employees/{id}/schedule/`, `GET /employees/who-cleans/` |
| Stays | все проживание, фильтры по статусу | `GET /stays/?status=active` |
| Reports | квартальные агрегаты | `GET /reports/quarterly/?quarter=1&year=2025` |

Фронтенд страницы отражают эти сущности: Dashboard, Rooms, Clients, Stays, Employees, Reports.

---

## Полезные заметки

- **CORS** настроен: фронтенд (5173) общается с API без прокси.
- **SQLite** по умолчанию — удобно для локалки. Переключение на Postgres через `.env`.
- **RTK Query** автоматически инвалидации делает — после check-in/out UI обновляет списки.
- Вместо Vite dev-сервера можно собрать `npm run build` и раздавать `frontend/dist`.
- Если понадобятся Docker/CI/авторизация — модульная структура уже готова к расширениям.

Удачной эксплуатации! Если что-то пошло не так, всегда можно `seed_hotel` или пере-прогнать миграции. 