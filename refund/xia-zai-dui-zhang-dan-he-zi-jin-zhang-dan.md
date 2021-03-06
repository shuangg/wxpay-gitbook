---
description: 本文是【浅析微信支付】系列文章的第九篇，主要讲解商户下载对账单接口和资金账单接口的实现和一些注意事项。
---

# 下载对账单和资金账单

浅析微信支付系列已经更新九篇了哟～，没有看过的朋友们可以看一下哦。

[浅析微信支付：申请退款、退款回调接口、查询退款](https://mp.weixin.qq.com/s/IyWjWB__-VsqKO8SL0DL3Q)

[浅析微信支付：查询订单和关闭订单](https://mp.weixin.qq.com/s/SG4sTHsUKKJF-_Qgpjh0jA)

[浅析微信支付：支付结果通知](https://mp.weixin.qq.com/s/Zr3ldgsMIg_cBrtuS2c7_g)

在商户平台中，商家也可以下载资金对账单，历史的交易清单，具体位置：商户平台 -&gt; 交易中心 -&gt; 账单管理。

如果要查看实时的流水记录，可以在微信APP中搜索小程序 `微信支付商户助手` 即可查看。

#### 1、下载对账单

以下为微信官方的`下载对账单`文档：

```text
https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=9_6
```

根据接口下载历史的交易账单，数据以文本表格的方式返回，第一行为表头，后面各行为对应的字段内容，字段内容跟查询订单或退款结果一致，具体字段说明可查阅相应接口。

此接口方便商家在自身系统中下载，不依赖于微信商户平台。

**1.1. 应用场景**

商户可以通过该接口下载历史交易清单。比如掉单、系统错误等导致商户侧和微信侧数据不一致，通过对账单核对后可校正支付状态。

```text
注意：
1、微信侧未成功下单的交易不会出现在对账单中。支付成功后撤销的交易会出现在对账单中，跟原支付单订单号一致；
2、微信在次日9点启动生成前一天的对账单，建议商户10点后再获取；
3、对账单中涉及金额的字段单位为“元”。
4、对账单接口只能下载三个月以内的账单。
5、对账单是以商户号纬度来生成的，如一个商户号与多个appid有绑定关系，则使用其中任何一个appid都可以请求下载对账单。对账单中的appid取自交易时候提交的appid，与请求下载对账单时使用的appid无关。
```

**1.2. 接口链接**

```text
https://api.mch.weixin.qq.com/pay/downloadbill
```

**1.3. 是否需要证书**

不需要

**1.4. 调用接口**

调用参数：

| 字段名称 | 变量名 | 必填 | 类型 | 描述 |
| :--- | :--- | :--- | :--- | :--- |
| 账单日期 | bill\_date | 是 | String\(8\) | 下载对账单的日期，格式：20140603 |
| 账单类型 | bill\_type | 是 | String\(8\) | ALL，返回当日所有订单信息，默认值SUCCESS，返回当日成功支付的订单REFUND，返回当日退款订单RECHARGE\_REFUND，返回当日充值退款订单 |
| 压缩账单 | tar\_type | 否 | String\(8\) | 非必传参数，固定值：GZIP，返回格式为.gzip的压缩包账单。不传则默认为数据流形式。 |

以下为调用示例代码：

```text
/**
 * 对账单下载
 */
private void doDownloadBill() {
    HashMap<String, String> data = new HashMap<String, String>();
    data.put("bill_date", "20161102");
    data.put("bill_type", "ALL");
    try {
        Map<String, String> r = wxpay.downloadBill(data);
        System.out.println(r);
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

**1.5. 返回结果**

成功时，数据以文本表格的方式返回，第一行为表头，后面各行为对应的字段内容，字段内容跟查询订单或退款结果一致，具体字段说明可查阅相应接口。

第一行为表头，根据请求下载的对账单类型不同而不同\(由bill\_type决定\),目前有：

当日所有订单 交易时间,公众账号ID,商户号,子商户号,设备号,微信订单号,商户订单号,用户标识,交易类型,交易状态,付款银行,货币种类,总金额,代金券或立减优惠金额,微信退款单号,商户退款单号,退款金额,代金券或立减优惠退款金额，退款类型，退款状态,商品名称,商户数据包,手续费,费率

当日成功支付的订单 交易时间,公众账号ID,商户号,子商户号,设备号,微信订单号,商户订单号,用户标识,交易类型,交易状态,付款银行,货币种类,总金额,代金券或立减优惠金额,商品名称,商户数据包,手续费,费率

当日退款的订单 交易时间,公众账号ID,商户号,子商户号,设备号,微信订单号,商户订单号,用户标识,交易类型,交易状态,付款银行,货币种类,总金额,代金券或立减优惠金额,退款申请时间,退款成功时间,微信退款单号,商户退款单号,退款金额,代金券或立减优惠退款金额,退款类型,退款状态,商品名称,商户数据包,手续费,费率

从第二行起，为数据记录，各参数以逗号分隔，参数前增加\`符号，为标准键盘1左边键的字符，字段顺序与表头一致。

倒数第二行为订单统计标题，最后一行为统计数据

总交易单数，总交易额，总退款金额，总代金券或立减优惠退款金额，手续费总金额

举例如下：

```text
交易时间,公众账号ID,商户号,子商户号,设备号,微信订单号,商户订单号,用户标识,交易类型,交易状态,付款银行,货币种类,总金额,代金券或立减优惠金额,微信退款单号,商户退款单号,退款金额,代金券或立减优惠退款金额,退款类型,退款状态,商品名称,商户数据包,手续费,费率
`2014-11-1016：33：45,`wx2421b1c4370ec43b,`10000100,`0,`1000,`1001690740201411100005734289,`1415640626,`085e9858e3ba5186aafcbaed1,`MICROPAY,`SUCCESS,`CFT,`CNY,`0.01,`0.0,`0,`0,`0,`0,`,`,`被扫支付测试,`订单额外描述,`0,`0.60%
`2014-11-1016：46：14,`wx2421b1c4370ec43b,`10000100,`0,`1000,`1002780740201411100005729794,`1415635270,`085e9858e90ca40c0b5aee463,`MICROPAY,`SUCCESS,`CFT,`CNY,`0.01,`0.0,`0,`0,`0,`0,`,`,`被扫支付测试,`订单额外描述,`0,`0.60%
总交易单数,总交易额,总退款金额,总代金券或立减优惠退款金额,手续费总金额
`2,`0.02,`0.0,`0.0,`0
```

#### 2、下载资金账单接口

以下为微信官方的`下载资金账单`文档：

```text
https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=9_18&index=7
```

**2.1. 应用场景**

商户可以通过该接口下载自2017年6月1日起 的历史资金流水账单。

说明：

```text
1、资金账单中的数据反映的是商户微信账户资金变动情况；
2、当日账单在次日上午9点开始生成，建议商户在上午10点以后获取；
3、资金账单中涉及金额的字段单位为“元”。
```

**2.2. 接口链接**

```text
https://api.mch.weixin.qq.com/pay/downloadfundflow
```

**2.3. 是否需要证书**

请求需要双向证书

**2.4. 调用接口**

调用参数：

| 字段名称 | 变量名 | 必填 | 类型 | 描述 |
| :--- | :--- | :--- | :--- | :--- |
| 签名类型 | sign\_type | 否 | String\(32\) | 签名类型，目前仅支持HMAC-SHA256 |
| 资金账单日期 | bill\_date | 是 | String\(8\) | 下载对账单的日期，格式：20140603 |
| 资金账户类型 | account\_type | 是 | String\(8\) | 账单的资金来源账户：Basic  基本账户、Operation 运营账户、Fees 手续费账户 |
| 压缩账单 | tar\_type | 否 | String\(8\) | 非必传参数，固定值：GZIP，返回格式为.gzip的压缩包账单。不传则默认为数据流形式。 |

此接口不常用，推荐使用微信商户平台下载。具体的实现请参考上面的官方文档。

#### 结语

以上为`下载对账单、资金账单`相关的解释和源码，特别需要注意的是`下载资金账单`接口需要特定的签名类型`HMAC-SHA256`，小伙伴们一定要注意哦，具体的源码可以看作者的github，里面对每个方法有详细的注释。

预告：下一篇文章 `如何使用沙箱环境测试`，敬请期待！！！

​如果想要提前一览源码的小伙伴，可以先看看我的 github，地址如下： ​ ​`​https://github.com/YClimb/wxpay-sdk/blob/master/README.md ​`

关注作者微信公众号，点击下方`讨论群`，扫码即可加入`微信支付讨论群`与小伙伴一起探讨哦～

到此本文就结束了，关注公众号查看更多推送！！！

![&#x5173;&#x6CE8;&#x6211;&#x7684;&#x516C;&#x4F17;&#x53F7;](../.gitbook/assets/er-wei-ma.jpg)



