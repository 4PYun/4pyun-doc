
# P云停车场接入

> 技术支持: dev@660pp.com<br/>
> 编写人: D.T.

```text
2018-05-23 新增全免优惠券以及不同计价券定义
2019-05-13 新增年卡、季度卡定义
2019-05-16 新增半年卡定义
2019-06-18 下发优惠新增优惠券名称
2019-07-10 完善无牌车出入场参数
2019-10-16 新增长期免费券
2019-10-31 固定车返回信息新增车主地址、车位号信息
2019-11-01 完善固定车续费接口
2019-11-09 新增支付来源
2019-12-20 新增对付费入场对支持
2020-01-05 获取订单支持返回`car_type`、`car_desc`字段用于特殊固定车
2020-03-31 新增预约停车流程
2020-04-11 定义车场折扣优惠券
2020-04-13 获取订单接口可返回月卡续费天数
2020-04-16 新增黑名单及固定车车辆管理
2020-04-30 新增车场设备列表
2020-05-17 无感同步新增`deduct_mode`参数
```

## 目录

[TOC]

## 1. 目的

本文档主要介绍了停车场系统如何和P云完成对接以实现移动支付流程, 并介绍了相关接口参数规定.

## 2 准备
### 2.1 技术对接方案
针对停车场的网络环境, P云提供多种对接方案
#### 2.1.1 已有一组可用的http服务
可以使用HTTP的方式进行对接. 由P云这边进行开发对接.

#### 2.1.2 还没有相关的HTTP服务
##### - SDK
道闸可以使用SDK, 根据本文档进行开发。

##### - 代理程序
使用P云的代理程序(PPProxy.zip)间接进行网络通信. 根据本文档进行开发.

### 2.2 术语说明
#### 2.2.1 上行与下行
上行: 指停车场系统主动请求P云

上行接口直接参考: https://api.4pyun.com/swagger-ui.html


下行: 指P云主动请求停车场系统
如果没有特殊说明, 本文的接口都是下行接口.

## 3. 对接需求概述
| 功能 | 功能概述 | 是否必须 | 接口数量 |
|:---|:---|:---|:---|
| 移动支付(包括自动支付) | 获取停车费用，并完成支付流程 | Y | 8 |
| 商户优惠券 | 商家给车主下发停车优惠券;车主支付时可使用该优惠券 | N | 2 |
| 月卡/储值卡/次卡 | 续费 | N | 2 |

### 3.1 移动支付

主要对接需求有:**获取停车支付订单**, **订单支付结果通知**, **同步车辆列表(下行)** 和 **设置车辆自动支付权限(下行)** 等. 进一步介绍见下表:

| 接口 | 接口概述 | 是否必须 |
|:---|:---|:---|
| 获取停车订单 | 获取停车费用,返回基本停车信息、优惠信息、已支付金额以及需支付金额. | Y |
| 订单支付结果通知 | 完成支付后，通过该接口完成订单同步. | Y |
| 停车场实时数据拉取 | 获取停场的车辆数和总车位等数据 | Y |
| 车辆自动支付权限开关 | 当车辆入场，由P云主动下发车辆享受后付费出场额度,当额度内，车辆出场直接放行；超过额度则取消本次预付费标记；P云根据出场纪录中返回的预付费实际支付额度完成用户扣款. | Y |
| 车辆出场锁定或解锁 | 用户可锁定车辆, 锁定后, 车辆不可离场 | N |
| 获取停车记录详情 | a. 停车信息: 出入场时间、停车时长、收费金额、优惠信息、收费员信息、出入场图片地址.<br/> b. 收费信息: 本地停车所有成功的收费记录，需要区分支付方式.| Y |
| 无牌车入场 | 用户扫描入场二维码后, P云会调用该接口完成入场开闸 | Y |

### 3.2 商户优惠券

大型商场都具有多种优惠, 面对种类繁多的优惠类型, P云为商场提供纯电子化优惠券的整体解决方案.

| 接口 | 接口概述 | 是否必须 |
|:---|:---|:---|
| 电子优惠券派发 | P云通过该接口派发电子优惠到具体某次停车 | Y |
| 电子优惠撤销 | 当车辆未出场前，电子优惠可被撤销. | Y |

### 3.3 月卡/次卡/储值卡
P云为停车场提供月卡/次卡/储值卡续费的移动支付方案.

| 接口 | 接口概述 | 是否必须 |
|:---|:---|:---|
| 车辆信息查询 | 提供根据车牌/客户信息查询在当前停车场办理月卡/储值/其他卡类信息(收费等信息). | Y |
| 车辆续费通知 | 用户完成续费后通过该接口通知停车场客户续费结果；例如：续费30天/充值100块. | Y |

## 4 移动支付接口详情
停车场在开发过程中应保证数据的正确有效;
在处理订单支付结果通知的时候应<font color='red'>**过滤重复通知**</font>(因为P云为保证支付结果通知到位, 可能发起多次支付结果通知！)以免订单的多次支付.

**自动支付注意事项:**
- P云会提供当前用户可用额度, 如果出场时停车费用大于用户可用额度, 应拒绝放行.
- 停车场系统需标记该车辆当次停车费由P云代收, 方便后续对账.
- 自动支付权限仅**当次**停车有效, 若用户出场后, 重新入场无效！
- 开通自动支付权限时需验证车辆状态, 如果车辆已出场应返回设置失败.
- 取消车辆自动支付权限时需验证车辆状态, 如果车辆已出场应返回设置失败.
- 若车辆自动支付权限已取消, 则车辆出场不放行(使用其他支付方式).
- 若重复受到开通自动支付权限, 停车场应更新当前用户自动支付信用额度等相关信息.
- 当停车场检测到当前用户信用额度不足, 应自动取消自动支付权限.

### 4.1 获取停车订单

**协议说明:**

当用户在P云场时获取停车订单时, P云将发送该协议请求到停车场系统, 通过车牌号/停车卡ID获取停车支付订单;

**付费入场**

当付费入场当前接口传递参数`gate_id`对应车场系统通道类型为入口, 厂商自行判断是否允许付费入场, 若不允许直接返回没有订单或返回错误信息即可; 反之直接返回付费入场的费用。
<p style="color:red">若为无牌车入场, 虚拟车牌会在支付结果同步时传递给车场系统。</p>


**时段月卡/多车共位**

在多月共位/时段月卡的场景, 可返回`car_type`, 用于当非临时车辆时(`car_type!=1`), 前端可选择跳转固定车续费或直接缴临停费用。


**请求:**

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| service| string | Y | 服务名: `service.parking.payment.billing`|
| version| string | Y | 版本号:`1.0`|
| charset| string | Y | 字符集:`UTF-8`|
| sign | string | Y | 签名|

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
|park_uuid | string | Y | 停车场编号, 由P云分配 |
|plate | string | N | 支付车牌号 |
|card_id | string | N | 支付停车卡物理ID |
| passport | string | N | 用户通行证ID, 无牌车传入 |
|gate_id | string | N | 通道编号ID, 无牌车付费时可传递触发开闸. |

