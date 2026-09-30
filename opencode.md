# Подключение opencode к подписке «Алиса AI Про»

opencode (<https://opencode.ai>) подключается к подписке **Алиса AI Про** как обычный
OpenAI-совместимый провайдер (`@ai-sdk/openai-compatible`). Авторизация выполняется по
IAM-токену Yandex Cloud, который выпускается командой `yc iam create-token`.
Токен вводится через команду `/connect` внутри интерфейса opencode.

Оформить подписку: <https://aistudio.yandex.ru/manage>

## Требования

- Установленный opencode (<https://opencode.ai/docs>).
- Установленный и настроенный Yandex Cloud CLI (`yc`).
- Аккаунт Yandex Cloud с активной подпиской «Алиса AI Про».
- Идентификатор каталога (`folder-id`), в котором оформлена подписка.

## 1. Запустите opencode в первый раз

В обычном терминале выполните:

```bash
opencode
```

Дождитесь загрузки интерфейса: при первом запуске opencode создаст необходимые
каталоги и служебные файлы. Затем закройте opencode.

Только после первого запуска переходите к редактированию JSON-конфигурации.

## 2. Опишите провайдера и модели

Откройте в текстовом редакторе **файл `~/.config/opencode/opencode.json`**
(либо `opencode.json` в корне проекта) и добавьте конфигурацию ниже. Если файл
конфигурации ещё отсутствует, создайте его после первого запуска opencode.
Если в нём уже есть настройки, добавьте провайдера `yandex` в раздел `provider`,
сохранив остальные параметры.

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

Замените `<folder-id>` на идентификатор вашего каталога в Yandex Cloud во всех
трёх местах и сохраните файл.

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

> IAM-токен в конфиг добавлять не нужно: он вводится через `/connect` (шаг 4)
> и хранится в отдельном хранилище авторизации opencode.

## 3. Перезапустите opencode

После сохранения конфигурации снова запустите opencode из обычного терминала:

```bash
opencode
```

Если предыдущий экземпляр ещё открыт, сначала закройте его. Новый запуск нужен,
чтобы opencode прочитал сохранённую конфигурацию провайдера и моделей.

## 4. Подключите Yandex через `/connect`

Получите IAM-токен в отдельном обычном терминале:

```bash
yc iam create-token
```

Скопируйте выведенный токен. В уже запущенном opencode введите **прямо в его
консоли**, в поле ввода:

```text
/connect
```

В открывшемся списке выберите **Yandex** — провайдера
**«Yandex Cloud (Алиса AI Про)»** из вашего конфига — и вставьте IAM-токен
в поле для ключа.

Токен сохранится в хранилище авторизации opencode
(`~/.local/share/opencode/auth.json`). В файле конфигурации секретов не остаётся.

После подключения выберите в интерфейсе opencode модель DeepSeek V4 Flash
или DeepSeek V4.1 Flash.

## Обновление токена

IAM-токен действует не более суток, поэтому его нужно периодически обновлять. Перед новым
рабочим днём или когда запросы начали возвращать ошибку авторизации выпустите токен заново
в обычном терминале:

```bash
yc iam create-token
```

Затем в консоли запущенного opencode снова выполните:

```text
/connect
```

Выберите Yandex и вставьте новый IAM-токен.

## Проверка подключения

В обычном терминале выполните, заменив `<folder-id>` на идентификатор вашего каталога:

```bash
curl https://apps.ai.api.cloud.yandex.net/v1/chat/completions \
  -H "Authorization: Bearer $(yc iam create-token)" \
  -H "OpenAI-Project: <folder-id>" \
  -H "Content-Type: application/json" \
  -d '{"model":"gpt://<folder-id>/deepseek-v4.1-flash/latest","messages":[{"role":"user","content":"ping"}],"max_tokens":16}'
```

Успешный ответ — объект `chat.completion` с `choices[].message` и `usage`.
