---
title: 现货 API 文档 

language_tabs: # must be one of https://git.io/vQNgJ
  - shell

toc_footers:
  - <a href='https://www.toobit.com'>TooBit</a>
includes:

search: true
---

# 更新日志

## 2022-10-16
  创建文档

# 介绍

## API Key 设置

- 很多接口需要API Key才可以访问. 请参考<a href='https://toobit.zendesk.com/hc/zh-cn/articles/13445077851545-How-to-Create-Your-API-Key'>这个页面</a>来设置API Key.
- 设置API Key的同时，为了安全，建议设置IP访问白名单.
- **永远不要把你的API key/secret告诉给任何人**

<aside class="warning">
如果不小心泄露了API key，请立刻删除此Key, 并可以另外生产新的Key.
</aside>

## API Key 权限设置

- 新创建的API的默认权限是 `只读`。

## 账户

### 现货账户

新注册的账号都会有一个现货(`SPOT`)账号。

# 基础信息

## API 基本信息

- 接口可能需要用户的 API Key，如何创建API-KEY请参考<a href='https://toobit.zendesk.com/hc/zh-cn/articles/13445077851545-How-to-Create-Your-API-Key'>这里</a>
- 本篇列出接口的baseurl: https://api.toobit.com
- 所有接口的响应都是 JSON 格式。
- 所有时间、时间戳均为UNIX时间，单位为毫秒。

### HTTP 返回代码

- HTTP `4XX` 错误码用于指示错误的请求内容、行为、格式。问题在于请求者。
- HTTP `403` 错误码表示违反WAF限制(Web应用程序防火墙)。
- HTTP `429` 错误码表示警告访问频次超限，你有义务停止发送请求。
- HTTP `5XX` 错误码用于指示服务侧的问题。

### 接口的基本信息

- `GET` 方法的接口, 参数必须在 `query string`中发送。
- `POST`, `PUT`, 和 `DELETE` 方法的接口,参数可以在内容形式为`application/x-www-form-urlencoded`的 `query string` 中发送，也可以在 `request body` 中发送。 如果你喜欢，也可以混合这两种方式发送参数。
- 对参数的顺序不做要求。
- 但如果同一个参数名在`query string`和`request body`中都有，`query string`中的会被优先采用。

## 访问限制

### 访问限制基本信息

- 在 `/api/v1/exchangeInfo` `rateLimits` 数组中包含与交易的有关`REQUEST_WEIGHT`和`ORDERS`速率限制相关的对象。这些在 限制种类 (`rateLimitType`) 下的 `枚举定义` 部分中进一步定义。
- 违反任何一个速率限制时，将返回429。
- 每一个接口均有一个相应的权重(weight)，有的接口根据参数不同可能拥有不同的权重。越消耗资源的接口权重就会越大。
- 收到429时，您有责任停止发送请求，不得滥用API。

<aside class="notice">
建议您尽可能多地使用websocket消息获取相应数据，以减少请求带来的访问限制压力。
</aside>

### 下单频率限制
- 当下单数超过限制时，会收到带有429 HTTP CODE 的响应。请检查 `GET` `/api/v1/exchangeInfo` 的下单频率限制 (rateLimitType = ORDERS) 并等待封禁时间结束。
- 下单频率限制是基于每个账户计数的。

### WEB SOCKET 连接限制

- Websocket服务器每秒最多接受5个消息。消息包括:
  - PING帧
  - PONG帧
  - JSON格式的消息, 比如订阅, 断开订阅.
- 如果用户发送的消息超过限制，连接会被断开连接。反复被断开连接的IP有可能被服务器屏蔽。


## 需要签名的接口
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

- 本篇所列出API接口的base url :  **wss://stream.toobit.com**
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

# 错误代码

错误由两部分组成：错误代码和消息。代码是通用的，但消息可以不同。

> 错误JSON格式

``` json
{  
  "code":-1121, 
  "msg":"Invalid symbol."
}
```

## 10xx-通用服务器或网络问题

### -1000 UNKNOWN
- 处理请求时发生未知错误。

### -1001 DISCONNECTED
- 内部错误；无法处理您的请求。请重试。

### -1002 UNAUTHORIZED
- 您无权限执行此请求。请求需要包含API密钥。我们建议在任何请求中包含API密钥。