**注:** `card_id`, `plate` 和`passport`为互斥参数. 若传递`card_id`则停车场应该根据停车卡查询停车信息返回停车订单;若传递`plate`停车场应根据车牌号返回停车订单; 而passport则是P云为无牌车生成的一个唯一标识.

*请求示例*
```json
{
  "charset":"UTF-8",
  "park_uuid":"aaaaaaa-ec98-46be-89e3-26bca7be833e",
  "plate":"粤B660PP",
  "service":"service.parking.payment.billing",
  "version":"1.0"
}
```

**应答:**

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
|service | string | Y | 服务名: `service.parking.payment.billing`|
| version| string | Y | 版本号:`1.0`|
| charset| string | Y | 字符集:`UTF-8`|
|result_code | string | Y | 状态码:<br/>值 含义<br/>1001  订单获取成功, 业务参数将返回.<br/>1002  未查询到停车信息.<br/>1003  月卡车辆, 不允许支付.<br/>1401  签名错误, 请检查配置.<br/>1500  接口处理异常. |
|message | string | Y | 状态码处理描述, 如:返回错误信息.|
| sign | string | Y | 签名|

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| plate | string | N | 识别车牌号码.|
| card_id | string | N | 无牌车获取订单返回本地的虚拟卡ID/虚拟车牌.|
| parking_serial | string | Y | 停车流水, 标识具体某次停车事件, 需保证该停车场下唯一, 付费入场可不返回。|
| parking_order | string | Y | 停车支付订单号, 需保证该停车场下唯一.注:同一停车场内不可重复！|
| enter_time | string | Y | 入场时间, 格式`yyyyMMddHHmmss`.|
| parking_time | int | Y | 停车时长(单位秒)|
| total_value | int | Y | 总停车费用(单位分), 为用户从入场到现在获取订单时的总费用.|
| free_value | int | Y | 已优惠金额(单位分), 为停车场在当前停车费用时已经给予的优惠金额, 如果包涵优惠时间则该值为则free_value填写该时间等价的优惠金额+其他有效优惠金额.|
| paid_value | int | Y | 已支付金额(单位分), 为当次停车用户已经支付的金额, 比如当用户先支付了一笔后, 超时未出场重新查询订单时须返回以支付金额.|
| pay_value | int | Y | 应支付金额(单位分), 这里停车场系统需处理如果结果为负数的情况直接返回无需支付.|
| enter_free_time | int | N | 入场免费时间(单位秒)|
| buffer_time | int | N | 当前系统预留出场时间(单位秒)|
| parking_number | string | N | 车辆所在位置信息, 例如: B660.|
| pay_type | int | N | 付费类型: 6 付费入场, 其他无效 |
| car_type | int | N | 车型:<br/>1.临停车辆<br/>2.非临时车 |
| car_desc | string | N | 车型类型说明 |
| recharge_expire_days | int | N | 允许过期续费天数: <br/>`<0` 不支持月卡续费<br/>`=0` 支持月卡续费不支持过期续费<br/>`>0`表示允许过期x天续费<br/>`=7`表示允许过期7天内续费|

*应答示例*
```json
{
  "buffer_time":"1320",
  "charset":"UTF-8",
  "enter_free_time":"1860",
  "enter_time":"20181130100904",
  "free_value":"0",
  "paid_value":"0",
  "parking_order":"2018113010535820181130100904034792608",
  "parking_serial":"20181130100904034792608",
  "parking_time":"2694",
  "pay_value":"500",
  "plate":"粤B660PP",
  "result_code":"1001",
  "service":"service.parking.payment.billing",
  "sign":"73DF19DA361CB673DD3FEFC7A6135BE6",
  "total_value":"500",
  "version":"1.0"
}
```

### 4.2 订单支付结果通知

**协议说明:**

当用户完成支付后, P云将主动发起支付结果通知, 通知客户端订单支付结果.

注:
 1. 为保证通知正常处理, 服务端可能发起多次支付结果通知, 客户端需做好去重逻辑.<br/>
 2. 对于已经受到支付结果通知的订单, 应应答通知成功, 已告知服务端不必继续通知.
</font>

**请求参数:**

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| service | string | Y | 服务名: `service.parking.payment.result` |
| version | string | Y | 版本号:`1.0`|
| charset | string | Y | 字符集:`UTF-8`|
| sign | string | Y | 签名|

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| park_uuid | string | Y | 停车场编号 |
| parking_serial | string | Y | 停车流水, 原车场返回, 付费入场可能为空!|
| parking_order | string | Y | 停车支付订单号, 原车场返回. |
| gate_id | string | N | 通道编号ID, 付费时可传递触发开闸. |
| plate | string | N | 车牌号码, 当无牌车付费入场时为虚拟车牌. |
| pay_serial | string | Y | P云支付流水, 对账可用. |
| pay_time | string | Y | 支付时间, 格式:`yyyyMMddHHmmss`. |
| value | int | Y | 支付金额(单位分) . |
| pay_origin | int | Y | 值 含义<br/>0    P云<br/>4   支付宝<br/>8    微信<br/>-     兼容其他未定义|
| pay_origin_desc | string | Y | 支付到账说明, 例如:P云|
| pay_source | string | N | 付款来源, 例如: 微信/支付宝/银联 |
| autopay_type | short | N | 自动扣费类型: 0非自动扣费, 1先出后扣, 2先扣后出 |

*请求示例*
```json
{
  "charset":"UTF-8",
  "park_uuid":"3bad72c0-6204-4b65-90aa-4323ddd1fb5a",
  "parking_order":"201811301049350025",
  "parking_serial":"8c598afd-2cb8-4cec-8f73-4fd4391d2fc6",
  "pay_origin":"4",
  "pay_origin_desc":"支付宝",
  "pay_source": "支付宝",
  "pay_serial":"20181130105240075500112137",
  "pay_time":"20181130105250",
  "service":"service.parking.payment.result",
  "sign":"E7D7371549C2B4E3AE38C3A028973020",
  "value":"500",
  "version":"1.0"
}
```

**应答参数:**

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| service | string | Y | 服务名: `service.parking.payment.result` |
| version | string | Y | 版本号:`1.0`|
| charset | string | Y | 字符集:`UTF-8`|
| result_code | string | Y | 状态码<br/>值 含义<br/>1001  接口处理成功.<br/>1401  签名错误, 请检查配置.<br/>1403  订单已撤销.1.车辆出场2.参考7章节<br/>1500  接口内部处理失败. |
| sign | string | Y | 签名|
| message | string | Y | 状态码处理描述, 如:返回错误信息. |

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| parking_serial | string | N | 付费入场返回实际停车流水! |

