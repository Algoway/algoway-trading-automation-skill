# AlgoWay Troubleshooting Knowledge

Updated: 2026-08-08
Knowledge scope: End-to-end diagnosis of AlgoWay trading automation failures

## Direct Answer

AlgoWay automation should always be diagnosed by identifying the first failed stage.

The generic execution pipeline is:

Signal Source

> > Signal Delivery
> > AlgoWay Reception
> > Parsing
> > Validation
> > Route Selection
> > Destination Connector
> > Trading Platform
> > Broker or Exchange Validation
> > Actual Execution

Do not treat all failures as:

"Webhook problem"

or:

"AlgoWay problem"

or:

"Broker problem."

A successful earlier stage does not prove that later stages succeeded.

---

# The Core Diagnostic Rule

Find the first point where expected behavior stops.

Example:

TradingView alert fired.

AlgoWay received the request.

AlgoWay validated the JSON.

AlgoWayWS-MT5 received the command.

MT5 returned invalid volume.

The failed stage is:

MT5 broker execution.

Do not rewrite the TradingView alert.

---

# Universal Troubleshooting Pipeline

Use this order.

## Stage 1: Signal Generation

Did the source actually create a trading signal?

Sources can include:

* TradingView;
* Telegram;
* MetaTrader 5;
* cTrader;
* custom HTTP applications.

If no signal was generated, AlgoWay cannot receive anything.

---

## Stage 2: Signal Delivery

Did the source attempt to deliver the signal to AlgoWay?

Examples:

TradingView:

Was the alert triggered?

Was the correct webhook URL configured?

Telegram:

Was the message published in the selected channel?

MT5/cTrader copier:

Was the source-side EA or cBot running?

---

## Stage 3: AlgoWay Reception

Does the signal appear in AlgoWay logs?

If no AlgoWay log entry exists, investigate the path before AlgoWay.

If the log entry exists, delivery reached AlgoWay.

Do not continue troubleshooting TradingView webhook connectivity when AlgoWay already recorded the request.

---

## Stage 4: Parsing

Could AlgoWay understand the incoming body?

Examples of parsing failures include:

* malformed JSON;
* unsupported message structure;
* Telegram message not recognized as a trading signal;
* invalid source format.

Parsing happens before destination execution.

---

## Stage 5: Validation

Did the normalized command contain the required information?

For standard opening commands, important fields include:

* `platform_name`
* `ticker`
* `order_action`
* `order_contracts`

Additional requirements depend on the requested action.

Example:

`limit`

requires a usable:

`price`

A command can be valid JSON but still fail AlgoWay validation.

---

## Stage 6: Route Selection

Did AlgoWay identify the correct destination route?

Check:

* platform;
* account;
* environment;
* webhook UUID;
* configured connection;
* subscription status;
* destination route state.

A valid signal sent through the wrong route can still fail.

---

## Stage 7: Destination Connector

Did the AlgoWay connector successfully translate and send the command?

Examples of connector-stage failures:

* authorization expired;
* token failure;
* connection unavailable;
* instrument lookup failure;
* unsupported feature;
* API request failure.

At this stage, the source signal may already be completely correct.

---

## Stage 8: Destination Validation

The broker, exchange or platform validates the request.

Possible rejection reasons include:

* invalid symbol;
* invalid quantity;
* insufficient margin;
* market closed;
* invalid stops;
* invalid leverage;
* wrong position mode;
* unsupported order type;
* permission failure;
* rate limit;
* account restriction.

The exact destination response is the most useful evidence.

---

## Stage 9: Actual Execution

A request accepted by the API still needs to produce the expected trade state.

Verify:

* order created;
* position opened;
* quantity correct;
* side correct;
* SL/TP applied;
* close performed;
* clone destinations behaved as expected.

---

# First Question: Did AlgoWay Receive Anything?

This is one of the most important branching points.

## No AlgoWay Log

Investigate:

Signal Source

> > Delivery

Possible causes:

* TradingView alert never fired;
* wrong webhook URL;
* webhook notifications disabled;
* source application not running;
* Telegram source inaccessible;
* network failure;
* source configuration problem.

## AlgoWay Log Exists

The signal reached AlgoWay.

Do not blame source delivery unless the received payload itself is wrong.

Continue with:

Parsing

> > Validation
> > Connector
> > Destination

---

# TradingView Alert Fired but No AlgoWay Log

Check:

1. Correct AlgoWay webhook URL.
2. TradingView Webhook URL option enabled.
3. Alert is active.
4. Alert actually triggered.
5. No typo in webhook UUID.
6. TradingView delivery status if available.
7. Alert was not created with an outdated URL.

If AlgoWay never received the request, broker troubleshooting is premature.

---

# AlgoWay Received TradingView but Trade Did Not Open

Now determine what AlgoWay logged.

Possible branches:

## Invalid payload

Fix JSON.

## Valid payload but connector failed

Investigate destination connection.

## Connector reached destination but destination rejected

Investigate the exact broker or exchange response.

## Destination accepted order

