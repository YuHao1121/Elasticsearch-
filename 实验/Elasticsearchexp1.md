# 《实验二：索引操作与文档操作练习》

**学院：省级示范性软件学院**
**题目：《实验二：索引操作与文档操作练习》
姓名：唐玉亮
学号：2100230021
班级：软工2202
日期：2024-09-14
实验环境：Elasticsearch8.12.2、 Kibana8.12.2**

## 实验目的

1. 掌握Elasticsearch 安装IK分词器安装方法

1. 掌握Elasticsearch 索引操作方法

1. 掌握Elasticsearch 文档操作训练

1. 掌握Elasticsearch 高级查询与DSL训练



## 实验内容

### 索引操作练习

#### 创建索引

##### 用户信息 (User Information) 索引创建代码

```json
PUT /user_information
{
  "mappings": {
    "properties": {
      "user_id": {
        "type": "keyword"
      },
      "name": {
        "type": "text",
        "analyzer": "standard"
      },
      "email": {
        "type": "keyword"
      },
      "date_of_birth": {
        "type": "date",
        "format": "yyyy-MM-dd"
      },
      "gender": {
        "type": "keyword"
      },
      "address": {
        "type": "text",
        "analyzer": "standard"
      },
      "phone_number": {
        "type": "keyword"
      },
      "registration_date": {
        "type": "date",
        "format": "yyyy-MM-dd"
      },
      "last_login": {
        "type": "date",
        "format": "yyyy-MM-dd"
      },
      "status": {
        "type": "keyword"
      }
    }
  }
}
```



##### 代码运行结果

``` json
{
  "acknowledged": true,
  "shards_acknowledged": true,
  "index": "user_information"
}
```

##### 索引查询代码

#查看user_information索引

``` json
GET /user_information
```

##### 索引查询结果

#查看user_information索引结果

``` json
{
  "user_information": {
    "aliases": {},
    "mappings": {
      "properties": {
        "address": {
          "type": "text",
          "analyzer": "standard"
        },
        "date_of_birth": {
          "type": "date",
          "format": "yyyy-MM-dd"
        },
        "email": {
          "type": "keyword"
        },
        "gender": {
          "type": "keyword"
        },
        "last_login": {
          "type": "date",
          "format": "yyyy-MM-dd"
        },
        "name": {
          "type": "text",
          "analyzer": "standard"
        },
        "phone_number": {
          "type": "keyword"
        },
        "registration_date": {
          "type": "date",
          "format": "yyyy-MM-dd"
        },
        "status": {
          "type": "keyword"
        },
        "user_id": {
          "type": "keyword"
        }
      }
    },
    "settings": {
      "index": {
        "routing": {
          "allocation": {
            "include": {
              "_tier_preference": "data_content"
            }
          }
        },
        "number_of_shards": "1",
        "provided_name": "user_information",
        "creation_date": "1726217071093",
        "number_of_replicas": "1",
        "uuid": "i-HcpcpUQLqMZZY9deDKiA",
        "version": {
          "created": "8500010"
        }
      }
    }
  }
}
```

##### 产品目录 (Product Catalog) 索引创建代码

``` json
PUT /product_catalog
{
  "mappings": {
    "properties": {
      "product_id":{
        "type":"keyword"
      },
      "name": {
        "type": "text",
        "analyzer": "standard"
      },
      "description": {
        "type": "text",
        "analyzer": "standard"
      },
      "category": {
        "type": "keyword"
      },
      "price": {
        "type": "double"
      },
      "stock_quantity": {
        "type": "integer"
      },
      "supplier": {
        "type": "keyword"
      },
      "release_date": {
        "type": "date",
        "format": "yyyy-MM-dd"
      },
      "tags": {
        "type": "keyword"
      },
      "rating": {
        "type": "float"
      }
    }
  }
}

```

##### 代码运行结果

``` json
{ 
  "acknowledged": true,
  "shards_acknowledged": true,
  "index": "product_catalog"
}
```

##### 索引查询代码

#查看product_catalog索引

``` json
GET /product_catalog
```

##### 索引查询结果

#查看product_catalog索引结果

``` json
{
  "product_catalog": {
    "aliases": {},
    "mappings": {
      "properties": {
        "category": {
          "type": "keyword"
        },
        "description": {
          "type": "text",
          "analyzer": "standard"
        },
        "index": {
          "properties": {
            "_id": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            },
            "_index": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            }
          }
        },
        "name": {
          "type": "text",
          "analyzer": "standard"
        },
        "price": {
          "type": "double"
        },
        "product_id": {
          "type": "keyword"
        },
        "rating": {
          "type": "float"
        },
        "release_date": {
          "type": "date",
          "format": "yyyy-MM-dd"
        },
        "stock_quantity": {
          "type": "integer"
        },
        "supplier": {
          "type": "keyword"
        },
        "tags": {
          "type": "keyword"
        }
      }
    },
    "settings": {
      "index": {
        "routing": {
          "allocation": {
            "include": {
              "_tier_preference": "data_content"
            }
          }
        },
        "number_of_shards": "1",
        "provided_name": "product_catalog",
        "creation_date": "1726217309391",
        "number_of_replicas": "1",
        "uuid": "_VThcvLgTP287AZpA3jOOQ",
        "version": {
          "created": "8500010"
        }
      }
    }
  }
}
```



##### 订单记录 (Order Records) 索引创建代码

