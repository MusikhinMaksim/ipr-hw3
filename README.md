## 6 баллов. Запуск через Docker Compose

Из папки hw_3_template было выполнено:

```bash
cp .env.example .env
docker compose up -d --build
```

Настройки заданы в `.env`:

- `APP_PORT` — порт приложения на компьютере, по умолчанию `8080`.
- `GREETING` — приветствие приложения. В `.env.example` задано
  `Hello from Maksim!`. Если переменная не задана или пустая,
  используется `Hello, Docker!`.

### Проверка приложения и сохранения данных

Выполнены запросы:

```bash
curl -fsS http://127.0.0.1:8080/
curl -fsS -X POST http://127.0.0.1:8080/increment
curl -fsS http://127.0.0.1:8080/readyz
```

Результаты:

- Приветствие: `Hello from Maksim!`.
- Значение счётчика: `0` -> `1`.
- Проверка готовности: `{"status": "ready", "storage": "sqlite"}`.

Затем контейнер был удалён и создан заново:

```bash
docker compose down
docker compose up -d
curl -fsS http://127.0.0.1:8080/
```

Значение счётчика осталось `1`, а hostname изменился с `01b1f4c0643e` на `cfc1d5a86be8`.

## 8 баллов. Redis

Теперь приложение использует Redis. В compose.yaml были добавлены строки:

- `STORAGE=redis`
- `REDIS_URL=redis://redis:6379/0`

Для обоих сервисов был настроен healthcheck.

### Запуск и проверка сохранения данных

```bash
docker compose up -d --build
docker compose ps
curl -fsS http://127.0.0.1:8080/
curl -fsS -X POST http://127.0.0.1:8080/increment
curl -fsS -X POST http://127.0.0.1:8080/increment
```

Счетчик изменился с `0` до `2`. В ответах указано `"storage": "redis"`.
Далее контейнер был удален и создан заново:

```bash
docker compose down
docker compose up -d
docker compose ps
curl -fsS http://127.0.0.1:8080/
```

Счетчик всё также равен `2`, а hostname изменился с `f091fee03d13` на `b241416592b3`.

### Проверка отключения Redis

```bash
docker compose stop redis
curl -s -o /dev/null -w '%{http_code}\n' http://127.0.0.1:8080/healthz
curl -s -o /dev/null -w '%{http_code}\n' http://127.0.0.1:8080/readyz
curl -s -o /dev/null -w '%{http_code}\n' -X POST http://127.0.0.1:8080/increment
```

Были получены HTTP-коды:

- `GET /healthz` — `200`.
- `GET /readyz` — `503`.
- `POST /increment` — `503`.

После этого Redis был вновь запущен:

```bash
docker compose start redis
curl -fsS http://127.0.0.1:8080/
curl -fsS -X POST http://127.0.0.1:8080/increment
```

GET вернул `2`, POST увеличил счетчик на `1`, hostname не изменился.