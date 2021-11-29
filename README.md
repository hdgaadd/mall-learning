# mall01

**内容：**

- 品牌类的CRUD

**conclusion**

- 放弃代码生成器

  主要是学习编写代码，那不如老老实实自己创建

**problems**

- 在分页查询时，需要传递一个PmsBrandExample对象，且分页返回的数据对应该类

  而supermarket_management分页查询时，不用传递对象，且不用创建分页返回数据对应的类

<br/>

<br/>

<br/>

# mall02

**内容**：添加swagger-ui

<br/>

<br/>

<br/>

# mall03

**内容**：发送验证码、验证验证码

<br/>

<br/>

<br/>

# mall04

**内容**：注册、登录、登录后返回token

**problems**

- 不用创建辅助UserDetailService类，根据username查询出用户实体类的方法

  sloved：在SecurityConfig类有该方法

  baidu：spring-boot-starter-security2.60使用

- 给swagger输入token就可以访问要权限的方法：是否swagger保存了token

  - swagger的按钮应该是把接下来的url请求后面都加上了保存的token

    在SecurityConfig类的jwtAuthenticationTokenFilter()

- 访问方法，怎么传送token去解密token：应该有个传送token的控制层方法

  - 没有传送token的控制层方法，每个方法都被Spring Security拦截，之后跳到登录的逻辑，但在登录逻辑中首先会判断，是否请求有没携带token，有的话解密成功即可访问

  <br/>

  <br/>

  <br/>

# mall05

**内容**：定时任务删除过期订单

**相关表**

- oms_order

  保存订单的商品信息

- oms_order_setting

- pms_sku_stock

**sql文件**

```
<select id="getTimeOutOrders" resultMap="orderDetailMap">
    SELECT
        o.id,
        o.order_sn,
        o.coupon_id,
        o.integration,
        o.member_id,
        o.use_integration,
        ot.id               ot_id,
        ot.product_name     ot_product_name,
        ot.product_sku_id   ot_product_sku_id,
        ot.product_sku_code ot_product_sku_code,
        ot.product_quantity ot_product_quantity
    FROM
        oms_order o
        LEFT JOIN oms_order_item ot ON o.id = ot.order_id
    WHERE
        o.status = 0
        AND o.create_time <  date_add(NOW(), INTERVAL -#{minute} MINUTE); //现在时间 - 时间限度，如果create time < 该结果则超时
</select>
```



```
<update id="updateOrderStatus">
    update oms_order
    set status=#{status}
    where id in
    <foreach collection="ids" item="id" separator="," open="(" close=")">
        #{id}
    </foreach>
</update>
```



```
<update id="releaseSkuStockLock">
    UPDATE pms_sku_stock
    SET
    lock_stock = CASE id
    <foreach collection="itemList" item="item">
        WHEN #{item.productSkuId} THEN lock_stock - #{item.productQuantity}
    </foreach>
    END
    WHERE
    id IN
    <foreach collection="itemList" item="item" separator="," open="(" close=")"> //""对应传递的@Param参数
        #{item.productSkuId}
    </foreach>
</update>
```



**后端类设计**

- OmsOrderDetil的List变量保存了所有超时订单

<br/>

<br/>

<br/>

# mall06

**内容**

- 从数据库获取所有的商品信息，保存在elasticsearch文档型数据库里
- 在elasticsearch实现根据商品某个字段进行搜索、根据某个字段从elasticsearch删除数据
- 把排序字段填充在NativeSearchQueryBuilder对象，再把该对象builder()创建的实例填充到elasticsearch里，实现elasticsearch根据字段进行排序，**本质上是查询所有后，使用ORDER BY进行数据排序**



**problems**

- None of the configured nodes are available: [{#transport#-1}{X7myC_lSTp2XlwysBUlibg}{127.0.0.1}{127.0.0.1:9300}]
  - 应该是elasticsearch没有启动成功

<br/>

<br/>

<br/>

# mall07

**内容**

- 把客户商品浏览记录录入Mongudb

<br/>

<br/>

<br/>

# mall08

**内容**

> 消息队列为RabbitMQ

- 创建`取消订单`消息队列、创建`延迟`消息队列，设置`延迟`消息队列延迟时间为60min
- 用户下单后，会触发逻辑：发送消息请求保存在`延迟`消息队列，在`延迟`消息队列保存60min后，把请求发送给`取消订单`消息队列
- `取消订单`消息队列接收到消息后，判断订单是否付款，如果未付款即取消订单
- 实现了用户未在规定时间60min内付款，就取消订单的功能

> 本质使用消息队列的延迟队列功能

<br/>

<br/>

<br/>

# mall09

**内容**

- 实现文件上传

<br/>

<br/>

<br/>

