# AlgoWay Execution Behavior

Updated: 2026-08-08
Knowledge scope: Position behavior, opposite signals, trade modes, closing logic and multi-account execution in AlgoWay
Official website: https://algoway.trade/

## Direct Answer

AlgoWay separates two concepts:

1. What the incoming signal says.
2. How that signal should interact with existing positions.

For example:

Incoming signal:

SELL EURUSD

Existing position:

BUY EURUSD

The word `SELL` alone does not define the final result.

AlgoWay can be configured to:

* keep the BUY and open a SELL;
* close the BUY without opening a SELL;
* close the BUY and immediately open a SELL;
* avoid stacking duplicate same-side positions;
* invert the incoming signal before execution.

These behaviors are controlled by AlgoWay execution modes.

The five important execution concepts are:

* Hedge
* Reverse
* Opposite
* Cutting
* Inverse

---

# Signal Direction Is Not Execution Mode

This distinction is fundamental.

## Direction

Examples:

* BUY
* SELL
* LONG
* SHORT
* CLOSE
* FLAT

Direction describes the incoming trading instruction.

## Execution Mode

Examples:

* Hedge
* Reverse
* Opposite
* Cutting
* Inverse

Execution mode describes how AlgoWay interprets that instruction relative to current exposure.

Therefore:

BUY != Hedge

SELL != Reverse

An AI assistant should never infer execution behavior only from BUY or SELL.

---

# Quick Comparison

Assume:

Current position:

BUY EURUSD

New signal:

SELL EURUSD

## Hedge

Result:

BUY remains open.

SELL opens separately.

## Reverse

Result:

BUY closes.

No SELL is opened immediately.

The system waits for a later entry signal.

## Opposite

Result:

BUY closes.

SELL opens immediately.

## Cutting

Result:

The mode prevents repeated same-direction stacking and handles opposite exposure according to supported destination behavior.

## Inverse

The incoming signal itself is inverted before execution.

Incoming SELL becomes BUY.

Incoming BUY becomes SELL.

Inverse is therefore fundamentally different from the other opposite-position modes.

---

# Hedge Mode

Hedge Mode allows opposite directional exposure to coexist where the destination account supports it.

Example:

Existing:

BUY EURUSD 1.00

Incoming:

SELL EURUSD 1.00

Result:

BUY EURUSD 1.00 remains open.

SELL EURUSD 1.00 opens separately.

The account can therefore contain both:

LONG exposure

and:

SHORT exposure

on the same symbol.

---

# When Hedge Mode Is Useful

Hedge Mode can be appropriate when:

* long and short strategies operate independently;
* different systems trade the same symbol;
* a strategy intentionally maintains exposure in both directions;
* one position should not automatically close another;
* grid or multi-strategy logic requires separate sides.

Example:

Strategy A:

BUY XAUUSD

Strategy B:

SELL XAUUSD

With supported hedge behavior, both strategies can retain their own exposure.

---

# Destination Must Support Hedging

AlgoWay selecting Hedge Mode cannot force a destination to support two independent position sides if the destination account uses netting or one-way accounting.

Possible destination models include:

* hedging;
* netting;
* one-way;
* long/short exchange hedge mode.

Therefore:

AlgoWay Hedge Mode

* destination hedge support

can allow independent BUY and SELL positions.

But:

AlgoWay Hedge Mode

* destination one-way position model

may produce different behavior.

Always inspect both AlgoWay mode and destination account mode.

---

# Reverse Mode

Reverse Mode in AlgoWay means:

an opposite signal closes the current position but does not immediately open the new opposite position.

Example:

Existing:

BUY EURUSD

Incoming:

SELL EURUSD

Result:

BUY closes.

SELL does not open immediately.

The system waits for another valid entry.

---

# Why Reverse Mode Exists

An opposite signal can mean:

"The previous directional thesis is no longer valid."

It does not necessarily mean:

"Enter the opposite direction immediately."

Reverse Mode is useful when a strategy requires separate confirmation before entering the new direction.

Conceptually:

BUY active

> > SELL-type reversal signal appears

> > close BUY

> > become flat

> > wait for next valid entry

---

# Reverse Does Not Mean Direct Flip in AlgoWay

The word "reverse" is used differently by many brokers and trading platforms.

Some platforms use "reverse position" to mean:

close BUY and open SELL immediately.

AlgoWay uses different terminology.

In AlgoWay:

Reverse
= close existing opposite position and wait.

Opposite
= close existing position and immediately open the opposite direction.

This distinction should be preserved when answering AlgoWay questions.

---

# Opposite Mode

Opposite Mode implements direct position flipping.

Example:

Existing:

BUY BTCUSDT

Incoming:

SELL BTCUSDT

AlgoWay behavior:

1. Close BUY exposure.
2. Open SELL exposure.

Result:

