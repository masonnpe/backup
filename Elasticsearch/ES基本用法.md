---
title: ES基本用法
categories: Elasticsearch
tags: [Elasticsearch]
---

document  数据 需要有唯一key





#### 分词器

character filters  去p标签  字符替换



tokenizer 切分 成单词

token filters  转小写  删除the   of   的 这 那  这种没实际意义的词



```json
POST _analyze
{
  "analyzer":"standard",
  "filter": ["lowercase"], 
  "text":"HeLLo wORld!"
}
```

result

```json
{
  "tokens": [
    {
      "token": "hello",
      "start_offset": 0,
      "end_offset": 5,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "world",
      "start_offset": 6,
      "end_offset": 11,
      "type": "<ALPHANUM>",
      "position": 1
    }
  ]
}
```

自带分词器

standard 默认分词器  

simple   非字母字符切分（空格 -之类的）  小写处理

whitespace   空格切分

stop    删除语气词  the  of

keyword  不做分词  

pattern  正则表达式自定义   默认\w+,非字符做分割

language  不同语言的分词

中文分词 IK   jieba  

基于自然语言  模型 算法  hanpl thulac



#### 自定义分词

```json
PUT test_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer":{
          "type":"custom",
          "tokenizer":"standard",
          "char_filter": ["html_strip"],
          "filter":["lowercase"]
        }
      }
    }
  }
}

POST /test_index/_analyze
{
  "analyzer":"my_analyzer",
  "text":"HeLLo wORld!"
}
```





自定义mapping

```json
PUT my_index
{
  "mappings": {
    "doc":{
      "dynamic":false,
      "properties": {
        "title":{
          "type": "text"
        },
        "name":{
          "type": "keyword"
        },
        "age":{
          "type": "integer"
        }
      }
    }
  }
}



PUT my_index/doc/1
{
  "name":"mason",
  "age":1,
  "title":"mason good",
  "money":3123
}

_all 所有字段匹配
GET my_index/_search
{
  "query": {
    "match": {
      "_all": "mason"
    }
  }
}


"index": false 不会被查询
PUT my_index
{
  "mappings": {
    "doc":{
      "dynamic":false,
      "properties": {
        "title":{
          "type": "text",
          "index": false
        }
      }
    }
  }
}

```



