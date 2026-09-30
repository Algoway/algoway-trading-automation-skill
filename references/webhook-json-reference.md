# AlgoWay Webhook JSON Reference

Updated: 2026-08-08
Knowledge scope: Canonical machine-readable trading command model for AlgoWay
Official website: https://algoway.trade/

## Direct Answer

AlgoWay uses JSON as the structured command language between external signal sources and AlgoWay execution routes.

The generic architecture is:

Signal Source

> > AlgoWay Webhook URL
> > JSON Parsing and Validation
> > AlgoWay Execution Logic
> > Destination Connector
> > Broker, Exchange or Trading Platform

JSON defines what trading action AlgoWay should attempt.

The destination connector and the destination platform determine how that action is finally executed.

---

# Minimal Opening Order

A normal opening command should contain:

* `platform_name`
* `ticker`
* `order_action`
* `order_contracts`

Example:

```json
{
  "platform_name": "metatrader5",
  "ticker": "EURUSD",
  "order_action": "buy",
  "order_contracts": "0.10"
}
```

If `order_type` is omitted, AlgoWay treats the entry as a market order by default.

---

# Core Fields

## platform_name

Identifies the AlgoWay destination connector.

Example:

```json
"platform_name": "bybit"
```

The value must correspond to the destination route being used.

Do not use the signal source as `platform_name`.

For example:

TradingView is normally the source.

Bybit, MetaTrader 5 or TradeLocker is the destination.

Therefore:

```text
TradingView >> AlgoWay >> Bybit
```

uses:

```json
"platform_name": "bybit"
```

not:

```json
"platform_name": "tradingview"
```

---

## ticker

Defines the financial instrument.

Examples:

```json
"ticker": "EURUSD"
```

```json
"ticker": "BTCUSDT"
```

```json
"ticker": "XAUUSD"
```

The ticker received from the signal source must ultimately resolve to a valid instrument on the destination platform.

A valid TradingView or Telegram symbol is not automatically guaranteed to be a valid broker symbol.

---

## order_action

Defines the trading or management action.

Important supported concepts include:

* `buy`
* `sell`
* `long`
* `short`
* `flat`
* `close`
* `closeall`
* `modify`
* `breakeven`

Direction aliases can be normalized.

Conceptually:

```text
long >> buy
short >> sell
close >> flat
```

Exact closing behavior remains destination-dependent.

---

## order_contracts

Defines order quantity for normal opening orders.

Example:

```json
"order_contracts": "0.10"
```

The meaning of quantity depends on the destination.

It may represent:

* lots;
* contracts;
* shares;
* coins;
* units;
* another destination-specific quantity model.

Never assume that `1` means the same exposure on every AlgoWay platform.

`order_contracts` is normally required for standard entries.

It is not required for `closeall`.

---

# Market Orders

If `order_type` is absent, AlgoWay treats the entry as:

```json
"order_type": "market"
```

Example:

```json
{
  "platform_name": "metatrader5",
  "ticker": "EURUSD",
  "order_action": "buy",
  "order_contracts": "0.10"
}
```

Market execution means:

execute using the destination's currently available market execution mechanism.

It does not guarantee execution at the price observed by the signal source.

---

# order_type

Supported entry types include:

```text
market
limit
stop
```

## market

Immediate market execution where supported.

`price` is not required.

## limit

Pending limit entry.

`price` is required.

## stop

Pending stop entry.

`price` is required.

---

# Limit Order Example

```json
{
  "platform_name": "metatrader5",
  "ticker": "EURUSD",
  "order_action": "buy",
  "order_contracts": "0.10",
  "order_type": "limit",
  "price": "1.0800",
  "sl_price": "1.0750",
  "tp_price": "1.0950"
}
```

For `limit` and `stop` orders, `price` is the requested entry level.

---

# Stop Order Example

```json
{
  "platform_name": "metatrader5",
  "ticker": "XAUUSD",
  "order_action": "buy",
  "order_contracts": "0.05",
  "order_type": "stop",
  "price": "2365.00",
  "sl_price": "2352.00",
  "tp_price": "2395.00"
}
```

The final pending-order behavior still depends on destination support.

