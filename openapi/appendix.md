# 附录

## I. 交易场景

|场景值|说明|
|---|---|
|PARKING|停车缴费(临停续费)|
|RECHARGE|停车缴费(固定车续费)|
|ENERGY|充电业务|
|LIVING|生活缴费|
|GENERIC|通用缴费|

### I.I 停车缴费
```
trade_scene=PARKING 场景填写到extra字段
```
| 字段名称 | 字段说明 | 类型 | 必填 | 示例 |
| :--- | --- | :---: | :--: | :--- |
| plate | 支付车牌 | string | Y | 粤TXXXXX |
| plate_color | 车牌颜色: 1.蓝色, 2.黄色, 3.白色, <br>4.黑色, 5.绿色, -1 未知 | string | Y | 1 |
| plate_type | 车牌类型: 0-中国大陆车牌, 1-境外车牌(含港澳车牌), 2-两轮车(电动车/摩托车等) | string | N | 1 |
| card_id | 停车卡ID, 无车牌可传递 | string | N | ABDDS |
| gate_id | 缴费通道ID | string | N | 001 |
| parking_serial | 一次停车唯一ID(和进出场记录推送parking_serial保持一致) | string | Y | XXXXX-ID |
| park_name | 停车场名称 | string | Y | XXXX-停车场 |
| enter_time | 入场时间, 单位毫秒 | string | Y | 1552976318722 |
| parking_time | 停车时长, 单位秒 | string | Y | 3600 |
| free_value | 车场优惠金额, 单位分 | int | Y | 100 |
| receipt | 微信点金计划页面显示按钮为`我要开票`, 1 显示, 其他无效 | string | N | 1 |
| discount | 支付发起方立减金额, 单位分 | int | N | 100 |


### I.II 固定车续费
```
trade_scene=RECHARGE 场景填写到extra字段
```
| 字段名称 | 字段说明 | 类型 | 必填 | 示例 |
| :--- | --- | :---: | :--: | :--- |
| plate | 支付车牌 | string | Y | 粤TXXXXX |
| plate_color | 车牌颜色: 1.蓝色, 2.黄色, 3.白色, <br>4.黑色, 5.绿色, -1 未知 | string | Y | 1 |
| plate_type | 车牌类型: 0-中国大陆车牌, 1-境外车牌(含港澳车牌), 2-两轮车(电动车/摩托车等) | string | N | 1 |
| type | 固定车类型:<br/>1 月卡<br/>2 储值<br/>3 储次<br/>4 储天<br/>5 年卡<br/>6 季度卡<br/>7 半年卡 | string | Y | 1 |
| charge_type | 固定车类型ID | string | Y | MON_001 |
| quantity | 续费数量:<br/>储值-续费金额, 单位分<br/>储次-续费次数<br/>储天-续费天数<br/>月卡-续费月数<br/>年卡-续费年数<br/>半年卡-续费半年数<br/>季度卡-续费季度数 | string | Y | 1 |
|renewal_start_time|时间类续费开始时间, 格式: yyyyMMdd| string | Y | 20230801 |
|renewal_end_time|时间类续费结束时间, 格式: yyyyMMdd| string | Y | 20230831 |
| receipt | 微信点金计划页面显示按钮为`我要开票`, 1 显示, 其他无效 | string | N | 1 |

### I.III 充电业务
```
trade_scene=ENERGY 场景填写到extra字段
```

| 字段名称           | 字段说明                       |   类型   | 必填 | 示例        |
|:---------------|----------------------------|:------:|:--:|:----------|
| plate          | 车牌号码                       | string | N  |           |
| mobile         | 手机号码                   | string | N  |           |
| device_no      | 充电桩号                       | string | Y  |           |
| port_no        | 充电枪号                       | string | Y  |           |
| quantity       | 充电电量(单位:0.001度), 例如1度传递1000     |  int   | Y  |           |
| energy_value   | 电费, 单位: 分                  |  int   | Y  |           |
| service_value  | 服务费, 单位: 分                   |  int   | Y  |           |
| start_time     | 开始充电时间, 格式: yyyy-MM-dd HH:mm:ss | string | Y  |           |
| end_time       | 结束充电时间, 格式: yyyy-MM-dd HH:mm:ss | string | Y  |           |
| replenish_time | 实际充电时长, 单位: 秒         |  int   | Y  |           |
| replenish_type | 充电方式: 1=扫码(默认)、2=刷卡、3=VIN码    |  int   | N  |           |
| energy_code    | 充电类型: CN_AC=交流(默认)、CN_DC=直流    | string | N  |           |
| device_type    | 设备类型: 0=汽车(默认)、1=电瓶车        |  int   | N  |           |
| state          | 充电状态: -1=充电异常、3=充电完成(默认)     |  int   | N  |           |
| state_desc     | 充电状态描述            | string | N  | APP主动停止充电 |


####  I.III.I 充电业务交易事项

| 值    | 描述       |
| ----- | ---------- |
| energy_value | 电费, 单位: 分 |
| service_value | 服务费 单位: 分 |