*应答示例*
```json
{
  "charset":"UTF-8",
  "message":"订单支付成功",
  "result_code":"1001",
  "service":"service.parking.payment.result",
  "sign":"F97B3D16E739C82C1DED0450775905CF",
  "version":"1.0"
}
```

### 4.3 停车场实时数据(停车位等)拉取
**协议说明:**

P云将以固定频率调用该接口获取总停车位和场中车辆等数据的获取.

**请求参数:**

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| service | string | Y | 服务名: `service.parking.realtime` |
| version | string | Y | 版本号:`1.0`|
| charset | string | Y | 字符集:`UTF-8`|
| sign | string | Y | 签名|

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| park_uuid | string | Y | 停车场编号 |

**应答参数:**

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| service | string | Y | 服务名: `service.parking.realtime` |
| version | string | Y | 版本号:`1.0`|
| charset | string | Y | 字符集:`UTF-8`|
| result_code | string | Y | 本次请求状态返回码, 示例:1001<br/>状态码  含义<br/>1001  获取成功, 业务参数将返回.<br/>1401  签名错误, 请检查配置.<br/>1500  接口处理异常. |
| sign | string | Y | 当前请求应答结果, 签名方法参考附录. |
| message | string | Y | 状态码处理描述, 如:返回错误信息. |

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| total | int | Y | 停车场总车位数. |
| parking | int | Y | 当前场中车辆数量. |

### 4.4 车辆自动支付权限开关

**协议说明:**

当用户满足自动支付条件时, P云主动向停车场系统发起设置车辆自动支付权限, 停车场可以根据`credits`在车场离场时选择是否触发主动扣费实现无感停车。

**请求参数:**

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| service | string | Y | 服务名: `service.parking.vip` |
| version | string | Y | 版本号:`1.0`|
| charset | string | Y | 字符集:`UTF-8`|
| sign | string | Y | 签名|

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| park_uuid | string | Y | 停车场编号 |
| credits | int | Y | 当前用户可用信用额度(分),  credits大于零则开启. |
| parking_serial | string | Y | 停车流水, 标识具体某次停车事件, 需保证该停车场下唯一. |
| pay_origin | int | N | 值    含义<br/>0   P云<br/>4   支付宝<br/>8   微信<br/>-    兼容其他未定义 |
| pay_origin_desc | string | N | 支付来源说明, 例如:P云 |
| pay_source | string | N | 付款来源, 例如: 微信/支付宝/银联 |
| locking | int | Y | 锁定车辆, 解锁后可出场:1锁定, 0不锁;|
| deduct_mode | string | N | 扣款模式 |

**应答参数:**

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| service | string | Y | 服务名: `service.parking.vip` |
| version | string | Y | 版本号:`1.0`|
| charset | string | Y | 字符集:`UTF-8`|
| result_code | string | Y | 本次请求状态返回码, 示例:1001<br/>状态码  含义<br/>1001  接口处理成功, 业务参数将返回.<br/>1002  停车记录不存在<br/>1401  签名错误, 请检查配置.1403     车辆已出场.<br/>1500  接口处理异常. |
| sign | string | Y | 当前请求应答结果, 签名方法参考附录. |
| message | string | Y | 状态码处理描述, 如:返回错误信息. |

### 4.5 车辆出场锁定或解锁

**协议说明:**

该协议主要用于车牌识别停车场, 当用户设置车辆安全锁定后, 用户离场前主动触发解锁.

**请求参数:**

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| service | string | Y | 服务名: `service.parking.locking` |
| version | string | Y | 版本号:`1.0`|
| charset | string | Y | 字符集:`UTF-8`|
| sign | string | Y | 签名|

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| park_uuid | string | Y | 停车场编号|
| parking_serial | string | Y | 停车流水, 标识具体某次停车事件, 需保证该停车场下唯一.|
| status | int | Y | 锁车状态: 1锁定, 0解锁|

**应答参数:**

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| service | string | Y | 服务名: `service.parking.locking` |
| version | string | Y | 版本号:`1.0`|
| charset | string | Y | 字符集:`UTF-8`|
| result_code | string | Y | 本次请求状态返回码, 示例:1001<br/>状态码  含义<br/>1001  接口处理成功, 业务参数将返回.<br/>1002  停车记录不存在<br/>1401  签名错误, 请检查配置.<br/>1403  车辆已出场.<br/>1400  参数错误, 请检查参数.<br/>1500  接口处理异常. |
| sign | string | Y | 当前请求应答结果, 签名方法参考附录. |
| message | string | Y | 状态码处理描述, 如:返回错误信息. |

### 4.6 获取停车记录详情

**协议说明:**

该协议主要用于处理异常订单, 方便P云拉取订单详情, 已方便对账以及异常订单处理.

注:最近出入场车辆查询(4.4)也返回相应停车信息以及支付信息.

**请求参数:**

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| service | string | Y | 服务名: `service.parking.detail` |
| version | string | Y | 版本号:`1.0`|
| charset | string | Y | 字符集:`UTF-8`|
| sign | string | Y | 签名|

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| park_uuid | string | Y | 停车场编号 |
| parking_serial | string | Y | 停车流水 |

**应答参数:**

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| service | string | Y | 服务名: `service.parking.detail`|
| version | string | Y | 版本号:`1.0`|
| charset | string | Y | 字符集:`UTF-8`|
| result_code | string | Y | 本次请求状态返回码, 示例:1001<br/>状态码  含义<br/>1001  获取成功, 业务参数将返回.<br/>1002  未查询到停车信息.<br/>1401  签名错误, 请检查配置.<br/>1500  接口处理异常. |
| sign | string | Y | 当前请求应答结果, 签名方法参考附录. |
| message | string | Y | 状态码处理描述, 如:返回错误信息. |

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| plate | string | N | 车牌号码 |
| card_no | string | N | 支付停车卡数字编号. |
| card_id | string | N | 支付停车卡物理ID, 停车场系统识别的. |
| car_type | int | N | 车型:<br/>1. 临停车辆<br/>2. 月卡车辆<br/>3. 贵宾车辆<br/>4. 储值车辆<br/>0.其他未知 |
| car_desc | string | N | 车型类型说明 |
| vehicle_type | int | N | 车型: 1 小车, 2大车, -1未知 |
| parking_serial | string | Y | 停车流水, 标识具体某次停车事件, 需保证该停车场下唯一. |
| enter_time | string | Y | 入场时间, 格式`yyyyMMddHHmmss`. |
| enter_image | string | Y | 车辆入场照片访问地址. |
| enter_gate | string | Y | 车辆入场闸口标识. |
| enter_security | string | Y | 入场值班人员姓名. |
| parking_time | int | Y | 停车时长(单位秒) . |
| free_value | int | N | 已优惠金额(单位分) . |
| leave_time | string | N | 出场时间, 格式:`yyyyMMddHHmmss`. |
| leave_image | string | N | 车辆出场照片访问地址. |
| leave_gate | string | N | 车辆出场闸口标识. |
| leave_security | string | N | 出场值班人员姓名. |
| total_value | int | Y | 总停车费用[应收]\(单位分\) |
| payments | array | Y | 支付记录列表(包括现金支付和移动支付) |

