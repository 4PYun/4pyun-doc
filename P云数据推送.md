## P云开放数据推送接口

> 技术支持：dev@660pp.com
>
> 编写人：wangxin

**更新日志**

| 时间       | 更新内容                             |
| ---------- | ------------------------------------ |
| 2019-07-08 | 初始编写                             |
| 2019-09-25 | 新增标准支付通知接口 |
| 2019-10-10 | 新增电子发票信息推送 |

### 目录

- <a href="#parking_enter">入场推送</a>
- <a href="#parking_leave">出场推送</a>
- <a href="#parking_payment">支付记录推送</a>
- <a href="#parking_invoice">电子发票推送</a>

### 1 接口约定

#### 1.1 公共参数

|字段|说明|示例|
|:---|:---|:---|
|app_id|应用ID, 由P云平台分配|op12defadfad|
|app_secret|应用密钥, 由P云平台分配用于计算签名|ohsh5Eegoquu8ehah0bahmei8thudiel|
|timestamp|当前请求时间戳, 单位毫秒|1570318158852|
|sign|签名数据，具体规则见签名算法|1A6FE20BDD05B654F8FD33A299D75DF3|
|sign_type|签名算法|MD5|


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

### 2 停车业务

#### <a id="parking_enter">2.1 入场推送</a>

- 描述：
- 请求参数

| 字段           | 类型   | 必须 | 说明                          |
| -------------- | ------ | ---- | ----------------------------- |
| park_uuid      | string | Y    | 平台停车场编号                |
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
| hint | string | N    | 提示说明 |

#### <a id="parking_leave">2.2 出场推送</a>

- 描述
- 请求参数

| 字段           | 类型   | 必须 | 说明                          |
| -------------- | ------ | ---- | ----------------------------- |
| park_uuid      | string | Y    | 平台停车场编号                |
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
| hint | string | N    | 提示说明 |

#### <a id="parking_payment">2.3 支付记录推送</a>

- 描述
- 请求参数

| 字段           | 类型   | 必须 | 说明               |
| -------------- | ------ | ---- | ------------------ |
| park_uuid      | string | Y    | 平台停车场编号     |
| park_name      | string | Y    | 平台停车场名称     |
| parking_serial | string | Y    | 平台停车流水       |
| pay_serial     | string | Y    | 平台支付流水       |
| pay_value      | string | Y    | 支付金额，单位：分 |
| pay_time       | string | Y    | 支付时间戳，毫秒   |

- 响应参数

| 字段    | 类型   | 必须 | 说明             |
| ------- | ------ | ---- | ---------------- |
| code    | string | Y    | 业务处理状态码   |
| message | string | N    | 业务处理状态说明 |
| hint | string | N    | 提示说明 |

#### <a id="parking_invoice">2.4 电子发票推送</a>

- 描述
- 请求参数

| 字段        | 类型     | 必须 | 说明                   |
| ----------- | -------- | ---- | ---------------------- |
| serial      | string   | Y    | 平台发票唯一标识       |
| plate_desc  | string   | Y    | 参与开票的车牌多个车牌 |
| create_time | string   | Y    | 用户提交开票时间       |
| buyer       | object   | Y    | 购方车主信息           |
| seller      | object   | Y    | 销售方物业信息         |
| third       | object   | Y    | 发票公司返回数据       |
| payment     | object[] | Y    | 开票相关联的支付信息   |

**Buyer**

| 字段            | 类型   | 必须 | 说明               |
| --------------- | ------ | ---- | ------------------ |
| tax_no          | string | Y    | 税号               |
| company_name    | string | Y    | 公司名称           |
| tax_type        | string | Y    | 类型 1:企业 2:个人 |
| email           | string | Y    | 邮箱               |
| mobile          | string | Y    | 手机号             |
| company_mobile  | string | Y    | 公司电话           |
| company_address | string | Y    | 公司地址           |

**Seller**

| 字段         | 类型   | 必须 | 说明     |
| ------------ | ------ | ---- | -------- |
| tax_no       | string | Y    | 税号     |
| address      | string | Y    | 地址     |
| mobile       | string | Y    | 电话     |
| bank_name    | string | Y    | 银行名称 |
| bank_account | string | Y    | 银行账户 |
| drawer       | string | Y    | 开票人   |
| payee        | string | Y    | 收款人   |
| reviewer     | string | Y    | 复核人   |
| tax_rate     | string | Y    | 税率     |
| product_no   | string | Y    | 商品编码 |

**Third**

| 字段         | 类型   | 必须 | 说明                           |
| ------------ | ------ | ---- | ------------------------------ |
| serial       | string | Y    | 请求流水                       |
| check_code   | string | Y    | 验证码                         |
| third_code   | string | Y    | 发票代码                       |
| third_serial | string | Y    | 发票号码                       |
| obtain_value | string | Y    | 开票金额                       |
| url          | string | Y    | pdf 链接                       |
| remark       | string | Y    | 备注                           |
| success_time | string | Y    | 开票成功时间                   |
| red_type     | int    | Y    | 红冲状态 0代表非红冲 1代表红冲 |
| tax_type     | int    | Y    | 开票类型 1:企业 2:个人         |

