## P云开放平台标准接口

> 技术支持：dev@660pp.com
>
> 编写人：wangxin

**更新日志**

| 时间       | 更新内容                                |
| ---------- | --------------------------------------- |
| 2019-07-08 | 初始编写                                |
| 2019-09-25 | 新增标准支付通知接口                    |
| 2019-10-10 | 新增电子发票信息推送                    |
| 2019-10-17 | 优化电子发票信息推送参数                |
| 2019-12-13 | 订单推送增加第三方优惠信息（积分+卡券） |
| 2020-06-01 | 新增月卡续费订单推送                    |
| 2020-06-11 | 新增电子发票状态同步                    |
| 2020-12-11 | 支付同步新增`pay_mode_desc`             |
| 2020-12-29 | 停车场出入场推送新增车场信息            |
| 2021-03-01 | 新增奖励优惠券接口                      |
| 2021-04-14 | 新增用户OAuth授权验证接口                      |

### 目录

- <a href="#api_spec">1 接口约定</a>
- <a href="#parking_apis">2 停车场业务</a>
 - <a href="#parking_enter">2.1 入场推送</a>
 - <a href="#parking_leave">2.2 出场推送</a>
 - <a href="#parking_invoice">2.3 电子发票推送</a>
 - <a href="#parking_payment">2.4 支付记录推送</a>
 - <a href="#parking_recharge">2.5 月卡续费记录推送</a>
- <a href="#payment_apis">3 支付业务</a>
 - <a href="#payment_sync">3.1 支付结果同步</a>
- <a href="#invoice_apis">4 电子发票</a>
 - <a href="#invoice_sync">4.1 电子发票状态同步</a>
 - <a href="#bonus_apis">5 奖励优惠券</a>
 - <a href="#bonus_precheck">5.1 奖励优惠券领取校验</a>
 - <a href="#bonus_grant">5.2 奖励优惠券领取通知</a>
 - <a href="#bonus_apply">5.3 奖励优惠券使用通知</a>
 - <a href="#bonus_apis">6 用户服务</a>
 - <a href="#bonus_precheck">6.1 用户信息查询</a>



### <a id="api_spec">1 接口约定</a>

#### 1.1 公共参数

| 字段       | 说明                                | 示例                             |
| :--------- | :---------------------------------- | :------------------------------- |
| app_id     | 应用ID, 由P云平台分配               | op12defadfad                     |
| app_secret | 应用密钥, 由P云平台分配用于计算签名 | ohsh5Eegoquu8ehah0bahmei8thudiel |
| timestamp  | 当前请求时间戳, 单位毫秒            | 1570318158852                    |
| sign       | 签名数据，具体规则见签名算法        | 1A6FE20BDD05B654F8FD33A299D75DF3 |
| sign_type  | 签名算法                            | MD5                              |


#### 1.2 签名算法

**算法执行步骤**

1. 设所有发送或接收到的数据为集合M，将集合M内非空参数值的参数按照参数名 `ASCII码` 从小到大排序（字典序），使用 `URL键值对` 的格式（即 `key1=value1&key2=value2...`）拼接成字符串 `stringA`。
2. `sign = stringA + "&app_secret=" + appSecret`，取 `MD5`（`32位不区分大小写`）。

**注意事项**

1. 参数名 ASCII码 从小到大排序（字典序）；
2. 如果参数值为空（即null或空字符串）不参与签名；
3. 参数名区分大小写；
4. 验证签名时，sign 参数不参与签名，将生成的签名与该 sign 值作校验；
5. 接口可能增加字段，验证签名时必须支持增加的扩展字段；

**示例**

**密钥：**

| 字段       | 示例值                           |
| ---------- | -------------------------------- |
| app_id     | op88641899bd20661                |
| app_secret | 29b72e85f56f9d20b2303d5289fe78c9 |

**输入参数：**

| 字段       | 示例值                               |
| ---------- | ------------------------------------ |
| park_uuid  | 40e06b24-7320-4a61-8d97-7ebccb364a87 |
| plate      | 粤B660PP                             |
| car_type   | 1                                    |
| enter_time | 1563242533431                        |

