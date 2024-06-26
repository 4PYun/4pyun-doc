# 车场接入文档

该部分内容供停车场系统集成厂商接入开发人员阅读，旨在为停车场系统接入开放平台提供相关指引。

车场系统通过接入该部分内容可实现:

- 临停缴费
- 固定车续费
- 电子优惠
- 无感停车
- 现金找零
- 远程开闸无人值守

等功能。

您可联系平台商务获得技术以及商务政策信息，平台将安排专人协助车场系统完成功能接入。

## 3. 对接要求
| 功能 | 功能概述 | 是否必须 |
|:---|:---|:---|
| 移动支付(包括自动支付) | 获取停车费用，并完成支付流程 | Y |
| 商户优惠券 | 商家给车主下发停车优惠券;车主支付时可使用该优惠券 | Y |
| 月卡/储值卡/次卡 | 续费 | Y |
| 停车记录同步 | 可实现全国各地城市平台数据上报要求 | Y |
| 无感支付 | 可实现微信及银行无感支付 | Y |

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
