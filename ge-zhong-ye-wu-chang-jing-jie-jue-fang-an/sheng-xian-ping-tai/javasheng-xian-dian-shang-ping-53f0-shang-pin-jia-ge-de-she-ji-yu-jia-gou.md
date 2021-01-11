# Java生鲜电商平台-商品价格的设计与架构

说明：Java开源生鲜电商平台-商品价格的设计与架构,主要是对商品的价格进行研究与系统架构.

一、常见的电商价格

* 市场价（List Price）：这个价格仅是用于显示，用于衬托网站销售价格的优惠程度；
* 销售价（Sales Price）：亦称我们的价格、零售价等，如果没有任何优惠的（包括促销优惠、会员等级优惠等），
就按这个价格进行销售。所有的优惠规则均是基于这个价格进行计算。

* 特价（Special Price）：优先级最高的定价，忽略所有的价格规则。
* SKU价格（SKU Price）：同一个产品，但是不同的SKU规格（或规格组合）价格不同。
所以在设计上，需要考虑如何基于产品SKU保存价格数据。

* 批发价（Wholesale Price）：纯粹和购买数量相关的价格。常见B2B网站。
* 折扣价（Discount Price）：基于“销售价格”进行促销规则计算，最后得到的折后价格。
有些折扣价格能够反映在产品上，有些则只能反映在总价上，这部分业务归到促销规则来说明。

* 采购价（Import Price）：购入该商品的价格。
* 成本价（Cost Price）：进货价 + 企业运营成本（管理、税费、人力、损耗、场所等）分摊。
用于大致的利润估算分析，也可以用于定价参考价自动计算。

二、价格分类

上面的价格可以分为四大类：

1.显示类价格
市场价格。除有很明显的市场价外，一般电商网站的市场价格都是往高来写，仅用来和销售价格形成反差。
2.管理类价格
包括进货价、成本价。

这类价格不是必须在电商系统中管理，可以在ERP或进销存系统中管理。

用于分析、统计和产品销售价的定价参考使用。
3.销售类价格
销售价、SKU价格、批发价。

就是基于产品本身或数量指定的价格，和市场

4.市场营销价
即：特价、折扣价。

就是基于营销策略所设置的各类规则计算得出的价格。

这类价格不是产品本身的价格，而是通过调用市场营销模块提供的接口计算得出。


三、业务分析

1.显示类价格
因为该价格没有多少业务意义，而且维护频率很低，可以直接在产品或产品SKU实体中维护。

1.管理类价格
采购价：也即进货价，即从供应商方采购该商品的采购价格。采购价格和批次相关，每一批的采购价格会有所不同。

根据产品销售金额减去总采购价可以得出指定时间范围内的毛利润。

成本价：基于每一批的采购价设置，即采购价 + 公司各类运营成本（含税）。一般是估算，比如是30%的运营成本，
那么成本价 = 采购价 × 1.3
根据产品销售金额减去总成本价可以出指定时间范围内的净利润（估算）。

采购价和成本价均和采购批次相关，所以建议纳入库存模块进行处理。

1.销售类价格
包括销售价和批发价，之所以放在销售类，是考虑这两种价格变动频率比较低，而且基本和市场促销活动无关。

参考销售价建议：      

对于网上销售的定价和线下销售的定价往往会有所不同。

对于运营部门在定价时，除了参考公司各个部门的讨论意见和建议等，

在网站系统内，根据成本价和期望利润率给出参考销售价建议是比较有实际价值的。      

期望利润率（Desired profit x% on sales) ：可以针对不同的产品目录来设置。      

参考销售价 = 成本价 × （1 + 期望利润率）

销售价格存在历史数据的需要，所以建议独立产品之外进行设计。

1.市场营销价
由于变更频率很大，而且往往是由若干个规则共同作用下计算获得的价格，所以纳入市场营销模块处理，
并提供接口供产品模块使用。

最终通过上面的描述与总结：复杂的数据库设计如下：


