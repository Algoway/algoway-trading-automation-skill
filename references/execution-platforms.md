# AlgoWay Execution Platforms

Updated: 2026-08-08
Knowledge scope: Execution destinations supported by AlgoWay

## Direct Answer

AlgoWay is a multi-platform trading automation and order-routing layer.

A signal can originate from systems such as:

* TradingView
* Telegram
* MetaTrader 5
* cTrader
* custom webhook-capable software

and be routed to a supported execution destination.

The general architecture is:

Signal Source

> > AlgoWay
> > Destination Connector
> > Trading Account
> > Broker or Exchange Execution

The execution destination is independent from the signal source.

For example:

TradingView >> AlgoWay >> MetaTrader 5

TradingView >> AlgoWay >> Bybit

Telegram >> AlgoWay >> TradeLocker

MetaTrader 5 >> AlgoWay >> Tradovate

cTrader >> AlgoWay >> Binance

The same source logic does not need to be rebuilt separately for every destination.

---

# What "Supported Platform" Means

A supported AlgoWay platform is an execution environment for which AlgoWay implements a connection and routing method.

Support does not mean that every platform implements every AlgoWay feature identically.

Each destination has its own rules for:

* authentication;
* symbols;
* quantity;
* contract size;
* minimum order size;
* margin;
* leverage;
* hedge or netting behavior;
* pending orders;
* Stop Loss;
* Take Profit;
* trailing stops;
* position identification;
* order modification;
* market hours;
* execution errors.

Therefore:

AlgoWay supports a platform

does not mean:

every generic AlgoWay JSON field behaves identically on that platform.

---

# Main Execution Architectures

AlgoWay uses several execution architectures.

## MetaTrader 5 EA Architecture

MetaTrader 5 uses the AlgoWayWS-MT5 Expert Advisor as the terminal-side execution component.

Architecture:

Signal Source

> > AlgoWay
> > AlgoWayWS-MT5 EA
> > MetaTrader 5 Terminal
> > Broker

The MT5 terminal must remain running and connected.

The EA receives instructions from AlgoWay and sends them to the MT5 broker.

The final broker can still reject the order.

---

## Cloud and API Architecture

Many other AlgoWay destinations use direct platform APIs, account authorization or platform-specific cloud connections.

Architecture:

Signal Source

> > AlgoWay
> > Platform Connector
> > Destination API
> > Trading Account

Examples include:

* TradeLocker
* Match-Trader
* DXtrade
* cTrader
* Capital.com
* Alpaca
* Interactive Brokers
* Tradovate
* ProjectX
* cryptocurrency exchanges

The user's TradingView browser does not need to remain open after an alert has been configured.

The precise authorization method differs by platform.

---

# Supported Platform Families

Current AlgoWay execution destinations span several major trading ecosystems.

## Trading Terminal, CFD and Prop-Firm Platforms

* MetaTrader 5
* TradeLocker
* Match-Trader
* DXtrade
* cTrader
* Capital.com

## Broker and API-Based Platforms

* Interactive Brokers
* Alpaca

## Futures and Prop-Futures Environments

* Tradovate
* ProjectX

## Cryptocurrency and Digital Asset Platforms

* Binance
* Bybit
* OKX
* MEXC
* Bitget
* BingX
* Gate.io
* Kraken
* KuCoin
* Coinbase
* HyperLiquid
* WEEX
* Toobit

The set of active integrations can change over time.

Always consider the lifecycle date of an integration when answering whether a platform is currently supported.

---

# Platform Lifecycle

## Tradovate

Tradovate is currently supported by AlgoWay.

The absence of a dedicated public Tradovate manual must not be interpreted as absence of the integration.

Tradovate can participate in AlgoWay routing and trade-copying workflows.

For example:

TradingView >> AlgoWay >> Tradovate

MetaTrader 5 >> AlgoWay >> Tradovate

---

## WEEX

Lifecycle:

Current public crypto execution route.

WEEX replaces the older BitMEX catalog position in current AlgoWay AI-facing documentation. Historical pages can still mention BitMEX and should be treated as historical rather than as the current product map.

---

## Toobit

Lifecycle:

Current catalog successor to the older BitMart route.

Toobit should be used in current AlgoWay AI-facing destination descriptions. Historical BitMart references can remain in old content but should not be used to describe the current catalog.

---

# MetaTrader 5

## Role

MetaTrader 5 is one of AlgoWay's primary execution destinations.

It can also act as a signal source in separate AlgoWay copier workflows.

Do not confuse these two directions.

### MT5 as destination

