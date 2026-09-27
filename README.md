# Oblodai MCP

Let an AI assistant integrate [Oblodai](https://oblodai.com) crypto payments for you — or run your
store with you. Oblodai hosts a [Model Context Protocol](https://modelcontextprotocol.io) server
that Claude, Cursor, VS Code, ChatGPT and any other MCP client can connect to. There is nothing to
install.

```
https://mcp.oblodai.com/mcp
```

Streamable HTTP; MCP revisions 2026-07-28 and 2025-11-25 back to 2024-11-05.

- **Without an API key** it is a reference: search the whole API, read every operation and
  webhook event, and get ready-to-run examples for all eight Oblodai SDKs. Enough to write the
  whole integration.
- **With an API key** it also works with your store: create invoices and wait for payment, look
  up payments and payouts, calculate and send payouts, refund, check the balance, debug webhook
  deliveries, and drive the sandbox.

Anything that moves money or changes settings is confirmed first (see [Safety](#safety)).

## Connect

The key goes in the `Authorization` header as `Bearer <public_id>:<secret>`. Leave the header out
for the reference mode.

### Claude Code

```sh
# reference mode, no key
claude mcp add --transport http oblodai https://mcp.oblodai.com/mcp
# with a key
claude mcp add --transport http oblodai https://mcp.oblodai.com/mcp \
  --header "Authorization: Bearer <public_id>:<secret>"
```

### Cursor, VS Code

Cursor: `~/.cursor/mcp.json`. VS Code: `.vscode/mcp.json`, with the key `servers` instead of
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

### Claude, ChatGPT and other apps

Add a custom connector with the URL `https://mcp.oblodai.com/mcp`. Apps that cannot send a
custom header get the reference mode.

### Read-only

Add the header `X-Oblodai-Mcp-Read-Only: 1` to hide every operation that changes something. A
key with a read-only role is safer still: the key's own role decides what it may do.

## What it offers

Always:

| Tool | What it does |
| --- | --- |
| `oblodai_api_search` | Find operations by what you want to do |
| `oblodai_code_example` | A runnable example of one operation in Python, Node, Go, PHP, Java, .NET, Ruby or Rust |
| `oblodai_api_call` | Call any operation by id (public ones without a key, the rest with one) |

Resources: `oblodai://overview` (the whole API), `oblodai://operation/{operation_id}`,
`oblodai://webhook-event/{event}`, `oblodai://type/{name}`, `oblodai://signing`.
Prompts: `integrate_payments` (language, framework), `debug_webhooks`.

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

The server is generated from the Oblodai API contract, so it always matches the current API.

## Safety

- An operation that moves money or changes settings runs in two steps: the first call does
  nothing and returns a summary with a `confirm_token`; the assistant shows it to you and repeats
  the call with the token. The token is bound to your key and to the exact arguments and expires
  in 5 minutes; a retried confirmed call is deduplicated.
- Read-only mode hides every such operation.
- The server holds no state and stores no keys; your key reaches the Oblodai API signed, as with
  the SDKs.

## SDKs

[Python](https://github.com/oblodai/oblodai-python) ·
[Node.js](https://github.com/oblodai/oblodai-node) ·
[Go](https://github.com/oblodai/oblodai-go) ·
[PHP](https://github.com/oblodai/oblodai-php) ·
[Java](https://github.com/oblodai/oblodai-java) ·
[.NET](https://github.com/oblodai/oblodai-dotnet) ·
[Ruby](https://github.com/oblodai/oblodai-ruby) ·
[Rust](https://github.com/oblodai/oblodai-rust)
