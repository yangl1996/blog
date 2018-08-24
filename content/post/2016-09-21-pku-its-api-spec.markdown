---
date: "2016-09-21T16:45:00Z"
image: /content/images/2016/09/IMG_0482.jpg
title: PKU ITS 网关的新 API
---

计算中心在上学期启用了一套新的网关 API，用来实现“断开指定连接”之类的功能。新的 API 用在了新版的网关客户端上。

不像 5428 端口的那个，这个新 API 强制 HTTPS，所以给反向稍稍带来了一点麻烦。我在 iPhone 上进行了 MitM 攻击截取通信，摸清了这套 API。下面是一个简单的文档。我没有测试“断开全部连接”功能，也还没有仔细检验文档的正确性。

可以看出，ITS 的新 API 对编程不是很友好，更倾向于直接简单解析返回值之后直接显示（就像客户端的做法一样）。

---

#### Usage

##### Request

* __Endpoint__: `POST https://its.pku.edu.cn/cas/ITSClient`
* __Content-Type__: `application/x-www-form-urlencoded`
* __Accept__: `*/*`

##### Response

* __Contect-Type__: `text/html;charset=utf-8`，但是实际上是 JSON。

---

#### Get News

获得 NOC 发布的最新通知（就是显示在客户端“通知”栏的内容）。

##### Request

* `cmd`: `getmsgs`

##### Response Example

```
{
  "succ" : "tz20160915093301;校园网紧急通知;2016.09.15;https:\/\/its.pku.edu.cn\/announce\/tz20160915093301.jsp;tz20160908161501;学校IPTV系统临时停播的通知;2016.09.08;https:\/\/its.pku.edu.cn\/announce\/tz20160908161501.jsp;tz20160908084901;校园网恢复正常访问通知;2016.09.08;https:\/\/its.pku.edu.cn\/announce\/tz20160908084901.jsp;tz20160904203001;校园网紧急通知;2016.09.04;https:\/\/its.pku.edu.cn\/announce\/tz20160904203001.jsp;tz20160829150201;旧短信平台下线通知;2016.08.29;https:\/\/its.pku.edu.cn\/announce\/tz20160829150201.jsp;tz20160828091201;2016年本校读研\/读博\/硕博连读账号迁移通知;2016.08.28;https:\/\/its.pku.edu.cn\/announce\/tz20160828091201.jsp;tz20160822155601;无线网络设备检修通知;2016.08.22;https:\/\/its.pku.edu.cn\/announce\/tz20160822155601.jsp;tz20160821180101;北京大学开通eduroam全球无线漫游服务;2016.08.21;https:\/\/its.pku.edu.cn\/announce\/tz20160821180101.jsp;tz20160819154120;校园网出口设备维护通知;2016.08.19;https:\/\/its.pku.edu.cn\/announce\/tz20160819154120.jsp;tz20160810113720;关于校园网出口设备临时故障的紧急通知;2016.08.10;https:\/\/its.pku.edu.cn\/announce\/tz20160810113720.jsp;"
}
```

---

#### Connect

连接网关。

##### Request

* `app`: `IPGWiOS1.2`
* `cmd`: `open`
* `ip`: 空
* `iprange`: `fee`（我没有测试免费地址）
* `lang`: 空
* `password`: IAAA 密码
* `username`: 学号

##### Response

注意，所有值都是 `string`。

* `IP`: 当前的 IP
* `SCOPE`: `international`（我没有测试免费地址）
* `FR_TYPE`: 空字符串
* `FR_DESC_EN`: `Unlimited`（我没有测试其他包月方案）
* `FR_TIME_EN`: `Unlimited`
* `BALANCE_CN`: 余额，一位小数，字符串类型（如 `"100.0"`）
* `CONNECTIONS`: 当前活跃连接数，整数，字符串类型（如 `"2"`）
* `DEFICIT` : 空字符串，我不清楚意义
* `ver` : `1.1`，可能是 API 版本号
* `FR_DESC_CN`: `90元不限时包月`
* `FR_TIME_CN`: `不限时`
* `FIXRATE`: `YES`
* `succ` : 空字符串
* `BALANCE_EN`: 余额，一位小数，字符串类型（如 `"100.0"`）

---

#### Disconnect

断开网关。

##### Request

* `cmd`: `close`
* `land`: 空

##### Response

* `succ`: `close_OK`

---

#### Get Connection

获取当前在线连接

##### Request

* `cmd`: `getconnections`
* `lang`: 空
* `password`: IAAA 密码
* `username`: 学号

##### Response

* `succ`: 当前连接描述，格式为：`<ip>;收费地址;<地点>;<连接时间YYYY-MM-DD HH:mm:ss>;`，若有多条记录就直接接下去。

##### Response Example

```
{
  "succ" : "1.2.3.4;收费地址;理科5;2016-09-21 20:32:00;5.6.7.8;收费地址;45甲;2016-09-16 04:03:14;9.10.11.12;收费地址;理科5;2016-09-21 20:34:09"
}
```

---

#### Disconnect Certain Session

断开指定 IP 的连接。

##### Request

* `cmd`: `disconnect`
* `ip`: 要断开连接的 IP
* `lang`: 空
* `password`: IAAA 密码
* `username`: 学号

##### Response

* `succ`: `断开连接成功`