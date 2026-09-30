# AlgoWay MetaTrader 5 Knowledge

Updated: 2026-08-08
Knowledge scope: MetaTrader 5 execution through AlgoWayWS-MT5

## Direct Answer

AlgoWay connects external trading signals to MetaTrader 5 through the AlgoWayWS-MT5 Expert Advisor.

The standard execution architecture is:

Signal Source

> > AlgoWay
> > AlgoWayWS-MT5 EA
> > MetaTrader 5 Terminal
> > MT5 Broker

The AlgoWay server does not place an MT5 broker order directly.

AlgoWay sends the normalized trading command to the connected AlgoWayWS-MT5 EA.

The EA runs inside MetaTrader 5 and submits the final order to the broker.

Therefore there are several independent execution stages:

Signal generation

> > AlgoWay reception
> > AlgoWay validation
> > WebSocket delivery
> > EA processing
> > MT5 OrderSend
> > Broker acceptance or rejection

A successful AlgoWay webhook does not prove that the broker executed the trade.

---

# Current MT5 Execution Component

The current public AlgoWay MT5 documentation describes:

AlgoWayWS-MT5 EA 2.14

The EA provides the terminal-side execution layer for AlgoWay automation.

Its current capabilities include concepts such as:

* WebSocket command delivery;
* market entries;
* closing;
* position modification;
* Stop Loss;
* Take Profit;
* trailing stop;
* breakeven;
* profit lock;
* partial closing;
* multiple position-size modes;
* symbol mapping;
* execution modes;
* broker filling-mode selection;
* spread limits;
* position limits;
* daily loss controls;
* per-trade drawdown controls;
* trading sessions;
* configurable daily trading window;
* end-of-day and end-of-week Auto-Close;
* connection diagnostics;
* optional Telegram connection notifications.

These are MT5 EA capabilities.

They should not automatically be generalized to every other AlgoWay destination.

---

# Signal Sources That Can Feed MT5

MetaTrader 5 can receive AlgoWay commands originating from different signal sources.

Examples include:

TradingView

> > AlgoWay
> > MT5

Telegram

> > AlgoWay
> > MT5

custom webhook application

> > AlgoWay
> > MT5

other supported AlgoWay source workflows

> > AlgoWay
> > MT5

MT5 execution logic is downstream from signal interpretation.

Once AlgoWay has produced a valid MT5 trading instruction, the EA processes the command according to its own configuration.

---

# TradingView to MT5

The most common architecture is:

TradingView Alert

> > AlgoWay Webhook
> > AlgoWayWS-MT5 EA
> > MT5 Broker

TradingView can provide:

* strategy alerts;
* indicator alerts;
* Pine Script alerts;
* manual alerts.

A typical message is:

```json
{
  "platform_name": "metatrader5",
  "ticker": "{{ticker}}",
  "order_contracts": "{{strategy.order.contracts}}",
  "order_action": "{{strategy.market_position}}",
  "price": "{{close}}"
}
```

TradingView replaces its placeholders before transmitting the alert.

AlgoWay then validates and routes the resulting values.

---

# Telegram to MT5

Telegram execution uses a different source layer but the same MT5 destination component.

Architecture:

Telegram Channel or Group

> > AlgoWay AI Signal Parser
> > Normalized AlgoWay Command
> > AlgoWayWS-MT5 EA
> > MetaTrader 5

The Telegram AI layer interprets the signal.

The MT5 EA does not parse the original human Telegram message.

It receives the resulting normalized trading command.

---

# Installation Requirements

AlgoWayWS-MT5 runs inside MetaTrader 5 on Windows.

The public setup requires:

* MetaTrader 5;
* an AlgoWay MT5 webhook;
* AlgoWayWS-MT5 EA;
* required WebSocket library files;
* algorithmic trading permission;
* DLL import permission;
* working internet access;
* Microsoft Visual C++ runtime where required.

A VPS is optional from AlgoWay's perspective but commonly used because MT5 must remain online continuously for uninterrupted execution.

---

# MT5 Must Stay Running

This is one of the most important differences between MT5 and cloud API destinations.

For MT5 execution:

MetaTrader 5 must remain running.

