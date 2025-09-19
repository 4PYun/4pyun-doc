# 同步充电记录

## 接口描述
1. 充电过程中定时上报充电进度；
2. 充电完成上报充电账单；
3. 充电启动失败上传记录；
4. 也可用于充电减免停车费场景。充电完成上传充电记录（车牌必须），平台会判断站点开启【减免停车费】时，匹配充电站对应停车场的优惠规则进行下发。

## 接口地址
- 测试环境：`https://dev-api.4pyun.com/gate/1.0/energy/internal/replenish/sync`
- 生产环境：`https://api.4pyun.com/gate/1.0/energy/internal/replenish/sync`

## 请求方式
`POST`
`application/json; charset=utf-8`

## 请求头
| 参数            |                        说明                        | 必须 | 说明 | 备注 |
|:--------------|:------------------------------------------------:|:--:|----|----|
| Authorization | <a href="https://doc.4pyun.com/openapi/">签名值</a> | Y  |    |    |

## 请求参数
| 参数           |   类型   | 必须 | 说明                                                                             | 备注                                                     |
|:-------------|:------:|:--:|--------------------------------------------------------------------------------|--------------------------------------------------------|
| app_id       | string | Y  | P云平台分配的应用标识                                                                    | 公共参数                                                   |
| timestamp    | int64  | Y  | 当前请求时间戳，毫秒                                                                     | 公共参数                                                   |
| sign_type    | string | Y  | 签名算法，固定值 `MD5`                                                                 | 公共参数                                                   |
| station_uuid | string | Y  | P云平台创建的充电站编号                                                                   |                                                        |
| order        | string | Y  | 本地充电单号                                                                         |                                                        |
| start_time   | string | Y  | 充电开始时间，UTC时间                                                                   | 例如：`2024-04-14T16:00:00.000Z` 表示 `2024-04-15 00:00:00` |
| end_time     | string | Y  | 充电开始时间，UTC时间                                                                   | 例如：`2024-04-14T16:00:00.000Z` 表示 `2024-04-15 00:00:00` |
| vin          | string | N  | 车架号                                                                            |                                                        |
| plate        | string | N  | 车牌号码                                                                           |                                                        |
| quantity     | int32  | Y  | 充电量，单位：0.001Kwh                                                                | 1表示0.001Kwh。例如1度则传递1000                                |
| energy_value | int32  | Y  | 充电电费，单位：分                                                                      |                                                        |
| fee_value    | int32  | Y  | 充电服务费，单位：分                                                                     |                                                        |
| state        | int32  | Y  | <a href="https://doc.4pyun.com/openapi/appendix.html#replenish_state">充电状态</a> |                                                        |
| state_desc   | string | Y  | 充电状态描述                                                                         |                                                        |
| device_no    | string | Y  | 本地设备编号                                                                         |                                                        |
| device_type  | int32  | Y  | <a href="https://doc.4pyun.com/openapi/appendix.html#device_type">设备类型</a>     |                                                        |
| port_no      | string | Y  | 本地枪口编号                                                                         |                                                        |
| energy_code  | string | Y  | <a href="https://doc.4pyun.com/openapi/appendix.html#energy_code">充电类型</a>     |                                                        |
| soc          | int32  | N  | SOC，单位：百分比                                                                     | 例如100则表示百分百                                            |
| mobile       | string | Y  | 手机号码                                                                           | 未收集到时可传递本地用户标识                                         |

## 请求示例
```json
{
    "app_id": "op00961963581daa7",
    "timestamp": 1758093248353,
    "sign_type": "MD5",
    "station_uuid": "49f0cc52-e8c7-41e3-b54d-af666b8cc11a",
    "order": "20250917151408743549",
    "start_time": "2025-09-17T06:14:08.320Z",
    "end_time": "2025-09-17T07:14:08.351Z",
    "vin": "LSTOP103212132001",
    "plate": "川A660N11",
    "quantity": 1000,
    "energy_value": 800,
    "fee_value": 200,
    "state": 3,
    "state_desc": "SOC已充满",
    "device_no": "TEST250911100001",
    "device_type": 0,
    "port_no": "1",
    "energy_code": "CN_AC",
    "soc": 100,
    "mobile": "13800138001"
}
```

## 响应参数
| 参数        |   类型   | 必须 | 说明    | 备注 |
|:----------|:------:|:--:|-------|----|
| code      | string | Y  | 状态码   |    |
| message   | string | N  | 状态说明  |    |
| hint      | string | N  | 提示说明  |    |
| seqno     | string | Y  | 请求序列号 |    |
| data_node | string | N  |       |    |
| time_cost | int32  | N  |       |    |

## 响应示例
```json
{
    "code": "1001",
    "seqno": "72639134177502211385242005141533",
    "data_node": "CN-South/DEV-3",
    "time_cost": 18
}
```

## 状态码
| 值    | 描述                         |
|------|----------------------------|
| 1001 | 成功                         |
| 1500 | 失败                         |
| 其它   | 参考 `message` 内容或 `hint` 提示 |