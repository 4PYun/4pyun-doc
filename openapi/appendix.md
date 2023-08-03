# 附录

## I. 交易场景

|场景值|说明|
|---|---|
|PARKING|停车缴费(临停续费)|
|ENERGY|充电业务|
|LIVING|生活缴费|

### I.I 停车缴费
```
trade_scend=PARKING 场景填写到extra字段
```
| 字段名称 | 字段说明 | 类型 | 必填 | 示例 |
| :--- | --- | :---: | :--: | :--- |
| plate | 支付车牌 | string | Y | 粤TXXXXX |
| plate_color | 车牌颜色: 1.蓝色, 2.黄色, 3.白色, <br>4.黑色, 5.绿色, -1 未知 | string | Y | 1 |
| card_id | 停车卡ID, 无车牌可传递 | string | N | ABDDS |
| gate_id | 缴费通道ID | string | N | 001 |
| parking_serial | 一次停车唯一ID | string | Y | XXXXX-ID |
| park_name | 停车场名称 | string | Y | XXXX-停车场 |
| enter_time | 入场时间, 单位毫秒 | string | Y | 1552976318722 |
| parking_time | 停车时长, 单位秒 | string | Y | 3600 |
| free_value | 车场优惠金额, 单位分 | int | Y | 100 |
| receipt | 微信点金计划页面显示按钮为`我要开票`, 1 显示, 其他无效 | string | N | 1 |

### I.II 充电业务
```
trade_scend=ENERGY 场景填写到extra字段
```

| 字段名称 | 字段说明 | 类型 | 必填 | 示例 |
| :--- | --- | :---: | :--: | :--- |
| plate | 车牌 | string | Y | 川A660B1 |
| plate_color | 车牌颜色 | int | Y | 1 |
| device_no | 本地充电桩设备编号 | string | Y | |
| port_no | 本地充电桩设备枪号 | string | Y | |
| vin | 车辆标识 | string | N | |
| energy_code | 充电类型；CN_AC：慢充；CN_DC：快充 | string | Y | CN_AC |

### I.III 生活缴费
```
trade_scend=LIVING 场景填写到extra字段
```

| 字段名称 | 字段说明 | 类型 | 必填 | 示例 |
| :--- | --- | :---: | :--: | :--- |
| room_no | 房号 | string | Y | 12栋-1单元-1301 |

## II. 坐标类型CoordType

| 值    | 描述       |
| ----- | ---------- |
| WGS84 | 地球坐标系 |
| BD09  | 百度坐标系 |
| GCJ02 | 火星坐标系 |
| ...   |            |