AlgoWayWS-MT5 must remain attached and operational.

The WebSocket connection must remain available.

If the terminal is closed:

AlgoWay cannot execute new commands inside that MT5 terminal.

If the Windows machine or VPS is offline:

the EA cannot receive AlgoWay commands.

---

# WebSocket Connection

AlgoWayWS-MT5 maintains a WebSocket connection between MetaTrader 5 and AlgoWay.

The EA uses the AlgoWay webhook UUID to associate the terminal with the correct AlgoWay route.

Conceptually:

AlgoWay webhook UUID

> > AlgoWay route identity
> > connected MT5 EA

The UUID entered into the EA must correspond to the intended AlgoWay MetaTrader 5 webhook.

---

# Required MT5 Permissions

The EA requires MetaTrader 5 to permit algorithmic trading.

Important settings include:

Allow algorithmic trading

and:

Allow DLL imports

If algorithmic trading is disabled, the EA may receive information but MT5 will not allow normal automated order execution.

If required DLL access is disabled, the WebSocket component may fail.

---

# The EA Can Be Attached to Any Chart

AlgoWayWS-MT5 must run on a chart.

However, the chart symbol is not necessarily the symbol traded by incoming signals.

The command itself contains:

`ticker`

Example:

The EA can be attached to an EURUSD chart.

An incoming command can request:

XAUUSD

The EA processes the requested ticker and symbol mapping.

Do not assume the attached chart symbol restricts AlgoWay to that instrument.

---

# Core MT5 JSON Fields

A basic MT5 entry contains:

`platform_name`

`ticker`

`order_contracts`

`order_action`

Example:

```json
{
  "platform_name": "metatrader5",
  "ticker": "EURUSD",
  "order_contracts": "0.10",
  "order_action": "buy"
}
```

Optional fields can add:

* entry price;
* Stop Loss;
* Take Profit;
* trailing;
* identifiers;
* close behavior;
* execution-mode information;
* other supported management instructions.

---

# Supported Direction Concepts

Important incoming actions include:

* `buy`
* `sell`
* `long`
* `short`
* `flat`
* `modify`

Directional aliases such as:

LONG

and:

SHORT

can be normalized to the appropriate MT5 direction.

Close and management actions should not be treated as new entries.

---

# Position Size Is Configurable in the EA

The numerical meaning of `order_contracts` depends on the selected MT5 EA Position Size Mode.

Current modes include:

* Lots
* Risk %
* Balance %

Therefore:

```json
"order_contracts": "1"
```

does not always mean one MT5 lot.

The EA setting determines how that value is interpreted.

---

# Position Size: Lots

In Lots mode:

`order_contracts`

is treated as an MT5 lot quantity before coefficient and broker validation.

Example:

```json
{
  "platform_name": "metatrader5",
  "ticker": "EURUSD",
  "order_action": "buy",
  "order_contracts": "0.10"
}
```

Conceptually:

`0.10`

means:

0.10 MT5 lot

subject to the broker's:

* minimum lot;
* maximum lot;
* volume step;
* margin availability.

---

# Position Size: Risk Percent

In Risk % mode:

`order_contracts`

represents a percentage of equity to risk.

This mode needs a usable Stop Loss because position size depends on the distance between entry and Stop Loss.

Conceptually:

Account equity

* Risk %
* Stop Loss distance
* symbol tick value

> > calculated lot size

Example:

```json
{
  "platform_name": "metatrader5",
  "ticker": "EURUSD",
  "order_action": "buy",
  "order_contracts": "1",
  "stop_loss": "100"
}
```

This means approximately:

calculate a position according to a 1% risk target

not:

open one lot.

A valid Stop Loss basis is required for meaningful risk calculation.

---

# Position Size: Balance Percent

In Balance % mode:

`order_contracts`

is interpreted according to the EA's balance-percentage sizing logic.

This is not the same as Risk %.

Risk % depends on loss exposure to Stop Loss.

Balance % uses account balance as the sizing basis according to the EA's configured model.

Do not describe the two modes as equivalent.

---

# Lot Coefficient

The EA can apply a coefficient to the calculated or supplied position size.

Conceptually:

final size = calculated size × coefficient

