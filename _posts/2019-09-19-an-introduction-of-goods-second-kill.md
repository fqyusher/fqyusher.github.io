---
layout: post
title:  "秒杀产品的理解与应对"
date:   2019-09-19 19:22:11 +0800
category: [技能, 原创, 并发]
tags: [redis, concurrency]
excerpt: "秒杀产品的理解与应对"
---

# 概述

本文记录自己对秒杀场景的理解与应对措施。

## 特征

**秒杀**主要发生在电商活动中，商家通过采取低价促销或者饥饿营销，提供较低廉的价格或有限的数量给大量用户抢购，想以此提升自身平台的口碑及增加部分品牌的曝光度。

由于关注度很高，用户量在活动开始前后一段时间出现激增的情况，通常会是平常的数十倍乃至上百倍，因此若不处理得当，很容易造成平台服务器奔溃，严重影响用户体验。

**秒杀**活动主要有三个阶段存在大流量访问：

-   **秒杀前:** 用户停留在活动页面，不断进行页面刷新，页面请求达到峰值。
-   **秒杀中:** 用户不断点击购买按钮，购买请求达到峰值。
-   **秒杀后:** 大部分用户未抢到商品，不断刷新页面等待其他用户退单，页面请求再次飙升。

## 应对措施

>   客户端

-   **CDN:** 静态资源存在CDN上。

-   **浏览器缓存:** 针对秒杀商品页面采取页面缓存，静态数据缓存在浏览器上。
-   **前端JS:** 抢购按钮置灰，通过JS脚本进行倒计时处理，到时解除点击限制；也可加入限制x秒内允许一次提交。

>   服务端

-   **Nginx:** 按连接数限流(ngx_http_limit_conn_module)、按请求速率限流(ngx_http_limit_req_module)。

-   **用户唯一标识:** 限制用户必须为注册用户，严格点可以要求用绑定手机号或身份证等信息，但这可能会造成伤敌一千自损八百的后果。

-   **读写分离:** 将秒杀商品信息缓存至Redis，提供读实例；根据`goods_status`状态来判断抢购是否开始；根据`goods_left=goods_count-goods_bought_num`的值来判断是否允许下单。

    ```json
    // 秒杀商品
    {
        "goods_count": 100, // 商品库存数量
        "goods_status": 0, // 商品抢购状态
        "goods_bought_num": 0, // 商品已抢数量
    }
    ```

-   **库存扣量:** 下单成功后，需要对库存量进行更新，这里使用Redis的Lua脚本来保证多个命令的原子性。

    ```lua
    -- 库存更新lua脚本
    local key = ARGV[1];
    local buy_num = tonumber(ARGV[2]);
    --[[ 若商品量大，则传入以下参数，供异步入库
    local user_id = tonumber(ARGV[3]);
    local goods_id = tonumber(ARGV[4]);
    ]]
    
    if not buy_num or buy_num == 0 then
        return 0;
    end
    
    local goods_vals = redis.call("HMGET", key, "goods_count", "goods_bought_num");
    local goods_count = tonumber(goods_vals[1]);
    local goods_bought_num = tonumber(goods_vals[2]);
    
    if not goods_count or not goods_bought_num then
        return 0;
    end
    
    if buy_num <= goods_count - goods_bought_num then
        redis.call("HINCRBY", key, "goods_bought_num", buy_num);
        --[[ 异步入库代码
        local order_info = json.encode({uid: user_id, goods_id: goods_id, buy_num: buy_num, order_time: os.time()});
        redis.call("LPUSH", "order_list:"..goods_id, order_info);
        ]]
        -- 
        return buy_num;
    end
    return 0;
    ```

>   数据层

-   下单入库:** 若秒杀商品较少时，可以直接操作数据库，若商品较多数以万计的话，直接入库会给数据库带来极大压力甚至导致奔溃，因此可以先将订单信息入Redis队列，采用异步消费入库。

    ```json
    // 订单详情 order_list:good_id
    {
        "uid": 123, // 用户id
        "goods_id": 456, // 商品id
        "buy_num": 1, // 购买数量
        "order_time": 1234567890, // 下单时间
        // 其他信息
    }
    ```

    ```shell
    redis> RPOP order_list:goods_id
    ```

-   **付款处理:** 一般秒杀商品下单成功后，系统会给用户一定的付款时间，如十五分钟，这样可以缓解支付服务端的压力。若中途用户取消订单或者超时，系统将更新`goods_bought_num`，那些还在不断刷新抢购商品页面的用户就能重新下单了。

-   **结束抢购:** 系统会在活动时间内定时计算数据库中的付款数量是否达到秒杀的库存数，若已达，则更新`goods_status`标记为活动结束。

-   队列削峰: 将下单和付款流程分开，下单操作使用Redis队列，用户点击抢购按钮，若库存足够，将用户唯一标识入队列，之后消费队列，消费成功则更新`goods_bought_num`，否则返回库存不足。使用Redis的有序集合，用户点击下单直接将用户唯一标示插入至集合中

## 扩展补充

>   Redis队列+集合

消费队列时，根据`SISMEMBER odrer_user_list uid`与`RPOP goods_queue:goods_id`结果做判断。

-   **队列:** 提前将商品库存存入队列中，然后依次消费库存。
-   **集合:** 用于存放用户唯一标识，若用户不在集合中，则允许调用消费程序；若超时付款或者取消付款，则移除。