**支付信息:**

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| parking_order | string | N | 停车支付订单号, 现金支付无需返回. |
| pay_serial | string | Y | 平台支付流水, 现金支付无需返回. |
| pay_type | int | Y | 支付类型: <br/>1 现金<br/>2 场中支付<br/>3 自动支付<br/>4 储值卡余额支付 |
| pay_time | string | Y | 支付时间, 格式: `yyyyMMddHHmmss`. |
| value | int | Y | 支付金额(单位分) . |

### 4.7 无牌车入场

**说明:**

为实现无人值守, 针对无牌车通过扫码后平台会调用该接口完成入场开闸.若短时间重复调用该接口则不重复写入入场记录.

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

**应答参数:**

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| service | string | Y | 服务名: `service.parking.direct.enter` |
| version | string | Y | 版本号:`1.0`|
| charset | string | Y | 字符集:`UTF-8`|
| result_code | string | Y | 本次请求状态返回码, 示例:1001<br/>状态码  含义<br/>1001  操作成功.<br/>1002  未检测到车辆.<br/>1401  签名错误, 请检查配置.<br/>1403  短时间重复入场.<br/>1500  接口处理异常. |
| sign | string | Y | 当前请求应答结果, 签名方法参考附录. |
| message | string | Y | 状态码处理描述, 如:返回错误信息. |

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| enter_time | string | Y | 入场时间, 格式: `yyyyMMddHHmmss`. |
| enter_gate | string | Y | 入口名称 |
| parking_time | int | Y | 停车时长, 单位s.|
| parking_serial | string | Y | 停车流水. |
| car_type | int | Y | 车型:<br/>1. 临停车辆<br/>2. 月卡车辆<br/>3. 贵宾车辆<br/>4. 储值车辆<br/>0.其他未知 |
| car_desc | string | Y | 车类说明 |

## 5 商户优惠券接口
商户优惠根据发放到停车系统后, 若未清除则本次停车均有效;出场时停车场系统应自动查询出优惠金额、已支付金额, 来判断是否需要收取现金.
### 5.1 发放商户优惠券

**协议说明:**

P云通过和停车场系统完成优惠对接, 将停车场优惠券实现纯电子化.
`无论用户移动支付或现金支付, 停车场系统均可读取到本次停车关联的停车优惠并自动抵扣停车费.`

**请求参数:**

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| service | string | Y | 服务名: `service.parking.discount.create` |
| version | string | Y | 版本号:`1.0`|
| charset | string | Y | 字符集:`UTF-8`|
| sign | string | Y | 签名|

| 字段   | 类型 | 必须 | 说明   |
| --- | --- | -- | ---|
| park_uuid | string | Y | 停车场编号|
| parking_serial | string | N | 停车流水, 标识具体某次停车事件, 需保证该停车场下唯一.      |
| plate | string | N | 忽略停车状态派发时传递        |
| grant_serial | string | Y | 优惠券派发流水.  |
| type | int | Y | 优惠类型: 1金额, 2时长, 3全免, 4不同计价券, 5长期免费券, 6折扣券 |
| value | int | Y | 优惠金额:<br/>当`type=1`时单位为分;<br/>当`type=2`时单位为分钟;<br/>当`type=3`时暂无定义;<br/>当`type=4`时为停车场计价规则ID;<br/>当`type=5`时为在派发后单位分钟后重复进出均可免费停车;<br/>当`type=6`时取值范围1-1000, 1表示0.1%, 800表示80%. |
| reason | string | N | 优惠给予原因, 例如:购物满300, 免费停车2小时.        |
| store_name | string | Y | 当前派发优惠的商家名称.       |
| store_code | string | Y | 当前派发优惠的商家唯一标识.     |
| coupon_name | string | Y | 优惠券名称 |
| coupon_code | string | Y | 平台优惠券ID |
| coupon_rule | string | N | 优惠券减免规则ID, 用于配置减免是减免前面还是减免后面一般一个停车场配置相同.       |
| charge_rule | string | N | 不同计价券收费规则ID, 用于配置改优惠券后本次停车使用不同计价规则, 一般每种优惠券不同.          |

**应答参数:**

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| service | string | Y | 服务名: `service.parking.discount.create` |
| version | string | Y | 版本号:`1.0`|
| charset | string | Y | 字符集:`UTF-8`|
| result_code | string | Y | 本次请求状态返回码, 示例:1001<br/>状态码  含义<br/>1002  停车信息未找到.<br/>1403  当前车辆已享受其他优惠.<br/>1401  签名错误, 请检查配置.<br/>1500  接口处理异常. |
| sign | string | Y | 当前请求应答结果, 签名方法参考附录. |
| message | string | Y | 状态码处理描述, 如:返回错误信息. |

注:
1. 一次停车在停车系统只允许存在一条有效优惠信息.
2. 优惠提交到停车场系统后, 默认立即生效, 且仅本次停车有效.
3. 该商户优惠给予后, 在获取停车订单时, 接口应返回相应的优惠信息.
4. 若提交enfore给予优惠券, 则本次停车已有的优惠使用本次提交的优惠替换.
5. 接口提交的voucher用于消费对账, 不用于账单核销, 停车场系统实现可作为优惠券唯一标识.

### 5.2 撤销商户优惠

**协议说明:**

用于在优惠发放错误是, 清空指定停车记录已发放的商户优惠.
1. 是否是在特定时间内才可以撤回还是任意时间都能撤回？
这样有点不合理, 如果车主离开商场了, 出场时发现优惠券没了, 这时候会引起争议的;嗯, 那是否限制发出去后10分钟后不可撤销.
2. 如果发出去5分钟内车主离场了, 但是6分钟后才发觉发错了, 这时候撤不回怎么办？
如果发现已经出场, 那么你们返回无法撤销, 车辆已出场.

**请求参数:**

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| service | string | Y | 服务名: `service.parking.discount.destroy`|
| version | string | Y | 版本号:`1.0`|
| charset | string | Y | 字符集:`UTF-8`|
| sign | string | Y | 签名|

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| park_uuid | string | Y | 停车场编号 |
| grant_serial | string | Y | 优惠券派发流水.|


**应答参数:**

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| service | string | Y | 服务名: `service.parking.discount.destroy` |
| version | string | Y | 版本号:`1.0`|
| charset | string | Y | 字符集:`UTF-8`|
| result_code | string | Y | 本次请求状态返回码, 示例:1001<br/>状态码  含义<br/>1002  停车信息未找到.<br/>1401  签名错误, 请检查配置.<br/>1500  接口处理异常. |
| sign | string | Y | 当前请求应答结果, 签名方法参考附录. |
| message | string | Y | 状态码处理描述, 如:返回错误信息. |

