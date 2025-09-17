# 充电减免停车费

## 接口描述
用于充电减免停车费场景。充电完成上传充电记录（车牌必须），平台会匹配充电站对应停车场的优惠规则进行下发。

``` sequence
title: 充电减免停车费下发流程

充电桩  -> P云 : 充电完成上报充电记录
P云 -> 停车场  : 根据充电信息下发停车优惠
```

## 注意事项
1. 需在充电结束时触发调用；
2. 一次充电单号唯一，同一个单号重复调用只会发一次减免；
3. 下发停车优惠时 `vin` 需上传正确格式的车牌；
4. 充电开始结束时间为UTC格式，需在北京时间基础上减8小时。例如北京时间 `2023-04-10 12:00:00`，转换为UTC后：`2023-04-10T04:00:00.000Z`；

## 请求地址
- 测试环境：https://dev-api.4pyun.com/gate/1.0/energy/internal/replenish
- 生产环境：https://api.4pyun.com/gate/1.0/energy/internal/replenish

## 请求方式
`POST`
`application/x-www-form-urlencoded`

## 请求参数
| 参数              |    类型    | 必须 | 说明                                               | 备注                     |
|:----------------|:--------:|:--:|--------------------------------------------------|------------------------|
| app_id          |  string  | Y  | P云平台分配的应用标识                                      | 公共参数                   |
| timestamp       |  int64   | Y  | 当前请求时间戳，毫秒                                       | 公共参数                   |
| sign            |  string  | Y  | <a href="https://doc.4pyun.com/openapi/">签名值</a> | 公共参数                   |
| sign_type       |  string  | Y  | 签名算法，固定值 `MD5`                                   | 公共参数                   |
| station_uuid    |  string  | Y  | PP充电平台充电站UUID（需先在PP平台创建充电站才可获得）                  |                        |
| device_no       |  string  | Y  | 本地设备编号（传递本地设备编号即可）                               | DEVICE1                |
| port_no         |  string  | Y  | 本地设备枪号（传递本地设备枪号即可）                               | 1                      |
| replenish_order |  string  | Y  | 本地充电单号                                           |                        |
| start_time      | datetime | Y  | 开始充电，`yyyy-MM-dd'T'HH:mm:ss'Z'`                  | 2023-04-10T17:37:30Z   |
| end_time        | datetime | Y  | 结束充电，`yyyy-MM-dd'T'HH:mm:ss'Z'`                  | 2023-04-10T17:37:30Z   |
| vin             |  string  | Y  | 车架号                                              | 充电停车优惠场景下该值必须传递停车记录的车牌 |
| plate           |  string  | Y  | 车牌号码                                             | 川A660PP                |
| quantity        |  int32   | Y  | 充电度数，0.001Kwh，1表示0.001Kwh，例如1度传递1000即可           | 1000                   |
| energy_value    |  int32   | Y  | 电费，分                                             | 1                      |
| fee_value       |  int32   | Y  | 服务费，分                                            | 1                      |
| total_value     |  int32   | Y  | 总金额，分                                            | 2                      |
| energy_code     |  string  | Y  | 充电类型；CN_AC：慢充；CN_DC：快充                           | CN_AC                  |
| ...             |          |    |                                                  |                        |

## 响应参数
`无`

## 状态码
| 值    | 描述                         |
|------|----------------------------|
| 1001 | 成功                         |
| 1002 | 停车记录不存在                    |
| 其它   | 参考 `message` 内容或 `hint` 提示 |