# Java生鲜电商平台-异常模块的设计与架构

说明：任何一个软件系统都会出现各式各样的异常与错误，我们需要根据异常的情况进行捕获与分析，改善自己的代码，让其更加的稳定的，快速的运行，那么作为一个``

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

2.1.对于业务层的异常，我们采用自定义异常进行获取

核心代码如下：


```
/**
 * service异常，业务异常继续于当前接口
 * 
 */
public class ServiceException extends RuntimeException {

    private static final long serialVersionUID = 4875141928739446984L;

    /**
     * 错误码
     */
    protected String code;

    /**
     * 错误信息
     */
    protected String message;

    public ServiceException(String code, String message) {
        super();
        this.code = code;
        this.message = message;
    }

    public ServiceException(String code, String message, Throwable t) {
        super();
        this.code = code;
        this.message = message;
    }

    public ServiceException(String message) {
        super(message);
        this.message = message;
    }

    public ServiceException(String message, Throwable t) {
        super(message, t);
        this.message = message;
    }

    public String getCode() {
        return code;
    }

    public void setCode(String code) {
        this.code = code;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }

    @Override
    public String toString() {
        return "ServiceException [code=" + code + ", message=" + message + "]";
    }
```
2.2.对于全局的异常，我们采用spring的统一进行处理，只需要在代码中进行异常的抛出即可

核心代码如下：



```
/**
 * 全局异常处理类.对后台直接抛往前台页面的异常进行封装处理.
 */
public class ExceptionHandler extends SimpleMappingExceptionResolver {

    private static final Logger logger = LoggerFactory.getLogger(ExceptionHandler.class);

    @Override
    protected ModelAndView doResolveException(HttpServletRequest request, HttpServletResponse response, Object handler,
            Exception ex) {

        ModelAndView mv = super.doResolveException(request, response, handler, ex);

        String url = WebUtils.getPathWithinApplication(request);

        logger.error("controller error.url=" + url, ex);

        /* 使用response返回 */
        response.setStatus(HttpStatus.OK.value()); // 设置状态码
        response.setContentType(MediaType.APPLICATION_JSON_VALUE); // 设置ContentType
        response.setCharacterEncoding("UTF-8"); // 避免乱码
        response.setHeader("Cache-Control", "no-cache, must-revalidate");
        
        JsonResult jsonResult=new JsonResult(JsonResultCode.FAILURE,"系统错误,请联系管理员","");
        
        if(ex instanceof ServiceException)
        {
            ServiceException serviceException=(ServiceException)ex;
            String code=serviceException.getCode();
            String message=serviceException.getMessage();
            jsonResult=new JsonResult(code,message,"");
        }
        try 
        {
            PrintWriter printWriter = response.getWriter();
            printWriter.write(JSONObject.fromObject(jsonResult).toString());
            printWriter.flush();
            printWriter.close();
        } catch (IOException e) 
        {
            logger.error("与客户端通讯异常:" + e.getMessage(), e);
        }
        logger.error("异常:" + ex.getMessage(), ex);
        return mv;
    }
}
```
spring中还需要加上一段这样的配置：


```
<!-- 全局异常处理.-->
<bean id="exceptionHandler" class="com.netcai.buyer.exception.ExceptionHandler"/>   
```
3.代码格式方面

说明：我们控制所有的请求的接口，都需要返回JsonResult对象，另外，我们规定“200”字符串表示请求成功，非“200”表示失败

相关的代码贴出来，请大家分享：


```
/**
 *  Controller层的 json格式对象
 */
public class JsonResult implements java.io.Serializable {

    private static final long serialVersionUID = 1L;
    
    /**
     * 返回的编码
     */
    private String code;
    
    /**
     * 返回的信息
     */
    private String message;
    
    /***
     * 返回的对象
     */
    private Object object;

    public JsonResult() {
        super();
    }
    
    public JsonResult(String code, String message, Object object) {
        super();
        this.code = code;
        this.message = message;
        this.object = object;
    }

    public String getCode() {
        return code;
    }

    public void setCode(String code) {
        this.code = code;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }

    public Object getObject() {
        return object;
    }

    public void setObject(Object object) {
        this.object = object;
    }
}
```


