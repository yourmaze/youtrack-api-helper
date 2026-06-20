# YouTrack API Helper

Небольшой shell-скрипт для работы с YouTrack REST API через permanent token.

В пакете нет реальных токенов, приватных `.env`-файлов, внутренних URL или рабочих ключей задач. Каждый пользователь создает свой токен и хранит его локально.

## Что внутри

- `yt-api` - обертка над `curl` для запросов в YouTrack REST API.
- `.env.example` - пример файла с настройками.
- `SKILL.md` - дополнительная инструкция для Codex, если хочется подключить helper как навык.

## Установка

Склонируйте репозиторий:

```bash
git clone https://github.com/yourmaze/youtrack-api-helper.git
cd youtrack-api-helper
chmod +x yt-api
```

Скрипт можно запускать из папки проекта:

```bash
./yt-api GET '/api/users/me?fields=id,login,fullName'
```

При желании папку можно добавить в `PATH`, чтобы запускать `yt-api` из любого места.

## Как получить токен YouTrack

1. Откройте YouTrack.
2. Перейдите в профиль пользователя.
3. Найдите раздел безопасности, authentication или permanent tokens.
4. Создайте новый permanent token.
5. Выберите scope `YouTrack`.
6. Скопируйте токен и сохраните его только в локальный файл с секретами.

Permanent token работает с правами пользователя, который его создал. Если у пользователя нет доступа к проекту или операции, API тоже вернет ошибку доступа.

## Настройка подключения

Создайте локальный файл с секретами:

```bash
mkdir -p ~/.config/youtrack-api-helper
cp .env.example ~/.config/youtrack-api-helper/youtrack.env
chmod 600 ~/.config/youtrack-api-helper/youtrack.env
```

Откройте файл:

```bash
nano ~/.config/youtrack-api-helper/youtrack.env
```

Пример содержимого:

```bash
YOUTRACK_URL=https://youtrack.example.com
YOUTRACK_TOKEN=perm:your-token
```

Если нужно использовать другой файл с секретами, передайте путь через переменную `YOUTRACK_ENV_FILE`:

```bash
YOUTRACK_ENV_FILE=/path/to/youtrack.env ./yt-api GET '/api/users/me?fields=id,login,fullName'
```

## Проверка подключения

```bash
./yt-api GET '/api/users/me?fields=id,login,fullName'
```

Если все настроено правильно, YouTrack вернет информацию о текущем пользователе.

## Прочитать задачу

```bash
./yt-api GET '/api/issues/PROJECT-123?fields=idReadable,summary,description'
```

Более подробный вариант:

```bash
./yt-api GET '/api/issues/PROJECT-123?fields=id,idReadable,summary,description,project(shortName,name),customFields(name,value(name,localizedName,presentation,text,id)),comments(id,text,author(login,fullName),created,updated)'
```

## Добавить комментарий

```bash
./yt-api POST '/api/issues/PROJECT-123/comments?fields=id,text' '{"text":"Текст комментария"}'
```

## Создать задачу

Создание задачи выполняется через `POST /api/issues`. Точный JSON зависит от проекта и его custom fields, поэтому перед автоматизацией лучше проверить id проекта, доступные поля и допустимые значения.

Минимальный пример:

```bash
./yt-api POST '/api/issues?fields=idReadable,summary' '{
  "project": {"shortName": "PROJECT"},
  "summary": "Короткий заголовок задачи",
  "description": "Описание задачи"
}'
```

Если нужно выставлять статус, версию, исполнителя, теги или другие поля, сначала получите метаданные проекта и поля через YouTrack API, затем добавьте нужные `customFields` в тело запроса.

## Правила безопасности

- Не коммитьте реальные `.env`-файлы.
- Не вставляйте токены в чат, задачи, логи или примеры команд.
- Используйте `fields=` в API-запросах, чтобы ответы были компактными.
- Операции чтения обычно можно выполнять сразу.
- Перед созданием задачи, комментарием или изменением полей сначала подготовьте черновик и получите явное подтверждение.
- Если YouTrack возвращает `401` или `403`, обновите права или создайте новый токен. Не используйте пароль вместо токена.
