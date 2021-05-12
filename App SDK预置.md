# App SDK 预置事件与预置属性

## 预置事件及其携带的预置属性

<table>
    <tr>
        <th>事件ID</th>
        <th>事件显示名</th>
        <th>携带属性ID</th>
        <th>携带属性显示名</th>
        <th>属性数据类型</th>
        <th>事件触发点</th>
    </tr>
    <tr>
        <td rowspan="4">$AppStart</td>
        <td rowspan="4">App启动</td>
        <td>$is_first_time</td>
        <td>是否首次</td>
        <td>布尔值</td>
        <td rowspan="4">启动App或从后台切换进入App时触发</td>
    </tr>
    <tr>
        <td>$resume_from_background</td>
        <td>是否从后台唤醒</td>
        <td>布尔值</td>
    </tr>
    <tr>
        <td>$screen_name</td>
        <td>页面名称</td>
        <td>字符串</td>
    </tr>
    <tr>
        <td>$title</td>
        <td>页面标题</td>
        <td>字符串</td>
    </tr>
    <tr>
        <td rowspan="3">$AppEnd</td>
        <td rowspan="3">App关闭</td>
        <td>$event_duration</td>
        <td>停留时长</td>
        <td>数值</td>
        <td rowspan="3">退出App或App进入后台时触发</td>
    </tr>
    <tr>
        <td>$screen_name</td>
        <td>页面名称</td>
        <td>字符串</td>
    </tr>
    <tr>
        <td>$title</td>
        <td>页面标题</td>
        <td>字符串</td>
    </tr>
    <tr>
        <td rowspan="4">$AppViewScreen</td>
        <td rowspan="4">App浏览页面</td>
        <td>$screen_name</td>
        <td>页面名称</td>
        <td>字符串</td>
        <td rowspan="4">打开一个Activity/ViewController页面时触发</td>
    </tr>
    <tr>
        <td>$title</td>
        <td>页面标题</td>
        <td>字符串</td>
    </tr>
    <tr>
        <td>$url</td>
        <td>页面地址</td>
        <td>字符串</td>
    </tr>
    <tr>
        <td>$referrer</td>
        <td>前向地址</td>
        <td>字符串</td>
    </tr>
    <tr>
        <td rowspan="8">$AppClick</td>
        <td rowspan="8">App元素点击</td>
        <td>$screen_name</td>
        <td>页面名称</td>
        <td>字符串</td>
        <td rowspan="8">点击控件时触发</td>
    </tr>
    <tr>
        <td>$title</td>
        <td>页面标题</td>
        <td>字符串</td>
    </tr>
    <tr>
        <td>$element_position</td>
        <td>元素位置</td>
        <td>字符串</td>
    </tr>
    <tr>
        <td>$element_id</td>
        <td>元素ID</td>
        <td>字符串</td>
    </tr>
    <tr>
        <td>$element_content</td>
        <td>元素内容</td>
        <td>字符串</td>
    </tr>
    <tr>
        <td>$element_type</td>
        <td>元素类型</td>
        <td>字符串</td>
    </tr>
    <tr>
        <td>$element_selector</td>
        <td>元素选择器</td>
        <td>字符串</td>
    </tr>
    <tr>
        <td>$element_path</td>
        <td>元素路径</td>
        <td>字符串</td>
    </tr>
    <tr>
        <td rowspan="10">$AppInstall</td>
        <td rowspan="10">App激活</td>
        <td>$browser</td>
        <td>浏览器名</td>
        <td>字符串</td>
        <td rowspan="10">需要调用 trackInstallation 接口采集采集该事件，且App 安装后首次打开才会触发，第二次打开不会再触发。</td>
    </tr>
    <tr>
        <td>$utm_source</td>
        <td>广告系列来源</td>
        <td>字符串</td>
    </tr>
    <tr>
        <td>$utm_medium</td>
        <td>广告系列媒介</td>
        <td>字符串</td>
    </tr>
    <tr>
        <td>$utm_term</td>
        <td>广告系列字词</td>
        <td>字符串</td>
    </tr>
    <tr>
        <td>$utm_content</td>
        <td>广告系列内容</td>
        <td>字符串</td>
    </tr>
    <tr>
        <td>$utm_campaign</td>
        <td>广告系列名称</td>
        <td>字符串</td>
    </tr>
    <tr>
        <td>$utm_matching_type</td>
        <td>渠道追踪匹配模式</td>
        <td>字符串</td>
    </tr>
    <tr>
        <td>$mactched_key</td>
        <td>渠道匹配关键词</td>
        <td>字符串</td>
    </tr>
    <tr>
        <td>$mactched_key_list</td>
        <td>渠道匹配关键词列表</td>
        <td>字符串</td>
    </tr>
    <tr>
        <td>$ios_install_source</td>
        <td>Ios安装来源</td>
        <td>字符串</td>
    </tr>
    <tr>
        <td>$AppCrashed</td>
        <td>App崩溃</td>
        <td>$app_crashed_reason</td>
        <td>崩溃原因</td>
        <td>字符串</td>
        <td>App崩溃时触发</td>
    </tr>
    <tr>
        <td>$AppStartPassively</td>
        <td>App被动启动</td>
        <td>$app_state</td>
        <td>App状态</td>
        <td>字符串</td>
        <td>iOS APP被系统调用</td>
    </tr>