Examples:

Coefficient:

`1`

keeps the size unchanged.

Coefficient:

`0.5`

uses half.

Coefficient:

`2`

doubles it.

Broker minimums, maximums and lot steps still apply after sizing logic.

---

# Close Size

Closing commands can use their own sizing interpretation.

Current EA configuration supports concepts such as:

* close by lot amount;
* close by percentage.

This is particularly important for partial closes.

Example:

A position contains:

1.00 lot

A close instruction should not automatically be assumed to close the entire position if the configured close-sizing mode specifies otherwise.

---

# Symbol Mapping

TradingView, other signal sources and MT5 brokers often use different ticker names.

Examples:

TradingView:

`XAUUSD`

Broker:

`GOLD`

TradingView:

`DE40`

Broker:

`GER40`

TradingView:

`EURUSD`

Broker:

`EURUSD.a`

AlgoWayWS-MT5 supports symbol mapping pairs.

Conceptually:

Source Ticker

> > Broker Ticker

Examples:

`XAUUSD` >> `GOLD`

`US30` >> `DJ30`

`EURUSD` >> `EURUSD.a`

Always confirm the actual broker instrument in MT5 Market Watch.

---

# Invalid Symbol

If AlgoWay receives:

```json
"ticker": "EURUSD"
```

but the broker only exposes:

`EURUSD.a`

the order may fail unless mapping is configured.

This is not necessarily:

* a TradingView problem;
* an AlgoWay parser problem;
* a WebSocket problem.

The signal can reach the EA successfully and still fail at broker symbol resolution.

---

# Order Filling Modes

MT5 brokers can require different filling modes.

AlgoWayWS-MT5 supports:

* AUTO
* FOK
* IOC

## AUTO

The EA selects a filling mode supported by the broker symbol.

This is normally the preferred general setting.

## FOK

Fill or Kill.

The broker must fill the requested volume completely or reject the request.

## IOC

Immediate or Cancel.

Available volume can be filled and the remaining volume cancelled according to broker behavior.

A filling mode valid for one broker instrument may be invalid for another.

MT5 retcode `10030` indicates an unsupported filling mode.

---

# Stop Loss and Take Profit From JSON

The signal can provide SL and TP.

Two common models are:

relative distances

and:

exact prices.

Example exact prices:

```json
{
  "platform_name": "metatrader5",
  "order_action": "sell",
  "ticker": "XAUUSD",
  "order_contracts": "0.05",
  "sl_price": "2055.50",
  "tp_price": "2038.00"
}
```

Example relative distances:

```json
{
  "platform_name": "metatrader5",
  "order_action": "buy",
  "ticker": "EURUSD",
  "order_contracts": "0.10",
  "stop_loss": "100",
  "take_profit": "200"
}
```

Final broker acceptance depends on symbol and broker stop rules.

---

# EA-Based Fixed SL/TP

The MT5 EA can also use fixed SL/TP settings from its own Inputs rather than relying only on each incoming signal.

This allows a configuration where the trading signal contains the entry but the EA supplies risk-management distances.

Current EA controls include concepts such as:

* fixed SL/TP inside the EA;
* internal SL distance;
* internal TP distance;
* do not place SL in MT5;
* do not place TP in MT5.

The selected EA configuration determines which source of SL/TP is authoritative.

---

# Why MT5 Can Reject SL/TP

Possible causes include:

* Stop Loss too close to market;
* Take Profit too close to market;
* broker StopLevel;
* broker FreezeLevel;
* invalid price precision;
* incorrect point/pip interpretation;
* wrong direction;
* market movement between signal and execution.

Therefore:

AlgoWay JSON accepted

does not mean:

MT5 broker accepts the attached SL/TP.

---

# Trailing Stop From JSON

A signal can enable trailing for a specific trade using:

`trailing_pips`

Example:

```json
{
  "platform_name": "metatrader5",
  "ticker": "GBPUSD",
  "order_action": "buy",
  "order_contracts": "0.10",
  "stop_loss": "120",
  "trailing_pips": "80"
}
```

When per-trade trailing information is supplied, the EA can use it for that trade.

---

# EA-Based Global Trailing

