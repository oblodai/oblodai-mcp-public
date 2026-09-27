<div align="center">

<a href="https://oblodai.com">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/oblodai/.github/main/brand/logo-white.svg">
    <img src="https://raw.githubusercontent.com/oblodai/.github/main/brand/logo-black.svg" alt="oblodai" height="52">
  </picture>
</a>

<h3>Официальный MCP-сервер платёжного шлюза <a href="https://oblodai.com">oblodai</a></h3>

Claude, Cursor, VS Code или ChatGPT подключат платежи и помогут вести магазин — ничего не нужно устанавливать.

<img src="https://img.shields.io/badge/MCP-2026--07--28%20%7C%202025--11--25-000000?style=flat-square" alt="MCP revisions">
<img src="https://img.shields.io/badge/transport-Streamable%20HTTP-000000?style=flat-square" alt="Streamable HTTP">
<a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-000000?style=flat-square" alt="License: MIT"></a>

[Documentation](https://docs.oblodai.com) · [Dashboard](https://my.oblodai.com) · [Read in English →](README.md)

</div>

---

Официальный сервер [Model Context Protocol](https://modelcontextprotocol.io) платёжного шлюза
**Oblodai**. Его держим мы; вашему ИИ-ассистенту нужен один адрес и, по желанию, API-ключ:

```
https://mcp.oblodai.com/mcp
```

Он знает весь API Oblodai — каждую операцию, поле, статус, ошибку и событие вебхука — и пишет с
вами код интеграции на любом из восьми языков SDK. С ключом он ещё и работает с вашим магазином:
счета, платежи, выплаты, возвраты, баланс, доставка вебхуков, песочница. Всё, что двигает деньги
или меняет настройки, ждёт вашего подтверждения.

## Содержание

- [Быстрый старт](#быстрый-старт)
- [Режимы](#режимы)
- [Где взять ключи](#где-взять-ключи)
- [Подключение](#подключение)
- [Что можно попросить](#что-можно-попросить)
- [Как выглядит денежная операция](#как-выглядит-денежная-операция)
- [Инструменты, ресурсы и промпты](#инструменты-ресурсы-и-промпты)
- [Лимиты](#лимиты)
- [Если что-то не работает](#если-что-то-не-работает)
- [Безопасность](#безопасность)
- [Вопросы и ответы](#вопросы-и-ответы)

## Быстрый старт

1. Добавьте сервер в ассистента — для Claude Code:
   ```sh
   claude mcp add --transport http oblodai https://mcp.oblodai.com/mcp
   ```
2. Попросите: *«Подключи оплату Oblodai к моему чекауту на Django»*. Ассистент найдёт нужные
   операции в API, прочитает их и напишет код на Python SDK.
3. Когда захотите, чтобы он работал с магазином, добавьте ключ песочницы (см.
   [Где взять ключи](#где-взять-ключи)) и попросите выставить тестовый счёт и оплатить его в песочнице.

## Режимы

| Режим | Что передаёте | Что может ассистент |
| --- | --- | --- |
| **Справочник** | ничего | искать по API, читать операции, события вебхуков и типы, брать примеры кода, вызывать публичные операции (например, список валют) |
| **Магазин** | `Authorization: Bearer <public_id>:<secret>` | всё выше, плюс инструменты магазина ниже и любую операцию API, доступную вашему ключу |
| **Только чтение** | ключ и `X-Oblodai-Mcp-Read-Only: 1` | режим магазина без единой операции, которая что-то меняет |

Режим определяется заголовками каждого запроса, поэтому ключ можно сменить без переподключения.

## Где взять ключи

У мерчанта **один API-ключ**. Он выдаётся в [кабинете](https://my.oblodai.com), раздел
**API keys**, как пара «публичный идентификатор + секрет». Сервер принимает его как
`Bearer <public_id>:<secret>`.

- **Сначала песочница.** Ключ песочницы (`test_…`) работает только с тестовыми балансами: кран их
  пополняет, имитация депозита оплачивает счёт. Настоящие деньги не двигаются. Работайте с ним,
  пока строите интеграцию.
- **Права.** API проверяет каждый вызов по вашему ключу так же, как для SDK. Если ассистенту нужно
  только смотреть, добавьте заголовок «только чтение» — тогда операции, которые что-то меняют, ему
  даже не предлагаются.
- **Не вставляйте секрет в чат.** Он пишется в настройки клиента; ассистент его не видит.

## Подключение

Подойдёт любой клиент, который говорит на MCP через Streamable HTTP. Для режима справочника
заголовок `Authorization` не нужен.

### Claude Code

```sh
claude mcp add --transport http oblodai https://mcp.oblodai.com/mcp \
  --header "Authorization: Bearer <public_id>:<secret>"
```

### Cursor

`~/.cursor/mcp.json` (или `.cursor/mcp.json` в проекте):

```json
{
  "mcpServers": {
    "oblodai": {
      "url": "https://mcp.oblodai.com/mcp",
      "headers": { "Authorization": "Bearer <public_id>:<secret>" }
    }
  }
}
```

### VS Code

`.vscode/mcp.json`:

```json
{
  "servers": {
    "oblodai": {
      "type": "http",
      "url": "https://mcp.oblodai.com/mcp",
      "headers": { "Authorization": "Bearer <public_id>:<secret>" }
    }
  }
}
```

### Codex CLI

Ключ остаётся в переменной окружения, а не в файле настроек:

```sh
export OBLODAI_MCP_TOKEN="<public_id>:<secret>"
codex mcp add oblodai --url https://mcp.oblodai.com/mcp --bearer-token-env-var OBLODAI_MCP_TOKEN
```

### Gemini CLI

```sh
gemini mcp add --transport http --header "Authorization: Bearer <public_id>:<secret>" \
  oblodai https://mcp.oblodai.com/mcp
```

### Claude, ChatGPT и другие приложения

Добавьте свой коннектор с адресом `https://mcp.oblodai.com/mcp`. Приложения, которые не умеют
передавать свой заголовок, получают режим справочника — его хватает, чтобы написать интеграцию.

## Что можно попросить

| Вы говорите | Что происходит |
| --- | --- |
| «Подключи оплату Oblodai к моему приложению на Next.js» | Разбирает путь платежа, пишет создание счёта, страницу возврата и обработчик вебхука на Node SDK |
| «Как проверить подпись вебхука Oblodai на PHP?» | Читает правила подписи и даёт готовую проверку |
| «Выставь счёт на 25 USDT для заказа 1042 и дождись оплаты» | Создаёт счёт и опрашивает статус, пока он не станет окончательным |
| «Почему вебхук по платежу `…` не дошёл до моего сервера?» | Показывает попытки доставки с HTTP-статусами и ответами |
| «Какой у меня баланс? Покажи выплаты за сегодня» | Читает баланс и историю выплат |
| «Верни платёж `…` полностью» | Показывает, что будет возвращено, и ждёт вашего «да» |
| «Пополни песочницу и имитируй оплату счёта `…`» | Использует кран песочницы и имитацию депозита |

Встроенные промпты `integrate_payments` (с `language` и `framework`) и `debug_webhooks` начинают два
самых частых разговора.

## Как выглядит денежная операция

Возвраты, выплаты и любая другая операция, которая двигает деньги или меняет настройки, — это два
вызова:

1. **Предпросмотр.** Первый вызов ничего не меняет. Он отвечает, что произойдёт — сумма, адрес,
   сеть, комиссия, — и выдаёт `confirm_token`.
2. **Ваш ответ.** Ассистент показывает вам предпросмотр. Только после вашего согласия он повторяет
   вызов с токеном.

```text
Вы:         Выплати 150 USDT на TQ3x…9fA в сети Tron.
Ассистент:  Предпросмотр выплаты: 150.00 USDT на TQ3x…9fA (Tron), комиссия сети 1.10 USDT
            (цифры для примера). Отправить?
Вы:         Да.
Ассистент:  Выплата создана: статус «processing».
```

Токен привязан к вашему ключу и ровно к этим аргументам, живёт 5 минут и срабатывает один раз.
Если между предпросмотром и подтверждением что-то изменилось, API откажет, а не отправит другую
сумму.

## Инструменты, ресурсы и промпты

Всегда доступны:

| Инструмент | Что делает |
| --- | --- |
| `oblodai_api_search` | Находит операции по описанию задачи |
| `oblodai_code_example` | Готовый пример операции на Python, Node, Go, PHP, Java, .NET, Ruby или Rust |
| `oblodai_api_call` | Вызывает любую операцию по id (публичные — без ключа, остальные — с ключом) |

С ключом — инструменты магазина:

<!-- tools -->
| Tool | Operation | Confirmation |
| --- | --- | --- |
| `payments_create` | Create a payment (invoice) (`createPayment`) | — |
| `payments_get` | Get payment status (`getPaymentInfo`) | — |
| `payments_list` | Payment history (`listPaymentHistory`) | — |
| `payments_refund` | Refund a payment (`refundPayment`) | yes |
| `payouts_create` | Create a payout (`createPayout`) | yes |
| `payouts_get` | Get payout status (`getPayoutInfo`) | — |
| `payouts_list` | Payout history (`listPayoutHistory`) | — |
| `payouts_calculate` | Calculate payout amount and fee (`calculatePayout`) | — |
| `balance_get` | Merchant balance (`getBalance`) | — |
| `webhooks_list_deliveries` | Webhook delivery log (`listWebhookDeliveries`) | — |
| `webhooks_send_test` | Test PAYMENT webhook (`sendTestPaymentWebhook`) | — |
| `sandbox_faucet` | Faucet: top up the test balance (`sandboxFaucet`) | — |
| `sandbox_simulate_deposit` | Simulate an on-chain deposit (`sandboxSimulateDeposit`) | — |
<!-- /tools -->

Инструменты, которые читают статус, принимают `wait: true` и опрашивают его, пока он не станет
окончательным, — до 60 секунд. Списки принимают `limit` и `cursor`.

Ресурсы: `oblodai://overview` (весь API на одной странице), `oblodai://operation/{operation_id}`,
`oblodai://webhook-event/{event}`, `oblodai://type/{name}`, `oblodai://signing` (подпись запросов и
вебхуков).

Промпты: `integrate_payments` (`language`, `framework`), `debug_webhooks`.

Сервер собирается из контракта API Oblodai, поэтому новая версия API доходит до него без ручных
правок — инструменты, поля и примеры всегда совпадают с API.

## Лимиты

| Что | Предел |
| --- | --- |
| Запросы без ключа | 60 в минуту с одного IP |
| Запросы с ключом | 300 в минуту на ключ |
| Тело запроса | 1 МБ |
| `wait` | до 60 секунд за вызов |
| `confirm_token` | 5 минут, один раз |

Сверх лимита сервер отвечает `429` с `Retry-After`. Собственные лимиты API для вашего ключа
действуют поверх.

## Если что-то не работает

| Что видно | Причина и что делать |
| --- | --- |
| `401` с «expected Authorization: Bearer <public_id>:<secret>» | Заголовок записан неверно: один пробел после `Bearer`, двоеточие между идентификатором и секретом |
| Видно только три инструмента | Ключ не дошёл до сервера — клиент не передал заголовок (режим справочника) |
| Среди инструментов магазина нет возвратов и выплат | Стоит заголовок «только чтение» |
| `403` в клиенте, который работает в браузере | Запросы из браузера не принимаются; используйте настольный клиент, IDE или CLI |
| `429` | Превышен лимит выше; подождите `Retry-After` |
| Операция отклонена из-за прав | API не разрешает её вашему ключу — проверьте ключ в кабинете |

## Безопасность

- Ничто, что двигает деньги или меняет настройки, не выполняется без двухшагового подтверждения
  выше; режим только чтения прячет такие операции совсем.
- Сервер ничего не хранит, в том числе ключи. Ключ используется только для подписи ваших вызовов —
  так же, как в SDK, — и API проверяет его на каждом вызове.
- Собственные защиты API для вашего ключа (белый список IP, лимиты) действуют на вызовы через
  ассистента так же, как на любые другие.
- Ассистент читает то, что вы просите прочитать. Секрет он не видит — тот остаётся в настройках
  клиента.

## Вопросы и ответы

**Нужно ли что-то устанавливать?** Нет. Сервер работает у Oblodai; вы добавляете его адрес в
ассистента.

**Может ли ассистент отправить деньги без меня?** Нет. Каждой денежной операции нужно ваше явное
согласие с предпросмотром, а заголовок «только чтение» убирает такие операции совсем.

**Какие ассистенты поддерживаются?** Любой MCP-клиент через Streamable HTTP: Claude Code, Claude,
Cursor, VS Code, Codex CLI, Gemini CLI, коннекторы ChatGPT и другие. Ревизии MCP 2026-07-28 и
2025-11-25 и старше, до 2024-11-05.

**Это тот же API, что в SDK?** Да — те же операции из того же контракта; примеры на всех восьми
языках SDK.

**Где исходный код?** Сам сервер закрыт. Этот репозиторий описывает, как к нему подключиться.

## SDK

[Python](https://github.com/oblodai/oblodai-python) ·
[Node.js](https://github.com/oblodai/oblodai-node) ·
[Go](https://github.com/oblodai/oblodai-go) ·
[PHP](https://github.com/oblodai/oblodai-php) ·
[Java](https://github.com/oblodai/oblodai-java) ·
[.NET](https://github.com/oblodai/oblodai-dotnet) ·
[Ruby](https://github.com/oblodai/oblodai-ruby) ·
[Rust](https://github.com/oblodai/oblodai-rust)

## Лицензия

MIT — см. [LICENSE](LICENSE).
