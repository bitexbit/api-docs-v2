---
title: API Docs v2 | bitextbit

# language_tabs: # must be one of https://git.io/vQNgJ
#   - shell
#   - ruby
#   - python
#   - javascript

# toc_footers:
#   - <a href='#'>Sign Up for a Developer Key</a>
#   - <a href='https://github.com/slatedocs/slate'>Documentation Powered by Slate</a>

# includes:
#   - errors

search: true

code_clipboard: true
---

# Introduction

Welcome to the bitexbit API docs!

Service [bitexbit.com](https://bitexbit.com) provides open API for trading operations and broadcasting of all trading events. 

# HTTP API (v2)

HTTP API endpoint available on **`https://bitexbit.com/api/v2/`**.

Some methods requires authorization read below.

<aside class="notice">
 We have limit in 2 calls per second from single account to authorization required methods and 100 calls per secong from single IP address to public methods.
</aside>

All HTTP methods accept `JSON` formats of requests and responses if it not specified by headers.

## Authentication

There are few types of authentication

- `FREE` - no authentication need
- `FULL` - need api key and signature

For `FULL` authentication you need:

- Generate API `key` and `secret` in [Profile Api](https://bitexbit.com/profile/api/profile/api).
- Make `nonce` - a number that must be unique and must increase with each call to the API
- Make `payload` - string, concatenated with `path`, `nonce`, and `body`. Where
  `path` - relative path without query params (example: `/api/v2/private/balance`) and
  `body` - for `POST` request - json string from request data, for `GET` request empty string.
- Make `signature` from `payload` and `secret`, using HMAC Authentication with SHA-256.
- Send next HTTP headers: `X-API-Key`, `X-Nonce`, `X-Signature`,
  where `X-API-Key` - `key`, `X-Nonce` - `nonce`, `X-Signature` - `signature` as hexadecimal string.


# HTTP API (v1)

Read more about API v1 [here](https://bitexbit.github.io/api-docs-v1)

# Success responses

Code | Meaning
-----|--------
200  | Returns is response is success. Body of response usually contains object that affected.
<!-- ## HTTP 200

Returns is response is success. Body of response usually contains object that affected.

Here are also batch endpoints. -->

# Errors

We are using standard HTTP statuses in responses.

| Error Code | Meaning                                                                                                                                                              |
| ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 400        | If response has http status 400 that means that client should change some input parameters to make it success. What exactly wrong returned in response body.         |
| 401        | In case when HTTP endpoint require authentication but provided authentication headers not valid - response with status 401 will be returned and the details in body. |
| 403        | You do not have access to do the operation. For example when you try to cancel orders that was created by other user.                                                |
| 429        | We have different rate limits to call API methods. If you reached this - response with status 429 will be returned.                                                  |
| 500        | Nobody is safe to make mistakes. This is the case when error happened on serer side. No details but we are see it.                                                   |
| 502        | Looks like we under high load. Try your request later.                                                                                                               |

# Public methods

Here all method you can use without any credentials:

- [Currencies](#currencies)
- [Pairs](#pairs)
- [Open-High-Low-Close-Volume](#open-high-low-close-volume)
- [Tickers](#tickers)
- [Orderbook Depth](#orderbook-depth)
- [Public Orders](#public-orders)
- [Public Trades](#public-trades)

## Currencies

Returns list of currencies on the exchange. Each element contains info describing currency.

### Request

`GET /api/v2/public/currencies`

### Authentication

`FREE`

### Response

> Response Example

```json
[
  {
    "code": "BTC",
    "name": "Bitcoin",
    "image_url": "https://bitexbit.com/media/currency/BTC/l.TpjkNtFB2Q9QQbBO.png",
    "precision": 8,
    "is_enabled": true
  },
  {
    "code": "ETH",
    "name": "Ethereum",
    "image_url": "https://bitexbit.com/media/currency/ETH/l.MN2H93w4xXpDIRhX.png",
    "precision": 5,
    "is_enabled": true
  }
]
```

Each object in array has next structure:

Key | Type | Description
----|------|------------
code       | string  | short **unique** name of currency
name       | string  | full name of currency
image_url  | string  | relative url to image of currency
precision  | integer | how many decimal places supported for currency
is_enabled | boolean | is currency enabled (can be temporary or permanent as well)

### Currency details

### Request

`GET /api/v2/public/currency/<code>`

### Query Parameters

Code | Type | Description
---- | ---- | -----------
code | string | short **unique** name of currency


## Pairs

Returns list of trading pairs listed on exchange. Each element contains info describing pair.

### Request

`GET /api/v2/public/pairs`

### Authentication

`FREE`

### Response

> Response Example

```json
[
  {
    "symbol": "BTC_USD",
    "label": "BTC/USD",
    "base": "BTC",
    "quote": "USD",
    "price_precision": 3,
    "amount_precision": 6,
    "min_amount": "0.00001000",
    "max_amount": "100000000",
    "min_value": "0.00001000",
    "can_create_order": true,
    "can_cancel_order": true,
    "is_enabled": true
  },
  {
    "symbol": "LTC_USD",
    "label": "LTC/USD",
    "base": "LTC",
    "quote": "USD",
    "price_precision": 4,
    "amount_precision": 8,
    "min_amount": "0.00001000",
    "max_amount": "100000000",
    "min_value": "0.00001000",
    "can_create_order": true,
    "can_cancel_order": true,
    "is_enabled": true
  }
]
```

Each object in array has next structure:


Key | Type | Description
--- | ---- | -----------
symbol | string | **unique** pair name, that build from first and second currency symbols divided by underscore symbol
label | string | human readable pair name
base | string | symbol of first currency. all amount in orders and trades measured in this currency
quote | string | symbol of second currency
price_precision | integer | ow many decimal digits can be in `price` when placing order
amount_precision | integer |  how many decimal digits can be in `amount` when placing order
min_amount | decimal string | only acceptable `amount` greater or equal this value when placing order
max_amount | decimal string | only acceptable `amount` lesser or equal this value when placing order
min_value | decimal string | minimal value of order in `quote_currency` (`amount` multiply by `price`)
can_create_order | boolean | can order be places or not by this pair
can_cancel_order | boolean | can order be cancelled or not by this pair
is_enabled | boolean | is pair enabled, orders can be places or canceled (can be temporary or permanent as well)

### Pair Details

`GET /api/v2/public/pairs/<symbol>`

Where `symbol` - string; identifier of pair

## Open-High-Low-Close-Volume

Returns list of candles by specified candle type and pair

### Request

`GET /api/v2/public/ohlcv`

### Authentication

`FREE`

### Parameters

Object with next items:

- `symbol` - string; identifier of pair **Required**
- `timeframe` - string; time interval of candle. Allowed values are `5` (15 min), `15` (15 min), `30` (30 min), `60` (1 hour), `240` (4 hours), `D` (24 hours). **Required**
- `since` - integer; timestamp, minimal `date` of candle
- `until` - integer; timestamp, maximal `date` of candle
- `limit` - integer; max number of results, by default is 100, max value is 1000

> Response Example

```json
[
  [
    1477958400,
    "715.00000000",
    "720.20000000",
    "715.00000000",
    "717.00000000",
    "0.01924864"
  ],
  [
    1478044800,
    "717.00000000",
    "717.50000000",
    "712.00000000",
    "716.50000000",
    "0.08771271"
  ]
]
```

### Response

Array of arrays; Each array in response has next items:

- `1` - integer; timestamp in milliseconds, date of opening candle
- `2` - decimal string; price at the time of opening the candle
- `3` - decimal string; maximal price
- `4` - decimal string; minimal price
- `5` - decimal string; price at the time of closing the candle
- `6` - decimal string; trading volume in currency1 of pair

## Tickers

Returns list of tickers objects for all trading pairs

### Request

`GET /api/v2/public/tickers`

### Authentication

`FREE`

### Parameters

Object with next items:

- `symbols` - string; identifier of pair or multiple pair identifiers, separated by comma, example - `BTC_USD,BTC_USDT`. Max number of pairs is 5.

### Response

> Response Example

```json
[
  {
    "date": 1547535929556,
    "symbol": "BTC_USD",
    "last": "3781.33900000",
    "buy": "3781.33900000",
    "sell": "3781.94500000",
    "change": "0.00000000",
    "vol1": "0.00000000",
    "vol2": "0.00000000",
    "high": "0.00000000",
    "low": "0.00000000"
  },
  {
    "date": 1547535923368,
    "symbol": "ETH_USD",
    "last": "132.33000000",
    "buy": "132.08000000",
    "sell": "132.33000000",
    "change": "0.00000000",
    "vol1": "0.00000000",
    "vol2": "0.00000000",
    "high": "0.00000000",
    "low": "0.00000000"
  }
]
```

Each object in array has next structure:

- `date` - integer; milliseconds timestamp when was last activity on market
- `symbol` - string; identifier of pair
- `low` - decimal string; lowest trade price last 24 hours
- `high` - decimal string; highest trade price last 24 hours
- `change` - decimal string; difference in percent between price that was 24 hours ago and latest trade; if no trades last 24 hours it will be `0`
- `last` - decimal string; price of last trade of pair
- `vol1` - decimal string; 24 hour trading volume on pair measured in first currency
- `vol2` - decimal string; 24 hour trading volume on pair measured in second currency
- `buy` - decimal string; current top buy price
- `sell` - decimal string; current top sell price

## Orderbook Depth

Get the depth of orderbook.

### Request

`GET /api/v2/public/depth`

### Authentication

`FREE`

### Parameters

Object with next items:

- `symbol` - string; identifier of orderbook. **Required**
- `limit` - integer; limit the number of returned records for each bids and asks. Maximum **50**

### Response

> Response Example

```json
{
  "date": 1568905003.866091,
  "asks": [
    [1, 1],
    [2, 4]
  ],
  "bids": [
    [1, 1],
    [2, 4]
  ],
  "sum_bids": 5,
  "sum_asks": 5
}
```

Object with next items:

- `date` - float; current timestamp
- `symbol` - string; current timestamp
- `asks` - array of pairs of aggregated price and amount
- `bids` - array of pairs of aggregated price and amount
- `sum_bids` - float; total amount of bids in orderbook
- `sum_asks` - float; total amount of asks in orderbook

## Public Orders

Returns public orders list

### Request

`GET /api/v2/public/orders`

### Authentication

`FREE`

### Parameters

- `symbol` - string; any symbol id
- `type` - string; buy-limit, sell-limit
- `status` - string; active, canceled, done
- `since` - integer; timestamp, minimal `date` of record
- `until` - integer; timestamp, maximal `date` of record
- `since_id` - integer; minimal `id` of record
- `until_id` - integer; maximal `id` of record
- `limit` - integer; max number of results, by default is 100, max value is 1000

### Response

> Response Example

```json
[
  {
    "id": 34,
    "date": 1478036172902,
    "type": "buy-limit",
    "symbol": "BTC_USD",
    "price": "433.00000000",
    "amount": "0.00692841",
    "amount_unfilled": "0.00692841",
    "amount_filled": null,
    "amount_cancelled": null,
    "value_filled": null,
    "price_avg": null,
    "done_at": null,
    "state": "submitted"
  },
  {
    "id": 28,
    "date": 1478033710751,
    "type": "buy-limit",
    "symbol": "BTC_USD",
    "price": "55.00000000",
    "amount": "0.00876952",
    "amount_unfilled": "0.00876952",
    "amount_filled": null,
    "amount_cancelled": null,
    "value_filled": null,
    "price_avg": null,
    "done_at": null,
    "state": "submitted"
  }
]
```

Each object in array has next structure:

- `id` - integer; order id
- `date` - integer; milliseconds timestamp when order created
- `type` - string; `buy-limit`, `sell-limit`
- `symbol` - string; identifier of pair
- `price` - string; limit price
- `amount` - string; amount of order when it is created
- `amount_unfilled` - string; current amount
- `amount_filled` - string; amount that matched already
- `amount_cancelled` - string; when order canceled this equals to unfilled amount at the moment of cancellation
- `value_filled` - string; amount of the second currency that filled
- `price_avg` - string; average price of filled amount
- `done_at` - string; milliseconds timestamp when order was filly filled or cancelled
- `state` - string; `submitted`, `partial-filled`, `filled`, `partial-canceled`, `canceled`

## Public Trades

Returns list of trades filtered by parameters

### Request

`GET /api/v2/public/trades`

### Authentication

`FREE`

### Parameters

Object with next items:

- `symbol` - string; identifier of pair
- `type` - string; `sell` or `buy`
- `since` - integer; timestamp, minimal `date` of record
- `until` - integer; timestamp, maximal `date` of record
- `since_id` - integer; minimal `id` of record
- `until_id` - integer; maximal `id` of record
- `limit` - integer; max number of results, by default is 100, max value is 1000

### Response

> Response Example

```json
[
  {
    "date": 1543252041327,
    "id": 17802900,
    "symbol": "BTC_USD",
    "price": "6609.00000000",
    "amount": "0.00075600",
    "type": "buy"
  },
  {
    "date": 1541591882904,
    "id": 17802242,
    "symbol": "BTC_USD",
    "price": "6610.00000000",
    "amount": "0.00100000",
    "type": "buy"
  },
  {
    "date": 1541591750140,
    "id": 17802190,
    "symbol": "BTC_USD",
    "price": "6636.71000000",
    "amount": "0.00002335",
    "type": "buy"
  }
]
```

Each object in array has next structure:

- `date` - number; timestamp, date of trade
- `id` - integer;
- `symbol` - string; identifier of pair
- `price` - decimal string;
- `amount` - decimal string;
- `type` - integer; `sell` or `buy`

# Private methods

Here all method that requires authentication:

- [Create Order](#create-order)
- [Cancel Order](#cancel-order)
- [Private Orders](#private-orders)
- [Private Trades](#private-trades)
- [Balance](#balance)
- [Trading Fees](#trading-fees)
- [Create coupon](#create-coupon)
- [Redeem coupon](#redeem-coupon)

## Create Order

Create order on platform

_Currently only limit order. If new order types will be added limit order will be by default_

### Request

`POST /api/v2/private/order/create`

### Authentication

`FULL`, `CAN_CREATE_ORDER`

### Parameters

Object with next items:

- `symbol` - string; trading pair, market for order
- `type` - string; `sell-limit` or `buy-limit`
- `price` - decimal string; desired price for order
- `amount` - decimal string; size of order measured in first currency of trading pair

### Response

> Response Example

```json
{
  "order": {
    "id": 33852806,
    "date": 1544617382079,
    "symbol": "BTC_USD",
    "type": "sell-limit",
    "price": "1000.00000000",
    "amount": "0.00150000",
    "amount_unfilled": "0.00150000",
    "amount_filled": "0.00000000",
    "amount_cancelled": null,
    "value_filled": "0.00000000",
    "fee_filled": "0.00000000",
    "price_avg": null,
    "done_at": null,
    "state": "submitted"
  },
  "trades": [],
  "funds": [
    {
      "symbol": "USD",
      "available": "5.15245011",
      "reserved": "1.49999876"
    }
  ]
}
```

Object with order data and funds

- `order` - object with next structure:
  - `id` - number; unique identifier of order
  - `date` - integer; timestamp when operation were performed
  - `symbol` - string; market of the order, equivalent to trading pair
  - `type` - string; `sell-limit` or `buy-limit`
  - `price` - decimal string; price with what order were created
  - `amount` - decimal string; amount with what order were created
  - `amount_unfilled` - decimal string; current amount
  - `amount_filled` - decimal string; amount that already matched
  - `amount_cancelled` - decimal string; when order canceled this equals to unfilled amount at the moment of cancellation
  - `value_filled` - decimal string; amount of the second currency that filled
  - `fee_filled` - decimal string; fee amount that already matched
  - `price_avg` - decimal string; average price of filled amount
  - `done_at` - integer; date when order was closed
  - `state` - string; `submitted`, `partial-filled`, `filled`, `partial-canceled`, `canceled`
- `trades` - array; if order were matched with side orders on placing there are some trades will be created;
  - `id` - number; unique identifier of trade
  - `price` - decimal string; price of trade
  - `amount` - decimal string; value of trade
  - `side_order` - integer; identifier of side order that was matched to current order
- `funds` - array; list of wallets that were changed by operation
  - `symbol` - string; symbol of currency
  - `available` - decimal string; available funds
  - `reserved` - decimal string; reserved funds

## Cancel Order

Cancel order on platform

### Request

`POST /api/v2/private/order/<id>/cancel`

### Authentication

`FULL`, `CAN_CANCEL_ORDER`

### Path parameters

Object with next items:

- `id` - number; id of order that must be canceled

### Response

> Response Example

```json
{
  "cancel": {
    "id": 458528,
    "date": 1544617682079
  },
  "order": {
    "id": 33852806,
    "date": 1544617382079,
    "symbol": "BTC_USD",
    "type": "sell-limit",
    "price": "1000.00000000",
    "amount": "0.00150000",
    "amount_unfilled": "0.00000000",
    "amount_filled": "0.00000000",
    "amount_cancelled": "0.00150000",
    "value_filled": "0.00000000",
    "fee_filled": "0.00000000",
    "price_avg": null,
    "done_at": 1544617682079,
    "state": "canceled"
  },
  "funds": [
    {
      "symbol": "USD",
      "available": "6.65244888",
      "reserved": "0.00000000"
    }
  ]
}
```

Object with next structure

- `cancel` - object;
  - `id` - number; identifier of operation
  - `date` - number; date of operation
- `order` - object;
  - `id` - number; unique identifier of order
  - `date` - integer; timestamp when operation were performed
  - `symbol` - string; market of the order, equivalent to trading pair
  - `type` - string; `sell-limit` or `buy-limit`
  - `price` - decimal string; price with what order were created
  - `amount` - decimal string; amount with what order were created
  - `amount_unfilled` - decimal string; current amount
  - `amount_filled` - decimal string; amount that already matched
  - `amount_cancelled` - decimal string; when order canceled this equals to unfilled amount at the moment of cancellation
  - `value_filled` - decimal string; amount of the second currency that filled
  - `fee_filled` - decimal string; fee amount that already matched
  - `price_avg` - decimal string; average price of filled amount
  - `done_at` - integer; date when order was closed
  - `state` - string; `submitted`, `partial-filled`, `filled`, `partial-canceled`, `canceled`
- `funds` - array; list of wallets that were changed by operation
  - `symbol` - string; symbol of currency
  - `available` - decimal string; available funds
  - `reserved` - decimal string; reserved funds

## Private Orders

Returns list of my orders filtered by parameters

### Request

`GET /api/v2/private/orders`

### Authentication

`FULL`

### Parameters

- `symbol` - string; any symbol id
- `type` - string; buy-limit, sell-limit
- `status` - string; active, canceled, done
- `since` - integer; timestamp, minimal `date` of record
- `until` - integer; timestamp, maximal `date` of record
- `since_id` - integer; minimal `id` of record
- `until_id` - integer; maximal `id` of record
- `limit` - integer; max number of results, by default is 100, max value is 1000

### Response

> Response Example

```json
[
  {
    "id": 34,
    "date": 1478036172902,
    "type": "buy-limit",
    "symbol": "BTC_USD",
    "price": "433.00000000",
    "amount": "0.00692841",
    "amount_unfilled": "0.00692841",
    "amount_filled": null,
    "amount_cancelled": null,
    "value_filled": null,
    "fee_filled": null,
    "price_avg": null,
    "done_at": null,
    "state": "submitted"
  },
  {
    "id": 28,
    "date": 1478033710751,
    "type": "buy-limit",
    "symbol": "BTC_USD",
    "price": "55.00000000",
    "amount": "0.00876952",
    "amount_unfilled": "0.00876952",
    "amount_filled": null,
    "amount_cancelled": null,
    "value_filled": null,
    "fee_filled": null,
    "price_avg": null,
    "done_at": null,
    "state": "submitted"
  }
]
```

Each object in array has next structure:

- `id` - integer; order id
- `date` - integer; milliseconds timestamp when order created
- `type` - string; `buy-limit`, `sell-limit`
- `price` - string; limit price
- `amount` - string; amount of order when it is created
- `amount_unfilled` - string; current amount
- `amount_filled` - string; amount that matched already
- `amount_cancelled` - string; when order canceled this equals to unfilled amount at the moment of cancellation
- `value_filled` - string; amount of the second currency that filled
- `fee_filled` - string; amount of fee pair for filled amount or value
- `price_avg` - string; average price of filled amount
- `done_at` - string; milliseconds timestamp when order was filly filled or cancelled
- `state` - string; `submitted`, `partial-filled`, `filled`, `partial-canceled`, `canceled`

## Private Trades

Returns list of my trades filtered by parameters

### Request

`GET /api/v2/private/trades`

### Authentication

`FULL`

### Parameters

Object with next items:

- `symbol` - string; identifier of pair
- `type` - string; `sell` or `buy`
- `since` - integer; timestamp, minimal `date` of record
- `until` - integer; timestamp, maximal `date` of record
- `since_id` - integer; minimal `id` of record
- `until_id` - integer; maximal `id` of record
- `limit` - integer; max number of results, by default is 100, max value is 1000

### Response

> Response Example

```json
[
  {
    "id": 17802902,
    "date": 1544617013258,
    "type": "buy",
    "symbol": "BTC_USD",
    "price": "6585.00000000",
    "amount": "1",
    "fee_amount": "0.01316842",
    "fee_rate": "0.00200000",
    "fee_symbol": "USD"
  }
]
```

Each object in array has next structure:

- `id` - integer; identifier of trade
- `date` - float; timestamp, date of trade
- `type` - string; `sell` or `buy`
- `symbol` - string; identifier of pair
- `price` - decimal string; trade price
- `amount` - decimal string; trade amount
- `fee_amount` - decimal string; fee value
- `fee_rate` - decimal string; fee rate (rate `0.002` mean `2%`)
- `fee_symbol` - string; symbol of fee currency

## Balance

Returns fund for each symbol available on account

### Request

`GET /api/v2/private/balance`

### Authentication

`FULL`

### Parameters

Object with next items:

- `symbols` - string; identifier of pair or multiple pair identifiers, separated by comma, example - `BTC_USD,BTC_USDT`. Max number of pairs is 5.

### Response

> Response Example

```json
[
  {
    "symbol": "BTC",
    "available": "0.00661226",
    "reserved": "0.00000000"
  },
  {
    "symbol": "USD",
    "available": "6.65245012",
    "reserved": "0.00000124"
  }
]
```

Each object in array has next structure:

- `symbol` - string; identifier of currency
- `available` - decimal string; available funds
- `reserved` - decimal string; reserved funds

Object; keys is currencies id of wallets that were changed by operation, values is available balance after operation

## Trading Fees

Returns account trading fees

### Request

`GET /api/v2/private/trading-fees`

### Authentication

`FULL`

### Response

> Response Example

```json
{
  "sell_fee_rate": "0.00200000",
  "buy_fee_rate": "0.00200000",
  "pair_fee_rates": [
    {
      "symbol": "BTC_USD",
      "sell_fee_rate": "0.00000000",
      "buy_fee_rate": "0.00000000"
    }
  ]
}
```

Object has next structure:

- `taker_fee_rate` - decimal string; fee rate for sell orders
- `maker_fee_rate` - decimal string; fee rate for buy orders
- `pair_fee_rates` - array; list of fee rates by pair (it overrides `taker_fee_rate`, `maker_fee_rate`)
  - `symbol` - string; identifier of currency
  - `maker_fee_rate` - decimal string; fee rate for sell orders
  - `taker_fee_rate` - decimal string; fee rate for buy orders

## Create coupon

Create Coupon

Read more about **Coupon**s ...

### Request

`POST /api/v2/private/coupon/create`

### Authentication

`FULL`, `CAN_CREATE_COUPON`

### Parameters

Object with next items:

- `symbol` - string; id of currency asset of Coupon
- `amount` - number; value of Coupon
- `recipient_email` - string; email of recipient, can be skipped of not need. but if set - only user with this email or owner can redeem it

### Response

> Response Example

```json
{
  "code": "BTClYyR9gLNrZCN35KY4DFxc64roF1RJKsxxgEuMyRdnmkp7qmrw5XGVjQrk2UZhccFD",
  "coupon": {
    "date": 1544695102296,
    "opid": 48480429,
    "amount": "0.05000000",
    "symbol": "BTC",
    "recipient_email": null
  },
  "funds": [
    {
      "symbol": "BTC",
      "available": "0.12060000",
      "reserved": "0.00000000"
    }
  ]
}
```

<aside class="notice">
If request was success
</aside>

Object has next structure:

- `code` - string; code of created coupon. it is your Coupon, save it and keep in safe
- `coupon` - object;
  - `date` - number; timestamp when operation was performed
  - `opid` - integer; internal id of operation
  - `amount` - decimal string; value of coupon
  - `symbol` - string; currency symbol of coupon
  - `recipient_email` - string; recipient on Coupon, if was requested
- `funds` - array; list of wallets that were changed by operation
  - `symbol` - string; symbol of currency
  - `available` - decimal string; available funds
  - `reserved` - decimal string; reserved funds

## Redeem coupon

Redeem Coupon

### Request

`POST /api/v2/private/coupon/redeem`

### Authentication

`FULL`, `CAN_REDEEM_COUPON`

### Parameters

Object with next items:

- `code` - string; Coupon that you have

### Response

> Response Example

```json
{
  "redeem": {
    "date": 1544696109.65948,
    "amount": "0.05000000",
    "symbol": "BTC",
  }
  "funds": [
    {
      "symbol": "BTC",
      "available": "0.12560000",
      "reserved": "0.00000000"
    }
  ]
}
```

<aside class="notice">
If request was success
</aside>

Object has next structure:

- `redeem` - object;
  - `date` - number; timestamp when operation was performed
  - `amount` - decimal string; value that was added to your funds
  - `symbol` - string; currency symbol of wallet that were promoted
- `funds` - array; list of wallets that were changed by operation
  - `symbol` - string; symbol of currency
  - `available` - decimal string; available funds
  - `reserved` - decimal string; reserved funds