``` json
PUT /order_records
{
  "mappings": {
    "properties": {
      "order_id": {
        "type": "keyword"
      },
      "customer_id": {
        "type": "keyword"
      },
      "order_date": {
        "type": "date",
        "format": "yyyy-MM-dd"
      },
      "status:": {
        "type": "keyword"
      },
      "total_amount": {
        "type": "double"
      },
      "items": {
        "type": "nested",
        "properties": {
          "product_id": {
            "type": "keyword"
          },
          "quantity": {
            "type": "integer"
          },
          "price": {
            "type": "double"
          }
        }
      },
      "shipping_address": {
        "type": "text",
        "analyzer": "standard"
      },
      "payment_method": {
        "type": "keyword"
      },
      "shipping_date": {
        "type": "date",
        "format": "yyyy-MM-dd"
      },
      "delivery_date": {
        "type": "date",
        "format": "yyyy-MM-dd"
      }
    }
  }
}
```

##### 代码运行结果

``` json
{
  "acknowledged": true,
  "shards_acknowledged": true,
  "index": "order_records"
}
```

##### 索引查询代码

``` json
GET /order_records
```

##### 索引查询结果

``` json
{
  "order_records": {
    "aliases": {},
    "mappings": {
      "properties": {
        "customer_id": {
          "type": "keyword"
        },
        "delivery_date": {
          "type": "date",
          "format": "yyyy-MM-dd"
        },
        "index": {
          "properties": {
            "_id": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            },
            "_index": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            }
          }
        },
        "items": {
          "type": "nested",
          "properties": {
            "price": {
              "type": "double"
            },
            "product_id": {
              "type": "keyword"
            },
            "quantity": {
              "type": "integer"
            }
          }
        },
        "order_date": {
          "type": "date",
          "format": "yyyy-MM-dd"
        },
        "order_id": {
          "type": "keyword"
        },
        "payment_method": {
          "type": "keyword"
        },
        "shipping_address": {
          "type": "text",
          "analyzer": "standard"
        },
        "shipping_date": {
          "type": "date",
          "format": "yyyy-MM-dd"
        },
        "status": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "status:": {
          "type": "keyword"
        },
        "total_amount": {
          "type": "double"
        }
      }
    },
    "settings": {
      "index": {
        "routing": {
          "allocation": {
            "include": {
              "_tier_preference": "data_content"
            }
          }
        },
        "number_of_shards": "1",
        "provided_name": "order_records",
        "creation_date": "1726217542458",
        "number_of_replicas": "1",
        "uuid": "MqiZCZPqQ5K4j6aq8_t5Hg",
        "version": {
          "created": "8500010"
        }
      }
    }
  }
}
```

#### 修改索引

##### 用户信息 (User Information) 索引修改

##### 代码运行结果

#### 删除索引

##### 代码

##### 代码运行结果

#### 查看所有

##### 代码

##### 代码运行结果

### 文档操作练习

#### 创建文档

##### 创建用户信息数据文档代码

##### 执行结果

##### 创建产品目录数据文档代码

##### 执行结果

##### 创建订单记录数据文档代码

##### 执行结果

#### 修改文档

##### 增量修改

##### 全量修改

#### 删除文档

##### 代码

##### 执行结果

##### 再次查询用户信息数据文档，可以看到id为001的数据已经不存在：

#### 查看文档

##### 通过id查询文档

##### 执行结果

##### 批量查询文档

##### 执行结果

##### 查询所有文档

##### 执行结果

### 高级查询&DSL练习

#### 关于用户信息数据查询

##### 1.查询所有女性用户的姓名和电子邮件。

##### 执行结果

##### 2.查找最后登录日期在2024年9月1日之后的所有活跃用户。

##### 执行结果

##### 3.查询住在"Anytown"的用户。

##### 执行结果

##### 4.查找出生日期在1990年之后的所有用户。

##### 执行结果

##### 5.查询所有状态为"inactive"的用户。

##### 执行结果

##### 6.查找注册日期在2023年1月1日到2023年12月31日之间的用户。

##### 执行结果

##### 7.查询名字为"Bob Smith"的用户的详细信息。

##### 执行结果

##### 8.查找电话号码以"123"开头的用户。

##### 执行结果

##### 9.查询电子邮件域为"example.com"的所有用户。

##### 执行结果

##### 查找所有名字中包含"Lee"的用户。

##### 执行结果

#### 关于产品目录数据查询

##### 1.查询所有类别为"Audio"的产品名称和价格。

##### 执行结果

##### 2.查找价格高于50美元的所有产品。

##### 执行结果

##### 3.查询库存数量少于100的产品。

##### 执行结果

##### 4.查找评分高于4.5的所有产品。

##### 执行结果

##### 5.查询标签中包含"smart"的所有产品。

##### 执行结果

##### 6.查找供应商为"TechCorp"的产品。

##### 执行结果

##### 7.查询发布日期在2023年6月1日之后的所有产品。

##### 执行结果

##### 8.查找描述中包含"wireless"的产品。

##### 执行结果

##### 9.查询价格在20美元到100美元之间的所有产品。

##### 执行结果

##### 10.查找产品名称中包含"Light"的所有产品。

##### 执行结果

#### 关于订单记录数据查询

##### 1.查询所有状态为"completed"的订单的订单ID和总金额。

##### 执行结果

##### 2.查找总金额大于100美元的所有订单。

##### 执行结果

##### 3.查询支付方式为"paypal"的订单。

##### 执行结果

##### 4.查找订单日期在2024年2月之后的所有订单。

##### 执行结果

##### 5.查询包含产品ID为"P001"的订单。

##### 执行结果

##### 6.查找所有状态为"cancelled"的订单的客户ID。

##### 执行结果

##### 7.查询发货日期在2024年1月15日之前的订单。

##### 执行结果

##### 8.查找使用"credit_card"支付的订单。

##### 执行结果

##### 9.查询总金额在50美元到200美元之间的所有订单。

##### 执行结果

##### 10.查找订单ID中包含"OR01"的所有订单。

##### 执行结果

## 问题及解决方法



# 

​	