**计算 sign 的过程如下：**

1. 参数排序后字符串拼接：

```
app_id=op88641899bd20661&car_type=1&enter_time=1563242533431&park_uuid=40e06b24-7320-4a61-8d97-7ebccb364a87&plate=粤B660PP&sign_type=MD5&timestamp=1563242932357&app_secret=29b72e85f56f9d20b2303d5289fe78c9
```

2. 使用 MD5 （32位不区分大小写）加密：

`1A6FE20BDD05B654F8FD33A299D75DF3`

### <a id="parking_apis">2 停车业务</a>

#### <a id="parking_enter">2.1 入场推送</a>

- 描述：
- 请求参数

| 字段           | 类型   | 必须 | 说明                          |
| -------------- | ------ | ---- | ----------------------------- |
| park_uuid      | string | Y    | 平台停车场编号                |
| park_name      | string | Y    | 平台停车场名称                |
| address        | string | Y    | 平台停车场地址                |
| area_code      | string | Y    | 平台停车场区域编码            |
| latitude       | string | Y    | 纬度(百度)                    |
| longitude      | string | Y    | 经度(百度)                    |
| parking_serial | string | Y    | 平台停车流水表示（record_id） |
| plate          | string | Y    | 车牌                          |
| plate_color    | string | Y    | 车牌颜色（见附录）            |
| car_type       | string | Y    | 车类（见附录）                |
| car_desc       | string | Y    | 车类描述                      |
| enter_time     | string | Y    | 入场时间时间戳字符，毫秒      |
| enter_image    | string | N    | 入场图片地址                  |
| enter_gate     | string | N    | 入场通道编号                  |
| enter_security | string | N    | 入口管理员                    |
| vehicle_type   | string | N    |                               |

- 响应参数

| 字段    | 类型   | 必须 | 说明             |
| ------- | ------ | ---- | ---------------- |
| code    | string | Y    | 业务处理状态码   |
| message | string | N    | 业务处理状态说明 |
| hint    | string | N    | 提示说明         |

#### <a id="parking_leave">2.2 出场推送</a>

- 描述
- 请求参数

| 字段           | 类型   | 必须 | 说明                          |
| -------------- | ------ | ---- | ----------------------------- |
| park_uuid      | string | Y    | 平台停车场编号                |
| park_name      | string | Y    | 平台停车场名称                |
| address        | string | Y    | 平台停车场地址                |
| area_code      | string | Y    | 平台停车场区域编码            |
| latitude       | string | Y    | 纬度(百度)                    |
| longitude      | string | Y    | 经度(百度)                    |
| parking_serial | string | Y    | 平台停车流水表示（record_id） |
| plate          | string | Y    | 车牌                          |
| plate_color    | string | Y    | 车牌颜色（见附录）            |
| car_type       | string | Y    | 车类（见附录）                |
| car_desc       | string | Y    | 车类描述                      |
| enter_time     | string | Y    | 入场时间时间戳字符，毫秒      |
| enter_image    | string | N    | 入场图片地址                  |
| enter_gate     | string | N    | 入场通道编号                  |
| leave_time     | string | Y    | 出场时间时间戳字符，毫秒      |
| leave_image    | string | N    | 出场图片地址                  |
| leave_gate     | string | N    | 出场通道编号                  |
| parking_time   | string | Y    | 停车时长，单位：秒            |
| total_value    | string | Y    | 总金额，单位：分              |
| free_value     | string | N    | 优惠金额，单位：分            |
| autopay_value  | string | N    | 无感支付的金额，单位：分      |

- 响应参数

| 字段    | 类型   | 必须 | 说明             |
| ------- | ------ | ---- | ---------------- |
| code    | string | Y    | 业务处理状态码   |
| message | string | N    | 业务处理状态说明 |
| hint    | string | N    | 提示说明         |

#### <a id="parking_payment">2.3 支付记录推送</a>

- 描述
- 请求参数

