# 事件优惠下发

一个第三方服务对应一个车场的下发优惠券类型固定，由平台配置


### 1.1) 请求地址

	https://api.4pyun.com/gate/1.0/parking/mcoupon/event

### 1.2) 调用方式

	HTTP POST FORM 表单提交

### 1.3) 特殊说明
	1:app_id,app_secret 用户身份id和加密密钥由平台方提供，对接方需提供公司全称然后给到商务提交给研发申请
	2:merchant,store_code分别代表停车场商户号和商家商户号，两个参数由停车场物业那边提供
	3:coupon_code是具体一种优惠券的券ID，目前物业创建好优惠券充值好券之后截图然后找研发同事复制出来该值
	4:返回状态码code:1001代表派发成功，其它状态码直接读取message提示信息
	5:特别注意即使http状态非200也要把返回内容读取出来，会返回具体错误原因


### 1.4) 请求参数


| 字段名称 | 类型 | 必填 | 字段说明 |
| --- | --- | --- | --- |
| app_id | string | true | 平台分配的接入应用ID |
| api\_store\_code | string | true | 车场对应第三方平台ID |
| api_type | string | true | 第三方服务类型(该字段由平台分配，固定) |
| event_id | string | true | 第三方订单号, 需保证在第三方平台唯一 |
| plate | string | true | 车牌号 |
| subject | string | true | 优惠事件描述，如“【粤BD07627】于【2020-09-08 17:15:58 - 2020-09-08 17:45:26】充电 1119.62 KWH 时长 29 分钟” |
| metadata | map<string, EventMetadata> | false | 业务元数据, 该参数key为表示业务类型的某一固定值，由平台提供；EventMetadata参数参考下方；该字段固定优惠规则可不传； |
| sign | string | true | 签名 |

<h3 id="EventMetadata">EventMetadata</h3>

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| name | string | true | 数据名称，该参数仅用于数据值说明 |
| value | int32 | true | 数据值，调用方的业务变量，该参数结合metadata的key会决定下发优惠方案 |



### 1.6) 请求返回结果参数说明

| 字段名称 | 类型 | 必填 | 字段说明 |
| --- | --- | --- | --- |
| code | string | false | <b>返回Code值:</b><br/>1001 优惠派发成功<br/>1002 车辆未入场<br/>1003 优惠限制<br/>1401 第三方优惠配置不存在<br/>1403 重复派发<br/>1405 车场下发失败<br/>1500 内部错误, 提示message的信息<br/> |
| message | string | false | none |
| hint | string | false | none |
| seqno | string | false | none |
| data_node | string | false | none |
| time_cost | integer(int64) | false | none |
| plate | string | true | 车牌号 |
| enter_time | string | true | 车辆入场时间 |



### 1.7) 请求返回结果示例:

```
正常返回
{
 "code": "1001",
 "data_node": "HS1-1",
 "seqno": "86213123",
 "payload": {
 "grant_serial": "xxxxx"
 },
 "time_cost": 100,
 "message": "操作成功"
}
```

```
参数错误
{
	"code": "400",
	"message": "请求参数错误",
	"hint": "`merchant` Required!",
	"seqno": "94929a9b0874aa46",
	"data_node": "CN-South/HS3-2",
	"path": "POST /gate/1.0/parking/mcoupon/grant/create"
}
```

```
服务器内部错误
{
	"code": "1500",
	"message": "没有匹配到停车记录",
	"seqno": "57804ae2e859f58a",
	"data_node": "CN-South/HS3-1",
	"time_cost": 239,
	"payload": {
		"grant_serial": ""
	}
}
```
