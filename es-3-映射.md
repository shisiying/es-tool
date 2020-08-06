# Mapping

Elasticsearch 映射 考点分布

Define a mapping that satisfies a given set of requirements

[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/mapping.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/mapping.html)

考点梳理：

1. 设置给定条件的mapping

2. 不同字段类型选型（简单字段：如keyword，integer；复杂字段如：join，nested等）

3. 不同字段的细节配置（如：正排索引docvalue取消等）

## Maaping的 全貌认知

映射（Mapping）可以理解为对文档及其字段进行索引或存储的方式。可以拿Mapping和关系型数据库中的schema类比，schema在关系型数据库中指：库表包含的字段及字段存储类型等基础信息。
下文中映射等价于Mapping。

Elasticsearch映射，描述了文档可能具有的字段、属性、每个字段的数据类型以及Lucene是如何索引和存储这些字段的。例如，使用映射定义：

• 哪些字符串字段应视为全文字段。

• 哪些字段包含数字、日期或地理位置。

• 日期值的格式。

• 自定义规则来控制动态添加字段的映射。

映射定义由以下两部分组成：

• 元字段 （Meta-fields）

元字段用于自定义如何处理文档的相关元数据。

元字段的示例包括文档的index，type，id和source字段。


2.1 元字段

各种元字段，它们都以一个下划线开头，例如 type 、 id 和 _source。

2.1.1 Identity 元字段

• _index：表示它所属的文档的索引。

• uid：type和_id的组合键。

• _id：表示文档的ID。

2.1.2 文档源元字段

• _source：表示代表文档正文的原始JSON对象

• size：它表示source字段的大小（以字节为单位）

2.1.3 索引元字段

• _field_names：表示给定文档中包含非空值的所有字段。

• _timestamp：与每个文档相关联的手动或自动生成的时间戳。

• _ttl：表示应该保持活动状态的时间，之后该时间将被删除。

2.1.4 路由元字段

• _parent：当必须创建父子关系时，使用此方法。

• _routing：一个专有值，有助于将给定文档路由到指定的分片。

2.1.5 其他元字段

• _meta：表示应用程序特定的元数据。

2.2 数据类型字段

Elasticsearch的文档支持多种数据类型。

2.2.1 核心数据类型：

核心数据类型是大多数系统都可用并支持的基本数据类型，例如

• 整型 integer,

• 双精度浮点型 double,

• 长整型 long,

• 短整型 short,

• 字节类型 byte,

• 单精度浮点型 float,

• 字符串类型 string（text 和 keyword）,

• 布尔类型 Boolean,

• 日期类型 date,

• 二进制类型 binary。

2.2.2 复杂数据类型

复杂数据类型是核心数据类型的组合，如：

• 数组类型 arrays,

• JSON 对象类型：Object,

• 嵌套数据类型: nested。

数组类型需要多啰嗦几句。

第一： 任何类型都可以包含一个或者多个元素，当数据包含多个元素的时候，它就是数组类型。

第二：数据类型要求一个组内的数据类型一致。

实战举例如下：

```cmd
PUT my_index_010/_doc/1
{
  "class_tags": [
    "新闻",
    "论坛",
    "博客",
    "电子报"
  ],
  "info_array": [
    {
      "name": "Mary",
      "age": 12
    },
    {
      "name": "John",
      "age": 10
    }
  ],
  "size_tags": [
    0,
    50,
    100
  ]
}
```

如上示例，定义三种数组类型：

• class_tags：媒体分类数组，数组元素是字符串类型

• info_array：个人信息数组，数组元素是object类型

• size_tags： 大小规模数组，数组元素是整型

# Nested类型及应用

2.2.3 Geo 数据类型

地理数据类型是用于保存详细信息（例如地点的地理位置）的数据类型，例如：

• geo_point用于标识纬度和经度。

2.2.4 专用数据类型

专用数据类型是那些具有本质上唯一的详细信息的数据类型，例如：

• IP地址，自动完成建议以及从字符串中计数令牌。

• completion， 补全建议导航搜索功能。

2.2.5 多字段类型multi-fields

上个例子，说明问题（考题中一种举例）。

```cmd
PUT my_index_011
{
  "mappings": {
    "properties": {
      "cont": {
        "type": "text",
        "analyzer": "english",
        "fields": {
          "keyword": {
            "type": "keyword"
          },
          "stand":{
            "type":"text",
            "analyzer":"standard"
          }
        }
      }
    }
  }
}
```


上述demo定义了类型cont，使用english分词器，基于英文关键词全文检索。

同时为cont定义了两个扩展类型：

• 其一：keyword，用于精准匹配。

• 其二：standard，用于全文检索。

公司项目中实战，我们往往对需要全文检索的字段设置：text类型，并且指定：ik_max_word等中文分词器。

除此之外，如果这个字段还需要整个字段聚合或者排序等操作，我们往往还会设置其为keyword类型。

举例如下：
```cmd
PUT my_index_012
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "ik_max_word",
        "fields": {
          "keyword": {
            "type": "keyword"
          }
        }
      }
    }
  }
}
以上，在你的公司里面是不是也很常见？
multi-fields是上面所有知识点的综合，一个知识点覆盖了Mapping 类型部分的绝大部分认知，所以是考试的重中之重。

```

## Mapping 类型