### -1003 TOO_MANY_REQUESTS
- 请求太多；请使用WebSocket进行实时更新。
- 请求太多；当前限制为每分钟%s个请求。请使用webSocket进行实时更新以避免轮询API。
- 请求太多；IP直到%s才被禁止。请使用WebSocket进行实时更新以避免禁止。

### -1006 UNEXPECTED_RESP
- 从消息总线收到意外响应。执行状态未知。OPENAPI服务器在执行请求中发现异常。请向客户服务报告。

### -1007 TIMEOUT
- 等待后端服务器响应的超时。发送状态未知；执行状态未知。

### -1014 UNKNOWN_ORDER_COMPOSITION
- 不支持的订单组合。

### -1015 TOO_MANY_ORDERS
- 达到速率限制。请放慢您的请求速度。
- 太多的新订单。
- 新订单太多；当前限制为%s每%s的订单数。

### -1016 SERVICE_SHUTTING_DOWN
- 此服务不再可用。

### -1020 UNSUPPORTED_OPERATION
- 不支持此操作。

### -1021 INVALID_TIMESTAMP
- 此请求的时间戳在recvWindow之外。
- 此请求的时间戳比服务器的时间早1000毫秒。
- 请检查您的本地时间和服务器时间之间的差异。

### -1022 INVALID_SIGNATURE
- 此请求的签名无效。

## 11xx - 2xxx Request issues

### -1100 ILLEGAL_CHARS
- 在参数中发现非法字符。
- 在参数“%s”中找到非法字符；合法范围为“%s”。

### -1101 TOO_MANY_PARAMETERS
- 为此端点发送的参数太多。
- 参数太多；期望'%s'并收到'%s'。
- 检测到的参数的重复值。

### -1102 MANDATORY_PARAM_EMPTY_OR_MALFORMED
- 未发送强制参数、为空/null或格式错误。
- 强制参数'%s'未发送，是空/null或格式错误。
- 必须发送参数'%s'或'%s'，但两者都是空/null！

### -1103 UNKNOWN_PARAM
- 发送了一个未知参数。
- 在BBEx Open Api中，每个请求至少需要一个参数。{Timestamp}。

### -1104 UNREAD_PARAMETERS
- 并非所有发送的参数都被读取。
- 并非所有发送的参数都被读取；读取'%s'参数，但发送了'%s'。

### -1105 PARAM_EMPTY
- 参数为空。
- 参数"%1！"为空。

### -1106 PARAM_NOT_REQUIRED
- 不需要时发送参数。
- 参数“%1！”在不需要时发送。

### -1111 BAD_PRECISION
- 精度高于为此资产定义的最大值。

### -1112 NO_DEPTH
- 书上没有符号的订单。

### -1114 TIF_NOT_REQUIRED
- 不需要时发送的TimeInForce参数。

### -1115 INVALID_TIF
- 无效的时间。
- 在当前版本中，此参数为空或GTC。

### -1116 INVALID_ORDER_TYPE
- 订单类型无效。
- 在当前版本中，ORDER_TYPE值是LIMIT或MARKET。

### -1117 INVALID_SIDE
- 无效边。
- ORDER_SIDE值是买入或卖出

### -1118 EMPTY_NEW_CL_ORD_ID
- 新客户端订单ID为空。

### -1119 EMPTY_ORG_CL_ORD_ID
- 原始客户端订单ID为空。

### -1120 BAD_INTERVAL
- 无效的间隔。

### -1121 BAD_SYMBOL
- 无效符号。

### -1125 INVALID_LISTEN_KEY
- 此listenKey不存在。

### -1127 MORE_THAN_XX_HOURS
- 查找间隔太大。
- 开始时间和结束时间之间超过%s小时。

### -1128 OPTIONAL_PARAMS_BAD_COMBO
- 可选参数的组合无效。

### -1130 INVALID_PARAMETER
- 为参数发送的数据无效。
- 为参数"%1！"发送的数据无效。

### -1132 ORDER_PRICE_TOO_HIGH
- 订单价格太高。

### -1133 ORDER_PRICE_TOO_SMALL
- 订单价格低于最低，请检查一般经纪人信息。

### -1134 ORDER_PRICE_PRECISION_TOO_LONG
- 订单价格小数太长，请检查一般经纪人信息。

