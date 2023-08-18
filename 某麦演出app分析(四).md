# 某麦演出app分析(四)---API数据生成逻辑分析

## 前情提要

在app界面层面的分析在上一篇[某麦演出app分析(三)---无视重试返回上一页](https://lqcoding.com/archives/x-mai-app-reverse-3.html)之后,可以做的已经不多了.

本身已经打算告一段落,但今天五月天广州演唱会突然重磅消息:**修改主题为诺亚方舟!**

这就意味着抢到票的几率又再再再再创新低.

想着既然整个购票过程都分析得七七八八了,不如就看能不能整个自动化!

**Lets go!!!**

## API基本要素

抢票过程自动化,本质就是自动调用API接口.而`http`接口调用无非就需要三件事: 

1. `API URL`
2. `cookie`
3. `parameter(请求参数)`

而一般的api设计都是有类似签名的参数作为保护的,就更别说阿里系这个级别的app了,所以整个思路是在root手机上运行大麦app,再通过`frida` HOOK 应用的函数,去生成签名和API需要的参数,再通过python去自动调用API.

首先搞清楚选票到生成支付订单一共有多少个API,再逐个API一项一项来~

## 购票业务过程

通过抓包可以知道,除了最开始的获取演出信息API之外,主要需要经历两个API:

1. `https://mtop.damai.cn/gw/mtop.trade.order.build/4.0/`
2. `https://mtop.damai.cn/gw/mtop.trade.order.create/4.0/`

以第一个API为例:

![image-20230808165105725](某麦演出app分析(四).assets/image-20230808165105725.png)

![image-20230808165130954](某麦演出app分析(四).assets/image-20230808165130954.png)

主要需要获取

1. header里面`x-`开口的value
2. `request body`中`wua`的value
3. `request body`中`data`的value 

## 源码分析

搜索字符`mtop.trade.order.build`找到类`UltronBuildOrder`

![image-20230808180238449](某麦演出app分析(四).assets/image-20230808180238449.png)

从`UltronBuildOrder`->`DMQueryKey`->`UltronDataManager`,最后定位到工厂函数`UltronDataManager.createBuildRequest`

![image-20230808180744346](某麦演出app分析(四).assets/image-20230808180744346.png)

在`DMOrderBuildRequest`中主要是设置api回调,其中call api过程可以继续深入分析`execute`:

![image-20230808181737949](某麦演出app分析(四).assets/image-20230808181737949.png)

`execute`是java接口`IDMRequester`的函数,由于`dMRequester`是`DMRequester`的实现

![image-20230808182118224](某麦演出app分析(四).assets/image-20230808182118224.png)

在`DMRequester`中`execute`最终会调用函数`dmRequestExecute`

![image-20230808182346611](某麦演出app分析(四).assets/image-20230808182346611-1691490227400-1.png)

`dmRequestExecute`的主要逻辑:

```
private boolean dmRequestExecute(Object obj, MMorderBuildRequestCallback mMorderBuildRequestCallback) {
        IDMContext iDMContext = this.idmContext;
        if (iDMContext instanceof MMsomeDataModelUltron) {
           ...
           ...//设置this.mmtopRequest的数据,这个会用于后面call api参数的生成
           ...
            MtopBusiness build = MtopBusiness.build(this.mmtopRequest);//MtopBusiness是call api的实例
            ...
            ...//设置build实例的参数
            ...
            //startRequest开始进行请求
            if (this.f28883n == null) {
                build.addListener((MtopListener) response).startRequest();
            } else {
                build.addListener((MtopListener) response).startRequest(this.f28883n);
            }
			...
			...
            return true;
        }
        return false;
    }
```

`startRequest`主要是调用`super.asyncRequest()`获取到一个`ApiID`实例,这个应该就是一个抽象的Api请求追踪实例:

![image-20230809121003662](某麦演出app分析(四).assets/image-20230809121003662.png)

而这个`ApiID`实例是在底层的`MtopBuilder.asyncRequest`生成:

![image-20230809141201852](某麦演出app分析(四).assets/image-20230809141201852.png)

其中要注意这个`filterManager.start(null, createMtopContext);`,其中的`FilterManager`就是调用接口底层逻辑的过滤器,它负责完成接调用前的数据处理逻辑和调用后的回调逻辑,可以看看`FilterManager`->`AbstractFilterManager`->`InnerFilterManagerImpl`:

![image-20230809141437372](某麦演出app分析(四).assets/image-20230809141437372.png)

`addBefore`是调用前的处理逻辑,只有`return "CONTINUE"`才会继续下一个过滤器,`return "STOP"`就会直接去到`addAfter`的过滤器中.

## `ProtocolParamBuilderBeforeFilter`

用生成大部分请求参数和生成签名的过滤器`ProtocolParamBuilderBeforeFilter`来做例子,在例子中,通过`ProtocolParamBuilder.buildParams`生成参数`map`之后,如果没问题就set到`mtopContext.protocolParams = map;`中

![image-20230811172951011](某麦演出app分析(四).assets/image-20230811172951011.png)

在`InnerProtocolParamBuilderImpl`中的`buildParams`,找到了很多熟悉的`key`:

![image-20230814173819540](某麦演出app分析(四).assets/image-20230814173819540.png)

这里就是生成签名的地方.

`HashMap<String, String> unifiedSign = iSign.getUnifiedSign(hashMap3, hashMap4, str5, str6, z2);`

## Hook

hook一下`getUnifiedSign`函数看看输入和输出值:

```
let InnerSignImpl = Java.use("mtopsdk.security.InnerSignImpl");
InnerSignImpl["getUnifiedSign"].implementation = function (
  hashMap,
  hashMap2,
  str,
  str2,
  z
) {
  console.log(
    `InnerSignImpl.getUnifiedSign is called: hashMap=${hashMap}, hashMap2=${hashMap2}, str=${str}, str2=${str2}, z=${z}`
  );
  let result = this["getUnifiedSign"](hashMap, hashMap2, str, str2, z);
  console.log(`InnerSignImpl.getUnifiedSign result=${result}`);
  return result;
};
```

根据结果,就而已在`python`调用`rpc`调用获取需要的`header value`了:

![image-20230814180227623](某麦演出app分析(四).assets/image-20230814180227623.png)

首先在js脚本写下一个`rpc`:

```
rpc.exports.getBuildOrderSign = getBuildOrderSign;
```

然后根据`getUnifiedSign`需要的参数,定义一下需要的数据实例:

```
function getBuildOrderSign() {
  let exParams = {
    atomSplit: "1",
    channel: "damai_app",
    coVersion: "2.0",
    coupon: "true",
    seatInfo: "",
    umpChannel: "10001",
    websiteLanguage: "zh_CN",
  };
  let requestData = {
    buyNow: "true",
    buyParam: "727188803162_1_5045629021941",
    exParams: JSON.stringify(exParams),
  };
  return getISign("mtop.trade.order.build","4.0", requestData);
}
```

最后主动调用`getUnifiedSign`:

```
function getISign(api,apiVersion, requestData, z = true) {
  let MtopConfig = Java.use("mtopsdk.mtop.global.MtopConfig");
  let mtopconfig = MtopConfig.$new("INNER");
  let UploadStatAppMonitorImpl = Java.use(
    "mtopsdk.mtop.stat.UploadStatAppMonitorImpl"
  );
  let NetworkPropertyServiceImpl = Java.use(
    "mtopsdk.mtop.network.NetworkPropertyServiceImpl"
  );
  mtopconfig.uploadStats = UploadStatAppMonitorImpl.$new();
  mtopconfig.networkPropertyService = NetworkPropertyServiceImpl.$new();

  let InnerSignImpl = Java.use("mtopsdk.security.InnerSignImpl");
  let isign = InnerSignImpl.$new();
  isign.init(mtopconfig);
  // console.log("isign:", isign);
  let reqAppKey = "xxxxxxxxxx";//reqAppKey有可能不一样
  let authCode = null;
  let HashMap = Java.use("java.util.HashMap");
  let hm1 = HashMap.$new(64);
  let hm2 = HashMap.$new();
  let map1 = buildHashMap1(api,apiVersion, requestData);
  console.log("show map1:");
  console.log(map1.data);
  for (let key in map1) {
    hm1.put(key, map1[key]);
  }
  let map2 = buildHashMap2();
  for (let key in map2) {
    hm2.put(key, map2[key] + "");
  }
  let result = isign.getUnifiedSign(hm1, hm2, reqAppKey, authCode, z);
  result = showJavaObjectString(result);
  return { ...result, ...map1 };
}
```

## 获取cookie

获取`cookie`的方式就想对简单很多了,可以直接在`web`端登陆获取,也能通过`rpc`主动调用函数获取:

```
function getCookies(str) {
  let CookieManager = Java.use("anetwork.channel.cookie.CookieManager");
  let cookies = CookieManager.i(str);
  // console.log("cookie:", cookies);
  let list = cookies.split(";");
  let result = {};
  list.forEach((item) => {
    let keyvalue = item.trim().split("=");
    if (keyvalue[0].indexOf("sgcookie") == -1)
      result[keyvalue[0]] = keyvalue[1];
  });
  console.log("result:", JSON.stringify(result));
  return result;
}
```

这个类的寻找过程太曲折了,有时间再补

## python 模拟调用

到这里可以说说是万事俱备了!开始写测试`python`代码了