问题1：我们通过 logstash 同步Mysql数据到Elasticsearch的时候，不在Elasticsearch做任何Mapping操作，也能写入数据。为什么？

问题2：如下的新索引的操作，我明明没有定义，怎么也能正常提交成功？

```cmd
PUT my_index_013/_bulk
{"index":{"_id":1}}
{"cont":"Each document has metadata associated with it", "visit_count":35,"publish_time":"2020-05-20T18:00:00"}
```

这两个问题的结果和我们传统的关系型数据库认知不一样了？

关系型数据库Oracle，Mysql，PostgreSql中：我们必须先创建库、再在库中指定好schema后创建好表后，才能插入数据。

而 Elasticsearch 不需要，这里就引出了一个概念——动态映射。

3.1 动态映射

动态映射是Elasticsearch的最重要功能之一，它试图摆脱束缚，让用户尽快开始探索数据。 核心：自动检测字段类型后添加新字段。

哪些字段类型支持动态检测呢？

• bool 类型

• float 类型

• long 类型

• Object 类型

• array 类型

• date 类型

除此之外的类型是不支持动态检测匹配的，都会适配为text类型。

动态映射的弊端是什么呢？

• 第一：字段匹配不准确，如：date类型匹配为keyword类型。

举例：

```cmd
DELETE my_index_014
PUT my_index_014/_doc/1
{
  "create_date": "2020-12-26 12:00:00"
}
GET my_index_014/_mapping
```

返回结果可知：create_date 是 text和keyword类型。不是我们期望的date类型。

是有解决方案的，如下，但也需要提前设置匹配规则。

```cmd
DELETE my_index_014
PUT my_index_014
{
  "mappings": {
    "dynamic_date_formats": ["yyyy-MM-dd HH:mm:ss"]
  }
}
```

• 第二：字段匹配相对准确，但不是用户期望的。

举例: 用户期望text类型支持ik分词，但默认的是standard标准分词器。

当然也会有解决方案，借助动态 template解决。

• 第三：占据多余的存储空间。

举例：string类型匹配为：text和keyword两种类型。意味着两次索引。

但实际用户极有可能只期望排序和聚合的keyword类型。

或者极有可能只需要存储text类型，如：网页正文内容只需要全文检索，不需要排序和聚合操作。

• 第四：Mapping可能错误泛滥。

不小心写错的查询语句，由于使用了put操作，很可能写入Mapping中也经常见。

基于此，实际工程开发实战中，建议：使用静态Mapping，提前定义好字段。

3.2 静态映射

静态Mapping或者官方叫做：explicit Mapping 显示映射。

前提：类似关系型数据库Mysql，我们在数据建模前，已经知道保存在文档中的数据的类型。 在这种情况下，每当我们要一起创建索引时，就很容易明确指定字段和类型。

如何严格禁止动态添加字段或者忽略动态添加字段了？

• 通过将dynamic参数设置为false（忽略新字段）。

• 通过将dynamic参数设置为strict（如果遇到未知字段，则引发异常）。

可以在文档和对象级别上禁用此行为。

以下示例中，将"dynamic": false，cont字段是可以写入，但不能被检索。
```cmd
DELETE my_index_015
PUT my_index_015
{
  "mappings": {
    "dynamic": false, 
    "properties": {
      "user": { 
        "properties": {
          "name": {
            "type": "text"
          },
          "social_networks": { 
            "dynamic": true,
            "properties": {}
          }
        }
      }
    }
  }
}
PUT my_index_015/_doc/1
{
  "cont": "Each document has metadata associated"
}
GET my_index_015/_search
{
  "query": {
    "match": {
      "cont": "document"
    }
  }
}
```

如果将"dynamic": "strict", 则写入操作会直接报错如下：

```cmd
{
  "error": {
    "root_cause": [
      {
        "type": "strict_dynamic_mapping_exception",
        "reason": "mapping set to strict, dynamic introduction of [cont] within [_doc] is not allowed"
      }
    ],
  },
  "status": 400
}
DELETE my_index_015
PUT my_index_015
{
  "mappings": {
    "dynamic": "strict", 
    "properties": {
      "user": { 
        "properties": {
          "name": {
            "type": "text"
          },
          "social_networks": { 
            "dynamic": true,
            "properties": {}
          }
        }
      }
    }
  }
}
PUT my_index_015/_doc/1
{
  "cont": "Each document has metadata associated"
}
```

实际业务中，考虑到Elasticsearch动态映射的4个缺点，如果业务恒定，没有动态添加字段，建议设置"dynamic": "strict"。

如果业务不恒定，需要动态添加字段，添加字段时间未知，建议使用 动态 template 实现（后续章节解读template 会详细解读）。


# Nested类型以及应用 &Join类型以及应用


考点来源

Configure an index so that it properly maintains the relationships of nested arrays of objects

[1] https://www.elastic.co/guide/en/elasticsearch/reference/7.2/nested.html

[2] https://www.elastic.co/guide/en/elasticsearch/reference/7.2/parent-join.html

[3] https://www.elastic.co/guide/en/elasticsearch/reference/7.2/search-request-inner-hits.html

[4] https://www.elastic.co/guide/en/elasticsearch/reference/7.2/joining-queries.html

[5] https://www.elastic.co/guide/en/elasticsearch/reference/7.2/query-dsl-terms-query.html#query-dsl-terms-lookup