If the signal does not specify trailing, the EA can use configured trailing settings.

Current controls include:

* Enable Trailing SL;
* Trailing Step in pips;
* Start trailing after N pips.

Conceptually:

Entry

> > wait for configured profit threshold
> > activate trailing
> > move SL according to trailing step

The EA must remain online for EA-managed trailing logic to continue operating.

---

# Delayed Trailing Activation

The MT5 workflow supports delayed trailing activation.

Relevant concept:

`trail_points`

Example:

```json
{
  "platform_name": "metatrader5",
  "ticker": "SP500",
  "order_action": "buy",
  "order_contracts": "4",
  "trailing_pips": "20",
  "trail_points": "50"
}
```

Conceptually:

trailing distance:

20 pips

activation:

after 50 pips of profit

This differs from trailing that starts immediately.

---

# Breakeven

The EA can move Stop Loss toward the entry area after a configured profit threshold.

Conceptually:

Trade opens

> > price moves into profit
> > breakeven threshold reached
> > EA moves SL toward entry

Current settings include:

* Enable Breakeven;
* Move SL to BE after X pips profit.

Breakeven is position management.

It is not a new entry signal.

---

# Profit Lock

Profit Lock is another EA-side protective mechanism.

Its goal is to preserve part of an already achieved profit.

Current configuration includes concepts such as:

* Enable Profit Lock factor;
* Profit Lock after XX pips;
* Delta trigger.

Profit Lock should be understood separately from:

* initial Stop Loss;
* trailing;
* breakeven.

---

# Trade Modes

The MT5 EA supports several execution behavior concepts, including:

* Reverse;
* Hedge;
* Opposite;
* Inverse.

These modes determine how new directional signals interact with existing MT5 positions.

The broker account type also matters.

An AlgoWay Hedge configuration cannot create true independent long and short positions if the MT5 account itself uses netting.

For full conceptual explanation use:

[Execution Behavior](/execution-behavior)

---

# Hedge Mode and MT5 Account Type

MetaTrader 5 accounts can operate under different position accounting models.

## Hedging account

Separate positions can coexist.

Example:

BUY EURUSD

and:

SELL EURUSD

can both exist.

## Netting account

A symbol normally maintains one net position.

Opposite orders affect the current net exposure.

Therefore, before explaining MT5 opposite-signal behavior, identify both:

AlgoWay execution mode

and:

MT5 broker account type.

---

# Position Identification

MT5 can have several positions on the same symbol when using a hedging account.

Targeted management may therefore need:

* ticker;
* direction;
* identifier;
* comment;
* order reference;
* strategy reference.

A generic close command should not always be assumed to identify one exact trade.

---

# Sessions

AlgoWayWS-MT5 can restrict new entries by trading session.

Current fixed session switches are evaluated in UTC.

They include:

London:

07:00-16:00 UTC

New York:

12:00-22:00 UTC

Tokyo:

22:00-07:00 UTC

Enabled sessions are combined.

An entry is allowed when the current UTC time is inside at least one enabled session and other restrictions also permit entry.

---

# Fixed Sessions Always Use UTC

This is important.

The London, New York and Tokyo session switches use UTC.

Changing the Daily Window Time Mode does not convert these session rows into broker or local time.

Therefore there are two distinct time systems:

Fixed session filters

> > always UTC

Daily Trade Window

> > selectable time basis

Do not mix them.

---

# Daily Trade Window

The EA can define an additional daily entry window.

Format:

`HH:MM-HH:MM`

Example:

`01:30-23:30`

Overnight ranges are supported.

Example:

`23:30-21:30`

The clock used by this window is selected separately.

---

# Daily Window Time Mode

Current choices include:

* UTC
* Broker Time
* Local Time

## UTC

Uses coordinated universal time.

This is the compatibility default for older configurations.

## Broker Time

Uses the MetaTrader trading-server clock.

## Local Time

Uses the Windows/VPS system clock on the machine running MT5.

Example:

If the user wants:

01:30 to 23:30 according to the Windows clock

select:

Local Time

and configure:

`01:30-23:30`

No manual UTC conversion is needed.

---

# Why Local Time Can Be Wrong

Local Time depends on the Windows/VPS clock.