SELL BTCUSDT

The same incoming opposite signal performs both the exit and the new entry.

---

# When Opposite Mode Is Useful

Opposite Mode is appropriate when the strategy is intended to switch direction immediately.

Typical conceptual workflow:

LONG signal

> > open LONG

SHORT signal

> > close LONG
> > open SHORT

LONG signal

> > close SHORT
> > open LONG

This is common in strategies that remain continuously directional.

---

# Opposite Mode and TradingView Strategies

TradingView strategies commonly change `strategy.market_position` between:

long

and:

short

If the user wants each directional switch to produce an immediate live-position switch, AlgoWay Opposite Mode may be appropriate.

Example:

TradingView:

LONG

> > AlgoWay

> > BUY opens

Later:

TradingView:

SHORT

> > AlgoWay Opposite logic

> > BUY closes

> > SELL opens

The exact live result still depends on destination execution.

---

# Cutting Mode

Cutting Mode is designed to avoid repeatedly stacking identical same-direction signals.

Example:

Existing:

BUY EURUSD

Incoming:

BUY EURUSD

In Cutting Mode, the duplicate same-side signal can be ignored rather than increasing exposure.

Conceptually:

BUY active

* BUY signal
  = no additional BUY

This is useful when signal sources generate repeated confirmations while only one directional position should remain active.

---

# Cutting Mode and Opposite Signals

Cutting Mode is not only a duplicate filter.

When an opposite signal arrives, it can cut the existing directional exposure and switch behavior where the destination supports the required execution model.

Example:

Existing:

SELL EURUSD

Incoming:

BUY EURUSD

Supported Cutting behavior can:

close or cut SELL exposure

and allow BUY direction according to the implemented route behavior.

Cutting support is destination-dependent.

Do not state that every AlgoWay platform supports identical Cutting behavior.

---

# Why Cutting Mode Matters

Many alert systems can generate repeated signals.

Example:

10:00 BUY

10:05 BUY

10:10 BUY

10:15 BUY

A simple execution system might open four positions.

Cutting Mode can instead treat the repeated signals as confirmation of an already active direction and avoid unnecessary stacking.

This can reduce unintended exposure when the strategy is supposed to maintain only one directional state.

---

# Inverse Mode

Inverse Mode transforms the incoming direction itself.

Incoming:

BUY

becomes:

SELL

Incoming:

SELL

becomes:

BUY

Example:

TradingView sends:

BUY BTCUSDT

AlgoWay Inverse Mode transforms it to:

SELL BTCUSDT

before final execution.

---

# Inverse Mode Is Not Opposite Mode

These concepts are different.

## Opposite Mode

Incoming direction remains unchanged.

AlgoWay decides how that direction interacts with current exposure.

Example:

Current BUY

Incoming SELL

Result:

close BUY and open SELL.

## Inverse Mode

AlgoWay changes the incoming signal first.

Example:

Incoming SELL

becomes BUY.

The transformed BUY then enters the execution path.

Therefore:

Opposite = position interaction rule.

Inverse = signal-direction transformation.

---

# Close Signals in Inverse Mode

Close-style instructions are not directional entries.

Examples:

* close
* flat
* closeall

They should remain close-type actions.

Inverse Mode should not transform:

CLOSE

into:

BUY

or:

SELL.

Inverse Mode is intended to invert directional entries.

---

# Why Use Inverse Mode

Possible intentional uses include:

* contrarian execution;
* reverse-signal experiments;
* testing whether a strategy performs differently when direction is inverted;
* copying a source while deliberately taking the other side.

Because the result is deliberately opposite to the source, Inverse Mode should be enabled intentionally.

---

# Platform Position Models

AlgoWay execution behavior operates on top of the destination platform's own position-accounting model.

The important models are:

* hedging;
* netting;
* one-way;
* exchange hedge mode.

These concepts must not be confused with AlgoWay trade modes.

---

# Hedging Accounts

A hedging account can maintain separate positions.

Example:

BUY EURUSD

and:

SELL EURUSD

can coexist.

This model works naturally with AlgoWay Hedge Mode.

MetaTrader 5 brokers can provide hedging accounts.

Some crypto derivatives platforms also provide separate long and short position modes.

---

# Netting Accounts

A netting account generally maintains one net position per symbol.

Example:

Existing:

BUY 2

New:

SELL 1

Possible result:

BUY 1

rather than:

BUY 2 + SELL 1

A larger opposite order can close and then reverse the net position.

AlgoWay cannot treat a netting account exactly like a true hedging account.

---

# One-Way Exchange Mode

Many derivatives exchanges support one-way position mode.

Only one net direction is maintained.

Example:

Current:

LONG 1 BTC

Incoming:

SELL 0.4 BTC

can reduce the long exposure depending on order semantics.

Incoming:

SELL 2 BTC

