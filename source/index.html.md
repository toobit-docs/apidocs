---
title:   Spot API Document 

language_tabs: # must be one of https://git.io/vQNgJ
  - shell

toc_footers:
  - <a href='https://www.toobit.com'>TooBit</a>
includes:

search: true
---

# Change Log

## 2022-10-16
  Create a document

# Introduction

## API Key Setup

- Some endpoints will require an API Key. Please refer to<a href='https://toobit.zendesk.com/hc/en-001/articles/13445077851545-How-to-Create-Your-API-Key'> this page </a> regarding API key creation.
- Once API key is created, it is recommended to set IP restrictions on the key for security reasons.
- **Never share your API key/secret key to ANYONE.**

<aside class="warning">
 If the API keys were accidentally shared, please delete them immediately and create a new key.
</aside>

## API Key Restrictions

- After creating the API key, the default restrictions is `Enable Reading`.

## Enabling Accounts

### Spot Account

A `SPOT` account is provided by default upon creation of a Account.

# General Info

## General API Information

- ome endpoints will require an API Key. Please refer to<a href='https://toobit.zendesk.com/hc/en-001/articles/13445077851545-How-to-Create-Your-API-Key'>this page</a>
- The base endpoint is: **https://api.toobit.com**
- All endpoints return either a JSON object or array.
- All time and timestamp related fields are in milliseconds.

### HTTP  Return Codes

- HTTP `4XX` return codes are used for malformed requests; the issue is on the sender's side.
- HTTP `403` return code is used when the WAF Limit (Web Application Firewall) has been violated.
- HTTP `429` return code is used when breaking a request rate limit.
- HTTP `5XX` return codes are used for internal errors; the issue is on TooBit's side. It is important to NOT treat this as a failure operation; the execution status is UNKNOWN and could have been a success.

### General Information on Endpoints

- For `GET `endpoints, parameters must be sent as a `query string`.
- For `POS`T, `PUT`, and `DELETE` endpoints, the parameters may be sent as a query string or in the request body with content type `application/x-www-form-urlencoded`. You may mix parameters between both the `query string` and `request body` if you wish to do so.
- Parameters may be sent in any order.
- If a parameter sent in both the `query string` and `request body`, the `query string` parameter will be used.

## LIMITS

### General Info on Limits

- The `/api/v1/exchangeInfo` `rateLimits` array contains objects related to the exchange's `REQUEST_WEIGHT` and `ORDERS` rate limits. These are further defined in the `ENUM definitions` section under `Rate limiters (rateLimitType)`.
- A 429 will be returned when either rate limit is violated.
- Each route has a weight which determines for the number of requests each endpoint counts for. Heavier endpoints and endpoints that do operations on multiple symbols will have a heavier weight.
- When a 429 is received, it's your obligation as an API to back off and not spam the API.

<aside class="notice">
 We recommend using the websocket for getting data as much as possible, as this will not count to the request rate limit.
</aside>

### Order Rate Limits
- When the order count exceeds the limit, you will receive a 429 error without the Retry-After header. Please check the Order Rate Limit rules using `GET` `api/v1/exchangeInfo` and wait for reactivation accordingly. 
- **The order rate limit is counted against each account.**

### Websocket Limits

- WebSocket connections have a limit of 5 incoming messages per second. A message is considered:
  - A PING frame
  - A PONG frame
  - A JSON controlled message (e.g. subscribe, unsubscribe)
- If the user sends more messages than the limit, the connection will be disconnected. IPs that are repeatedly disconnected may be blocked by the server.

## Endpoint security type

- Each endpoint has a security type that determines how you will interact with it. This is stated next to the NAME of the endpoint.
  - If no security type is stated, assume the security type is NONE.
- API-keys are passed into the Rest API via the `X-BB-APIKEY` header.
- API-keys and secret-keys **are case sensitive**.
- API-keys can be configured to only access certain types of secure endpoints. For example, one API-key could be used for TRADE only, while another API-key can access everything except for TRADE routes.
- By default, API-keys can access all secure routes.

| Security Type		    | Description | 
| ----------- | ---- | 
| NONE | Endpoint can be accessed freely. |
| TRADE | Endpoint requires sending a valid API-Key and signature. |
| USER_DATA | Endpoint requires sending a valid API-Key and signature. |
| USER_STREAM | Endpoint requires sending a valid API-Key. |
| MARKET_DATA | Endpoint requires sending a valid API-Key. |

- `TRADE` and `USER_DATA` endpoints are `SIGNED` endpoints.

## SIGNED (TRADE, USER_DATA) Endpoint security
- `SIGNED` endpoints require an additional parameter, `signature`, to be sent in the `query string` or `request body`.
- Endpoints use `HMAC SHA256` signatures. The `HMAC SHA256` signature is a keyed `HMAC SHA256 `operation. Use your `secretKey` as the key and `totalParams` as the value for the HMAC operation.
- The signature is not case sensitive.
- totalParams is defined as the query string concatenated with the request body.

### Timing security
- A `SIGNED `endpoint also requires a parameter, `timestamp`, to be sent which should be the millisecond timestamp of when the request was created and sent.
- An additional parameter, `recvWindow`, may be sent to specify the number of milliseconds after `timestamp `the request is valid for. If `recvWindow` is not sent, **it defaults to 5000**.

> The logic is as follows:

``` javascript
if (timestamp < (serverTime + 1000) && (serverTime - timestamp) <= recvWindow) {
  // process request
} else {
  // reject request
}
```

**Serious trading is about timing.**  Networks can be unstable and unreliable, which can lead to requests taking varying amounts of time to reach the servers. With `recvWindow,` you can specify that the request must be processed within a certain number of milliseconds or be rejected by the server.
<aside class="notice">
It is recommended to use a small recvWindow of 5000 or less! The max cannot go beyond 60,000!
</aside>

### SIGNED Endpoint Examples for POST /api/v1/spot/order - HMAC Keys
Here is a step-by-step example of how to send a vaild signed payload from the Linux command line using `echo`, `openssl`, and `curl`.

| Key    | Value | 
| --------- | ---- | 
| apiKey| SRQGN9M8Sr87nbfKsaSxm33Y6CmGVtUu9Erz73g9vHFNn36VROOKSaWBQ8OSOtSq |
| secretKey| 30lfjDT51iOG1kYZnDoLNynOyMdIcmQyO1XYfxzYOmQfx9tjiI98Pzio4uhZ0Uk2 |

| Parameter    | Value | 
| --------- | ---- | 
| symbol | BTCUSDT |
| side | SELL |
| type| LIMIT |
| timeInForce| GTC |
| quantity| 1 | 
| price| 4000 |
| recvWindow| 100000 |
| timestamp| 1668481902307|

#### Example 1:  As a query string

> Example 1:

> HMAC SHA256 signature:

``` shell
$ echo -n "symbol=BTCUSDT&side=SELL&type=LIMIT&timeInForce=GTC&quantity=1&price=400&recvWindow=100000&timestamp=1668481902307" | openssl dgst -sha256 -hmac "30lfjDT51iOG1kYZnDoLNynOyMdIcmQyO1XYfxzYOmQfx9tjiI98Pzio4uhZ0Uk2"
(stdin)= 8420e499e71cce4a00946db16543198b6bcae01791bdb75a06b5a7098b156468
```

``` bash
curl command:
(HMAC SHA256)
$ curl -H "X-BB-APIKEY: SRQGN9M8Sr87nbfKsaSxm33Y6CmGVtUu9Erz73g9vHFNn36VROOKSaWBQ8OSOtSq" -X POST 'https://api.toobit.com/api/v1/spot/order' -d 'symbol=BTCUSDT&side=SELL&type=LIMIT&timeInForce=GTC&quantity=1&price=400&recvWindow=100000&timestamp=1668481902307&signature=8420e499e71cce4a00946db16543198b6bcae01791bdb75a06b5a7098b156468'

```
- **queryString**
  symbol=BTCUSDT <br>
  &side=SELL <br>
  &type=LIMIT <br>
  &timeInForce=GTC <br>
  &quantity=1 <br>
  &price=400 <br>
  &recvWindow=100000 <br>
  &timestamp=1668481902307



#### Example 2: As a request body

``` shell
Example 2:
HMAC SHA256 signature:
$ echo -n "symbol=BTCUSDT&side=SELL&type=LIMIT&timeInForce=GTC&quantity=1&price=400&recvWindow=100000&timestamp=1668481902307" | openssl dgst -sha256 -hmac "30lfjDT51iOG1kYZnDoLNynOyMdIcmQyO1XYfxzYOmQfx9tjiI98Pzio4uhZ0Uk2"
(stdin)= 8420e499e71cce4a00946db16543198b6bcae01791bdb75a06b5a7098b156468
```

``` bash
curl command:
(HMAC SHA256)
$ curl -H "X-BB-APIKEY: SRQGN9M8Sr87nbfKsaSxm33Y6CmGVtUu9Erz73g9vHFNn36VROOKSaWBQ8OSOtSq" -X POST 'https://api.toobit.com/api/v1/spot/order' -d 'symbol=BTCUSDT&side=SELL&type=LIMIT&timeInForce=GTC&quantity=1&price=400&recvWindow=100000&timestamp=1668481902307&signature=8420e499e71cce4a00946db16543198b6bcae01791bdb75a06b5a7098b156468'
```
- **requestBody**
  symbol=BTCUSDT <br>
  &side=SELL <br>
  &type=LIMIT <br>
  &timeInForce=GTC <br>
  &quantity=1 <br>
  &price=400 <br>
  &recvWindow=100000 <br>
  &timestamp=1668481902307



#### Example 3: Mixed query string and request body

- **queryString**:symbol=BTCUSDT&side=SELL&type=LIMIT&timeInForce=GTC
- **requestBody**:quantity=1&price=400&recvWindow=100000&timestamp=1668481902307

``` shell
Example 3:
HMAC SHA256 signature:
$ echo -n "symbol=BTCUSDT&side=SELL&type=LIMIT&timeInForce=GTCquantity=1&price=400&recvWindow=10000000&timestamp=1668481902307" | openssl dgst -sha256 -hmac "30lfjDT51iOG1kYZnDoLNynOyMdIcmQyO1XYfxzYOmQfx9tjiI98Pzio4uhZ0Uk2"
(stdin)= 59ef0b2085ebb99cca5b6445c202d99add17be2d5d1861c0f4aa17bc785ac4d5
```

``` bash
curl command:
(HMAC SHA256)
$ curl -H "X-BB-APIKEY: SRQGN9M8Sr87nbfKsaSxm33Y6CmGVtUu9Erz73g9vHFNn36VROOKSaWBQ8OSOtSq" -X POST 'https://api.toobit.com/api/v1/spot/order?symbol=BTCUSDT&side=SELL&type=LIMIT&timeInForce=GTC' -d 'quantity=1&price=400&recvWindow=10000000&timestamp=1668481902307&signature=59ef0b2085ebb99cca5b6445c202d99add17be2d5d1861c0f4aa17bc785ac4d5'

```

Note that the signature is different in example 3. There is no & between "GTC" and "quantity=1".

## Public API Definitions

### Terminology

These terms will be used throughout the documentation, so it is recommended especially for new users to read to help their understanding of the API.

- `base asset`  refers to the asset that is the `quantity` of a symbol. For the symbol BTCUSDT, BTC would be the base asset.
- `quote asset` refers to the asset that is the `price` of a symbol. For the symbol BTCUSDT, USDT would be the quote asset.

### ENUM definitions

#### Order status (status):
- NEW - New order, no deal yet
- PARTIALLY_FILLED - Partial Sale
- FILLED - Full Deal
- CANCELED - Cancelled
- PENDING_CANCEL - Waiting for Cancellation
- REJECTED - Rejected

#### Order types (orderTypes, type):
- LIMIT - Limit Order
- MARKET -  Market List
- LIMIT_MAKER - maker Limit Order
- STOP_LOSS (unavailable now) 
- STOP_LOSS_LIMIT (unavailable now) 
- TAKE_PROFIT (unavailable now) 
- TAKE_PROFIT_LIMIT (unavailable now) 
- MARKET_OF_PAYOUT (unavailable now) 

#### Order side (side):
- BUY -  buy the bill
- SELL - sell order

#### Time in force (timeInForce):
- GTC - Good Till Cancel 
- IOC - Immediate or Cancel 
- FOK - Fill or Kill 

#### Kline/Candlestick chart intervals:
m -> minutes; h -> hours; d -> days; w -> weeks; M -> months

- 1m
- 3m
- 5m
- 15m
- 30m
- 1h
- 2h
- 4h
- 6h
- 8h
- 12h
- 1d
- 1w
- 1M

#### Rate limiters (rateLimitType)
- REQUESTS_WEIGHT
- ORDERS

#### Rate limit intervals (interval)
- SECOND
- MINUTE
- DAY


# Wallet Endpoints

## Withdraw (USER_DATA)
- `POST /api/v1/account/withdraw`

Submit a withdraw request.

### Weight：1

> Response

``` json
{
"status": 0,
"success": true,
"needBrokerAudit": false, // Do you need a brokerage review?
"id": "423885103582776064", // Withdrawal successful order id
"refuseReason":"" // failure rejection reason
}
```

### Parameters

| Name     | Type      | Mandatory      | Description           |
| ----------- | ------- | ------------- | -------------- |
| coin | STRING | YES | asset |
| clientOrderId | LONG | YES | 	client id for withdraw |
| address | STRING  | YES | Withdrawal address (Note: the withdrawal address must be maintained in the PC terminal or APP terminal in the common address list inside the address) |
| addressExt | STRING | NO | tag |
| quantity  | DECIMAL | YES | Number of coin withdrawals |
| chainType | STRING | NO | chain type, The chainType of USDT is OMNI ERC20 TRC20 respectively, and the default is OMNI |


##  Withdrawal records (USER_DATA)
- `GET /api/v1/account/withdrawOrders  (HMAC SHA256)`

### Weight：5

> Response

``` json
[
    {
        "time":"1536232111669",
        "id ":"90161227158286336",
        "accountId":"517256161325920",
        "coinId ":"BHC",
        "coinName":"BHC",
        "address":"0x815bF1c3cc0f49b8FC66B21A7e48fCb476051209",
        "addressExt":"address tag",
        "quantity":"14", // Withdrawal amount
        "arriveQuantity":"14", // Amount received
        "statusCode":"PROCESSING_STATUS",
        "status":3,
        "txId ":"",
        "txIdUrl ":"",
        "walletHandleTime":"1536232111669",
        "feeCoinId ":"BHC",
        "feeCoinName ":"BHC",
        "fee":"0.1",
        "requiredConfirmTimes ":0, // Number of confirmation requests
        "confirmTimes ":0, // number of confirmations
        "kernelId":"", // Exclusive to BEAM and GRIN
        "isInternalTransfer": false // Whether internal transfer
    }
]
```

### Parameters

| Name     | Type      | Mandatory      | Description           |
| ----------- | ------- | ------------- | -------------- |
| coin | STRING | NO | asset |
| startTime | LONG | NO | start timestamp |
| endTime | LONG | NO | end timestamp |
| fromId | LONG | NO | From which OrderId to start fetching |
| withdrawOrderId | LONG | NO | Withdrawal order ID |
| limit | INT | NO | default 500; maximum 1000 |
| recvWindow | LONG | NO | recv window |
| timestamp | LONG | YES | Timestamp |

## Deposit Address (USER_DATA)
- `GET /api/v1/account/deposit/address  (HMAC SHA256)`

Fetch deposit address with network.

### Weight：1

> Response

``` json
    {
        "canDeposit":false,//Is it possible to recharge
        "address":"0x815bF1c3cc0f49b8FC66B21A7e48fCb476051209",
        "addressExt":"address tag",
        "minQuantity":"100",//minimum amount
        "requiredConfirmTimes ":1,//Arrival confirmation number
        "canWithdrawConfirmNum ":12,//Withdrawal confirmation number
        "coinType":"ERC20_TOKEN"
    }
```

### Parameters
| Name     | Type      | Mandatory      | Description                                                                                 |
| ----------- | ------- | ------------- |---------------------------------------------------------------------------------------------|
| coin | STRING | YES | asset                                                                                       |
| chainType | STRING | YES | chain type, The chainType of USDT is OMNI ERC20 TRC20 respectively, and the default is OMNI |

## Deposit History (USER_DATA)
 - `GET /api/v1/account/depositOrders  (HMAC SHA256)`

### Weight： 5

> Response

``` json
[
  {
        "id": 100234,
        "coin": "EOS",
        "coinName": "EOS",
        "address": "deposit2bb",
        "addressTag": "19012584",
        "fromAddress": "clarkkent",
        "fromAddressTag": "19029901",
        "time": 1499865549590,
        "quantity": "1.01",
        "status": "1",
        "statusCode": "1",
        "requiredConfirmTimes": "5",
        "confirmTimes": "5",
        "txId": "98A3EA560C6B3336D348B6C83F0F95ECE4F1F5919E94BD006E5BF3BF264FACFC",
        "txIdUrl": ""
  }
]
```

### Parameters

| Name     | Type      | Mandatory      | Description                     |
| ----------- | ------- | ------------- |---------------------------------|
| coin | STRING | NO | asset                           |
| startTime | LONG | NO | start timestamp                 |
| endTime | LONG | NO | end timestamp                   |
| fromId | LONG | NO | From which Id to start crawling |
| limit | INT | NO | Default 500; Max 1000           |
| recvWindow | LONG | NO |       recv window                          |
| timestamp | LONG | YES |        时间戳                         |

Notes:

- If fromId is set, it will filter the orders smaller than id. Otherwise, the most recent order information will be returned.

# Market Data Endpoints

## Test Connectivity

-  `GET /api/v1/ping`

est connectivity to the Rest API.

### Weight:1

> Response

``` json
{}
```

### Parameters

NONE

## Check Server Time

- `GET /api/v1/time`

Test connectivity to the Rest API and get the current server time.

### Weight:1

> Response

``` json
{
  "serverTime": 1538323200000
}
```

### Parameters

NONE

## Exchange Information

- `GET /api/v1/exchangeInfo`

Current exchange trading rules and symbol information

### Weight:1

> Response

``` json
{
  "timezone": "UTC",
  "serverTime": "1668407511495",
  "brokerFilters": [],
  "symbols": [
    {
      "filters": [
        {
          "minPrice": "0.01",
          "maxPrice": "100000.00000000",
          "tickSize": "0.01",
          "filterType": "PRICE_FILTER"
        },
        {
          "minQty": "0.01",
          "maxQty": "100000.00000000",
          "stepSize": "0.0001",
          "filterType": "LOT_SIZE"
        },
        {
          "minNotional": "10",
          "filterType": "MIN_NOTIONAL"
        },
        {
          "minTradeQuantity": "0", // Minimum Trading Volume (Futures)
          "maxTradeQuantity": "0", // Maximum trading volume (futures)
          "minTradeAmount": "0", // Minimum Volume (Spot)
          "maxTradeAmount": "0", // Maximum Turnover (Spot)
          "minBuyPrice": "0", // Minimum buy price (spot)
          "limitMaxSellPrice": "0", // Limit maximum selling price 
          "limitMinTradeQuantity": "0", //  Limit minimum trading volume (spot)
          "limitMaxTradeQuantity": "0", // Maximum trading volume of limit price (spot)
          "marketMinTradeQuantity": "0", // Market Minimum Trading Volume (Spot)
          "marketMaxTradeQuantity": "0", // Market Max Trade Quantity (spot)
          "limitBuyMarkPriceRate": "0", // the rate at which the limit buy cannot be higher than the mark price
          "limitSellMarkPriceRate": "0", // the rate at which the limit sell price cannot be higher than the marked price
          "limitMaxDelegateOrderQuantity": 0, // Limit the maximum number of pending orders for a delegate order
          "limitMaxConditionOrderQuantity": 0, // Limit the maximum number of pending orders for condition orders
          "marketBuyMarkPriceRate": "0", //  the market buy price cannot be higher than the "mark (futures)/latest (spot)" price ratio
          "marketSellMarkPriceRate": "0", // the rate at which the market sell price cannot be higher than the "mark(futures)/latest(spot)" price
          "noAllowMarketStartAt": "1674057600000", // market order start time is not allowed
          "noAllowMarketEndAt": "1674057600000", // do not allow to use market order end time
          "limitLimitOrderStartAt": "1674057600000", //Limit Limit order start time
          "limitLimitOrderEndAt": "1674057600000", // Limit Limit order end time
          "limitAtMinPrice": "0", // the lowest price within the limit time
          "limitAtMaxPrice": "0", // the maximum price within the time limit
          "filterType": "TRADE_RULE"
        }
      ],
      "symbol": "ETHUSDT",
      "symbolName": "ETHUSDT",
      "status": "TRADING",
      "baseAsset": "ETH",
      "baseAssetName": "ETH",
      "baseAssetPrecision": "0.0001",
      "quoteAsset": "USDT",
      "quoteAssetName": "USDT",
      "quotePrecision": "0.01",
      "icebergAllowed": false,
      "isAggregate": false,
      "allowMargin": true
    },
    {
      "filters": [
        {
          "minPrice": "0.01",
          "maxPrice": "100000.00000000",
          "tickSize": "0.01",
          "filterType": "PRICE_FILTER"
        },
        {
          "minQty": "0.0005",
          "maxQty": "100000.00000000",
          "stepSize": "0.000001",
          "filterType": "LOT_SIZE"
        },
        {
          "minNotional": "1",
          "filterType": "MIN_NOTIONAL"
        }
      ],
      "symbol": "BTCUSDT",
      "symbolName": "BTCUSDT",
      "status": "TRADING",
      "baseAsset": "BTC",
      "baseAssetName": "BTC",
      "baseAssetPrecision": "0.000001",
      "quoteAsset": "USDT",
      "quoteAssetName": "USDT",
      "quotePrecision": "0.01",
      "icebergAllowed": false,
      "isAggregate": false,
      "allowMargin": true
    },
    {
      "filters": [
        {
          "minPrice": "0.01",
          "maxPrice": "100000.00000000",
          "tickSize": "0.01",
          "filterType": "PRICE_FILTER"
        },
        {
          "minQty": "0.01",
          "maxQty": "100000.00000000",
          "stepSize": "0.01",
          "filterType": "LOT_SIZE"
        },
        {
          "minNotional": "0.01",
          "filterType": "MIN_NOTIONAL"
        }
      ],
      "symbol": "XRPUSDT",
      "symbolName": "XRPUSDT",
      "status": "TRADING",
      "baseAsset": "XRP",
      "baseAssetName": "XRP",
      "baseAssetPrecision": "0.01",
      "quoteAsset": "USDT",
      "quoteAssetName": "USDT",
      "quotePrecision": "0.01",
      "icebergAllowed": false,
      "isAggregate": false,
      "allowMargin": false
    }
  ],
  "rateLimits": [
    {
      "rateLimitType": "REQUEST_WEIGHT",
      "interval": "MINUTE",
      "intervalUnit": 1,
      "limit": 3000
    },
    {
      "rateLimitType": "ORDERS",
      "interval": "SECOND",
      "intervalUnit": 60,
      "limit": 60
    }
  ],
  "options": [],
  "contracts": [
    {
      "filters": [
        {
          "minPrice": "0.1",
          "maxPrice": "100000.00000000",
          "tickSize": "0.1",
          "filterType": "PRICE_FILTER"
        },
        {
          "minQty": "1",
          "maxQty": "100000.00000000",
          "stepSize": "1",
          "filterType": "LOT_SIZE"
        },
        {
          "minNotional": "0.000000001",
          "filterType": "MIN_NOTIONAL"
        },
        {
          "minTradeQuantity": "0", // Minimum Trading Volume (Futures)
          "maxTradeQuantity": "0", // Maximum trading volume (futures)
          "minTradeAmount": "0", // Minimum Volume (Spot)
          "maxTradeAmount": "0", // Maximum Turnover (Spot)
          "minBuyPrice": "0", // Minimum buy price (spot)
          "limitMaxSellPrice": "0", // Limit maximum selling price 
          "limitMinTradeQuantity": "0", //  Limit minimum trading volume (spot)
          "limitMaxTradeQuantity": "0", // Maximum trading volume of limit price (spot)
          "marketMinTradeQuantity": "0", // Market Minimum Trading Volume (Spot)
          "marketMaxTradeQuantity": "0", // Market Max Trade Quantity (spot)
          "limitBuyMarkPriceRate": "0", // the rate at which the limit buy cannot be higher than the mark price
          "limitSellMarkPriceRate": "0", // the rate at which the limit sell price cannot be higher than the marked price
          "limitMaxDelegateOrderQuantity": 0, // Limit the maximum number of pending orders for a delegate order
          "limitMaxConditionOrderQuantity": 0, // Limit the maximum number of pending orders for condition orders
          "marketBuyMarkPriceRate": "0", //  the market buy price cannot be higher than the "mark (futures)/latest (spot)" price ratio
          "marketSellMarkPriceRate": "0", // the rate at which the market sell price cannot be higher than the "mark(futures)/latest(spot)" price
          "noAllowMarketStartAt": "1674057600000", // market order start time is not allowed
          "noAllowMarketEndAt": "1674057600000", // do not allow to use market order end time
          "limitLimitOrderStartAt": "1674057600000", //Limit Limit order start time
          "limitLimitOrderEndAt": "1674057600000", // Limit Limit order end time
          "limitAtMinPrice": "0", // the lowest price within the limit time
          "limitAtMaxPrice": "0", // the maximum price within the time limit
          "filterType": "TRADE_RULE"
        }
      ],
      "symbol": "BTC-SWAP-USDT",
      "symbolName": "BTC-SWAP-USDTUSDT",
      "status": "TRADING",
      "baseAsset": "BTC-SWAP-USDT",
      "baseAssetPrecision": "1",
      "quoteAsset": "USDT",
      "quoteAssetPrecision": "0.1",
      "icebergAllowed": false,
      "inverse": false,
      "index": "BTCUSDT",
      "marginToken": "USDT",
      "marginPrecision": "0.0001",
      "contractMultiplier": "0.0001",
      "underlying": "BTC",
      "riskLimits": [
        {
          "riskLimitId": "200000133",
          "quantity": "1000000.0",
          "initialMargin": "0.01",
          "maintMargin": "0.005"
        },
        {
          "riskLimitId": "200000134",
          "quantity": "2000000.0",
          "initialMargin": "0.02",
          "maintMargin": "0.01"
        },
        {
          "riskLimitId": "200000135",
          "quantity": "3000000.0",
          "initialMargin": "0.03",
          "maintMargin": "0.015"
        },
        {
          "riskLimitId": "200000136",
          "quantity": "4000000.0",
          "initialMargin": "0.04",
          "maintMargin": "0.02"
        }
      ]
    },
    {
      "filters": [
        {
          "minPrice": "0.1",
          "maxPrice": "100000.00000000",
          "tickSize": "0.1",
          "filterType": "PRICE_FILTER"
        },
        {
          "minQty": "1",
          "maxQty": "100000.00000000",
          "stepSize": "1",
          "filterType": "LOT_SIZE"
        },
        {
          "minNotional": "0.000001",
          "filterType": "MIN_NOTIONAL"
        }
      ],
      "symbol": "BTC-SWAP",
      "symbolName": "BTC-SWAP",
      "status": "TRADING",
      "baseAsset": "BTC-SWAP",
      "baseAssetPrecision": "1",
      "quoteAsset": "USDT",
      "quoteAssetPrecision": "0.1",
      "icebergAllowed": false,
      "inverse": true,
      "index": "BTCUSDT",
      "marginToken": "BTC",
      "marginPrecision": "0.00000001",
      "contractMultiplier": "1.0",
      "underlying": "BTC",
      "riskLimits": [
        {
          "riskLimitId": "200000137",
          "quantity": "1000000.0",
          "initialMargin": "0.01",
          "maintMargin": "0.005"
        },
        {
          "riskLimitId": "200000138",
          "quantity": "2000000.0",
          "initialMargin": "0.02",
          "maintMargin": "0.01"
        },
        {
          "riskLimitId": "200000139",
          "quantity": "3000000.0",
          "initialMargin": "0.03",
          "maintMargin": "0.015"
        },
        {
          "riskLimitId": "200000140",
          "quantity": "4000000.0",
          "initialMargin": "0.04",
          "maintMargin": "0.02"
        }
      ]
    }
  ],
  "coins": [
    {
      "coinId": "ETH",
      "coinName": "ETH",
      "coinFullName": "Ethereum",
      "allowWithdraw": true,
      "allowDeposit": true,
      "chainTypes": []
    },
    {
      "coinId": "USDT",
      "coinName": "USDT",
      "coinFullName": "TetherUS",
      "allowWithdraw": true,
      "allowDeposit": true,
      "chainTypes": [
        {
          "chainType": "ERC20",
          "withdrawFee": "0.1",
          "minWithdrawQuantity": "10",
          "maxWithdrawQuantity": "1000",
          "minDepositQuantity": "1",
          "allowDeposit": true,
          "allowWithdraw": true
        },
        {
          "chainType": "TRC20",
          "withdrawFee": "0.1",
          "minWithdrawQuantity": "10",
          "maxWithdrawQuantity": "1000",
          "allowDeposit": true,
          "allowWithdraw": true
        },
        {
          "chainType": "OMNI",
          "withdrawFee": "0.1",
          "minWithdrawQuantity": "10",
          "maxWithdrawQuantity": "1000",
          "allowDeposit": true,
          "allowWithdraw": true
        }
      ]
    },
    {
      "coinId": "BTC",
      "coinName": "BTC",
      "coinFullName": "Bitcoin",
      "allowWithdraw": false,
      "allowDeposit": false,
      "chainTypes": []
    },
    {
      "coinId": "UNI",
      "coinName": "UNI",
      "coinFullName": "uniswap",
      "allowWithdraw": false,
      "allowDeposit": false,
      "chainTypes": []
    },
    {
      "coinId": "XRP",
      "coinName": "XRP",
      "coinFullName": "XRP",
      "allowWithdraw": false,
      "allowDeposit": false,
      "chainTypes": []
    },
    {
      "coinId": "EOS",
      "coinName": "EOS1",
      "coinFullName": "EOS",
      "allowWithdraw": true,
      "allowDeposit": true,
      "chainTypes": []
    },
    {
      "coinId": "JET",
      "coinName": "JET",
      "coinFullName": "JET",
      "allowWithdraw": false,
      "allowDeposit": false,
      "chainTypes": []
    }
  ]
}

```

### Parameters

NONE

## Order Book
- `GET /quote/v1/depth`

### 权重：

Adjusted based on the limit:：

| Limit	     | Weight      | 
| ----------- | ------- |
| 5, 10, 20, 50, 100 | 1 | 
| 500 | 5 |
| 1000  | 10 |

> Response

``` json
{
  "b": [
    [
      "3.90000000",   // price
      "431.00000000"  // quantity
    ],
    [
      "4.00000000",
      "431.00000000"
    ]
  ],
  "a": [
    [
      "4.00000200",  // price
      "12.00000000"  // quantity
    ],
    [
      "5.10000000",
      "28.00000000"
    ]
  ]
}
```

### Parameters

| Name     | Type      | Mandatory      | Description           |
| ----------- | ------- | ------------- | -------------- |
| symbol | STRING| YES |symbol |
| limit | INT | NO | Default 100; Max 100. |

Notes:
If you set `limit=0`, a lot of data will be returned.

## Recent Trades List

- `GET /quote/v1/trades`

Get recent trades.

### Weight： 1

> Response

``` json
[
  {
    "p": "4.00000100",
    "q": "12.00000000",
    "t": 1499865549590,
    "ibm": true  // Transaction direction isBuyerMaker
  }
]
```

### Parameters

| Name     | Type      | Mandatory      | Description           |
| ----------- | ------- | ------------- | -------------- |
| symbol | STRING| YES | |
| limit | INT | NO | Default 60; Max 60. |

## Kline/Candlestick Data
- `GET /quote/v1/klines`

Kline/candlestick bars for a symbol. <br>
Klines are uniquely identified by their open time.

### Weight：1

> Response

``` json
[
  [
    1499040000000,      // Kline open time
    "0.01634790",       // Open price
    "0.80000000",       // High price
    "0.01575800",       //  Low price
    "0.01577100",       // Close price
    "148976.11427815",  // Volume
    1499644799999,      //  Kline Close time
    "2434.19055334",    // Quote asset volume
    308,                // Number of trades
    "1756.87402397",    // Taker buy base asset volume
    "28.46694368"       // Taker buy quote asset volume
  ]
]
```

### Parameters

| Name     | Type      | Mandatory      | Description           |
| ----------- | ------- | ------------- |-----------------------|
| symbol | STRING| YES | symbol                |
| interval | ENUM | YES | interval              |
| startTime | LONG | NO | start timestamp           |
| endTime | LONG | NO | end timestamp              |
| limit | INT | NO | Default 100; Max 100. |

- If `startTime` and `endTime` are not sent, only the latest K line will be returned.

## 24hr Ticker Price Change Statistics
- `GET /quote/v1/ticker/24hr`

24 hour rolling window price change statistics. Careful when accessing this with no symbol.

### Weight： 

1 if only one `symbol` was sent; 40 if no `symbol` was sent.

> Response

``` json
[
    {
        "t": 1538725500422,   // time
        "a": "1.10000000",    // highest selling price
        "b": "1.00000000",    // highest bid
        "s": "ETHBTC",        // symbol 
        "c": "4.00000200",    // latest transaction price
        "o": "99.00000000",   // opening price
        "h": "100.00000000",  // highest price 
        "l": "0.10000000",    // lowest price
        "v": "8913.30000000", // Total trade volume (in base asset)
        "qv": "15.30000000"   // otal trade volume (in quote asset)
    }
]
```

### Parameters

| Name     | Type      | Mandatory      | Description           |
| ----------- | ------- | ------------- | -------------- |
| symbol | STRING| NO |symbol |

- If the symbol is not sent, all symbol data will be returned.

## Symbol Price Ticker

- `GET /quote/v1/ticker/price`

Latest price for a symbol or symbols.

### Weight：1

> Response

``` json
[
  {
    "s": "LTCBTC",     // symbol
    "p": "4.00000200"  // price
  }
]
```
### Parameters

| Name     | Type      | Mandatory      | Description           |
| ----------- | ------- | ------------- | -------------- |
| symbol | STRING| NO |symbol |

- If the symbol is not sent, all symbol data will be returned.

## Symbol Order Book Ticker
- `GET /quote/v1/ticker/bookTicker`

Best price/qty on the order book for a symbol or symbols.

### Weight：1

> Response

``` json
[
  {
      "t": 1672035413265,     // time
      "s": "LTCBTC",            // symbol          
      "b": "4.00000000",        // bidPrice
      "bq": "431.00000000",     // bidQty
      "a": "4.00000200",        // askPrice
      "aq": "9.00000000"        // askQty
  }
]
```
### Parameters

| Name     | Type      | Mandatory      | Description           |
| ----------- | ------- | ------------- | -------------- |
| symbol | STRING| NO |symbol |

- If the symbol is not sent, the best order book price for all symbols will be returned.

## Merge Depth

- `GET /quote/v1/depth/merged`

### Weight： 1

> Response

``` json
{
    "t": 1672035413265,//time
    "b": [//Buy Depth High to Low
        [
            "16851.95",//price
            "0.003321"//quantity
        ],
        [
            "16851.87",
            "0.005456"
        ],
        [
            "16851.47",
            "0.002219"
        ]
    ],
    "a": [//Sell Depth Low to High
        [
            "16870.19",
            "0.003838"
        ],
        [
            "16873.05",
            "0.00361"
        ],
        [
            "16873.06",
            "0.002623"
        ]
    ]
}
```

### Parameters

| Name     | Type      | Mandatory      | Description           |
| ----------- | ------- | ------------- | -------------- |
| symbol | STRING| YES | symbol |
| scale | INT | NO | Gears: `0`,`1`,`2`,`3`,`4`,`5` For example: `0 `means gear `1`, `1` means gear `2` |
| limit | INT | NO | limit |

# Websocket Market Streams

- The base endpoint is: wss://stream.toobit.com
- The URL format for direct access is:  wss://#HOST/quote/ws/v1

| Name     | value      | 
| ----------- | ------- |
| topic | `realtimes`, `trade`, `kline_$interval`, `depth`,`markPrice`,`markPriceKline_$interval`,`index``indexKline_$interval` |
| event | `sub`, `cancel`, `cancel_all`  |
| interval | `1m`, `5m`, `15m`, `30m`, `1h`, `2h`, `6h`, `12h`, `1d`, `1w`, `1M` |

## Live Subscribing/Unsubscribing to streams

### Subscribe to a stream:

- Request

`{`

  `"symbol": "$symbol0, $symbol1",`

  `"topic": "$topic",`

  `"event": "sub",`

   ` "params": {`

  `"limit": "$limit", // kline return upper limit is 2000, the default is 1`
        
  ` "binary": "false" //Whether the returned data is compressed, the default is false`

  `}`

`}`

### Unsubscribe to a stream:

- Request

`{`

  `"symbol": "$symbol0, $symbol1",`

  `"topic": "$topic",`

  `"event": "cancel",`

   ` "params": {`

  `"limit": "$limit", // kline return upper limit is 2000, the default is 1`
        
  ` "binary": "false" //Whether the returned data is compressed, the default is false`

  `}`

`}`

## Heartbeat

Every once in a while, the client needs to send a ping frame, and the server will reply with a pong frame, otherwise the server will actively disconnect within 5 minutes.

> Payblad

``` json
{
    "pong": 1535975085052
}
```

### Request

`{`

  `"ping": 1535975085052`

`}`

## Trade Streams

Push the information of each transaction transaction by transaction. A deal, or the definition of a transaction, is that there is only one taker and one maker trading with each other.
After successfully connecting to the server, the server will first push a recent 60 transactions. After this push, each push is a real-time transaction.
The variable "v" can be understood as a transaction ID. This variable is globally incremented and unique. For example: Suppose there have been 3 transactions in the past 5 seconds, namely `ETHUSDT`, `BTCUSDT`, `BHTBTC`. Their "v" will be consecutive values (112, 113, 114).

> Payload

``` json
{
    "symbol": "BTCUSDT",
    "symbolName": "BTCUSDT",
    "topic": "trade",
    "params": {
        "realtimeInterval": "24h",
        "binary": "false"
    },
    "data": [
        {
            "v": "1291465821801168896", 
            "t": 1668690723096, //time
            "p": "399", // price
            "q": "1", // quantity
            "m": false // true = buy, false = sell
        },
        {
            "v": "1291465842546196481",
            "t": 1668690725569,
            "p": "399",
            "q": "1",
            "m": false
        }
    ],
    "f": true, // is it the first to return
    "sendTime": 1668753154192,
    "shared": false
}
```

### Request:

`{`

  `"symbol": "$symbol0, $symbol1",`'

 ` "topic": "trade",`

  `"event": "sub",`

 ` "params": {`

  `  "binary": false // Whether data returned is in binary format`

 ` }`

`}`

## Kline/Candlestick Streams

The Kline/Candlestick Stream push updates to the current klines/candlestick every second.

### Kline/Candlestick chart intervals:

m -> minutes; h -> hours; d -> days; w -> weeks; M -> months

- 1m
- 5m
- 15m
- 30m
- 1h
- 2h
- 4h
- 6h
- 12h
- 1d
- 1w
- 1M

> Payload

``` json
{
    "symbol": "BTCUSDT",
    "symbolName": "BTCUSDT",
    "topic": "kline",
    "params": {
        "realtimeInterval": "24h",
        "klineType": "1m",
        "binary": "false"
    },
    "data": [
        {
            "t": 1668753840000,// Event time
            "s": "BTCUSDT",// symbol
            "sn": "BTCUSDT",// symbol name
            "c": "445",//Close price
            "h": "445",//High price
            "l": "445",//Low price
            "o": "445",//Open price
            "v": "0"//Base asset volume
        }
    ],
    "f": true,// Is it the first return
    "sendTime": 1668753854576,
    "shared": false
}
```

### Request:

`{`

  `"symbol": "$symbol0, $symbol1",`

 ` "topic": "kline_"+$interval,`

  `"event": "sub",`

 ` "params": {`
    
  `  "binary": false`

  `}`

`}`

## Individual Symbol Ticker Streams

24-hour complete ticker information refreshed second by symbol

> Payload

``` json
{
    "symbol": "BTCUSDT",
    "symbolName": "BTCUSDT",
    "topic": "realtimes",
    "params": {
        "realtimeInterval": "24h",
        "binary": "false"
    },
    "data": [
        {
            "t": 1668753480049, //time
            "s": "BTCUSDT", //symbol
            "sn": "BTCUSDT", // symbol name
            "c": "445", //Close price
            "h": "445", //High price
            "l": "310", // Low price
            "o": "311", //Open price
            "v": "3747.7597191", //Total traded base asset volume
            "qv": "1426443.9553995", // Total traded quote asset volume
            "m": "0.4309", // margin
            "e": 301 // trade id
        }
    ],
    "f": true, 
    "sendTime": 1668753481048,
    "shared": false
}
```

### Request:

`{`

`  "symbol": "$symbol0, $symbol1",`

`  "topic": "realtimes",`

`  "event": "sub",`

`  "params": {`

`    "binary": false`

`  }`

`}`

## Partial Book Depth Streams

Symbol's depth information.

- Order book snapshot frequency: every 300ms, if the book changes.
- Order book snapshot frequency depth: bids and asks 300 each
- Order book version change trigger event：
  - The order enters the order book
  - Order leaves the order book
  - Order Quantity Change
  - Order Completed

> Payload

``` json
{
  "symbol": "BTCUSDT",
  "topic": "depth",
  "data": [{
    "s": "BTCUSDT", //Symbol
    "t": 1565600357643, //time
    "v": "112801745_18", // Base asset volume
    "b": [ //Bids
      ["11371.49", "0.0014"], //[price, quantity]
      ["11371.12", "0.2"],
      ["11369.97", "0.3523"],
      ["11369.96", "0.5"],
      ["11369.95", "0.0934"],
      ["11369.94", "1.6809"],
      ["11369.6", "0.0047"],
      ["11369.17", "0.3"],
      ["11369.16", "0.2"],
      ["11369.04", "1.3203"],
    "a": [//Asks
      ["11375.41", "0.0053"], //[price, quantity]
      ["11375.42", "0.0043"],
      ["11375.48", "0.0052"],
      ["11375.58", "0.0541"],
      ["11375.7", "0.0386"],
      ["11375.71", "2"],
      ["11377", "2.0691"],
      ["11377.01", "0.0167"],
      ["11377.12", "1.5"],
      ["11377.61", "0.3"]
    ]
  }],
  "f": true//Is it the first return
}
```

### Request:

``` json
{

  "symbol": "$symbol0, $symbol1",
  "topic": "depth",
  "event": "sub",
  "params": {
    "binary": false
  }
```


## Diff. Depth Stream

> Payload

``` json
{
  "symbol": "BTCUSDT",
  "topic": "diffDepth",
  "data": [{
    "e": 0,
    "t": 1565687625534,
    "v": "115277986_18",
    "b": [
      ["11316.78", "0.078"],
      ["11313.16", "0.0052"],
      ["11312.12", "0"],
      ["11309.75", "0.0067"],
      ["11309.58", "0"],
      ["11306.14", "0.0073"]
    ],
    "a": [
      ["11318.96", "0.0041"],
      ["11318.99", "0.0017"],
      ["11319.12", "0.0017"],
      ["11319.22", "0.4516"],
      ["11319.23", "0.0934"],
      ["11319.24", "3.0665"]
    ]
  }],
  "f": false //Is it the first return
}
```

### Request:

`{`

  `"symbol": "$symbol0, $symbol1",`

  `"topic": "diffDepth",`

  `"event": "sub",`

  `"params": {`

  `  "binary": false`

  `}`

`}`

Push the changing part of the order book (if any) every second.
In incremental depth information, the quantity is not necessarily equal to the quantity corresponding to the price. If the quantity=0, it means that the price in the last push is no longer available. If the quantity>0, the quantity at this time is the quantity corresponding to the updated price
Suppose there is such an item in the returned data we received:<br>

`["0.00181860", "155.92000000"]// price, quantity`

If the next returned data contains:<br>

`["0.00181860", "12.3"]`

This means that the quantity corresponding to this price has changed, and the changed quantity has been updated
If the next returned data contains:

`["0.00181860", "0"]`

This means that the quantity corresponding to this price has disappeared and will be deleted in the client.

# Spot Account/Trade

## Test New Order  (TRADE)
- `POST /api/v1/spot/orderTest (HMAC SHA256)`

Test new order creation and signature/recvWindow long. Creates and validates a new order but does not send it into the matching engine.

### Weight：1

> Response

``` json
{}
```

### Parameters
Same as `POST /api/v1/spot/order`

## New Order  (TRADE)
- `POST /api/v1/spot/order  (HMAC SHA256)`

Send in a new order.

### Weight：1

> Response

``` json
{
    "accountId": "1287091689761137921",
    "symbol": "BTCUSDT",
    "symbolName": "BTCUSDT",
    "clientOrderId": "1668483032042259",
    "orderId": "1289723583082363136",
    "transactTime": "1668483032058",
    "price": "400",
    "origQty": "1",
    "executedQty": "0",
    "status": "FILLED",
    "timeInForce": "GTC",
    "type": "LIMIT",
    "side": "SELL"
}
```

### Parameters

| Name     | Type      | Mandatory      | Description                                                                                                          |
| ----------- | ------- | ------------- |----------------------------------------------------------------------------------------------------------------------|
| symbol | STRING | YES | symbol                                                                                                               |
| assetType | ENUM | NO | `CASH`、`MARGIN`，only supports `CASH`                                                                                 |
| side | ENUM | YES | `BUY` or `SELL `                                                                                                     |
| type | ENUM | YES | See enumeration definition for details: order type                                                                   |
| timeInForce | ENUM | NO | For details, see enumeration definition: valid methods                                                               |
| quantity | DECIMAL | YES | quantity                                                                                                             |
| price | DECIMAL | NO | price                                                                                                                |
| newClientOrderId | STRING | NO | A unique id among open orders. Automatically generated if not sent.                                                  |
| stopPrice | DECIMAL | NO | Used with `STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT`, and `TAKE_PROFIT_LIMIT` orders. **currently unavailable**   |
| icebergQty | DECIMAL | NO | Used with `LIMIT`, `STOP_LOSS_LIMIT`, and ` TAKE_PROFIT_LIMIT` to create an iceberg order. **currently unavailable** |
| recvWindow | LONG | NO | recv window                                                                                                          |
| timestamp | LONG | YES |     timestamp                                                                                                                 |

Additional mandatory parameters based on `type`:

| Type     | Additional mandatory parameters  | 
| ----------- | ------- | 
| `LIMIT` | `quantity`,` price` |
| `MARKET` |  `quantity` |
| `STOP_LOSS` | `quantity`, `stopPrice` **currently unavailable** |
| `STOP_LOSS_LIMIT` | `timeInForce`, `quantity`, `price`, `stopPrice`  **currently unavailable**|
| `TAKE_PROFIT` | `quantity`, `stopPrice` **currently unavailable** |
| `TAKE_PROFIT_LIMIT` | `timeInForce`, `quantity`, `price`, `stopPrice` **currently unavailable** |
| `LIMIT_MAKER` | `quantity`, `price` |

## Place Multiple Orders (TRADE)

- `POST /api/v1/spot/batchOrders      (HMAC SHA256)`

### Weight：1

Create new orders in batches, up to `20` orders at a time, must be the same `symbol`.

> example：

``` json
curl  -H "Content-Type:application/json" -H "X-BB-APIKEY: SRQGN9M8Sr87nbfKsaSxm33Y6CmGVtUu9Erz73g9vHFNn36VROOKSaWBQ8OSOtSq" -X POST -d '[   
{     "newClientOrderId": "pl12241234567898",     
      "symbol": "BTCUSDT",     
      "side": "SELL",     
      "type": "LIMIT",     
      "price": 17001,     
      "quantity": 1   
},   
{     "newClientOrderId": "pl12241234567899",     
      "symbol": "BTCUSDT",     
      "side": "SELL",     
      "type": "LIMIT",     
      "price": 17002,     
      "quantity": 1  
 } ]' 'https://api.toobit.com/api/v1/spot/batchOrders?timestamp=1671880913657&signature=7548b6834613afed3b7d3b0b9bfb0e0b3e3799c46db3ea6b952439fde35cb88f'

```

> Response
success :
``` json
{
        "code": 0,
        "result": [{
                "code": 0,
                "order": {
                        "accountId": "1287091689761137921",
                        "symbol": "BTCUSDT",
                        "symbolName": "BTCUSDT",
                        "clientOrderId": "pl12241234567898",
                        "orderId": "202212241234567898",
                        "transactTime": "1671880251836",
                        "price": "17001",
                        "origQty": "1",
                        "executedQty": "0",
                        "status": "NEW",
                        "timeInForce": "GTC",
                        "type": "LIMIT",
                        "side": "SELL"
                }
        }, {
                "code": 0,
                "order": {
                        "accountId": "1287091689761137921",
                        "symbol": "BTCUSDT",
                        "symbolName": "BTCUSDT",
                        "clientOrderId": "pl12241234567899",
                        "orderId": "202212241234567899",
                        "transactTime": "1671880251853",
                        "price": "17002",
                        "origQty": "1",
                        "executedQty": "0",
                        "status": "NEW",
                        "timeInForce": "GTC",
                        "type": "LIMIT",
                        "side": "SELL"
                }
        }]
}
```
fail :
``` json
{
        "code": 0,
        "result": [{
                "code": -1149,
                "msg": "Create order failed"
        }, {
                "code": -1149,
                "msg": "Create order failed"
        }]
}
```

### Parameters

| Name     | Type      | Mandatory      | Description             |
| ----------- | ------- | ------------- |-------------------------|
|  |  list<JSON> | YES | `RequestBody` parameter |
| recvWindow | LONG | NO | recv window             |
| timestamp | LONG | YES |     timestamp                    |

**The batchOrders in RequestBody should fill in the order parameters in list of JSON format**

| Name     | Type      | Mandatory      | Description                                           |
| ----------- | ------- | ------------- |-------------------------------------------------------|
| symbol | STRING | YES | symbol                                                |
| side | ENUM | YES | `BUY` or `SELL `                                      |
| type | ENUM | YES | See enumeration definition for details: `orderType`   |
| timeInForce | ENUM | NO | See enumeration definition for details: `timeInForce` |
| quantity | DECIMAL | YES |       quantity                                                |
| price | DECIMAL | NO |           price                                            |
| newClientOrderId | STRING | YES | The ID of the order, defined by the user              |

Depending on the order `type`, certain parameters are mandatory:

| Type     | Additional mandatory parameters      | 
| ----------- | ------- | 
| `LIMIT` | `timeInForce`, `quantity`,` price` |
| `MARKET` |  `quantity` |
| `STOP_LOSS` | `quantity`, `stopPrice` **currently unavailable** |
| `STOP_LOSS_LIMIT` | `timeInForce`, `quantity`, `price`, `stopPrice`  **currently unavailable**|
| `TAKE_PROFIT` | `quantity`, `stopPrice` **currently unavailable** |
| `TAKE_PROFIT_LIMIT` | `timeInForce`, `quantity`, `price`, `stopPrice` **currently unavailable** |

## Cancel Order  (TRADE)
- `DELETE /api/v1/spot/order  (HMAC SHA256)`

Cancel an active order.

### Weight：1

> Response

``` json
{
  "symbol": "LTCBTC",
  "orderId": "1",
  "clientOrderId": "9t1M2K0Ya092",
  "price": "0.1",
  "origQty": "1.0",
  "executedQty": "0.0",
  "status": "CANCELED",
  "timeInForce": "GTC",
  "type": "LIMIT",
  "side": "BUY",
  "transactTime": "1499827319559"
}
```

### Parameters
| Name     | Type      | Mandatory      | Description    |
| ----------- | ------- | ------------- |----------------|
| orderId | LONG | NO | order id       |
| clientOrderId | STRING | NO | client order id |
| recvWindow | LONG | NO |   recv window             |
| timestamp | LONG | YES |    Timestamp            |

Either `orderId `or `clientOrderId `must be sent.

##  Cancel All Open Orders (TRADE)
- `DELETE /api/v1/spot/openOrders (HMAC SHA256)`

### Weight：5

> Response

``` json
{
  "success":true
}
```

### Parameters
| Name     | Type      | Mandatory      | Description     |
| ----------- | ------- | ------------- |-----------------|
| symbol | STRING | NO | symbol          |
| side | ENUM| NO | `BUY` or `SELL` |
| recvWindow | LONG | NO | recv window     |
| timestamp | LONG | YES | Timestamp       |

## Cancel Multiple Orders (TRADE)

- `DELETE /api/v1/spot/cancelOrderByIds (HMAC SHA256)`

Cancel orders in batches according to the order id, **maximum 100 items at a time**

### Weight：5

> Response

success :
``` json
{
  "code":0, // 0 On behalf of successful execution
  "result":[] //
}
```

Some or all of the cancellations failed:
``` json
{
   "code":0,
   "result":[
       {
          "orderId":"202212231234567895",
          "code":-2013 //
       },
       {
           "orderId":"202212231234567896",
           "code":-2013
       }
   ]
}
```

### Parameters
| Name     | Type      | Mandatory      | Description                          |
| ----------- | ------- | ------------- |--------------------------------------|
| ids | STRING | YES | order Id （Multiple separated by `,`） |
| recvWindow | LONG | NO | recv window                          |
| timestamp | LONG | YES |     timestamp                                 |

Note: **code** returns `0` to indicate that the order cancellation request has been executed. Whether it is successful or not depends on the results in `result`. If `result` is empty, it means that all of them are successful, and if `orderId` is not empty, it means that the cancellation failed The order id, `code` represents the reason for the cancellation failure.


## Query Order  (USER_DATA)
- `GET /api/v1/spot/order (HMAC SHA256)`

### Weight：1

> Response

``` json
{
  "symbol": "LTCBTC",
  "orderId": "1",
  "clientOrderId": "9t1M2K0Ya092",
  "price": "0.1",
  "origQty": "1.0",
  "executedQty": "0.0",
  "cumulativeQuoteQty": "0.0",
  "status": "NEW",
  "timeInForce": "GTC",
  "type": "LIMIT",
  "side": "BUY",
  "stopPrice": "0.0",
  "icebergQty": "0.0",
  "time": "1499827319559",
  "updateTime": "1499827319559",
  "isWorking": true
}
```

### Parameters
| Name     | Type      | Mandatory      | Description    |
| ----------- | ------- | ------------- |----------------|
| orderId | LONG | NO | order id       |
| origClientOrderId | STRING | NO | client order id |
| recvWindow | LONG | NO |       recv window         |
| timestamp | LONG | YES |    timestamp            |

Notes： 

- Either `orderId` or o`rigClientOrderId` must be sent.
- For some historical orders cumulativeQuoteQty will be < 0, meaning the data is not available at this time.

## Current Open Orders (USER_DATA)
- `GET /api/v1/spot/openOrders (HMAC SHA256)`

Get all open orders on a symbol. **Careful** when accessing this with no symbol.

### Weight ：1

> Response

``` json
[
  {
    "symbol": "LTCBTC",
    "orderId": "1",
    "clientOrderId": "t7921223K12",
    "price": "0.1",
    "origQty": "1.0",
    "executedQty": "0.0",
    "cumulativeQuoteQty": "0.0",
    "status": "NEW",
    "timeInForce": "GTC",
    "type": "LIMIT",
    "side": "BUY",
    "stopPrice": "0.0",
    "icebergQty": "0.0",
    "time": "1499827319559",
    "updateTime": "1499827319559",
    "isWorking": true
  }
]
```

### Parameters
| Name     | Type      | Mandatory      | Description            |
| ----------- | ------- | ------------- |------------------------|
| orderId | LONG | NO | order id               |
| symbol | STRING | NO | symbol                 |
| limit | INT | NO | Default 500; Max 1000. |
| recvWindow | LONG | NO |       recv window                  |
| timestamp | LONG | YES |       timestamp                 |

Notes：

- If orderId is set, it will filter orders smaller than orderId. Otherwise, the most recent order information will be returned.

## All Orders (USER_DATA)

- `GET /api/v1/spot/tradeOrders (HMAC SHA256)`

Get all account orders; active, canceled, or filled.

### Weight：5

> Response

``` json
[
  {
    "symbol": "LTCBTC",
    "orderId": "1",
    "clientOrderId": "987yjj2Ym",
    "price": "0.1",
    "origQty": "1.0",
    "executedQty": "0.0",
    "cumulativeQuoteQty": "0.0",
    "status": "NEW",
    "timeInForce": "GTC",
    "type": "LIMIT",
    "side": "BUY",
    "stopPrice": "0.0",
    "icebergQty": "0.0",
    "time": "1499827319559",
    "updateTime": "1499827319559",
    "isWorking": true
  }
]
```

### Parameters
| Name     | Type      | Mandatory      | Description            |
| ----------- | ------- | ------------- |------------------------|
| orderId | LONG | NO | order id               |
| symbol | STRING | NO | symbol                 |
| startTime | LONG | NO | start timestamp        |
| endTime | LONG| NO | end timestamp          |
| limit | INT | NO | Default 500; Max 1000. |
| recvWindow | LONG | NO | recv window            |
| timestamp | LONG | YES |        timestamp                |

## Account Information (USER_DATA)
- `GET /api/v1/account`

### Weight：5 

> Response

``` json
{
    "balances": [
        {
            "asset": "BTC",  // asset
            "assetId": "BTC",  // asset id
            "assetName": "BTC",  // asset name
            "total": "995.899",  // total number
            "free": "995.899", //available number
            "locked": "0" //frozen number
        }
    ]
}
```

### Parameters
| Name     | Type      | Mandatory      | Description           |
| ----------- | ------- | ------------- | -------------- |
| recvWindow | LONG | NO | recv window |
| timestamp | LONG | YES | Timestamp |

## Account Trade List (USER_DATA)
- `GET /api/v1/account/trades`

### Weight：5

> Response

``` json
[
    {
        "id": "1291291745779199489",
        "symbol": "BTCUSDT",
        "symbolName": "BTCUSDT",
        "orderId": "1290805676579237376",
        "matchOrderId": "1291291745191996928",
        "price": "314",
        "qty": "0.71433122",
        "commission": "0.22430000308",
        "commissionAsset": "USDT",
        "time": "1668669971575",
        "isBuyer": false,
        "isMaker": true,
        "fee": {
            "feeCoinId": "USDT",
            "feeCoinName": "USDT",
            "fee": "0.22430000308"
        },
        "feeCoinId": "USDT",
        "feeAmount": "0.22430000308",
        "makerRebate": "0"
    }
]
```

### Parameters
| Name     | Type      | Mandatory      | Description                        |
| ----------- | ------- | ------------- |------------------------------------|
| symbol | STRING | NO | symbol                             |
| startTime | LONG | NO | start timestamp                    |
| endTime | LONG | NO | end timestamp                      |
| fromId | LONG | NO  | from id                            |
| toId | LONG | NO | end id                             |
| limit | INT | NO | Number of items displayed per page |
| recvWindow | LONG | NO | recv window                        |
| timestamp | LONG | YES |       timestamp                             |

Notes：

- If only fromId is set，it will get orders < that fromId in descending order.
- If only toId is set, it will get orders > that toId in ascending order.
- If fromId is set and toId is set, it will get orders < that fromId and > that toId in descending order.
- If fromId is not set and toId it not set, most recent order are returned in descending order.

## Query Sub-account (USER_DATA)

- `GET /api/v1/account/subAccount`


### Weight：5

> Response：

``` json
[
    {
        "accountId": "122216245228131",
        "nickName": "",
        "accountType": 1,
        "accountIndex": 0 
    },
    {
        "accountId": "482694560475091200",
        "nickName": "createSubAccountByCurl", 
        "accountType": 1, // 1 Spot Account 3 Futures Account
        "accountIndex": 1
    }
]
```

### Parameters

| Name    | Type  |    Mandatory           | Description           |
| ----------------- | ---- | ------- | ------------- |
| recvWindow | LONG | NO | recv window |
| timestamp | LONG | YES | Timestamp |


## New Account Transfer

- `POST /api/v1/account/assetTransfer`

### Weight：1

> Response：

``` json
{
    "success":"true" 
}
```

### Parameters
| Name    | Type  |    Mandatory           | Description     |
| ----------------- | ---- | ------- |-----------------|
| fromAccountId | LONG | YES | from account id |
| toAccountId | LONG | YES | to account id   |
| coin | STRING | YES | coin            |
| quantity | DECIMAL | YES |       quantity          |
| recvWindow | LONG | NO |    recv window             |
| timestamp | LONG | YES |    timestamp             |


## Get Account Transaction History List (USER_DATA)
- `GET /api/v1/account/balanceFlow`

### Weight：5

> Response：

``` json
[
    {
        "id": "539870570957903104",
        "accountId": "122216245228131",
        "coin": "BTC",
        "coinId": "BTC",
        "coinName": "BTC",
        "flowTypeValue": 51, 
        "flowType": "USER_ACCOUNT_TRANSFER", 
        "flowName": "Transfer", 
        "change": "-12.5", 
        "total": "379.624059937852365", 
        "created": "1579093587214"
    },
    {
        "id": "536072393645448960",
        "accountId": "122216245228131",
        "coin": "USDT",
        "coinId": "USDT",
        "coinName": "USDT",
        "flowTypeValue": 7,
        "flowType": "AIRDROP",
        "flowName": "Airdrop",
        "change": "-2000",
        "total": "918662.0917630848",
        "created": "1578640809195"
    }
]
```

### Parameters
| Name    | Type  |    Mandatory           | Description                             |
| ----------------- | ---- | ------- |-----------------------------------------|
| accountType | INT | NO | Account corresponding to `account_type` |
| coin | STRING | NO | coin                                    |
| flowType | INT | NO | transfer：3                              |
| fromId | LONG | NO | from id                                 |
| endId | LONG | NO | end id                                  |
| startTime | LONG | NO | start timestamp                         |
| endTime | LONG | NO | end timestamp                           |
| limit | INT | NO | limit                                   |
| recvWindow | LONG | NO |       recv window                                  |
| timestamp | LONG | YES |       timestamp                                  |

## Get API KEY Type (USER_DATA)

- `GET /api/v1/account/checkApiKey`

> Response

``` json
{
    "accountType": "master"
}
```

### 参数
| Name     | Type      | Mandatory      | Description |
| ----------- | ------- | ------------- |-------------|
| recvWindow | LONG | NO | recv window |
| timestamp | LONG | YES |    timestamp         |

-  accountType:
  - master
  - spot
  - contract

# User Data Streams

- The base API endpoint is : **https://api.toobit.com** 
- A User Data Stream  `listenKey` is valid for 60 minutes after creation.
- Doing a  `PUT` on a `listenKey` will extend its validity for 60 minutes.
- Doing a `DELETE` on a `listenKey` will close the stream and invalidate the `listenKey` .
- Doing a `POST` on an account with an active `listenKey` will return the currently active `listenKey `and extend its validity for 60 minutes.
- The base websocket endpoint is: **wss://stream.toobit.com**
- User feeds are accessible via `/api/v1/ws/<listenKey>` (e.g. `wss://#HOST/api/v1/ws/<listenKey>`)
- Each link is valid for no more than 24 hours, please properly handle disconnection and reconnection.
- User feed payloads are not guaranteed to be up during busy times; make sure to order updates with `E`

## Listen Key (SPOT)

### Create a ListenKey  (USER_STREAM)
- `POST /api/v1/userDataStream`

Start a new user data stream. The stream will close after 60 minutes unless a keepalive is sent.
#### Weight：1

> Response

``` json
{
  "listenKey": "1A9LWJjuMwKWYP4QQPw34GRm8gz3x5AephXSuqcDef1RnzoBVhEeGE963CoS1Sgj"
}
```

#### Parameters
| Name     | Type      | Mandatory      | Description           |
| ----------- | ------- | ------------- | -------------- |
| recvWindow | LONG | NO | recv window |
| timestamp | LONG | YES | timestamp |

### Ping/Keep-alive a ListenKey (USER_STREAM)
- `PUT /api/v1/userDataStream`

Keepalive a user data stream to prevent a time out. User data streams will close after 60 minutes. It's recommended to send a ping about every 30 minutes.

#### Weight：1

> Response

``` json
{}
```

#### Parameters
| Name     | Type      | Mandatory      | Description           |
| ----------- | ------- | ------------- | -------------- |
| listenKey | STRING | YES | |
| recvWindow | LONG | NO |  |
| timestamp | LONG | YES |  |

### Close a ListenKey (USER_STREAM)

- `DELETE /api/v1/userDataStream`

#### Weight：1

> Response

``` json
{}
```

#### Parameters
| Name     | Type      | Mandatory      | Description |
| ----------- | ------- | ------------- |-------------|
| listenKey | STRING | YES | listenKey   |
| recvWindow | LONG | NO | recv window |
| timestamp | LONG | YES |    timestamp         |

## Payload: Account Update

> Payload

``` json
[
  {
    "e": "outboundAccountInfo",   // Event type
    "E": 1499405658849,           // Event time
    "T": true,                    // Can trade
    "W": true,                    // Can withdraw
    "D": true,                    // Can deposit
    "B": [                        // Balances changed 
      {
        "a": "LTC",               // Asset 
        "f": "17366.18538083",    // Free amount 
        "l": "0.00000000"         // Locked amount 
      }
    ]
  }
]
```

Whenever the account balance changes, an event `outboundAccountInfo` is sent containing the assets that may have been moved by the event that generated the balance change.


## Payload: Order Update

> Payload

``` json
[
  {
    "e": "executionReport",        // Event type 
    "E": 1499405658658,            // Event time 
    "s": "ETHBTC",                 // Symbol 
    "c": 1000087761,               // Client order ID 
    "S": "BUY",                    // Side 
    "o": "LIMIT",                  // Order type 
    "f": "GTC",                    // Time in force 
    "q": "1.00000000",             // Order quantity 
    "p": "0.10264410",             // Order price 
    "X": "NEW",                    // Current order status 
    "i": 4293153,                  // Order ID 
    "l": "0.00000000",             // Last executed quantity 
    "z": "0.00000000",             // Cumulative filled quantity 
    "L": "0.00000000",             // Last executed price 
    "n": "0",                      // Commission amount 
    "N": null,                     // Commission asset 
    "u": true,                     // Is the trade normal, ignore for now 
    "w": true,                     // Is the order working? Stops will have
    "m": false,                    // Is this trade the maker side?
    "O": 1499405658657,            // Order creation time 
    "Z": "0.00000000"              // Cumulative quote asset transacted quantity 
  }
]
```

Order updates are updated through the `executionReport` event. Check out the API docs and the relevant enum definitions below.
The average price can be found by dividing Z by z.

### Execution Type

- NEW  
- PARTIALLY_FILLED 
- FILLED 
- CANCELED 
- REJECTED 

## Payload: Ticket push

> Payload

``` json
[
    {
        "e": "ticketInfo",                // Event type 
        "E": "1668693440976",             // Event time 
        "s": "BTCUSDT",                   // Symbol 
        "q": "0.205",                     // quantity 
        "t": "1668693440899",             // time 
        "p": "441.0",                     // price 
        "T": "1291488620385157122",       // ticketId
        "o": "1291488620167835136",       // orderId 
        "c": "1668693440093",             // clientOrderId 
        "O": "1291354087841869312",       // matchOrderId 
        "a": "1286424214388204801",       // accountId 
        "A": "1270447370291795457",       // matchAccountId 
        "m": false,                       // isMaker 
        "S": "SELL"                       // side  SELL or BUY
    }
]
```

