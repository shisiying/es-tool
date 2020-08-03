# 分词

0、针对考点

Define and use a custom analyzer that satisfies a given set of requirements

[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/analysis.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/analysis.html
)

[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/analysis-custom-analyzer.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/analysis-custom-analyzer.html)

## 为什么需要分词？

中文分词是自然语言处理的基础。

- 1. 语义维度：单字很多时候表达不了语义，词往往能表达。分词相当于预处理，能使后面和语义有关的分析更准确。
- 2. 存储维度：如果所有文章按照单字来索引，需要的存储空间和搜索计算时间就要多的多。
- 3. 时间维度：通过倒排索引，我们能以o(1) 的时间复杂度，通过词组找到对应的文章。

同理的，英文或者其他语种也需要分词。

设计索引的Mapping阶段，要根据业务用途确定是否需要分词，如果不需要分词，建议设置keyword类型；需要分词，设置为text类型并指定分词器。

分词使用的时机：

1）创建或更新文档时，会对文档做分词处理。

2）查询时，会对查询语句进行分词处理。

## 文档转换为倒排索引，发生了什么?

Analyzer的由如下三部分组成：

- character filters 字符过滤
字符过滤器将原始文本作为字符流接收，并可以通过添加，删除或更改字符来转换字符流。
字符过滤分类如下：

1. HTML Strip Character Filter.

用途：删除HTML元素，如\<b>，并解码HTML实体，如＆amp 。

2. Mapping Character Filter

用途：替换指定的字符。

3. Pattern Replace Character Filter

用途：基于正则表达式替换指定的字符。


- tokenizers 文本切分为分词

接收字符流（如果包含了4.1字符过滤，则接收过滤后的字符流；否则，接收原始字符流），将其分词。

同时记录分词后的顺序或位置(position)，以及开始值（start_offset）和偏移值(end_offset-start_offset)。

tokenizers分类如下：

1. Standard Tokenizer

2. Letter Tokenizer

3. Lowercase Tokenizer


- token filters分词后再过滤

针对tokenizers处理后的字符流进行再加工，比如：转小写、删除（删除停用词）、新增（添加同义词）等。

## Elasticsearch自带的Analyzer

5.1 Standard Analyzer

标准分析器是默认分词器，如果未指定，则使用该分词器。

它基于Unicode文本分割算法，适用于大多数语言。

5.2 Whitespace Analyzer

基于空格字符切词。

5.3 Stop Analyzer

在simple Analyzer的基础上，移除停用词。

5.4 Keyword Analyzer

不切词，将输入的整个串一起返回。

更多分词器参考官方文档。

## 自定义分词器的模板
自定义分词器的在Mapping的Setting部分设置。
PUT my_custom_index
{
"settings":{
"analysis":{
"char_filter":{},
"tokenizer":{},
"filter":{},
"analyzer":{}
}
}
}

脑海中还是上面的三部分组成的图示。
其中：

1. "char_filter":{},——对应字符过滤部分；

2. "tokenizer":{},——对应文本切分为分词部分；

3. "filter":{},——对应分词后再过滤部分；

4. "analyzer":{}——对应分词器组成部分，其中会包含：1. 2. 3。
