# 索引增删改查

1、考点覆盖


1.1 定义满足给定条件的索引

Define an index that satisfies a given set of requirements

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/indices.html

1.2 索引增、删、改、查操作

Perform index, create, read, update, and delete operations on the documents of an index

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/docs.html

# 别名


考点分布


从别名分类、索引别名实践、索引别名的好处、索引别名常见问题及坑解读、字段别名实践一把五个方面进行详细解读。

0.1 Define and use index aliases

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/indices-aliases.html

考点梳理：

• 新建索引指定别名

• 新建模板指定别名

• 为已有索引添加别名

本文扩展了考点，添加了字段别名的讲解，便于对别名一并认知、加深理解。

实际认证考试中的别名会以以下方式：

• 举例1：创建索引，指定索引的别名为：XXX，索引的其他满足XXX条件等。

• 举例2：创建满足如下条件XXX的模板，条件之一是指定别名


## 索引别名的好处

### 大数据量的管理


场景： 实战中，可能需要基于时间的数据保留策略（利用rollover机制实现），并从系统中删除旧数据。

使用索引别名：

• 好处1：来简化从Elasticsearch中删除数据的过程。

• 好处2：在没有任何停机时间的情况下从Elasticsearch中删除最旧的数据，不会出现任何查询中断，也不会进行任何客户端更改。


### 用户无感知（0宕机的前提下）的重建索引

实战中，索引的设计可能不是一步到位。

随着业务的扩展，可能会在开发的中后期，调整索引Mapping结构，

比如：

• 1）ik_smart改成ik_max_word分词以高效分词，

• 2）long类型改成keyword以提升检索效率，

• 3）修改索引分片数以便于机器横向扩展，

• 4）索引分成更小粒度的索引等以提升性能。

通常的做法，都需要借助：reindex操作完成索引的迁移。

如果要确保线上环境的可靠运行且用户无感知（即无需告知用户，不影响用户的业务），使用别名指向更改前和更改后的索引是绝佳方案。

实战举例:
```cmd
POST /_aliases?pretty
{
 "actions": [
   {
     "remove": {
       "index": "visitor_logs_2018",
       "alias": "visitor_logs"
     }
   },
   {
     "add": {
       "index": "visitor_logs_2018_01",
       "alias": "visitor_logs"
     }
   }
 ]
}
```

试想一下，如果没有索引别名呢？

答案：

• 1、无法保证查询的连续性；

• 2、无法保证线上业务查询的可靠性（需要告知用户，业务中断一段时间）。

### 总结

• 实战中，一般在开发中后期才发现索引别名的妙处。正如文中分析：1、高效索引管理；2、用户无感知维护数据修改更新。

• 建议：相同索引别名的物理索引有一致的Mapping和数据结构，以提升检索效率。

• 注意：发挥索引别名在检索方面的优势，在写入和更新还得使用物理索引。

# 模版

0、考点覆盖

0.1 Define and use an index template for a given pattern that satisfies a given set of requirements

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/indices-templates.html

考点梳理：

• 创建满足给定条件的索引模板

• 组合考点创建模板同时：指定mapping，指定setting，指定ingest，指定analyzer，指定别名，指定order优先级

0.2 Define and use a dynamic template that satisfies a given set of requirements

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/dynamic-templates.html

考点梳理：

• 创建满足给定模板条件的索引，如：text_*开头指定为text类型

• 创建满足给定模板条件的模板，可以结合2.4 一起考！

实战中，你会发现：template是高效的的工具，可全局设置多个索引且批量生效，避免的不必要的返工。

相比之下Mapping和别名优势如下：

• Mapping有助于我们保持数据库结构的一致性，并为我们提供Elasticsearch丰富的数据类型以及更复杂的自定义Mapping和分析类型。

• 别名Alias对于最大限度地无需停服完成索引切换起到重要作用。

因此，当我们新系统准备选型Elasticsearch作为核心数据存储时，优先注意数据建模；数据建模的过程中要整合template、alias和mapping的综合优势，才能保证模型的健壮性。

## Elasticsearch模板进阶实战

当template和Mapping的dynamic_templates结合就相当于放了大招。

直接拿个实战例子说明问题。

需求1：默认如果不显示指定Mapping，数值类型的值会被映射会long类型，但实际业务数值都比较小，会有存储浪费。需要将默认值改成integer。

需求2：date_*开头的字符统一匹配为date日期类型

```cmd
PUT _template/sample_dynamic_template
{
  "index_patterns":[
    "sample*"
    ],
    "mappings":{
      "dynamic_templates":[
        
        {
          "handle_integers":{
            "match_mapping_type":"long",
            "mapping":{
              "type":"integer"
            }
          }
        },
        
        {
          "handle_date":{
            "match":"date_*",
            "unmatch":"*_text",
            "mapping":{
              "type":"date"
            }
          }
        }
      ]
    }
}

DELETE sampleindex

PUT sampleindex/_doc/1
{
  "value":123,
  "date_curtime":"1574494620000"
}

```