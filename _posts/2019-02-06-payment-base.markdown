---
layout:     post
title:      "支付系统基础服务"
subtitle:   " \"支付系统\""
date:       2019-02-06 11:00:00
author:     "xiaofeng"
header-img: "img/WechatIMG15.jpeg"
tags:
    - 支付 
    - 技术
---

> **part1 服务性能保障**

业务的快速发展，服务的性能瓶颈就快速的暴露出来。不能派单，不能下单，不能接单，客户端不能登陆，司机端集体掉线。尤其害怕在早晚高峰时段出现。支付系统是公司差不多（吃的第一个螃蟹），14年下半年就完成了重构。通过分库、缓存、解耦等几个方面保障服务高性能。

### #压测数据

首先看一下当时的一些压测数据。环境4台apache服务器（具体配置忘记了，应该是8核16G），单点DB（和其他服务共用），压测工具loadrunner。400Vuser开始，压测开始增减用户并观察TPS，RT，服务器资源使用情况。
结果：500Vuser 6000TPS RT 0.603


![](https://tva1.sinaimg.cn/large/00831rSTly1gdgmmfkgvqj30o0080x59.jpg)
![](https://tva1.sinaimg.cn/large/00831rSTly1gdgmmjscx1j30o006iavr.jpg)
![](https://tva1.sinaimg.cn/large/00831rSTly1gdgmmotut8j30o006qe2a.jpg)
![](https://tva1.sinaimg.cn/large/00831rSTly1gdgmo497hmj30o008wafp.jpg)

### #数据库分库
    根据的支付ID分片到至多256片数据库中。配置项：

```
$account_db_rule = array(
    array(
        'host'      => '127.0.01',
        'range'     => '1-128',  //数据库片范围
        'filter_db' => array(    //过滤的数据库片，有特殊规则需要独立分配到其他数据库的片值
            array(
                'host'  => '127.0.0.1',
                'shardId' => 100,
            ),
        ),
    ),
    array(
        'host'  => '127.0.0.1',
        'range' => '129-256',
    ),
);
```
    DB路由层部分代码

```
    public static function getDbSharding($accountId = null){
       
        if(!empty($accountId)){
            if(is_numeric($accountId)){
                $idRule = $config->metadata->idRule;
                return ceil($accountId / $idRule);
            }else{
                throw new \Exception('account_id error', 401);
            }
        }else{
            return mt_rand(2,$config->metadata->shardCount);
        }
    }

    if(empty($accountId)&&!empty(self::$_adapter)){
        $dbName = self::getFirstKeyByPrefix('account');
        $db = self::$_adapter[$dbName];
        return $db;
    }
        $shardId = self::getDbSharding($accountId);
        $dbName = "account_".$shardId;

```
分库的问题：

1. 分库的一个重要问题是如何保证事务的一致性.当时解决方案是通过二段式提交解决事务的不一致，如果检测到一个库的操作出现问题，对本次操作进行回滚，并通知前端业务。
2. 还是会出现冷热库的情况(大的渠道商数据量还是很大)，要对数据（主要是交易记录）进行退出。db路由增加account_out维度。
3. BI支持，当时大数据工具在公司BI部门还没开始尝试，需要帮助他们做数据聚合。方式就是大小表，不同维度拆表等。
    

### #缓存
支付业务特殊性，缓存使用不是特别多。主要用在卡bin，银行等基础信息。使用缓存的策略上，切记不要把缓存当做数据库使用。缓存失效时间也要改为随机失效，否则一起回源，数据库的压力剧增进而影响服务稳定。

### #解耦

梳理好业务流程，尽可能把产品流程设计成异步化。当时司机提现服务最初是同步操作，司机-》平台-》第三方通道，司机端轮询等结果。开始提现动作是随时随机的，司机数量也不多，没什么问题。后来提现请求多了，整体的数据监控上看到的尖刺就多了起来（第三方通道不是专线，三方通道也要等银行端同步反馈）。服务就改为司机-》平台 --异步--》第三方通道。
这样过了好久，提现规则改为每周的第某天统一开始，8点放开提现后，瞬间大量请求把第三方通道服务给瞬间压挂了...(当时除了头部那两家，其他对接服务质量都不太高，头部那两家也会出问题)。后来服务改为司机-》平台 --延时异步--》第三方通道。





> part2 高可用

服务上云之后，运维的压力小了很多，加减机器不用常住机房了。选择云服务的要求，一是服务要稳定，二是响应要及时（要7*24电话，群支持）。服务目标如果是9995，也要考虑同城双活的服务架构。

![](https://tva1.sinaimg.cn/large/00831rSTly1gdl1bgfqtij30kp0ogadm.jpg)

> part3 可伸缩性

云+jekins+ansible能够保证运维部门快速水平扩展服务。


ansible介绍：ansible是一种自动化运维工具,基于paramiko开发的,并且基于模块化工作，Ansible是一种集成IT系统的配置管理、应用部署、执行特定任务的开源平台，它是基于python语言，由Paramiko和PyYAML两个关键模块构建。集合了众多运维工具的优点，实现了批量系统配置、批量程序部署、批量运行命令等功能.ansible是基于模块工作的,本身没有批量部署的能力.真正具有批量部署的是ansible所运行的模块，ansible只是提供一种框架.ansible不需要在远程主机上安装client/agents，因为它们是基于ssh来和远程主机通讯的.



参考：

[凤凰牌老熊](http://doc.cocolian.cn/)
[ansible](https://www.jianshu.com/p/c82737b5485c)
[ansible-github](https://github.com/ansible/ansible)
