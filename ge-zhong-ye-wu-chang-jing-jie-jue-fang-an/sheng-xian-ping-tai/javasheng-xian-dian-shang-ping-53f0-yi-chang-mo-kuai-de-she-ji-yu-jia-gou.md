# Java生鲜电商平台-异常模块的设计与架构

说明：任何一个软件系统都会出现各式各样的异常与错误，我们需要根据异常的情况进行捕获与分析，改善自己的代码，让其更加的稳定的，快速的运行，那么作为一个

B2B的Java开源生鲜电商平台，我们的异常需要思考以下几个维度。

1. 运行的代码异常

说明：
1.代码在运行的过程中，难免出现各种异常与错误，我们采用Log4j进行日志的记录。

2.在分层代码解耦过程中，我们统一在Controller进行异常的捕获与日志记录。

相关的运行的代码异常架构如下：


```
/**
     * (商家店铺)商品信息列表
     * @param request
     * @param response
     * @return
     */
    @RequestMapping(value = "/goods/list", method = { RequestMethod.GET})
    public JsonResult goodsList(HttpServletRequest request, HttpServletResponse response) {
        try
        {
            //业务逻辑处理
            return new JsonResult(JsonResultCode.FAILURE, "系统错误,请稍后重试","");
        }catch(Exception ex){
            logger.error("[GoodsController][goodsList] exception :",ex);
            return new JsonResult(JsonResultCode.FAILURE, "系统错误,请稍后重试","");
        }
    }
```

说明：由于spring的事物控制在service层，所以service层不抓取异常。所有的异常都在controller进行抓取，而且抓取的是任何的异常

异常的记录采用某个类[]某个方法[] exception,这种方式，然后把所有的异常的堆栈信息进行打印出来，方便进行查看与分析，定位等

2.全局异常

说明：对于有可能出现的全局异常，我们采用Spring的全局异常处理器，进行代码的统一处理。 

2.1，对于业务层的异常，我们采用自定义异常进行获取

核心代码如下：