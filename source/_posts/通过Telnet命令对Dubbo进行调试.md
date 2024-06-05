---
title: 通过Telnet命令对Dubbo进行调试
date: 2024-06-05 20:28:36
tags:
---

# 背景

线上有个权益充值卡兑换失败了，需要修改生产数据让用户重新兑换。灵机一动想到可以通过跳板机连接线上Dubbo服务，我们自己直接帮用户执行兑换权益这一操作

# 相关代码

```bash
# 连接Dubbo
telnet 127.0.0.1 20905
# 查看服务列表
ls 
# 查看指定服务暴露的方法
ls com.yihuan.easychange.sale.bean.dubboservice.bussiness.yanxuan.point.UserSignInRewardRuleDubboService:1.0.0
# 查看指定服务暴露的方法（详细）
ls -l com.yihuan.easychange.sale.bean.dubboservice.bussiness.yanxuan.point.UserSignInRewardRuleDubboService:1.0.0
# 将指定服务设置为默认服务
cd com.yihuan.easychange.sale.bean.dubboservice.bussiness.yanxuan.point.UserSignInRewardRuleDubboService:1.0.0
# 调用默认服务中的方法
invoke exchangeCard({"id": 1621, "phone": "13038016671", "userId":212159})

telnet 127.0.0.1 20907
cd com.yihuan.easychange.sale.bean.dubboservice.bussiness.bangmai.BangmaiOrderSettleDubboService
invoke settle({"id":"608290359457923072","orderNum":"10202401040000033802","jingpaiOrderNum":"","name":"魅族 魅蓝 S6 32G 大陆国行","appendDetectPictureUrls":[],"recycleOrderNo":"20231211599541192765755392","productId":600743841389187072,"merchandiseId":"8600615997254291456","brandId":9,"brand":"魅族","modelId":675,"model":"魅族 魅蓝 S6","skuId":0,"sku":"32G 大陆国行","evaluationLevel":"8新","productPrice":11100,"settlePrice":9990,"serviceCharge":1110,"shipPrice":0,"amountPayable":11100,"amountPaid":11100,"status":4,"statusDescription":0,"statusChangeTime":1704959588362,"payMethod":1,"instalmentPlan":0,"expireTime":1704432523176,"expireCountDown":0,"payTime":1704346130314,"sendTime":1704354788283,"completeTime":1704959588362,"cancelTime":0,"sellerId":642221,"sellerName":"小当_355929","sellerAvatar":"https://yihuandl.oppo.com/default/20220216/063040_1493895599261618176_1645007440516.png","buyerId":659633,"buyerName":"小当_067395","buyerPhone":"17870067395","buyerInstallChannel":"5","buyerInstallChannelName":"小米","buyerMessage":"","consigneeName":"刘晓锋","consigneeProvince":"江西省","consigneeCity":"赣州市","consigneeCounty":"南康区","consigneeAddress":"横寨中学","consigneePhone":"17870067395","expressCompany":"顺丰快递","expressNo":"SF1669168697479","afterSaleStatus":4,"gmtCreated":1704346123000,"gmtCreatedBegin":0,"gmtCreatedEnd":0,"remark":""})
```

# 参考资料

[telnet 命令演示 以及 Dubbo常见错误解决方法 - 灰信网（软件开发博客聚合）](https://www.freesion.com/article/6156686650/)

```bash
count XxxService: 统计 1 次服务任意方法的调用情况
count XxxService 10: 统计 10 次服务任意方法的调用情况
count XxxService xxxMethod: 统计 1 次服务方法的调用情况
count XxxService xxxMethod 10: 统计 10 次服务方法的调用情况
```