| 字段           | 类型     | 必须 | 说明               |
| -------------- | -------- | ---- | ------------------ |
| park_uuid      | string   | Y    | 平台停车场编号     |
| park_name      | string   | Y    | 平台停车场名称     |
| parking_serial | string   | Y    | 平台停车流水       |
| pay_serial     | string   | Y    | 平台支付流水       |
| pay_value      | string   | Y    | 支付金额，单位：分 |
| pay_time       | string   | Y    | 支付时间戳，毫秒   |
| discounts      | object[] | Y    | 优惠信息集         |

**Discount**

| 字段          | 类型   | 必须 | 说明                                                      |
| ------------- | ------ | ---- | --------------------------------------------------------- |
| type          | string | Y    | 优惠类型，参考：com.chinaroad.api.v1.parking.DiscountType |
| token         | string | Y    | 优惠凭证令牌                                              |
| openid        | string | Y    | 用户ID                                                    |
| mobile        | string | Y    | 手机号                                                    |
| relief_amount | string | Y    | 抵扣数量                                                  |
| relief_value  | string | Y    | 抵扣金额(分)                                              |

- 响应参数

| 字段    | 类型   | 必须 | 说明             |
| ------- | ------ | ---- | ---------------- |
| code    | string | Y    | 业务处理状态码   |
| message | string | N    | 业务处理状态说明 |
| hint    | string | N    | 提示说明         |

#### <a id="parking_recharge">2.4 月卡续费记录通知</a>

- 描述
- 请求参数

| 字段       | 类型   | 必须 | 说明                                       |
| ---------- | ------ | ---- | ------------------------------------------ |
| park_uuid  | string | Y    | 平台停车场编号                             |
| park_name  | string | Y    | 平台停车场名称                             |
| pay_serial | string | Y    | 平台支付流水                               |
| pay_value  | string | Y    | 支付金额，单位：分                         |
| pay_time   | string | Y    | 支付时间戳，毫秒                           |
| plate      | string | Y    | 车牌号                                     |
| quantity   | int    | Y    | 续费数量，月卡:月份数，储值卡:续费金额(分) |
| vip_type   | int    | Y    | 月卡类型(见附录)                           |
| start_time | string | N    | 时间类固定车续费开始时间                   |
| end_time   | string | N    | 时间类固定车续费结束时间                   |

- 响应参数

| 字段    | 类型   | 必须 | 说明             |
| ------- | ------ | ---- | ---------------- |
| code    | string | Y    | 业务处理状态码   |
| message | string | N    | 业务处理状态说明 |
| hint    | string | N    | 提示说明         |

#### <a id="parking_invoice">2.5 电子发票推送</a>

- 描述
- 请求参数

| 字段           | 类型     | 必须 | 说明                                                    |
| -------------- | -------- | ---- | ------------------------------------------------------- |
| invoice_id     | string   | Y    | P云发票唯一标识                                         |
| create_time    | string   | Y    | 用户提交开票时间                                        |
| subject        | string   | Y    | 开票内容                                                |
| verify_code    | string   | Y    | 发票校验码                                              |
| obtain_value   | string   | Y    | 开票金额, 单位分                                        |
| request_serial | string   | Y    | 请求流水                                                |
| invoice_code   | string   | Y    | 发票代码                                                |
| invoice_no     | string   | Y    | 发票号码                                                |
| result_time    | string   | Y    | 开票成功时间                                            |
| tax_type       | string   | Y    | 类型, 参考: com.chinaroad.api.v1.parking.InvoiceTaxType |
| red_rush       | string   | Y    | 红冲状态, 参考: com.chinaroad.api.v1.consts.Bool        |
| remark         | string   | Y    | 开票备注                                                |
| identity       | string   | Y    | 用户ID                                                  |
| buyer          | object   | Y    | 购买方 `InvoiceBuyer`                                   |
| seller         | object   | Y    | 销售方 `InvoiceSeller`                                  |
| goods_list     | object[] | Y    | 销售商品 `ParkingInvoiceGoods`                          |

**InvoiceBuyer**