## 6 固定车接口

### 6.1 车辆信息查询

**协议说明:**

P云发起向停车场, 查询指定车辆固定车信息, 停车场返回对应车牌/客户信息。

**请求参数:**

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| service | string | Y | 服务名: `service.parking.vip.query` |
| version | string | Y | 版本号:`1.0`|
| charset | string | Y | 字符集:`UTF-8`|
| sign | string | Y | 签名|

| 字段 | 类型 | 必须 | 说明|
| --- | --- | -- | --- |
| park_uuid | string | Y | 停车场编号 |
| plate | string | Y | 绑定车牌号码. |

**应答参数:**

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| service | string | Y | 服务名: `service.parking.vip.query` |
| version | string | Y | 版本号:`1.0`|
| charset | string | Y | 字符集:`UTF-8`|
| result_code | string | Y | 本次请求状态返回码, 示例:1001<br/>状态码  含义<br/>1001  接口处理成功, 业务参数将返回.<br/>1002  没有查到相关固定车记录.<br/>1401  签名错误, 请检查配置.<br/>1500  接口处理异常. |
| sign | string | Y | 当前请求应答结果, 签名方法参考附录. |
| message | string | Y | 状态码处理描述, 如:返回错误信息. |

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| vips | array | N | 固定车集合, 对象参考后面固定车信息. |

**固定车信息:**

| 字段 | 类型 | 必须 | 说明    |
| --- | --- | --- | ---|
| realname | string | Y | 客户姓名. |
| plate | string | Y | 绑定车牌号码.|
| card_no | string | N | 停车卡编号(若存在)   |
| card_id | string | N | 停车卡ID(若存在).  |
| charge_type | string | N | 收费标准类型标识; 例如:MON_A、12.|
| charge_desc | string | N | 当前固定车收费标准描述;例如: 月卡B.   |
| charge_price | int | N | 收费单价, 单位分.   |
| mobile | string | Y | 绑定的手机号.|
| id_card | string | Y | 绑定的身份证号.     |
| create_time | string | Y | 办理时间, 格式: `yyyyMMdd`  |
| expire_time | string | Y | 过期时间, 格式: `yyyyMMdd`  |
| type| int | Y | 固定车类型:<br/>1 月卡<br/>2 储值<br/>3 储次<br/>4 储天<br/>5 年卡<br/>6 季度卡<br/>7 半年卡<br/>0 免费停车 |
| balance | int | Y | 当前余额<br/>储值: 为余额, 单位分;<br/>储次:为剩余次数;<br/>储天:为剩余天数;<br/>月卡/年卡/半年卡/季度卡:为剩余天数.|
| address | string | N | 车主单位/住宅 |
| park_space_no | string | N | 车位号 |

<font color='red'>注: 若月卡当前到期则balance=0, 若昨天过期balance=-1, 按照此规则计算</font>

### 6.2 车辆续费通知

**协议说明:**
P云发起请求, 对固定车进行充值续费。

**请求参数:**

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| service | string | Y | 服务名: `service.parking.vip.renewal` |
| version | string | Y | 版本号:`1.0`|
| charset | string | Y | 字符集:`UTF-8`|
| sign | string | Y | 签名|

| 字段 | 类型 | 必须 | 说明 |
| --- | --- | --- | ---|
| park_uuid | string | Y | 停车场编号|
| plate | string | N | 绑定车牌号码.     |
| card_no | string | N | 停车卡编号(若存在)  |
| card_id | string | N | 停车卡ID(若存在). |
| pay_time | string | Y | 续费时间, 格式: `yyyyMMddHHmmss`|
| pay_serial | string | Y | 支付流水, 用于对账. |
| pay_value | int | Y | 客户支付金额, 单位分.|
| type| int | Y | 固定车类型:<br/>1 月卡<br/>2 储值<br/>3 储次<br/>4 储天<br/>5 年卡<br/>6 季度卡<br/>7 半年卡<br/>0 免费停车  |
| value | int | Y | 续费金额:<br/>储值: 续费金额, 单位分;<br/>储次:续费次数;<br/>储天:续费天数;<br/>月卡/年卡/半年卡/季度卡: 续费天数. |
| quantity | int | Y | 续费数量:<br/>储值: 续费金额, 单位分;<br/>储次:续费次数;<br/>储天:续费天数;<br/>月卡:续费月数;<br/>年卡:续费年数;<br/>半年卡:续费半年数;<br/>季度卡:续费季度数.|
| pay_origin | int | Y | 值 含义<br/>0    P云<br/>4   支付宝<br/>8    微信<br/>-     兼容其他未定义 |
| pay_origin_desc | string | Y | 支付来源说明, 例如:P云 |
| pay_source | string | N | 付款来源, 例如: 微信/支付宝/银联 |
| renewal_start_time | string | Y | 时间类续费开始时间, 格式: `yyyyMMddHHmmss` |
| renewal_end_time | string | Y | 时间类续费结束时间, 格式: `yyyyMMddHHmmss` |

**说明**

1. 实现该接口如果是时间类固定车，建议直接按照`renewal_start_time`~`renewal_end_time`插入新的有效时间, 可实现过期续费; 例如：月卡A当前有效期为`2019-01-01~2019-10-31`:

- 若在`2019-10-31`之前续费一个月，则接口下发的时间范围为: `2019-11-01～2019-11-30`,
- 若在`2019-11-05`续费, 月卡已过期，若停车场允许过期7天内续费，则续费后有两种时间计算方式：
  - **从过期时间开始** - 则接口下发时间范围为: `2019-11-01~2019-11-30`;
  - **从续费时间开始计算** - 则接口下发时间范围为: `2019-11-05~2019-12-05`。

2. 车场本地其他固定车计算延期方式不是固定的可根据下发的参数自己选择适合自己的计算方式。

**应答参数:**

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| service | string | Y | 服务名: `service.parking.vip.renewal` |
| version | string | Y | 版本号:`1.0`|
| charset | string | Y | 字符集:`UTF-8`|
| result_code | string | Y | 本次请求状态返回码, 示例:1001<br/>状态码  含义<br/>1001  接口处理成功, 业务参数将返回.<br/>1002  没有查到相关固定车记录.<br/>1401  签名错误, 请检查配置.<br/>1500  接口处理异常. |
| sign | string | Y | 当前请求应答结果, 签名方法参考附录. |
| message | string | Y | 状态码处理描述, 如:返回错误信息. |

### 6.3 新增黑名单车辆

**协议说明:**
P云发起请求, 将指定车辆设置为车场黑名单

**请求参数:**

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| service | string | Y | 服务名: `service.parking.blacklist.create` |
| version | string | Y | 版本号:`1.0`|
| charset | string | Y | 字符集:`UTF-8`|
| sign | string | Y | 签名|

