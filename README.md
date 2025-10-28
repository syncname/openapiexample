# Blog API (OpenAPI 3.0)

Краткое описание:
- **Назначение**: учебная спецификация OpenAPI для блога с JWT-аутентификацией, CRUD для постов, комментариями и пагинацией.
- **Где спецификация**: `docs/openapi/openapi.yaml` (разделена по файлам через `$ref`).
- **Мок-сервер**: Stoplight Prism.
- **UI**: Swagger UI.

## Быстрый старт (Docker Compose)


Переменные окружения (опционально):
- `PRISM_PORT` (по умолчанию `4010`)
- `SWAGGER_PORT` (по умолчанию `8080`)

Запуск тестового примера
```bash
SPEC_PATH=basic.yaml docker compose up -d
```

Запуск сервисов (по умолчанию multi-file `docs/openapi/openapi.yaml`):
```bash
docker compose up -d
```

Выбор спецификации через `SPEC_PATH` (относительно `docs/`):
- Multi-file (по умолчанию): ничего указывать не нужно
- Single-file: `docs/openapi.yaml`
- Произвольный путь: любой файл внутри `docs/`

Чтобы запустить контейнер с определенной спецификацией можно использовать
следующие команды в зависимости от версии docker compose
```bash
# single-file спецификация (файл в docs/)
SPEC_PATH=openapi.yaml docker compose up -d
# или
SPEC_PATH=openapi.yaml docker-compose up -d

# многофайловая структура (корневой файл в docs/openapi/)
SPEC_PATH=openapi/openapi.yaml docker compose up -d
# или
SPEC_PATH=openapi/openapi.yaml docker-compose up -d
```




Сервисы:
- Prism (мок API): http://localhost:${PRISM_PORT:-4010}
- Swagger UI: http://localhost:${SWAGGER_PORT:-8080}

Остановка:
```bash
docker compose down
```

Обновление образов:
```bash
docker compose pull && docker compose up -d
```

## Структура
- `docs/` — артефакты спецификации и документации.
  - `openapi/openapi.yaml` — корневой файл спецификации с ссылками на части.
  - `openapi/paths/*.yaml` — эндпоинты.
  - `openapi/components/**` — компоненты (schemas, parameters, headers, responses, securitySchemes).
- `docker-compose.yml` — запуск мок-сервера и Swagger UI.

## Валидация yaml файлов
- Редактируйте файлы в `docs/openapi/**`.

### yq: проверка YAML и ссылок
- **Что это**: `yq` — CLI для работы с YAML (аналог `jq` для JSON).
- **Установка (ubuntu/snap)**:
```bash
sudo snap install yq
```
- **Проверка корректности корневого YAML**:
```bash
yq e '.' docs/openapi/openapi.yaml >/dev/null && echo OK
```
- **Поиск всех $ref** (для диагностики разрывов ссылок):
```bash
yq e '.. | select(tag=="!!map" and has("$ref")) | ."$ref"' docs/openapi/openapi.yaml
```
- **Быстрая валидация всех YAML в каталоге**:
```bash
find docs/openapi -type f -name "*.yaml" -print0 | xargs -0 -n1 -I{} sh -c 'yq e "." "{}" >/dev/null || echo "YAML ERROR: {}"'
```

Примечание: `yq` не валидирует соответствие спецификации OpenAPI — он проверяет синтаксис YAML и помогает исследовать/инспектировать ссылки и структуры. Для схемной валидации используйте валидаторы OpenAPI (например, `swagger-cli validate`).

### Redocly CLI: валидация OpenAPI

**Установка:**
```bash
npm install -g @redocly/cli
```

**Валидация модульной структуры:**
```bash
redocly lint docs/openapi/openapi.yaml
```

**Валидация монолитного файла:**
```bash
redocly lint docs/openapi.yaml 
```

**Сборка многофайловой спецификации в один файл:**
```bash
redocly bundle docs/openapi/openapi.yaml -o dist/openapi.yaml
```
