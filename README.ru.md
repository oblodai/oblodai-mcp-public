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
**Oblodai**. Его держим мы; вашему ИИ-ассистенту нужен один адрес:

```
https://mcp.oblodai.com/mcp
```

- **Без API-ключа** это справочник: поиск по всему API, описание каждой операции и каждого
  события вебхука, готовые примеры для всех восьми SDK Oblodai. Этого хватает, чтобы написать всю
  интеграцию.
- **С API-ключом** он ещё и работает с вашим магазином: выставляет счета и ждёт оплаты, находит
  платежи и выплаты, считает и отправляет выплаты, делает возвраты, показывает баланс, разбирает
  доставку вебхуков, управляет песочницей.

Всё, что двигает деньги или меняет настройки, сначала подтверждается (см. [Безопасность](#безопасность)).

## Где взять ключи

У мерчанта **один API-ключ**. Он выдаётся в [кабинете](https://my.oblodai.com), раздел
**API keys**, как пара «публичный идентификатор + секрет». Сервер принимает его в заголовке
`Authorization` как `Bearer <public_id>:<secret>`; без заголовка — режим справочника. Что можно
ассистенту, решает роль ключа: если ему нужно только смотреть, дайте ключ только на чтение.

## Подключение

### Claude Code

```sh
# справочник, без ключа
claude mcp add --transport http oblodai https://mcp.oblodai.com/mcp
# с ключом
claude mcp add --transport http oblodai https://mcp.oblodai.com/mcp \
  --header "Authorization: Bearer <public_id>:<secret>"
```

### Cursor, VS Code

Cursor: `~/.cursor/mcp.json`. VS Code: `.vscode/mcp.json`, с ключом `servers` вместо
`mcpServers`.

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

### Claude, ChatGPT и другие приложения

Добавьте свой коннектор с адресом `https://mcp.oblodai.com/mcp`. Приложения, которые не умеют
передавать свой заголовок, получают режим справочника.

### Только чтение

Заголовок `X-Oblodai-Mcp-Read-Only: 1` скрывает все операции, которые что-то меняют.

## Что умеет

Всегда:

| Инструмент | Что делает |
| --- | --- |
| `oblodai_api_search` | Находит операции по описанию задачи |
| `oblodai_code_example` | Готовый пример операции на Python, Node, Go, PHP, Java, .NET, Ruby или Rust |
| `oblodai_api_call` | Вызывает любую операцию по id (публичные — без ключа, остальные — с ключом) |

Ресурсы: `oblodai://overview` (весь API), `oblodai://operation/{operation_id}`,
`oblodai://webhook-event/{event}`, `oblodai://type/{name}`, `oblodai://signing`.
Промпты: `integrate_payments` (язык, фреймворк), `debug_webhooks`.

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

Сервер собирается из контракта API Oblodai, поэтому всегда совпадает с текущим API.

## Безопасность

- Операция, которая двигает деньги или меняет настройки, идёт в два шага: первый вызов ничего не
  делает и возвращает сводку с `confirm_token`; ассистент показывает её вам и повторяет вызов с
  токеном. Токен привязан к вашему ключу и к точным аргументам, живёт 5 минут; повтор
  подтверждённого вызова не выполнится дважды.
- Режим только чтения скрывает все такие операции.
- Сервер ничего не хранит, в том числе ключи; ключ доходит до API Oblodai подписанным, как в SDK.

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
