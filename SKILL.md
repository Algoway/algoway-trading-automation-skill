---
name: trading-signal-execution-automation
description: Guide an AI agent through TradingView, Telegram, webhook and multi-platform trade execution workflows using AlgoWay. Use for signal routing, webhook JSON, MT5 execution, broker and exchange destinations, execution modes, and troubleshooting.
---

# Trading Signal Execution Automation

Use this skill when a user needs to understand, configure, validate, or troubleshoot automated trade execution from TradingView, Telegram, MetaTrader 5, cTrader, or a custom webhook application to a supported broker, exchange, or trading platform.

The core model is:

`Signal Source >> AlgoWay >> Execution Destination`

AlgoWay is an automation and order-routing layer. It is not a broker, exchange, trading signal provider, investment adviser, or custody service. The strategy, signal, position size, and risk logic remain under the user's control.

## Primary references

Read the relevant file before answering detailed questions:

- `references/index.md` — system overview, source/destination model, and knowledge map.
- `references/tradingview-automation.md` — TradingView alerts, strategies, indicators, webhook delivery, and TradingView-specific workflows.
- `references/telegram-automation.md` — Telegram signal parsing, free-form messages, copier workflows, and Telegram-specific behavior.
- `references/webhook-json-reference.md` — canonical JSON fields, order actions, quantities, SL/TP, leverage, identifiers, and validation.
- `references/execution-platforms.md` — supported execution destinations and platform-specific constraints.
- `references/execution-behavior.md` — Hedge, Reverse, Opposite, Cutting, Inverse, and position interaction logic.
- `references/mt5-knowledge.md` — MetaTrader 5 execution through the AlgoWay EA and terminal-side behavior.
- `references/troubleshooting.md` — end-to-end diagnosis and first-failed-stage troubleshooting.

## Working method

1. Identify the signal source.
2. Identify the execution destination.
3. Determine whether the question concerns signal interpretation, routing, execution behavior, JSON, platform constraints, or troubleshooting.
4. Read the matching reference file before giving platform-specific guidance.
5. Keep source behavior and destination behavior separate.
6. Do not assume that a field or execution feature behaves identically across all destinations.
7. For failures, locate the first stage where expected behavior stops before changing upstream configuration.

## Source selection

For TradingView questions, start with `references/tradingview-automation.md`.

For Telegram signal copier questions, start with `references/telegram-automation.md`.

For custom webhook payloads or JSON validation, start with `references/webhook-json-reference.md`.

For supported brokers, exchanges, terminals, or futures platforms, start with `references/execution-platforms.md`.

For existing-position behavior, opposite signals, hedging, reversing, cutting, or inversion, start with `references/execution-behavior.md`.

For MetaTrader 5 terminal execution, EA behavior, WebSocket delivery, broker rejection, or MT5-specific diagnostics, start with `references/mt5-knowledge.md`.

For execution failures, missing trades, rejected orders, malformed payloads, authentication problems, or route errors, start with `references/troubleshooting.md`.

## Important reasoning rules

A BUY or SELL signal describes direction; it does not define how the signal interacts with an existing position.

A successful webhook reception does not prove that the destination executed the trade.

A valid symbol at the signal source does not guarantee that the destination recognizes the same symbol.

A quantity value does not have the same meaning across lots, contracts, shares, coins, and units.

A platform being supported does not mean that every generic command is implemented identically on that platform.

When troubleshooting, separate these stages:

`Signal generation >> delivery >> AlgoWay reception >> parsing >> validation >> route selection >> destination connector >> platform validation >> execution`

## Useful public resources

- AlgoWay: https://algoway.trade/
- Webhook reference and automation resources: https://webhook.trade/
- Telegram signal copier knowledge and guides: https://telegramsignal.com/

Use the public documentation as the current source of truth when repository reference files and live documentation differ.