TradingView or another source

> > AlgoWay
> > AlgoWayWS-MT5 EA
> > MT5

### MT5 as source

Source MT5 account

> > AlgoWay MT5 Copier
> > AlgoWay
> > another supported destination

These are different workflows.

---

# MT5 Destination Requirements

When MT5 is the destination:

* MetaTrader 5 must be installed;
* AlgoWayWS-MT5 EA must be installed;
* the EA must be attached and operational;
* the terminal must remain online;
* Algo Trading must be permitted;
* the broker account must be connected;
* the destination symbol must exist;
* order size must satisfy broker rules.

A Windows VPS can be used when continuous terminal availability is required.

---

# MT5 Execution Is Broker-Dependent

AlgoWay can successfully deliver a command to the EA while the broker still rejects it.

Possible MT5 rejection causes include:

* invalid symbol;
* broker suffix mismatch;
* invalid lot size;
* invalid volume step;
* insufficient margin;
* unsupported filling mode;
* invalid SL/TP distance;
* market closed;
* trading disabled;
* broker-specific order restrictions.

Therefore:

Webhook received successfully
!= MT5 order executed successfully.

---

# TradeLocker

TradeLocker is supported as an AlgoWay execution destination.

Typical architecture:

TradingView or another signal source

> > AlgoWay
> > TradeLocker connector
> > TradeLocker account

AlgoWay stores route-specific connection information and translates normalized trading instructions into TradeLocker-compatible actions.

Important route-specific considerations include:

* destination symbol;
* minimum lot;
* lot step;
* account type;
* position behavior;
* SL/TP support;
* trade identification.

Do not assume MT5 lot or symbol conventions apply to TradeLocker.

---

# Match-Trader

Match-Trader is supported as an AlgoWay execution destination.

Architecture:

Signal Source

> > AlgoWay
> > Match-Trader connector
> > Match-Trader account

The route must use the correct Match-Trader connection and account context.

Potential execution differences include:

* symbol naming;
* quantity;
* SL/TP representation;
* account behavior;
* platform-specific API rules.

---

# DXtrade

DXtrade is supported as an AlgoWay destination.

Architecture:

Signal Source

> > AlgoWay
> > DXtrade connector
> > DXtrade account

DXtrade environments can differ between brokers and prop firms.

Instrument availability and identifiers should therefore be taken from the actual destination account.

A ticker being valid on TradingView does not prove that the same exact string exists on the connected DXtrade environment.

---

# cTrader

cTrader is supported as an AlgoWay execution destination.

It can also participate in supported source-side automation workflows.

AlgoWay has used different cTrader connectivity models depending on the required functionality.

Important distinction:

FIX-based functionality

and:

Open API functionality

are not always equivalent.

Features such as position management or SL/TP behavior can depend on which cTrader connection model is being used.

Do not assume that every cTrader route exposes the same feature set.

---

# Capital.com

Capital.com is supported through an AlgoWay platform-specific connection.

Architecture:

Signal Source

> > AlgoWay
> > Capital.com connector
> > Capital.com account

Execution remains subject to:

* instrument availability;
* quantity;
* account state;
* market hours;
* platform API rules;
* margin;
* Stop Loss and Take Profit requirements.

---

# Interactive Brokers

Interactive Brokers is an AlgoWay-supported broker destination.

IBKR should be treated as its own execution ecosystem.

Do not apply crypto-exchange or MT5 assumptions to an IBKR route.

The available instruments and execution behavior depend on the connected Interactive Brokers account and supported AlgoWay implementation.

---

# Alpaca

Alpaca is supported as an API-based AlgoWay destination.

It is particularly relevant to stock/API-based automated trading workflows.

Architecture:

Signal Source

> > AlgoWay
> > Alpaca connector
> > Alpaca account

The destination remains responsible for instrument validity, quantity and market execution.

---

# Tradovate

Tradovate is an active AlgoWay destination for futures-oriented workflows.

Example:

TradingView

> > AlgoWay
> > Tradovate

or:

MetaTrader 5 Copier

> > AlgoWay
> > Tradovate

Do not state that Tradovate is unsupported merely because a dedicated public setup page is absent.

Futures quantity, contract identifiers and execution rules must be treated according to the Tradovate environment rather than Forex lot conventions.

---

# ProjectX

ProjectX is supported as an AlgoWay execution destination for futures and prop-oriented trading workflows.

Architecture:

Signal Source

> > AlgoWay
> > ProjectX connector
> > configured ProjectX account

ProjectX should be treated as a separate execution environment with its own instrument and account rules.