| 字段 | 类型 | 必须 | 说明 |
| --- | --- | --- | ---|
| park_uuid | string | Y | 停车场编号|
| plate | string | Y | 车牌号码     |
| realname | string | N | 车主姓名 |
| reason | string | N | 原因 |
| operator | string | N | 操作员姓名 |
| start_time | string | Y | 开始时间, 格式: `yyyyMMddHHmmss` |
| end_time | string | Y | 结束时间, 格式: `yyyyMMddHHmmss` |

**说明**
- 若存在记录直接更新

**应答参数:**

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| service | string | Y | 服务名: `service.parking.blacklist.create` |
| version | string | Y | 版本号:`1.0`|
| charset | string | Y | 字符集:`UTF-8`|
| result_code | string | Y | 本次请求状态返回码, 示例:1001<br/>状态码  含义<br/>1001  接口处理成功, 业务参数将返回.<br/>1401  签名错误, 请检查配置.<br/>1500  接口处理异常. |
| sign | string | Y | 当前请求应答结果, 签名方法参考附录. |
| message | string | Y | 状态码处理描述, 如:返回错误信息. |

### 6.4 移除黑名单车辆

**协议说明:**
P云发起请求, 将指定车辆从车场黑名单移除

**请求参数:**

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| service | string | Y | 服务名: `service.parking.blacklist.delete` |
| version | string | Y | 版本号:`1.0`|
| charset | string | Y | 字符集:`UTF-8`|
| sign | string | Y | 签名|

| 字段 | 类型 | 必须 | 说明 |
| --- | --- | --- | ---|
| park_uuid | string | Y | 停车场编号|
| plate | string | Y | 车牌号码     |
| reason | string | N | 原因 |
| operator | string | N | 操作员姓名 |

**说明**
- 若不存在记录直接返回操作成功！

**应答参数:**

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| service | string | Y | 服务名: `service.parking.blacklist.delete` |
| version | string | Y | 版本号:`1.0`|
| charset | string | Y | 字符集:`UTF-8`|
| result_code | string | Y | 本次请求状态返回码, 示例:1001<br/>状态码  含义<br/>1001  接口处理成功, 业务参数将返回.<br/>1401  签名错误, 请检查配置.<br/>1500  接口处理异常. |
| sign | string | Y | 当前请求应答结果, 签名方法参考附录. |
| message | string | Y | 状态码处理描述, 如:返回错误信息. |

### 6.5 获取固定车收费标准

**协议说明:**
P云发起请求, 查询车场固定车收费标准

**请求参数:**

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| service | string | Y | 服务名: `service.parking.charge_type.list` |
| version | string | Y | 版本号:`1.0`|
| charset | string | Y | 字符集:`UTF-8`|
| sign | string | Y | 签名|

| 字段 | 类型 | 必须 | 说明 |
| --- | --- | --- | ---|
| park_uuid | string | Y | 停车场编号|

**说明**
- 返回车场所有有效状态的收费标准

**应答参数:**

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| service | string | Y | 服务名: `service.parking.charge_type.list` |
| version | string | Y | 版本号:`1.0`|
| charset | string | Y | 字符集:`UTF-8`|
| result_code | string | Y | 本次请求状态返回码, 示例:1001<br/>状态码  含义<br/>1001  接口处理成功, 业务参数将返回.<br/>1401  签名错误, 请检查配置.<br/>1500  接口处理异常. |
| sign | string | Y | 当前请求应答结果, 签名方法参考附录. |
| message | string | Y | 状态码处理描述, 如:返回错误信息. |

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| row | array | N | 收费类型集合, 对象参考后面收费类型信息. |

**收费类型信息:**

| 字段 | 类型 | 必须 | 说明    |
| --- | --- | --- | ---|
| id | string | Y | 车场收费规则ID. |
| name | string | Y | 收费规则名称|
| type| int | Y | 适用固定车类型:<br/>1 月卡<br/>2 储值<br/>3 储次<br/>4 储天<br/>5 年卡<br/>6 季度卡<br/>7 半年卡<br/>0 免费停车  |
| desc | string | Y | 固定车收费描述 |
| price | int | Y | 固定车收费单价, 单位分, 若返回云端可自动配置收费模版。 |
| create_time | string | Y | 创建时间, 格式: `yyyyMMddHHmmss` |
| update_time | string | Y | 更新时间, 格式: `yyyyMMddHHmmss` |


### 6.6 新增固定车车辆

**协议说明:**
P云发起请求, 将指定车辆设置为车场固定车

**请求参数:**

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| service | string | Y | 服务名: `service.parking.vip.create` |
| version | string | Y | 版本号:`1.0`|
| charset | string | Y | 字符集:`UTF-8`|
| sign | string | Y | 签名|

| 字段 | 类型 | 必须 | 说明 |
| --- | --- | --- | ---|
| park_uuid | string | Y | 停车场编号|
| plate | string | Y | 车牌号码     |
| realname | string | N | 车主姓名 |
| mobile | string | N | 联系电话 |
| id_card | string | N | 车主身份证号码 |
| card_id | string | N | 固定车卡ID |
| start_time | string | Y | 开始时间, 格式: `yyyyMMddHHmmss` |
| end_time | string | Y | 结束时间, 格式: `yyyyMMddHHmmss` |
| charge_type | string | Y | 收费类型 |
| value | int | Y | 续费金额:<br/>储值: 续费金额, 单位分;<br/>储次:续费次数;<br/>储天:续费天数;<br/>月卡/年卡/半年卡/季度卡: 续费天数. |
| quantity | int | Y | 续费数量:<br/>储值: 续费金额, 单位分;<br/>储次:续费次数;<br/>储天:续费天数;<br/>月卡:续费月数;<br/>年卡:续费年数;<br/>半年卡:续费半年数;<br/>季度卡:续费季度数.|
| origin_value | int | Y | 原始价格, 单位分 |
| pay_value | int | Y | 用户实际付款金额, 单位分 |
| pay_origin | int | Y | 值 含义<br/>0    P云<br/>4   支付宝<br/>8    微信<br/>-     兼容其他未定义|
| pay_origin_desc | string | Y | 支付到账说明, 例如:P云|
| pay_source | string | N | 付款来源, 例如: 微信/支付宝/银联 |
| address | string | N | 车主单位/住宅 |
| park_space_no | string | N | 车位号 |
| operator | string | N | 操作员姓名 |

**说明**
- 若存在记录则返回`1403`拒绝创建!

**应答参数:**

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| service | string | Y | 服务名: `service.parking.vip.create` |
| version | string | Y | 版本号:`1.0`|
| charset | string | Y | 字符集:`UTF-8`|
| result_code | string | Y | 本次请求状态返回码, 示例:1001<br/>状态码  含义<br/>1001  接口处理成功, 业务参数将返回.<br/>1403 固定车已存在拒绝创建<br/>1401  签名错误, 请检查配置.<br/>1500  接口处理异常. |
| sign | string | Y | 当前请求应答结果, 签名方法参考附录. |
| message | string | Y | 状态码处理描述, 如:返回错误信息. |

