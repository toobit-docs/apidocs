---
title: 现货 API 文档 

language_tabs: # must be one of https://git.io/vQNgJ
  - shell

toc_footers:
  - <a href='https://www.toobit.com'>Toobit</a>
includes:

search: true
---

# 更新日志

# 介绍

## API Key 设置

- 很多接口需要API Key才可以访问. 请参考<a href=“https://toobit.zendesk.com/hc/zh-cn/articles/13445077851545-How-to-Create-Your-API-Key”>这个页面</a>来设置API Key.
- 设置API Key的同时，为了安全，建议设置IP访问白名单.
- **永远不要把你的API key/secret告诉给任何人**

<aside class="warning">
如果不小心泄露了API key，请立刻删除此Key, 并可以另外生产新的Key.
</aside>

## API Key 权限设置

- 新创建的API的默认权限是 `只读`。

## 账户

### 现货账户

新注册的币安账号都会有一个现货(`SPOT`)账号。

# 基础信息

## API 基本信息

- 接口可能需要用户的 API Key，如何创建API-KEY请参考<a href="#">这里</a>
- 本篇列出接口的baseurl: https://bapi.wcsbapp.com
- 所有接口的响应都是 JSON 格式。
- 所有时间、时间戳均为UNIX时间，单位为毫秒。

### HTTP 返回代码

- HTTP `4XX` 错误码用于指示错误的请求内容、行为、格式。问题在于请求者。
- HTTP `403` 错误码表示违反WAF限制(Web应用程序防火墙)。
- HTTP `429` 错误码表示警告访问频次超限，即将被封IP。
- HTTP `418` 表示收到429后继续访问，于是被封了。
- HTTP `5XX` 错误码用于指示服务侧的问题。

### 接口的基本信息

- `GET` 方法的接口, 参数必须在 `query string`中发送。
- `POST`, `PUT`, 和 `DELETE` 方法的接口,参数可以在内容形式为`application/x-www-form-urlencoded`的 `query string` 中发送，也可以在 `request body` 中发送。 如果你喜欢，也可以混合这两种方式发送参数。
- 对参数的顺序不做要求。
- 但如果同一个参数名在`query string`和`request body`中都有，`query string`中的会被优先采用。

## 访问限制

### 访问限制基本信息

- 以下 是`intervalLetter` 作为头部值:

  - SECOND => S
  - MINUTE => M
  - HOUR => H
  - DAY => D
- 在 `/api/v1/exchangeInfo` `rateLimits` 数组中包含与交易的有关`RAW_REQUESTS`，`REQUEST_WEIGHT`和`ORDERS`速率限制相关的对象。这些在 限制种类 (`rateLimitType`) 下的 `枚举定义` 部分中进一步定义。
- 违反任何一个速率限制时，将返回429。

### IP 访问限制

- 每一个接口均有一个相应的权重(weight)，有的接口根据参数不同可能拥有不同的权重。越消耗资源的接口权重就会越大。
- 收到429时，您有责任停止发送请求，不得滥用API。
- 收到429后仍然继续违反访问限制，会被封禁IP，并收到418错误码
- 频繁违反限制，封禁时间会逐渐延长，从最短2分钟到最长3天。
- Retry-After的头会与带有418或429的响应发送，并且会给出以秒为单位的等待时长(如果是429)以防止禁令，或者如果是418，直到禁令结束。
- **访问限制是基于IP的，而不是API Key**

<aside class="notice">
建议您尽可能多地使用websocket消息获取相应数据，以减少请求带来的访问限制压力。
</aside>

### 下单频率限制
- 当下单数超过限制时，会收到带有429但不含Retry-After头的响应。请检查 GET api/v3/exchangeInfo 的下单频率限制 (rateLimitType = ORDERS) 并等待封禁时间结束。
- 被拒绝或不成功的下单并不保证回报中包含以上头内容。
- 下单频率限制是基于每个账户计数的。

### WEB SOCKET 连接限制

- Websocket服务器每秒最多接受5个消息。消息包括:
  - PING帧
  - PONG帧
  - JSON格式的消息, 比如订阅, 断开订阅.
- 如果用户发送的消息超过限制，连接会被断开连接。反复被断开连接的IP有可能被服务器屏蔽。