| 字段              | 类型   | 必须 | 说明                                                         |
| ----------------- | ------ | ---- | ------------------------------------------------------------ |
| tax_no            | string | Y    | 税号                                                         |
| tax_type          | string | Y    | 类型 1:企业 2:个人 com.chinaroad.api.v1.parking.InvoiceTaxType |
| email             | string | Y    | 邮箱                                                         |
| telephone         | string | Y    | 联系电话                                                     |
| company_name      | string | Y    | 公司名称                                                     |
| company_telephone | string | Y    | 公司电话                                                     |
| company_address   | string | Y    | 公司地址                                                     |

**InvoiceSeller**

| 字段         | 类型   | 必须 | 说明     |
| ------------ | ------ | ---- | -------- |
| tax_no       | string | Y    | 税号     |
| address      | string | Y    | 地址     |
| telephone    | string | Y    | 电话     |
| bank_name    | string | Y    | 银行名称 |
| bank_account | string | Y    | 银行账户 |
| drawer       | string | Y    | 开票人   |
| payee        | string | Y    | 收款人   |
| reviewer     | string | Y    | 复核人   |
| tax_rate     | string | Y    | 税率     |
| product_no   | string | Y    | 商品编码 |

**ParkingInvoiceGoods**

| 字段         | 类型   | 必须 | 说明                                                         |
| ------------ | ------ | ---- | ------------------------------------------------------------ |
| business     | string | Y    | ```临时车或者月卡 临时车缴费:car:parking:cashier,月卡充值:car:parking:recharge``` |
| plate        | string | Y    | 车牌号                                                       |
| park_uuid    | string | Y    | 车场ID                                                       |
| park_name    | string | Y    | 车场名称                                                     |
| merchant     | string | Y    | 车场商户号                                                   |
| pay_serial   | string | Y    | 支付流水                                                     |
| value        | string | Y    | 订单金额，分                                                 |
| pay_value    | string | Y    | 支付金额，分                                                 |
| obtain_value | string | Y    | 开票金额，分                                                 |


- 响应参数

| 字段    | 类型   | 必须 | 说明             |
| ------- | ------ | ---- | ---------------- |
| code    | string | Y    | 业务处理状态码   |
| message | string | N    | 业务处理状态说明 |

- 请求示例

```
{
 "app_id": "op660pp",
 "buyer": {
  "company_address": "广东省深圳市南山区",
  "company_name": "深圳市神州路路通网络科技有限公司",
  "company_telephone": "19925333063",
  "email": "dev@660pp.com",
  "tax_no": "1571280012061",
  "tax_type": "1",
  "telephone": "19925333063"
 },
 "create_time": 1571280012056,
 "goods_list": [
  {
"business": "car:parking:cashier",
"merchant": "62662601",
"pay_serial": "1571280012063",
"pay_value": 10000,
"value": 10000
  },
  {
"business": "car:parking:cashier",
"merchant": "62662601",
"pay_serial": "1571280012063",
"pay_value": 20000,
"value": 20000
  },
  {
"business": "car:parking:cashier",
"merchant": "62662601",
"pay_serial": "1571280012063",
"pay_value": 15000,
"value": 15000
  }
 ],
 "invoice_code": "1571280012056",
 "invoice_id": "1571280012056",
 "invoice_no": "1571280012056",
 "obtain_value": 10000,
 "red_rush": 1,
 "remark": "REMARK",
 "request_serial": "1571280012056",
 "result_time": 1571280012056,
 "seller": {
  "address": "广东省深圳市",
  "bankAccount": "1571280012062",
  "bankName": "中国农业银行",
  "operator": "张三",
  "payee": "李四",
  "productNo": "1571280012062",
  "reviewer": "李雷",
  "taxNo": "1",
  "taxRate": "0",
  "telephone": "19925333063"
 },
 "subject": "1571280012056",
 "tax_type": 0,
 "timestamp": 1571280012063,
 "verify_code": "1571280012056"
}
```

### <a id="payment_apis">3 支付业务</a>

#### <a id="payment_sync">3.1 支付结果同步</a>

- 描述
  合作方实现改接口用于接受支付结果通知, 若通知未能返回`1001`不成功则会重复通知多次。