### 6.7 移除固定车车辆

**协议说明:**
P云发起请求, 将指定车辆从车场固定车移除

**请求参数:**

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| service | string | Y | 服务名: `service.parking.vip.delete` |
| version | string | Y | 版本号:`1.0`|
| charset | string | Y | 字符集:`UTF-8`|
| sign | string | Y | 签名|

| 字段 | 类型 | 必须 | 说明 |
| --- | --- | --- | ---|
| park_uuid | string | Y | 停车场编号|
| plate | string | Y | 车牌号码     |
| card_id | string | N | 固定车卡ID     |
| reason | string | N | 原因 |
| operator | string | N | 操作员姓名 |

**说明**
- 若不存在记录直接返回操作成功！

**应答参数:**

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| service | string | Y | 服务名: `service.parking.vip.delete` |
| version | string | Y | 版本号:`1.0`|
| charset | string | Y | 字符集:`UTF-8`|
| result_code | string | Y | 本次请求状态返回码, 示例:1001<br/>状态码  含义<br/>1001  接口处理成功, 业务参数将返回.<br/>1401  签名错误, 请检查配置.<br/>1500  接口处理异常. |
| sign | string | Y | 当前请求应答结果, 签名方法参考附录. |
| message | string | Y | 状态码处理描述, 如:返回错误信息. |

## 7 预约停车

### 7.1 预约停车申请

**说明:**

在可预约车场提交预约停车申请, 提交的预约时间范围不表示车辆实际出入场时间, 由车场系统本地根据时间范围控制是否可入场。

**请求参数:**

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| service | string | Y | 服务名: `service.parking.reserve.apply` |
| version | string | Y | 版本号:`1.0`|
| charset | string | Y | 字符集:`UTF-8`|
| sign | string | Y | 签名|

| 字段 | 类型 | 必须 | 说明 |
| --- | --- | --- | ---|
| park_uuid | string | Y | 停车场编号 |
| apply_no | string | Y | 预约单号 |
| plate | string | N | 车牌号码 |
| start_time | string | Y | 预约开始时间, 格式: `yyyyMMddHHmmss` |
| end_time | string | Y | 预约结束时间, 格式: `yyyyMMddHHmmss` |
| mobile | string | Y | 联系电话 |
| realname | string | Y | 车主姓名 |
| reason | string | Y | 预约原因 |
| pay_serial | string | N | 预约支付流水 |
| pay_value | string | N | 支付金额, 单位分 |


**应答参数:**

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| service | string | Y | 服务名: `service.parking.reserve.apply` |
| version | string | Y | 版本号:`1.0`|
| charset | string | Y | 字符集:`UTF-8`|
| result_code | string | Y | 本次请求状态返回码, 示例:1001<br/>状态码  含义<br/>1001  预约成功.<br/>1002 无可预约车位.<br/>1401  签名错误, 请检查配置.<br/>1500  接口处理异常. |
| sign | string | Y | 当前请求应答结果, 签名方法参考附录. |
| message | string | Y | 状态码处理描述, 如:返回错误信息. |

### 7.2 取消预约停车申请

**说明:**

在可预约车场提交预约停车申请, 提交的预约时间范围不表示车辆实际出入场时间, 由车场系统本地根据时间范围控制是否可入场。

**请求参数:**

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| service | string | Y | 服务名: `service.parking.reserve.cancel` |
| version | string | Y | 版本号:`1.0`|
| charset | string | Y | 字符集:`UTF-8`|
| sign | string | Y | 签名|

| 字段 | 类型 | 必须 | 说明 |
| --- | --- | --- | ---|
| park_uuid | string | Y | 停车场编号 |
| apply_no | string | Y | 预约单号 |
| plate | string | N | 车牌号码 |
| reason | string | Y | 取消原因 |

**应答参数:**

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| service | string | Y | 服务名: `service.parking.reserve.cancel` |
| version | string | Y | 版本号:`1.0`|
| charset | string | Y | 字符集:`UTF-8`|
| result_code | string | Y | 本次请求状态返回码, 示例:1001<br/>状态码  含义<br/>1001  操作成功<br/>1401  签名错误, 请检查配置.<br/>1500  接口处理异常. |
| sign | string | Y | 当前请求应答结果, 签名方法参考附录. |
| message | string | Y | 状态码处理描述, 如:返回错误信息. |

## 8 其他接口

### 8.1 车场设备列表

**协议说明:**

P云在需要停车场本地文件业务处理时通过本协议获取指定文件.


**请求参数:**

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| service | string | Y | 服务名: `service.parking.device.list` |
| version | string | Y | 版本号:`1.0`|
| charset | string | Y | 字符集:`UTF-8`|
| sign | string | Y | 签名|

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| park_uuid | string | Y | 停车场编号 |


**应答参数:**

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| service | string | Y | 服务名: `service.parking.device.list` |
| version | string | Y | 版本号:`1.0`|
| charset | string | Y | 字符集:`UTF-8`|
| result_code | string | Y | 本次请求状态返回码, 示例:1001<br/>状态码  含义<br/>1001  接口处理成功, 业务参数将返回.<br/>1401  签名错误, 请检查配置.<br/>1500  接口处理异常. |
| sign | string | Y | 当前请求应答结果, 签名方法参考附录. |
| message | string | Y | 状态码处理描述, 如:返回错误信息. |

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| device_list | array | N | 设备集合, 对象参考后面设备信息. |

**设备信息:**

| 字段 | 类型 | 必须 | 说明    |
| --- | --- | --- | ---|
| name | string | Y | 设备名称 |
| type | int | Y | 设备类型:<br/>1 相机<br/>2 入口通道控制器<br/>3 出口通道控制器<br/> 4 出入口通道控制器 |
| no | string | N | 设备编号 |
| uuid | string | N | 设备ID |
| vendor | string | N | 制造商, 例如: 大华/海康 |
| model | string | N | 设备型号, 例如: TZ-O1 |
| status | int | Y | 状态:<br/> -1 已离线<br/> 0 未知<br/> 1 正常运行 |
| status_desc | string | Y | 状态描述, 例如: 脱机 |
| mac_addr | string | N | 网卡MAC地址, 例如: ac:de:48:a0:11:24 |
| ipv4_addr | string | N | 网卡IP地址, 例如: 172.12.1.29 |
| uptime | string | N | 启动时间, 格式`yyyyMMddHHmmss`.|
| timestamp | string | N | 设备时间, 格式`yyyyMMddHHmmss`.|

### 8.2 停车场文件拉取

**协议说明:**

P云在需要停车场本地文件业务处理时通过本协议获取指定文件.


