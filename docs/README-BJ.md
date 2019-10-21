## 1、查询号码归属地接口（H码）- 支持异网

接口地址：`/query-acc-info`
接口类型：POST + FormData
接口参数：

| 参数名 |参数说明  |参数类型  |
| --- | --- | --- |
| accNbr |11位手机号码  | String |

请求示例：

| 参数名 | 参数值 |
| --- | --- |
| accNbr |15301310218  |

接口返回值：

```json
{
  "code" : 0,
  "message" : "查询号码归属成功！",
  "data" : {
    "areaName" : "北京",
    "areaCode" : "010"
  },
  "success" : true
}
```

## 2、查询用户 PartyId

接口地址：`/query-party-id`
接口类型：POST + FormData
接口参数：

| 参数名 |参数说明  |参数类型  |
| --- | --- | --- |
| accNbr | 查询号码：手机号码/身份证号 | String |
| accNbrType | 查询号码类型：2-手机号码、3-身份证号 |String  |

请求示例：
| 参数名 | 参数值 |
| --- | --- |
| accNbr |13366503515  |
| accNbrType | 2 |

接口返回值：
```json
{
  "code" : 0,
  "message" : "获取 PartyId 成功",
  "data" : {
    "partyId" : "103012904452"
  },
  "success" : true
}
```

## 3、查询用户名下的手机号码、宽带号码、ITV账号

接口地址：`/query-product-list`
接口类型：POST + FormData
接口参数：

| 参数名 |参数说明  |参数类型  |
| --- | --- | --- |
| accNbr | 查询号码：手机号码/partyId/身份证号 | String |
| accNbrType | 查询号码类型：1-手机号码、2-partyId、13-身份证号 |String  |

请求示例：

| 参数名 | 参数值 |
| --- | --- |
| accNbr |15301310218  |
| accNbrType | 1 |

接口返回值：
```json
{
  "code" : 0,
  "message" : "查询成功！",
  "data" : {
    "broadbands" : [ "B24608101" ],
    "mobiles" : [ "18901230213", "17701323712", "18910837005", "15301310218" ],
    "itvs" : [ "B92563442" ]
  },
  "success" : true
}
```



## 4、查询用户在用主套餐信息

接口地址：`/query-major-product`
接口类型：POST + FormData
接口参数：

| 参数名 |参数说明  |参数类型  |
| --- | --- | --- |
| accNbr |11位手机号码  | String |


请求示例：

| 参数名 | 参数值 |
| --- | --- |
| accNbr |13381113974  |

接口返回值：

```json
{
    "code": 0,
    "message": "",
    "data": {
        "resultCode": "0",
        "resultMsg": "成功",
        "compProds": {
            "compProdName": "小明月租卡201705-融合产品主套餐（已调整为国内流量同价）"
        },
        "orderOffer": {
            "offer": {
                "offerId": "105617667894",
                "offerSpecId": "993035687",
                "offerSpecName": "小明月租卡201705-融合产品主套餐（已调整为国内流量同价）",
                "startDt": "2019-07-01 00:00:00",
                "endDt": "3000-01-01 00:00:00",
                "summary": "(993035687)小明月租卡201705-融合产品主套餐（已调整为国内流量同价）",
                "roleName": "一般构成成员",
                "params": ""
            }
        }
    },
    "success": true
}
```

接口返回值说明：

| 参数名 |参数说明  |参数类型  |
| --- | --- | --- |
| offerId |销售品实例id  | String |
| offerSpecId |销售品规格ID  | String |
| offerSpecName |销售品规格名称  | String |
| startDt |生效时间  | String |
| endDt |endDt  | String |



