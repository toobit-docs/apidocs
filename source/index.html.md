---
title: U本位合约 API 文档

language_tabs: # must be one of https://git.io/vQNgJ
  - shell

toc_footers:
  - <a href='https://www.toobit.com'>TooBit </a>
includes:

search: true
---


# 更新日志
## 2022-10-16

  文档创建

# 基础信息

## Rest 基本信息

- 接口可能需要用户的 API Key，如何创建API-KEY请参考<a href='https://toobit.zendesk.com/hc/zh-cn/articles/13445077851545-%E5%A6%82%E4%BD%95%E5%88%9B%E5%BB%BA-API-%E5%AF%86%E9%92%A5-'>这里</a>
- 本篇列出REST接口的baseurl **https://api.toobit.com**
- 所有接口的响应都是JSON格式
- 所有时间、时间戳均为UNIX时间，单位为毫秒
- 所有数据类型采用JAVA的数据类型定义

## 访问限制
- 在  `/api/v1/exchangeInfo`的`rateLimits` 数组里包含有REST接口(不限于本篇的REST接口)的访问限制。包括带权重的访问频次限制、下单速率限制
- 违反上述任何一个访问限制都会收到HTTP 429，这是一个警告
- 每条线路有一个weight特性，这个决定了这个请求占用多少容量（比如weight=2说明这个请求占用两个请求的量）。返回数据多的端点或者在多个symbol执行任务的端点可能有更高的weight
- 多次反复违反频率限制和/或者没有在收到429后停止发送请求的用户将会被收到封禁IP（错误码418）
- IP封禁会被跟踪和 调整封禁时长（对于反复违反规定的用户，时间从 2分钟到3天不等）

## 接口鉴权类型
- 每个接口都有自己的鉴权类型，鉴权类型决定了访问时应当进行何种鉴权
- 如果需要 API-key, 应当在HTTP头中以X-BB-APIKEY字段传递
- API-key 与 API-secret 是大小写敏感的
- 可以在网页用户中心修改API-key 所具有的权限，例如读取账户信息、发送交易指令、发送提现指令

| 鉴权类型    | 描述 | 
| --------- | ---- | 
| NONE      | 不需要鉴权的接口 | 
| TRADE     | 需要有效的API-KEY和签名 |
| USER_DATA | 需要有效的API-KEY和签名 |
| USER_STREAM | 需要有效的API-KEY |
| MARKET_DATA|  需要有效的API-KEY |

## 需要签名的接口 (TRADE 与 USER_DATA)
- 调用这些接口时，除了接口本身所需的参数外，还需要传递`signature`即签名参数
- 签名使用`HMAC SHA256`算法. API-KEY所对应的API-Secret作为 `HMAC SHA256` 的密钥，其他所有参数作为`HMAC SHA256`的操作对象，得到的输出即为签名
- 签名大小写不敏感
- 当同时使用query string和request body时，`HMAC SHA256`的输入query string在前，request body在后
### 时间同步安全
- 签名接口均需要传递timestamp参数, 其值应当是请求发送时刻的unix时间戳(毫秒)
- 服务器收到请求时会判断请求中的时间戳，如果是5000毫秒之前发出的，则请求会被认为无效。这个时间窗口值可以通过发送可选参数`recvWindow`来自定义
- 另外，如果服务器计算得出客户端时间戳在服务器时间的‘未来’一秒以上，也会拒绝请求。

``` javascript
逻辑伪代码：
if (timestamp < (serverTime + 1000) && (serverTime - timestamp) <= recvWindow) {
  // process request
} else {
  // reject request
}
```

**关于交易时效性** 互联网状况并不100%可靠，不可完全依赖,因此你的程序本地到币安服务器的时延会有抖动. 这是我们设置recvWindow的目的所在，如果你从事高频交易，对交易时效性有较高的要求，可以灵活设置recvWindow以达到你的要求。
<aside class="notice">
不推荐使用5秒以上的recvWindow
</aside>

### POST /api/v1/futures/order的示例
以下是在linux bash环境下使用 echo openssl 和curl工具实现的一个调用接口下单的示例 apikey、secret仅供示范

| Key    | Value | 
| --------- | ---- | 
| apiKey| SRQGN9M8Sr87nbfKsaSxm33Y6CmGVtUu9Erz73g9vHFNn36VROOKSaWBQ8OSOtSq |
| secretKey| 30lfjDT51iOG1kYZnDoLNynOyMdIcmQyO1XYfxzYOmQfx9tjiI98Pzio4uhZ0Uk2 |

| 参数    | 取值 | 
| --------- | ---- | 
| symbol | BTCUSDT |
| side | SELL |
| type| LIMIT |
| timeInForce| GTC |
| quantity| 1 | 
| price| 4000 |
| recvWindow| 100000 |
| timestamp| 1668481902307|

#### 示例 1: 所有参数通过 query string 发送
``` shell
示例1:
HMAC SHA256 签名:
$ echo -n "symbol=BTCUSDT&side=SELL&type=LIMIT&timeInForce=GTC&quantity=1&price=400&recvWindow=100000&timestamp=1668481902307" | openssl dgst -sha256 -hmac "30lfjDT51iOG1kYZnDoLNynOyMdIcmQyO1XYfxzYOmQfx9tjiI98Pzio4uhZ0Uk2"
(stdin)= 8420e499e71cce4a00946db16543198b6bcae01791bdb75a06b5a7098b156468
```

``` bash
curl 调用:
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



#### 示例 2: 所有参数通过 request body 发送

``` shell
示例2:
HMAC SHA256 签名:
$ echo -n "symbol=BTCUSDT&side=SELL&type=LIMIT&timeInForce=GTC&quantity=1&price=400&recvWindow=100000&timestamp=1668481902307" | openssl dgst -sha256 -hmac "30lfjDT51iOG1kYZnDoLNynOyMdIcmQyO1XYfxzYOmQfx9tjiI98Pzio4uhZ0Uk2"
(stdin)= 8420e499e71cce4a00946db16543198b6bcae01791bdb75a06b5a7098b156468
```

``` bash
curl 调用:
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



#### 示例 3: 混合使用 query string 与 request body

- **queryString**:symbol=BTCUSDT&side=SELL&type=LIMIT&timeInForce=GTC
- **requestBody**:quantity=1&price=400&recvWindow=100000&timestamp=1668481902307

``` shell
示例3:
HMAC SHA256 签名:
$ echo -n "symbol=BTCUSDT&side=SELL&type=LIMIT&timeInForce=GTCquantity=1&price=400&recvWindow=10000000&timestamp=1668481902307" | openssl dgst -sha256 -hmac "30lfjDT51iOG1kYZnDoLNynOyMdIcmQyO1XYfxzYOmQfx9tjiI98Pzio4uhZ0Uk2"
(stdin)= 59ef0b2085ebb99cca5b6445c202d99add17be2d5d1861c0f4aa17bc785ac4d5
```

``` bash
curl 调用:
(HMAC SHA256)
$ curl -H "X-BB-APIKEY: SRQGN9M8Sr87nbfKsaSxm33Y6CmGVtUu9Erz73g9vHFNn36VROOKSaWBQ8OSOtSq" -X POST 'https://api.toobit.com/api/v1/spot/order?symbol=BTCUSDT&side=SELL&type=LIMIT&timeInForce=GTC' -d 'quantity=1&price=400&recvWindow=10000000&timestamp=1668481902307&signature=59ef0b2085ebb99cca5b6445c202d99add17be2d5d1861c0f4aa17bc785ac4d5'

```

