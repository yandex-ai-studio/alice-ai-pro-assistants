# Подключение opencode к подписке «Алиса AI Про»

opencode (<https://opencode.ai>) подключается к подписке **Алиса AI Про** как обычный
OpenAI-совместимый провайдер (`@ai-sdk/openai-compatible`). Авторизация выполняется по
IAM-токену Yandex Cloud, который выпускается командой `yc iam create-token`.

Оформить подписку: <https://aistudio.yandex.ru/manage>

## Требования

- Установленный opencode (<https://opencode.ai/docs>).
- Установленный Yandex Cloud CLI (`yc`).
- Аккаунт Yandex Cloud с активной подпиской «Алиса AI Про».
- Идентификатор каталога (`folder-id`), в котором оформлена подписка.

## 1. Опишите провайдера и модели

**Файл `~/.config/opencode/opencode.json`** (либо `opencode.json` в корне проекта):

```json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "yandex": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "Yandex Cloud (Алиса AI Про)",
      "options": {
        "baseURL": "https://apps.ai.api.cloud.yandex.net/v1",
        "headers": {
          "OpenAI-Project": "<folder-id>"
        }
      },
      "models": {
        "deepseek-v4-flash": {
          "id": "gpt://<folder-id>/deepseek-v4-flash/latest",
          "name": "DeepSeek V4 Flash",
          "interleaved": { "field": "reasoning_content" },
          "limit": { "context": 1000000, "output": 32768 }
        },
        "deepseek-v4.1-flash": {
          "id": "gpt://<folder-id>/deepseek-v4.1-flash/latest",
          "name": "DeepSeek V4.1 Flash",
          "interleaved": { "field": "reasoning_content" },
          "limit": { "context": 512000, "output": 32768 },
          "modalities": {
            "input": ["text", "image"],
            "output": ["text"]
          }
        }
      }
    }
  }
}
```

Замените `<folder-id>` на идентификатор вашего каталога в Yandex Cloud.

Что за что отвечает:

| Поле | Значение |
|---|---|
| `options.baseURL` | Адрес Yandex Cloud для подписок: `https://apps.ai.api.cloud.yandex.net/v1`. |
| `options.headers."OpenAI-Project"` | Каталог подписки. |
| `models.<ключ>.id` | URI модели в Yandex Cloud (`gpt://<folder-id>/<model>/latest`). |
| `models.<ключ>.modalities` | У `deepseek-v4.1-flash` добавлен вход `image` — это вижен-модель. |
| `models.<ключ>.interleaved.field` | Модель отдаёт рассуждения в поле `reasoning_content`. |

Модели DeepSeek, доступные в подписке:

| Модель | id | Контекст | Вход |
|---|---|---|---|
| DeepSeek V4 Flash | `gpt://<folder-id>/deepseek-v4-flash/latest` | 1 000 000 | текст |
| DeepSeek V4.1 Flash | `gpt://<folder-id>/deepseek-v4.1-flash/latest` | 512 000 | текст, изображения |

> Ключ в конфиге не нужен: токен вводится при входе (шаг 2) и хранится в отдельном
> хранилище авторизации opencode.

## 2. Войдите в аккаунт

```bash
opencode auth login
```

Выберите провайдера «Yandex Cloud (Алиса AI Про)» и вставьте IAM-токен, полученный командой
`yc iam create-token`. Токен сохранится в хранилище авторизации opencode
(`~/.local/share/opencode/auth.json`) — в файле конфигурации секретов не остаётся.

## 3. Запустите opencode

```bash
opencode
```

Провайдер появится в списке как «Yandex Cloud (Алиса AI Про)»; модели DeepSeek V4 Flash и
DeepSeek V4.1 Flash выбираются в интерфейсе opencode.

## Обновление токена

IAM-токен действует не более суток, поэтому его нужно периодически обновлять. Перед новым
рабочим днём или когда запросы начали возвращать ошибку авторизации выпустите токен заново
и обновите его:

```bash
yc iam create-token
opencode auth login
```

## Проверка подключения

```bash
curl https://apps.ai.api.cloud.yandex.net/v1/chat/completions \
  -H "Authorization: Bearer $(yc iam create-token)" \
  -H "OpenAI-Project: <folder-id>" \
  -H "Content-Type: application/json" \
  -d '{"model":"gpt://<folder-id>/deepseek-v4.1-flash/latest","messages":[{"role":"user","content":"ping"}],"max_tokens":16}'
```

Успешный ответ — объект `chat.completion` с `choices[].message` и `usage`.