## 数据来源
- 因为API系统是异步的, 所以返回的数据有延时很正常, 也在预期之中。
- 在每个接口中，列出了其数据的来源，可以用于理解数据的时效性。

系统一共有3个数据来源，按照更新速度的先后排序。排在前面的数据最新，在后面就有可能存在延迟。

- 撮合引擎 - 表示数据来源于撮合引擎
- 缓存 - 表示数据来源于内部或者外部的缓存
- 数据库 - 表示数据直接来源于数据库

<aside class="notice">
有些接口有不止一个数据源, 比如 `缓存 => 数据库`, 这表示接口会先从第一个数据源检查，如果没有数据，则检查下一个数据源。
</aside>

## 接口鉴权类型

- 每个接口都有自己的鉴权类型，鉴权类型决定了访问时应当进行何种鉴权。
- 鉴权类型会在本文档中各个接口名称旁声明，如果没有特殊声明即默认为 NONE。
- 如果需要 API-keys，应当在HTTP头中以 X-BB-APIKEY字段传递。
- API-keys 与 secret-keys 是大小写敏感的。
- API-keys可以被配置为只拥有访问一些接口的权限。 例如, 一个 API-key 仅可用于发送交易指令, 而另一个 API-key 则可访问除交易指令外的所有路径。
- 默认 API-keys 可访问所有鉴权路径.

| 鉴权类型	    | 描述 | 
| ----------- | ---- | 
| NONE | 端点可以自由访问。 |
| TRADE | 端点需要发送有效的API-Key和签名。 |
| USER_DATA | 端点需要发送有效的API-Key和签名 |
| USER_STREAM | 端点需要发送有效的API-Key。 |
| MARKET_DATA | 端点需要发送有效的API-Key。 |

### SIGNED (TRADE、USER_DATA ) Endpoint security

- 调用SIGNED 接口时，除了接口本身所需的参数外，还需要在query string 或 request body中传递 signature, 即签名参数。
- 签名使用HMAC SHA256算法. API-KEY所对应的API-Secret作为 HMAC SHA256 的密钥，其他所有参数作为HMAC SHA256的操作对象，得到的输出即为签名。
- 签名 大小写不敏感.
- "totalParams"定义为与"request body"串联的"query string"。

### 时间同步安全
- 签名接口均需要传递 timestamp参数，其值应当是请求发送时刻的unix时间戳(毫秒)。
- 服务器收到请求时会判断请求中的时间戳，如果是5000毫秒之前发出的，则请求会被认为无效。这个时间空窗值可以通过发送可选参数 recvWindow来定义。

**关于交易时效性** 互联网状况并不完全稳定可靠,因此你的程序本地到币安服务器的时延会有抖动。这是我们设置recvWindow的目的所在，如果你从事高频交易，对交易时效性有较高的要求，可以灵活设置recvWindow以达到你的要求。


# 钱包接口

## 提币 (USER_DATA)
- `POST /api/v1/account/withdraw`

提交一个提币请求。

### 权重：1

> 响应

``` json
{
"status": 0,
"success": true,
"needBrokerAudit": false, // 是否需要券商审核
"id": "423885103582776064", // 提币成功订单id
"refuseReason":"" // 失败拒绝原因
}
```

### 参数

| 参数名称     | 类型      | 是否必需      | 描述           |
| ----------- | ------- | ------------- | -------------- |
| coin | STRING | YES | 资产 |
| clientOrderId | LONG | YES | 	自定义提币ID |
| address | STRING  | YES | 提币地址(注意：提现地址必须是在PC端或者APP端维护在常用地址列表里面的地址) |
| addressExt | STRING | NO | tag |
| quantity  | DECIMAL | YES | 提币数量 |
| chainType | STRING | NO | chain type, USDT的chainType分别是OMNI ERC20 TRC20，默认OMNI |


## 获取提币记录 (USER_DATA)
- `GET /api/v1/account/withdrawOrders  (HMAC SHA256)`

### 权重：5

