# 广告 JS SDK 接入指南

本文档介绍如何通过 JS SDK 接入广告功能，包括广告节点设置、广告渲染及手动控制渲染的实现。

---

## 接入流程

### 1. 引入 SDK

在页面中引入 P云提供的 JS SDK：
- 切勿直接把文件保存本地引入将无法收到安全更新!!!

```html
<script src="https://assets.4pyun.com/js/4pyun-sdk-1.1.1.min.js"></script>
```

---

### 2. 设置广告节点

在页面中添加广告节点容器：

```html
<div id="image-banner"></div>
```

---

### 3. 渲染广告

通过 `pyun.advertingPull` 方法渲染广告，以下是完整的代码示例：

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

### 4. 手动控制广告渲染

SDK 默认会直接渲染广告节点。如果需要手动控制渲染（例如通过弹窗形式展示广告），可以按照以下方式实现：

#### HTML 结构

```html
<div id="banner-wrapper" style="display:none">
    <div id="image-banner"></div>
</div>
```

#### JS 实现

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
    },
    success: (data) => {
        // 广告加载成功回调
        console.log('广告加载成功:', data);
        // 显示弹窗
        document.getElementById('banner-wrapper').style.display = 'block';
    },
    failure: (cause) => {
        // 广告加载失败回调
        console.warn('广告加载失败:', cause);
    },
});
```

---

## 测试参数

以下是测试时可使用的参数示例：

```js
app_id = 'op00961963581daa7';
```

---

## 注意事项

1. **必填参数**：
   - `el`、`app_id`、`space_id`、`displayed`、`provider_code` 和 `external` 为必填参数，确保正确传递。
   - `personas` 为选填参数，用于精准投放。

2. **回调处理**：
   - 在 `success` 回调中处理广告加载成功的逻辑，例如显示广告或弹窗。
   - 在 `failure` 回调中处理广告加载失败的逻辑，例如记录日志或提示用户。

3. **用户体验**：
   - 建议在广告加载过程中添加加载提示或进度条，提升用户体验。

---

以上是广告 JS SDK 的完整接入流程。如有疑问，请联系技术支持。