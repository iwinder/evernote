---
title: Elasticsearch6聚合 
tags: Elasticsearch
grammar_cjkRuby: true
---

两个主要概念：

| 名称 | 解释| 概念类比 |
| --- | --- | --- |
| Buckets(桶) | 满足特定条件的文档的集合。| 类似于 SQL 的分组（GROUP BY）|
| Metrics(指标) | 对桶内的文档进行统计计算。| 类似于 COUNT() 、 SUM() 、 MAX() 等统计方法 |

每个聚合都是一个或者多个桶和零个或者多个指标的组合。这些是 Elasticsearch2时的内容， Elasticsearch6新提出了Matrix(矩阵聚合)、Pipeline（管道聚合）。

- Matrix(矩阵聚合)

在多个字段（fields ）上运行，并根据从请求的文档字段中提取的值生成矩阵结果的聚合。
与Metrics和Buckets聚合不同，此聚合模式尚不支持脚本。

- Pipeline（管道聚合）
这一类聚合的数据源是其他聚合的输出，然后进行相关指标的计算。

聚合的真正强大所在：**聚合可以嵌套**。

聚合操作数据的双重表示。因此，当在绝对值大于2 ^ 53的long上运行时，结果可能是近似的。

### 构建聚合


在查询请求体中以```aggregations```(也可命名为```aggs```)节点按如下语法构建聚合的基本结构：
```
"aggregations" : {
    "<aggregation_name>" : { <!--聚合的名字 -->
        "<aggregation_type>" : { <!--聚合的类型 -->
            <aggregation_body>   <!--聚合体：对哪些字段进行聚合 -->
        }
        [,"meta" : {  [<meta_data_body>] } ]?  <!--元 -->
        [,"aggregations" : { [<sub_aggregation>]+ } ]? <!--在聚合里面在定义子聚合 -->
    }
    [,"<aggregation_name_2>" : { ... } ]*   <!--聚合的名字 -->
}
```

[ElasticSearch6（五） restful风格 聚合查询-管道聚合](https://blog.csdn.net/weixin_41651116/article/details/81750480)

[elasticsearch系列六：聚合分析（聚合分析简介、指标聚合、桶聚合）](https://blog.csdn.net/qq_26676207/article/details/81019521)
