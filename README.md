# CRM — запуск системы

Docker Compose для запуска CRM целиком: BFF, два микросервиса, PostgreSQL и Redis.

## Архитектура

```
Фронт (Next.js :3000)
        │ HTTP + cookie
        ▼
crm-gateway (BFF, :3001) ── Better Auth, уведомления, чат, BullMQ
        │ TCP                     │ TCP
        ▼                         ▼
crm-users-service (:4002)   crm-leads-service (:4003)
        │                         │
     users_db                  leads_db
```

| Репозиторий                                                           | Что делает                                                     |
| --------------------------------------------------------------------- | -------------------------------------------------------------- |
| [crm-gateway](https://github.com/kolya-zalev/crm-gateway)             | BFF: HTTP API для фронта, авторизация, уведомления и чат (SSE) |
| [crm-users-service](https://github.com/kolya-zalev/crm-users-service) | сотрудники, роли, команда                                      |
| [crm-leads-service](https://github.com/kolya-zalev/crm-leads-service) | лиды, заметки, задачи, активности, аналитика                   |
| [crm-shared](https://github.com/kolya-zalev/crm-shared)               | общие контракты (npm-пакет, клонировать не нужно)              |
| **crm-infra**                                                         | этот репозиторий — запуск всего вместе                         |

## Быстрый старт

1. Склонировать репозитории **в одну папку, рядом друг с другом**:

   ```bash
   mkdir crm && cd crm
   git clone https://github.com/kolya-zalev/crm-infra.git
   git clone https://github.com/kolya-zalev/crm-gateway.git
   git clone https://github.com/kolya-zalev/crm-users-service.git
   git clone https://github.com/kolya-zalev/crm-leads-service.git
   ```

   Compose собирает сервисы из соседних папок (`../crm-gateway` и т.д.).

2. Секрет:

   ```bash
   cd crm-infra
   cp .env.example .env
   # BETTER_AUTH_SECRET=$(openssl rand -base64 32)
   ```

3. Запуск:
   ```bash
   docker compose up -d --build
   ```
   Первая сборка — несколько минут. Миграции применяются автоматически при старте каждого сервиса.

## Адреса

| Что        | Адрес                                                                                |
| ---------- | ------------------------------------------------------------------------------------ |
| API        | http://localhost:3001                                                                |
| Swagger    | http://localhost:3001/api/docs                                                       |
| PostgreSQL | localhost:5432 (`crm` / `crm123`) — базы `crm_db`, `auth_db`, `users_db`, `leads_db` |
| Redis      | localhost:6379                                                                       |

Микросервисы наружу не открыты — доступны только из docker-сети.

## Полезные команды

```bash
docker compose ps                     # статус
docker logs -f crm-backend            # логи gateway
docker compose up -d --build api      # пересобрать один сервис
docker compose down                   # остановить (данные сохраняются)
```

⚠️ `docker compose down -v` **удаляет тома — все данные всех баз**.

## Заметки

- `name: crm-server` в compose — имя проекта сохранено после переезда из монорепо, чтобы использовались существующие тома `crm-server_pgdata` и `crm-server_redisdata`.
- `postgres-init.sql` создаёт базы только при **первом** запуске на пустом томе.
