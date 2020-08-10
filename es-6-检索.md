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

