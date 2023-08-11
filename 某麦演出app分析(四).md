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
