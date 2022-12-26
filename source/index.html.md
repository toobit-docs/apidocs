---
title: U本位合约 API 文档 ENGLISH

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


# Market Data Endpoints


# Websocket Market Streams

# 

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