# 充电设备同步

## 接口描述
1. 同步设备信息至平台；
2. 设备状态变更时同步至平台等。

## 接口地址
- 测试环境：`https://dev-api.4pyun.com/gate/1.0/energy/internal/device/sync`
- 生产环境：`https://api.4pyun.com/gate/1.0/energy/internal/device/sync`

## 请求方式
`POST`
`application/json; charset=utf-8`

## 请求头
| 参数            |                        说明                        | 必须 | 说明 | 备注 |
|:--------------|:------------------------------------------------:|:--:|----|----|
| Authorization | <a href="https://doc.4pyun.com/openapi/">签名值</a> | Y  |    |    |

## 请求参数
| 参数           |    类型     | 必须 | 说明                                                                          | 备注   |
|:-------------|:---------:|:--:|-----------------------------------------------------------------------------|------|
| app_id       |  string   | Y  | P云平台分配的应用标识                                                                 | 公共参数 |
| timestamp    |   int64   | Y  | 当前请求时间戳，毫秒                                                                  | 公共参数 |
| sign_type    |  string   | Y  | 签名算法，固定值 `MD5`                                                              | 公共参数 |
| station_uuid |  string   | Y  | P云平台创建的充电站编号                                                                |      |
| no           |  string   | Y  | 本地设备编号                                                                      |      |
| name         |  string   | Y  | 设备名称                                                                        |      |
| type         |   int32   | Y  | <a href="https://doc.4pyun.com/openapi/appendix.html#device_type">设备类型</a>  |      |
| state        |   int32   | Y  | <a href="https://doc.4pyun.com/openapi/appendix.html#device_state">设备状态</a> |      |
| state_desc   |  string   | Y  | 设备状态描述                                                                      |      |
| energy_code  |  string   | Y  | <a href="https://doc.4pyun.com/openapi/appendix.html#energy_code">充电类型</a>  |      |
| port         | object[]  | Y  | 枪口数据                                                                        |      |
| metadata     | map<K, V> | Y  | 设备元数据                                                                       |      |

`port`

| 参数           |   类型   | 必须 | 说明                                                                          | 备注         |
|:-------------|:------:|:--:|-----------------------------------------------------------------------------|------------|
|              |        |    |                                                                             |            |
| no           | string | Y  | 本地枪口编号                                                                      | 例如：DA02M01 |
| state        | int32  | Y  | <a href="https://doc.4pyun.com/openapi/appendix.html#port_state">枪口状态</a>   |            |
| state_desc   | string | Y  | 枪口状态描述                                                                      |            |
| alias        | string | Y  | 枪口别名                                                                        | 例如：A       |
| space_status | int32  | N  | <a href="https://doc.4pyun.com/openapi/appendix.html#space_status">车位状态</a> |            |

## 请求示例
```json
{
  "app_id": "op00961963581daa7",
  "timestamp": 1758093419429,
  "sign_type": "MD5",
  "station_uuid": "49f0cc52-e8c7-41e3-b54d-af666b8cc11a",
  "no": "TEST250911100001",
  "name": "测试设备",
  "type": 0,
  "state": 1,
  "state_desc": "可用",
  "energy_code": "CN_AC",
  "metadata": {
    "DEVICE_PROTOCOL_VERSION": "ykc1.6",
    "DEVICE_CHARGE_CURRENT": "22000",
    "DEVICE_CHARGE_VOLTAGE_UPPER": "6400",
    "DEVICE_SOFTWARE_VERSION": "S1.0.2.111",
    "DEVICE_SIM_ICCID": "10021212133123213323",
    "DEVICE_CHARGE_VOLTAGE": "3200",
    "DEVICE_HARDWARE_VERSION": "H1.0.2",
    "DEVICE_CHARGE_POWER": "1200000000"
  },
  "port": [
    {
      "no": "TEST01",
      "state": 1,
      "state_desc": "空闲",
      "alias": "A",
      "space_status": 0
    },
    {
      "no": "TEST02",
      "state": -1,
      "state_desc": "故障",
      "alias": "B",
      "space_status": -1
    }
  ]
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
    "seqno": "04936140153231863496067068140019",
    "data_node": "CN-South/DEV-3",
    "time_cost": 107
}
```

## 状态码
| 值    | 描述                         |
|------|----------------------------|
| 1001 | 成功                         |
| 1500 | 失败                         |
| 其它   | 参考 `message` 内容或 `hint` 提示 |