**请求参数:**

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| service | string | Y | 服务名: `service.remote.file.pull` |
| version | string | Y | 版本号:`1.0`|
| charset | string | Y | 字符集:`UTF-8`|
| sign | string | Y | 签名|

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| path | string | Y | 需拉取文件路径 |
| folder | string | Y | 指定的上传目录. |
| public | int | Y | 指定文件访问权限, 1公开, 0私有 |
| expires | long | N | 上传有效时间, 单位ms. |
| api | string | Y | 上传接口地址.<br/>例: https://files.660pp.com/u/ |
| access_key | string | Y | 上传访问密钥. |
| secret_key | string | Y | 上传签名密钥. |

**应答参数:**

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| service | string | Y | 服务名: `service.remote.file.pull` |
| version | string | Y | 版本号:`1.0`|
| charset | string | Y | 字符集:`UTF-8`|
| result_code | string | Y | 本次请求状态返回码, 示例:1001<br/>状态码  含义<br/>1001  获取成功, 业务参数将返回.<br/>1400  参数错误, 请检查参数.<br/>1401  签名错误, 请检查配置.<br/>1404  文件不存在.<br/>1500  接口处理异常. |
| sign | string | Y | 当前请求应答结果, 签名方法参考附录. |
| message | string | Y | 状态码处理描述, 如:返回错误信息. |

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| url | string | Y | 成功上传后文件访问路径.|

### 8.3 停车场命令执行

**协议说明:**

P云在需要停车场本地文件业务处理时通过本协议获取指定文件.


**请求参数:**

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| service | string | Y | 服务名: `service.remote.command.execute`|
| version | string | Y | 版本号:`1.0`|
| charset | string | Y | 字符集:`UTF-8`|
| sign | string | Y | 签名|

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| file | string | Y | 执行命令文件名/路径.|
| workspaces | string | N | 执行名称工作目录.|
| args | string | N | 执行参数.|
| wait_exit | int | N | 等待执行结束, 0不等待, 1等待.|
| timeout | int | N | 执行超时时间, 单位s.|

**应答参数:**

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| service | string | Y | 服务名: `service.remote.command.execute` |
| version | string | Y | 版本号:`1.0`|
| charset | string | Y | 字符集:`UTF-8`|
| result_code | string | Y | 本次请求状态返回码, 示例:1001<br/>状态码  含义<br/>1001  获取成功, 业务参数将返回.<br/>1400  参数错误, 请检查参数.<br/>1401  签名错误, 请检查配置.<br/>1500  接口处理异常. |
| sign | string | Y | 当前请求应答结果, 签名方法参考附录. |
| message | string | Y | 状态码处理描述, 如:返回错误信息. |

| 字段 | 类型 | 必须 | 说明|
| --- | --- | --- | --- |
| exit_code | int | N | 执行完成退出码, 当waite_exit=1返回. |
| output | string | N | 执行输出结果, 当waite_exit=1返回. |


## 附录

### A 业务流程

#### A1 场内支付

```sequence

P云->停车场系统:1.获取停车订单
停车场系统-->>P云:2.返回停车信息
P云->P云:3.引导完成支付
P云->停车场系统:4.支付结果通知
停车场系统-->>P云:5应答支付结果受理结果

```

#### A2 自动支付
```sequence

P云->停车场系统:1.1拉取进出场车辆列表
Note left of P云:定时触发
停车场系统-->>P云:1.2返回进出场车辆列表

P云->停车场系统:2.1发起开通车辆自动支付权限
停车场系统-->>P云:2.2应答自动支付开通结果

P云->停车场系统:3.1发起取消车辆自动支付权限
停车场系统-->>P云:3.2应答自动支付取消结果

```

#### A3 商户优惠
```sequence

P云->停车场系统:1.发放商户优惠
停车场系统-->>P云:2.优惠受理结果

P云->停车场系统:3.获取停车订单
停车场系统-->>P云:4.返回停车信息

P云->P云:5.引导完成支付

P云->停车场系统:6.支付结果通知
停车场系统-->>P云:7.应答后附非取消结果

```

### B 二维码的相关说明
P云将为每个停车场生成一个停车场唯一的二维码，车主直接使用微信/支付宝扫码进入车牌输入页面.
用户输入车牌后, 根据是否是车牌类型, 如临时车, 月卡等类型, 会进行不同的行为:
- 临时车: 支付页面
- 月卡/次卡/储值卡: 月卡/次卡/储值卡的续费行为

### C 签名
使用MD5算法进行签名.
#### C1 签名流程
1.先将参数串按照参数名(不包括sign自己)进行升序排序(若有多个相同参数名则继续按照其参数值进行升序排序).
2.以http url参数风格等到待签名的字符串.
如:
``` java
"a=v&b=0&c=1900000109&d=102&key=7d15453e97507ef794cf7b0519d"
```
3.使用**标准MD5**算法对待签名字符串进行签名.
4.签名结果全部转换成大写后, 即为我们所需的订单MD5校验码
5.将其写入sign字段.

#### C2 注意事项
1. 只有本文档协议中出现的参数才参与签名.
2. 拼凑值不要有空格.
3. 空值参数无需传递, 且不能包含到待签名数据中.
4. 签名时将字符转变成字节流时统一使用UTF-8.
5. 所有的请求和响应都需要进行签名.
6. 签名以带过来的实际参数为准, 即带过来的参数去掉空值参数后都要参与MD5签名.


### D 特殊情况和相应的处理策略
#### D1 对于同一次停车的多个支付订单的处理策略
场中支付时, P云主动获取停车订单;停车场应保证只有最新的一条订单是可支付的;即:
- A. 每次获取订单都应将本次停车之前未成功支付的订单状态标记为已撤销.
- B. 关于中央现金支付: 若是中央支付, 用户先获取订单, 然后在收费亭完成现金支付后, 本次停车之前的所有未受到支付通知的订单都标记为已撤销.

<font color='red'>
注: 简单的说就是对于当次停车, 只有最新的一条支付记录是可处理的.如果最新的支付记录是现金支付, 那么之前的未支付订单状态都不可被支付.
</font>

#### D2 通知抵达时车辆已出场
若用户获取订单后未支付, 走现金支付了;之后用户出场后, 又去完成了支付, 此时支付结果通知抵达时, 停车场系统应返回订单已撤销;我方直接给用户办理退款.

#### D3 通知抵达时车辆有更新支付成功订单
若用户获取订单后支付, 在用户刚完成支付时停车场离线, 支付通知不能抵达停车场.此时用户到交费处支付现金(多是中央支付处的情况), 此时用户仍然尚未出场, 停车场网络恢复后, 支付通知抵达停车场, 由于有新的订单, 所以应返回订单撤销, 我方直接给用户办理退款.

#### D4 订单支付后再次收到通知
为了避免网络情况对服务质量的影响, 若P云的支付通知未能成功一次性通知到停车场, P云会多次发送支付通知, 停车场应限制如果同一笔订单成功处理后, 后续同笔订单的支付通知直接返回已处理.
