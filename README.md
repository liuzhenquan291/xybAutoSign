# xybAutoSign
校友邦自动签到，支持批量签到签出与云函数部署

### 特性

- 原生支持批量签到与签退
- 同时支持账号密码和小程序登录方式
- 友好的任务异常处理与日志信息输出
- 支持API组件化使用
- 支持Webhooks事件回调
- 支持腾讯云函数(SCF)部署

-----

### 基本使用

#### 需求与依赖

- Python3.6及以上版本
- 安装`requests`库

你可以参考以下任意一条pip命令完成安装

```
1. pip install requests
2. python -m pip install requests
```

> 如果你运行在Linux环境，请视情况将`pip`和`python`替换为`pip3`和`python3`

#### 配置用户信息

在编辑配置文件之前，你需要先确认你的校友邦账号的登录方式，目前支持小程序OpenID登录方式和账号密码登录方式，并确认签到地址的相关信息，并按照以下指引填写相关字段

**若同时填写了两种方式，将优先采用账号密码登录**

##### 账号密码登录

账号密码登录方式，你需要配置用户名和密码

- 用户名`username`
- 密码`password`

##### 小程序OpenID登录

你需要提供你绑定微信帐号的信息

- 小程序OpenID：`openid`
- 小程序UnionID：`unionid`

> 关于`openid`与`unionid`的获取，可参考[CncCbz/xybSign](https://github.com/CncCbz/xybSign/blob/main/README.md)
>
> `unionid`实际上并不会用于登陆验证流程，但为了保险起见建议正确填写

##### 地址信息

在`location`字段中配置以下字段，作为你的签到地址信息

**`behavior`动作已经停用，部分字段已经废弃，但由于兼容性考虑在配置中仍会保留**

- 省份`province`（已弃用）
- 城市`country`（已弃用）
- 区（县）`city`（已弃用）
- 行政区划代码`adcode`，可自行百度进行在线查询，注意不要与邮政编码混淆
- 详细地址`address`

如果你的实习**没有限制签到范围**，将无法获取到定位的坐标信息，则需要额外填写以下字段，未配置坐标信息将会提示相关异常，这通常会发生在集中安排的实习中

- 纬度`lat`
- 经度`lng`

**特别注意：当同时配置经度和纬度为非`0`值时，将视为有效坐标，并无视实习获取的签到坐标，不再以签到范围的坐标为准。请在配置前再三确认你的配置场景，以免导致外勤签到**

> 坐标可以通过[拾取坐标系统](https://api.map.baidu.com/lbsapi/getpoint/index.html)进行获取，需注意百度地图的坐标拾取结果是经度在前

在`accounts.json`配置用户信息，配置的JSON对象父元素为数组(列表)，错误的格式会导致解析错误，默认已经为其配置了一个空配置，请直接修改默认的空值作为第一个用户，以下是其中一个对象的参考示例

**并非所有字段都是必填，请注意对照上文说明**

```javascript
{
    //用于微信登陆
    "openid": "ooruxxxxxxxxxxxxxxxxxxxxxxl0",  //校友邦openId
    "unionid": "oHYxxxxxxxxxxxxxxxxxxxxxxQhE",  //校友邦unionId
    //用于账号密码登录
    "username": "xxxxxx",  //用户名
    "password": "xxxxxx",  //密码
    //以下是签到位置相关信息
    "location": {
        "province": "xx省",  //省份，已弃用
        "country": "xx市",  //城市，已弃用
        "city": "xx区",  //区（县），已弃用
        "adcode": 440000,  //行政区划代码
        "address": "xxxx"  //详细地址
        //以下是无签到范围坐标信息时，需要填写的坐标
        "lat": 0,
        "lng": 0
    }
}
```

#### 使用

直接运行`xyb.py`，将会自动进行批量签到，如需要进行签退，将`347`行位置修改为`xyb.sign_out_all()`

-----

### 高级用法

#### Webhooks

可以在`webhooks.py`中找到预定义的两个hook函数，分别为`on_sign_in`和`on_sign_out`，分别为签到事件和签退事件回调，回调函数只有一个参数`data`，其格式为

**受具体登录方式的影响，部分字段的值可能为空**

```javascript
{
    "openid": "ooruxxxxxxxxxxxxxxxxxxxxxxl0",  //校友邦openId，可能为空
    "username": "xxxxxx"  //登录的用户名，可能为空
    "loginer_id": "00000",  //用户ID，每个用户唯一
    "name": "xxx",  //用户姓名
    "phone": "134xxxxxxxx",  //用户手机号
    "train_type": true,  //自主与集中实习类型，true为自主安排，false为集中实习
    "train_id": "00000",  //实习ID
    "post_type": true,  //签到范围限制，true为存在限制签到范围，false为不存在
    "sign_type": true,  //签到与签出类型，true为签到，false为签出
    "result": true,  //任务执行成功情况
    "is_sign_in": true,  //当前是否已签到
    "is_sign_out": false  //当前是否已签出
}
```

可以自行结合以上提供的事件信息进行额外联动操作，如执行命令，发送web请求，机器人通知，微信公众号通知等

#### 腾讯云函数(SCF)部署

你需要在腾讯云拥有一个账号并[创建新的云函数](https://console.cloud.tencent.com/scf/list-create) ，其中必须配置如下

- 创建方式为**自定义创建**
- 函数类型为**事件函数**
- 函数名称可自取
- 部署方式为**代码部署**
- 运行环境为**Python3.6**
- 提交方法可使用**本地上传zip包**或者**本地上传文件夹**将本项目上传
- 执行方法为`index.main_handler`

展开高级配置

- 内存选择**64MB**或者**128MB**
- 初始化超时时间建议**10秒以上**
- 执行超时时间建议**3-30秒左右**
- 以上内存与超时相关配置请结合账户的数量进行调整
- 网络配置勾选**公网访问**(默认勾选无需额外操作)
- 执行配置建议勾选**异步执行**

触发器配置在这里暂不创建，后续创建函数后再配置

函数创建完成后，进入并选择左侧的触发管理，进入触发器创建页，创建一个触发器，第一个为**签到**触发器

- 触发版本选择默认流量即可
- 触发方式选择**定时触发**
- 定时任务名称为**SignIn**
- 触发周期选择**自定义触发周期**
- 在下面出现的Cron表达式中，定义每日签到的时间，表达式可参考[腾讯云文档](https://cloud.tencent.com/document/product/583/9708) ，一个每天上午10:00:13自动签到的例子为`13 0 10 * * * *`
- 附加信息为否，并保持立即启用为勾选

创建好一个签到触发器后，便可以再创建一个签退触发器，其选项除了以下内容不一样之外，其余均与上述签到触发器保持一致

- 定时任务名称为**SignOut**
- Cron表达式中，定义每日签退的时间，表达式可参考[腾讯云文档](https://cloud.tencent.com/document/product/583/9708) ，一个每天下午19:00:40自动签退的例子为`40 0 19 * * * *`

创建好两个触发器后，便可以在相应的时间自动进行签到签退操作了

*API文档将在后续更新中添加*

-----

#### 鸣谢

[CncCbz/xybSign](https://github.com/CncCbz/xybSign)