- 请求参数

| 字段          | 类型   | 必须 | 说明                                 |
| ------------- | ------ | ---- | ------------------------------------ |
| merchant      | string | Y    | 商户号                               |
| pay_order     | string | Y    | 支付订单号                           |
| channel       | string | Y    | 支付渠道                             |
| pay_serial    | string | Y    | 平台支付流水                         |
| value         | string | Y    | 支付金额，单位：分                   |
| status        | short  | Y    | 交易状态: 1支付成功、-1失败、0支付中 |
| trade_time    | long   | Y    | 交易时间, 单位ms                     |
| pay_mode      | short  | Y    | 到账方式, 参考附录                   |
| pay_mode_desc | string | Y    | 到账方式描述, 例如: 微信、支付宝     |

- 响应参数

| 字段    | 类型   | 必须 | 说明             |
| ------- | ------ | ---- | ---------------- |
| code    | string | Y    | 业务处理状态码   |
| message | string | N    | 业务处理状态说明 |
| hint    | string | N    | 提示说明         |

### <a id="invoice_apis">4 电子发票</a>

#### <a id="invoice_sync">4.1 电子发票状态同步</a>

- 描述
  合作方实现改接口用于接受电子发票开票结果通知, 若通知未能返回`1001`不成功则会重复通知多次。

- 请求参数

| 字段          | 类型   | 必须 | 说明                              |
| ------------- | ------ | ---- | --------------------------------- |
| merchant      | string | Y    | 商户号                            |
| obtain_order  | string | Y    | 合作方开票请求订单                |
| obtain_serial | string | Y    | 平台方开票唯一标识                |
| status        | short  | Y    | 开票状态: 2 开票成功、-1 开票失败 |
| status_desc   | string | N    | 状态说明                          |
| invoice_code  | string | N    | 发票代码                          |
| invoice_no    | string | N    | 发票号码                          |
| verify_code   | string | N    | 发票校验码                        |
| invoice_url   | string | N    | 发票链接                          |
| invoice_time  | long   | N    | 交易时间, unix时间戳, 单位ms      |

- 响应参数

| 字段    | 类型   | 必须 | 说明             |
| ------- | ------ | ---- | ---------------- |
| code    | string | Y    | 业务处理状态码   |
| message | string | N    | 业务处理状态说明 |
| hint    | string | N    | 提示说明         |

### <a id="bonus_apis">5 奖励优惠券</a>

#### <a id="bonus_precheck">5.1 奖励优惠券领取校验</a>

- 描述
  合作方实现该接口，用于查询用户是否符合领券要求。

- 请求方式

  `POST`

- 请求参数

| 字段      | 类型   | 必须 | 说明         |
| --------- | ------ | ---- | ------------ |
| user_code | string | Y    | 用户唯一标识 |
| mobile    | string | N    | 手机号       |

- 响应参数

| 字段    | 类型   | 必须 | 说明             |
| ------- | ------ | ---- | ---------------- |
| code    | string | Y    | 业务处理状态码   |
| message | string | N    | 业务处理状态说明 |
| hint    | string | N    | 提示说明         |

####  <a id="bonus_grant">5.2 奖励优惠券领取通知</a>

- 描述
  合作方实现该接口，用于通知用户领券结果。

- 请求方式

  `POST`

- 请求参数

| 字段      | 类型   | 必须 | 说明                                      |
| --------- | ------ | ---- | ----------------------------------------- |
| user_code | string | Y    | 用户唯一标识                              |
| mobile    | string | N    | 手机号                                    |
| status    | short  | Y    | 状态，0未发放，1发放中，2已发放，-1已失效 |
| bonus_id  | string | Y    | 奖励优惠券ID                              |
| value     | int    | Y    | 奖励面额，单位分                          |
| effect_time | timestamp | Y | 生效时间 |
| expire_time | timestamp | Y | 失效时间 |

- 响应参数

| 字段    | 类型   | 必须 | 说明             |
| ------- | ------ | ---- | ---------------- |
| code    | string | Y    | 业务处理状态码   |
| message | string | N    | 业务处理状态说明 |
| hint    | string | N    | 提示说明         |