### -1135 ORDER_QUANTITY_TOO_BIG
- 订单数量太大。

### -1136 ORDER_QUANTITY_TOO_SMALL
- 订单数量低于最低数量。

### -1137 ORDER_QUANTITY_PRECISION_TOO_LONG
- 订购数量小数太长。

### -1138 ORDER_PRICE_WAVE_EXCEED
- 订单价格超出允许范围。

### -1139 ORDER_HAS_FILLED
- 订单已完成。

### -1140 ORDER_AMOUNT_TOO_SMALL
- 交易金额低于最低金额。

### -1141 ORDER_DUPLICATED
- 客户端订单重复

### -1142 ORDER_CANCELLED
- 订单已被取消

### -1143 ORDER_NOT_FOUND_ON_ORDER_BOOK
- 在订单簿上找不到

### -1144 ORDER_LOCKED
- 订单已被锁定

### -1145 ORDER_NOT_SUPPORT_CANCELLATION
- 此订单类型不支持取消

### -1146 ORDER_CREATION_TIMEOUT
- 订单创建超时

### -1147 ORDER_CANCELLATION_TIMEOUT
- 订单取消超时

### -2010 NEW_ORDER_REJECTED
- 新订单被拒绝

### -2011 CANCEL_REJECTED
- 取消订单被拒绝

### -2013 NO_SUCH_ORDER
- 订单不存在

### -2014 BAD_API_KEY_FMT
- API密钥格式无效。

### -2015 REJECTED_MBX_KEY
- 操作的API、IP或权限无效。

### -2016 NO_TRADING_WINDOW
- 找不到该品种的交易窗口。试试股票代码/24小时。

## 过滤器故障

| 报错信息	     | 描述           |
| ----------- | -------------- |
| "Filter failure: PRICE_FILTER" | 价格太高，太低，和/或不遵循交易品种的分时大小规则。|
| "Filter failure: LOT_SIZE" | 数量太高、太低和/或未遵循符号的步长规则。|
| "Filter failure: MIN_NOTIONAL" |价格*数量太低，不能作为该品种的有效订单。|
| "Filter failure: MAX_NUM_ORDERS" |	客户在交易对上有太多挂单。|
| "Filter failure: MAX_ALGO_ORDERS"| 账户有太多未平仓止损和/或在交易对上执行获利指令。 |
| "Filter failure: ICEBERG_PARTS" | ICEBERG 订单会分成太多部分； icebergQty太小。|

## 订单拒绝错误

| 报错信息	     | 描述           |
| ----------- | -------------- |
|"Unknown order sent." | 	找不到订单(通过"orderId"，"clientOrderId"，"origClientOrderId")|
|"Duplicate order sent." | clientOrderId已经被使用|
|"Market is closed." | 该交易对不在交易范围 |
|"Account has insufficient balance for requested action." | 没有足够的资金来完成行动  |
|"Market orders are not supported for this symbol." | 交易对上未启用`MARKET` |
|"Iceberg orders are not supported for this symbol." | 交易对上未启用`icebergQty` |
|"Stop loss orders are not supported for this symbol." | 交易对上未启用 `STOP_LOSS` |
|"Stop loss limit orders are not supported for this symbol." | 交易对上未启 `STOP_LOSS_LIMIT` |
|"Take profit orders are not supported for this symbol." | 交易对上未启用`TAKE_PROFIT`|
|"Take profit limit orders are not supported for this symbol." | 交易对上未启用`TAKE_PROFIT_LIMIT`|
|"Price* QTY is zero or less." | `price` * `quantity`太小|
|"IcebergQty exceeds QTY." | `icebergQty` 必须少于订单数量|
|"This action disabled is on this account." | 联系客户支持； 该帐户已禁用了某些操作。|
|"Unsupported order combination" | 不允许组合`orderType`, `timeInForce`, `stopPrice`, 和/或 `icebergQty` 。|
|"Order would trigger immediately." | 与最后交易价格相比，订单的止损价无效。|
|"Cancel order is invalid. Check origClOrdId and orderId." | 未发送`origClientOrderId` 或`orderId` 。|
|"Order would immediately match and take." | `LIMIT_MAKER `订单类型将立即匹配并进行交易，而不是纯粹的生成订单。|