注意在例子3里有一点不一样，"GTC"和"quantity=1"之间没有&。

## 公开API参数

### 术语解释

- `base asset` 指一个交易对的交易对象，即写在靠前部分的资产名
- `quote asset` 指一个交易对的定价资产，即写在靠后部分资产名

### 枚举定义

#### 交易对类型:
- FUTURE 期货

#### 资产类型:
- CASH - 现金
- MARGIN - 保证金

#### 订单状态 (status):
- NEW - 新订单，暂无成交
- PARTIALLY_FILLED - 部分成交
- FILLED - 完全成交
- CANCELED - 已取消
- PENDING_CANCEL - 等待取消
- REJECTED - 被拒绝

#### 订单类型 (type):
- LIMIT - 限价单
- MARKET - 市价单
- LIMIT_MAKER - maker限价单
- STOP 计划委托
- STOP_SHORT_PROFIT 止盈
- STOP_LONG_PROFIT 止盈
- STOP_LONG_LOSS  止损
- STOP_SHORT_LOSS 止损
- LIQUI_IOC_ORDER 强平
- LIQUI_ADL_ORDER ADL

### 计划委托 或者 止盈止损订单状态 (status):

- ORDER_NEW 新建委托
- ORDER_FILLED 委托触发成功
- ORDER_REJECTED 委托拒绝
- ORDER_CANCELED 取消委托
- ORDER_FAILED 委托触发失败

### 止盈止损类型(type):

- STOP_LONG_PROFIT  多仓止盈
- STOP_LONG_LOSS 多仓止损
- STOP_SHORT_PROFIT 计划委托-空仓止盈
- STOP_SHORT_LOSS 计划委托-空仓止损

#### 订单方向 (side):
- BUY_OPEN   开多买单开仓买入
- BUY_CLOSE  平空买单平仓买入
- SELL_OPEN   开空卖单开仓卖出
- SELL_CLOSE  平多卖单平仓卖出

#### 价格类型 (priceType):

- INPUT 系统将会用你输入的价格来撮合订单。
- OPPONENT 订单会以对手盘最优价格撮合。
- QUEUE 订单会以相同方向的最优价格撮合。
- OVER 订单会以对手盘的最优价格 + 超价（浮动）撮合。
- MARKET 订单会以 最新成交价 * (1 ± 5%) 撮合。

#### 有效方式(timeInForce):
- GTC - Good Till Cancel 成交为止
- IOC - Immediate or Cancel 无法立即成交(吃单)的部分就撤销
- FOK - Fill or Kill 无法全部立即成交就撤销

#### K线间隔:
m -> 分钟; h -> 小时; d -> 天; w -> 周; M -> 月

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

#### 频率限制类型：
- REQUESTS_WEIGHT
- ORDERS

#### 频率限制区间
- SECOND
- MINUTE
- DAY

# 行情接口

## 测试服务器连通性 PING 
-  `GET /api/v1/ping`

测试能否联通

### 权重: 1

### 参数：NONE

> 响应: 

```json
{}

```

## 获取服务器时间
- `GET /api/v1/time`

获取服务器时间

### 权重: 1

### 参数: NONE

> 响应: 

```json
{
  "serverTime": 1538323200000
}
```

## 获取交易规则和交易对
- `GET /api/v1/exchangeInfo`

获取交易规则和交易对

### 权重：1

### 参数：NONE

> 响应:

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
## 深度信息 

- `GET /quote/v1/depth`

### 权重:

| limit    |  权重  | 
| -------- | ---- | 
| 5, 10, 20, 50, 100 | 1 |
| 500 | 5 |
| 1000 | 10 |

> 响应：

``` json
{
  "b": [
    [
      "3.90000000",   // 价格
      "431.00000000"  // 数量
    ],
    [
      "4.00000000",
      "431.00000000"
    ]
  ],
  "a": [
    [
      "4.00000200",  // 价格
      "12.00000000"  // 数量
    ],
    [
      "5.10000000",
      "28.00000000"
    ]
  ]
}
```

### 参数
| 名称    | 类型  |    是否必须           | 描述           |
| ----------------- | ---- | ------- | ------------- |
| symbol | STRING | YES | 交易对 |
| limit | INT | NO | 默认 100; 最大 100|

注意：如果设置limit=0会返回很多数据。

## 合并深度

- `GET /quote/v1/depth/merged`

合并深度接口。

### 权重：1

> 响应

``` json
{
    "t": 1672035413265,//时间
    "b": [//买入深度高到低
        [
            "16851.95",//价格
            "0.003321"//数量
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
    "a": [//卖出深度低到高
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

### 参数
| 名称    | 类型  |    是否必须           | 描述           |
| ----------------- | ---- | ------- | ------------- |
| symbol | STRING | YES | 交易对 |
| scale | INT |  NO | 档位: `0`,`1`,`2`,`3`,`4`,`5` 例如：0表示1档，1表示2档  |
| limit | INT |  NO | 限制条数 |


## 近期成交

- `GET /quote/v1/trades`

获取当前最新成交（最多60）

### 权重：1

>响应：

``` json
[
  {
    "p": "4.00000100",
    "q": "12.00000000",
    "t": 1499865549590,
    "ibm": true  // 成交方向 isBuyerMaker
  }
]

