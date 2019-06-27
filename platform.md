## Platform 

用户中心颁发的有效证书（token），有效时间2小时，过期后请重新获取。
该鉴权方式只支持部分高级接口，不支持的部分请要使用read token

### 使用资格
该功能仅对部分用户开放，如需使用请联系客服。

### 获取证书 token
1. 准备我方提供的client_id,client_secret
1. [post] https://account.blockin.com/oauth/v1/token
参数client_id，client_secret，grant_type，scopes
1. grant_type传固定值"client_credentials"
1. scopes目前有subaccount-create，watcher-create，address-update，cloud四种，对应具体的接口权限。用空格分隔成字符串提交。

会得到类似返回
```json
{
     "access_token" : "eyJ0e***************************kjerg",
     "expires_in" : 7200,
     "token_type" : "bearer",
     "scope" : "cloud"
}
```
access_token就是我们需要的token，expires_in是有效期


## 可用API
api的相关信息如返回的格式，携带token的方式请统一参考
[api.md](./api.md)

### 创建子账户 需要subaccount-create
**请求参数**

|字段|类型|必须|备注|
|---|---|---|---|
|name|string|true|子账户名称|
|coin_type|string|true|默认的币种|
|payment_id|string|false|xmr、etn的支付id|


**请求URL**

```
POST https://api-prod.poolin.com/api/public/v2/platform/subaccount/create
```

**返回结果**

``` json
{
    "err_no": 0,
    "data": {
        "puid": 9999888,
        "name": "111ttbtc",
        "default_coin_type": "zec",
        "coins": {},
        "created_at": 1561537835
    }
}

```
如果err_no不为0，是以下的数字的话，请对应错误原因修改。

|错误码|原因|
|---|---|
|100|目前最多允许9999个子账户，更多需要单独申请|
|101|子账户名称已经存在了|
|102|24小时内只能创建100个子账户，超过频率会报错，需要更高频率单独申请|
|501，505|地址验证失败|
|504|payment_id验证失败|
|507|意外的错误|

**返回字段说明**

|返回值字段|说明|
|---|---|
|puid|子账户id|
|name|名称|
|default_coin_type|默认币种|

### 创建观察者、观察者token  需要watcher-create
**请求参数**

|字段|类型|必须|备注|
|---|---|---|---|
|name|string|true|子账户名称|
|coin_types|array|true|观察者能看到的币种,如['btc']|
|puid|int|true|子账户id|


**请求URL**

```
POST https://api-prod.poolin.com/api/public/v2/platform/watcher/token/create
```

**返回结果**

``` json
{
    "err_no": 0,
    "data": {
        "token": "wowEuP*************EiL",
        "puid": 9999888,
        "coin_types": "btc",
        "regions": "*",
        "name": "ttttt",
        "created_at": 1561538162,
        "expired_at": 1577349362,
        "subaccount_name": "111ttbtc"
    }
}

```
如果err_no不为0，是以下的数字的话，请对应错误原因修改。

|错误码|原因|
|---|---|
|100|目前最多50个生效的观察者|


**返回字段说明**

|返回值字段|说明|
|---|---|
|token|观察者token|
|puid|子账户id|
|coin_types|支持的币种信息|
|name|观察者名称|
|created_at|创建时间|
|expired_at|过期时间|
|subaccount_name|子账户名称|


### 更新子账户地址 需要 address-update
**请求参数**

|字段|类型|必须|备注|
|---|---|---|---|
|address|int|true|要更新的地址|
|coin_type|string|true|是要更新哪一个币种的地址|


**请求URL**

```
POST https://api-prod.poolin.com/api/public/v2/platform/subaccount/{puid}/update/address
```

**返回结果**

``` json
{
    "err_no": 0,
    "data": null
}

```
如果err_no不为0，是以下的数字的话，请对应错误原因修改。

|错误码|原因|
|---|---|
|100|子账户不存在|
|501，505|地址验证失败|
|504|payment_id验证失败|


**返回字段说明**

|返回值字段|说明|
|---|---|


