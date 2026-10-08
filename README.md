# Kittygram

[![Main Kittygram workflow](https://github.com/lisa-evt/kittygram_final/actions/workflows/main.yml/badge.svg)](https://github.com/lisa-evt/kittygram_final/actions/workflows/main.yml)

[![Python](https://img.shields.io/badge/Python-3.12-blue?logo=python)](https://www.python.org/)
[![Django](https://img.shields.io/badge/Django-5.1.1-green?logo=django)](https://www.djangoproject.com/)
[![DRF](https://img.shields.io/badge/DRF-3.15.2-red)](https://www.django-rest-framework.org/)

## Описание

**Kittygram** — социальная сеть для владельцев котиков. Пользователь регистрируется, добавляет своих питомцев (имя, цвет, год рождения, фото, достижения) и смотрит котиков других пользователей.

Проект решает простую задачу: хранить и показывать коллекцию питомцев в одном месте, вместо россыпи фотографий в мессенджерах. Данные доступны через веб-интерфейс и через REST API.

**Технологии**

- Backend: Python, Django, Django REST Framework, PostgreSQL
- Frontend: React
- Веб-сервер: nginx (gateway)
- Инфраструктура: Docker, Docker Compose
- CI/CD: GitHub Actions (проверка flake8, тесты backend и frontend, публикация образов на Docker Hub, автоматический деплой на сервер, уведомление в Telegram)

## Установка

Для запуска нужны [Git](https://git-scm.com/) и [Docker](https://docs.docker.com/get-docker/) с плагином Docker Compose.

**1. Клонируйте репозиторий**

```bash
git clone https://github.com/<ВАШ_ЛОГИН_GITHUB>/kittygram_final.git
cd kittygram_final
```

**2. Создайте файл `.env`** в корне проекта по образцу `.env.example`:

```env
SECRET_KEY=ваш_секретный_ключ
DEBUG=True
ALLOWED_HOSTS=localhost,127.0.0.1

POSTGRES_DB=django_db
POSTGRES_USER=django_user
POSTGRES_PASSWORD=придумайте_пароль
DB_HOST=db
DB_PORT=5432
```

Сгенерировать `SECRET_KEY` можно командой:

```bash
python -c "from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())"
```

**3. Соберите и запустите контейнеры**

```bash
docker compose up -d --build
```

**4. Выполните миграции и соберите статику**

```bash
docker compose exec backend python manage.py migrate
docker compose exec backend python manage.py collectstatic --no-input
docker compose exec backend cp -r /app/collected_static/. /backend_static/static/
```

**5. (Опционально) Создайте суперпользователя**

```bash
docker compose exec backend python manage.py createsuperuser
```

После этого проект доступен по адресу http://localhost:9000/, админ-панель — http://localhost:9000/admin/, API — http://localhost:9000/api/.

**Остановить проект**

```bash
docker compose down
```

### Деплой на сервер

Деплой настроен через GitHub Actions: при пуше в `main` workflow прогоняет тесты, собирает и публикует образы на Docker Hub, копирует `docker-compose.production.yml` на сервер и перезапускает контейнеры. Для этого в **Settings → Secrets and variables → Actions** нужны секреты: `DOCKER_USERNAME`, `DOCKER_PASSWORD`, `HOST`, `USER`, `SSH_KEY`, `DJANGO_SECRET_KEY`, `TELEGRAM_TO`, `TELEGRAM_TOKEN`. На сервере в папке проекта должен лежать свой `.env` (с `DEBUG=False` и `ALLOWED_HOSTS` с вашими доменами).

## Примеры запросов к API

Ниже `http://localhost:9000` — адрес локального запуска; на сервере используйте свой домен. Список эндпоинтов и формат полей сверьте со своим проектом.

**Регистрация пользователя**

```bash
curl -X POST http://localhost:9000/api/users/ \
  -H "Content-Type: application/json" \
  -d '{"username": "cat_lover", "password": "strong_password_123"}'
```

**Получение токена**

```bash
curl -X POST http://localhost:9000/api/token/login/ \
  -H "Content-Type: application/json" \
  -d '{"username": "cat_lover", "password": "strong_password_123"}'
```

Ответ:

```json
{"auth_token": "9a1b2c3d4e5f..."}
```

**Список котиков**

```bash
curl http://localhost:9000/api/cats/ \
  -H "Authorization: Token 9a1b2c3d4e5f..."
```

**Добавление котика**

```bash
curl -X POST http://localhost:9000/api/cats/ \
  -H "Authorization: Token 9a1b2c3d4e5f..." \
  -H "Content-Type: application/json" \
  -d '{"name": "Барсик", "color": "#808080", "birth_year": 2021, "achievements": []}'
```

Пример ответа:

```json
{
  "id": 1,
  "name": "Барсик",
  "color": "#808080",
  "birth_year": 2021,
  "achievements": [],
  "owner": 1,
  "age": 5,
  "image": null,
  "image_url": null
}
```

**Список достижений**

```bash
curl http://localhost:9000/api/achievements/ \
  -H "Authorization: Token 9a1b2c3d4e5f..."
```

**Выход (удаление токена)**

```bash
curl -X POST http://localhost:9000/api/token/logout/ \
  -H "Authorization: Token 9a1b2c3d4e5f..."
```