```

### 参数
| 名称    | 类型  |    是否必须           | 描述           |
| ----------------- | ---- | ------- | ------------- |
| symbol | STRING | YES | 交易对 |
| limit | INT | NO | 默认 60; 最大 60|


## K线数据

- `GET /quote/v1/klines`

symbol的k线/烛线图数据,K线会根据开盘时间而辨别。

### 权重：1

> 响应：

``` json
[
  [
    1499040000000,      // 开盘时间
    "0.01634790",       // 开盘价
    "0.80000000",       // 最高价
    "0.01575800",       // 最低价
    "0.01577100",       // 收盘价
    "148976.11427815",  // 交易量
    1499644799999,      // 收盘时间
    "2434.19055334",    // Quote asset数量
    308,                // 交易次数
    "1756.87402397",    // Taker buy base asset数量
    "28.46694368"       // Taker buy quote asset数量
  ]
]
```

### 参数
| 名称    | 类型  |    是否必须           | 描述           |
| ----------------- | ---- | ------- | ------------- |
| symbol | STRING | YES | 交易对 |
| interval | ENUM | YES | 时间间隔 |
| startTime | LONG | NO | 开始时间 |
| endTime | LONG | NO | 结束时间|
| limit | INT | NO | 默认 100; 最大 1000|

注意: 如果startTime和endTime没有发送，只有最新的K线会被返回。


## 价格指数K线数据 

- `GET /api/quote/v1/index/klines`

获取某个交易对的价格指数K线

> 响应：

``` json
{
    "code": 200,
    "data": [
        {
            "t": 1669155300000,//时间戳
            "s": "ETHUSDT",//币对
            "sn": "ETHUSDT",//币对名称
            "c": "1127.1",//收盘价
            "h": "1130.81",//最高价
            "l": "1126.17",//最低价
            "o": "1130.8",//开盘价
            "v": "0"//成交量
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

### 参数
| 名称    | 类型  |    是否必须           | 描述           |
| ----------------- | ---- | ------- | ------------- |
| symbol | STRING | YES | 交易对 |
| interval | ENUM | YES | 时间间隔 |
| from | LONG | YES | 开始时间 |
| to | LONG | YES | 结束时间|
| limit | INT | NO | 限制条数 |


## 标记价格K线数据

- `GET /api/quote/v1/markPrice/klines`

> 响应：

``` json
{
    "code": 200,
    "data": [
        {
            "symbol": "BTCUSDT",//币对
            "time": 1670157900000,//时间戳
            "low": "16991.14096",//最低价
            "open": "16991.78288",//开盘价
            "high": "16996.30641",//最高价
            "close": "16996.30641",//收盘价
            "volume": "0",//成交量
            "curId": 1670157900000
        }
    ]
}
```

### 参数
| 名称    | 类型  |    是否必须           | 描述           |
| ----------------- | ---- | ------- | ------------- |
| symbol | STRING | YES | 交易对 |
| interval | ENUM | YES | 时间间隔 |
| from | LONG | YES | 开始时间 |
| to | LONG | YES | 结束时间|
| limit | INT | NO | 限制条数 |


## 最新标记价格
- `GET /quote/v1/markPrice`

获取某个交易对的标记价格

> 响应：

``` json
{
  "exchangeId": 301,//交易所ID
  "symbolId": "BTC-SWAP-USDT",//币对
  "price": "17042.54471",//标记价格
  "time": 1670897454000//时间戳
}
```

### 参数
| 名称    | 类型  |    是否必须           | 描述           |
| ----------------- | ---- | ------- | ------------- |
| symbol | STRING | YES | 交易对 |

## 最新资金费率
- `GET /api/v1/futures/fundingRate` 

> 响应：

``` json
[
    {
        "symbol": "BTC-SWAP-USDT", // 交易对
        "lastFundingTime": "1668398400000", //
        "nextFundingTime": "1668427200000", //下次资金费结算时间
        "rate": "0.0018099173553719" //该次结算资金费率。
    }
]
```

### 参数
| 名称    | 类型  |    是否必须           | 描述           |
| ----------------- | ---- | ------- | ------------- |
| symbol | STRING | YES | 交易对 |

## 查询资金费率历史
- `GET /api/v1/futures/historyFundingRate`

> 响应：

``` json
[
  { 
    "id": "3434343434343",
    "symbol": "BTC-PERP-REV", // 交易对
    "fundingTime": "1570708800000", //资金费率结算时间
    "fundingRate": "0.00321" //资金费率
  }
]
```

### 参数
| 名称    | 类型  |    是否必须           | 描述               |
| ----------------- | ---- | ------- |------------------|
| symbol | STRING | YES | 交易对              |
| fromId | LONG | NO | 起始id             |
| endId | LONG | NO | 结束id             |
| limit | INT | NO | 返回条数 默认20 最大1000 |

## 24hr价格变动情况
- `GET /quote/v1/ticker/24hr`

24小时价格变化数据。注意 如果没有发送symbol，会返回很多数据。

### 权重： 如果只有一个symbol为1，不带symbol为40。

> 响应：

``` json
[
    {
        "t": 1538725500422,   // 时间
        "a": "1.10000000",    // 最高卖价
        "b": "1.00000000",    // 最高买价
        "s": "ETHBTC",        // symbol 
        "c": "4.00000200",    // 最新成交价
        "o": "99.00000000",   // 开盘价
        "h": "100.00000000",  // 最高价 
        "l": "0.10000000",    // 最低价
        "v": "8913.30000000", // 成交量
        "qv": "15.30000000"   // 成交额
    }
]
```

### 参数
| 名称    | 类型  |    是否必须           | 描述           |
| ----------------- | ---- | ------- | ------------- |
| symbol | STRING | NO | 交易对 |
| realtimeInterval | ENUM | NO | `value`值如下：`24h`、`1d`、`1d+8`|

- 如果symbol没有被发送，所有symbol的数据都会被返回。


## 最新价格
- `GET /quote/v1/ticker/price`

返回最近价格

### 权重：1

> 响应：

``` json
[
  {
    "s": "LTCBTC",     // 交易对
    "p": "4.00000200"  // 最新价
  }
]
```
### 参数
| 名称    | 类型  |    是否必须           | 描述           |
| ----------------- | ---- | ------- | ------------- |
| symbol | STRING | NO | 交易对 |

- 如果symbol没有发送，所有symbol的最新价都会被返回。

## 当前最优挂单
- `GET /quote/v1/ticker/bookTicker`

单个或者多个symbol的最佳买单卖单价格。

### 权重：1

> 响应：

``` json
[
  {
      "t": 132222222222222,     // 时间
      "s": "LTCBTC",            // 交易对          
      "b": "4.00000000",        // 最高买价
      "bq": "431.00000000",     // 最高买价对应的数量
      "a": "4.00000200",        // 最高卖价
      "aq": "9.00000000"        // 最高卖价对应的数量
  }
]
```

### 参数
| 名称    | 类型  |    是否必须           | 描述           |
| ----------------- | ---- | ------- | ------------- |
| symbol | STRING | NO | 交易对 |

- 如果symbol没有被发送，所有symbol的最佳订单簿价格都会被返回。

# Websocket 行情推送接口

## 实时订阅/取消数据流

- 以下数据可以通过websocket发送以实现订阅或取消订阅数据流。示例如下。
- 直接访问时URL格式为  wss://#HOST/quote/ws/v1


| 名称    | 值  |  
| ----------------- | ---- | 
| topic | `realtimes`(实时行情), `trade`(最新成交), `kline_$interval`(k线), `depth`（深度） | 
| event | `sub`(订阅), `cancel`(取消), `cancel_all`(取消全部)  |
| interval | 1m, 5m, 15m, 30m, 1h, 2h, 6h, 12h, 1d, 1w, 1M |

### 请求订阅数据样例:

`{`

  `"symbol": "$symbol0, $symbol1", //交易所ID+币对`

  `"topic": "$topic",`

  `"event": "sub",`
  
  `  "params": {`

  `      "limit": "$limit",  // kline返回上限是2000，默认为1`

  `      "binary": "false"  // 返回的数据是否是压缩过的，默认为false`

  `  }`

`}`

## 心跳

每隔一段时间，客户端需要发送ping帧，服务端会回复pong帧，否则服务端会在5分钟内主动断开链接。

### 请求

`{`
    `"ping": 1535975085052`
`}`

> 响应：

``` json
{
    "pong": 1535975085052
}
```

## 最新合约价格

逐笔交易推送每一笔成交的信息。成交，或者说交易的定义是仅有一个吃单者与一个挂单者相互交易。<br>
在成功连接到服务器后，服务器首先会推送一条最近的60条成交。在这条推送之后，每条推送都是实时的成交。<br>
变量“v”可以理解成一个交易ID。这个变量是全局递增的并且独特的。<br>
例如：假设过去5秒有3笔交易发生，分别是`ETHUSDT`、`BTCUSDT`、`BHTBTC`。它们的“v”会为连续的值（112，113，114）。

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
            "v": "1291465821801168896", // 参见解释
            "t": 1668690723096, //时间戳
            "p": "399", // 价格
            "q": "1", // 数量
            "m": false // true = 买, false = 卖
        },
        {
            "v": "1291465842546196481",
            "t": 1668690725569,
            "p": "399",
            "q": "1",
            "m": false
        }
    ],
    "f": true, // 是不是第一个返回
    "sendTime": 1668753154192,
    "shared": false
}
```

### 请求订阅数据样例:

`{`

  `"symbol": "$symbol0, $symbol1",//币对`

  `"topic": "trade",`

  `"event": "sub",`

  `"params": {`

  `  "binary": false // Whether data returned is in binary format`

  `}`

`}`



## 最新标记价格

合约标记价格。

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
            "markPrice": "16792.28",//当前指数价
            "formula": "(16792.28[HUOBI])/1",//来源
            "time": 1668754084000
        }
    ],
    "f": false,//是否为第一个返回
    "sendTime": 1668754084738,
    "shared": false
}
```

### 请求订阅数据样例:

`{`

  `"symbol": "$symbol0, $symbol1", //币对`

  `"topic": "markPrice",`

  `"event": "sub"`

`}`




## K线

K线stream逐秒推送所请求的K线种类(最新一根K线)的更新

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
            "t": 1668753840000,//k线开始时间
            "s": "BTCUSDT",// symbol
            "sn": "BTCUSDT",// symbol name
            "c": "445",//收盘价
            "h": "445",//最高价
            "l": "445",//最低价
            "o": "445",//开盘价
            "v": "0"//交易量
        }
    ],
    "f": true,// 是否为第一个返回
    "sendTime": 1668753854576,
    "shared": false
}
```

### 请求订阅数据样例

`{`

  `"symbol": "$symbol0, $symbol1",//币对`

  `"topic": "kline_"+$间隔,`

  `"event": "sub",`

  `"params": {`

  `  "binary": false`

  `}`

`}`


### K线/蜡烛图间隔:

订阅Kline需要提供间隔参数，最短为分钟线，最长为月线。支持以下间隔:
m -> 分钟; h -> 小时; d -> 天; w -> 周; M -> 月

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



## 按Symbol的完整Ticker

按Symbol逐秒刷新的24小时完整ticker信息

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
            "t": 1668753480049, //时间戳
            "s": "BTCUSDT", //symbol
            "sn": "BTCUSDT", // symbol name
            "c": "445", //收盘价
            "h": "445", //最高价
            "l": "310", //最低价
            "o": "311", //开盘价
            "v": "3747.7597191", //交易量
            "qv": "1426443.9553995", //交易额
            "m": "0.4309", // margin
            "e": 301 // 交易id
        }
    ],
    "f": true, // 是否为第一个返回
    "sendTime": 1668753481048,
    "shared": false
}
```

### 请求订阅数据样例

`{`

  `"symbol": "$symbol0, $symbol1",//币对`

  `"topic": "realtimes",`

  `"event": "sub",`

  `"params": {`

  `  "binary": false`

  `}`

`}`




## 有限档深度信息

> Payload

``` json
{
  "symbol": "BTCUSDT",
  "topic": "depth",
  "data": [{
    "s": "BTCUSDT", //Symbol
    "t": 1565600357643, //时间戳
    "v": "112801745_18", //见上面解释
    "b": [ //Bids
      ["11371.49", "0.0014"], //[价格, 数量]
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
    "a":[//Asks
      ["11375.41", "0.0053"], //[价格, 数量]
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
  "f": true//是否为第一个返回
}
```

### 请求订阅数据样例

`{`

  `"symbol": "$symbol0, $symbol1",//币对`

 ` "topic": "depth",`

  `"event": "sub",`

  `"params": {`

   ` "binary": false`

  `  }`

`}`




## 增量深度信息

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
  "f": false //是否为第一个返回值
}
```

### 请求订阅数据样例

`{`

  `"symbol": "$symbol0, $symbol1",//币对`

  `"topic": "diffDepth",`

  `"event": "sub",`

  `"params": {`

  ` "binary": false` 
  `}`

`}`




每秒推送订单簿的变化部分（如果有）。<br>
在增量深度信息中，数量不一定等于对应价格的数量。如果数量=0，这说明在上一条推送中的这个价格已经没有了。如果数量>0，这时的数量为更新后的这个价格所对应的数量<br>
假设我们收到的返回数据中有这样一条：<br>
`["0.00181860", "155.92000000"]// 价格，数量` <br>
如果下一条返回数据中有： <br>
`["0.00181860", "12.3"]` <br>
这说明这个价格对应的数量有变更，已经更新变更的数量<br>
如果下一条返回数据中有：<br>
`["0.00181860", "0"]`<br>
这说明这个价格对应的数量已经消失，将会在客户端中删除。

# 账户和交易接口
<aside class="warning">
 考虑到剧烈行情下, RESTful接口可能存在查询延迟，我们强烈建议您优先从Websocket user data stream推送的消息来获取订单，成交，仓位等信息。
</aside>




## 查询子账户(暂时忽略 2022-01-04给最终版)

- `GET /api/v1/subAccount`


### 权重：5

### 参数

| 名称    | 类型  |    是否必须           | 描述           |
| ----------------- | ---- | ------- | ------------- |
| timestamp | LONG | YES | 时间戳 |
| recvWindow | LONG | NO | recv窗口 |

> 响应：

``` json
[
    {
        "uid":"122216245228131",  // 子账户uid
        "email":"c123456_mo3nXl@spyzn8.com", // 子账户的邮箱
        "createTime":154443332212,  // 子账户创建时间
        "status": 1  // 1:启用 2:禁用
    },
    {
        "uid":"122216245228132",   // 子账户uid
        "email":"c12345_mo3nXl@spyzn8.com",  // 子账户的邮箱
        "createTime":1544433328002, // 子账户创建时间
        "status": 1  // 1:启用 2:禁用
    }
]
```



## 母子账户万能划转

- `POST /api/v1/subAccount/transfer`

改接口支持的划转操作有:
  - 母账户操作 母账户`现货账户`、`U本位合约账户`划转到任意子账户`现货账户`、`U本位合约账户`
  - 母账户操作 母账户`现货账户`、`U本位合约账户`之间的划转
  - 母账户操作 某一个子账户的`现货账户`、`U本位合约账户`之间的划转
  - 子用户操作 当前子账户`现货账户`、`U本位合约账户`到母用户`现货账户`、`U本位合约账户`的划转
  - 子用户操作 当前子用户`现货账户`、`U本位合约账户`之间的划转

### 权重：1

> 响应：

``` json
{
    "code": 200, // 成功
    "msg": "success" // 响应消息
}
```

### 参数
| 名称              | 类型      |    是否必须           | 描述      |
|-----------------|---------| ------- |---------|
| fromUid         | LONG    | YES | 源账户id   |
| toUid           | LONG    | YES | 目标账户id  |
| fromAccountType | String  | YES | 源账户类型   |
| toAccountType   | String    | YES | 目标账户类型  |
| coin            | STRING  | YES | tokenID |
| quantity        | DECIMAL | YES | 转账数量    |
| timestamp       | LONG    | YES | 时间戳     |
| recvWindow      | LONG    | NO | recv窗口  |


accountType：
`MAIN`: 现货
`FUTURES`:  U本位合约



## 获取划转历史
- `GET /api/v1/account/balanceFlow`

获取现货账户与合约账户之间的资金划转历史记录

### 权重：5

> 响应：

``` json
[
    {
        "id": "539870570957903104",
        "accountId": "122216245228131",
        "coin": "BTC",
        "coinId": "BTC",
        "coinName": "BTC",
        "flowTypeValue": 51, // 流水类型
        "flowType": "USER_ACCOUNT_TRANSFER", // 流水类型名称
        "flowName": "Transfer", // 流水类型说明
        "change": "-12.5", // 变动值
        "total": "379.624059937852365", // 变动后当前tokenId总资产
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

### 参数
| 名称    | 类型  |    是否必须           | 描述           |
| ----------------- | ---- | ------- | ------------- |
| accountType | INT | NO | 账户对应的account_type |
| coin | STRING | NO | tokenID |
| flowType | INT | NO | 划转：3 |
| fromId | LONG | NO | 顺向查询数据 |
| endId | LONG | NO | 反向查询数据 |
| startTime | LONG | NO | 开始时间 |
| endTime | LONG | NO | 结束时间 |
| limit | INT | NO | 每页记录数 |
| timestamp | LONG | YES | 时间戳 |
| recvWindow | LONG | NO | recv窗口 |


## 变换逐全仓模式 (TRADE)
- `POST /api/v1/futures/marginType `

变换用户在指定symbol合约上的保证金模式：逐仓或全仓。

### 权重：1

> 响应：

``` json
{
    "code":200, //响应码 200 = 成功
    "symbol":"BTC-SWAP-USDT", //交易对
    "marginType":"CROSS" //开仓类型 CROSS：全仓 ISOLATED：逐仓
}
```

### 参数
| 名称    | 类型  |    是否必须           | 描述           |
| ----------------- | ---- | ------- | ------------- |
| symbol | STRING | YES | 交易对 |
| marginType | ENUM | YES | `CROSS`：全仓 `ISOLATED`：逐仓 |
| timestamp | LONG | YES | 时间戳 |
| recvWindow | LONG | NO | recv窗口 |

## 调整开仓杠杆 (TRADE)

- `POST /api/v1/futures/leverage `

调整用户在指定symbol合约的开仓杠杆。

### 权重：1

>响应:

``` json
{
    "code": 200, //响应码 200 = 成功
    "symbol":"BTC-SWAP-USDT", //交易对
    "leverage":"20" //杠杠倍数
}
```

### 参数
| 名称    | 类型  |    是否必须           | 描述           |
| ----------------- | ---- | ------- | ------------- |
| symbol | STRING | YES | 交易对 |
| leverage | INT | YES | 杠杠倍数 |
| timestamp | LONG | YES | 时间戳 |
| recvWindow | LONG | NO | recv窗口 |


## 查询杠杆倍数和仓位模式(USER_DATA)
- `GET /api/v1/futures/accountLeverage`

获取用户所有合约交易对的杠杆倍数和仓位类型。这个API需要你的请求签名。

> 响应：

``` json
[
    {
        "symbol":"BTC-SWAP-USDT", //交易对
        "leverage":"20", //杠杆倍数
        "marginType":"CROSS" //开仓类型 CROSS：全仓 ISOLATED：逐仓
    }
]
```

### 参数
| 名称    | 类型  |    是否必须           | 描述           |
| ----------------- | ---- | ------- | ------------- |
| symbol | STRING | YES | 交易对 |
| timestamp | LONG | YES | 时间戳 |
| recvWindow | LONG | NO | recv窗口 |


## 下单 (TRADE)
`POST /api/v1/futures/order`

> 响应：

``` json
{
    "time": "1668418485058", // 订单生成时的时间戳
    "updateTime": "1668418485058", // 订单上次更新的时间戳
    "orderId": "1289182123551455488", //订单ID
    "clientOrderId": "test1115", //用户定义的订单ID
    "symbol": "BTC-SWAP-USDT", //交易对
    "price": "19000", //订单价格
    "leverage": "2", //订单杠杆
    "origQty": "10", //订单数量
    "executedQty": "0", //订单已执行数量
    "avgPrice": "0", //平均交易价格
    "marginLocked": "9.5", //该订单锁定的保证金。这包括实际需要的保证金外加开仓和平仓所需的费用。
    "type": "LIMIT", // 订单类型（LIMIT和STOP）
    "side": "BUY_OPEN", // 订单方向（BUY_OPEN、SELL_OPEN、BUY_CLOSE、SELL_CLOSE）
    "timeInForce": "GTC", // 时效单（Time in Force)类型(GTC、FOK、IOC、LIMIT_MAKER)
    "status": "NEW", //订单状态（NEW、PARTIALLY_FILLED、FILLED、CANCELED、REJECTED）
    "priceType": "INPUT" //价格类型（INPUT、OPPONENT、QUEUE、OVER、MARKET）
}
```

### 参数
| 名称    | 类型  |    是否必须           | 描述           |
| ----------------- | ---- | ------- | ------------- |
| symbol | STRING | YES | 交易对 |
| side | ENUM | YES | 下单方向，方向类型为 `BUY_OPEN`、`SELL_OPEN`、`BUY_CLOSE`、`SELL_CLOSE`|
| type | ENUM | YES | 订单类型，支持订单类型为 `LIMIT`和`STOP` |
| quantity | LONG | YES | 订单的合约数量（张） |
| price | DECIMAL | NO | 订单价格 (`LIMIT`&`INPUT`)订单 **强制需要** |
| priceType | ENUM | NO | 价格类型，支持的价格类型为 `INPUT`、`OPPONENT`、`QUEUE`、`OVER`、`MARKET` |
| stopPrice | DECIMAL | NO | 计划委托的触发价格。`type` = `STOP`订单 **强制需要** |
| timeInForce | ENUM | NO | `LIMIT`订单的时间指令（Time in Force），目前支持的类型为`GTC`、`FOK`、`IOC`、`LIMIT_MAKER` |
| newClientOrderId | STRING | YES | 订单的ID，用户自己定义 |
| timestamp | LONG | YES | 时间戳 |
| recvWindow | LONG | NO | recv窗口 |

### 订单方向 (side):
- BUY_OPEN   开多买单开仓买入
- BUY_CLOSE  平空买单平仓买入
- SELL_OPEN   开空卖单开仓卖出
- SELL_CLOSE  平多卖单平仓卖出

### 价格类型 (priceType):

- INPUT 系统将会用你输入的价格来撮合订单。
- OPPONENT 订单会以对手盘最优价格撮合。
- QUEUE 订单会以相同方向的最优价格撮合。
- OVER 订单会以对手盘的最优价格 + 超价（浮动）撮合。
- MARKET 订单会以 最新成交价 * (1 ± 5%) 撮合。

## 查询订单 (USER_DATA)
- `GET /api/v1/futures/order`

### 权重：1

> 响应：

``` json
{
    "time": "1668418485058", // 订单生成时的时间戳
    "updateTime": "1668418485058", // 订单上次更新的时间戳
    "orderId": "1289182123551455488", //订单ID
    "clientOrderId": "test1115", //用户定义的订单ID
    "symbol": "BTC-SWAP-USDT", //交易对
    "price": "19000", //订单价格
    "leverage": "2", //订单杠杆
    "origQty": "10", //订单数量
    "executedQty": "0", //订单已执行数量
    "avgPrice": "0", //平均交易价格
    "marginLocked": "9.5", //该订单锁定的保证金。这包括实际需要的保证金外加开仓和平仓所需的费用。
    "type": "LIMIT", // 订单类型（LIMIT和STOP）
    "side": "BUY_OPEN", // 订单方向（BUY_OPEN、SELL_OPEN、BUY_CLOSE、SELL_CLOSE）
    "timeInForce": "GTC", // 时效单（Time in Force)类型(GTC、FOK、IOC、LIMIT_MAKER)
    "status": "NEW", //订单状态（NEW、PARTIALLY_FILLED、FILLED、CANCELED、REJECTED）
    "priceType": "INPUT" //价格类型（INPUT、OPPONENT、QUEUE、OVER、MARKET）
}
```

### 参数
| 名称    | 类型  |    是否必须           | 描述           |
| ----------------- | ---- | ------- | ------------- |
| orderId | LONG | NO | 订单ID |
| origClientOrderId | STRING | NO | 用户定义的订单ID |
| type | ENUM | NO | 订单类型（LIMIT和STOP） |
| timestamp | LONG | YES | 时间戳 |
| recvWindow | LONG | NO | recv窗口 |
 
注意: 

- `orderId` 或者 `origClientOrderId` 必须发送其中之一

## 撤销订单 (TRADE)
- `DELETE /api/v1/futures/order`

### 权重：1

> 响应：

``` json

{
    "time": "1668418485058", // 订单生成时的时间戳
    "updateTime": "1668418485058", // 订单上次更新的时间戳
    "orderId": "1289182123551455488", //订单ID
    "clientOrderId": "test1115", //用户定义的订单ID
    "symbol": "BTC-SWAP-USDT", //交易对
    "price": "19000", //订单价格
    "leverage": "2", //订单杠杆
    "origQty": "10", //订单数量
    "executedQty": "0", //订单已执行数量
    "avgPrice": "0", //平均交易价格
    "marginLocked": "9.5", //该订单锁定的保证金。这包括实际需要的保证金外加开仓和平仓所需的费用。
    "type": "LIMIT", // 订单类型（LIMIT和STOP）
    "side": "BUY_OPEN", // 订单方向（BUY_OPEN、SELL_OPEN、BUY_CLOSE、SELL_CLOSE）
    "timeInForce": "GTC", // 时效单（Time in Force)类型(GTC、FOK、IOC、LIMIT_MAKER)
    "status": "NEW", //订单状态（NEW、PARTIALLY_FILLED、FILLED、CANCELED、REJECTED）
    "priceType": "INPUT" //价格类型（INPUT、OPPONENT、QUEUE、OVER、MARKET）
}


```

### 参数
| 名称    | 类型  |    是否必须           | 描述           |
| ----------------- | ---- | ------- | ------------- |
| orderId | LONG | NO | 订单ID |
| origClientOrderId | STRING | NO | 用户定义的订单ID |
| type | ENUM | NO | 订单类型（LIMIT和STOP） |
| symbol | STRING | NO | 交易对 |
| timestamp | LONG | YES | 时间戳 |
| recvWindow | LONG | NO | recv窗口 |

注意：

- `orderId` 或者 `origClientOrderId` 必须发送其中之一

## 撤销全部订单 (TRADE)

- `DELETE /api/v1/futures/batchOrders`

### 权重：5

> 响应:

``` json
{
  "message":"success",
  "timestamp":1541161088303
}
```

### 参数

| 名称    | 类型  |    是否必须           | 描述           |
| ----------------- | ---- | ------- | ------------- |
| symbol | STRING | YES | 交易对 （或者用,分割的交易对的list）|
| side | ENUM | NO | `BUY`或`SELL` |
| timestamp | LONG | YES | 时间戳 |
| recvWindow | LONG | NO | recv窗口 |

## 查看当前全部挂单 (USER_DATA)
- `GET /api/v1/futures/openOrders`

### 权重：1

> 响应：

``` json
[
  {
    "time": "1570760254539", //订单生成时的时间戳
    "updateTime": "1570760254539", //订单上次更新的时间戳
    "orderId": "469965509788581888", // 订单ID
    "clientOrderId": "1570760253946",  //用户定义的订单ID
    "symbol": "BTC-PERP-REV", // 交易对
    "price": "8502.34", // 订单价格
    "leverage": "20", //订单杠杆
    "origQty": "222", // 订单数量
    "executedQty": "0", // 订单已执行数量
    "avgPrice": "0", // 平均交易价格
    "marginLocked": "0.00130552", // 该订单锁定的保证金。这包括实际需要的保证金外加开仓和平仓所需的费用。
    "type": "LIMIT", // 订单类型（LIMIT和STOP）
    "side": "BUY_OPEN", // 订单方向（BUY_OPEN、SELL_OPEN、BUY_CLOSE、SELL_CLOSE）
    "timeInForce": "GTC", //时效单（Time in Force)类型(GTC、FOK、IOC、LIMIT_MAKER)
    "status": "NEW", // 订单状态（NEW、PARTIALLY_FILLED、FILLED、CANCELED、REJECTED）
    "priceType": "INPUT" //价格类型（INPUT、OPPONENT、QUEUE、OVER、MARKET）
  }
]
```


### 参数
| 名称    | 类型  |    是否必须           | 描述                   |
| ----------------- | ---- | ------- |----------------------|
| symbol | STRING | NO | 交易对                  |
| orderId | LONG | NO | 订单ID                 |
| type | ENUM | YES | 订单类型（`LIMIT`、`STOP`） |
| limit | INT | NO | 默认20 最大1000          |
| timestamp | LONG | YES | 时间戳                  |
| recvWindow | LONG | NO | recv窗口               |

注意：

- 如果发送了`orderId`，会返回所有< `orderId`的订单。如果没有则会返回最新的未完成订单。

## 查询当前持仓 (USER_DATA)

- `GET /api/v1/futures/positions`

返回现在的仓位信息，这个API需要请求签名。

### 权重：5

> 响应

``` json
[
    {
        "symbol": "BTC-SWAP-USDT",
        "side": "LONG",//仓位方向
        "avgPrice": "24000", //平均开仓价格
        "position": "10", //开仓数量（张）
        "available": "10", //可平仓数量（张）
        "leverage": "2", //仓位现在杠杆
        "lastPrice": "16854.4", //合约最新市场成交价
        "positionValue": "24", //仓位价值
        "flp": "12077.8", //强制平仓价格
        "margin": "11.9825", //仓位保证金
        "marginRate": "0.4992", //当前仓位的保证金率
        "unrealizedPnL": "0", //当前仓位的未实现盈亏
        "profitRate": "0", //当前仓位的盈利率
        "realizedPnL": "-0.018" //当前 合约 的已实现盈亏
    }
]
```
### 参数
| 名称    | 类型  |    是否必须           | 描述           |
| ----------------- | ---- | ------- | ------------- |
| symbol | STRING | NO | 交易对 |
| side | ENUM | NO | 仓位方向，`LONG`（多仓）或者`SHORT`（空仓）。 |
| timestamp | LONG | YES | 时间戳 |
| recvWindow | LONG | NO | recv窗口 |

注意：

- 如果`symbol`没有发送，所有的合约仓位信息都会被返回。
- 如果`side`没有发送，两个方向的仓位信息都会被返回。

## 查询历史订单 (USER_DATA)
- `GET /api/v1/futures/historyOrders`

### 权重：5

> 响应：

``` json
[
  {
    "time": "1570760254539", //订单生成时的时间戳
    "updateTime": "1570760254539", //订单上次更新的时间戳
    "orderId": "469965509788581888", // 订单ID
    "clientOrderId": "1570760253946",  //用户定义的订单ID
    "symbol": "BTC-PERP-REV", // 交易对
    "price": "8502.34", // 订单价格
    "leverage": "20", //订单杠杆
    "origQty": "222", // 订单数量
    "executedQty": "0", // 订单已执行数量
    "avgPrice": "0", // 平均交易价格
    "marginLocked": "0.00130552", // 该订单锁定的保证金。这包括实际需要的保证金外加开仓和平仓所需的费用。
    "type": "LIMIT", // 订单类型（LIMIT和STOP）
    "side": "BUY_OPEN", // 订单方向（BUY_OPEN、SELL_OPEN、BUY_CLOSE、SELL_CLOSE）
    "timeInForce": "GTC", //时效单（Time in Force)类型(GTC、FOK、IOC、LIMIT_MAKER)
    "status": "CANCELED", // 订单状态（NEW、PARTIALLY_FILLED、FILLED、CANCELED、REJECTED）
    "priceType": "INPUT" //价格类型（INPUT、OPPONENT、QUEUE、OVER、MARKET）
  }
]
```

### 参数
| 名称    | 类型  |    是否必须           | 描述           |
| ----------------- | ---- | ------- | ------------- |
| symbol | STRING | NO | 交易对 |
| orderId | LONG | NO | 订单ID |
| type | ENUM | YES | 订单类型（`LIMIT`、`STOP`） |
| limit | INT | NO | |
| timestamp | LONG | YES | 时间戳 |
| recvWindow | LONG | NO | recv窗口 |

注意：

- 如果发送了orderId，会返回所有< orderId的订单。如果没有则会返回最新的订单。

## 账户余额 (USER_DATA)
- `GET  /api/v1/futures/balance`

返回合约账户余额，这个端点需要请求签名。

### 权重：5

> 响应：

``` json

[
     {
        "asset": "USDT", //资产
        "balance": "999999999999.982", //总余额
        "availableBalance": "1899999999978.4995", //可用保证金，包含未实现盈亏
        "positionMargin": "11.9825", //仓位保证金
        "orderMargin": "9.5" ,//委托保证金（下单锁定）
        "crossUnRealizedPnl": "10.01" //全仓未实现盈亏
    }
]
```
### 参数
| 名称    | 类型  |    是否必须           | 描述           |
| ----------------- | ---- | ------- | ------------- |
| timestamp | LONG | YES | 时间戳 |
| recvWindow | LONG | NO | recv窗口 |

## 调整逐仓保证金 (TRADE)
- `POST  /api/v1/futures/positionMargin`

针对逐仓模式下的仓位，调整其逐仓保证金资金。

### 权重：1

> 响应：

``` json
{
    "code":200, //响应码 200 = 成功
    "msg":"success", //响应消息
    "symbol":"BTC-PERP-REV", //交易对
    "margin":15, //更新后的仓位保证金
    "timestamp":1541161088303 //更新时间戳
}
```

### 参数
| 名称    | 类型  |    是否必须           | 描述           |
| ----------------- | ---- | ------- | ------------- |
| symbol | STRING | YES | 交易对 |
| side | ENUM | YES | 仓位方向，`LONG`（多仓）或者`SHORT`（空仓） |
| amount | DECIMAL | YES | 增加（正值）或者减少（负值）保证金的数量。请注意这个数量指的是合约标的定价资产（即合约结算的标的） |
| timestamp | LONG | YES | 时间戳 |
| recvWindow | LONG | NO | recv窗口 |

## 账户成交历史 (USER_DATA)
- `GET /api/v1/futures/userTrades`

获取某交易对的成交历史

### 权重：5

> 响应

``` json

[
    {
        "time": "1668425281370", //订单生成是的时间戳
        "id": "1289239136943831296", //成交ID
        "orderId": "1289239134670518528", //订单ID
        "matchOrderId": "1287169326781135104", //成交对手订单ID
        "symbol": "BTC-SWAP-USDT", //交易对
        "price": "24000",//成交价格
        "qty": "9", //成交数量
        "commissionAsset": "USDT", //手续费类型（Token名称）
        "commission": "0.0162", //实际手续费
        "makerRebate": "0", // 负maker返佣
        "type": "LIMIT", //订单类型（LIMIT、MARKET)
        "side": "BUY_OPEN", //订单方向（BUY_OPEN、SELL_OPEN、BUY_CLOSE、SELL_CLOSE）
        "realizedPnl": "0" //成交盈亏
    }
]
```

### 参数
| 名称    | 类型  |    是否必须           | 描述           |
| ----------------- | ---- | ------- | ------------- |
| symbol | STRING | NO | 交易对 |
| limit | INT | NO | 返回限制(最大值为1000) |
| fromId | LONG | NO | 从TradeId开始（用来查询成交订单） |
| toId | LONG | NO | 到TradeId结束（用来查询成交订单） |
| timestamp | LONG | YES | 时间戳 |
| recvWindow | LONG | NO | recv窗口 |

## 风险限额查询 (USER_DATA)
- `GET /api/v1/futures/riskLimit`

查询风险限额，这个API端点需要请求签名。

### 权重：5

> 响应：

``` json
[
            {
                "riskLimitId": "200000133", //风险限额id
                "riskLimitAmount": "1000000.0", //风险限额(最大持仓量)
                "maintainMargin": "0.005", //维持保证金率
                "initialMargin": "0.01", //起始保证金率
                "side": "SELL_OPEN" //订单方向（BUY_OPEN、SELL_OPEN、BUY_CLOSE、SELL_CLOSE）
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
### 参数
| 名称    | 类型  |    是否必须           | 描述           |
| ----------------- | ---- | ------- | ------------- |
| symbol | STRING | YES | 交易对 |
| timestamp | LONG | YES | 时间戳 |
| recvWindow | LONG | NO | recv窗口 |

## 设置风险限额(USER_DATA)
- `POST /api/v1/futures/setRiskLimit`

### 权重：1

> 响应：

``` json
  { 
    "success":  true
  }
```

### 参数
| 名称    | 类型  |    是否必须           | 描述           |
| ----------------- | ---- | ------- | ------------- |
| symbol | STRING | YES | 交易对 |
| riskLimitId | LONG | YES | |
| isLong | BOOLEAN | YES | true:LONG（多仓） false:SHORT（空仓） |
| timestamp | LONG | YES | 时间戳 |
| recvWindow | LONG | NO | recv窗口 |

## 用户手续费率 (USER_DATA)
- `GET /api/v1/futures/commissionRate`

### 权重：5

> 响应：

``` json
{
    "openMakerFee": "0.000006", //开仓挂单的手续费费率
    "openTakerFee": "0.0001", //开仓吃单的手续费费率
    "closeMakerFee": "0.0002", //平仓挂单的手续费费率
    "closeTakerFee": "0.0004" //平仓吃单的手续费费率
}
```

### 参数
| 名称    | 类型  |    是否必须           | 描述           |
| ----------------- | ---- | ------- | ------------- |
| symbol | STRING | YES | 交易对 |
| timestamp | LONG | YES | 时间戳 |
| recvWindow | LONG | NO | recv窗口 |

# Websocket 账户信息推送

公共WSS说明

- 对listenKey执行PUT将使其有效期延长60分钟。
- 对listenKey执行DELETE将关闭流。
- 用户信息流可通过 `/api/v1/ws/<listenKey>`访问   (例如`wss://#HOST/api/v1/ws/<listenKey>`)
- 到api端点的单个连接仅在24小时内有效；预计在24小时时断开
- 用户信息流有效负载不保证在繁忙时段处于正常状态；确保使用E订购更新

## 生成listenKey (USER_STREAM)
- `POST /api/v1/listenKey`

创建一个新的user data stream，返回值为一个listenKey，即websocket订阅的stream名称。

### 权重： 1

> 响应

``` json
{
  "listenKey": "1A9LWJjuMwKWYP4QQPw34GRm8gz3x5AephXSuqcDef1RnzoBVhEeGE963CoS1Sgj"
}
```

### 参数
| 名称    | 类型  |    是否必须           | 描述           |
| ----------------- | ---- | ------- | ------------- |
| timestamp | LONG | YES | 时间戳 |
| recvWindow | LONG | NO | recv窗口 |

## 延长listenKey有效期 (USER_STREAM)
- `PUT /api/v1/listenKey`

有效期延长至本次调用后60分钟

### 权重： 1

> 响应

``` json
{
}
```

### 参数
| 名称    | 类型  |    是否必须           | 描述           |
| ----------------- | ---- | ------- | ------------- |
| listenKey | STRING | YES | |
| timestamp | LONG | YES | 时间戳 |
| recvWindow | LONG | NO | recv窗口 |

## 关闭listenKey (USER_STREAM)

- `DELETE /api/v1/listenKey`

### 权重： 1

> 响应

``` json
{
}
```

### 参数
| 名称    | 类型  |    是否必须           | 描述           |
| ----------------- | ---- | ------- | ------------- |
| listenKey | STRING | YES | |
| timestamp | LONG | YES | 时间戳 |
| recvWindow | LONG | NO | recv窗口 |

## Balance推送

账户更新事件的 `event type` 固定为 `outboundContractAccountInfo`

> Balance Payload

``` json
{
  "e": "outboundContractAccountInfo",                // 事件类型
  "E": 1564745798939,                   // 事件时间
  "T": true ,                  // Can trade 可否交易
  "W": true ,                  // Can withdraw 可否提币
  "D": true ,                  // Can deposit 可否充币
  "B": [                        // Balances changed 余额变更
    {
      "a": "LTC",               // Asset 资产
      "f": "17366.18538083",    // Free amount 可用金额,不包含未实现盈亏, 用未实现盈亏下单是会未负数
      "l": "0.00000000"         // 冻结金额：仓位保证金 + 委托保证金(下单锁定)
    }
  ]
}
    
```

- 当账户信息有变动时，会推送此事件：
  - 仅当账户信息有变动时，才会推送此事件
 

## Position更新推送

> Payload

``` json
[
    {
        "e": "outboundContractPositionInfo",                // Event type 事件类型
        "E": "1668693440976",             // Event time 事件时间
        "A": "1270447370291795457",      // accountId 账户ID
        "s": "BTCUSDT",                   // Symbol 币对
        "S": "LONG",                      // side 多空方向
        "p": "441.0",                     // avgPrice 持仓平均 价格
        "P": "1291488620385157122",       // position 总仓位 (张)
        "a": "1000",                      // available 可用仓位 (张)
        "f": "1291488620167835136",       // flp 强平价格
        "m": "18.2",                      // margin 仓位保证金
        "r": "44",                        // realizedPnL已实现盈亏
        "mt": "CROSS",                    // marginType仓位类型
        "rr": "89",                       // riskRate 账户风险率 达到100触发强平
        "up": "12",                      // unrealizedPnL 未实现盈亏
        "pr": "0.003",                  // profitRate 当前仓位盈利率
        "pv": "123"                     // positionValue 仓位价值(USDT)
    }
]
```

- position 信息：仅当symbol仓位有变动时推送。
- 字段 `mt` 代表仓位类型 `CROSS` 全仓; `ISOLATED` 逐仓


## 订单更新推送

当有新订单创建、订单有新成交或者新的状态变化时会推送此类事件

> Payload

``` json

{
  "e": "contractExecutionReport",        // Event type 事件类型
  "E": 1499405658658,            // Event time 事件时间
  "s": "ETHBTC",                 // Symbol 币对
  "c": 1000087761,               // Client order ID 客户订单id
  "S": "BUY",                    // Side 订单方向
  "o": "LIMIT",                  // type 订单类型
  "f": "GTC",                    // Time in force 有效方式
  "q": "1.00000000",             // origQty 数量
  "p": "0.10264410",             // price 价格
  "X": "NEW",                    // status 订单状态
  "i": 4293153,                  // orderId 订单id
  "l": "0.00000000",             // Last executed quantity 上次数量
  "z": "0.00000000",             // executedQty 交易数量
  "L": "0.00000000",             // Last executed price 上次价格
  "n": "0",                      // Commission amount 佣金
  "N": null,                     // Commission asset 佣金资产
  "u": true,                     // Is the trade normal, ignore for now 是否正常
  "w": true,                     // Is the order working Stops will have
  "m": false,                    // Is this trade the maker side
  "O": 1499405658657,            // Order creation time 创建时间
  "Z": "0.00000000",              // Cumulative quote asset transacted quantity 交易金额
  "la": "20",                     // leverage 杠杆倍数
  "mt": "CROSS"                     // marginType 订单类型
}
```


- 字段 `mt` 代表仓位类型 `CROSS` 全仓; `ISOLATED` 逐仓
- 平均价格可以通过`Z`除以`z`来获得。


### 订单方向

- BUY 买入
- SELL 卖出

### 订单类型

- MARKET 市价单
- LIMIT 限价单
- LIMIT_MAKER  maker限价单
- STOP 计划委托
- STOP_SHORT_PROFIT 止盈
- STOP_LONG_PROFIT 止盈
- STOP_LONG_LOSS  止损
- STOP_SHORT_LOSS 止损
- LIQUI_IOC_ORDER 强平
- LIQUI_ADL_ORDER ADL

### 订单状态

- NEW  新订单，暂无成交
- PARTIALLY_FILLED  部分成交
- FILLED  完全成交
- CANCELED  已取消
- PENDING_CANCEL  等待取消
- REJECTED  被拒绝

### 有效方式

- GTC
- IOC
- FOK

### 计划委托 或者 止盈止损订单状态

- ORDER_NEW 新建委托
- ORDER_FILLED 委托触发成功
- ORDER_REJECTED 委托拒绝
- ORDER_CANCELED 取消委托
- ORDER_FAILED 委托触发失败

### 止盈止损类型

- STOP_LONG_PROFIT  多仓止盈
- STOP_LONG_LOSS 多仓止损
- STOP_SHORT_PROFIT 计划委托-空仓止盈
- STOP_SHORT_LOSS 计划委托-空仓止损


## 交易推送

自己有成交时会推送

> Payload

``` json
[
    {
        "e": "ticketInfo",                // Event type 事件类型
        "E": "1668693440976",             // Event time 事件时间
        "s": "BTCUSDT",                   // Symbol 币对
        "q": "0.205",                     // quantity 数量
        "t": "1668693440899",             // time 时间
        "p": "441.0",                     // price 价格
        "T": "1291488620385157122",       // ticketId
        "o": "1291488620167835136",       // orderId 订单id
        "c": "1668693440093",             // clientOrderId 客户订单id
        "O": "1291354087841869312",       // matchOrderId 对手方订单ID
        "a": "1286424214388204801",       // accountId 账户id
        "A": "1270447370291795457",       // matchAccountId 对手方账户ID
        "m": false,                       // isMaker 
        "S": "SELL"                       // side  
    }
]
```


