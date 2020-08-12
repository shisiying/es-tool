# 检索

相关文档

- https://www.elastic.co/guide/en/elasticsearch/reference/7.1/term-level-queries.html
- https://www.elastic.co/guide/en/elasticsearch/reference/7.1/full-text-queries.html


## 基于词项

### 基于TERM词项查询

● Term 是表达语意的最⼩小单位。搜索和利利⽤用统计语⾔言模型进⾏行行⾃自然语⾔言处理理都需要处理理 Term

特点

● Term Level Query: Term Query / Range Query / Exists Query / Prefix Query /Wildcard Query

● 在 ES 中，Term 查询，对输⼊入不不做分词。会将输⼊入作为⼀一个整体，在倒排索引中查找准确的词项，并 且使⽤用相关度算分公式为每个包含该词项的⽂文档进⾏行行相关度算分 – 例例如“Apple Store”

● 可以通过 Constant Score 将查询转换成⼀一个 Filtering，避免算分，并利利⽤用缓存，提⾼高性能



### 结构化搜索

Types of term-level queries

exists query

Returns documents that contain any indexed value for a field.

fuzzy query

Returns documents that contain terms similar to the search term. Elasticsearch measures similarity, or fuzziness, using a Levenshtein edit distance.

ids query

Returns documents based on their document IDs.

prefix query

Returns documents that contain a specific prefix in a provided field.

range query

Returns documents that contain terms within a provided range.

regexp query

Returns documents that contain terms matching a regular expression.
term query


Returns documents that contain an exact term in a provided field.

terms query
Returns documents that contain one or more exact terms in a provided field.

terms_set query

Returns documents that contain a minimum number of exact terms in a provided field. You can define the minimum number of matching terms using a field or script.

type query

Returns documents of the specified type.

## 基于全文的查询

基于全⽂文本的查找

● Match Query / Match Phrase Query / Query String Query

● 索引和搜索时都会进⾏行行分词，查询字符串串先传递到⼀一个合适的分词器器，然后⽣生成⼀一个供查询的词
项列列表

● 查询时候，先会对输⼊入的查询进⾏行行分词，然后每个词项逐个进⾏行行底层的查询，最终将结果进⾏行行合 并。并为每个⽂文档⽣生成⼀一个算分。- 例例如查 “Matrix reloaded”，会查到包括 Matrix 或者 reload 的所有结果。



# 单字符串多字段查询

- Dis Max Query

Disjunction 定义

逻辑或（logical or）又称逻辑析取（logical disjunction）。理解成或就很好解释了。

Disjunction max  通俗解释为：多个之间取或的最大值。

2、官方文档直翻译

将任何与任一查询匹配的文档作为结果返回，但只将最佳匹配的评分作为查询的评分结果返回 。

3、上个例子 解读一下

```cmd
DELETE my_index
PUT /my_index/_doc/1
{
    "title": "Quick brown rabbits",
    "body":  "Brown rabbits are commonly seen."
}
PUT /my_index/_doc/2
{
    "title": "Keeping pets healthy",
    "body":  "My quick brown fox eats rabbits on a regular basis."
}
POST my_index/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "match": {
            "title": {
              "query": "Brown fox"
            }
          }
        },
        {
          "match": {
            "body": "Brown fox"
          }
        }
      ]
    }
  }
}
```

返回结果发现：文档1的评分 大于 文档2。

原因是啥？——简单解释：和评分有关系，文档1 title和body 都包含了：brown字段。


再看加上 dis_max 查询后，

```cmd
POST my_index/_search
{
  "query": {
    "dis_max": {
      "queries": [
        {
          "match": {
            "title": "Brown fox"
          }
        },
        {
          "match": {
            "body": "Brown fox"
          }
        }
      ]
    }
  }
}
```

第二篇文档排序在第一篇前面了。

原因： 将任何与任一查询匹配的文档作为结果返回，但只将最佳匹配的评分作为查询的评分结果返回 。

也就是说：body中包含了 brown fox 两个字段，则该文档的评分就取的body 字段的评分作为总评分。

4、 "tie_breaker" 字段啥作用呢？

"tie_breaker" : 0.7——（中文含义：平局）缓冲剂的作用。

参考上面的例子，如果直接body字段评分返回，title字段就不起作用了。

如果我想让title字段也起作用怎么办？

加上：tie_breaker（取值范围：0到1）

评分计算通俗解释：body评分+title评分*tie_breaker。作为总评分返回。

- Multi Match

Multi-match query 的目的

多字段匹配

2、best_fields

• 为默认值，如果不指定，默认best_fields匹配。

• 含义：多个字段中，返回评分最高的。

• 类似：dis_max query。

• 等价举例：（两个一起看，加深理解）

默认 best_fields 与 dis_max等价

```cmd
POST blogs/_search
{
  "query": {
    "multi_match": {
      "type": "best_fields",
      "query": "Quick pets",
      "fields": [
        "title",
        "body"
      ],
      "tie_breaker": 0.2
    }
  }
}
```

• 与上述best_fields等价

```cmd
POST blogs/_search
{
  "query": {
    "dis_max": {
      "queries": [
        {
          "match": {
            "title": "Quick pets"
          }
        },
        {
          "match": {
            "body": "Quick pets"
          }
        }
      ],
      "tie_breaker": 0.2
    }
  }
}
```

3、most_fields
-含义：匹配多个字段，返回的综合评分（非最高分）
-类似：bool+多字段匹配。
-等价举例：（两个一起看，加深理解）
most_fields 与下面的bool等价

```cmd
GET /titles/_search
{
  "query": {
    "multi_match": {
      "query": "barking dogs",
      "type": "most_fields",
      "fields": [
        "title^10",
        "title.std"
      ]
    }
  }
}
```

• 与上面的most_fields等价

```cmd
GET titles/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "match": {
            "title": {
              "query": "barking dogs",
              "boost": 10
            }
          }
        },
        {
          "match": {
            "title.std": "barking dogs"
          }
        }
      ]
    }
  }
}
```

4、cross_fields
• 含义：跨字段匹配——待查询内容在多个字段中都显示。
• 类似：bool+dis_max组合。
• 等价举例：（两个一起看，加深理解）
与下面的bool查询逻辑一致

```cmd
GET test003/_validate/query?explain=true
{
  "query": {
    "multi_match": {
      "query": "Will Smith",
      "type": "cross_fields",
      "fields": [
        "first_name",
        "last_name"
      ],
      "operator": "and"
    }
  }
}
GET test003/_v
```

alidate/query?explain=true
返回：     "explanation" : "+blended(terms:[first_name:will, last_name:will]) +blended(terms:[first_name:smith, last_name:smith])"
与上面的cross_fields 基本等价，评分不一致，待深究

```cmd
POST test003/_validate/query?explain=true
{
  "query": {
    "bool": {
      "must": [
        {
          "dis_max": {
            "queries": [
              {
                "match": {
                  "first_name": "Will"
                }
              },
              {
                "match": {
                  "last_name": "Will"
                }
              }
            ]
          }
        },
        {
          "dis_max": {
            "queries": [
              {
                "match": {
                  "first_name": "Smith"
                }
              },
              {
                "match": {
                  "last_name": "Smith"
                }
              }
            ]
          }
        }
      ]
    }
  }
}
```