## 云算力功能的基本情况 需要cloud
**我们不允许一个子账户既出让算力，也受让算力。同一时间只能是一种状态。现在仅支持比特币btc的转移，单位只能是T。从美东区域接入的算力暂时无法被转移。**


### 云算力-合约、转移列表
` 云算力现在仅支持btc ` 
显示最新的100条未删除云算力分配记录。

**请求参数**

|字段|类型|必须|备注|
|---|---|---|---|
|coin_type|string|true|币种，目前仅支持btc|
|puid|int|true|子账户id|


**请求URL**

```
GET https://api-prod.poolin.com/api/public/v2/platform/cloud/transfer-hashrate?coin_type=btc&puid=9001000
```

**返回结果**

``` json
{
    "err_no": 0,
    "data": {
        "id": 101,
        "name": "suba",
        "to": "subb",
        "hashrate": 23,
        "unit": "T",
        "start_ts": 1557936000,
        "end_ts": 1558022400,
        "status": 3
    }
}

```

**返回字段说明**

|返回值字段|说明|
|---|---|
|id|转移合约id|
|name|转让人|
|to|受让人|
|hashrate|算力值|
|unit|算力单位|
|start_ts|开始时间戳|
|end_ts|过期时间戳|
|status|1未开始 2生效中 3过期|


### 云算力-可转让算力查询
` 云算力现在仅支持btc `
目前最多只能转让实时算力的60%，多个合约累计计算。 

**请求参数**

|字段|类型|必须|备注|
|---|---|---|---|
|coin_type|string|true|币种，目前仅支持btc|
|puid|int|true|子账户id|


**请求URL**

```
GET https://api-prod.poolin.com/api/public/v2/platform/cloud/allow-sell?coin_type=btc&puid=9001000
```

**返回结果**

``` json
{
    "err_no": 0,
    "data": {
        "can_sell": 89,
        "unit": "T",
    }
}

```

**返回字段说明**

|返回值字段|说明|
|---|---|
|can_sell|允许转移的算力大小|
|unit|算力单位|


### 云算力-受让方查询
` 云算力现在仅支持btc `
 

**请求参数**

|字段|类型|必须|备注|
|---|---|---|---|
|puid|int|true|出让方puid|
|name|string|true|受让方名称，长度4-30|


**请求URL**

```
GET https://api-prod.poolin.com/api/public/v2/platform/cloud/check-name?name=testname&puid=9001000
```

**返回结果**

``` json
{
    "err_no": 0,
    "data": true
}
|错误码|原因|
|---|---|
|101|受让方出让算力中|
|103|出让方有来自别人的算力|

```

**返回字段说明**

|返回值字段|说明|
|---|---|
|data|是否存在受让方|


### 云算力-删除合约
` 云算力现在仅支持btc ` 

**请求参数**

|字段|类型|必须|备注|
|---|---|---|---|
|puid|int|true|合约所属转让人的puid|
|id|int|true|合约id|


**请求URL**

```
POST https://api-prod.poolin.com/api/public/v2/platform/cloud/delete
```

**返回结果**

``` json
{
    "err_no": 0,
    "data": null
}

```

**返回字段说明**

|返回值字段|说明|
|---|---|


### 云算力-创建合约、转移算力
` 云算力现在仅支持btc ` 

**请求参数**

|字段|类型|必须|备注|
|---|---|---|---|
|puid|int|true|出让人子账户id|
|coin_type|string|true|币种，目前仅支持btc|
|unit|string|true|转让算力单位，目前只能是T|
|hashrate|int|true|转让的算力值|
|to|string|true|受让方子账户名|
|start_time|int|true|开始时间|
|end_time|int|true|结束时间|


**请求URL**

```
POST https://api-prod.poolin.com/api/public/v2/platform/cloud/{puid}/transfer
```

**返回结果**

``` json
{
    "err_no": 0,
    "data": 123
}

```
|错误码|原因|
|---|---|
|100|受让方不存在|
|101|受让方出让算力中|
|102|不允许出让超过60%的算力|
|103|出让方有来自别人的算力|


**返回字段说明**

|返回值字段|说明|
|---|---|
|data|合约id|
