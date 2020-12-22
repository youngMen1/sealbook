# Java生鲜电商平台-搜索模块的设计与架构

说明：搜索模块针对的是买家用户，在找菜品找的很费劲下的一种查询方面。目前也是快速的检索商品。

对于移动端的APP买家用户而言，要求的速度在3秒内完成。支持模糊查询，由于业务实战表面，整个搜索频率不到18%-25%之间

同时业务也不算很大，所以并没采用java全文检索技术.(lucene等)。这里采用的就是基本的模糊查询。

1.搜索维度的是思考。

1.1.买家搜索的内容很有可能是针对菜品的本身属性而言，所以涉及到的内容有商品名称，商品别名，商品标签，商品描述，规格的名称，加工方式等。

1.2.我们知道模糊搜索会导致索引失效，同时整个查询性能也是有影响的。

1.3.业务形态显示有些热点的词语与内容可以做JVM缓存以提高整个单品的购买率。比如土豆现在分析出很多人要，如果我们可以跟某个商家谈好，一天

需要10w斤土豆的量进行供应，那么整个页面会出现这个关键字的默认显示。这个是后端灵活配置的。

2.对于买家搜索的关键字，我们需要数据库进行记录，这样可以从系统级别算出买家会需要什么，可以进行针对性的营销.

相关数据库表的设计如下：



```
CREATE TABLE `buyer_search` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '自动增加ID',
  `buyer_id` bigint(20) DEFAULT NULL COMMENT '买家ID',
  `words` bigint(20) DEFAULT NULL COMMENT '卖家ID',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='买家搜索记录表';
```
补充说明：目的其实是很明确的，就是记录买家搜索的关键字以便分析与研究用。为了更好的体验用户。
相关业务代码如下：


```
/**
     * APP全文搜索 新
     * @param request
     * @param response
     * @param keyword
     * @return
     */
    @RequestMapping(value = "/search/new", method = { RequestMethod.GET, RequestMethod.POST })
    public JsonResult newSearchGoods(HttpServletRequest request, HttpServletResponse response,Long userId,Long regionId,String keyword){
        try{
            logger.info("SearchController.search.keyword:搜索内容：" + keyword);
            //搜索结果按商家显示
            //List<SellerVo> list = sellerService.searchSeller(regionId,keyword);
            //搜索结果按商品显示
            List<NewSearchVo> list = goodsService.newSearchGoods(userId,regionId,keyword);
            return new JsonResult(JsonResultCode.SUCCESS, "查询信息成功", list);
        }catch(Exception ex){
            logger.error("[SearchController][newSearchGoods] exception :",ex);
            return new JsonResult(JsonResultCode.FAILURE, "系统错误,请稍后重试","");
        }
    }
```
VO对象如下：


```
/**
 * 搜索显示类(APP全局搜索)
 */
public class NewSearchVo implements Serializable{

    private static final long serialVersionUID = 1L;
        
    /**
     * 来源于users的ID
     */
    private Long sellerId;

    /**
     * 店铺名称
     */
    private String sellerName;
    
    /**
     * 店铺别名,可以理解为简称
     */
    private String sellerAlias;
    
    /**
     * 店铺logo
     */
    private String sellerLogo;
    
    /**
     * 店铺评级,默认为0
     */
    private int sellerRank;
    
    /**
     * 店铺评分
     */
    private Double sellerGrade;
    
    /**
     * 搜索后商品列表
     */
    private List<GoodsVo> searchItemList;

    public Long getSellerId() {
        return sellerId;
    }

    public void setSellerId(Long sellerId) {
        this.sellerId = sellerId;
    }

    public String getSellerName() {
        return sellerName;
    }

    public void setSellerName(String sellerName) {
        this.sellerName = sellerName;
    }

    public String getSellerAlias() {
        return sellerAlias;
    }

    public void setSellerAlias(String sellerAlias) {
        this.sellerAlias = sellerAlias;
    }

    public String getSellerLogo() {
        return sellerLogo;
    }

    public void setSellerLogo(String sellerLogo) {
        this.sellerLogo = sellerLogo;
    }

    public int getSellerRank() {
        return sellerRank;
    }

    public void setSellerRank(int sellerRank) {
        this.sellerRank = sellerRank;
    }

    public Double getSellerGrade() {
        return sellerGrade;
    }

    public void setSellerGrade(Double sellerGrade) {
        this.sellerGrade = sellerGrade;
    }

    public List<GoodsVo> getSearchItemList() {
        return searchItemList;
    }

    public void setSearchItemList(List<GoodsVo> searchItemList) {
        this.searchItemList = searchItemList;
    }
}
```
3.数据查询性能暂时的基本满足要求，也贴出来给大家一起参考，目的是共同学习与思考.


```
<!-- 全局搜索商品  新-->
    <select id="newSearchGoods" resultMap="newSearchResult">
        <include refid="newSearchSelect" />
        <include refid="newSearchFrom" />
        <include refid="searchWhere" />
        <if test=" keyword != null">
            and (g.goods_name like concat('%',#{keyword},'%')
            or g.goods_as like concat('%',#{keyword},'%')
            or g.goods_label like concat('%',#{keyword},'%')
            or g.goods_brand like concat('%',#{keyword},'%')
            or g.goods_desc like concat('%',#{keyword},'%') 
            or gf.format_name like concat('%',#{keyword},'%') 
            or pm.method_name like concat('%',#{keyword},'%') 
            or s.seller_name like concat('%',#{keyword},'%') 
            or exists(select 1 from category where category_id = g.category_id and category_name like concat('%',#{keyword},'%') )
            ) 
        </if>
        <include refid="searchOrderBy" />
    </select>
```
总结，由于搜索这块所涉及的业务相对而言比较少，功能也比较单一，含金量不是很高，所以互相学习。
对于扩展方案，如果这块的业务发现很大，可以采用中文分词记录，进行数据的挖掘，已经冷热点数据的一个分离等等，这个后期大家有需要的话，我们再研究。
相关业务运营截图如下：

![](/static/image/641237-20180520102324430-708423450.png)
![](/static/image/641237-20180520102334722-557084576.png)
![](/static/image/641237-20180520102344000-1864787448.png)