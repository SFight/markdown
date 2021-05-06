# Es索引创建

```shell
PUT /mainnofulltext
{
  "settings": {
    "analysis": {
      "analyzer": {
        "ik_smart_pinyin": {
          "type": "custom",
          "tokenizer": "ik_smart",
          "filter": ["my_pinyin", "word_delimiter"]
        },
        "ik_max_word_pinyin": {
          "type": "custom",
          "tokenizer": "ik_max_word",
          "filter": ["my_pinyin", "word_delimiter"]
        },
        "standard_pinyin": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["my_pinyin", "word_delimiter"]
        }
      },
      "filter": {
        "my_pinyin": {
          "type" : "pinyin",
          "keep_separate_first_letter" : true,
          "keep_full_pinyin" : true,
          "keep_original" : true,
          "limit_first_letter_length" : 16,
          "lowercase" : true,
          "remove_duplicated_term" : true 
        }
      }
    }
  }, 
  "mappings": {
    "properties": {
      "bid": { "type": "integer" },
      "sup_id": { "type": "integer" },
      "pub_id": { "type": "integer" },
      "isli": { 
        "type": "text", 
        "fields": 
        { 
          "keyword": { "type": "text", "analyzer": "keyword" }
        }
      },
      "isbn_mark": { 
        "type": "text", 
        "fields": 
        { 
          "keyword": { "type": "text", "analyzer": "keyword" }
        }
      },
      "lang": { 
        "type": "text",
        "fields": 
        { 
          "keyword": { "type": "text", "analyzer": "keyword" }
        }
      },
      "date": { "type": "date" },
      "year": { "type": "integer" },
      "price": { "type": "integer" },
      "textbook": { "type": "integer" },
      "isfull": { "type": "integer" },
      "true_pages": { "type": "text" },
      "pages": { "type": "integer" },
      "ptype1": { "type": "integer" },
      "ptype2": { "type": "integer" },
      "cip": { 
        "type": "text",
        "fields": 
        { 
          "keyword": { "type": "text", "analyzer": "keyword" }
        }
      },
      "lib_class": { 
        "type": "text", "fielddata": true, "analyzer": "ik_max_word",
        "fields": 
        { 
          "keyword": { "type": "text", "analyzer": "keyword" }
        }
      },
      "lib_classes": { 
        "type": "text", "fielddata": true, "analyzer": "ik_max_word",
        "fields": 
        { 
          "keyword": { "type": "text", "analyzer": "keyword" }
        }
      },
      "isbn_short": { 
        "type": "text",
        "fields": 
        { 
          "keyword": { "type": "text", "analyzer": "keyword" }
        }
      },
      "name": { 
        "type": "text", "analyzer": "ik_max_word",
        "fields": 
        { 
          "keyword": { "type": "text", "analyzer": "keyword" }
        }
      },
      "name_main": { 
        "type": "text", "analyzer": "ik_max_word",
        "fields": 
        { 
          "keyword": { "type": "text", "analyzer": "keyword" }
        }
      },
      "name_sub": { 
        "type": "text", "analyzer": "ik_max_word",
        "fields": 
        { 
          "keyword": { "type": "text", "analyzer": "keyword" }
        }
      },
      "name_coordinate": { 
        "type": "text", "analyzer": "ik_max_word",
        "fields": 
        { 
          "keyword": { "type": "text", "analyzer": "keyword" }
        }
      },
      "eng_name": { 
        "type": "text", "analyzer": "ik_max_word",
        "fields": 
        { 
          "keyword": { "type": "text", "analyzer": "keyword" }
        }
      },
      "ser_name": { 
        "type": "text", "analyzer": "ik_max_word",
        "fields": 
        { 
          "keyword": { "type": "text", "analyzer": "keyword" }
        }
      },
      "ser_name_sub": { 
        "type": "text", "analyzer": "ik_max_word",
        "fields": 
        { 
          "keyword": { "type": "text", "analyzer": "keyword" }
        }
      },
      "ser_name_coordinate": { 
        "type": "text", "analyzer": "ik_max_word",
        "fields": 
        { 
          "keyword": { "type": "text", "analyzer": "keyword" }
        }
      },
      "volume_name": { 
        "type": "text", "analyzer": "ik_max_word",
        "fields": 
        { 
          "keyword": { "type": "text", "analyzer": "keyword" }
        }
      },
      "author": { 
        "type": "text", "fielddata": true, "analyzer": "ik_max_word",
        "fields": 
        { 
          "keyword": { "type": "text", "analyzer": "keyword" }
        }
      },
      "translator": { 
        "type": "text", "analyzer": "ik_max_word",
        "fields": 
        { 
          "keyword": { "type": "text", "analyzer": "keyword" }
        }
      },
      "edt_name": { 
        "type": "text", "analyzer": "ik_max_word",
        "fields": 
        { 
          "keyword": { "type": "text", "analyzer": "keyword" }
        }
      },
      "pub": { 
        "type": "text", "analyzer": "ik_max_word",
        "fields": 
        { 
          "keyword": { "type": "text", "analyzer": "keyword" }
        }
      },
      "pubsplit": { 
        "type": "text", "fielddata": true, "analyzer": "whitespace",
        "fields": 
        { 
          "pinyin": { "type": "text", "analyzer": "pinyin" }
        }
      },
      "descript": { 
        "type": "text", "analyzer": "ik_max_word"
      },
      "slogan": { 
        "type": "text", "analyzer": "ik_max_word"
      },
      "edt_view": { 
        "type": "text", "analyzer": "ik_max_word"
      },
      "catalog": {
        "type": "text", "analyzer": "ik_max_word"
      },
      "fullcontent": { 
        "type": "text", "analyzer": "ik_max_word"
      },
      "tags": { 
        "type": "text", "analyzer": "whitespace"
      },
      "category": { 
        "type": "text", "analyzer": "whitespace"
      },
      "categoryids": { 
        "type": "text", "fielddata": true, "analyzer": "whitespace"
      },
      "kids": { 
        "type": "text", "analyzer": "whitespace"
      },
      "authorsplit": { 
        "type": "text", "fielddata": true, "analyzer": "whitespace"
      },
      "name_full": { 
        "type": "text", "analyzer": "ik_max_word"
      },
      "issell": { "type": "integer" }
    }
  }
}
```