**InvoicePayment**

| 字段       | 类型   | 必须 | 说明                                                         |
| ---------- | ------ | ---- | ------------------------------------------------------------ |
| business   | string | Y    | ```临时车或者月卡 临时车缴费:car:parking:cashier,月卡充值:car:parking:recharge``` |
| plate      | string | Y    | 车牌                                                         |
| park_uuid  | string | Y    | 停车场唯一id                                                 |
| merchant   | string | Y    | 商户号                                                       |
| pay_serial | string | Y    | 支付流水                                                     |
| value      | int    | Y    | 订单金额                                                     |
| pay_value  | int    | Y    | 支付金额                                                     |

- 响应参数

| 字段    | 类型   | 必须 | 说明             |
| ------- | ------ | ---- | ---------------- |
| code    | string | Y    | 业务处理状态码   |
| message | string | N    | 业务处理状态说明 |

- 请求示例

```
{
    "seller": {
        "payee": "韩梅梅",
        "address": "深圳市南山区粤海街道科技园讯美科技广场3号楼",
        "tax_no": "1570863273859",
        "mobile": "0755-86261859",
        "bank_name": "中国农业银行",
        "drawer": "李雷",
        "reviewer": "韩梅梅",
        "product_no": "1570863273859",
        "tax_rate": "0",
        "bank_account": "1234567890"
    },
    "create_time": "2019-10-12 14:54:33",
    "third": {
        "third_code": "1570863273860",
        "serial": "1570863273860",
        "tax_type": 1,
        "check_code": "1570863273860",
        "red_type": 1,
        "remark": "REMARK",
        "third_serial": "1570863273860",
        "success_time": "2019-10-12 14:54:33",
        "obtain_value": "1000",
        "url": "http://660pp.net"
    },
    "serial": "1570863273857",
    "sign": "DF4E35D0E6B3434A2D70FB29BF999ECC",
    "payment": [
        {
            "park_uuid": "49f0cc52-e8c7-41e3-b54d-af666b8cc11a",
            "business": "car:parking:cashier",
            "pay_serial": "1570863273862",
            "pay_value": 60000,
            "merchant": "62626601",
            "plate": "川A660P1",
            "value": 60000
        },
        {
            "park_uuid": "49f0cc52-e8c7-41e3-b54d-af666b8cc11a",
            "business": "car:parking:cashier",
            "pay_serial": "1570863273862",
            "pay_value": 40000,
            "merchant": "62626601",
            "plate": "川A660P2",
            "value": 40000
        }
    ],
    "app_id": "op660pp",
    "sign_type": "MD5",
    "buyer": {
        "tax_no": "1570863273858",
        "tax_type": "1",
        "company_name": "深圳市神州路路通网络科技有限公司",
        "mobile": "0755-86261859",
        "email": "dev@660pp.com",
        "company_mobile": "0755-86261859",
        "company_address": "深圳市南山区粤海街道科技园讯美科技广场3号楼"
    },
    "timestamp": "1570863273873"
}
```



### 3 支付业务

#### <a id="parking_enter">3.1 支付结果同步</a>

- 描述
合作方实现改接口用于接受支付结果通知, 若通知未能返回`1001`不成功则会重复通知多次。

- 请求参数

| 字段           | 类型   | 必须 | 说明               |
| -------------- | ------ | ---- | ------------------ |
| merchant      | string | Y    | 商户号     |
| order      | string | Y    | 支付订单号     |
| channel | string | Y    | 支付渠道       |
| pay_serial     | string | Y    | 平台支付流水       |
| value      | string | Y    | 支付金额，单位：分 |
| status      | short | Y    | 交易状态: 1支付成功、-1失败、0支付中 |

- 响应参数

| 字段    | 类型   | 必须 | 说明             |
| ------- | ------ | ---- | ---------------- |
| code    | string | Y    | 业务处理状态码   |
| message | string | N    | 业务处理状态说明 |
| hint | string | N    | 提示说明 |


### 附录

#### A 业务状态码

| 状态码 | 说明         |
| ------ | ------------ |
| 1001   | 请求成功     |
| 1400   | 请求参数错误 |
| 1500   | 服务内部错误 |
| ...    | ...          |

#### B 车牌颜色 `plate_color`

| 颜色值 | 说明 |
| ------ | ---- |
| -1     | 未知 |
| 1      | 蓝色 |
| 2      | 黄色 |
| 3      | 白色 |
| 4      | 黑色 |
| 5      | 绿色 |

#### C 车类 `car_type`

| 颜色值 | 说明           |
| ------ | -------------- |
| -1     | 未知           |
| 1      | 临停车辆       |
| 2      | 包月车辆       |
| 3      | 贵宾，免费车辆 |
| 4      | 储值车辆       |