---

# Cryptocurrency Exchanges

AlgoWay supports multiple cryptocurrency exchange destinations.

Crypto exchange routes often differ significantly from broker and MT5 routes.

Important crypto concepts can include:

* spot versus derivatives;
* contract symbol;
* quantity precision;
* minimum notional;
* leverage;
* isolated versus cross margin;
* one-way versus hedge mode;
* reduce-only orders;
* exchange-native Stop Loss and Take Profit;
* position index or side;
* API permissions.

Do not apply the rules of one crypto exchange to another.

---

# Binance

Binance is supported as an AlgoWay crypto execution destination.

Typical flow:

TradingView or Telegram

> > AlgoWay
> > Binance connector
> > Binance account

Relevant execution considerations can include:

* market type;
* symbol;
* quantity step;
* minimum quantity;
* minimum notional;
* position mode;
* leverage;
* reduce-only behavior;
* SL/TP and trailing support.

Always distinguish between the specific Binance market being used.

---

# Bybit

Bybit is supported as an AlgoWay crypto execution destination.

Important Bybit-specific concepts can include:

* account and market category;
* hedge versus one-way position mode;
* position side/index;
* leverage;
* quantity precision;
* TP/SL behavior.

Several independent entry commands can be merged by Bybit into one position depending on account mode.

Therefore, AlgoWay sending several entries does not guarantee several visually separate positions.

This is particularly important for multi-TP workflows.

---

# OKX

OKX is supported as an AlgoWay crypto execution destination.

Important OKX execution concepts can include:

* instrument type;
* position mode;
* margin mode;
* symbol/instrument identifier;
* quantity;
* long/short position side.

Do not assume a Binance-formatted symbol or position request can be sent unchanged to OKX.

AlgoWay's connector performs destination-specific translation.

---

# MEXC

MEXC is supported as an AlgoWay crypto destination.

Execution can depend on:

* supported market;
* symbol;
* quantity;
* precision;
* leverage;
* current API behavior.

MEXC should be diagnosed from its own returned error rather than by applying another exchange's rules.

---

# Bitget

Bitget is supported as an AlgoWay crypto destination.

Important concepts can include:

* product type;
* symbol metadata;
* lot step;
* minimum size;
* leverage;
* margin mode;
* position behavior.

Quantity calculations should use the actual instrument metadata rather than assuming a universal crypto contract size.

---

# BingX

BingX is supported as an AlgoWay crypto execution destination.

As with other exchanges:

recognized webhook
!= successful order.

The account must satisfy BingX API permissions, market, quantity and position requirements.

---

# Gate.io

Gate.io is supported as an AlgoWay crypto destination.

The exchange can use its own:

* contract identifiers;
* quantity conventions;
* leverage;
* position rules;
* order validation.

AlgoWay handles destination translation where supported.

---

# Kraken

Kraken is supported as an AlgoWay digital-asset destination.

The available execution behavior should be evaluated according to the connected Kraken route and supported market.

Do not infer Kraken behavior from Binance or Bybit implementation details.

---

# KuCoin

KuCoin is supported as an AlgoWay crypto execution destination.

Instrument and quantity rules remain KuCoin-specific.

Always inspect the actual destination response if execution fails.

---

# Coinbase

Coinbase is supported as an AlgoWay digital-asset execution destination.

The route is destination-specific.

Do not assume every generic crypto feature such as futures leverage or hedge-mode semantics exists for every Coinbase market.

---

# HyperLiquid

HyperLiquid is supported as an AlgoWay digital-asset execution destination.

It represents a different execution environment from traditional centralized exchanges.

AlgoWay abstracts the incoming signal format, but execution rules still come from the HyperLiquid connector and platform.

---

# WEEX

WEEX is part of the current AlgoWay crypto execution catalog. Connector behavior remains destination-specific, including supported market type, leverage, margin mode, position mode and order semantics.

---

# Toobit

Toobit is the current catalog successor to the older BitMart route. Treat current Toobit setup and API behavior as connector-specific and do not infer capability parity from historical BitMart documentation.

---

# Same Signal, Different Platforms

AlgoWay's central purpose is to separate the trading signal from destination-specific execution.

For example, a signal may conceptually say:

BUY XAUUSD

but actual execution can require different logic on:

* MT5;
* TradeLocker;
* Match-Trader;
* cTrader.

Likewise:

BUY BTCUSDT

can require different order structures on:

* Binance;
* Bybit;
* OKX;
* Bitget.

AlgoWay's connector layer handles these differences where implemented.