> 响应

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
        "quantity":"14", // 提币金额
        "arriveQuantity":"14", // 到账金额
        "statusCode":"PROCESSING_STATUS",
        "status":3,
        "txId ":"",
        "txIdUrl ":"",
        "walletHandleTime":"1536232111669",
        "feeCoinId ":"BHC",
        "feeCoinName ":"BHC",
        "fee":"0.1",
        "requiredConfirmTimes ":0, // 要求确认数
        "confirmTimes ":0, // 确认数
        "kernelId":"", // BEAM 和 GRIN 独有
        "isInternalTransfer": false // 是否内部转账
    }
]
```

### 参数

| 参数名称     | 类型      | 是否必需      | 描述           |
| ----------- | ------- | ------------- | -------------- |
| coin | STRING | NO | 资产 |
| startTime | LONG | NO | 开始时间戳 |
| endTime | LONG | NO | 结束时间戳 |
| fromId | LONG | NO | 从哪个OrderId起开始抓取 |
| withdrawOrderId | LONG | NO | 提现订单ID |
| limit | INT | NO | 默认 500; 最大 1000 |
| recvWindow | LONG | NO | recv窗口 |
| timestamp | LONG | YES | 时间戳 |

## 获取充值地址 (USER_DATA)
- `GET /api/v1/account/deposit/address  (HMAC SHA256)`

### 权重：1

> 响应

``` json
    {
        "canDeposit":false,//是否可充值
        "address":"0x815bF1c3cc0f49b8FC66B21A7e48fCb476051209",//地址
        "addressExt":"address tag",
        "minQuantity":"100",//最小金额
        "requiredConfirmTimes ":1,//到账确认数
        "canWithdrawConfirmNum ":12,//提币确认数
        "coinType":"ERC20_TOKEN"//链类型
    }
