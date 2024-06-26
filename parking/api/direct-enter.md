# 人工/无牌车入场

**说明:**

为实现无人值守, 针对人工/无牌车通过扫码后平台会调用该接口完成入场开闸，若短时间重复调用该接口则不重复写入入场记录。

**请求参数:**

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| service | string | Y | 服务名: `service.parking.direct.enter` |
| version | string | Y | 版本号:`1.0`|
| charset | string | Y | 字符集:`UTF-8`|
| sign | string | Y | 签名|

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| park_uuid | string | Y | 停车场编号 |
| passport | string | Y | 用户通行证ID, 停车场可用作虚拟卡ID, 出场将传入, 例如: 无A123456 |
| gate_id | string | Y | 通道编号ID |
| plate | string | Y | 月卡虚拟车牌, 和passport等价, 例如: 无A123456 |
| mobile | string | N | 用户手机号 |
| enter_time | string | N | 入场时间, 格式: `yyyyMMddHHmmss`. |
| operator | string | N | 操作员 |
| reason | string | N | 入场说明 |

**应答参数:**

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| service | string | Y | 服务名: `service.parking.direct.enter` |
| version | string | Y | 版本号:`1.0`|
| charset | string | Y | 字符集:`UTF-8`|
| result_code | string | Y | 本次请求状态返回码:<br/>1000 需付费入场<br/>1001  操作成功<br/>1002  未检测到车辆<br/>1403  短时间重复入场<br/>1500  接口处理异常|
| sign | string | Y | 当前请求应答结果, 签名方法参考附录. |
| message | string | Y | 状态码处理描述, 如:返回错误信息. |

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| plate | string | Y | 当前识别到的车牌/虚拟车牌 |
| plate_type | int | N | 车牌类型: 0-中国大陆车牌, 1-境外车牌(含港澳), 2-两轮电动车车牌 |
| plate_color | int | N | 车牌颜色: 1.蓝色, 2.黄色, 3.白色, 4.黑色, 5.绿色, -1 未知 |
| enter_time | string | Y | 入场时间, 格式: `yyyyMMddHHmmss`. |
| enter_gate | string | Y | 入口名称 |
| parking_time | int | Y | 停车时长, 单位s.|
| parking_serial | string | Y | 停车流水. |
| car_type | int | Y | 车型:<br/>1. 临停车辆<br/>2. 月卡车辆<br/>3. 贵宾车辆<br/>4. 储值车辆<br/>0.其他未知 |
| car_desc | string | Y | 车类说明 |
| parking_order | string | N | 需付费情况下的付费订单号, 限制单个车场唯一 |
| pay_value | int | N | 应支付金额, 单位:分|