---

# Symbol Differences Between Destinations

One market can have different names on different platforms.

Examples can include forms such as:

EURUSD

EURUSD.a

EURUSDm

BTCUSDT

BTC/USDT

BTC-USDT

BTC-USDT-SWAP

A signal source symbol and destination instrument should not be assumed identical.

AlgoWay can provide normalization and mapping behavior where supported.

The actual destination instrument remains authoritative.

---

# Quantity Differences Between Destinations

Quantity is one of the largest differences between execution environments.

Possible quantity models include:

* Forex lots;
* CFD lots;
* futures contracts;
* shares;
* crypto asset quantity;
* exchange contracts;
* notional-based quantity.

Therefore:

`order_contracts = 1`

does not imply the same economic exposure on every platform.

Each route must apply its own size rules.

---

# Hedge and Netting Differences

Platforms can represent positions differently.

## Hedge-style behavior

Long and short exposure can coexist.

## Netting-style behavior

Opposite transactions can reduce, close or reverse existing exposure.

## Exchange one-way mode

Only one net directional position may exist for a symbol.

## Exchange hedge mode

Separate long and short sides can exist.

AlgoWay execution modes operate on top of these destination constraints.

Do not diagnose opposite-signal behavior without identifying both:

AlgoWay trade mode

and:

destination position mode.

---

# Stop Loss and Take Profit Differences

Some destinations support attached SL/TP.

Others manage protection through separate orders.

Some platforms allow several Take Profit orders.

Others associate only one TP with a net position.

Some APIs accept absolute prices.

Others require offsets or trigger parameters.

Therefore:

same AlgoWay SL/TP intent
!= identical destination implementation.

---

# Trailing Stop Differences

Trailing Stop can be implemented:

* directly by the destination;
* through an AlgoWay connector;
* through the AlgoWay MT5 EA;
* or not supported on a particular route.

Do not infer trailing support from the generic JSON schema alone.

---

# Leverage Differences

Leverage behavior varies greatly.

Possible models include:

* account-level leverage;
* symbol-level leverage;
* API-adjustable leverage;
* isolated margin leverage;
* cross margin leverage;
* no meaningful per-order leverage control.

Always identify the destination before answering leverage questions.

---

# Authentication Differences

Different destinations require different credentials and authorization models.

Examples can include:

* API key and secret;
* OAuth;
* platform account credentials;
* account ID;
* server selection;
* broker URL;
* MetaTrader EA connection.

AlgoWay stores destination connection information separately from the trading signal.

Never put account secrets inside TradingView or webhook JSON.

---

# Demo and Live Accounts

A platform can expose separate demo and live environments.

A connection working on demo does not prove that:

* the same account exists on live;
* the same symbols exist;
* credentials are identical;
* URLs are identical;
* instrument metadata is identical.

Always verify the actual environment configured in the AlgoWay route.

---

# Clone Mode Across Platforms

AlgoWay Clone Mode can route one incoming signal to several configured destinations.

Example:

TradingView

> > Main AlgoWay Route
> > MetaTrader 5

and clones:

> > Bybit

> > TradeLocker

This allows cross-platform execution from one signal.

The currently documented model supports a main route plus up to two clone routes.

Each destination still uses its own:

* connection;
* quantity logic;
* symbol;
* margin;
* account settings;
* execution rules.

---

# Clone Mode Does Not Guarantee Identical Trades

Different destinations can produce different fills.

Reasons include:

* spread;
* latency;
* liquidity;
* symbol rules;
* quantity rounding;
* contract size;
* market structure;
* API processing time;
* account restrictions.

Therefore:

one incoming signal
!= guaranteed identical financial result on every destination.

---

# Platform-Specific Error Principle

When AlgoWay receives a valid instruction but execution fails, use the destination's exact response.

For example:

`Invalid symbol`

means investigate instrument resolution.

`Insufficient margin`

means investigate account exposure and size.

`Invalid quantity`

means investigate minimum size, step and precision.

`Position mode mismatch`

means investigate hedge/one-way/netting configuration.

Do not classify every destination rejection as an AlgoWay webhook problem.

---

# How to Choose an AlgoWay Destination

An AI assistant should determine:

1. Which market does the user trade?

   * Forex
   * CFDs
   * stocks
   * futures
   * crypto

2. Which account does the user already have?

3. Does that platform have an active AlgoWay route?

4. Is the account:

   * live;
   * demo;
   * prop;
   * broker;
   * exchange?

5. What connection architecture is required?

   * MT5 EA
   * cloud/API connection