</table>

## 所有事件都携带的预置属性

| 属性ID              | 属性数据类型 | 属性显示名称 | 属性说明                                                     |
| ------------------- | ------------ | ------------ | ------------------------------------------------------------ |
| $app_version        | 字符串       | 应用版本     | APP  的应用版本                                              |
| $lib                | 字符串       | SDK类型      | SDK  类型，比如 Android/iOS                                  |
| $lib_version        | 字符串       | SDK版本      | SDK  版本                                                    |
| $manufacturer       | 字符串       | 设备制造商   | 设备制造商                                                   |
| $model              | 字符串       | 设备型号     | 设备型号                                                     |
| $os                 | 字符串       | 操作系统     | 操作系统                                                     |
| $os_version         | 字符串       | 操作系统版本 | 操作系统版本                                                 |
| $screen_height      | 数值         | 屏幕高度     | 屏幕高度                                                     |
| $screen_width       | 数值         | 屏幕宽度     | 屏幕宽度                                                     |
| $wifi               | 布尔值       | 是否 WiFi    | 事件触发时是否为 WiFi                                        |
| $carrier            | 字符串       | 运营商名称   | 事件触发时设备  SIM 卡的运营商名称                           |
| $network_type       | 字符串       | 网络类型     | 事件触发时的网络类型                                         |
| $is_first_day       | 布尔值       | 是否首日访问 | 表示是否是首日触发事件                                       |
| $ip                 | 字符串       | IP           | 后端通过解析  HTTP 请求而得到                                |
| $country            | 字符串       | 国家         | 由IP解析得到                                                 |
| $province           | 字符串       | 省份         | 由IP解析得到                                                 |
| $city               | 字符串       | 城市         | 由IP解析得到                                                 |
| $device_id          | 字符串       | 设备ID       | Android  端主要取 Android ID ，iOS 端先尝试获取 IDFA，如果获取不到，则取 IDFV |
| $screen_orientation | 字符串       | 屏幕方向     | 只有在开启 enableTrackScreenOrientation: 时才会采集          |
| $latitude           | 数值         | GPS信息      | 纬度*10的六次方，只有在开启  enableTrackGPSLocation: 时才会采集 |
| $longitude          | 数值         | GPS信息      | 经度*10的六次方，只有在开启  enableTrackGPSLocation: 时才会采集 |

## 追踪并进行渠道匹配和回传时的预置事件属性

| 属性ID                     | 属性数据类型 | 属性显示名称         | 属性说明                                                     |
| -------------------------- | ------------ | -------------------- | ------------------------------------------------------------ |
| $channel_device_info       | 字符串       | 是否不进行追踪回调   | App  渠道追踪自定义事件时进行渠道匹配，可以调用 - trackChannelEvent:properties:  对待匹配的事件进行追踪，后台匹配到渠道信息后会将结果回传到渠道商。该字段记录用于渠道匹配的设备信息，比如 IMEI、Android ID、Mac  地址、IDFA。具体使用，可以参考 SDK 的高级功能模块。 |
| $is_channel_callback_event | 布尔值       | 是否进行渠道匹配回调 | App  渠道追踪自定义事件时进行渠道匹配，可以调用 - trackChannelEvent:properties: 对待匹配的事件进行追踪，后台匹配到渠道信息后会将结果回传到渠道商。具体使用，可以参考  SDK 的高级功能模块。 |

## 预置用户属性

| 属性ID             | 属性数据类型 | 属性显示名称       | 属性说明                                                     |
| ------------------ | ------------ | ------------------ | ------------------------------------------------------------ |
| $first_visit_time  | 日期时间     | 首次访问时间       | 调用  trackInstallation 接口后，新用户首次启动App， 会给此属性赋值 |
| $utm_source        | 字符串       | 首次广告系列来源   |                                                              |
| $utm_medium        | 字符串       | 首次广告系列媒介   |                                                              |
| $utm_term          | 字符串       | 首次广告系列字词   |                                                              |
| $utm_content       | 字符串       | 首次广告系列内容   |                                                              |
| $utm_campaign      | 字符串       | 首次广告系列名称   |                                                              |
| $utm_matching_type | 字符串       | 渠道追踪匹配模式   |                                                              |
| $matched_key       | 字符串       | 渠道匹配关键字     |                                                              |
| $matching_key_list | 字符串       | 渠道匹配关键字列表 | 渠道匹配关键字列表，包含所有可能用于渠道匹配的  key          |