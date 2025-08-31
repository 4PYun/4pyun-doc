# 会员券包产品接入指南

本文档介绍如何接入会员券包产品的支付流程，并在支付按钮点击时添加广告拦截逻辑。

接入该场景前须先引入广告SDK

- [H5 JS-SDK接入](./adverting-jsapi.html)
- [小程序广告组件接入](./adverting-jsapi.html)

**<font color="red">特别说明：</font>**

由于券包业务特殊性需要流量主拉取广告必须传递当前原始订单交易金额`trade_value`保证广告投放效果！

*广告拉取示例:*

```js
pyun.advertingPull({
    el: 'image-banner', // 必填，广告节点的 ID
    app_id: '由 P 云分配的接入应用 ID', // 必填
    openid: '合作方用户 ID，用于精准投放追踪用户', // 选填
    space_id: '由 P 云分配的广告位 ID', // 必填
    displayed: 1, // 必填，是否展示广告
    provider_code: '流量主编号', // 必填，项目流量主编号，兼容 P 云或客户内部项目编号
    external: 1, // 必填，外部流量主标志：0 表示否，1 表示是（仅当 provider_code 非 P 云提供时设置为 1）
    personas: { // 选填，为精准投放传入业务参数
        plate: '粤BAAAAAA', // 车牌信息，至少包含前两位
        trade_value: 100, // 交易金额, 用于提升广告投放点击率, 单位: 分
    },
    success: (data) => {
        // 广告加载成功回调
        console.log('广告加载成功:', data);
    },
    failure: (cause) => {
        // 广告加载失败回调
        console.warn('广告加载失败:', cause);
    },
});

```

---

# 1. 方法说明

广告事件拦截（如会员券包/权益购买拦截弹窗）。

**方法签名：**
```javascript
/**
 * Function callback：拦截回调函数，参数为对象，常见字段如下：
 * - action：`REDIRECT`（跳转购买）或 `NONE`（放弃）
 * - redirect：跳转方法（当 action 为 `REDIRECT` 时可用）
 * Boolean enforce：是否强制执行拦截（可选，默认 false）
 */
advertingIntercept(callback, enforce)
```

---

# 2 接入流程

## 2.1 H5接入指引

### 2.1.1 交互逻辑

在支付按钮点击时，需调用广告拦截逻辑 `pyun.advertingIntercept`。根据拦截回调的 `event.action` 值，执行不同的业务逻辑：

- **`REDIRECT`**：用户确认购买，业务方需下单并通过广告组件完成支付跳转。
- **其他值**：用户取消购买，业务方继续自己的支付流程。

**代码示例：**

```javascript
// STEP-1: 拦截广告流程
if (pyun && typeof pyun.advertingIntercept == 'function') {
    pyun.advertingIntercept((event) => {
        if (event.action === 'REDIRECT') {
            // 用户确认购买
            alert(`ACTION=${event.action}, 用户确认购买，业务方下单后由广告组件跳转完成业务支付和广告产品购买!`);
        } else {
            // 用户取消购买
            alert(`ACTION=${event.action}, 用户取消购买，业务方继续自己的支付流程即可!`);
        }
        this.doRealPay(event);
    });
} else {
    // 说明JS SDK未正确加载，需检查原因，业务方继续自己的支付流程即可!
    this.doRealPay(event);
}
```

---

### 2.1.2 支付逻辑实现

在广告拦截回调中，调用 `doRealPay` 方法完成支付逻辑。以下是支付逻辑的实现示例：

```javascript
function doRealPay(event) {
    /**
     * 调用支付接口
     * 示例函数, 替换成自己的方法!!!
     */
    payment((reply) => {
        /**
         * 参数说明：
         * - `depute_id`：托收订单ID, 交易预下单返回
         */
        let args = {
            depute_id: reply.payload.pay_id,
        };

        // STEP-2: 根据拦截结果跳转支付
        if (event && event.action === 'REDIRECT') {
            event.redirect(args); // 广告组件跳转支付
        } else {
            // 正常支付流程
            console.log('正常跳转支付');
        }
    });
}
```

## 2.2 小程序接入指引

### 2.2.1 半屏小程序的申请
1. 打开 [微信公众平台](https://mp.weixin.qq.com/)
2. 登录微信公众平台后，找到 **设置**-**第三方设置** 滑到页面下方找到**半屏小程序管理**-**我调用的**-**添加**
3. 输入PP停车的小程序APPID：**wxa204074068ad40ef** 并添加即可

### 2.2.2 交互逻辑

在支付按钮点击时，需调用选中广告节点执行拦截逻辑 `advertingIntercept`。根据拦截回调的 `event.action` 值，执行不同的业务逻辑：

- **`REDIRECT`**：用户确认购买，业务方需下单并通过广告组件完成支付跳转。
- **其他值**：用户取消购买，业务方继续自己的支付流程。

**代码示例：**


```
index.wxml

<pp-adv
    id="pyun"
    ...其余广告参数
/>

```


```javascript
index.js

// STEP-1: 拦截广告流程
this.selectComponent('#pyun').advertingIntercept((event) => {
    if (event.action === 'REDIRECT') {
        // 用户确认购买
        console.log(`ACTION=${event.action}, 用户确认购买，业务方下单后由广告组件跳转完成业务支付和广告产品购买!`);
    } else {
        // 用户取消购买
        console.log(`ACTION=${event.action}, 用户取消购买，业务方继续自己的支付流程即可!`);
    }
    this.doRealPay(event);
})
```

### 2.2.3 支付逻辑实现

在广告拦截回调中，调用 `doRealPay` 方法完成支付逻辑。以下是支付逻辑的实现示例：

```javascript
doRealPay(event) {
    /**
     * 调用支付接口
     * 示例函数, 替换成自己的方法!!!
     */
    payment((reply) => {
        /**
         * 参数说明:
         * - `depute_id`：托收订单ID, 交易预下单返回
         */
        let args = {
            depute_id: reply.payload.pay_id,
        };

        // STEP-2: 根据拦截结果跳转支付
        if (event && event.action === 'REDIRECT') {
            event.redirect(args); // 广告组件跳转支付(将会执行小程序跳转，需要申请半屏跳转)
        } else {
            // 正常支付流程
            console.log('正常跳转支付');
        }
    });
}
```

#### 2.2.4 小程序支付回调

支付成功后会返回对应的参数(目标小程序可在 App.onLaunch，App.onShow 中获取到这份数据。) [详见文档](https://developers.weixin.qq.com/miniprogram/dev/api/navigate/wx.navigateToMiniProgram.html)

返回参数参考[支付结果异步通知](./../api/trade-notify.html)

---

# 3 注意事项

1. **拦截回调处理**：
   - 确保在 `event.action === 'REDIRECT'` 时调用 `event.redirect(args)`，完成支付跳转。
   - 其他情况下，需根据业务需求处理支付流程。

2. **参数校验**：
   - `depute_id`取[交易预请求](./trade-prepare.html)返回的`pay_id`字段。

3. **用户体验**：
   - 在支付流程中，建议添加加载提示或进度条，提升用户体验。

---

## 示例场景

- 用户点击支付按钮后，广告拦截逻辑会判断用户是否确认购买。
- 如果用户确认购买，广告组件将跳转至支付页面完成支付。
- 如果用户取消购买，业务方可继续执行自己的支付流程。

---

以上是会员券包/权益产品接入的完整流程。如有疑问，请联系技术支持。