#### <a id="bonus_apply">5.3奖励优惠券使用通知</a>

- 描述
  合作方实现该接口，接受用户用券通知。

  bonus_id和提交奖励事件参与申请的应答消息bonus节点的id属性具有关联性，

  接口实现方匹配每条奖励记录进行核销结果处理

- 请求方式

  `POST`

- 请求参数

| 字段      | 类型      | 必须 | 说明          |
| --------- | --------- | ---- | ------------- |
| user_code | string    | Y    | 用户唯一标识  |
| mobile    | string    | N    | 手机号        |
| status    | short     | Y    | 状态，1已核销 |
| bonus_id  | string    | Y    | 奖励优惠券ID  |
| value     | int       | Y    | 核销金额      |
| apply_time | timestamp | Y    | 核销时间      |

- 响应参数

| 字段    | 类型   | 必须 | 说明             |
| ------- | ------ | ---- | ---------------- |
| code    | string | Y    | 业务处理状态码   |
| message | string | N    | 业务处理状态说明 |
| hint    | string | N    | 提示说明         |


### <a id="bonus_apis">6 用户服务</a>

#### <a id="bonus_precheck">6.1 用户信息查询</a>

- 描述
  合作方实现该接口，用于验证查询用户基础信息。

- 请求方式

  `POST`

- 请求参数

| 字段      | 类型   | 必须 | 说明         |
| --------- | ------ | ---- | ------------ |
| oauth_code | string | Y    | OAuth授权码 |

- 响应参数

| 字段    | 类型   | 必须 | 说明             |
| ------- | ------ | ---- | ---------------- |
| code    | string | Y    | 业务处理状态码   |
| message | string | N    | 业务处理状态说明 |
| hint    | string | N    | 提示说明         |
| account    | string | N    | 第三方账号 |
| email    | string | N    | 邮箱 |
| mobile    | string | N    | 手机号 |
| nickname    | string | Y    | 昵称 |

### 附录

#### A 业务状态码

| 状态码 | 说明         |
| ------ | ------------ |
| 1001   | 请求成功     |
| 1400   | 请求参数错误 |
| 1500   | 服务内部错误 |
| ...    | ...          |

#### B 车牌颜色 `plate_color`

| 值   | 说明 |
| ---- | ---- |
| -1   | 未知 |
| 1    | 蓝色 |
| 2    | 黄色 |
| 3    | 白色 |
| 4    | 黑色 |
| 5    | 绿色 |

#### C 车类 `car_type`

| 值   | 说明           |
| ---- | -------------- |
| -1   | 未知           |
| 1    | 临停车辆       |
| 2    | 包月车辆       |
| 3    | 贵宾，免费车辆 |
| 4    | 储值车辆       |

#### `InvoiceTaxType`

| 值   | 说明         |
| ---- | ------------ |
| 0    | 线下标记开票 |
| 1    | 企业开票     |
| 2    | 个人开票     |

#### `Bool`

| 值   | 说明 |
| ---- | ---- |
| 0    | No   |
| 1    | Yes  |

#### `Business`

| 值                     | 说明         |
| ---------------------- | ------------ |
| `car:parking:cashier`  | 停车场中支付 |
| `car:parking:recharge` | 月卡充值     |

#### `DiscountType`

| 值   | 说明         |
| ---- | ------------ |
| 1    | 金额优惠     |
| 2    | 时间优惠     |
| 3    | 全免类型优惠 |
| 4    | 不同计价优惠 |
| 5    | 长期有效券   |

#### `VipType`

| 值   | 说明   |
| ---- | ------ |
| 1    | 包月车 |
| 2    | 储值卡 |
| 3    | 次卡   |
| 4    | 储天卡 |
| 5    | 年卡   |
| 6    | 季度卡 |
| 7    | 半年卡 |
| 0    | 免费   |

#### `UserGender`

| 值   | 说明   |
| ---- | ------ |
| -1   | 其他  |
| 0    | 女性 |
| 1    | 男性 |
| 99   | 未知 |