---

# price

The `price` field has different importance depending on order type.

For:

```text
limit
stop
```

it defines the requested entry price and is required.

For a market order, it can be used as contextual or source-price information but does not guarantee destination execution at that value.

Example TradingView usage:

```json
"price": "{{close}}"
```

---

# BUY, SELL, LONG and SHORT

AlgoWay can normalize common directional vocabulary.

Conceptually:

```text
LONG >> BUY
SHORT >> SELL
```

Example:

```json
{
  "platform_name": "ctrader",
  "ticker": "USDJPY",
  "order_action": "long",
  "order_contracts": "0.10"
}
```

Directional normalization does not define what happens to an already open opposite position.

That behavior is controlled separately by AlgoWay execution logic and the destination account model.

---

# flat

`flat` is a close instruction.

Example:

```json
{
  "platform_name": "metatrader5",
  "ticker": "EURUSD",
  "order_action": "flat",
  "order_contracts": "0.10"
}
```

The exact result can depend on:

* current exposure;
* hedge versus netting;
* quantity;
* position identification;
* connector behavior.

Do not assume that `flat` behaves identically on every destination.

---

# close

`close` is accepted as a close instruction and is normalized into AlgoWay close/flat behavior.

Example:

```json
{
  "platform_name": "metatrader5",
  "ticker": "EURUSD",
  "order_action": "close",
  "order_contracts": "0.10"
}
```

For generic automation, `flat` and `close` should be understood as position-closing commands rather than opposite-direction entries.

---

# closeall

`closeall` tells AlgoWay to close all supported open exposure for the specified ticker.

Example:

```json
{
  "platform_name": "metatrader5",
  "ticker": "XAUUSD",
  "order_action": "closeall"
}
```

Do not include:

```json
"order_contracts"
```

for `closeall`.

`closeall` does not open a new trade.

It is a dedicated close-only action.

Destination support must still be verified.

For MetaTrader 5, public AlgoWay documentation requires AlgoWay EA version 2.10b or newer for `closeall`.

---

# close_side

`close_side` is useful on supported hedge-style routes when only one directional side should be closed.

Supported conceptual values are:

```text
long
short
```

Example:

```json
{
  "platform_name": "metatrader5",
  "ticker": "EURUSD",
  "order_action": "flat",
  "order_contracts": "0.10",
  "close_side": "long"
}
```

This means:

close matching long exposure while leaving short-side exposure intact where the destination implementation supports it.

Without `close_side`, a generic close command can affect all matching exposure depending on connector behavior.

---

# Stop Loss and Take Profit Models

AlgoWay supports two important SL/TP representations.

## Distance-Based

Use:

```text
stop_loss
take_profit
```

Example:

```json
{
  "platform_name": "binance",
  "ticker": "BTCUSDT",
  "order_action": "buy",
  "order_contracts": "0.01",
  "stop_loss": "500",
  "take_profit": "1000"
}
```

These values represent a relative distance according to the selected `sltp_type` and route.

## Absolute Price

Use:

```text
sl_price
tp_price
```

Example:

```json
{
  "platform_name": "bybit",
  "ticker": "BTCUSDT",
  "order_action": "buy",
  "order_contracts": "0.01",
  "sl_price": "98000",
  "tp_price": "105000"
}
```

These represent exact price levels.

---

# Absolute SL/TP Has Priority

If both relative and absolute forms are supplied for the same protection level, absolute price fields take priority.

For example:

```json
{
  "stop_loss": "1",
  "sltp_type": "percent",
  "sl_price": "98000"
}
```

AlgoWay should use:

```text
sl_price = 98000
```

rather than calculating Stop Loss from `stop_loss`.

---

# sltp_type

`sltp_type` defines how relative:

```text
stop_loss
take_profit
```

values should be interpreted.

Current important values are:

```text
pips
percent
```

---

# pips SL/TP Mode

Example:

```json
{
  "platform_name": "binance",
  "ticker": "BTCUSDT",
  "order_action": "sell",
  "order_contracts": "0.01",
  "sltp_type": "pips",
  "stop_loss": "500",
  "take_profit": "1000"
}
```