### I.IV 生活缴费
```
trade_scene=LIVING 场景填写到extra字段
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


## III. 支付渠道定义 <a id="channel">`channel`</a>

| 值      | 描述                                                                           |
|--------|------------------------------------------------------------------------------|
| 100002 | 支付宝APP支付                                                                     |
| 200001 | 微信APP支付                                                                      |
| 200201 | 微信JSAPI支付                                                                    |
| 800072 | 农信支付<br>- 主扫：v2.0主扫支付<br>- 被扫：v2.0被扫支付                                       |
| 800071 | 银联支付<br>- 主扫：银联 【统一】扫码支付<br>- 被扫：...                                         |
| 800001 | 建行支付<br>- 主扫：动态码聚合支付文档1.6<br>- 被扫：互联网被扫支付接口文档V1.5<br> - 无感：建行互联网无感支付接口文档V1.2 |



## IV. 固定车类型 <a id="vip_type">`vip_type`</a>

| 值    | 描述       |
| ----- | ---------- |
| 1 | 月卡 |
| 2  | 储值卡 |
| 3  | 储次 |
| 4  | 储天 |
| 5  | 年卡 |
| 6  | 季度卡 |
| 7  | 半年卡 |

## 颜色定义 <a id="color_type">`color_type`</a>

| 值  | 描述 |
|----|----|
| -1 | 未知 |
| 1  | 蓝色 |
| 2  | 黄色 |
| 3  | 白色 |
| 4  | 黑色 |
| 5  | 绿色 |
| 6  | 红色 |
| 7  | 银色 |
| 8  | 灰色 |
| 9  | 橙色 |

## 车类 <a id="car_type">`car_type`</a>

| 值  | 描述      |
|----|---------|
| -1 | 未知      |
| 1  | 临停车辆    |
| 2  | 包月车辆    |
| 3  | 贵宾/免费车辆 |
| 4  | 储值车辆    |


## 车型 <a id="vehicle_type">`vehicle_type`</a>

| 值  | 描述    |
|----|-------|
| -1 | 未知    |
| 1  | 小型车辆  |
| 2  | 大型车辆  |
| 3  | 超大型车辆 |
| 4  | 中型车辆  |
| 5  | 电动车 |

## <a id="device_type">设备类型</a>

| 值 | 描述        |
|---|-----------|
| 0 | 汽车充电桩     |
| 1 | 两轮电瓶车充电设备 |

## <a id="energy_code">充电类型</a>

| 值     | 描述   |
|-------|------|
| CN_AC | 交流慢充 |
| CN_DC | 直流快充 |

## <a id="device_state">设备状态</a>

| 值  | 描述   |
|----|------|
| 1  | 可用   |
| -1 | 故障   |
| -2 | 离线   |

## <a id="port_state">枪口状态</a>

| 值  | 描述  |
|----|-----|
| 1  | 空闲  |
| 2  | 充电中 |
| 3  | 空占中 |
| -1 | 故障  |

## <a id="space_status">车位状态</a>

| 值  | 描述 |
|----|----|
| -1 | 未知 |
| 1  | 空闲 |
| 2  | 占用 |

## <a id="replenish_state">充电状态</a>

| 值  | 描述        |
|----|-----------|
| 0  | 未充电       |
| -1 | 启动失败，充电异常 |
| 2  | 充电中       |
| 3  | 充电完成      |

## <a id="pay_mode">到账方式</a>

| 值  | 描述      |
|----|---------|
| 99 | 其他第三方渠道 |

## <a id="energy_device_metadata">充电设备元数据</a>

| 值                           | 描述                  |
|-----------------------------|---------------------|
| DEVICE_HARDWARE_VERSION     | 设备硬件版本              |
| DEVICE_SOFTWARE_VERSION     | 设备软件版本              |
| DEVICE_PROTOCOL_VERSION     | 设备通信协议版本            |
| DEVICE_MODEL                | 设备型号                |
| DEVICE_CHARGE_CURRENT       | 设备额定充电电流, 单位0.01A   |
| DEVICE_CHARGE_VOLTAGE       | 设备额定充电电压下限, 单位0.01V |
| DEVICE_CHARGE_VOLTAGE_UPPER | 设备额定充电电压上限, 单位0.01V |
| DEVICE_CHARGE_POWER         | 设备额定充电功率, 单位0.0001W |
| DEVICE_NET_TYPE             | 设备网络类型              |
| DEVICE_SIM_ICCID            | 设备SIM卡ICCID         |
| DEVICE_SIM_OPERATOR         | 设备SIM卡运营商           |

## <a id="energy_replenish_metadata">充电订单元数据</a>

| 值                     | 描述              |
|-----------------------|-----------------|
| ENERGY_CHARGE_CURRENT | 充电电流, 单位0.01A   |
| ENERGY_CHARGE_VOLTAGE | 充电电压, 单位0.01V   |
| ENERGY_CHARGE_POWER   | 充电功率, 单位0.0001W |

