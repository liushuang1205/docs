# App SDK预置

## 预置事件及其携带的预置属性

| 事件ID | 事件显示名 | 携带属性ID | 携带属性显示名 | 属性数据类型 | 事件触发点 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| $AppStart | App启动 | $is\_first\_time | 是否首次 | 布尔值 | 启动App或从后台切换进入App时触发 |
| $resume\_from\_background | 是否从后台唤醒 | 布尔值 |  |  |  |
| $screen\_name | 页面名称 | 字符串 |  |  |  |
| $title | 页面标题 | 字符串 |  |  |  |
| $AppEnd | App关闭 | $event\_duration | 停留时长 | 数值 | 退出App或App进入后台时触发 |
| $screen\_name | 页面名称 | 字符串 |  |  |  |
| $title | 页面标题 | 字符串 |  |  |  |
| $AppViewScreen | App浏览页面 | $screen\_name | 页面名称 | 字符串 | 打开一个Activity/ViewController页面时触发 |
| $title | 页面标题 | 字符串 |  |  |  |
| $url | 页面地址 | 字符串 |  |  |  |
| $referrer | 前向地址 | 字符串 |  |  |  |
| $AppClick | App元素点击 | $screen\_name | 页面名称 | 字符串 | 点击控件时触发 |
| $title | 页面标题 | 字符串 |  |  |  |
| $element\_position | 元素位置 | 字符串 |  |  |  |
| $element\_id | 元素ID | 字符串 |  |  |  |
| $element\_content | 元素内容 | 字符串 |  |  |  |
| $element\_type | 元素类型 | 字符串 |  |  |  |
| $element\_selector | 元素选择器 | 字符串 |  |  |  |
| $element\_path | 元素路径 | 字符串 |  |  |  |
| $AppInstall | App激活 | $browser | 浏览器名 | 字符串 | 需要调用 trackInstallation 接口采集采集该事件，且App 安装后首次打开才会触发，第二次打开不会再触发。 |
| $utm\_source | 广告系列来源 | 字符串 |  |  |  |
| $utm\_medium | 广告系列媒介 | 字符串 |  |  |  |
| $utm\_term | 广告系列字词 | 字符串 |  |  |  |
| $utm\_content | 广告系列内容 | 字符串 |  |  |  |
| $utm\_campaign | 广告系列名称 | 字符串 |  |  |  |
| $utm\_matching\_type | 渠道追踪匹配模式 | 字符串 |  |  |  |
| $mactched\_key | 渠道匹配关键词 | 字符串 |  |  |  |
| $mactched\_key\_list | 渠道匹配关键词列表 | 字符串 |  |  |  |
| $ios\_install\_source | Ios安装来源 | 字符串 |  |  |  |
| $AppCrashed | App崩溃 | $app\_crashed\_reason | 崩溃原因 | 字符串 | App崩溃时触发 |
| $AppStartPassively | App被动启动 | $app\_state | App状态 | 字符串 | iOS APP被系统调用 |

## 所有事件都携带的预置属性

| 属性ID | 属性数据类型 | 属性显示名称 | 属性说明 |
| :--- | :--- | :--- | :--- |
| $app\_version | 字符串 | 应用版本 | APP  的应用版本 |
| $lib | 字符串 | SDK类型 | SDK  类型，比如 Android/iOS |
| $lib\_version | 字符串 | SDK版本 | SDK  版本 |
| $manufacturer | 字符串 | 设备制造商 | 设备制造商 |
| $model | 字符串 | 设备型号 | 设备型号 |
| $os | 字符串 | 操作系统 | 操作系统 |
| $os\_version | 字符串 | 操作系统版本 | 操作系统版本 |
| $screen\_height | 数值 | 屏幕高度 | 屏幕高度 |
| $screen\_width | 数值 | 屏幕宽度 | 屏幕宽度 |
| $wifi | 布尔值 | 是否 WiFi | 事件触发时是否为 WiFi |
| $carrier | 字符串 | 运营商名称 | 事件触发时设备  SIM 卡的运营商名称 |
| $network\_type | 字符串 | 网络类型 | 事件触发时的网络类型 |
| $is\_first\_day | 布尔值 | 是否首日访问 | 表示是否是首日触发事件 |
| $ip | 字符串 | IP | 后端通过解析  HTTP 请求而得到 |
| $country | 字符串 | 国家 | 由IP解析得到 |
| $province | 字符串 | 省份 | 由IP解析得到 |
| $city | 字符串 | 城市 | 由IP解析得到 |
| $device\_id | 字符串 | 设备ID | Android  端主要取 Android ID ，iOS 端先尝试获取 IDFA，如果获取不到，则取 IDFV |
| $screen\_orientation | 字符串 | 屏幕方向 | 只有在开启 enableTrackScreenOrientation: 时才会采集 |
| $latitude | 数值 | GPS信息 | 纬度\*10的六次方，只有在开启  enableTrackGPSLocation: 时才会采集 |
| $longitude | 数值 | GPS信息 | 经度\*10的六次方，只有在开启  enableTrackGPSLocation: 时才会采集 |

## 追踪并进行渠道匹配和回传时的预置事件属性

| 属性ID | 属性数据类型 | 属性显示名称 | 属性说明 |
| :--- | :--- | :--- | :--- |
| $channel\_device\_info | 字符串 | 是否不进行追踪回调 | App  渠道追踪自定义事件时进行渠道匹配，可以调用 - trackChannelEvent:properties:  对待匹配的事件进行追踪，后台匹配到渠道信息后会将结果回传到渠道商。该字段记录用于渠道匹配的设备信息，比如 IMEI、Android ID、Mac  地址、IDFA。具体使用，可以参考 SDK 的高级功能模块。 |
| $is\_channel\_callback\_event | 布尔值 | 是否进行渠道匹配回调 | App  渠道追踪自定义事件时进行渠道匹配，可以调用 - trackChannelEvent:properties: 对待匹配的事件进行追踪，后台匹配到渠道信息后会将结果回传到渠道商。具体使用，可以参考  SDK 的高级功能模块。 |

## 预置用户属性

| 属性ID | 属性数据类型 | 属性显示名称 | 属性说明 |
| :--- | :--- | :--- | :--- |
| $first\_visit\_time | 日期时间 | 首次访问时间 | 调用  trackInstallation 接口后，新用户首次启动App， 会给此属性赋值 |
| $utm\_source | 字符串 | 首次广告系列来源 |  |
| $utm\_medium | 字符串 | 首次广告系列媒介 |  |
| $utm\_term | 字符串 | 首次广告系列字词 |  |
| $utm\_content | 字符串 | 首次广告系列内容 |  |
| $utm\_campaign | 字符串 | 首次广告系列名称 |  |
| $utm\_matching\_type | 字符串 | 渠道追踪匹配模式 |  |
| $matched\_key | 字符串 | 渠道匹配关键字 |  |
| $matching\_key\_list | 字符串 | 渠道匹配关键字列表 | 渠道匹配关键字列表，包含所有可能用于渠道匹配的  key |

