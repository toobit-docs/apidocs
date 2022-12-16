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
### 参数


## 账户类型查询

- GET `/linear-swap-api/v3/swap_unified_account_type`

#### 备注
 - 此接口用于客户查询的账号类型，当前U本位合约有统一账户和非统一账户（全仓逐仓账户）类型。统一账户类型资产放在USDT一个账户上，全仓逐仓账户类型资产放在不同的币对。
 - 统一账户类型为最新升级的，当前不支持API下单。若需要用用API下单请切换账户类型为非统一账户。

### 请求参数

无

> Response: 

```json
{
    "code":200,
    "msg":"ok",
    "data":{
        "account_type":2
    },
    "ts":1668057324200
}

```

### 返回参数

| 参数名称    | 是否必须 | 类型      | 描述            | 取值范围           |
| ----------------- | ---- | ------- | ------------- | -------------- |
| code            | true | int  | 状态码        |  |
| msg            | true | string  | 结果描述        |  |
| ts                | true | long    | 时间戳 |                |
| \<data\>          |  true    |         |               |          |
| account_type        | true | int | 账户类型          |     1:非统一账户（全仓逐仓账户）2:统一账户        |
| \</data\>         |   true   |         |        |                |