```
CREATE TABLE `goods` (
  `goods_id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT '商品id(SKU)',
  `goods_name` varchar(100) NOT NULL DEFAULT '' COMMENT '商品名称',
  `shop_id` int(10) unsigned NOT NULL DEFAULT '1' COMMENT '店铺id',
  `category_id` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '商品分类id',
  `category_id_1` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '一级分类id',
  `category_id_2` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '二级分类id',
  `category_id_3` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '三级分类id',
  `brand_id` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '品牌id',
  `group_id_array` varchar(255) NOT NULL DEFAULT '' COMMENT '店铺分类id 首尾用,隔开',
  `promotion_type` tinyint(3) NOT NULL DEFAULT '0' COMMENT '促销类型 0无促销，1团购，2限时折扣',
  `promote_id` int(11) NOT NULL DEFAULT '0' COMMENT '促销活动ID',
  `goods_type` tinyint(4) NOT NULL DEFAULT '1' COMMENT '实物或虚拟商品标志 1实物商品 0 虚拟商品 2 F码商品',
  `market_price` decimal(10,2) NOT NULL DEFAULT '0.00' COMMENT '市场价',
  `price` decimal(19,2) NOT NULL DEFAULT '0.00' COMMENT '商品原价格',
  `promotion_price` decimal(10,2) NOT NULL DEFAULT '0.00' COMMENT '商品促销价格',
  `cost_price` decimal(19,2) NOT NULL DEFAULT '0.00' COMMENT '成本价',
  `point_exchange_type` tinyint(3) NOT NULL DEFAULT '0' COMMENT '积分兑换类型 0 非积分兑换 1 只能积分兑换 ',
  `point_exchange` int(11) NOT NULL DEFAULT '0' COMMENT '积分兑换',
  `give_point` int(11) NOT NULL DEFAULT '0' COMMENT '购买商品赠送积分',
  `is_member_discount` int(1) NOT NULL DEFAULT '0' COMMENT '参与会员折扣',
  `shipping_fee` decimal(10,2) NOT NULL DEFAULT '0.00' COMMENT '运费 0为免运费',
  `shipping_fee_id` int(11) NOT NULL DEFAULT '0' COMMENT '售卖区域id 物流模板id  ns_order_shipping_fee 表id',
  `stock` int(10) NOT NULL DEFAULT '0' COMMENT '商品库存',
  `max_buy` int(11) NOT NULL DEFAULT '0' COMMENT '限购 0 不限购',
  `clicks` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '商品点击数量',
  `min_stock_alarm` int(11) NOT NULL DEFAULT '0' COMMENT '库存预警值',
  `sales` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '销售数量',
  `collects` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '收藏数量',
  `star` tinyint(3) unsigned NOT NULL DEFAULT '5' COMMENT '好评星级',
  `evaluates` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '评价数',
  `shares` int(11) NOT NULL DEFAULT '0' COMMENT '分享数',
  `province_id` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '一级地区id',
  `city_id` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '二级地区id',
  `picture` int(11) NOT NULL DEFAULT '0' COMMENT '商品主图',
  `keywords` varchar(255) NOT NULL DEFAULT '' COMMENT '商品关键词',
  `introduction` varchar(255) NOT NULL DEFAULT '' COMMENT '商品简介，促销语',
  `description` text NOT NULL COMMENT '商品详情',
  `QRcode` varchar(255) NOT NULL DEFAULT '' COMMENT '商品二维码',
  `code` varchar(50) NOT NULL DEFAULT '' COMMENT '商家编号',
  `is_stock_visible` int(1) NOT NULL DEFAULT '0' COMMENT '页面不显示库存',
  `is_hot` int(1) NOT NULL DEFAULT '0' COMMENT '是否热销商品',
  `is_recommend` int(1) NOT NULL DEFAULT '0' COMMENT '是否推荐',
  `is_new` int(1) NOT NULL DEFAULT '0' COMMENT '是否新品',
  `is_pre_sale` int(11) DEFAULT '0',
  `is_bill` int(1) NOT NULL DEFAULT '0' COMMENT '是否开具增值税发票 1是，0否',
  `state` tinyint(3) NOT NULL DEFAULT '1' COMMENT '商品状态 0下架，1正常，10违规（禁售）',
  `sort` int(11) NOT NULL DEFAULT '0' COMMENT '排序',
  `img_id_array` varchar(1000) DEFAULT NULL COMMENT '商品图片序列',
  `sku_img_array` varchar(1000) DEFAULT NULL COMMENT '商品sku应用图片列表  属性,属性值，图片ID',
  `match_point` float(10,2) DEFAULT NULL COMMENT '实物与描述相符（根据评价计算）',
  `match_ratio` float(10,2) DEFAULT NULL COMMENT '实物与描述相符（根据评价计算）百分比',
  `real_sales` int(10) NOT NULL DEFAULT '0' COMMENT '实际销量',
  `goods_attribute_id` int(11) NOT NULL DEFAULT '0' COMMENT '商品类型',
  `goods_spec_format` text NOT NULL COMMENT '商品规格',
  `goods_weight` decimal(8,2) NOT NULL DEFAULT '0.00' COMMENT '商品重量',
  `goods_volume` decimal(8,2) NOT NULL DEFAULT '0.00' COMMENT '商品体积',
  `shipping_fee_type` int(11) NOT NULL DEFAULT '1' COMMENT '计价方式1.重量2.体积3.计件',
  `extend_category_id` varchar(255) DEFAULT NULL,
  `extend_category_id_1` varchar(255) DEFAULT NULL,
  `extend_category_id_2` varchar(255) DEFAULT NULL,
  `extend_category_id_3` varchar(255) DEFAULT NULL,
  `supplier_id` int(11) NOT NULL DEFAULT '0' COMMENT '供货商id',
  `sale_date` int(11) DEFAULT '0' COMMENT '上下架时间',
  `create_time` int(11) DEFAULT '0' COMMENT '商品添加时间',
  `update_time` int(11) DEFAULT '0' COMMENT '商品编辑时间',
  `min_buy` int(11) NOT NULL DEFAULT '0' COMMENT '最少买几件',
  `virtual_goods_type_id` int(11) DEFAULT '0' COMMENT '虚拟商品类型id',
  `production_date` int(11) NOT NULL DEFAULT '0' COMMENT '生产日期',
  `shelf_life` varchar(50) NOT NULL DEFAULT '' COMMENT '保质期',
  PRIMARY KEY (`goods_id`)
) ENGINE=InnoDB  DEFAULT CHARSET=utf8 COMMENT='商品表';
```