```

### 参数
| 参数名称     | 类型      | 是否必需      | 描述           |
| ----------- | ------- | ------------- | -------------- |
| coin | STRING | YES | 资产 |
| chainType | STRING | YES | chain type, USDT的chainType分别是OMNI ERC20 TRC20，默认OMNI |

## 获取充值历史 (USER_DATA)
 - `GET /api/v1/account/depositOrders  (HMAC SHA256)`

 ### 权重： 5

> 响应

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

 ### 参数
| 参数名称     | 类型      | 是否必需      | 描述           |
| ----------- | ------- | ------------- | -------------- |
| coin | STRING | NO | 资产 |
| startTime | LONG | NO | 开始时间戳 |
| endTime | LONG | NO | 结束时间戳 |
| fromId | LONG | NO | 从哪个Id起开始抓取 |
| limit | INT | NO | 默认 500; 最大 1000 |
| recvWindow | LONG | NO | recv窗口 |
| timestamp | LONG | YES | 时间戳 |、

注意：

- 如果fromId设定好了，会筛选订单小于id的。否则会返回最近的订单信息。

# 现货账户和交易接口

## 测试下单 (TRADE)
- `POST /api/v1/spot/orderTest (HMAC SHA256)`

用于测试订单请求，但不会提交到撮合引擎

### 权重：1

> 响应

``` json
{}
```

### 参数
同于 `POST /api/v1/spot/order`

## 下单 (TRADE)
- `POST /api/v1/spot/order  (HMAC SHA256)`

发送下单。

### 权重：1

> 响应

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

### 参数

| 参数名称     | 类型      | 是否必需      | 描述           |
| ----------- | ------- | ------------- | -------------- |
| symbol | STRING | YES | 交易对 |
| assetType | ENUM | NO | `CASH`、`MARGIN`，只支持`CASH` |
| side | ENUM | YES | `BUY`或`SELL `|
| type | ENUM | YES | 详见枚举定义：订单类型 |
| timeInForce | ENUM | NO | 详见枚举定义：有效方式 |
| quantity | DECIMAL | YES | 数量 |
| price | DECIMAL | NO | 价格 |
| newClientOrderId | STRING | NO | 一个自己给订单定义的ID，如果没有发送会自动生成。 |
| stopPrice | DECIMAL | NO | 与 `STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT`, 和`TAKE_PROFIT_LIMIT` 订单一起使用. **当前不可用** |
| icebergQty | DECIMAL | NO | 与 `LIMIT`, `STOP_LOSS_LIMIT`, 和` TAKE_PROFIT_LIMIT` 来创建冰山订单. **当前不可用** |
| recvWindow | LONG | NO | recv窗口 |
| timestamp | LONG | YES | 时间戳 |

基于订单 `type`不同，强制要求某些参数:

| 类型     | 额外强制参数      | 
| ----------- | ------- | 
| `LIMIT` | `timeInForce`, `quantity`,` price` |
| `MARKET` |  `quantity` |
| `STOP_LOSS` | `quantity`, `stopPrice` **当前不可用** |
| `STOP_LOSS_LIMIT` | `timeInForce`, `quantity`, `price`, `stopPrice`  **当前不可用**|
| `TAKE_PROFIT` | `quantity`, `stopPrice` **当前不可用** |
| `TAKE_PROFIT_LIMIT` | `timeInForce`, `quantity`, `price`, `stopPrice` **当前不可用** |
| `LIMIT_MAKER` | `quantity`, `price` |

## 撤销订单 (TRADE)
- `DELETE /api/v1/spot/order  (HMAC SHA256)`

取消有效订单。

### 权重：1

> 响应

``` json
{
  "symbol": "LTCBTC",
  "orderId": 1,
  "clientOrderId": "9t1M2K0Ya092",
  "price": "0.1",
  "origQty": "1.0",
  "executedQty": "0.0",
  "status": "CANCELED",
  "timeInForce": "GTC",
  "type": "LIMIT",
  "side": "BUY",
  "transactTime": 1499827319559
}
```

### 参数
| 参数名称     | 类型      | 是否必需      | 描述           |
| ----------- | ------- | ------------- | -------------- |
| orderId | LONG | NO | 订单ID |
| clientOrderId | STRING | NO | 客户订单ID |
| recvWindow | LONG | NO | recv窗口 |
| timestamp | LONG | YES | 时间戳 |

`orderId` 或 `origClientOrderId` 必须至少发送一个

##  批量撤单 (TRADE)
- `DELETE /api/v1/spot/openOrders (HMAC SHA256)`

### 权重：5

> 响应

``` json
{
  "success":true
}
```

### 参数
| 参数名称     | 类型      | 是否必需      | 描述           |
| ----------- | ------- | ------------- | -------------- |
| symbol | STRING | NO | 现货名称（多个用,隔开） |
| side | ENUM| NO | BUY或SELL |


## 查询订单  (USER_DATA)
- `GET /api/v1/spot/order (HMAC SHA256)`

### 权重：1

> 响应

``` json
{
  "symbol": "LTCBTC",
  "orderId": 1,
  "clientOrderId": "9t1M2K0Ya092",
  "price": "0.1",
  "origQty": "1.0",
  "executedQty": "0.0",
  "cummulativeQuoteQty": "0.0",
  "status": "NEW",
  "timeInForce": "GTC",
  "type": "LIMIT",
  "side": "BUY",
  "stopPrice": "0.0",
  "icebergQty": "0.0",
  "time": 1499827319559,
  "updateTime": 1499827319559,
  "isWorking": true
}
```

### 参数
| 参数名称     | 类型      | 是否必需      | 描述           |
| ----------- | ------- | ------------- | -------------- |
| orderId | LONG | NO | 订单ID |
| origClientOrderId | STRING | NO | 客户订单ID |
| recvWindow | LONG | NO | recv窗口 |
| timestamp | LONG | YES | 时间戳 |

注意：

- 单一 orderId 或者 origClientOrderId 必须被发送。
- 对于某些历史数据 cummulativeQuoteQty 可能会 < 0, 这说明数据当前不可用。

## 当前挂单 (USER_DATA)
- `GET /api/v1/spot/openOrders (HMAC SHA256)`

获取交易对的所有当前挂单， 请小心使用不带交易对参数的调用。

### 权重 ：1

> 响应

``` json
[
  {
    "symbol": "LTCBTC",
    "orderId": 1,
    "clientOrderId": "t7921223K12",
    "price": "0.1",
    "origQty": "1.0",
    "executedQty": "0.0",
    "cummulativeQuoteQty": "0.0",
    "status": "NEW",
    "timeInForce": "GTC",
    "type": "LIMIT",
    "side": "BUY",
    "stopPrice": "0.0",
    "icebergQty": "0.0",
    "time": 1499827319559,
    "updateTime": 1499827319559,
    "isWorking": true
  }
]
```

### 参数
| 参数名称     | 类型      | 是否必需      | 描述           |
| ----------- | ------- | ------------- | -------------- |
| orderId | LONG | NO | 订单ID |
| symbol | STRING | NO | 交易对 |
| limit | INT | NO | 默认 500; 最多 1000. |
| recvWindow | LONG | NO | recv窗口 |
| timestamp | LONG | YES | 时间戳 |

注意：

- 如果orderId设定好了，会筛选订单小于orderId的。否则会返回最近的订单信息。

## 查询所有订单 (USER_DATA)
- `GET /api/v1/spot/tradeOrders (HMAC SHA256)`

获取所有帐户订单； 有效，已取消或已完成。

### 权重：5

> 响应

``` json
[
  {
    "symbol": "LTCBTC",
    "orderId": 1,
    "clientOrderId": "987yjj2Ym",
    "price": "0.1",
    "origQty": "1.0",
    "executedQty": "0.0",
    "cummulativeQuoteQty": "0.0",
    "status": "NEW",
    "timeInForce": "GTC",
    "type": "LIMIT",
    "side": "BUY",
    "stopPrice": "0.0",
    "icebergQty": "0.0",
    "time": 1499827319559,
    "updateTime": 1499827319559,
    "isWorking": true
  }
]
```

### 参数
| 参数名称     | 类型      | 是否必需      | 描述           |
| ----------- | ------- | ------------- | -------------- |
| orderId | LONG | NO | 订单ID |
| symbol | STRING | NO | 交易对 |
| startTime | LONG | NO | 开始时间戳 |
| endTime | LONG| NO | 结束时间戳 |
| limit | INT | NO | 默认 500; 最多 1000. |
| recvWindow | LONG | NO | recv窗口 |
| timestamp | LONG | YES | 时间戳 |

## 账户信息 (USER_DATA)
- `GET /api/v1/account`

### 权重：5 

> 响应

``` json
{
    "balances": [
        {
            "asset": "BTC", //资产
            "assetId": "BTC", //资产id
            "assetName": "BTC", //资产名称
            "total": "995.899", //总数量
            "free": "995.899", //可用数
            "locked": "0" //冻结数
        }
    ]
}
```

### 参数
| 参数名称     | 类型      | 是否必需      | 描述           |
| ----------- | ------- | ------------- | -------------- |
| recvWindow | LONG | NO | recv窗口 |
| timestamp | LONG | YES | 时间戳 |

## 账户成交历史 (USER_DATA)
- `GET /api/v1/account/trades`

### 权重：5

> 响应

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

### 参数
| 参数名称     | 类型      | 是否必需      | 描述           |
| ----------- | ------- | ------------- | -------------- |
| symbol | STRING | NO | 交易对 |
| startTime | LONG | NO | 开始时间戳 |
| endTime | LONG | NO | 结束时间戳 |
| fromId | LONG | NO  | |
| toId | LONG | NO | |
| limit | INT | NO | 每页显示条数 |
| recvWindow | LONG | NO | recv窗口 |
| timestamp | LONG | YES | 时间戳 |

注意：

- 如果只有fromId，会返回订单号小于fromId的，倒序排列。
- 如果只有toId，会返回订单号小于toId的，升序排列。
- 如果同时有fromId和toId, 会返回订单号在fromId和toId的，倒序排列。
- 如果fromId和toId都没有，会返回最新的成交记录，倒序排列。

# Websocket账户信息推送

- 本篇所列出API接口的base url : 
- 用于订阅账户数据的 `listenKey` 从创建时刻起有效期为60分钟
- 可以通过 PUT 一个 `listenKey` 延长60分钟有效期
- 可以通过DELETE一个 `listenKey` 立即关闭当前数据流，并使该`listenKey` 无效
- 在具有有效listenKey的帐户上执行`POST`将返回当前有效的`listenKey`并将其有效期延长60分钟
- websocket接口的baseurl: 
- 每个链接有效期不超过24小时，请妥善处理断线重连。
- 用户信息流有效负载不保证在繁忙时段处于正常状态；确保使用E订购更新

## Listen Key (现货账户)

### 生成 Listen Key (USER_STREAM)
- `POST /api/v1/userDataStream`

开始一个新的数据流。除非发送 keepalive，否则数据流于60分钟后关闭。如果该帐户具有有效的listenKey，则将返回该listenKey并将其有效期延长60分钟。

#### 权重：1

> 响应

``` json
{
  "listenKey": "1A9LWJjuMwKWYP4QQPw34GRm8gz3x5AephXSuqcDef1RnzoBVhEeGE963CoS1Sgj"
}
```

#### 参数
| 参数名称     | 类型      | 是否必需      | 描述           |
| ----------- | ------- | ------------- | -------------- |
| recvWindow | LONG | NO | recv窗口 |
| timestamp | LONG | YES | 时间戳 |

### 延长 Listen Key 有效期 (USER_STREAM)
- `PUT /api/v1/userDataStream`

有效期延长至本次调用后60分钟,建议每30分钟发送一个 ping 。

#### 权重：1

> 响应

``` json
{}
```

#### 参数
| 参数名称     | 类型      | 是否必需      | 描述           |
| ----------- | ------- | ------------- | -------------- |
| listenKey | STRING | YES | |
| recvWindow | LONG | NO | recv窗口 |
| timestamp | LONG | YES | 时间戳 |

### 关闭 Listen Key (USER_STREAM)
- `DELETE /api/v1/userDataStream`
#### 权重：1

> 响应

``` json
{}
```

#### 参数
| 参数名称     | 类型      | 是否必需      | 描述           |
| ----------- | ------- | ------------- | -------------- |
| listenKey | STRING | YES | |
| recvWindow | LONG | NO | recv窗口 |
| timestamp | LONG | YES | 时间戳 |

## Payload: 账户更新

> Payload

``` json
{
  "e": "outboundAccountInfo",   // Event type事件类型
  "E": 1499405658849,           // Event time事件时间
  "T": true,                    // Can trade? 可否交易
  "W": true,                    // Can withdraw? 可否提币
  "D": true,                    // Can deposit? 可否充币
  "B": [                        // Balances changed 余额变更
    {
      "a": "LTC",               // Asset 资产
      "f": "17366.18538083",    // Free amount 可用金额
      "l": "0.00000000"         // Locked amount 冻结金额
    }
  ]
}
```

每当帐户余额发生更改时，都会发送一个事件`outboundAccountInfo`，其中包含可能由生成余额变动的事件而变动的资产。


## Payload: 订单更新

> Payload

``` json
{
  "e": "executionReport",        // Event type 事件类型
  "E": 1499405658658,            // Event time 事件时间
  "s": "ETHBTC",                 // Symbol 币对
  "c": 1000087761,               // Client order ID 客户订单id
  "S": "BUY",                    // Side 订单方向
  "o": "LIMIT",                  // Order type 订单类型
  "f": "GTC",                    // Time in force 有效方式
  "q": "1.00000000",             // Order quantity 数量
  "p": "0.10264410",             // Order price 价格
  "X": "NEW",                    // Current order status 订单状态
  "i": 4293153,                  // Order ID 订单id
  "l": "0.00000000",             // Last executed quantity 上次数量
  "z": "0.00000000",             // Cumulative filled quantity 交易数量
  "L": "0.00000000",             // Last executed price 上次价格
  "n": "0",                      // Commission amount 佣金
  "N": null,                     // Commission asset 佣金资产
  "u": true,                     // Is the trade normal, ignore for now 是否正常
  "w": true,                     // Is the order working? Stops will have
  "m": false,                    // Is this trade the maker side?
  "O": 1499405658657,            // Order creation time 创建时间
  "Z": "0.00000000"              // Cumulative quote asset transacted quantity 交易金额
```

订单更新会通过 `executionReport` 事件更新。查看API文档和下面的相关枚举定义。
平均价格可以通过Z除以z来找到。

### 执行类型

- NEW  新订单
- PARTIALLY_FILLED 
- FILLED 
- CANCELED 订单被取消
- REJECTED 

## Payload: Ticket推送

> 响应

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
        "S": "SELL"                       // side  SELL or BUY
    }
]
```
