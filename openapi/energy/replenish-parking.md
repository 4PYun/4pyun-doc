# 充电减免停车费

``` sequence
title: 充电减免停车费下发流程

充电桩  -> P云 : 充电完成上报充电记录
P云 -> 停车场  : 根据充电信息下发停车优惠
```

### 请求地址

```
https://api.4pyun.com/gate/1.0/energy/internal/replenish
```

### 调用方式

```
HTTP FORM 表单提交
```

### 特殊说明

**注意事项：**

1. 需在充电结束时触发调用；
2. 一次充电单号唯一；
3. 下发停车优惠时 `vin` 需上传车牌；
4. 充电开始结束时间不止要转换为UTC格式，还要在北京时间的基础上减8小时。例如北京时间 `2023-04-10 12:00:00`，转换UTC后为：`2023-04-10T04:00:00.000Z`；

**参数说明：**

1. `appId` 和 `appSecret` 由P云平台分配；
2. `station_uuid` 为P云平台 `充电服务商` 板块充电站的编号，需先创建充电站才可获得；


### 请求参数

| 参数              | 类型     | 必须 | 描述                                         | 示例值                              |
|-----------------|--------|----|--------------------------------------------|----------------------------------|
| app_id          | string | Y  | 平台分配的接入应用ID                                | op1234567723122                  |
| timestamp       | string | Y  | 请求发送时间戳(ms), 用于防重放攻击服务器直接受和服务器时间差10分钟内的请求。 | 1552976318722                    |
| sign            | string | Y  | 请求数据签名                                     | C65FCAC2D3FB5E2D3D4AD93DD20C8C39 |
| station_uuid    | string | Y  | PP充电平台充电站UUID（需先在PP平台创建充电站才可获得）            |                                  |
| device_no       | string | Y  | 本地设备编号（传递本地设备编号即可）                         | DEVICE1                          |
| port_no         | string | Y  | 本地设备枪号（传递本地设备枪号即可）                         | 1                                |
| replenish_order | string | Y  | 本地充电单号                                     |                                  |
| start_time      | string | Y  | 开始充电，`yyyy-MM-dd'T'HH:mm:ss'Z'`            | 2023-04-10T17:37:30Z             |
| end_time        | string | Y  | 结束充电，`yyyy-MM-dd'T'HH:mm:ss'Z'`            | 2023-04-10T17:37:30Z             |
| vin             | string | Y  | 车辆标识。充电停车优惠场景下该值必须传递停车记录的车牌                | 川A660PP                          |
| quantity        | string | Y  | 充电度数，0.001Kwh，1表示0.001Kwh，例如1度传递1000即可     | 1000                             |
| energy_value    | string | Y  | 电费，分                                       | 1                                |
| fee_value       | string | Y  | 服务费，分                                      | 1                                |
| total_value     | string | Y  | 总金额，分                                      | 2                                |
| energy_code     | string | Y  | 充电类型；CN_AC：慢充；CN_DC：快充                     | CN_AC                            |
| ...             |        |    |                                            |                                  |

### 请求示例

```json
{
    "fee_value": 561,
    "total_value": 1156,
    "quantity": "5682",
    "replenish_order": "20230410183256K7fh6t",
    "end_time": "2023-04-10T18:32:56Z",
    "sign": "4EC351C604ECB191964EB67565AA8E87",
    "device_no": "S1",
    "energy_code": "CN_AC",
    "start_time": "2023-04-10T17:32:56Z",
    "energy_value": 595,
    "vin": "川A660N2",
    "station_uuid": "8f5fdb60-9374-4c11-bdc2-a32d8369258c",
    "app_id": "op00961963581daa7",
    "timestamp": "1681122776000",
    "port_no": "1"
}
```

### 响应参数
| 字段名称    | 字段说明    |   类型   | 必填 | 备注              |
|:--------|:--------|:------:|:--:|:----------------|
| code    | 请求状态码   | string | Y  | 1001            |
| message | 返回描述    | string | Y  | 返回描述            |
| hint    | 返回错误说明  | string | N  | 返回具体错描述指导       |
| seqno   | 服务器日志标示 | string | Y  | 查日志用到查问题尽量提供这个值 |


