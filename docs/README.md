# Документация (docs)

Ниже схема директорий спецификации и краткие пояснения.

```
docs/
  openapi/
    openapi.yaml                # Корневой файл с $ref на части
    paths/
      auth.login.yaml           # /auth/login
      users.yaml                # /users (GET/POST)
      posts.yaml                # /posts (GET/POST)
      posts.postId.yaml         # /posts/{postId} (GET/PUT/DELETE)
      posts.postId.comments.yaml# /posts/{postId}/comments (GET/POST)
      status.yaml               # /status (GET)
      cart.yaml                 # /cart (GET)
    components/
      securitySchemes/
        BearerAuth.yaml         # JWT Bearer
      parameters/
        PostIdParam.yaml        # Параметр пути postId
        PageParam.yaml          # Параметр page
        LimitParam.yaml         # Параметр limit
      headers/
        TotalCount.yaml         # Заголовок X-Total-Count
      responses/
        ValidationError.yaml    # 400 ошибка валидации
        UnauthorizedError.yaml  # 401 не авторизован
        NotFoundError.yaml      # 404 не найдено
      schemas/
        LoginRequest.yaml
        AuthResponse.yaml
        User.yaml
        UserCreate.yaml
        Post.yaml
        PostCreate.yaml
        PostUpdate.yaml
        PostList.yaml
        Comment.yaml
        CommentCreate.yaml
        Pagination.yaml
        Error.yaml
        ValidationError.yaml
```

Принципы:
- Компоненты переиспользуются через `$ref` из `openapi.yaml`.
- `paths/*` содержит операции в точности по ресурсам.
- Схемы данных (модели) лежат в `components/schemas`.
- Обновления выполняются атомарно: изменили схему — обновили ссылки.

## yq: что проверяем и как использовать
- **Назначение**: CLI для чтения/проверки YAML; удобно для проверки синтаксиса и поиска `$ref`.
- **Проверка синтаксиса всех файлов**:
```bash
find ./openapi -type f -name "*.yaml" -print0 | xargs -0 -n1 -I{} sh -c 'yq e "." "{}" >/dev/null || echo "YAML ERROR: {}"'
```
- **Проверка, что корневой файл читается**:
```bash
yq e '.' ./openapi/openapi.yaml >/dev/null && echo OK
```
- **Список всех `$ref` для ручной проверки путей**:
```bash
yq e '.. | select(tag=="!!map" and has("$ref")) | ."$ref"' ./openapi/openapi.yaml
```
- **Примечание**: `yq` не валидирует саму спецификацию OpenAPI. Для этого используйте отдельные инструменты (например, `swagger-cli validate`). 