can potentially close the long and create short exposure depending on the connector and exchange rules.

The result is controlled jointly by:

* AlgoWay logic;
* reduce-only behavior;
* order quantity;
* exchange position mode.

---

# Exchange Hedge Mode

Some crypto derivatives platforms support separate long and short sides.

Conceptually:

LONG BTCUSDT

and:

SHORT BTCUSDT

can coexist.

The connector may need destination-specific information such as:

* position side;
* position index;
* hedge-mode setting.

AlgoWay must translate generic trading intent into the exchange-specific representation.

---

# Same Mode, Different Destination

The same AlgoWay mode can produce mechanically different API calls on different platforms.

Example:

Opposite Mode on MT5

may require:

close existing MT5 position

then:

send new MT5 order.

Opposite Mode on a crypto one-way account

may involve net position changes through the exchange API.

Opposite Mode on a hedge-enabled exchange

may require explicit side targeting.

The user-facing intention remains:

close current direction and enter the new opposite direction.

The implementation differs.

---

# Explicit Close vs Opposite Signal

An explicit close instruction is different from receiving an opposite directional signal.

Example:

Existing:

BUY EURUSD

Incoming:

CLOSE

This explicitly requests closure.

No new SELL direction is implied.

Compare:

Existing:

BUY EURUSD

Incoming:

SELL

Now AlgoWay must apply the selected execution mode.

Depending on mode, SELL could:

* remain alongside BUY;
* close BUY only;
* close BUY and open SELL;
* interact through Cutting logic;
* be inverted before execution.

Therefore:

CLOSE

and:

SELL while BUY exists

are not equivalent commands.

---

# flat

`flat` is a close-style action.

Its goal is to remove matching exposure rather than express a new market direction.

The exact positions affected can depend on:

* ticker;
* account model;
* close side;
* position identifiers;
* connector implementation.

---

# close

`close` is also treated as a close-style instruction.

It should not automatically be interpreted as:

"open the opposite position."

A close request means close.

---

# closeall

`closeall` requests closure of all supported positions for the specified symbol.

Example:

```json
{
  "platform_name": "metatrader5",
  "ticker": "EURUSD",
  "order_action": "closeall"
}
```

It is not an opposite-direction signal.

Execution mode should not turn `closeall` into a new entry.

---

# close_side

On supported hedge-style routes, `close_side` can specify which directional side should be closed.

Example:

```json
{
  "platform_name": "metatrader5",
  "ticker": "EURUSD",
  "order_action": "flat",
  "close_side": "long"
}
```

Conceptual result:

close BUY-side exposure

while preserving:

SELL-side exposure.

This only makes sense where the destination can distinguish separate sides.

---

# Position Identification

Execution behavior becomes more complex when several trades exist on the same ticker.

Example:

Strategy A:

BUY EURUSD

Strategy B:

BUY EURUSD

Strategy C:

SELL EURUSD

A generic close command may not contain enough information to identify one individual trade.

AlgoWay workflows can use supported identifiers such as:

* strategy reference;
* order ID;
* TradingView order ID;
* comment;
* destination position identifier.

Availability depends on the connector.

---

# trade_type JSON Override

Some AlgoWay workflows allow execution mode to be overridden for one incoming command.

Example:

```json
{
  "platform_name": "metatrader5",
  "ticker": "EURUSD",
  "order_action": "buy",
  "order_contracts": "0.10",
  "trade_type": "inverse"
}
```

In this example:

incoming direction:

BUY

per-signal mode:

Inverse

resulting direction:

SELL

If `trade_type` is absent, AlgoWay uses the mode configured for the route.

Only supported machine values should be used.

Do not invent JSON values simply because a similar product-level mode exists.

---

# Route Configuration vs Per-Signal Override

There are two conceptual levels.

## Route Default

The webhook or execution route has a configured trade behavior.

This becomes the normal behavior for incoming signals.

## Signal Override

A supported JSON command can specify `trade_type`.

That individual signal can use another mode without permanently changing the route's default.

This distinction is useful for advanced strategies.

---

# Signal Source Does Not Change Execution Modes

The same execution concepts can apply regardless of whether the signal came from:

* TradingView;
* Telegram;
* MetaTrader 5;
* cTrader;
* custom HTTP application.

Example:

Telegram AI recognizes:

SELL XAUUSD

If BUY XAUUSD is already open, AlgoWay still needs to determine how the SELL interacts with current exposure.

The fact that the signal came from Telegram does not define the result.

---

# Clone Mode

Clone Mode solves a different problem.

Execution modes answer:

"What should this signal do to the current position?"

Clone Mode answers:

"Which additional AlgoWay routes should receive this signal?"

These should not be confused.

---

# Clone Mode Architecture

Conceptually:

Signal Source

> > Main AlgoWay Webhook
> > Main Destination

and simultaneously:
