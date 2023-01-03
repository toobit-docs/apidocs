---
title: USDT-M API Document

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


# General Info

## General API Information

- Some endpoints will require an API Key. Please refer to<a href='https://toobit.zendesk.com/hc/en-001/articles/13445077851545-How-to-Create-Your-API-Key'>this page</a>
- The base endpoint is: **https://api.toobit.com**
- All endpoints return either a JSON object or array.
- All time and timestamp related fields are in milliseconds.
- All data types adopt definition in JAVA.

## LIMITS
- The  `/api/v1/exchangeInfo` `rateLimits` array contains objects related to the exchange's `RAW_REQUEST`, `REQUEST_WEIGHT`, and `ORDER `rate limits. 
- A 429 will be returned when either rate limit is violated.
- Each line has a weight characteristic, which determines how much capacity the request occupies (for example, weight=2 indicates that the request occupies the amount of two requests). Endpoints that return a lot of data or perform tasks on multiple symbols may have a higher weight

## Endpoint Security Type
- Each endpoint has a security type that determines the how you will interact with it.
- API-keys are passed into the Rest API via the `X-BB-APIKEY`header.
- API-keys and secret-keys are **case sensitive**.
- API-keys can be configured to only access certain types of secure endpoints. For example, one API-key could be used for TRADE only, while another API-key can access everything except for TRADE routes.

| Security Type    | Description | 
| --------- | ---- | 
| NONE      | Endpoint can be accessed freely. | 
| TRADE     | Endpoint requires sending a valid API-Key and signature.|
| USER_DATA | Endpoint requires sending a valid API-Key and signature. |
| USER_STREAM | Endpoint requires sending a valid API-Key. |
| MARKET_DATA|  Endpoint requires sending a valid API-Key. |

## SIGNED (TRADE and USER_DATA) Endpoint Security
- `SIGNED` endpoints require an additional parameter, signature, to be sent in the `query string` or `request body`.
- The signature uses the `HMAC SHA256` algorithm. The API-Secret corresponding to the API-KEY is used as the key of `HMAC SHA256`, and all other parameters are used as the operation object of `HMAC SHA256`, and the output obtained is the signature
- The signature is not case sensitive.
- When using query string and request body at the same time, the input query string of `HMAC SHA256` comes first, and the request body comes after
### Timing Security
- A SIGNED endpoint also requires a parameter, timestamp, to be sent which should be the millisecond timestamp of when the request was created and sent.
- When the server receives the request, it will judge the timestamp in the request. If it is sent before 5000 milliseconds, the request will be considered invalid. This time window value can be customized by sending the optional parameter `recvWindow`
- In addition, if the server calculates that the client's timestamp is more than one second "in the future" of the server's time, the request will also be rejected.

> The logic is as follows:

``` javascript
if (timestamp < (serverTime + 1000) && (serverTime - timestamp) <= recvWindow) {
  // process request
} else {
  // reject request
}
```

**Serious trading is about timing** 互Networks can be unstable and unreliable, which can lead to requests taking varying amounts of time to reach the servers. With recvWindow, you can specify that the request must be processed within a certain number of milliseconds or be rejected by the server.
<aside class="notice">
 It is recommended to use a small recvWindow of 5000 or less!
</aside>

### SIGNED Endpoint Examples for POST /api/v1/futures/order

Here is a step-by-step example of how to send a vaild signed payload from the Linux command line using echo, openssl, and curl.

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

#### Example 1: As a query string
``` shell
Example 1:
HMAC SHA256 signature:
$ echo -n "symbol=BTCUSDT&side=SELL&type=LIMIT&timeInForce=GTC&quantity=1&price=400&recvWindow=100000&timestamp=1668481902307" | openssl dgst -sha256 -hmac "30lfjDT51iOG1kYZnDoLNynOyMdIcmQyO1XYfxzYOmQfx9tjiI98Pzio4uhZ0Uk2"
(stdin)= 8420e499e71cce4a00946db16543198b6bcae01791bdb75a06b5a7098b156468
```

``` bash
curl command:
(HMAC SHA256)
$ curl -H "X-BB-APIKEY: SRQGN9M8Sr87nbfKsaSxm33Y6CmGVtUu9Erz73g9vHFNn36VROOKSaWBQ8OSOtSq" -X POST 'https://api.toobit.com/api/v1/futures/order' -d 'symbol=BTCUSDT&side=SELL&type=LIMIT&timeInForce=GTC&quantity=1&price=400&recvWindow=100000&timestamp=1668481902307&signature=8420e499e71cce4a00946db16543198b6bcae01791bdb75a06b5a7098b156468'

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

Note that the signature is different in example 3.There is no & between "GTC" and "quantity=1".

## Public Endpoints Info

### Terminology

- `base asset` refers to the asset that is the `quantity` of a symbol.
- `quote asset`refers to the asset that is the `price` of a symbol.

### ENUM definitions

#### Symbol type:
- FUTURE 

#### Asset Type:
- CASH 
- MARGIN 

#### Order status:
- NEW 
- PARTIALLY_FILLED 
- FILLED 
- CANCELED 
- PENDING_CANCEL 
- REJECTED 

#### Order type:
- MARKET 
- LIMIT 
- LIMIT_MAKER  
- STOP 
- STOP_SHORT_PROFIT 
- STOP_LONG_PROFIT 
- STOP_LONG_LOSS  
- STOP_SHORT_LOSS 
- LIQUI_IOC_ORDER 
- LIQUI_ADL_ORDER 

### Plan order or stop loss order status

- ORDER_NEW 
- ORDER_FILLED 
- ORDER_REJECTED 
- ORDER_CANCELED 
- ORDER_FAILED 

### Take profit and stop loss type

- STOP_LONG_PROFIT  Long position take profit
- STOP_LONG_LOSS Long position stop loss
- STOP_SHORT_PROFIT Plan entrustment-short position take profit
- STOP_SHORT_LOSS Plan order - short position stop loss

#### Order side :
- BUY 
- SELL 

#### Time in force :
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

#### Rate limiters :
- REQUESTS_WEIGHT
- ORDERS

#### Rate limit intervals
- SECOND
- MINUTE
- DAY

# Market Data Endpoints

## Test Connectivity
-  `GET /api/v1/ping`

Test connectivity to the Rest API.

### Weight: 1

### Parameters：

NONE

> Response: 

```json
{}

```

## Check Server Time
- `GET /api/v1/time`

Test connectivity to the Rest API and get the current server time.

### Weight: 1

### Parameters: 

NONE

> Response: 

```json
{
  "serverTime": 1538323200000
}
```

## Exchange Information
- `GET /api/v1/exchangeInfo`

Current exchange trading rules and symbol information

### Weight：1

### Parameters：

NONE

> Response:

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
        }
      ],
      "exchangeId": "301",
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
      "exchangeId": "301",
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
      "exchangeId": "301",
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
        }
      ],
      "exchangeId": "301",
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
      "exchangeId": "301",
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
      "orgId": "9001",
      "coinId": "ETH",
      "coinName": "ETH",
      "coinFullName (tokenFullName)": "Ethereum",
      "allowWithdraw": true,
      "allowDeposit": true,
      "chainTypes": []
    },
    {
      "orgId": "9001",
      "coinId": "USDT",
      "coinName": "USDT",
      "coinFullName (tokenFullName)": "TetherUS",
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
      "orgId": "9001",
      "coinId": "BTC",
      "coinName": "BTC",
      "coinFullName": "Bitcoin",
      "allowWithdraw": false,
      "allowDeposit": false,
      "chainTypes": []
    },
    {
      "orgId": "9001",
      "coinId": "UNI",
      "coinName": "UNI",
      "coinFullName": "uniswap",
      "allowWithdraw": false,
      "allowDeposit": false,
      "chainTypes": []
    },
    {
      "orgId": "9001",
      "coinId": "XRP",
      "coinName": "XRP",
      "coinFullName (tokenFullName)": "XRP",
      "allowWithdraw": false,
      "allowDeposit": false,
      "chainTypes": []
    },
    {
      "orgId": "9001",
      "coinId": "EOS",
      "coinName": "EOS1",
      "coinFullName (tokenFullName)": "EOS",
      "allowWithdraw": true,
      "allowDeposit": true,
      "chainTypes": []
    },
    {
      "orgId": "9001",
      "coinId": "JET",
      "coinName": "JET",
      "coinFullName (tokenFullName)": "JET",
      "allowWithdraw": false,
      "allowDeposit": false,
      "chainTypes": []
    }
  ]
}


```
## Order Book 

- `GET /quote/v1/depth`

### Weight:

Adjusted based on the limit:

| Limit    |  Weight  | 
| -------- | ---- | 
| 5, 10, 20, 50, 100 | 1 |
| 500 | 5 |
| 1000 | 10 |

> Response：

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
| Name    | Type  |    Mandatory           | Description           |
| ----------------- | ---- | ------- | ------------- |
| symbol | STRING | YES |  |
| limit | INT | NO | Default 100; Max 100|

Notes：If `limit=0` is set, a lot of data will be returned.

## Merge Depth

- `GET /quote/v1/depth/merged`

Merged deep interface.

### Weight：1

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
| Name    | Type  |    Mandatory           | Description           |
| ----------------- | ---- | ------- | ------------- |
| symbol | STRING | YES |  |
| scale | INT |  NO | Gears: `0`,`1`,`2`,`3`,`4`,`5` For example: `0 `means gear `1`, `1` means gear `2` |
| limit | INT |  NO |  |


## Recent Trades List

- `GET /quote/v1/trades`

Get recent market trades

### Weight：1

>Response：

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
| Name    | Type  |    Mandatory           | Description           |
| ----------------- | ---- | ------- | ------------- |
| symbol | STRING | YES |  |
| limit | INT | NO | Default 60; Max 60|


## Kline/Candlestick Data

- `GET /quote/v1/klines`

Kline/candlestick bars for a symbol. Klines are uniquely identified by their open time.

### Weight：1

> Response：

``` json
[
  [
    1499040000000,      // Open time
    "0.01634790",       // Open price
    "0.80000000",       // High price
    "0.01575800",       // Low price
    "0.01577100",       // Close price
    "148976.11427815",  // Volume
    1499644799999,      // Close time
    "2434.19055334",    // Quote asset volume
    308,                //  Number of trades
    "1756.87402397",    // Taker buy base asset volume
    "28.46694368"       // Taker buy quote asset volume
  ]
]
```

### Parameters
| Name    | Type  |    Mandatory           | Description           |
| ----------------- | ---- | ------- | ------------- |
| symbol | STRING | YES |  |
| interval | ENUM | YES |  |
| startTime | LONG | NO | start timestamp |
| endTime | LONG | NO | en d timestamp|
| limit | INT | NO | Default 100; Max 1000|

Notes: If startTime and endTime are not sent, only the latest K line will be returned.


## Index Price Kline/Candlestick Data 

- `GET /api/quote/v1/index/klines`

Kline/candlestick bars for the index price of a pair.

> Response：

``` json
{
    "code": 200,
    "data": [
        {
            "t": 1669155300000,//time
            "s": "ETHUSDT",// symbol
            "sn": "ETHUSDT",//symbol name
            "c": "1127.1",//Close price
            "h": "1130.81",//High price
            "l": "1126.17",//Low price
            "o": "1130.8",//Open price
            "v": "0"//Volume
        },
        {
            "t": 1669156200000,
            "s": "ETHUSDT",
            "sn": "ETHUSDT",
            "c": "1129.44",
            "h": "1129.54",
            "l": "1127.1",
            "o": "1127.1",
            "v": "0"
        }
]
}
```

### Parameters
| Name    | Type  |    Mandatory           | Description           |
| ----------------- | ---- | ------- | ------------- |
| symbol | STRING | YES |  |
| interval | ENUM | YES |  |
| from | LONG | YES | start timestamp |
| to | LONG | YES | end timestamp|
| limit | INT | NO |  |


## Mark Price Kline/Candlestick Data

- `GET /api/quote/v1/markPrice/klines`

Kline/candlestick bars for the mark price of a symbol.

> Response：

``` json
{
    "code": 200,
    "data": [
        {
            "symbol": "BTCUSDT",// Symbol
            "time": 1670157900000,// time
            "low": "16991.14096",//Low price
            "open": "16991.78288",//Open price
            "high": "16996.30641",// High prce
            "close": "16996.30641",// Close price
            "volume": "0",// Volume
            "curId": 1670157900000
        }
    ]
}
```

### Parameters
| Name    | Type  |    Mandatory           | Description           |
| ----------------- | ---- | ------- | ------------- |
| symbol | STRING | YES |  |
| interval | ENUM | YES |  |
| from | LONG | YES | start timestamp |
| to | LONG | YES | end timestamp|
| limit | INT | NO |  |


## Mark Price
- `GET /quote/v1/markPrice`

Get the mark price of a trading pair.

> Response：

``` json
{
  "exchangeId": 301,
  "symbolId": "BTC-SWAP-USDT",// Symbol
  "price": "17042.54471",// Mark price
  "time": 1670897454000// time
}
```

### Parameters
| Name    | Type  |    Mandatory           | Description           |
| ----------------- | ---- | ------- | ------------- |
| symbol | STRING | YES |  |

## Funding Rate
- `GET /api/v1/futures/fundingRate` 

> Response：

``` json
[
    {
        "symbol": "BTC-SWAP-USDT", // Symbol
        "lastFundingTime": "1668398400000", //
        "nextFundingTime": "1668427200000", //Next fund fee settlement time
        "rate": "0.0018099173553719" //The settlement funding rate.
    }
]
```

### Parameters
| Name    | Type  |    Mandatory           | Description           |
| ----------------- | ---- | ------- | ------------- |
| symbol | STRING | YES |  |

## Get Funding Rate History

- `GET /api/v1/futures/historyFundingRate`

> Response：

``` json
[
  { 
    "id": "3434343434343",
    "symbol": "BTC-PERP-REV", // Symbol
    "fundingTime": "1570708800000", 
    "fundingRate": "0.00321" 
  }
]
```

### Parameters
| Name    | Type  |    Mandatory           | Description           |
| ----------------- | ---- | ------- | ------------- |
| symbol | STRING | YES |  |
| fromId | LONG | NO | start id |
| endId | LONG | NO | end id |
| limit | INT | NO |  |

## 24hr Ticker Price Change Statistics

- `GET /quote/v1/ticker/24hr`

24 hour rolling window price change statistics.<br>
**Careful** when accessing this with no symbol.

### Weight： 

1 for a single symbol;<br>
40 when the symbol parameter is omitted

> Response：

``` json
[
    {
        "t": 1538725500422,   // time
        "a": "1.10000000",    // highest selling price
        "b": "1.00000000",    // highest bid
        "s": "ETHBTC",        // symbol 
        "c": "4.00000200",    // latest transaction price
        "o": "99.00000000",   // open price
        "h": "100.00000000",  // high price 
        "l": "0.10000000",    // low price
        "v": "8913.30000000", // Base asset volume
        "qv": "15.30000000"   // Quote asset volume
    }
]
```

### Parameters
| Name    | Type  |    Mandatory           | Description           |
| ----------------- | ---- | ------- | ------------- |
| symbol | STRING | NO |  |
| realtimeInterval | ENUM | NO | `24h`,`1d`,`1d+8` |

- If the symbol is not sent, tickers for all symbols will be returned in an array.


## Symbol Price Ticker
- `GET /quote/v1/ticker/price`

Latest price for a symbol or symbols.

### Weight：1

> Response：

``` json
[
  {
    "s": "LTCBTC",     // Symbol
    "p": "4.00000200"  // Latest price
  }
]
```
### Parameters
| Name    | Type  |    Mandatory           | Description           |
| ----------------- | ---- | ------- | ------------- |
| symbol | STRING | NO |  |

- If the symbol is not sent, tickers for all symbols will be returned in an array.

## Symbol Order Book Ticker

- `GET /quote/v1/ticker/bookTicker`

Best price/qty on the order book for a symbol or symbols.

### Weight：1

> Response：

``` json
[
  {
      "t": 1535975085052,     // Time
      "s": "LTCBTC",            // Symbol         
      "b": "4.00000000",        // bidPrice
      "bq": "431.00000000",     // bidQty
      "a": "4.00000200",        // askPrice
      "aq": "9.00000000"        // askQty
  }
]
```

### Parameters
| Name    | Type  |    Mandatory           | Description           |
| ----------------- | ---- | ------- | ------------- |
| symbol | STRING | NO |  |

- If the symbol is not sent, bookTickers for all symbols will be returned in an array.

# Websocket Market Streams

- The base endpoint is:  wss://stream.toobit.com
- The URL format for direct access is: wss://#HOST/quote/ws/v1

## Live Subscribing/Unsubscribing to streams

- The following data can be sent via websocket to implement subscription or unsubscription data stream.


| Name    | Value  |  
| ----------------- | ---- | 
| topic | `realtimes`, `trade`, `kline_$interval`, `depth` | 
| event | `sub`, `cancel`, `cancel_all`  |
| interval | 1m, 5m, 15m, 30m, 1h, 2h, 6h, 12h, 1d, 1w, 1M |

### Subscribe to a stream Request:

`{`

  `"symbol": "$symbol0, $symbol1", `

  `"topic": "$topic",`

  `"event": "sub",`
  
  `  "params": {`

  `      "limit": "$limit",  `

  `      "binary": "false" `

  `  }`

`}`

## Unsubscribe to a stream Request:

`{`

  `"symbol": "$symbol0, $symbol1", `

  `"topic": "$topic",`

  `"event": "cancel",`
  
  `  "params": {`

  `      "limit": "$limit",  `

  `      "binary": "false" `

  `  }`

`}`

## Heartbeat

Every once in a while, the client needs to send a ping frame, and the server will reply with a pong frame, otherwise the server will actively disconnect within 5 minutes.

### Request

`{`
    `"ping": 1535975085052`
`}`

> Respose：

``` json
{
    "pong": 1535975085052
}
```

## Latest Contract Price Stream

Push the information of each transaction transaction by transaction. A deal, or the definition of a transaction, is that there is only one taker and one maker trading with each other. <br>
After successfully connecting to the server, the server will first push a recent 60 transactions. After this push, each push is a real-time transaction. <br>
The variable "v" can be understood as a transaction ID. This variable is globally incremented and unique. <br>
For example: Suppose there have been 3 transactions in the past 5 seconds, namely `ETHUSDT`, `BTCUSDT`, `BHTBTC`. Their "v" will be consecutive values (112, 113, 114).

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
            "v": "1291465821801168896", // Volume
            "t": 1668690723096, // time
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

  `"symbol": "$symbol0, $symbol1",// Symbol`

  `"topic": "trade",`

  `"event": "sub",`

  `"params": {`

  `  "binary": false // Whether data returned is in binary format`

  `}`

`}`



## Mark Price Stream

Contract mark price.

> Payload

``` json
{
    "symbol": "BTCUSDT",
    "symbolName": "BTCUSDT",
    "topic": "markPrice",
    "params": {
        "realtimeInterval": "24h"
    },
    "data": [
        {
            "symbol": "BTCUSDT",//symbol
            "markPrice": "16792.28",// mark price 
            "formula": "(16792.28[HUOBI])/1",// source
            "time": 1668754084000
        }
    ],
    "f": false,
    "sendTime": 1668754084738,
    "shared": false
}
```

### Request:

`{`

  `"symbol": "$symbol0, $symbol1", //Symbol`

  `"topic": "markPrice",`

  `"event": "sub"`

`}`




## Kline/Candlestick Streams

The K-line stream pushes the update of the requested K-line type (the latest K-line) every second.

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
            "t": 1668753840000,// start timestamp
            "s": "BTCUSDT",// symbol
            "sn": "BTCUSDT",// symbol name
            "c": "445",//close price
            "h": "445",//high price
            "l": "445",//low price
            "o": "445",//open price
            "v": "0"// volume
        }
    ],
    "f": true,
    "sendTime": 1668753854576,
    "shared": false
}
```

### Request:

`{`

  `"symbol": "$symbol0, $symbol1",// symbol`

  `"topic": "kline_"+$interval,`

  `"event": "sub",`

  `"params": {`

  `  "binary": false`

  `}`

`}`


### K线/Kline/Candlestick chart intervals:

To subscribe to Kline, you need to provide an interval parameter, the shortest is the minute line, and the longest is the monthly line. The following intervals are supported:
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



## Individual Symbol Ticker Streams

24-hour complete ticker information refreshed second by symbol.

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
            "c": "445", // close price
            "h": "445", //high price
            "l": "310", //low price
            "o": "311", //open price
            "v": "3747.7597191", // Total traded base asset volume
            "qv": "1426443.9553995", // Total traded quote asset volume
            "m": "0.4309", // margin
            "e": 301 // 
        }
    ],
    "f": true, 
    "sendTime": 1668753481048,
    "shared": false
}
```

### Requese:

`{`

  `"symbol": "$symbol0, $symbol1",// symbol`

  `"topic": "realtimes",`

  `"event": "sub",`

  `"params": {`

  `  "binary": false`

  `}`

`}`




## Partial Book Depth Streams

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
      ["11369.04", "1.3203"]
    ],
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
  "f": true
}
```

### Request:

`{`

  `"symbol": "$symbol0, $symbol1",// symbol`

 ` "topic": "depth",`

  `"event": "sub",`

  `"params": {`

   ` "binary": false`

  `  }`

`}`


## Diff. Book Depth Streams

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
  "f": false 
}
```

### Request:

`{`

  `"symbol": "$symbol0, $symbol1",//Symbol`

  `"topic": "diffDepth",`

  `"event": "sub",`

  `"params": {`

  ` "binary": false` 
  `}`

`}`


Push the changing part of the order book (if any) every second. <br>
In incremental depth information, the quantity is not necessarily equal to the quantity corresponding to the price. If the quantity=0, it means that the price in the last push is no longer available. If the quantity>0, the quantity at this time is the quantity corresponding to the updated price<br>
Assume that there is such an item in the returned data we received:<br>
`["0.00181860", "155.92000000"]// price, quantity` <br>
If the next returned data contains: <br>
`["0.00181860", "12.3"]` <br>
This means that the quantity corresponding to this price has changed, and the changed quantity has been updated<br>
If the next returned data contains:<br>
`["0.00181860", "0"]`<br>
This means that the quantity corresponding to this price has disappeared and will be deleted in the client.

# Account/Trades Endpoints
<aside class="warning">
Considering the possible data latency from RESTful endpoints during an extremely volatile market, it is highly recommended to get the order status, position, etc from the Websocket user data stream.
</aside>

## New Future Account Transfer

- `POST /api/v1/account/assetTransfer`

Execute the transfer between the spot account and the contract account

### Weight：1

> Response：

``` json
{
    "success":"true" 
}
```

### Parameters
| Name    | Type  |    Mandatory           | Description           |
| ----------------- | ---- | ------- | ------------- |
| fromAccountId | LONG | YES |  |
| toAccountId | LONG | YES |  |
| coin | STRING | YES |  |
| quantity | DECIMAL | YES |  |
| recvWindow | LONG | NO |  |
| timestamp | LONG | YES |  |

## Query Sub-account (USER_DATA)

- `GET /api/v1/account/subAccount`


### Weight：5

> Response：

``` json
[
    {
        "accountId": "122216245228131",
        "accountName": "",
        "accountType": 1,
        "accountIndex": 0 
    },
    {
        "accountId": "482694560475091200",
        "accountName": "createSubAccountByCurl", 
        "accountType": 1, // 1 Spot Account 3 Futures Account
        "accountIndex": 1
    },
    {
        "accountId": "422446415267060992",
        "accountName": "",
        "accountType": 3,
        "accountIndex": 0
    },
    {
        "accountId": "482711469199298816",
        "accountName": "createSubAccountByCurl",
        "accountType": 3,
        "accountIndex": 1
    },
]
```

### Parameters

| Name    | Type  |    Mandatory           | Description           |
| ----------------- | ---- | ------- | ------------- |
| recvWindow | LONG | NO |  |
| timestamp | LONG | YES |  |


## Get Future Account Transaction History List (USER_DATA)
- `GET /api/v1/account/balanceFlow`

Obtain the history of fund transfers between the spot account and the contract account.

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
| Name    | Type  |    Mandatory           | Description           |
| ----------------- | ---- | ------- | ------------- |
| accountType | INT | NO |  |
| accountIndex | INT | NO |  |
| coin | STRING | NO |  |
| flowType | INT | NO | transfer：3 |
| fromId | LONG | NO |  |
| endId | LONG | NO |  |
| startTime | LONG | NO | start timestamp |
| endTime | LONG | NO | end timestamp |
| limit | INT | NO |  |
| recvWindow | LONG | NO |  |
| timestamp | LONG | YES |  |

## Change Margin Type (TRADE)
- `POST /api/v1/futures/marginType `

Change the user's margin mode on the specified symbol contract: isolated margin or cross margin.

### Weight：1

> Response：

``` json
{
    "code":200, 
    "symbol":"BTC-SWAP-USDT", //symbol
    "marginType":"CROSS" // CROSS ISOLATED
}
```

### Parameters
| Name    | Type  |    Mandatory           | Description           |
| ----------------- | ---- | ------- | ------------- |
| symbol | STRING | YES |  |
| marginType | ENUM | YES | `CROSS` `ISOLATED` |
| recvWindow | LONG | NO |  |
| timestamp | LONG | YES |  |

## Change Initial Leverage (TRADE)

- `POST /api/v1/futures/leverage `

Adjust the user's opening leverage in the specified symbol contract.

### Weight：1

>Response:

``` json
{
    "code": 200, 
    "symbol":"BTC-SWAP-USDT", 
    "leverage":"20" 
}
```

### Parameters
| Name    | Type  |    Mandatory           | Description           |
| ----------------- | ---- | ------- | ------------- |
| symbol | STRING | YES |  |
| leverage | INT | YES |  |
| recvWindow | LONG | NO |  |
| timestamp | LONG | YES |  |

## Get the leverage multiple and position mode (USER_DATA)
- `GET /api/v1/futures/accountLeverage`

Obtain the leverage multiples and position types of all contract trading pairs of the user. This API requires your request to be signed.

> Response：

``` json
[
    {
        "symbol":"BTC-SWAP-USDT", //symbol
        "leverage":"20", 
        "marginType":"CROSS" // CROSS;ISOLATED
    }
]
```

### Parameters
| Name    | Type  |    Mandatory           | Description           |
| ----------------- | ---- | ------- | ------------- |
| symbol | STRING | YES |  |
| recvWindow | LONG | NO |  |
| timestamp | LONG | YES |  |


## New Order (TRADE)
`POST /api/v1/futures/order`

> Response：

``` json
{
    "time": "1668418485058", // time
    "updateTime": "1668418485058", 
    "orderId": "1289182123551455488", 
    "clientOrderId": "test1115", 
    "symbol": "BTC-SWAP-USDT", // symbol
    "price": "19000", // price
    "leverage": "2", // leverage
    "origQty": "10", // quantity
    "executedQty": "0", //Executed Quantity
    "avgPrice": "0", //average  price
    "marginLocked": "9.5", //The margin locked for this order.
    "type": "LIMIT", // type（LIMIT and STOP）
    "side": "BUY_OPEN", // side（BUY_OPEN、SELL_OPEN、BUY_CLOSE、SELL_CLOSE）
    "timeInForce": "GTC", // GTC、FOK、IOC、LIMIT_MAKER
    "status": "NEW", //NEW、PARTIALLY_FILLED、FILLED、CANCELED、REJECTED
    "priceType": "INPUT" //INPUT、OPPONENT、QUEUE、OVER、MARKET
}
```

### Parameters
| Name    | Type  |    Mandatory           | Description           |
| ----------------- | ---- | ------- | ------------- |
| symbol | STRING | YES |  |
| side | ENUM | YES |  `BUY_OPEN`、`SELL_OPEN`、`BUY_CLOSE`、`SELL_CLOSE`|
| type | ENUM | YES |  `LIMIT` and `STOP` |
| quantity | LONG | YES |  |
| price | DECIMAL | NO | `LIMIT`&`INPUT` **Mandatory need** |
| priceType | ENUM | NO |  `INPUT`、`OPPONENT`、`QUEUE`、`OVER`、`MARKET` |
| stopPrice | DECIMAL | NO | `type` = `STOP` order **Mandatory need** |
| timeInForce | ENUM | NO | The time command (Time in Force) of `LIMIT` order, the currently supported types are `GTC`, `FOK`, `IOC`, `LIMIT_MAKER` |
| newClientOrderId | STRING | YES | The ID of the order, defined by the user |
| timestamp | LONG | YES | |
| recvWindow | LONG | NO | |


## Query Order (USER_DATA)
- `GET /api/v1/futures/order`

### Weight：1

> Response：

``` json
{
    "time": "1668418485058", // create time
    "updateTime": "1668418485058", 
    "orderId": "1289182123551455488", 
    "clientOrderId": "test1115", //User defined order ID
    "symbol": "BTC-SWAP-USDT", //symbol
    "price": "19000", //price
    "leverage": "2", //leverage
    "origQty": "10", //quantity
    "executedQty": "0", //Executed Quantity
    "avgPrice": "0", //average  price
    "marginLocked": "9.5", //The margin locked for this order.
    "type": "LIMIT", // LIMIT or STOP
    "side": "BUY_OPEN", // BUY_OPEN、SELL_OPEN、BUY_CLOSE、SELL_CLOSE
    "timeInForce": "GTC", // GTC、FOK、IOC、LIMIT_MAKER
    "status": "NEW", //NEW、PARTIALLY_FILLED、FILLED、CANCELED、REJECTED
    "priceType": "INPUT" //INPUT、OPPONENT、QUEUE、OVER、MARKET
}
```

### Parameters
| Name    | Type  |    Mandatory           | Description           |
| ----------------- | ---- | ------- | ------------- |
| orderId | LONG | NO |  |
| origClientOrderId | STRING | NO |  |
| type | ENUM | NO | `LIMIT` or `STOP` |
| timestamp | LONG | YES | |
| recvWindow | LONG | NO | |
 
Notes: 

- Either `orderId` or `origClientOrderId` must be sent

## Cancel Order (TRADE)
- `DELETE /api/v1/futures/order`

### Weight：1

> Response：

``` json
Synchronous Cancellation Response：

{
    "time": "1668418485058", // create time
    "updateTime": "1668418485058", 
    "orderId": "1289182123551455488", 
    "clientOrderId": "test1115", //User defined order ID
    "symbol": "BTC-SWAP-USDT", 
    "price": "19000", 
    "leverage": "2", 
    "origQty": "10", //quantity
    "executedQty": "0", //Executed Quantity
    "avgPrice": "0", //avg price
    "marginLocked": "9.5", //The margin locked for this order.
    "type": "LIMIT", 
    "side": "BUY_OPEN", 
    "timeInForce": "GTC", 
    "status": "NEW", 
    "priceType": "INPUT" 
}

Asynchronous Cancellation Response：

{
  "isCancelled":true  
}

```

### Parameters
| Name    | Type  |    Mandatory           | Description           |
| ----------------- | ---- | ------- | ------------- |
| orderId | LONG | NO |  |
| origClientOrderId | STRING | NO | User defined order ID |
| type | ENUM | NO | `LIMIT` or `STOP` |
| fastCancel | INT | NO | Default `0`(Synchronous Cancellation)，`1`Asynchronous order cancellation |
| symbol | STRING | NO |  |
| timestamp | LONG | YES | |
| recvWindow | LONG | NO | |

Notes：

- Either `orderId` or `origClientOrderId` must be sent

## Cancel Orders (TRADE)

- `DELETE /api/v1/futures/batchOrders`

### Weight：5

> Response:

``` json
{
  "message":"success",
  "timestamp":1541161088303
}
```

### Parameters

| Name    | Type  |    Mandatory           | Description           |
| ----------------- | ---- | ------- | ------------- |
| symbol | STRING | YES | |
| side | ENUM | NO | `BUY` or `SELL` |
| timestamp | LONG | YES | |
| recvWindow | LONG | NO | |

## Query Current Open Order  (USER_DATA)
- `GET /api/v1/futures/openOrders`

### Weight：1

> Response：

``` json
[
  {
    "time": "1570760254539", //create time
    "updateTime": "1570760254539", 
    "orderId": "469965509788581888", 
    "clientOrderId": "1570760253946",  //User defined order ID
    "symbol": "BTC-PERP-REV", 
    "price": "8502.34", 
    "leverage": "20", 
    "origQty": "222", //Quantity
    "executedQty": "0", // Executed Quantity
    "avgPrice": "0", // avg price
    "marginLocked": "0.00130552", // The margin locked for this order.
    "type": "LIMIT", 
    "side": "BUY_OPEN", 
    "timeInForce": "GTC", 
    "status": "NEW", 
    "priceType": "INPUT" 
  }
]
```


### Parameters
| Name    | Type  |    Mandatory           | Description           |
| ----------------- | ---- | ------- | ------------- |
| symbol | STRING | NO |  |
| orderId | LONG | NO |  |
| type | ENUM | YES | `LIMIT` or `STOP` |
| limit | INT | NO | |
| timestamp | LONG | YES | |
| recvWindow | LONG | NO | |

Notes：

- If `orderId` is sent, all orders < `orderId` will be returned. If not, the latest outstanding order will be returned.

## Query  Position (USER_DATA)

- `GET /api/v1/futures/positions`

Returns the current position information, this API requires a request signature.

### Weight: 5

> Response

``` json
[
    {
        "symbol": "BTC-SWAP-USDT",
        "side": "LONG",
        "avgPrice": "24000", 
        "position": "10", //Number of open positions (pieces)
        "available": "10", //Quantity that can be closed (pieces)
        "leverage": "2", 
        "lastPrice": "16854.4", 
        "positionValue": "24", //Position value
        "flp": "12077.8", //liquidation price
        "margin": "11.9825", 
        "marginRate": "0.4992", 
        "unrealizedPnL": "0", //The unrealized profit and loss of the current position
        "profitRate": "0", //Profit rate of current position
        "realizedPnL": "-0.018" //Realized profit and loss
    }
]
```
### Parameters
| Name    | Type  |    Mandatory           | Description           |
| ----------------- | ---- | ------- | ------------- |
| symbol | STRING | NO |  |
| side | ENUM | NO | position direction `LONG` or `SHORT` |
| recvWindow | LONG | NO |  |
| timestamp | LONG | YES |  |

Notes：

- If `symbol` is not sent, all contract position information will be returned.
- If `side` is not sent, position information in both directions will be returned.

## Query History Orders (USER_DATA)
- `GET /api/v1/futures/historyOrders`

### Weight：5

> Response：

``` json
[
  {
    "time": "1570760254539", //create tiem
    "updateTime": "1570760254539", 
    "orderId": "469965509788581888", 
    "clientOrderId": "1570760253946",  //User defined order ID
    "symbol": "BTC-PERP-REV", 
    "price": "8502.34", 
    "leverage": "20", 
    "origQty": "222", // quantity
    "executedQty": "0", // Executed Quantity
    "avgPrice": "0", // avg price
    "marginLocked": "0.00130552", // The margin locked for this order.
    "type": "LIMIT", 
    "side": "BUY_OPEN", 
    "timeInForce": "GTC", 
    "status": "CANCELED", 
    "priceType": "INPUT" 
  }
]
```

### Parameters
| Name    | Type  |    Mandatory           | Description           |
| ----------------- | ---- | ------- | ------------- |
| symbol | STRING | NO |  |
| orderId | LONG | NO |  |
| type | ENUM | YES | `LIMIT` or `STOP` |
| limit | INT | NO | |
| timestamp | LONG | YES | |
| recvWindow | LONG | NO | |

Notes：

- If orderId is sent, all orders < orderId will be returned. If not, the latest order will be returned.

## Futures Account Balance (USER_DATA)
- `GET  /api/v1/futures/balance`


### Weight：5

> Response：

``` json

[
     {
        "asset": "USDT", // asset
        "balance": "999999999999.982", // total
        "availableBalance": "1899999999978.4995", //available balance
        "positionMargin": "11.9825", //position Margin
        "orderMargin": "9.5", //order Margin
        "crossUnRealizedPnl": "10.01" //The unrealized profit and loss of cross position
    }
]
```
### Parameters

### Parameters
| Name    | Type  |    Mandatory           | Description           |
| ----------------- | ---- | ------- | ------------- |
| timestamp | LONG | YES | |
| recvWindow | LONG | NO | |


## Modify Isolated Position Margin (TRADE)
- `POST  /api/v1/futures/positionMargin`


### Weight：1

> Response：

``` json
{
    "code":200, 
    "msg":"success", 
    "symbol":"BTC-PERP-REV", 
    "margin":15, 
    "timestamp":1541161088303 
}
```

### Parameters
| Name    | Type  |    Mandatory           | Description           |
| ----------------- | ---- | ------- | ------------- |
| symbol | STRING | YES |  |
| positionSide | ENUM | YES | `LONG` or `SHORT` |
| amount | DECIMAL | YES | Increase (positive value) or decrease (negative value) the amount of margin. Please note that this quantity refers to the underlying pricing asset of the contract (that is, the underlying of the contract settlement) |
| recvWindow | LONG | NO |  |
| timestamp | LONG | YES |  |

## Account Trade List (USER_DATA)
- `GET /api/v1/futures/userTrades`

Get trades for a specific account and symbol.

### Weight：5

> Response:

``` json

[
    {
        "time": "1668425281370", //create time
        "id": "1289239136943831296", //Trade Id
        "orderId": "1289239134670518528", 
        "matchOrderId": "1287169326781135104", //Counterparty order ID
        "symbol": "BTC-SWAP-USDT", 
        "price": "24000",
        "qty": "9", //quantity
        "commissionAsset": "USDT", 
        "commission": "0.0162", 
        "makerRebate": "0", 
        "type": "LIMIT", 
        "side": "BUY_OPEN", //BUY_OPEN、SELL_OPEN、BUY_CLOSE、SELL_CLOSE
        "realizedPnl": "0" 
    }
]
```

### Parameters
| Name    | Type  |    Mandatory           | Description           |
| ----------------- | ---- | ------- | ------------- |
| symbol | STRING | NO |  |
| limit | INT | NO |  |
| fromId | LONG | NO | Start from TradeId (used to query transaction orders) |
| toId | LONG | NO | To the end of TradeId (used to query transaction orders) |
| recvWindow | LONG | NO |  |
| timestamp | LONG | YES |  |

## Query Risk Limit (USER_DATA)
- `GET /api/v1/futures/riskLimit`

To query the risk limit, this API endpoint requires a request signature.

### Weight：5

> Response：

``` json
[
            {
                "riskLimitId": "200000133", 
                "riskLimitAmount": "1000000.0", //risk limit(Maximum position)
                "maintainMargin": "0.005", //maintenance margin rate
                "initialMargin": "0.01", //initial margin rate
                "side": "SELL_OPEN" 
            },
            {
                "riskLimitId": "200000133",
                "riskLimitAmount": "1000000.0",
                "maintainMargin": "0.005",
                "initialMargin": "0.01",
                "side": "BUY_OPEN"
            }
 ]
```
### Parameters
| Name    | Type  |    Mandatory           | Description           |
| ----------------- | ---- | ------- | ------------- |
| symbol | STRING | YES |  |
| recvWindow | LONG | NO |  |
| timestamp | LONG | YES |  |

## Set Risk Limits (USER_DATA)
- `POST /api/v1/futures/setRiskLimit`

### Weight：1

> Response：

``` json
  { 
    "success":  true
  }
```

### Parameters
| Name    | Type  |    Mandatory           | Description           |
| ----------------- | ---- | ------- | ------------- |
| symbol | STRING | YES |  |
| riskLimitId | LONG | YES | |
| isLong | BOOLEAN | YES | true:`LONG`;` false`:SHORT |
| recvWindow | LONG | NO |  |
| timestamp | LONG | YES |  |

## User Commission Rate (USER_DATA)
- `GET /api/v1/futures/commissionRate`

### Weight：5

> Response：

``` json
{
    "openMakerFee": "0.000006", //The commission rate for opening pending orders
    "openTakerFee": "0.0001", //The commission rate for open position taker
    "closeMakerFee": "0.0002", //The commission rate for closing pending orders
    "closeTakerFee": "0.0004" //The commission rate for closing a taker order
}
```

### Parameters
| Name    | Type  |    Mandatory           | Description           |
| ----------------- | ---- | ------- | ------------- |
| symbol | STRING | YES |  |
| recvWindow | LONG | NO |  |
| timestamp | LONG | YES |  |

# User Data Streams

- The base API endpoint is : **https://api.toobit.com**
- A User Data Stream `listenKey` is valid for 60 minutes after creation.
- Doing a PUT on a `listenKey` will extend its validity for 60 minutes.
- Doing a DELETE on a `listenKey` will close the stream and invalidate the listenKey .
- Doing a POST on an account with an active `listenKey` will return the currently active `listenKey` and extend its validity for 60 minutes.
- The base websocket endpoint is: **wss://stream.toobit.com**
- User feeds are accessible via `/api/v1/ws/<listenKey>` (e.g. `wss://#HOST/api/v1/ws/<listenKey>`)
- Each link is valid for no more than 24 hours, please properly handle disconnection and reconnection.
- User feed payloads are not guaranteed to be up during busy times; make sure to order updates with `E`


## Start User Data Stream (USER_STREAM)
- `POST /api/v1/listenKey`

Start a new user data stream. The stream will close after 60 minutes unless a keepalive is sent. If the account has an active `listenKey`, that listenKey will be returned and its validity will be extended for 60 minutes.

### Weight： 1

> Response:

``` json
{
  "listenKey": "1A9LWJjuMwKWYP4QQPw34GRm8gz3x5AephXSuqcDef1RnzoBVhEeGE963CoS1Sgj"
}
```

### Parameters
| Name    | Type  |    Mandatory           | Description           |
| ----------------- | ---- | ------- | ------------- |
| timestamp | LONG | YES |  |
| recvWindow | LONG | NO |  |

## Keepalive User Data Stream (USER_STREAM)
- `PUT /api/v1/listenKey`

Keepalive a user data stream to prevent a time out. User data streams will close after 60 minutes. It's recommended to send a ping about every 60 minutes.

### Weight： 1

> Response:

``` json
{
}
```

### Parameters
| Name    | Type  |    Mandatory           | Description           |
| ----------------- | ---- | ------- | ------------- |
| listenKey | STRING | YES | |
| timestamp | LONG | YES |  |
| recvWindow | LONG | NO |  |

## Close User Data Stream (USER_STREAM)

- `DELETE /api/v1/listenKey`

### Weight： 1

> Response:

``` json
{}
```

### Parameters
| Name    | Type  |    Mandatory           | Description           |
| ----------------- | ---- | ------- | ------------- |
| listenKey | STRING | YES | |
| timestamp | LONG | YES |  |
| recvWindow | LONG | NO |  |

## Event: Balance 

The `event type` of the account update event is fixed to `ACCOUNT_UPDATE`

> Payload

``` json
{
  "e": "ACCOUNT_UPDATE",                // event type
  "E": 1564745798939,                   // event time
  "T": 1564745798938 ,                  // Can trade 
  "W": 1564745798938 ,                  // Can withdraw 
  "D": 1564745798938 ,                  // Can deposit 
  "B": [                        // Balances changed 
    {
      "a": "LTC",               // Asset 
      "f": "17366.18538083",    // Free amount 
      "l": "0.00000000"         // Locked amount 
    }
  ]
}
    
```


- When the account information changes, this event will be pushed:
   - This event will only be pushed when there is a change in account information
   


## Event:  Position Update


> Position Payload

``` json
[
    {
        "e": "outboundContractPositionInfo",                // Event type 
        "E": "1668693440976",             // Event time 
        "A": "1270447370291795457",      // account id
        "s": "BTCUSDT",                   // Symbol 
        "S": "LONG",                      // side 
        "p": "441.0",                     // avg price
        "P": "1291488620385157122",       // total position
        "a": "1000",                      // Available positions
        "f": "1291488620167835136",       // liquidation price
        "m": "18.2",                      // Position Margin
        "r": "44",                        // Realized profit and loss
        "mt": "CROSS"                     // Position type
    }
]
```

- position information: push only when there is a change in the symbol position.
- The field `mt` represents the position type `CROSS` cross margin; `ISOLATED` isolated margin

## Event: Order 

This type of event will be pushed when a new order is created, an order has a new deal, or a new state change.

> Order Payload

``` json

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
  "w": true,                     // Is the order working Stops will have
  "m": false,                    // Is this trade the maker side
  "O": 1499405658657,            // Order creation time 
  "Z": "0.00000000"              // Cumulative quote asset transacted quantity 

```

The average price can be found by dividing `Z` by `z`

### Side

- BUY 
- SELL 

### Order Type

- MARKET 
- LIMIT 
- LIMIT_MAKER  
- STOP Plan entrusted
- STOP_SHORT_PROFIT take profit
- STOP_LONG_PROFIT take profit
- STOP_LONG_LOSS  stop loss
- STOP_SHORT_LOSS stop loss
- LIQUI_IOC_ORDER  liquidation
- LIQUI_ADL_ORDER ADL

### Order Status

- NEW  
- PARTIALLY_FILLED  
- FILLED  
- CANCELED  
- PENDING_CANCEL  
- REJECTED  

### Time in force

- GTC
- IOC
- FOK

### Plan order or stop loss order status

- ORDER_NEW 
- ORDER_FILLED 
- ORDER_REJECTED 
- ORDER_CANCELED 
- ORDER_FAILED 

### Take profit and stop loss type

- STOP_LONG_PROFIT  Long position take profit
- STOP_LONG_LOSS Long position stop loss
- STOP_SHORT_PROFIT Plan entrustment-short position take profit
- STOP_SHORT_LOSS Plan order - short position stop loss




## Event: Trade Update

> Trade Payload

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