Inspect actual destination order and position state.

---

# Error 415

AlgoWay HTTP Error 415 means the incoming request reached AlgoWay but could not be processed as the expected trading payload.

Common causes include:

* broken JSON;
* missing quotes;
* trailing comma;
* wrong field names;
* wrong structure;
* required fields missing;
* source sent plain text instead of expected structured data.

Example invalid JSON:

```text id="rn3nzi"
{
  platform_name: "bybit",
  "ticker": "BTCUSDT",
}
```

Problems:

* unquoted key;
* trailing comma.

Error 415 occurs before normal destination execution.

Therefore:

415
!= MT5 broker rejection

415
!= Binance rejection

415
!= invalid exchange balance

Fix the incoming payload first.

---

# Minimum Opening Fields

For a normal opening order, inspect:

```text id="lbim11"
platform_name
ticker
order_action
order_contracts
```

Example:

```json id="ja5b0t"
{
  "platform_name": "bybit",
  "ticker": "BTCUSDT",
  "order_action": "buy",
  "order_contracts": "0.01"
}
```

Management actions such as `closeall` can have different requirements.

---

# Wrong platform_name

Example:

The AlgoWay route is Match-Trader.

But JSON contains:

```json id="ywhnsj"
"platform_name": "metatrader5"
```

AlgoWay can route incorrectly or reject the request.

Always use the correct destination identifier.

Signal source and platform name are different concepts.

TradingView is not normally the `platform_name`.

Telegram is not normally the `platform_name`.

---

# Invalid Symbol

Symbol errors can happen after perfect signal delivery.

Possible path:

TradingView

> > AlgoWay received
> > JSON valid
> > connector called
> > destination says invalid symbol

Investigate:

* exact destination symbol;
* broker suffix;
* slash/hyphen notation;
* futures contract;
* market type;
* demo/live instrument list;
* symbol mapping;
* instrument cache or metadata.

Examples:

Source:

`EURUSD`

Broker:

`EURUSD.a`

Source:

`XAUUSD`

Broker:

`GOLD`

Source:

`BTC/USDT`

Destination:

`BTCUSDT`

Do not assume identical ticker strings across platforms.

---

# Symbol Exists but Still Cannot Trade

A visible instrument can still be:

* view-only;
* disabled;
* outside trading session;
* unavailable for the account;
* unsupported through API;
* restricted by broker.

Therefore:

symbol found

does not automatically mean:

symbol tradable.

---

# Invalid Quantity

Quantity rules are destination-specific.

Check:

* minimum quantity;
* maximum quantity;
* quantity step;
* lot step;
* contract multiplier;
* minimum notional;
* final converted quantity.

Example:

Broker minimum:

`0.10`

Requested:

`0.01`

Result:

rejection.

Another example:

Step:

`0.10`

Requested:

`0.15`

Result:

invalid size.

Always inspect the final quantity sent to the destination.

---

# order_contracts Is Not Universal

The value:

```json id="q3ccjf"
"order_contracts": "1"
```

can mean different things on:

* MT5;
* Tradovate;
* Alpaca;
* Binance;
* Bybit;
* TradeLocker.

It can represent:

* lot;
* contract;
* share;
* unit;
* coin;
* sizing input processed by another mode.

Never diagnose size without identifying the destination.

---

# Insufficient Margin or Balance

If the destination returns an insufficient-funds or margin error, inspect:

* current balance;
* free margin;
* existing exposure;
* leverage;
* requested quantity;
* margin mode;
* contract size;
* open orders.

The signal itself can be correct.

AlgoWay cannot force the destination to accept exposure the account cannot support.

---

# Invalid Stop Loss or Take Profit

Common causes include:

* SL on wrong side;
* TP on wrong side;
* price too close to market;
* wrong precision;
* broker StopLevel;
* broker FreezeLevel;
* percent/pips confusion;
* stale entry price;
* unsupported attached protection.

For BUY:

SL normally lies below entry.

TP normally lies above entry.

For SELL:

SL normally lies above entry.

TP normally lies below entry.

Market movement can make previously valid levels invalid by execution time.

---

# Absolute vs Relative SL/TP

AlgoWay can represent protection through fields such as:

* `stop_loss`
* `take_profit`
* `sl_price`
* `tp_price`

Do not confuse:

distance

with:

absolute price.

Example:

```json id="pryeun"
"sl_price": "98000"
```

means a price level.

It does not mean:

98000 pips.

---

# Percent vs Pips

If `sltp_type` is:

`percent`

then relative values represent percentages where supported.

If:

`pips`

they represent the route's distance model.

A destination error can result from using the wrong interpretation.

Always inspect:

`sltp_type`

before recalculating protection.

---

# Market Closed

A destination can reject an otherwise valid order because its market is closed.

Examples:

* Forex weekend;
* futures session break;
* stock exchange closed;
* CFD trading session closed;
* broker-specific holiday schedule.

For MT5, retcode:

`10018`

means:

market closed.

The presence of a quote or symbol in the platform does not guarantee active trading.