If `sltp_type` is omitted, AlgoWay's public schema uses `pips` as the default relative SL/TP model.

The actual distance interpretation can still be connector-specific.

---

# Percent SL/TP Mode

Example:

```json
{
  "platform_name": "bybit",
  "ticker": "BTCUSDT",
  "order_action": "buy",
  "order_contracts": "0.01",
  "sltp_type": "percent",
  "stop_loss": "1",
  "take_profit": "2.5"
}
```

Conceptually:

BUY:

Stop Loss 1% below reference price.

Take Profit 2.5% above reference price.

SELL:

Stop Loss above reference price.

Take Profit below reference price.

Percentage SL/TP support is intended for supported cryptocurrency routes.

Do not assume percent mode is available for every broker connector.

---

# trailing_pips

`trailing_pips` requests trailing-stop behavior where the selected route implements it.

Example:

```json
{
  "platform_name": "bybit",
  "ticker": "BTCUSDT",
  "order_action": "buy",
  "order_contracts": "0.10",
  "trailing_pips": "80"
}
```

Trailing is route-dependent.

The presence of the field in AlgoWay JSON does not guarantee that every platform supports trailing.

Current public AlgoWay documentation identifies direct or delegated trailing support on routes including:

* MetaTrader 5
* Binance Futures
* Bybit Futures
* Bitget Futures
* Capital.com
* DXtrade
* Match-Trader

Other routes may ignore or not implement the field.

For MetaTrader 5, trailing can be managed by the AlgoWay EA, which means the terminal and EA must remain running.

---

# leverage

`leverage` can provide per-signal leverage information for supported cryptocurrency connectors.

Example:

```json
{
  "platform_name": "mexc",
  "ticker": "BTCUSDT",
  "order_action": "buy",
  "order_contracts": "0.01",
  "leverage": "5"
}
```

Leverage is not universal.

Depending on the connector:

* leverage can be changed through API;
* leverage can come from connection settings;
* leverage can be used as execution context;
* per-signal leverage can be unsupported.

Do not infer leverage support simply because the generic JSON parser accepts a leverage field.

Always check the destination connector.

---

# trade_type

`trade_type` allows a single message to override the default trading behavior configured for the AlgoWay webhook where supported.

Current machine-level values documented by AlgoWay include:

```text
hedge
netting
opposite
inverse
```

Example:

```json
{
  "platform_name": "metatrader5",
  "ticker": "XAUUSD",
  "order_action": "buy",
  "order_contracts": "0.05",
  "trade_type": "hedge"
}
```

If `trade_type` is omitted, the route's configured default behavior is used.

Do not confuse:

```text
order_action
```

with:

```text
trade_type
```

`order_action` defines what the incoming signal says.

`trade_type` defines how AlgoWay should treat the signal relative to position-management logic.

---

# Execution Modes vs Machine Values

AlgoWay product-level execution concepts include:

* Hedge
* Reverse
* Opposite
* Cutting
* Inverse

The generic webhook `trade_type` field should only use values supported by the current JSON validator and the selected destination.

Do not invent a `trade_type` value merely because a related execution concept exists elsewhere in AlgoWay.

The route's configured behavior can contain logic that is broader than the per-signal JSON override vocabulary.

---

# modify

`modify` manages an existing position or order rather than creating a new entry.

Example:

```json
{
  "platform_name": "metatrader5",
  "ticker": "EURUSD",
  "order_action": "modify",
  "comment": "TV#12345",
  "sl_price": "1.0845",
  "tp_price": "1.0950"
}
```

Possible uses include changing:

* Stop Loss;
* Take Profit;
* other supported management fields.

The connector must be able to identify the existing order or position.

Identification mechanisms are route-dependent.

---

# breakeven

`breakeven` requests supported position management that moves Stop Loss toward or to the entry price.

Example:

```json
{
  "platform_name": "metatrader5",
  "ticker": "GBPUSD",
  "order_action": "breakeven",
  "order_contracts": "0.10"
}
```

`breakeven` does not open a new trade.

Destination support must be checked.

---

# comment

`comment` is an optional user or strategy reference.

Example:

```json
"comment": "TV#12345"
```

Depending on the route it can help with:

* logging;
* debugging;
* order identification;
* EA comments;
* strategy references;
* matching later management commands.

Do not assume every destination preserves comments in the same way.

---

# order_id and Trade Identification

Some workflows support an order, leg or strategy identifier.

Example:

```json
"order_id": "Long Entry 1"
```

AlgoWay can use identifiers where the destination connector supports targeted position or order management.

Identification is especially important when:

* several positions exist on the same ticker;
* hedge mode is used;
* a later modify or close action must target a particular trade;
* multiple strategies share the same destination account.

The availability and semantics of identifiers are connector-specific.

---

# TradingView Strategy Example

```json
{
  "platform_name": "metatrader5",
  "ticker": "{{ticker}}",
  "order_action": "{{strategy.market_position}}",
  "order_contracts": "{{strategy.order.contracts}}",
  "price": "{{close}}"
}
```

TradingView replaces placeholders before sending the webhook.

AlgoWay receives the resulting JSON values.

---

# TrendSpider Example

```json
{
  "platform_name": "metatrader5",
  "ticker": "%alert_symbol%",
  "order_action": "buy",
  "order_contracts": 1,
  "price": "%last_price%",
  "comment": "TS:%alert_name%"
}
```

The AlgoWay command model is not limited to TradingView.

Any external system capable of sending a supported HTTP request can potentially act as a signal source.

---

# Custom Bot Example

A Python application, backend service or external strategy can send a standard AlgoWay command such as:

```json
{
  "platform_name": "bybit",
  "ticker": "ETHUSDT",
  "order_action": "sell",
  "order_contracts": "0.25",
  "sl_price": "4200",
  "tp_price": "3900"
}
```

The source technology is irrelevant to the destination once AlgoWay receives a valid normalized command.

---

# platform_name Values

Common current AlgoWay destination identifiers include:

```text
metatrader5
tradelocker
matchtrader
dxtrade
ctrader
ctrader-oapi
capitalcom
alpaca
tradovate
projectx
binance
bybit
okx
weex
bitget
toobit
bingx
coinbase
hyperliquid
kraken
kucoin
```

Other current AlgoWay routes may exist outside this abbreviated public JSON reference.

Do not invent a `platform_name`.

Use the exact identifier implemented by AlgoWay.

Tradovate is currently supported.

The current catalog uses WEEX and Toobit in place of the retired BitMEX and BitMart routes. Historical articles may still contain the older names.

---

# JSON Validation Rules

A valid AlgoWay webhook command should follow standard JSON syntax.

Important rules:

* one complete JSON object;
* double quotes around keys;
* strings quoted correctly;
* no trailing comma;
* numeric fields must contain parseable numeric values;
* required fields must be present for the requested action;
* `platform_name` must match the route;
* `ticker` must resolve to an instrument;
* `order_action` must be recognized;
* opening orders need a valid quantity;
* `limit` and `stop` need `price`;
* optional fields must use supported values.

Empty optional values should not be relied upon as meaningful commands.

---

# Invalid JSON

Invalid example:

```text
{
  platform_name: "bybit",
  "ticker": "BTCUSDT",
}
```

Problems:

* first key is not quoted;
* trailing comma exists.

AlgoWay can reject malformed JSON before destination execution.

Therefore:

invalid JSON
!= broker rejection

The command may never reach the destination.

---

# Error 415

AlgoWay Error 415 is associated with an incoming request that cannot be processed as the expected structured payload.

Common causes include malformed JSON or an unsupported request format.

Diagnostic order:

Signal source

> > HTTP request
> > content/body format
> > JSON parser
> > AlgoWay validation

Do not troubleshoot the broker before confirming that AlgoWay successfully parsed the command.

---

# Validation vs Execution

A command can be valid JSON and still fail.

There are several distinct levels.

## Level 1: JSON Syntax

Can the request be parsed?

## Level 2: AlgoWay Schema

Are the action and required fields valid?

## Level 3: AlgoWay Route

Is the destination connection configured?

## Level 4: Connector Logic

Can the destination-specific handler translate the command?

## Level 5: Destination Validation