# 小程序接入广告

## 1.引入npm包

    npm install ad-4pyun-sdk

## 2.构建

*   uni-app框架：[小程序自定义组件支持](https://uniapp.dcloud.net.cn/tutorial/miniprogram-subject.html#%E5%B0%8F%E7%A8%8B%E5%BA%8F%E8%87%AA%E5%AE%9A%E4%B9%89%E7%BB%84%E4%BB%B6%E6%94%AF%E6%8C%81/)

1.  在根目录新建wxcomponents文件夹
2.  打开node\_modules/ad-4pyun-sdk文件，copy目录中整个src文件到wxcomponents目录下，按需重命名使用
3.  目录结构



    ┌─wxcomponents              微信小程序自定义组件存放目录
    │   └──ad-4pyun-sdk      微信小程序自定义组件
    │        ├─index.js
    │        ├─index.wxml
    │        ├─index.json
    │        └─index.wxss
    ├─pages
    ├─main.js
    ├─App.vue
    ├─manifest.json
    └─pages.json

*   小程序原生：[小程序npm支持](https://developers.weixin.qq.com/miniprogram/dev/devtools/npm.html)

1.  小程序开发者工具 -> 详情(工具右上角) -> 本地设置 -> 使用npm模块
2.  小程序开发者工具 -> 工具 -> 构建 npm
3.  构建成功后小程序代码包中将产出 "miniprogram\_npm" 文件夹

## 3.使用

*   uni-app框架：[小程序自定义组件支持](https://uniapp.dcloud.net.cn/tutorial/miniprogram-subject.html#%E5%B0%8F%E7%A8%8B%E5%BA%8F%E8%87%AA%E5%AE%9A%E4%B9%89%E7%BB%84%E4%BB%B6%E6%94%AF%E6%8C%81/)

1.  找到需要引入广告的页面
2.  页面的json文件中做如下配置



    //pages.json
    {
      "pages": [
        {
          "path": "ad-4pyun-sdk/ad-4pyun-sdk,
          "style": {
              "usingComponents": {
                  "ad-4pyun-sdk": "/wxcomponents/ad-4pyun-sdk/index"
              }
          }
        }
      ]
    }

1.  在页面中引入该组件



    <template>
      <ad-4pyun-sdk 
        space_id="{{space_id}}" 
        openid="{{openid}}"
        app_id="{{app_id}}"
        provider_code="{{provider_code}}"
        personas="{{personas}}"
        external="1"  
      >
      </ad-4pyun-sdk>
    </template>

    <script>
      export default {
        data() {
            return {
                space_id: '',
                openid: '',
                app_id: '',
                provider_code: '',
                personas: {
                    mobile: '131xxxxxxxx',
                    plate: '粤BAAAAA',
                }
            }
        }
      }
    </script>

*   小程序原生

1.  找到需要引入广告的页面
2.  页面的json文件中做如下配置



    // index.json
    {
        "usingComponents": {
            "ad-4pyun-sdk": "ad-4pyun-sdk"
        }
    }

1.  在页面中引入该组件

```
// index.wxml

<ad-4pyun-sdk 
  space_id="{{space_id}}" 
  openid="{{openid}}"
  app_id="{{app_id}}"
  provider_code="{{provider_code}}"
  personas="{{personas}}"
>
</ad-4pyun-sdk>

// index.js
data: {
  space_id: '',
  openid: '',
  app_id: ''
  provider_code: '',
  personas: {
    mobile: '131xxxxxxxx',
    plate: '粤BAAAAA',
  }
}

```

## 4.参数说明

| 变量             | 类型     | 必填 | 可选值          | 说明                                        |
| -------------- | ------ | -- | ------------ | ----------------------------------------- |
| env            | string | 否  | release、test   | 请求环境<br />release：正式环境（默认）<br />test：测试环境 |
| space\_id      | string | 是  | -               | 由P云分配广告位ID                                |
| openid         | string | 是  | -               | 合作方用户ID，用于精准投放追踪用户                        |
| app\_id        | string | 是  | -               | 由P云分配接入应用ID                               |
| provider\_code | string | 是  | -               | 流量主编号                                     |
| personas       | object | 否  | -               | 为精准投放传入业务，详见下方说明                          |
| external       | string | 否  |  '0'、'1'(默认) | 外部流量主标志: 0 否, 1是(仅当provider_code非P云提供设置为1) |

### personas参数说明

| 变量     | 类型     | 必填 | 可选值 | 说明  |
| ------ | ------ | -- | --- | --- |
| plate  | string | 否  | -   | 车牌  |
| mobile | string | 否  | -   | 手机号 |

## 5.特殊场景说明

*   跳转webview的情况：
    当广告点击后跳转的目标是H5链接的情况下，会跳转到PP停车小程序里的webview打开H5

**需要准备的工作：**
1. 打开 [微信公众平台](https://mp.weixin.qq.com/)
2. 登录微信公众平台后，找到 **设置**-**第三方设置** 滑到页面下方找到**半屏小程序管理**-**我调用的**-**添加**
3. 输入PP停车的小程序APPID：**wxa204074068ad40ef** 并添加即可

## 6.请求域名配置
1. 打开 [微信公众平台](https://mp.weixin.qq.com/)
2. 开发管理-开发设置-服务器域名
3. 配置下该域名https://ad-api.4pyun.com