## 配置远程字典

```shell
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
    <comment>IK Analyzer 扩展配置</comment>
    <!--用户可以在这里配置自己的扩展字典 -->
    <entry key="ext_dict"></entry>
     <!--用户可以在这里配置自己的扩展停止词字典-->
    <entry key="ext_stopwords"></entry>
    <!--用户可以在这里配置远程扩展字典 -->
    <entry key="remote_ext_dict">http://192.168.1.221:8888/es/custom.dic</entry>
    <!--用户可以在这里配置远程扩展停止词字典-->
    <entry key="remote_ext_stopwords">http://192.168.1.221:8888/es
        /custom-stop.dic</entry>
</properties>
```

**在plugins/分词器/config目录下的IKAnalyzer.cfg.xml中**

## 配置热词索引

```shell
# 创建索引
PUT /hotmainnofulltext
{
  "mappings": {
    "properties": {
      "kw": {
        "type": "text",
        "fielddata": true, 
        "analyzer": "keyword"
      }
    }
  }
}
# 新增热词
POST /hotmainnofulltext/_doc
{
  "kw": "计算机科学与技术"
}

# 统计热词
POST /hotmainnofulltext/_search
{
  "aggs": {
    "kw": {
      "terms": {
        "field": "kw",
        "order": {
          "_count": "desc"
        }, 
        "size": 10
      }
    }
  }
}
```

## 默认服务端口

| 服务名称        | 地址        | 端口   | 用户名   | 密码     |
| ----------- | --------- | ---- | ----- | ------ |
| ES          | localhost | 9200 |       |        |
| Kibana      | localhost | 5601 |       |        |
| CanalServer | localhost | 8089 | admin | 123456 |
| ElasticHD   | localhost | 9999 |       |        |