# Error Codes

Errors consist of two parts: an error code and a message. Codes are universal, but messages can vary.


> The error JSON payload

``` json
{  
  "code":-1121, 
  "msg":"Invalid symbol."
}
```

## 10xx - General Server or Network issues

### -1000 UNKNOWN
- An unknown error occurred while processing the request.

### -1001 DISCONNECTED
- Internal error; unable to process your request. Please try again.

### -1002 UNAUTHORIZED
- You are not authorized to execute this request.

### -1003 TOO_MANY_REQUESTS
- Too many requests queued.
- Too much request weight used; current limit is %s request weight per %s. Please use WebSocket Streams for live updates to avoid polling the API.
- Way too much request weight used; IP banned until %s. Please use WebSocket Streams for live updates to avoid bans.

### -1006 UNEXPECTED_RESP
- An unexpected response was received from the message bus. Execution status unknown.

### -1007 TIMEOUT
- Timeout waiting for response from backend server. Send status unknown; execution status unknown.

### -1014 UNKNOWN_ORDER_COMPOSITION
- Unsupported order combination.

### -1015 TOO_MANY_ORDERS
- Reach the rate limit .Please slow down your request speed.
- Too many new orders.
- Too many new orders; current limit is %s orders per %s.

### -1016 SERVICE_SHUTTING_DOWN
- This service is no longer available.

### -1020 UNSUPPORTED_OPERATION
- This operation is not supported.

### -1021 INVALID_TIMESTAMP
- Timestamp for this request is outside of the recvWindow.
- Timestamp for this request was 1000ms ahead of the server's time.
- Please check the difference between your local time and server time .

### -1022 INVALID_SIGNATURE
- Signature for this request is not valid.

## 11xx - 2xxx Request issues

### -1100 ILLEGAL_CHARS
- Illegal characters found in a parameter.
- Illegal characters found in parameter '%s'; legal range is '%s'.

### -1101 TOO_MANY_PARAMETERS
- Too many parameters sent for this endpoint.
- Too many parameters; expected '%s' and received '%s'.
- Duplicate values for a parameter detected.

### -1102 MANDATORY_PARAM_EMPTY_OR_MALFORMED
- A mandatory parameter was not sent, was empty/null, or malformed.
- Mandatory parameter '%s' was not sent, was empty/null, or malformed.
- Param '%s' or '%s' must be sent, but both were empty/null!

### -1103 UNKNOWN_PARAM
- An unknown parameter was sent.
- In BBEx Open Api , each request requires at least one parameter. {Timestamp}.

### -1104 UNREAD_PARAMETERS
- Not all sent parameters were read.
- Not all sent parameters were read; read '%s' parameter(s) but was sent '%s'.

### -1105 PARAM_EMPTY
- A parameter was empty.
- Parameter '%s' was empty.

### -1106 PARAM_NOT_REQUIRED
- A parameter was sent when not required.
- Parameter '%s' sent when not required.

### -1111 BAD_PRECISION
- Precision is over the maximum defined for this asset.

### -1112 NO_DEPTH
- No orders on book for symbol.

### -1114 TIF_NOT_REQUIRED
- TimeInForce parameter sent when not required.

### -1115 INVALID_TIF
- Invalid timeInForce.
- In the current version, this parameter is either empty or GTC.

### -1116 INVALID_ORDER_TYPE
- Invalid orderType.
- In the current version , ORDER_TYPE values is LIMIT or MARKET.

### -1117 INVALID_SIDE
- Invalid side.
- ORDER_SIDE values is BUY or SELL

### -1118 EMPTY_NEW_CL_ORD_ID
- New client order ID was empty.

### -1119 EMPTY_ORG_CL_ORD_ID
- Original client order ID was empty.

### -1120 BAD_INTERVAL
- Invalid interval.

### -1121 BAD_SYMBOL
- Invalid symbol.

### -1125 INVALID_LISTEN_KEY
- This listenKey does not exist.

### -1127 MORE_THAN_XX_HOURS
- Lookup interval is too big.
- More than %s hours between startTime and endTime.

### -1128 OPTIONAL_PARAMS_BAD_COMBO
- Combination of optional parameters invalid.

### -1130 INVALID_PARAMETER
- Invalid data sent for a parameter.
- Data sent for paramter '%s' is not valid.

### -1132 ORDER_PRICE_TOO_HIGH
- Order price too high.

### -1133 ORDER_PRICE_TOO_SMALL
- Order price lower than the minimum,please check general broker info.

### -1134 ORDER_PRICE_PRECISION_TOO_LONG
- Order price decimal too long,please check general broker info.

### -1135 ORDER_QUANTITY_TOO_BIG
- Order quantity too large.

### -1136 ORDER_QUANTITY_TOO_SMALL
- Order quantity lower than the minimum.

### -1137 ORDER_QUANTITY_PRECISION_TOO_LONG
- Order quantity decimal too long.

### -1138 ORDER_PRICE_WAVE_EXCEED
- Order price exceeds permissible range.

### -1139 ORDER_HAS_FILLED
- Order has been filled.

### -1140 ORDER_AMOUNT_TOO_SMALL
- Transaction amount lower than the minimum.

### -1141 ORDER_DUPLICATED
- Duplicate clientOrderId

### -1142 ORDER_CANCELLED
- Order has been canceled

### -1143 ORDER_NOT_FOUND_ON_ORDER_BOOK
- Cannot be found on order book

### -1144 ORDER_LOCKED
- Order has been locked

### -1145 ORDER_NOT_SUPPORT_CANCELLATION
- This order type does not support cancellation

### -1146 ORDER_CREATION_TIMEOUT
- Order creation timeout

### -1147 ORDER_CANCELLATION_TIMEOUT
- Order cancellation timeout

### -1193 ORDER_COUNT_LIMIT
- Create order count limit

### -1194 MARKET_ORDER_FORBIDDEN
- Create market order forbidden

### -1195 LIMIT_ORDER_PRICE_TOO_SMALL
- Create limit order price too small

### -1196 LIMIT_ORDER_PRICE_TOO_BIG
- Create limit order price too big

### -1197 LIMIT_ORDER_BUY_PRICE_TOO_BIG
- Create limit order buy price too big

### -1198 LIMIT_ORDER_SELL_PRICE_TOO_SMALL
- Create limit order sell price too small

### -1199 ORDER_BUY_QUANTITY_TOO_SMALL
- Create order buy quantity too small

### -1200 ORDER_BUY_QUANTITY_TOO_BIG
- Create order buy quantity too big

### -1201 LIMIT_ORDER_SELL_PRICE_TOO_BIG
- Create limit order sell price too big

### -1202 ORDER_SELL_QUANTITY_TOO_SMALL
- Create order sell quantity too small

### -1203 ORDER_SELL_QUANTITY_TOO_BIG
- Create order sell quantity too big
- 
### -1206 ORDER_AMOUNT_TOO_BIG
- Orders over the maximum transaction amount

### -2010 NEW_ORDER_REJECTED
- NEW_ORDER_REJECTED

### -2011 CANCEL_REJECTED
- CANCEL_REJECTED

### -2013 NO_SUCH_ORDER
- Order does not exist.

### -2014 BAD_API_KEY_FMT
- API-key format invalid.

### -2015 REJECTED_MBX_KEY
- Invalid API-key, IP, or permissions for action.

### -2016 NO_TRADING_WINDOW
- No trading window could be found for the symbol. Try ticker/24hrs instead.

## Filter failures

| Error message	     | Description           |
| ----------- | -------------- |
| "Filter failure: PRICE_FILTER" | price is too high, too low, and/or not following the tick size rule for the symbol.|
| "Filter failure: LOT_SIZE" | quantity is too high, too low, and/or not following the step size rule for the symbol.|
| "Filter failure: MIN_NOTIONAL" |price* quantity is too low to be a valid order for the symbol.|
| "Filter failure: MAX_NUM_ORDERS" |	Account has too many open orders on the symbol.|
| "Filter failure: MAX_ALGO_ORDERS"| Account has too many open stop loss and/or take profit orders on the symbol. |
| "Filter failure: ICEBERG_PARTS" | Iceberg order would break into too many parts; icebergQty is too small.|

## Order Rejection Issues

| Error message	     | Description           |
| ----------- | -------------- |
|"Unknown order sent." | The order (by either orderId, clOrdId, origClOrdId) could not be found|
|"Duplicate order sent." | The clOrdId is already in use|
|"Market is closed." | The symbol is not trading |
|"Account has insufficient balance for requested action." | Not enough funds to complete the action  |
|"Market orders are not supported for this symbol." | MARKET is not enabled on the symbol |
|"Iceberg orders are not supported for this symbol." | icebergQty is not enabled on the symbol |
|"Stop loss orders are not supported for this symbol." | STOP_LOSS is not enabled on the symbol |
|"Stop loss limit orders are not supported for this symbol." | STOP_LOSS_LIMIT is not enabled on the symbol |
|"Take profit orders are not supported for this symbol." | TAKE_PROFIT is not enabled on the symbol|
|"Take profit limit orders are not supported for this symbol." | TAKE_PROFIT_LIMIT is not enabled on the symbol |
|"Price* QTY is zero or less." | price* quantity is too low|
|"IcebergQty exceeds QTY." | icebergQty must be less than the order quantity|
|"This action disabled is on this account." | Contact customer support; some actions have been disabled on the account.|
|"Unsupported order combination" | The orderType, timeInForce, stopPrice, and/or icebergQty combination isn't allowed.|
|"Order would trigger immediately." | The order's stop price is not valid when compared to the last traded price. |
|"Cancel order is invalid. Check origClOrdId and orderId." | No origClOrdId or orderId was sent in.|
|"Order would immediately match and take." | LIMIT_MAKER order type would immediately match and trade, and not be a pure maker order.|
