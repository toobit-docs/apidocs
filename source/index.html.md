---
title: U本位合约 API 文档

language_tabs: # must be one of https://git.io/vQNgJ
  - shell

toc_footers:
  - <a href='https://baidu.com'>baidu </a>
includes:

search: true
---


# 更新日志
## 2022-12-16
- 接口

# 基础信息

## Rest 基本信息

- 接口可能需要用户的 API Key，如何创建API-KEY请参考<a href=''>这里</a>
- 本篇列出REST接口的baseurl **https://fapi.**
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

### POST /api/v1/spot/order的示例
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
$ curl -H "X-BB-APIKEY: SRQGN9M8Sr87nbfKsaSxm33Y6CmGVtUu9Erz73g9vHFNn36VROOKSaWBQ8OSOtSq" -X POST 'https://openapi.wcsbapp.com/api/v1/spot/order' -d 'symbol=BTCUSDT&side=SELL&type=LIMIT&timeInForce=GTC&quantity=1&price=400&recvWindow=100000&timestamp=1668481902307&signature=8420e499e71cce4a00946db16543198b6bcae01791bdb75a06b5a7098b156468'

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
$ curl -H "X-BB-APIKEY: SRQGN9M8Sr87nbfKsaSxm33Y6CmGVtUu9Erz73g9vHFNn36VROOKSaWBQ8OSOtSq" -X POST 'https://openapi.wcsbapp.com/api/v1/spot/order' -d 'symbol=BTCUSDT&side=SELL&type=LIMIT&timeInForce=GTC&quantity=1&price=400&recvWindow=100000&timestamp=1668481902307&signature=8420e499e71cce4a00946db16543198b6bcae01791bdb75a06b5a7098b156468'
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
$ curl -H "X-BB-APIKEY: SRQGN9M8Sr87nbfKsaSxm33Y6CmGVtUu9Erz73g9vHFNn36VROOKSaWBQ8OSOtSq" -X POST 'https://openapi.wcsbapp.com/api/v1/spot/order?symbol=BTCUSDT&side=SELL&type=LIMIT&timeInForce=GTC' -d 'quantity=1&price=400&recvWindow=10000000&timestamp=1668481902307&signature=59ef0b2085ebb99cca5b6445c202d99add17be2d5d1861c0f4aa17bc785ac4d5'

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
#### 订单状态:
- NEW - 新订单，暂无成交
- PARTIALLY_FILLED - 部分成交
- FILLED - 完全成交
- CANCELED - 已取消
- PENDING_CANCEL - 等待取消
- REJECTED - 被拒绝

#### 订单类型:
- LIMIT - 限价单
- MARKET - 市价单
- LIMIT_MAKER - maker限价单
- STOP_LOSS (unavailable now) - 暂无
- STOP_LOSS_LIMIT (unavailable now) - 暂无
- TAKE_PROFIT (unavailable now) - 暂无
- TAKE_PROFIT_LIMIT (unavailable now) - 暂无
- MARKET_OF_PAYOUT (unavailable now) - 暂无
#### 订单方向:
- BUY - 买单
- SELL - 卖单
#### 有效方式:
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
- 3d
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





# Websocket 行情推送接口



# 账户和交易接口
<aside class="warning">
 考虑到剧烈行情下, RESTful接口可能存在查询延迟，我们强烈建议您优先从Websocket user data stream推送的消息来获取订单，成交，仓位等信息。
</aside>

## 划转
执行现货账户与合约账户之间的划转, <a href="#">详情请见这里</a>

## 获取划转历史
获取现货账户与合约账户之间的资金划转历史记录,<a href="#">详情请见这里</a>

## 更改持仓模式(TRADE)

## 查询持仓模式(USER_DATA)

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
| quantity | DECIMAL | YES | 订单的合约数量 |
| price | DECIMAL | NO | 订单价格 (`LIMIT`&`INPUT`)订单 **强制需要** |
| priceType | ENUM | NO | 价格类型，支持的价格类型为 `INPUT`、`OPPONENT`、`QUEUE`、`OVER`、`MARKET` |
| stopPrice | DECIMAL | NO | 计划委托的触发价格。`type` = `STOP`订单 **强制需要** |
| timeInForce | ENUM | NO | `LIMIT`订单的时间指令（Time in Force），目前支持的类型为`GTC`、`FOK`、`IOC`、`LIMIT_MAKER` |
| newClientOrderId | STRING | YES | 订单的ID，用户自己定义 |
| timestamp | LONG | YES | |
| recvWindow | LONG | NO | |



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
| timestamp | LONG | YES | |
| recvWindow | LONG | NO | |
 
注意: 

- `orderId` 或者 `origClientOrderId` 必须发送其中之一

## 撤销订单 (TRADE)
- `DELETE /api/v1/futures/order`

### 权重：1

> 响应：

``` json
同步撤单响应：

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

异步撤单响应：

{
  "isCancelled":true  //订单是否还在book上存在，如果本来就不在book上 返回true。所以除非连接match 超时，其他情况返回均为true
}

```

### 参数
| 名称    | 类型  |    是否必须           | 描述           |
| ----------------- | ---- | ------- | ------------- |
| orderId | LONG | NO | 订单ID |
| origClientOrderId | STRING | NO | 用户定义的订单ID |
| type | ENUM | NO | 订单类型（LIMIT和STOP） |
| fastCancel | INT | NO | 默认`0`(同步撤单)，`1`异步撤单 |
| symbol | STRING | NO | 交易对 |
| timestamp | LONG | YES | |
| recvWindow | LONG | NO | |

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
| timestamp | LONG | YES | |
| recvWindow | LONG | NO | |

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
| 名称    | 类型  |    是否必须           | 描述           |
| ----------------- | ---- | ------- | ------------- |
| symbol | STRING | NO | 交易对 |
| orderId | LONG | NO | 订单ID |
| type | ENUM | YES | 订单类型（`LIMIT`、`STOP`） |
| limit | INT | NO | |
| timestamp | LONG | YES | |
| recvWindow | LONG | NO | |

注意：

- 如果发送了`orderId`，会返回所有< `orderId`的订单。如果没有则会返回最新的未完成订单。

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
| timestamp | LONG | YES | |
| recvWindow | LONG | NO | |

注意：

- 如果发送了orderId，会返回所有< orderId的订单。如果没有则会返回最新的订单。

## 调整开仓杠杆 (TRADE)
