<div align="center">

<a href="https://oblodai.com">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/oblodai/.github/main/brand/logo-white.svg">
    <img src="https://raw.githubusercontent.com/oblodai/.github/main/brand/logo-black.svg" alt="oblodai" height="52">
  </picture>
</a>

<h3>Official MCP server for the <a href="https://oblodai.com">oblodai</a> payment gateway</h3>

Let Claude, Cursor, VS Code or ChatGPT integrate payments and run your store — nothing to install.

<img src="https://img.shields.io/badge/MCP-2026--07--28%20%7C%202025--11--25-000000?style=flat-square" alt="MCP revisions">
<img src="https://img.shields.io/badge/transport-Streamable%20HTTP-000000?style=flat-square" alt="Streamable HTTP">
<a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-000000?style=flat-square" alt="License: MIT"></a>

[Documentation](https://docs.oblodai.com) · [Dashboard](https://my.oblodai.com) · [Read in Russian →](README.ru.md)

</div>

---

The official [Model Context Protocol](https://modelcontextprotocol.io) server of the **Oblodai**
payment gateway. Oblodai hosts it; your AI assistant needs one address and, optionally, an API key:

```
https://mcp.oblodai.com/mcp
```

It knows the whole Oblodai API — every operation, field, status, error and webhook event — and
writes integration code with you in any of the eight SDK languages. Given a key, it also works with
your store: invoices, payments, payouts, refunds, balance, webhook deliveries and the sandbox.
Anything that moves money or changes settings waits for your confirmation.

## Contents

- [Quick start](#quick-start)
- [Modes](#modes)
- [Where to get keys](#where-to-get-keys)
- [Connect your assistant](#connect-your-assistant)
- [What to ask](#what-to-ask)
- [How a money operation looks](#how-a-money-operation-looks)
- [Tools, resources and prompts](#tools-resources-and-prompts)
- [Limits](#limits)
- [Troubleshooting](#troubleshooting)
- [Safety and privacy](#safety-and-privacy)
- [FAQ](#faq)

## Quick start

1. Add the server to your assistant — for Claude Code:
   ```sh
   claude mcp add --transport http oblodai https://mcp.oblodai.com/mcp
   ```
2. Ask: *“Integrate Oblodai payments into my Django checkout.”* The assistant searches the API,
   reads the operations it needs and writes the code with the Python SDK.
3. When you want it to work with your store, add a sandbox key (see [Where to get keys](#where-to-get-keys))
   and ask it to create a test invoice and pay it in the sandbox.

## Modes

| Mode | You pass | The assistant can |
| --- | --- | --- |
| **Reference** | nothing | search the API, read operations, webhook events and types, get code examples, call public operations (e.g. the list of currencies) |
| **Store** | `Authorization: Bearer <public_id>:<secret>` | everything above, plus the store tools below and any operation of the API your key may call |
| **Read-only** | a key and `X-Oblodai-Mcp-Read-Only: 1` | the store mode without a single operation that changes something |

The mode is decided per request by the headers, so one client can switch keys without reconnecting.

## Where to get keys

A merchant has **one API key**, issued in the [dashboard](https://my.oblodai.com) → **API keys**: a
public id and a secret. The server takes it as `Bearer <public_id>:<secret>`.

- **Sandbox first.** A sandbox key (`test_…`) works against test balances only: the faucet tops them
  up, a simulated deposit pays an invoice. Nothing real moves. Use it while you build.
- **Permissions.** The API checks every call against your key exactly as it does for the SDKs. For
  an assistant that only needs to look, add the read-only header — then no operation that changes
  something is even offered.
- **Never paste the secret into the chat.** Put it into the client configuration; the assistant
  never sees it.

## Connect your assistant

Any client that speaks MCP over Streamable HTTP works. Leave the `Authorization` header out for the
reference mode.

### Claude Code

```sh
claude mcp add --transport http oblodai https://mcp.oblodai.com/mcp \
  --header "Authorization: Bearer <public_id>:<secret>"
```

### Cursor

`~/.cursor/mcp.json` (or `.cursor/mcp.json` in a project):

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

The key stays in the environment, not in the config:

```sh
export OBLODAI_MCP_TOKEN="<public_id>:<secret>"
codex mcp add oblodai --url https://mcp.oblodai.com/mcp --bearer-token-env-var OBLODAI_MCP_TOKEN
```

### Gemini CLI

```sh
gemini mcp add --transport http --header "Authorization: Bearer <public_id>:<secret>" \
  oblodai https://mcp.oblodai.com/mcp
```

### Claude, ChatGPT and other apps

Add a custom connector with the URL `https://mcp.oblodai.com/mcp`. Apps that cannot send a custom
header get the reference mode — enough to write the integration.

## What to ask

| You say | What happens |
| --- | --- |
| “Integrate Oblodai payments into my Next.js app.” | Reads the payment flow, writes invoice creation, the return page and the webhook handler with the Node SDK |
| “How do I verify an Oblodai webhook signature in PHP?” | Reads the signing rules and gives a runnable check |
| “Create a 25 USDT invoice for order 1042 and wait until it is paid.” | Creates the invoice and polls its status until it is final |
| “Why did the webhook for payment `…` not reach my server?” | Lists the delivery attempts with their HTTP statuses and responses |
| “What is my balance? Show today's payouts.” | Reads the balance and the payout history |
| “Refund payment `…` in full.” | Shows what will be refunded and waits for your “yes” |
| “Top up my sandbox and simulate a deposit to invoice `…`.” | Uses the sandbox faucet and deposit simulation |

The built-in prompts `integrate_payments` (with `language` and `framework`) and `debug_webhooks`
start the two most common conversations.

## How a money operation looks

Refunds, payouts and every other operation that moves money or changes settings take two calls:

1. **Preview.** The first call changes nothing. It answers with what would happen — amount,
   address, network, fee — and a `confirm_token`.
2. **Your answer.** The assistant shows you the preview. Only after you agree does it repeat the call
   with the token.

```text
You:        Pay out 150 USDT to TQ3x…9fA on Tron.
Assistant:  Payout preview: 150.00 USDT to TQ3x…9fA (Tron), network fee 1.10 USDT
            (example figures). Send it?
You:        Yes.
Assistant:  Payout created: status "processing".
```

The token is bound to your key and to exactly these arguments, expires in 5 minutes and cannot be
used twice. If anything changes between the preview and the confirmation, the API refuses instead of
sending a different amount.

## Tools, resources and prompts

Always available:

| Tool | What it does |
| --- | --- |
| `oblodai_api_search` | Find operations by what you want to do |
| `oblodai_code_example` | A runnable example of one operation in Python, Node, Go, PHP, Java, .NET, Ruby or Rust |
| `oblodai_api_call` | Call any operation by id (public ones without a key, the rest with one) |

With a key, the store tools:

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

Tools that read a status accept `wait: true` and poll until the status is final, for up to 60
seconds. Lists take `limit` and `cursor`.

Resources: `oblodai://overview` (the whole API on one page), `oblodai://operation/{operation_id}`,
`oblodai://webhook-event/{event}`, `oblodai://type/{name}`, `oblodai://signing` (request and webhook
signatures).

Prompts: `integrate_payments` (`language`, `framework`), `debug_webhooks`.

The server is generated from the Oblodai API contract, so a new API version reaches it without hand
edits — the tools, fields and examples always match the API.

## Limits

| What | Limit |
| --- | --- |
| Requests without a key | 60 a minute per client IP |
| Requests with a key | 300 a minute per key |
| Request body | 1 MB |
| `wait` | up to 60 seconds per call |
| `confirm_token` | 5 minutes, one use |

Over a limit the server answers `429` with `Retry-After`. The API's own limits for your key apply
on top.

## Troubleshooting

| Symptom | Cause and fix |
| --- | --- |
| `401` with “expected Authorization: Bearer <public_id>:<secret>” | The header is malformed: one space after `Bearer`, a colon between the id and the secret |
| Only three tools are listed | No key reached the server — the client did not send the header (reference mode) |
| Store tools are missing refunds and payouts | The read-only header is set |
| `403` in a browser-based client | Browser origins are not accepted; use a desktop, IDE or CLI client |
| `429` | A limit above; wait for `Retry-After` |
| An operation is refused with a permission error | The API does not allow it for your key — check the key in the dashboard |

## Safety and privacy

- Nothing that moves money or changes settings runs without the two-step confirmation above;
  read-only mode hides such operations entirely.
- The server holds no state and stores no keys. Your key is used only to sign the calls you make,
  exactly as the SDKs sign them, and the API checks it on every call.
- The API's own controls for your key (IP allowlist, limits) apply to calls made through the
  assistant just as to any other.
- The assistant reads what you ask it to read. It does not see the secret, which stays in the client
  configuration.

## FAQ

**Do I need to install anything?** No. The server runs at Oblodai; you add its address to your
assistant.

**Can the assistant send money without me?** No. Every money operation needs your explicit
confirmation of a preview, and the read-only header removes such operations altogether.

**Which assistants are supported?** Any MCP client over Streamable HTTP: Claude Code, Claude,
Cursor, VS Code, Codex CLI, Gemini CLI, ChatGPT connectors and others. MCP revisions 2026-07-28 and
2025-11-25 back to 2024-11-05.

**Is it the same API as the SDKs?** Yes — the same operations, generated from the same contract;
examples are in all eight SDK languages.

**Where is the source code?** The server itself is not open source. This repository documents how to
connect to it.

## SDKs

[Python](https://github.com/oblodai/oblodai-python) ·
[Node.js](https://github.com/oblodai/oblodai-node) ·
[Go](https://github.com/oblodai/oblodai-go) ·
[PHP](https://github.com/oblodai/oblodai-php) ·
[Java](https://github.com/oblodai/oblodai-java) ·
[.NET](https://github.com/oblodai/oblodai-dotnet) ·
[Ruby](https://github.com/oblodai/oblodai-ruby) ·
[Rust](https://github.com/oblodai/oblodai-rust)

## License

MIT — see [LICENSE](LICENSE).