### 响应码
| 值    | 描述 |
|------| --- |
| 1001 | 成功 |
| 1002 | 停车记录不存在 |
| 其它   | 参考message |

### 响应示例

> 正常返回

```json
{
    "code": "1001",
    "seqno": "32632654459897019054641535481549",
    "data_node": "CN-South/DEV-1",
    "time_cost": 148,
    "message": "减免成功"
}
```

### 示例代码

```java
@Test
public void execute() throws APIRemoteException {
    SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd'T'HH:mm:ss.SSS'Z'");
    format.setTimeZone(TimeZone.getTimeZone("UTC"));
        
    // 平台分配的密钥
    String appId = "op00961963581daa7";
    String appSecret = "6409292d66625a2a0912acfc61ed956c";
    // 接口地址
    String url = "https://dev-api.4pyun.com/gate/1.0/energy/internal/replenish";

    // 请求参数
    int energyValue = Integer.parseInt(RandomUtils.randomNumeric(3));
    int feeValue = Integer.parseInt(RandomUtils.randomNumeric(3));
    TreeMap<String, String> params = new TreeMap<String, String>() {{
        put("app_id", appId);
        put("timestamp", String.valueOf(System.currentTimeMillis()));

        put("station_uuid", "8f5fdb60-9374-4c11-bdc2-a32d8369258c");
        put("device_no", "S1");
        put("port_no", "1");
        put("energy_code", "CN_AC");
        put("replenish_order", DateUtils.format(new Date(), DateUtils.DEFAULT_TIME_FORMAT) + RandomUtils.randomAlphanumeric(6));
        put("start_time", format.format(DateTime.now().minusHours(1).toDate()));
        put("end_time", format.format(new Date()));
        put("vin", "川A660N2");
        put("quantity", RandomUtils.randomNumeric(4));
        put("energy_value", String.valueOf(energyValue));
        put("fee_value", String.valueOf(feeValue));
        put("total_value", String.valueOf(energyValue + feeValue));
    }};

    /*
     * 计算签名
     * S1: 先过滤掉 sign 和值为空的参数；
     * S2: 在将所有业务参数按照参数名进行升序排序；
     * S3: 以 `HTTP URL` 风格拼接成待签名的字符串，如：`a=1&b=2&c=123`
     * S4: 再将密钥以 `&app_secret=您的密钥` 形式拼接到上一步待签名的字符串后面，如：`a=1&b=2&c=123&app_secret=您的密钥`
     * S5: 使用标准 MD5 算法对上述字符串进行签名（忽略大小写）；
     * S6: 将其写入 `sign` 进行 `HTTP POST` 请求即可。
     */
    String plain = params.entrySet()
            .stream()
            .filter(e -> StringUtils.isNotBlank(e.getValue()))
            .filter(e -> !"sign".equals(e.getKey()))
            .map(e -> e.getKey() + "=" + e.getValue())
            .collect(Collectors.joining("&"));
    plain += "&app_secret=" + appSecret;
    String sign = MD5.encryptHEX(plain);
    params.put("sign", sign);
    // plain: app_id=op00961963581daa7&device_no=S1&end_time=2023-04-12T09:40:18Z&energy_code=CN_AC&energy_value=676&fee_value=341&mobile=19925333063&port_no=1&quantity=9033&replenish_order=20230412094017HYynTf&start_time=2023-04-12T08:40:18Z&station_uuid=8f5fdb60-9374-4c11-bdc2-a32d8369258c&timestamp=1681263617993&total_value=1017&vin=川A660N2&app_secret=6409292d66625a2a0912acfc61ed956c, sign: D47024DF345A1143F080401FC50A2B8D
    log.info("plain: {}, sign: {}", plain, sign);

    // HTTP 请求
    this.doPost(url, params);
}
```
