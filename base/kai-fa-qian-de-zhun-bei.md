---
description: 本文是【浅析微信支付】系列文章的第三篇，主要会讲一下在开发前的一些注意事项。
---

# 开发前的准备

浅析微信支付系列已经更新两篇了哟～，没有看过的朋友们可以看一下。

[浅析微信支付：前篇大纲](https://mp.weixin.qq.com/s/pH0GGwUANJxJt-YHs6y_pQ)

[浅析微信支付：微信支付简单介绍（小程序、公众号、App、H5）](https://mp.weixin.qq.com/s/h9Mx4IBik0MLKSj4c_Qolg)

#### 1、开发必备

因为作者是Java开发，所以这个系列仅针对Java开发人员，其他语言的同学也可以简单看看，了解一下，大概的步骤都是一样的；

接下来我们来了解一下开发所需要的准备：

```text
1. 注册一个服务号，并认证通过
2. 开通微信支付功能
3. 如果不是管理员，则需要添加服务号和商户平台的用户管理权限
4. 配置公众号 `JS接口安全域名` 和 `网页授权域名`
5. 在商户平台开通相应的支付产品功能
6. 在商户平台设置支付的相关ip授权
```

**1.1. 注册一个服务号，并认证通过**

微信公众平台地址：`https://mp.weixin.qq.com` ，开发者可以使用公司邮箱，根据微信的官方引导注册 `服务号`，一定要是公司的邮箱，以后用经常用到的；

注册完成之后，进行微信认证，路径：点击左上方头像 -&gt; 选择认证详情 -&gt; 在出来的界面按要求申请认证即可。

**1.2. 开通微信支付功能**

认证通过以后，可在微信公众平台申请开通微信支付，路径：平台首页 -&gt; 点击左侧微信支付 -&gt; 点击支付申请 -&gt; 根据官方引导一步步申请即可。

不会的小伙伴可以百度一下，很多的例子可以参考，这里就不重复造轮子了。

开通微信支付需要注册登陆 `微信商户平台`，微信支付相关的信息都需要在这个平台上进行操作。

注册登陆商户平台，进入账户中心 -&gt; 支付申请 -&gt; 按要求填写即可；

PS：一定要注意服务号和商户平台必须是一个账户主体，也就是认证的公司需要一致，否则不是同一个商户。

**1.3. 如果不是管理员，则需要添加服务号和商户平台的用户管理权限**

通常开通微信公众平台和商户平台的人都是管理员，也就是你的老大等人员，我们开发者需要登陆使用功能时也不会使用管理员，所以需要添加自己微信号的权限；

微信公众平台的权限叫做 `运营者微信号`，在公众平台的左侧 -&gt; 人员设置中添加，需要管理员为我们绑定一个长期的运营者账号；

商户平台地址：`https://pay.weixin.qq.com`

微信商户平台的权限叫做 `员工账号`，在商户平台 -&gt; 账户中心 -&gt; 左侧员工账号管理 -&gt; 选择某个角色（通常是管理员）-&gt; 新增账号 -&gt; 按要求填写之后即可；

如果添加后开发者还是没有需要的相关权限，可以在角色右上方 `配置权限` 中授权修改。

**1.4. 配置公众号 JS接口安全域名 和 网页授权域名**

首先，微信强制规定，如果要使用公众号支付、H5支付、小程序支付等产品时，必须获取到用户的openid，也就是用户唯一标识，如果获取呢？公众号支付需要网页授权，而网页授权就必须配置 `JS接口安全域名` 和 `网页授权域名`这两个域名，小程序支付也一致；

不同点是，公众号支付的域名可以是http/https，而小程序则必须是https；

配置路径：公众平台 -&gt; 左侧公众号设置 -&gt; 功能设置 -&gt; JS接口安全域名/网页授权域名

需要下载微信的安全配置文件，放到咋们的服务器上，根据 授权的域名+认证文件 可以访问后即可配置完成；需要注意的是，每次修改认证域名都会再次重新认证域名，所以认证以后文件请不要轻易删除。

PS：设置IP白名单，在IP白名单内的IP来源，获取access\_token接口才可调用成功。路径：公众平台首页 -&gt; 基本配置

**1.5. 在商户平台开通相应的支付产品功能**

登陆微信商户平台，进入产品中心，可以开通需要的支付产品，如公众号支付、扫码支付、刷卡支付、H5支付；

需要注意的是，在商户平台上小程序也属于公众号支付，不需要单独开通。

PS：如果公司需要做提现等功能，需要直接向用户付款，那么需要开通 `企业付款到零钱` 产品功能，此功能主要用来解决合理的商户对用户付款需求，最终金额会直接到用户微信零钱中；

如果公司需要向用户银行卡付款，则需要开通 `企业付款到个人银行卡` 产品功能，该功能提供由商户直接付钱至指定银行卡账户的能力，主要用来解决合理的商户对用户付款需求。

如果公司提现是用公众号为用户发放红包，那么需要开通 `现金红包` 产品功能，企业向指定用户发放现金红包，红包会显示在服务号中，需要用户领取，用户在客户端领取到红包之后，所得金额进入微信钱包，可用于转账、支付或提取到银行卡。

现金红包PS：开通条件：入账方式为即时入账至商户号，结算周期为T+1的商户需满足以下两个条件：1.入驻满90天，2.连续正常交易30天。其余结算周期的商户无限制

**1.6. 在商户平台设置支付的相关ip授权**

这里以公众号支付为例，开通公众号支付后，这是还不能进行开发，我们需要拿到商户的几个重要信息：

```text
1. APP_ID(公众平台获取)：公众号/小程序开发者ID(AppID) -> 公众平台首页 -> 基本配置 
2. APP_SECRET(公众平台获取)：开发者密码(AppSecret) -> 公众平台首页 -> 基本配置 
3. MCH_ID(商户平台获取)：商户号 -> 商户平台首页 -> 账户中心 -> 账户信息
4. API_KEY(商户平台获取)：API密钥 -> 商户平台首页 -> 账户中心 -> API安全
5. APICLIENT_CERT(商户平台获取)：安全证书路径 -> 商户平台首页 -> 账户中心 -> API安全
```

PS：如果需要将小程序和公众号联通，需要在 公众平台首页 -&gt; 基本配置 中绑定同一个微信开放平台帐号

#### 2、开发工具

如果以上都完成以后，我们就可以进入开发了；开发前，我们还需要 `绑定开发者账号` 和下载微信官方的 `web开发者工具`，路径：公众平台首页 -&gt; 左侧开发者工具 -&gt; web开发者工具 -&gt; 绑定开发者微信号；开发者工具下载链接：`https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1455784140`，在此链接最下面，可以下载客户端。

#### 结语

开发者工具支持公众号网页、小程序等开发，如果前面的准备工作都完成了，那么我们随后就能进入微信开发的实际代码层面操作，下一篇为大家讲解 `微信公众号网页授权`，因为要使用公众号、小程序支付，必须先获取用户授权，拿到用户openid和unionid，然后再进行支付等操作。

PS：unionid是微信跨平台时保证用户唯一标识，openid是用户单平台的唯一标识；简单理解：如果咋们商户有一个服务号，一个小程序，那么同一个用户在服务号和小程序中的openid都是不一样的，怎么区分这是同一个用户？就需要使用unionid，他能保证在同一个商户平台下，同一个用户只有一个unionid。

​如果想要提前一览源码的小伙伴，可以先看看我的 github，地址如下： `https://github.com/YClimb/wxpay-sdk/blob/master/README.md`

加作者私人微信，作者微信号如下 `yclimb`，标明 `微信支付` 可拉入微信支付讨论群与小伙伴一起探讨哦，一定要标明 `微信支付` 哦～

到此本文就结束了，关注公众号查看更多推送！！！

![&#x5173;&#x6CE8;&#x6211;&#x7684;&#x5FAE;&#x4FE1;&#x516C;&#x4F17;&#x53F7;](../.gitbook/assets/er-wei-ma.jpg)