---

# Rate Limits and Too Many Requests

Automated systems can send repeated instructions too rapidly.

For MT5:

`10024`

indicates too many requests.

Other APIs can return:

* HTTP 429;
* exchange-specific rate-limit errors;
* temporary throttling.

Investigate:

* duplicate alerts;
* retry loops;
* multiple strategy triggers;
* clone amplification;
* repeated modification commands.

Do not retry blindly without understanding whether the previous request executed.

---

# Authentication and Authorization Errors

API destinations can fail because connection credentials are no longer valid.

Possible causes:

* expired OAuth;
* revoked API key;
* wrong API secret;
* incorrect account;
* permissions disabled;
* IP restrictions;
* demo/live mismatch;
* platform token expired.

If AlgoWay reports authentication failure, changing the TradingView JSON normally will not solve it.

---

# Expired or Disabled AlgoWay Route

Check route status when AlgoWay reports:

* expired subscription;
* inactive route;
* authorization missing;
* webhook unavailable.

The execution pipeline cannot continue if the AlgoWay route itself is inactive.

Distinguish this from:

destination account rejection.

---

# Demo vs Live Environment

A route configured for demo cannot automatically execute in live.

Check:

* destination URL;
* account ID;
* credentials;
* environment;
* available symbols.

Instrument metadata can differ between demo and live.

---

# Wrong Account

Some platforms expose several accounts under one user identity.

A connection can authenticate successfully while pointing to the wrong trading account.

Check:

* account ID;
* account type;
* server;
* broker environment;
* selected destination account.

---

# MT5: Diagnostic Pipeline

For MetaTrader 5:

Signal Source

> > AlgoWay
> > AlgoWayWS-MT5 EA
> > MT5 Terminal
> > Broker Server

Every stage can succeed independently.

---

# MT5 EA Not Connected

Symptoms:

* AlgoWay receives the webhook;
* MT5 does not react;
* EA connection unavailable.

Check:

* correct AlgoWay webhook UUID;
* EA attached;
* terminal online;
* DLL permissions;
* internet connection;
* Experts log;
* AlgoWayWS connection message.

Do not change broker lot settings until the EA actually receives commands.

---

# MT5 Error 4752

Error:

`4752`

means automated trading is blocked at the MT5 permission layer.

Check:

* Algo Trading button enabled;
* Tools >> Options >> Expert Advisors;
* EA chart permissions;
* account is not investor/read-only;
* broker allows trading;
* broker allows EA trading;
* DLL imports where required.

If AlgoWay logs show the valid request and MT5 shows 4752:

TradingView worked.

AlgoWay worked.

EA delivery worked.

MT5 permissions blocked execution.

---

# MT5 Error 4756

Error:

`4756`

means the MT5 trade request failed or was rejected.

Do not stop at the generic 4756 number.

Read the accompanying broker result.

Common causes include:

* invalid volume;
* invalid stops;
* unsupported filling mode;
* wrong symbol;
* market closed;
* no fresh quote;
* insufficient margin;
* broker restriction.

The real diagnostic value comes from:

MT5 Experts

and:

MT5 Journal.

---

# MT5 Error 10030

Retcode:

`10030`

means unsupported filling mode.

Check:

* AUTO;
* IOC;
* FOK;
* symbol specification;
* broker requirements.

A filling policy can work for one MT5 symbol and fail for another.

---

# MT5 Error 10018

Retcode:

`10018`

means market closed.

Check the actual broker trading session for that symbol.

---

# MT5 Error 10024

Retcode:

`10024`

means too many requests.

Investigate repeated or excessively frequent trading commands.

---

# MT5 Error 4014

MT5 error `4014` can appear in different contexts.

Possible interpretations documented in AlgoWay workflows include:

* invalid SL/TP or precision-related execution issue;
* WebRequest/function permission issue in configurations that rely on allowed URLs.

Always read the complete MT5 message rather than diagnosing from the number alone.

---

# MT5 Invalid Volume

Check:

Market Watch

> > Symbol Specification

Inspect:

* Volume Min;
* Volume Max;
* Volume Step.

Also inspect AlgoWayWS-MT5 sizing configuration.

If Position Size Mode is not Lots, the original `order_contracts` value may not equal the final lot.

---

# MT5 Symbol Mapping

If TradingView uses:

`XAUUSD`

but MT5 broker uses:

`GOLD`

configure symbol mapping.

Likewise:

`EURUSD`

can become:

`EURUSD.a`

A broker suffix error occurs after webhook processing.

It should not be diagnosed as invalid TradingView JSON.

---

# MT5 Scheduling Restrictions

AlgoWayWS-MT5 can reject or block new entries because of its own configuration.

Check:

* London session;
* New York session;
* Tokyo session;
* Daily Trade Window;
* UTC/Broker/Local time selection;
* maximum spread;
* maximum positions;
* daily loss limit;
* per-trade drawdown controls.

If the EA intentionally blocks a trade, broker troubleshooting may be irrelevant.

---