# 币游宝平台接入文档

币游宝开放平台：https://biwan.wanlege.com/openplat/#/login  

服务端API功能对接说明：[点击查看](./币游宝对接文档（服务端）.md)

客户端SDK接入示例1（该示例由cocos creaotr编写）：https://github.com/MonkeyJacky/byb_sdk_test_sample.git

客户端SDK接入示例2（该示例由android studio原生编写）：

## 目录
* [准备工作](#准备工作)
* [授权登录](#授权登录)
  * [H5授权登录](H5授权登录)
  * [Android授权登录](Android授权登录)
* [支付](#支付)
  * [H5支付](H5支付)
  * [Android支付](Android支付)


### 准备工作

    币游宝登录是基于OAuth2.0协议标准构建的授权登录系统。
    
    在进行币游宝OAuth2.0授权登录接入之前，在币游宝开放平台注册开发者帐号，并拥有一个已审核通过的移动应用，
    并获得相应的AppID和AppSecret，可开始接入流程。

    目前移动应用上币游宝登录只提供原生的登录方式，需要用户安装币游宝客户端才能配合使用。

### 授权登录

币游宝 OAuth2.0 授权登录目前适用于拥有 server 端的应用授权。该模式整体流程为：

    1. 第三方发起币游宝授权登录请求，用户允许授权第三方应用后，币游宝会拉起应用，并且带上授权临时票据Code参数；

    2. 通过Code参数加上AppID和AppSecret等，通过API换取Access_token；

    3. 通过Access_token进行接口调用，获取用户基本数据资源或帮助用户实现基本操作；
    
    4. 获取到的用户基本数据中OpenID可作为用户唯一识别标识。

客户端SDK具体使用说明请下载文档及相应示例查看。

#### H5授权登录

H5授权登录时序图参考如下：  
![图片加载失败，请刷新页面](https://biwan.wanlege.com/source/app/h5-auth.jpg "币游宝H5授权登录时序图")


H5游戏授权动作已由币游宝平台App完成，开发者只需要在代码逻辑中调用sdk初始化接口后即可使用其他功能接口。  
初始化接口：BYBApi.init(AppID)；  
获取Code：BYBApi.getCode()；  
获取AccessToken：BYBApi.getAccessToken()；  
代码示例如下：  
```javascript
_BYBApiInit() {
    var _this = this;
    BYBApi.init(9); //9为应用对应的AppID

    var callback = function (data) {
      if (data != null) {
        if (data.code == 0) {
          //获取参数OpenID nick IsActived
          //OpenID 用户唯一识别码
          //nick 用户昵称
          //IsActived 用户是否已激活，未激活无法发起支付
          _this.hud.setPlayerInfo(data.data.nick, data.data.OpenID);
        }
      }
    };
    //具体的Api接口说明参见Api文档
    HttpMng.userInfoReq(BYBApi.getCode(), BYBApi.getAccessToken(), callback);
},
```

#### Android授权登录

APP授权登录时序图参考如下：  
![图片加载失败，请刷新页面](https://biwan.wanlege.com/source/app/app-auth.jpg "币游宝APP授权登录时序图")

首先，需要完成SDK的初始化，才能进一步调用其他接口，BYBHandler用来接收从币游宝平台授权、支付返回后的参数；  
```java
static BYBHandler sdkListener = new BYBHandler() {
    @Override
    public void handleBack(final String data) {
        JSONObject obj = formdata2obj(data);
        try {
            if (obj.getString("ReqType").equals("auth")) {
                String code = obj.getString("code");
                if (code.equals("0")) {
                    Code = obj.getString("Code");
                    dialogTip(_context, "授权成功");
                } else {
                    dialogTip(_context, "授权失败");
                }
            }
            else if (obj.getString("ReqType").equals("pay")) {
                if (obj.getString("code").equals("0")) {
                    dialogTip(_context, "支付成功");
                } else {
                    dialogTip(_context, "支付失败");
                }
            }
        } catch (JSONException e) {
            e.printStackTrace();
        }
    }
};

@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    BYBSDK.init(this, "2", "bybtestapp", sdkListener);
}
```

第三方需要在自己的游戏中添加币游宝登录按钮，通过调用授权接口拉起币游宝应用完成进一步授权；  
```java
BYBSDK.authorise(); //授权接口，调用后会拉起币游宝App，并显示授权界面；
```

### 支付

币游宝支付整体流程为：

    1. 玩家在第三方应用上选择商品下单；
    
    2. 第三方客户端将授权时获得的Access_token传递给第三方服务器，由服务器统一调用下单Api；
    
    3. 第三方服务器将下单Api返回的预支付数据签名后返回给第三方客户端；
    
    4. 第三方客户端将返回的预支付数据传入币游宝SDK支付接口，拉起币游宝App完成进一步支付；
    
    5. 支付完成后，币游宝App会返回支付结果；


客户端SDK具体使用说明请下载文档及相应示例查看。

H5和APP的支付流程相同，支付时序图参考如下：  
![图片加载失败，请刷新页面](https://biwan.wanlege.com/source/app/pay.jpg "币游宝支付时序图")

#### H5支付

```javascript
BYBApi.pay(prepayid, nonce, timestamp, sign); //H5支付接口，调用后会拉起币游宝App，并显示支付界面
```

#### Android支付

```java
//Android支付接口，调用后会拉起币游宝App，并显示支付界面
public static void pay(final String prepayid, final String nonce, final String timestamp, final String sign);
```

