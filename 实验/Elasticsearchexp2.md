# 《实验三：聚合操作练习》

**学院：省级示范性软件学院**

**课程：高级数据库技术与应用**

**题目：《实验三：聚合操作练习》**

**姓名：唐玉亮**

**学号：2100230021**

**班级：软工2202**

**日期：2024-10-13**

**实验环境：Elasticsearch8.12.2、 Kibana8.12.2**

## 实验目的

**学习并熟练使用Elasticsearch聚合操作**

## 笔记

### Status聚合

#### Stats 聚合提供一组基本的统计信息,包括count_value, min, max, avg 和 sum。

部分用法如下：

``` json
//分组
GET /logs/_bulk
{
    "aggs"：{ //代表聚合名称
		"price_group":{ //名称，随意起名
    		"terms":{ //分组：对什么分组，由下面的“field”决定
    			"field":"price" //分组字段
			}
		}	   	 
	},
	"size":0 //表示只要统计结果，其它原始数据不显示
}

//求平均值
GET /logs/_bulk
{
    "aggs"：{ //代表聚合名称
		"price_avg":{ //名称，随意起名
    		"avg":{ //平均值
    			"field":"price" //分组字段
			}
		}	   	 
	},
	"size":0 //表示只要统计结果，其它原始数据不显示
}


//先分组再求和or求平均
//需要注意的是求和是对分组之后的值求和，所以我们要注意子聚合操作要在分组内部，也就是要注意正确嵌套
GET /ecommerce/_search
{
  "aggs":{ //代表聚合名称
    "product_category_group":{ //名称，随意起名
      "terms":{ //分组：对什么分组，由下面的“field”决定
        "field": "product_category" //分组字段
      },
      "aggs":{ //对产品类别分完组之后再进行聚合操作
        "total_amount":{ //表示总销售额
          "sum":{ //对...求和
            "field":"total_amount" //求和字段为total_amount：总金额
          }
        }
      }
    }
    
  },
  "size":0 //表示只要统计结果，其它原始数据不显示
}
```

### Date Histogram 聚合

Date Histogram 聚合专门用于日期字段的直方图。

用法如下：

``` json
{
  "aggs": {
    "sales_by_month": {
      "date_histogram": {//表示按...分组
        "field": "order_date",
        "calendar_interval": "month",
        "format": "yyyy-MM-dd"
      }
    }
  }
}
```

### Range聚合

Range 聚合根据数值范围进行分组。

用法如下：

```json
//示例：计算每个年龄段（18-30，31-50，51+）的客户数量
GET /ecommerce/_search
{
  "aggs": {
    "customer_age_range": {
      "range": {
        "field":"customer_age",
        "ranges": [
          {"from": 18, "to":30},
          {"from": 31, "to":50},
          {"from": 51}
        ]
      }
    }
  },
  "size":0 //表示只要统计结果，其它原始数据不显示
}
```



## 实验内容

### 电商数据分析

#### 索引表述信息

1. order_id: 订单ID
2. order_date: 订单日期
3. customer_id: 客户ID
4. customer_name: 客户姓名
5. customer_gender: 客户性别
6. customer_age: 客户年龄
7. customer_city: 客户所在城市
8. product_id: 产品ID
9. product_name: 产品名称
10. product_category: 产品类别
11. quantity: 购买数量
12. price: 单价
13. total_amount: 总金额
14. payment_method: 支付方式
15. is_returned: 是否退货

#### 创建索引

代码：

``` json
PUT /ecommerce
{
  "mappings": {
    "properties": {
      "order_id": { "type": "keyword" },
      "order_date": { "type": "date" },
      "customer_id": { "type": "keyword" },
      "customer_name": { "type": "keyword" },
      "customer_gender": { "type": "keyword" },
      "customer_age": { "type": "integer" },
      "customer_city": { "type": "keyword" },
      "product_id": { "type": "keyword" },
      "product_name": { "type": "keyword" },
      "product_category": { "type": "keyword" },
      "quantity": { "type": "integer" },
      "price": { "type": "float" },
      "total_amount": { "type": "float" },
      "payment_method": { "type": "keyword" },
      "is_returned": { "type": "boolean" }
    }
  }
}

```



结果：

```json
{
  "acknowledged": true,
  "shards_acknowledged": true,
  "index": "ecommerce"
}
```

#### 创建文档

代码：

```json
POST /ecommerce/_bulk
{"index": {"_id": "1"}}
{"order_id": "6945", "order_date": "2023-03-06", "customer_id": "C242", "customer_name": "Bob Smith", "customer_gender": "female", "customer_age": 60, "customer_city": "Philadelphia", "product_id": "P041", "product_name": "Pro Accessory", "product_category": "Sports", "quantity": 4, "price": 205.89, "total_amount": 823.56, "payment_method": "Credit Card", "is_returned": false}
{"index": {"_id": "2"}}
{"order_id": "5629", "order_date": "2023-11-14", "customer_id": "C262", "customer_name": "Frank Garcia", "customer_gender": "female", "customer_age": 52, "customer_city": "Dallas", "product_id": "P097", "product_name": "Ultra Accessory", "product_category": "Sports", "quantity": 3, "price": 475.18, "total_amount": 1425.54, "payment_method": "Debit Card", "is_returned": true}
{"index": {"_id": "3"}}
{"order_id": "4488", "order_date": "2023-03-28", "customer_id": "C342", "customer_name": "Frank Garcia", "customer_gender": "female", "customer_age": 62, "customer_city": "New York", "product_id": "P001", "product_name": "Ultra Device", "product_category": "Books", "quantity": 5, "price": 722.89, "total_amount": 3614.45, "payment_method": "Credit Card", "is_returned": false}
{"index": {"_id": "4"}}
{"order_id": "7960", "order_date": "2023-09-23", "customer_id": "C408", "customer_name": "Eva Johnson", "customer_gender": "male", "customer_age": 37, "customer_city": "San Antonio", "product_id": "P079", "product_name": "Ultra Accessory", "product_category": "Home Appliances", "quantity": 2, "price": 485.3, "total_amount": 970.6, "payment_method": "Cash on Delivery", "is_returned": false}
{"index": {"_id": "5"}}
{"order_id": "2111", "order_date": "2023-02-18", "customer_id": "C981", "customer_name": "Eva Garcia", "customer_gender": "female", "customer_age": 19, "customer_city": "Los Angeles", "product_id": "P091", "product_name": "Mega Widget", "product_category": "Books", "quantity": 5, "price": 61.52, "total_amount": 307.6, "payment_method": "Debit Card", "is_returned": false}
{"index": {"_id": "6"}}
{"order_id": "2121", "order_date": "2023-01-10", "customer_id": "C030", "customer_name": "Jack Davis", "customer_gender": "male", "customer_age": 18, "customer_city": "Chicago", "product_id": "P070", "product_name": "Ultra Tool", "product_category": "Home Appliances", "quantity": 3, "price": 860.58, "total_amount": 2581.74, "payment_method": "Debit Card", "is_returned": true}
{"index": {"_id": "7"}}
{"order_id": "1211", "order_date": "2023-07-06", "customer_id": "C672", "customer_name": "Jack Williams", "customer_gender": "female", "customer_age": 64, "customer_city": "San Diego", "product_id": "P092", "product_name": "Mega Device", "product_category": "Electronics", "quantity": 4, "price": 389.1, "total_amount": 1556.4, "payment_method": "Cash on Delivery", "is_returned": true}
{"index": {"_id": "8"}}
{"order_id": "7394", "order_date": "2023-08-15", "customer_id": "C719", "customer_name": "Frank Rodriguez", "customer_gender": "female", "customer_age": 59, "customer_city": "San Diego", "product_id": "P083", "product_name": "Super Device", "product_category": "Beauty", "quantity": 2, "price": 593.15, "total_amount": 1186.3, "payment_method": "Credit Card", "is_returned": true}
{"index": {"_id": "9"}}
{"order_id": "7593", "order_date": "2023-07-22", "customer_id": "C145", "customer_name": "Bob Johnson", "customer_gender": "female", "customer_age": 33, "customer_city": "Dallas", "product_id": "P031", "product_name": "Pro Tool", "product_category": "Furniture", "quantity": 3, "price": 740.32, "total_amount": 2220.96, "payment_method": "PayPal", "is_returned": false}
{"index": {"_id": "10"}}
{"order_id": "3304", "order_date": "2023-03-03", "customer_id": "C111", "customer_name": "Jack Jones", "customer_gender": "female", "customer_age": 53, "customer_city": "Chicago", "product_id": "P027", "product_name": "Elite Device", "product_category": "Jewelry", "quantity": 1, "price": 489.24, "total_amount": 489.24, "payment_method": "Credit Card", "is_returned": true}
{"index": {"_id": "11"}}
{"order_id": "7928", "order_date": "2023-05-02", "customer_id": "C441", "customer_name": "Henry Williams", "customer_gender": "male", "customer_age": 57, "customer_city": "San Diego", "product_id": "P005", "product_name": "Elite Accessory", "product_category": "Groceries", "quantity": 3, "price": 989.17, "total_amount": 2967.51, "payment_method": "Debit Card", "is_returned": true}
{"index": {"_id": "12"}}
{"order_id": "9071", "order_date": "2023-04-01", "customer_id": "C798", "customer_name": "Grace Williams", "customer_gender": "male", "customer_age": 66, "customer_city": "Phoenix", "product_id": "P055", "product_name": "Ultra Widget", "product_category": "Furniture", "quantity": 4, "price": 332.32, "total_amount": 1329.28, "payment_method": "Cash on Delivery", "is_returned": false}
{"index": {"_id": "13"}}
{"order_id": "4022", "order_date": "2023-08-04", "customer_id": "C763", "customer_name": "Jack Martinez", "customer_gender": "female", "customer_age": 23, "customer_city": "Phoenix", "product_id": "P026", "product_name": "Ultra Widget", "product_category": "Books", "quantity": 5, "price": 826.39, "total_amount": 4131.95, "payment_method": "Debit Card", "is_returned": false}
{"index": {"_id": "14"}}
{"order_id": "1363", "order_date": "2023-04-28", "customer_id": "C727", "customer_name": "Frank Davis", "customer_gender": "female", "customer_age": 67, "customer_city": "New York", "product_id": "P012", "product_name": "Super Widget", "product_category": "Toys", "quantity": 3, "price": 248.55, "total_amount": 745.65, "payment_method": "Credit Card", "is_returned": false}
{"index": {"_id": "15"}}
{"order_id": "6142", "order_date": "2023-04-27", "customer_id": "C005", "customer_name": "Charlie Davis", "customer_gender": "male", "customer_age": 28, "customer_city": "San Jose", "product_id": "P048", "product_name": "Ultra Gadget", "product_category": "Home Appliances", "quantity": 4, "price": 510.43, "total_amount": 2041.72, "payment_method": "PayPal", "is_returned": true}
{"index": {"_id": "16"}}
{"order_id": "2769", "order_date": "2023-01-03", "customer_id": "C393", "customer_name": "Alice Rodriguez", "customer_gender": "male", "customer_age": 19, "customer_city": "San Jose", "product_id": "P093", "product_name": "Ultra Accessory", "product_category": "Jewelry", "quantity": 3, "price": 317.89, "total_amount": 953.67, "payment_method": "Cash on Delivery", "is_returned": true}
{"index": {"_id": "17"}}
{"order_id": "3810", "order_date": "2023-07-30", "customer_id": "C808", "customer_name": "Ivy Garcia", "customer_gender": "male", "customer_age": 52, "customer_city": "Philadelphia", "product_id": "P013", "product_name": "Elite Gadget", "product_category": "Electronics", "quantity": 3, "price": 541.31, "total_amount": 1623.93, "payment_method": "Debit Card", "is_returned": false}
{"index": {"_id": "18"}}
{"order_id": "4772", "order_date": "2023-04-12", "customer_id": "C043", "customer_name": "David Garcia", "customer_gender": "male", "customer_age": 18, "customer_city": "Los Angeles", "product_id": "P009", "product_name": "Elite Gadget", "product_category": "Beauty", "quantity": 3, "price": 953.55, "total_amount": 2860.65, "payment_method": "Credit Card", "is_returned": false}
{"index": {"_id": "19"}}
{"order_id": "9792", "order_date": "2023-02-25", "customer_id": "C974", "customer_name": "Henry Miller", "customer_gender": "male", "customer_age": 37, "customer_city": "Los Angeles", "product_id": "P012", "product_name": "Ultra Device", "product_category": "Beauty", "quantity": 1, "price": 898.19, "total_amount": 898.19, "payment_method": "Debit Card", "is_returned": false}
{"index": {"_id": "20"}}
{"order_id": "8033", "order_date": "2023-02-17", "customer_id": "C280", "customer_name": "Bob Rodriguez", "customer_gender": "female", "customer_age": 63, "customer_city": "New York", "product_id": "P084", "product_name": "Ultra Device", "product_category": "Electronics", "quantity": 1, "price": 847.14, "total_amount": 847.14, "payment_method": "Debit Card", "is_returned": false}
{"index": {"_id": "21"}}
{"order_id": "4105", "order_date": "2023-11-30", "customer_id": "C605", "customer_name": "Henry Martinez", "customer_gender": "male", "customer_age": 24, "customer_city": "San Antonio", "product_id": "P003", "product_name": "Elite Widget", "product_category": "Groceries", "quantity": 3, "price": 871.75, "total_amount": 2615.25, "payment_method": "Cash on Delivery", "is_returned": false}
{"index": {"_id": "22"}}
{"order_id": "5339", "order_date": "2023-05-25", "customer_id": "C323", "customer_name": "David Johnson", "customer_gender": "female", "customer_age": 41, "customer_city": "Dallas", "product_id": "P054", "product_name": "Elite Gadget", "product_category": "Fashion", "quantity": 4, "price": 501.88, "total_amount": 2007.52, "payment_method": "Credit Card", "is_returned": false}
{"index": {"_id": "23"}}
{"order_id": "2291", "order_date": "2023-02-22", "customer_id": "C180", "customer_name": "Bob Smith", "customer_gender": "male", "customer_age": 53, "customer_city": "Chicago", "product_id": "P065", "product_name": "Ultra Tool", "product_category": "Groceries", "quantity": 5, "price": 229.9, "total_amount": 1149.5, "payment_method": "PayPal", "is_returned": true}
{"index": {"_id": "24"}}
{"order_id": "6857", "order_date": "2023-03-15", "customer_id": "C552", "customer_name": "Alice Smith", "customer_gender": "male", "customer_age": 21, "customer_city": "San Diego", "product_id": "P099", "product_name": "Elite Widget", "product_category": "Jewelry", "quantity": 2, "price": 621.51, "total_amount": 1243.02, "payment_method": "Credit Card", "is_returned": false}
{"index": {"_id": "25"}}
{"order_id": "9291", "order_date": "2023-07-12", "customer_id": "C424", "customer_name": "Grace Garcia", "customer_gender": "male", "customer_age": 63, "customer_city": "San Diego", "product_id": "P062", "product_name": "Super Device", "product_category": "Electronics", "quantity": 1, "price": 302.94, "total_amount": 302.94, "payment_method": "Debit Card", "is_returned": true}
{"index": {"_id": "26"}}
{"order_id": "1897", "order_date": "2023-01-20", "customer_id": "C056", "customer_name": "Grace Martinez", "customer_gender": "female", "customer_age": 40, "customer_city": "San Diego", "product_id": "P018", "product_name": "Mega Tool", "product_category": "Sports", "quantity": 5, "price": 635.06, "total_amount": 3175.3, "payment_method": "PayPal", "is_returned": false}
{"index": {"_id": "27"}}
{"order_id": "5953", "order_date": "2023-04-14", "customer_id": "C363", "customer_name": "Henry Brown", "customer_gender": "female", "customer_age": 70, "customer_city": "New York", "product_id": "P024", "product_name": "Pro Widget", "product_category": "Home Appliances", "quantity": 2, "price": 882.01, "total_amount": 1764.02, "payment_method": "Cash on Delivery", "is_returned": true}
{"index": {"_id": "28"}}
{"order_id": "3156", "order_date": "2023-03-12", "customer_id": "C878", "customer_name": "Henry Martinez", "customer_gender": "male", "customer_age": 22, "customer_city": "Houston", "product_id": "P087", "product_name": "Mega Accessory", "product_category": "Jewelry", "quantity": 1, "price": 535.02, "total_amount": 535.02, "payment_method": "Debit Card", "is_returned": false}
{"index": {"_id": "29"}}
{"order_id": "6906", "order_date": "2023-12-20", "customer_id": "C071", "customer_name": "Bob Martinez", "customer_gender": "male", "customer_age": 69, "customer_city": "Philadelphia", "product_id": "P034", "product_name": "Mega Accessory", "product_category": "Home Appliances", "quantity": 2, "price": 639.65, "total_amount": 1279.3, "payment_method": "PayPal", "is_returned": false}
{"index": {"_id": "30"}}
{"order_id": "8191", "order_date": "2023-08-02", "customer_id": "C458", "customer_name": "Ivy Smith", "customer_gender": "female", "customer_age": 42, "customer_city": "Dallas", "product_id": "P016", "product_name": "Elite Tool", "product_category": "Books", "quantity": 2, "price": 366.21, "total_amount": 732.42, "payment_method": "PayPal", "is_returned": false}
{"index": {"_id": "31"}}
{"order_id": "3182", "order_date": "2023-10-12", "customer_id": "C207", "customer_name": "Bob Garcia", "customer_gender": "female", "customer_age": 33, "customer_city": "San Diego", "product_id": "P047", "product_name": "Ultra Widget", "product_category": "Jewelry", "quantity": 3, "price": 833.95, "total_amount": 2501.85, "payment_method": "Debit Card", "is_returned": true}
{"index": {"_id": "32"}}
{"order_id": "4709", "order_date": "2023-05-21", "customer_id": "C451", "customer_name": "Grace Martinez", "customer_gender": "male", "customer_age": 40, "customer_city": "Chicago", "product_id": "P021", "product_name": "Mega Device", "product_category": "Groceries", "quantity": 3, "price": 106.01, "total_amount": 318.03, "payment_method": "PayPal", "is_returned": true}
{"index": {"_id": "33"}}
{"order_id": "1550", "order_date": "2023-02-25", "customer_id": "C029", "customer_name": "Frank Smith", "customer_gender": "female", "customer_age": 44, "customer_city": "San Jose", "product_id": "P037", "product_name": "Elite Tool", "product_category": "Groceries", "quantity": 4, "price": 743.62, "total_amount": 2974.48, "payment_method": "Debit Card", "is_returned": false}
{"index": {"_id": "34"}}
{"order_id": "5735", "order_date": "2023-08-19", "customer_id": "C765", "customer_name": "David Davis", "customer_gender": "female", "customer_age": 42, "customer_city": "Philadelphia", "product_id": "P037", "product_name": "Super Accessory", "product_category": "Groceries", "quantity": 3, "price": 724.34, "total_amount": 2173.02, "payment_method": "Credit Card", "is_returned": true}
{"index": {"_id": "35"}}
{"order_id": "2189", "order_date": "2023-01-20", "customer_id": "C988", "customer_name": "Charlie Brown", "customer_gender": "male", "customer_age": 24, "customer_city": "San Diego", "product_id": "P004", "product_name": "Ultra Widget", "product_category": "Jewelry", "quantity": 1, "price": 206.12, "total_amount": 206.12, "payment_method": "Credit Card", "is_returned": true}
{"index": {"_id": "36"}}
{"order_id": "4261", "order_date": "2023-04-16", "customer_id": "C199", "customer_name": "Grace Brown", "customer_gender": "female", "customer_age": 30, "customer_city": "New York", "product_id": "P027", "product_name": "Mega Tool", "product_category": "Furniture", "quantity": 1, "price": 291.97, "total_amount": 291.97, "payment_method": "Cash on Delivery", "is_returned": false}
{"index": {"_id": "37"}}
{"order_id": "3166", "order_date": "2023-03-21", "customer_id": "C949", "customer_name": "Ivy Rodriguez", "customer_gender": "female", "customer_age": 24, "customer_city": "Los Angeles", "product_id": "P008", "product_name": "Mega Widget", "product_category": "Books", "quantity": 1, "price": 600.69, "total_amount": 600.69, "payment_method": "PayPal", "is_returned": false}
{"index": {"_id": "38"}}
{"order_id": "7539", "order_date": "2023-08-15", "customer_id": "C581", "customer_name": "Alice Williams", "customer_gender": "male", "customer_age": 69, "customer_city": "San Antonio", "product_id": "P010", "product_name": "Super Tool", "product_category": "Home Appliances", "quantity": 4, "price": 584.18, "total_amount": 2336.72, "payment_method": "Credit Card", "is_returned": false}
{"index": {"_id": "39"}}
{"order_id": "4588", "order_date": "2023-02-23", "customer_id": "C131", "customer_name": "Alice Smith", "customer_gender": "female", "customer_age": 62, "customer_city": "San Diego", "product_id": "P057", "product_name": "Elite Accessory", "product_category": "Electronics", "quantity": 5, "price": 553.25, "total_amount": 2766.25, "payment_method": "PayPal", "is_returned": true}
{"index": {"_id": "40"}}
{"order_id": "4199", "order_date": "2023-11-10", "customer_id": "C494", "customer_name": "Charlie Jones", "customer_gender": "male", "customer_age": 45, "customer_city": "Houston", "product_id": "P020", "product_name": "Elite Tool", "product_category": "Beauty", "quantity": 1, "price": 342.63, "total_amount": 342.63, "payment_method": "Cash on Delivery", "is_returned": true}
{"index": {"_id": "41"}}
{"order_id": "9196", "order_date": "2023-02-06", "customer_id": "C534", "customer_name": "Alice Garcia", "customer_gender": "female", "customer_age": 32, "customer_city": "Phoenix", "product_id": "P020", "product_name": "Mega Device", "product_category": "Furniture", "quantity": 1, "price": 635.12, "total_amount": 635.12, "payment_method": "PayPal", "is_returned": false}
{"index": {"_id": "42"}}
{"order_id": "1941", "order_date": "2023-06-23", "customer_id": "C475", "customer_name": "Bob Rodriguez", "customer_gender": "female", "customer_age": 48, "customer_city": "Chicago", "product_id": "P090", "product_name": "Ultra Device", "product_category": "Electronics", "quantity": 4, "price": 215.28, "total_amount": 861.12, "payment_method": "Cash on Delivery", "is_returned": false}
{"index": {"_id": "43"}}
{"order_id": "9818", "order_date": "2023-05-27", "customer_id": "C532", "customer_name": "Bob Johnson", "customer_gender": "male", "customer_age": 47, "customer_city": "Phoenix", "product_id": "P084", "product_name": "Super Tool", "product_category": "Home Appliances", "quantity": 3, "price": 827.32, "total_amount": 2481.96, "payment_method": "PayPal", "is_returned": false}
{"index": {"_id": "44"}}
{"order_id": "1809", "order_date": "2023-07-13", "customer_id": "C317", "customer_name": "Charlie Miller", "customer_gender": "male", "customer_age": 36, "customer_city": "San Jose", "product_id": "P021", "product_name": "Pro Gadget", "product_category": "Jewelry", "quantity": 3, "price": 121.06, "total_amount": 363.18, "payment_method": "Credit Card", "is_returned": true}
{"index": {"_id": "45"}}
{"order_id": "1361", "order_date": "2023-01-29", "customer_id": "C138", "customer_name": "Charlie Davis", "customer_gender": "female", "customer_age": 50, "customer_city": "San Diego", "product_id": "P078", "product_name": "Ultra Accessory", "product_category": "Toys", "quantity": 2, "price": 948.74, "total_amount": 1897.48, "payment_method": "Cash on Delivery", "is_returned": false}
{"index": {"_id": "46"}}
{"order_id": "7019", "order_date": "2023-03-01", "customer_id": "C165", "customer_name": "Grace Martinez", "customer_gender": "male", "customer_age": 18, "customer_city": "Phoenix", "product_id": "P005", "product_name": "Elite Accessory", "product_category": "Furniture", "quantity": 5, "price": 963.32, "total_amount": 4816.6, "payment_method": "Debit Card", "is_returned": true}
{"index": {"_id": "47"}}
{"order_id": "2564", "order_date": "2023-12-31", "customer_id": "C983", "customer_name": "Grace Martinez", "customer_gender": "male", "customer_age": 34, "customer_city": "San Diego", "product_id": "P067", "product_name": "Super Gadget", "product_category": "Home Appliances", "quantity": 5, "price": 919.89, "total_amount": 4599.45, "payment_method": "Cash on Delivery", "is_returned": false}
{"index": {"_id": "48"}}
{"order_id": "8300", "order_date": "2023-08-19", "customer_id": "C556", "customer_name": "Jack Martinez", "customer_gender": "female", "customer_age": 25, "customer_city": "Philadelphia", "product_id": "P069", "product_name": "Pro Tool", "product_category": "Toys", "quantity": 2, "price": 345.59, "total_amount": 691.18, "payment_method": "Credit Card", "is_returned": false}
{"index": {"_id": "49"}}
{"order_id": "1156", "order_date": "2023-06-15", "customer_id": "C438", "customer_name": "David Davis", "customer_gender": "male", "customer_age": 59, "customer_city": "Houston", "product_id": "P020", "product_name": "Mega Tool", "product_category": "Beauty", "quantity": 2, "price": 687.43, "total_amount": 1374.86, "payment_method": "Cash on Delivery", "is_returned": true}
{"index": {"_id": "50"}}
{"order_id": "2897", "order_date": "2023-03-13", "customer_id": "C320", "customer_name": "Charlie Rodriguez", "customer_gender": "male", "customer_age": 68, "customer_city": "San Jose", "product_id": "P060", "product_name": "Pro Tool", "product_category": "Jewelry", "quantity": 5, "price": 918.59, "total_amount": 4592.95, "payment_method": "Credit Card", "is_returned": false}
{"index": {"_id": "51"}}
{"order_id": "5636", "order_date": "2023-05-12", "customer_id": "C643", "customer_name": "Jack Martinez", "customer_gender": "male", "customer_age": 33, "customer_city": "San Jose", "product_id": "P073", "product_name": "Ultra Tool", "product_category": "Fashion", "quantity": 4, "price": 141.53, "total_amount": 566.12, "payment_method": "Credit Card", "is_returned": false}
{"index": {"_id": "52"}}
{"order_id": "3983", "order_date": "2023-08-05", "customer_id": "C973", "customer_name": "Frank Johnson", "customer_gender": "female", "customer_age": 44, "customer_city": "San Jose", "product_id": "P084", "product_name": "Super Widget", "product_category": "Furniture", "quantity": 5, "price": 171.0, "total_amount": 855.0, "payment_method": "Cash on Delivery", "is_returned": false}
{"index": {"_id": "53"}}
{"order_id": "5290", "order_date": "2023-01-02", "customer_id": "C369", "customer_name": "Jack Williams", "customer_gender": "male", "customer_age": 56, "customer_city": "Los Angeles", "product_id": "P095", "product_name": "Ultra Gadget", "product_category": "Jewelry", "quantity": 2, "price": 634.53, "total_amount": 1269.06, "payment_method": "Cash on Delivery", "is_returned": false}
{"index": {"_id": "54"}}
{"order_id": "8195", "order_date": "2023-11-25", "customer_id": "C782", "customer_name": "Jack Martinez", "customer_gender": "male", "customer_age": 62, "customer_city": "Phoenix", "product_id": "P040", "product_name": "Ultra Gadget", "product_category": "Books", "quantity": 1, "price": 595.85, "total_amount": 595.85, "payment_method": "PayPal", "is_returned": true}
{"index": {"_id": "55"}}
{"order_id": "6632", "order_date": "2023-03-01", "customer_id": "C596", "customer_name": "Frank Garcia", "customer_gender": "female", "customer_age": 23, "customer_city": "Philadelphia", "product_id": "P080", "product_name": "Super Tool", "product_category": "Fashion", "quantity": 1, "price": 641.04, "total_amount": 641.04, "payment_method": "Credit Card", "is_returned": true}
{"index": {"_id": "56"}}
{"order_id": "1364", "order_date": "2023-12-18", "customer_id": "C701", "customer_name": "Charlie Smith", "customer_gender": "male", "customer_age": 67, "customer_city": "Phoenix", "product_id": "P060", "product_name": "Pro Gadget", "product_category": "Fashion", "quantity": 1, "price": 978.14, "total_amount": 978.14, "payment_method": "Cash on Delivery", "is_returned": false}
{"index": {"_id": "57"}}
{"order_id": "7109", "order_date": "2023-08-25", "customer_id": "C480", "customer_name": "Bob Jones", "customer_gender": "female", "customer_age": 43, "customer_city": "San Antonio", "product_id": "P010", "product_name": "Elite Widget", "product_category": "Fashion", "quantity": 1, "price": 29.28, "total_amount": 29.28, "payment_method": "Debit Card", "is_returned": true}
{"index": {"_id": "58"}}
{"order_id": "5865", "order_date": "2023-04-29", "customer_id": "C658", "customer_name": "Frank Garcia", "customer_gender": "male", "customer_age": 58, "customer_city": "Los Angeles", "product_id": "P004", "product_name": "Super Gadget", "product_category": "Groceries", "quantity": 3, "price": 798.19, "total_amount": 2394.57, "payment_method": "Credit Card", "is_returned": true}
{"index": {"_id": "59"}}
{"order_id": "2212", "order_date": "2023-10-09", "customer_id": "C863", "customer_name": "Henry Garcia", "customer_gender": "female", "customer_age": 50, "customer_city": "Chicago", "product_id": "P085", "product_name": "Mega Gadget", "product_category": "Home Appliances", "quantity": 2, "price": 291.02, "total_amount": 582.04, "payment_method": "Debit Card", "is_returned": false}
{"index": {"_id": "60"}}
{"order_id": "6114", "order_date": "2023-05-21", "customer_id": "C365", "customer_name": "Charlie Garcia", "customer_gender": "male", "customer_age": 36, "customer_city": "Los Angeles", "product_id": "P080", "product_name": "Super Accessory", "product_category": "Jewelry", "quantity": 1, "price": 254.71, "total_amount": 254.71, "payment_method": "PayPal", "is_returned": false}
{"index": {"_id": "61"}}
{"order_id": "2867", "order_date": "2023-10-13", "customer_id": "C854", "customer_name": "Charlie Martinez", "customer_gender": "male", "customer_age": 21, "customer_city": "Houston", "product_id": "P031", "product_name": "Pro Widget", "product_category": "Beauty", "quantity": 4, "price": 679.3, "total_amount": 2717.2, "payment_method": "Debit Card", "is_returned": false}
{"index": {"_id": "62"}}
{"order_id": "5073", "order_date": "2023-12-25", "customer_id": "C285", "customer_name": "Bob Smith", "customer_gender": "male", "customer_age": 29, "customer_city": "San Diego", "product_id": "P078", "product_name": "Elite Accessory", "product_category": "Home Appliances", "quantity": 3, "price": 473.43, "total_amount": 1420.29, "payment_method": "Cash on Delivery", "is_returned": false}
{"index": {"_id": "63"}}
{"order_id": "9574", "order_date": "2023-06-26", "customer_id": "C023", "customer_name": "Charlie Garcia", "customer_gender": "female", "customer_age": 40, "customer_city": "San Jose", "product_id": "P076", "product_name": "Mega Widget", "product_category": "Furniture", "quantity": 2, "price": 115.01, "total_amount": 230.02, "payment_method": "Cash on Delivery", "is_returned": true}
{"index": {"_id": "64"}}
{"order_id": "5086", "order_date": "2023-01-28", "customer_id": "C965", "customer_name": "Jack Jones", "customer_gender": "female", "customer_age": 33, "customer_city": "Philadelphia", "product_id": "P036", "product_name": "Ultra Device", "product_category": "Fashion", "quantity": 4, "price": 326.84, "total_amount": 1307.36, "payment_method": "PayPal", "is_returned": false}
{"index": {"_id": "65"}}
{"order_id": "2320", "order_date": "2023-11-17", "customer_id": "C789", "customer_name": "David Smith", "customer_gender": "male", "customer_age": 21, "customer_city": "Chicago", "product_id": "P041", "product_name": "Elite Accessory", "product_category": "Sports", "quantity": 5, "price": 498.44, "total_amount": 2492.2, "payment_method": "PayPal", "is_returned": true}
{"index": {"_id": "66"}}
{"order_id": "2463", "order_date": "2023-12-07", "customer_id": "C038", "customer_name": "Jack Jones", "customer_gender": "male", "customer_age": 66, "customer_city": "Dallas", "product_id": "P017", "product_name": "Super Device", "product_category": "Beauty", "quantity": 3, "price": 499.37, "total_amount": 1498.11, "payment_method": "Credit Card", "is_returned": true}
{"index": {"_id": "67"}}
{"order_id": "9420", "order_date": "2023-06-25", "customer_id": "C336", "customer_name": "Grace Brown", "customer_gender": "female", "customer_age": 24, "customer_city": "Los Angeles", "product_id": "P035", "product_name": "Elite Device", "product_category": "Fashion", "quantity": 3, "price": 70.95, "total_amount": 212.85, "payment_method": "Cash on Delivery", "is_returned": true}
{"index": {"_id": "68"}}
{"order_id": "1322", "order_date": "2023-01-27", "customer_id": "C457", "customer_name": "Jack Garcia", "customer_gender": "male", "customer_age": 19, "customer_city": "Los Angeles", "product_id": "P076", "product_name": "Ultra Tool", "product_category": "Beauty", "quantity": 5, "price": 742.85, "total_amount": 3714.25, "payment_method": "Debit Card", "is_returned": true}
{"index": {"_id": "69"}}
{"order_id": "5018", "order_date": "2023-07-10", "customer_id": "C409", "customer_name": "Frank Martinez", "customer_gender": "male", "customer_age": 47, "customer_city": "San Jose", "product_id": "P021", "product_name": "Ultra Gadget", "product_category": "Fashion", "quantity": 5, "price": 266.16, "total_amount": 1330.8, "payment_method": "Debit Card", "is_returned": false}
{"index": {"_id": "70"}}
{"order_id": "7009", "order_date": "2023-05-10", "customer_id": "C923", "customer_name": "Jack Williams", "customer_gender": "male", "customer_age": 62, "customer_city": "Dallas", "product_id": "P001", "product_name": "Ultra Tool", "product_category": "Books", "quantity": 1, "price": 811.29, "total_amount": 811.29, "payment_method": "Debit Card", "is_returned": false}
{"index": {"_id": "71"}}
{"order_id": "5884", "order_date": "2023-05-29", "customer_id": "C528", "customer_name": "Charlie Johnson", "customer_gender": "female", "customer_age": 18, "customer_city": "Philadelphia", "product_id": "P031", "product_name": "Elite Gadget", "product_category": "Books", "quantity": 1, "price": 379.51, "total_amount": 379.51, "payment_method": "Cash on Delivery", "is_returned": false}
{"index": {"_id": "72"}}
{"order_id": "6635", "order_date": "2023-01-15", "customer_id": "C575", "customer_name": "Frank Brown", "customer_gender": "male", "customer_age": 51, "customer_city": "New York", "product_id": "P089", "product_name": "Elite Accessory", "product_category": "Electronics", "quantity": 4, "price": 781.98, "total_amount": 3127.92, "payment_method": "Cash on Delivery", "is_returned": true}
{"index": {"_id": "73"}}
{"order_id": "9082", "order_date": "2023-10-10", "customer_id": "C116", "customer_name": "Ivy Brown", "customer_gender": "male", "customer_age": 37, "customer_city": "Houston", "product_id": "P008", "product_name": "Elite Widget", "product_category": "Sports", "quantity": 4, "price": 111.95, "total_amount": 447.8, "payment_method": "Debit Card", "is_returned": false}
{"index": {"_id": "74"}}
{"order_id": "8042", "order_date": "2023-11-25", "customer_id": "C041", "customer_name": "Bob Davis", "customer_gender": "female", "customer_age": 67, "customer_city": "Chicago", "product_id": "P062", "product_name": "Ultra Accessory", "product_category": "Books", "quantity": 2, "price": 352.53, "total_amount": 705.06, "payment_method": "Credit Card", "is_returned": true}
{"index": {"_id": "75"}}
{"order_id": "4047", "order_date": "2023-01-19", "customer_id": "C200", "customer_name": "Ivy Miller", "customer_gender": "female", "customer_age": 27, "customer_city": "San Diego", "product_id": "P056", "product_name": "Pro Device", "product_category": "Electronics", "quantity": 2, "price": 298.64, "total_amount": 597.28, "payment_method": "Debit Card", "is_returned": true}
{"index": {"_id": "76"}}
{"order_id": "4072", "order_date": "2023-12-01", "customer_id": "C126", "customer_name": "Grace Williams", "customer_gender": "female", "customer_age": 31, "customer_city": "Houston", "product_id": "P083", "product_name": "Pro Gadget", "product_category": "Groceries", "quantity": 2, "price": 480.09, "total_amount": 960.18, "payment_method": "Debit Card", "is_returned": false}
{"index": {"_id": "77"}}
{"order_id": "7911", "order_date": "2023-12-31", "customer_id": "C503", "customer_name": "Bob Jones", "customer_gender": "female", "customer_age": 21, "customer_city": "Los Angeles", "product_id": "P005", "product_name": "Elite Gadget", "product_category": "Electronics", "quantity": 3, "price": 192.63, "total_amount": 577.89, "payment_method": "Debit Card", "is_returned": true}
{"index": {"_id": "78"}}
{"order_id": "3590", "order_date": "2023-06-30", "customer_id": "C003", "customer_name": "Henry Smith", "customer_gender": "male", "customer_age": 23, "customer_city": "San Jose", "product_id": "P099", "product_name": "Pro Widget", "product_category": "Home Appliances", "quantity": 4, "price": 46.63, "total_amount": 186.52, "payment_method": "Cash on Delivery", "is_returned": true}
{"index": {"_id": "79"}}
{"order_id": "4306", "order_date": "2023-07-09", "customer_id": "C589", "customer_name": "Bob Rodriguez", "customer_gender": "male", "customer_age": 42, "customer_city": "Philadelphia", "product_id": "P051", "product_name": "Ultra Accessory", "product_category": "Beauty", "quantity": 3, "price": 570.6, "total_amount": 1711.8, "payment_method": "Cash on Delivery", "is_returned": true}
{"index": {"_id": "80"}}
{"order_id": "4255", "order_date": "2023-02-02", "customer_id": "C003", "customer_name": "Henry Miller", "customer_gender": "male", "customer_age": 66, "customer_city": "San Antonio", "product_id": "P076", "product_name": "Ultra Accessory", "product_category": "Toys", "quantity": 4, "price": 901.45, "total_amount": 3605.8, "payment_method": "Debit Card", "is_returned": false}
{"index": {"_id": "81"}}
{"order_id": "1162", "order_date": "2023-12-25", "customer_id": "C018", "customer_name": "Grace Smith", "customer_gender": "female", "customer_age": 63, "customer_city": "New York", "product_id": "P067", "product_name": "Ultra Device", "product_category": "Groceries", "quantity": 4, "price": 155.11, "total_amount": 620.44, "payment_method": "Cash on Delivery", "is_returned": false}
{"index": {"_id": "82"}}
{"order_id": "9635", "order_date": "2023-05-12", "customer_id": "C237", "customer_name": "Henry Johnson", "customer_gender": "female", "customer_age": 61, "customer_city": "New York", "product_id": "P079", "product_name": "Ultra Widget", "product_category": "Beauty", "quantity": 2, "price": 773.0, "total_amount": 1546.0, "payment_method": "Debit Card", "is_returned": true}
{"index": {"_id": "83"}}
{"order_id": "6795", "order_date": "2023-06-14", "customer_id": "C336", "customer_name": "Ivy Jones", "customer_gender": "male", "customer_age": 54, "customer_city": "Philadelphia", "product_id": "P010", "product_name": "Mega Device", "product_category": "Electronics", "quantity": 4, "price": 394.85, "total_amount": 1579.4, "payment_method": "Cash on Delivery", "is_returned": false}
{"index": {"_id": "84"}}
{"order_id": "6288", "order_date": "2023-09-22", "customer_id": "C327", "customer_name": "Jack Smith", "customer_gender": "male", "customer_age": 68, "customer_city": "Houston", "product_id": "P072", "product_name": "Ultra Tool", "product_category": "Home Appliances", "quantity": 3, "price": 919.6, "total_amount": 2758.8, "payment_method": "Credit Card", "is_returned": false}
{"index": {"_id": "85"}}
{"order_id": "5127", "order_date": "2023-01-13", "customer_id": "C243", "customer_name": "David Williams", "customer_gender": "female", "customer_age": 69, "customer_city": "Philadelphia", "product_id": "P043", "product_name": "Elite Device", "product_category": "Home Appliances", "quantity": 3, "price": 954.01, "total_amount": 2862.03, "payment_method": "PayPal", "is_returned": false}
{"index": {"_id": "86"}}
{"order_id": "2322", "order_date": "2023-10-03", "customer_id": "C543", "customer_name": "Frank Miller", "customer_gender": "male", "customer_age": 49, "customer_city": "Los Angeles", "product_id": "P099", "product_name": "Super Widget", "product_category": "Beauty", "quantity": 5, "price": 971.79, "total_amount": 4858.95, "payment_method": "PayPal", "is_returned": false}
{"index": {"_id": "87"}}
{"order_id": "9268", "order_date": "2023-01-07", "customer_id": "C136", "customer_name": "Henry Rodriguez", "customer_gender": "male", "customer_age": 63, "customer_city": "Houston", "product_id": "P046", "product_name": "Super Widget", "product_category": "Furniture", "quantity": 3, "price": 896.47, "total_amount": 2689.41, "payment_method": "Debit Card", "is_returned": false}
{"index": {"_id": "88"}}
{"order_id": "7091", "order_date": "2023-05-04", "customer_id": "C551", "customer_name": "Grace Brown", "customer_gender": "male", "customer_age": 61, "customer_city": "Chicago", "product_id": "P016", "product_name": "Ultra Gadget", "product_category": "Toys", "quantity": 4, "price": 95.13, "total_amount": 380.52, "payment_method": "Debit Card", "is_returned": false}
{"index": {"_id": "89"}}
{"order_id": "2123", "order_date": "2023-10-05", "customer_id": "C483", "customer_name": "Henry Rodriguez", "customer_gender": "male", "customer_age": 55, "customer_city": "San Antonio", "product_id": "P006", "product_name": "Mega Device", "product_category": "Jewelry", "quantity": 2, "price": 499.08, "total_amount": 998.16, "payment_method": "PayPal", "is_returned": true}
{"index": {"_id": "90"}}
{"order_id": "8013", "order_date": "2023-04-29", "customer_id": "C431", "customer_name": "Eva Rodriguez", "customer_gender": "male", "customer_age": 65, "customer_city": "Phoenix", "product_id": "P046", "product_name": "Elite Device", "product_category": "Furniture", "quantity": 4, "price": 889.3, "total_amount": 3557.2, "payment_method": "Credit Card", "is_returned": false}
{"index": {"_id": "91"}}
{"order_id": "1329", "order_date": "2023-02-27", "customer_id": "C198", "customer_name": "David Johnson", "customer_gender": "male", "customer_age": 64, "customer_city": "Dallas", "product_id": "P001", "product_name": "Pro Tool", "product_category": "Electronics", "quantity": 1, "price": 734.57, "total_amount": 734.57, "payment_method": "Cash on Delivery", "is_returned": true}
{"index": {"_id": "92"}}
{"order_id": "1937", "order_date": "2023-06-27", "customer_id": "C928", "customer_name": "Jack Garcia", "customer_gender": "female", "customer_age": 27, "customer_city": "Los Angeles", "product_id": "P002", "product_name": "Ultra Accessory", "product_category": "Toys", "quantity": 3, "price": 241.4, "total_amount": 724.2, "payment_method": "Cash on Delivery", "is_returned": false}
{"index": {"_id": "93"}}
{"order_id": "1472", "order_date": "2023-03-17", "customer_id": "C365", "customer_name": "Frank Miller", "customer_gender": "male", "customer_age": 40, "customer_city": "Los Angeles", "product_id": "P077", "product_name": "Elite Tool", "product_category": "Furniture", "quantity": 4, "price": 150.69, "total_amount": 602.76, "payment_method": "Debit Card", "is_returned": false}
{"index": {"_id": "94"}}
{"order_id": "5187", "order_date": "2023-06-07", "customer_id": "C725", "customer_name": "Ivy Martinez", "customer_gender": "female", "customer_age": 49, "customer_city": "Dallas", "product_id": "P025", "product_name": "Super Accessory", "product_category": "Jewelry", "quantity": 1, "price": 904.01, "total_amount": 904.01, "payment_method": "Credit Card", "is_returned": false}
{"index": {"_id": "95"}}
{"order_id": "1737", "order_date": "2023-09-15", "customer_id": "C977", "customer_name": "Ivy Brown", "customer_gender": "female", "customer_age": 40, "customer_city": "New York", "product_id": "P031", "product_name": "Elite Widget", "product_category": "Sports", "quantity": 5, "price": 977.22, "total_amount": 4886.1, "payment_method": "Cash on Delivery", "is_returned": true}
{"index": {"_id": "96"}}
{"order_id": "3054", "order_date": "2023-02-16", "customer_id": "C872", "customer_name": "David Miller", "customer_gender": "female", "customer_age": 52, "customer_city": "San Jose", "product_id": "P077", "product_name": "Ultra Gadget", "product_category": "Jewelry", "quantity": 5, "price": 493.64, "total_amount": 2468.2, "payment_method": "Debit Card", "is_returned": false}
{"index": {"_id": "97"}}
{"order_id": "5606", "order_date": "2023-06-12", "customer_id": "C053", "customer_name": "Charlie Williams", "customer_gender": "female", "customer_age": 55, "customer_city": "San Antonio", "product_id": "P027", "product_name": "Elite Device", "product_category": "Jewelry", "quantity": 1, "price": 698.18, "total_amount": 698.18, "payment_method": "Cash on Delivery", "is_returned": true}
{"index": {"_id": "98"}}
{"order_id": "6873", "order_date": "2023-12-19", "customer_id": "C209", "customer_name": "Henry Davis", "customer_gender": "male", "customer_age": 63, "customer_city": "New York", "product_id": "P012", "product_name": "Super Gadget", "product_category": "Electronics", "quantity": 2, "price": 287.91, "total_amount": 575.82, "payment_method": "Cash on Delivery", "is_returned": true}
{"index": {"_id": "99"}}
{"order_id": "2082", "order_date": "2023-02-27", "customer_id": "C184", "customer_name": "Charlie Rodriguez", "customer_gender": "male", "customer_age": 45, "customer_city": "San Jose", "product_id": "P043", "product_name": "Super Tool", "product_category": "Toys", "quantity": 4, "price": 621.0, "total_amount": 2484.0, "payment_method": "PayPal", "is_returned": false}
{"index": {"_id": "100"}}
{"order_id": "1211", "order_date": "2023-04-11", "customer_id": "C549", "customer_name": "Alice Garcia", "customer_gender": "male", "customer_age": 62, "customer_city": "Houston", "product_id": "P047", "product_name": "Ultra Device", "product_category": "Electronics", "quantity": 5, "price": 758.18, "total_amount": 3790.9, "payment_method": "PayPal", "is_returned": false}

```

结果：

```json
{
  "errors": false,
  "took": 75,
  "items": [
    {
      "index": {
        "_index": "ecommerce",
        "_id": "1",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 0,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "2",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 1,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "3",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 2,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "4",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 3,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "5",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 4,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "6",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 5,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "7",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 6,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "8",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 7,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "9",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 8,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "10",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 9,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "11",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 10,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "12",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 11,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "13",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 12,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "14",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 13,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "15",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 14,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "16",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 15,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "17",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 16,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "18",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 17,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "19",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 18,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "20",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 19,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "21",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 20,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "22",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 21,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "23",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 22,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "24",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 23,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "25",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 24,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "26",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 25,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "27",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 26,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "28",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 27,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "29",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 28,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "30",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 29,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "31",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 30,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "32",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 31,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "33",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 32,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "34",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 33,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "35",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 34,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "36",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 35,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "37",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 36,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "38",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 37,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "39",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 38,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "40",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 39,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "41",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 40,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "42",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 41,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "43",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 42,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "44",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 43,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "45",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 44,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "46",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 45,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "47",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 46,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "48",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 47,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "49",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 48,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "50",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 49,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "51",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 50,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "52",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 51,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "53",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 52,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "54",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 53,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "55",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 54,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "56",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 55,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "57",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 56,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "58",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 57,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "59",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 58,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "60",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 59,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "61",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 60,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "62",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 61,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "63",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 62,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "64",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 63,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "65",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 64,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "66",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 65,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "67",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 66,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "68",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 67,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "69",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 68,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "70",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 69,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "71",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 70,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "72",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 71,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "73",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 72,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "74",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 73,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "75",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 74,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "76",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 75,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "77",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 76,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "78",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 77,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "79",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 78,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "80",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 79,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "81",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 80,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "82",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 81,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "83",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 82,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "84",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 83,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "85",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 84,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "86",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 85,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "87",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 86,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "88",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 87,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "89",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 88,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "90",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 89,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "91",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 90,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "92",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 91,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "93",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 92,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "94",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 93,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "95",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 94,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "96",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 95,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "97",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 96,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "98",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 97,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "99",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 98,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "ecommerce",
        "_id": "100",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 99,
        "_primary_term": 1,
        "status": 201
      }
    }
  ]
}
```

#### 练习题目

##### 统计每个产品类别的总销售额。

代码：

``` json
GET /ecommerce/_search
{
  "aggs":{ //代表聚合名称
    "product_category_group":{ //名称，随意起名
      "terms":{ //分组：对什么分组，由下面的“field”决定
        "field": "product_category" //分组字段
      },
      "aggs":{ //对产品类别分完组之后再进行聚合操作
        "total_amount":{ //表示总销售额
          "sum":{ //对...求和
            "field":"total_amount" //求和字段为total_amount：总金额
          }
        }
      }
    }
    
  },
  "size":0 //表示只要统计结果，其它原始数据不显示
}
```

结果：

``` json
{
  "took": 234,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 100,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "product_category_group": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": "Jewelry",
          "doc_count": 14,
          "total_amount": {
            "value": 17477.37028503418
          }
        },
        {
          "key": "Electronics",
          "doc_count": 13,
          "total_amount": {
            "value": 18941.559997558594
          }
        },
        {
          "key": "Home Appliances",
          "doc_count": 13,
          "total_amount": {
            "value": 25865.190231323242
          }
        },
        {
          "key": "Beauty",
          "doc_count": 11,
          "total_amount": {
            "value": 22708.94012451172
          }
        },
        {
          "key": "Furniture",
          "doc_count": 10,
          "total_amount": {
            "value": 17228.31996154785
          }
        },
        {
          "key": "Books",
          "doc_count": 9,
          "total_amount": {
            "value": 11878.820098876953
          }
        },
        {
          "key": "Groceries",
          "doc_count": 9,
          "total_amount": {
            "value": 16172.980072021484
          }
        },
        {
          "key": "Fashion",
          "doc_count": 8,
          "total_amount": {
            "value": 7073.110048294067
          }
        },
        {
          "key": "Toys",
          "doc_count": 7,
          "total_amount": {
            "value": 10528.830047607422
          }
        },
        {
          "key": "Sports",
          "doc_count": 6,
          "total_amount": {
            "value": 13250.500122070312
          }
        }
      ]
    }
  }
}
```

##### 计算每个城市的平均订单金额。

代码：

``` json
GET /ecommerce/_search
{
  "aggs":{ //代表聚合名称
    "customer_city_group":{ //名称，随意起名
      "terms":{ //分组：对什么分组，由下面的“field”决定
        "field": "customer_city" //分组字段
      },
      "aggs":{ //对产品类别分完组之后再进行聚合操作
        "total_amount_avg":{ //表示总销售额
          "avg":{ //对...求平均
            "field":"total_amount" //求平均字段为total_amount：总金额
          }
        }
      }
    }
  },
  "size":0 //表示只要统计结果，其它原始数据不显示
}
```

结果：

``` json
{
  "took": 6,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 100,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "customer_city_group": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": "Los Angeles",
          "doc_count": 13,
          "total_amount_avg": {
            "value": 1482.7977142333984
          }
        },
        {
          "key": "San Diego",
          "doc_count": 13,
          "total_amount_avg": {
            "value": 1878.4761915940505
          }
        },
        {
          "key": "San Jose",
          "doc_count": 12,
          "total_amount_avg": {
            "value": 1587.2216771443684
          }
        },
        {
          "key": "Philadelphia",
          "doc_count": 11,
          "total_amount_avg": {
            "value": 1370.19365345348
          }
        },
        {
          "key": "New York",
          "doc_count": 10,
          "total_amount_avg": {
            "value": 1801.9510040283203
          }
        },
        {
          "key": "Chicago",
          "doc_count": 9,
          "total_amount_avg": {
            "value": 1062.1610989040798
          }
        },
        {
          "key": "Houston",
          "doc_count": 9,
          "total_amount_avg": {
            "value": 1735.199978298611
          }
        },
        {
          "key": "Dallas",
          "doc_count": 8,
          "total_amount_avg": {
            "value": 1291.8024978637695
          }
        },
        {
          "key": "Phoenix",
          "doc_count": 8,
          "total_amount_avg": {
            "value": 2315.7625274658203
          }
        },
        {
          "key": "San Antonio",
          "doc_count": 7,
          "total_amount_avg": {
            "value": 1607.7128516605922
          }
        }
      ]
    }
  }
}
```



##### 找出销量最高的前5个产品。

代码：

``` json
GET /ecommerce/_search
{
  "aggs":{ //代表聚合名称
    "top5_product_group":{ //名称，随意起名
      "terms":{ //分组：对什么分组，由下面的“field”决定
        "field": "product_id", //分组字段
        "size":5,
        "order": {
          "total_quantity": "desc" //降序分组
        }
      },
      "aggs": { //对前五求总销量
        "total_quantity": {
          "sum": {
            "field":"quantity"
          }
        }
      }
    }
  },
  "size":0 //表示只要统计结果，其它原始数据不显示
```

结果：

``` json
{
  "took": 4,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 100,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "top5_product_group": {
      "doc_count_error_upper_bound": -1,
      "sum_other_doc_count": 84,
      "buckets": [
        {
          "key": "P031",
          "doc_count": 4,
          "total_quantity": {
            "value": 13
          }
        },
        {
          "key": "P005",
          "doc_count": 3,
          "total_quantity": {
            "value": 11
          }
        },
        {
          "key": "P021",
          "doc_count": 3,
          "total_quantity": {
            "value": 11
          }
        },
        {
          "key": "P076",
          "doc_count": 3,
          "total_quantity": {
            "value": 11
          }
        },
        {
          "key": "P099",
          "doc_count": 3,
          "total_quantity": {
            "value": 11
          }
        }
      ]
    }
  }
}
```

##### 计算男性和女性客户的平均年龄。

代码：

``` json
GET /ecommerce/_search
{
  "aggs":{ //代表聚合名称
    "customer_gender_group":{ //名称，随意起名
      "terms":{ //分组：对什么分组，由下面的“field”决定,
        "field": "customer_gender" //分组字段
      },
      "aggs":{ //对性别分完组之后再进行聚合操作
        "avg_customer_age":{ //表示平均年龄
          "avg":{ //对...求平均
            "field":"customer_age" 
          }
        }
      }
    }
    
  },
  "size":0 //表示只要统计结果，其它原始数据不显示
}
```

结果：

``` json
{
  "took": 2,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 100,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "customer_gender_group": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": "male",
          "doc_count": 55,
          "avg_customer_age": {
            "value": 45.61818181818182
          }
        },
        {
          "key": "female",
          "doc_count": 45,
          "avg_customer_age": {
            "value": 43.888888888888886
          }
        }
      ]
    }
  }
}
```

##### 统计每种支付方式的使用次数和总金额。

代码：

``` json
GET /ecommerce/_search
{
  "aggs":{ //代表聚合名称
    "payment_method_group":{ //名称，随意起名
      "terms":{ //分组：对什么分组，由下面的“field”决定,
        "field": "payment_method"  //分组字段
      },
      "aggs":{ //分完组之后再进行聚合操作
        "payment_method_usage":{
          "value_count": {//统计次数
            "field": "payment_method"
          }
        },
        "total_sales":{ //表示总金额
          "sum":{ //对...求和
            "field":"total_amount" 
          }
        }
      }
    }
  },
  "size":0 //表示只要统计结果，其它原始数据不显示
}
```

结果：

``` json
{
  "took": 1,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 100,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "payment_method_group": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": "Cash on Delivery",
          "doc_count": 29,
          "payment_method_usage": {
            "value": 29
          },
          "total_sales": {
            "value": 38746.55044555664
          }
        },
        {
          "key": "Debit Card",
          "doc_count": 29,
          "payment_method_usage": {
            "value": 29
          },
          "total_sales": {
            "value": 48975.19040107727
          }
        },
        {
          "key": "Credit Card",
          "doc_count": 22,
          "payment_method_usage": {
            "value": 22
          },
          "total_sales": {
            "value": 36358.470153808594
          }
        },
        {
          "key": "PayPal",
          "doc_count": 20,
          "payment_method_usage": {
            "value": 20
          },
          "total_sales": {
            "value": 37045.40998840332
          }
        }
      ]
    }
  }
}
```

##### 计算每月的总销售额。

代码：

``` json
GET /ecommerce/_search
{
"aggs": {
    "sales_by_month": {
      "date_histogram": { //表示按月份分组
        "field": "order_date",
        "calendar_interval": "month"
      },
      "aggs": {
        "total_sales": {
          "sum": {
            "field": "total_amount"
          }
        }
      }
    }
  },
  "size": 0 //表示只要统计结果，其它原始数据不显示
}
```

结果：

``` json
{
  "took": 26,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 100,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "sales_by_month": {
      "buckets": [
        {
          "key_as_string": "2023-01-01T00:00:00.000Z",
          "key": 1672531200000,
          "doc_count": 12,
          "total_sales": {
            "value": 24381.61993408203
          }
        },
        {
          "key_as_string": "2023-02-01T00:00:00.000Z",
          "key": 1675209600000,
          "doc_count": 11,
          "total_sales": {
            "value": 18870.850006103516
          }
        },
        {
          "key_as_string": "2023-03-01T00:00:00.000Z",
          "key": 1677628800000,
          "doc_count": 10,
          "total_sales": {
            "value": 17959.33026123047
          }
        },
        {
          "key_as_string": "2023-04-01T00:00:00.000Z",
          "key": 1680307200000,
          "doc_count": 9,
          "total_sales": {
            "value": 18775.959869384766
          }
        },
        {
          "key_as_string": "2023-05-01T00:00:00.000Z",
          "key": 1682899200000,
          "doc_count": 10,
          "total_sales": {
            "value": 11713.169967651367
          }
        },
        {
          "key_as_string": "2023-06-01T00:00:00.000Z",
          "key": 1685577600000,
          "doc_count": 9,
          "total_sales": {
            "value": 6771.1600341796875
          }
        },
        {
          "key_as_string": "2023-07-01T00:00:00.000Z",
          "key": 1688169600000,
          "doc_count": 7,
          "total_sales": {
            "value": 9110.010131835938
          }
        },
        {
          "key_as_string": "2023-08-01T00:00:00.000Z",
          "key": 1690848000000,
          "doc_count": 8,
          "total_sales": {
            "value": 12135.870210647583
          }
        },
        {
          "key_as_string": "2023-09-01T00:00:00.000Z",
          "key": 1693526400000,
          "doc_count": 3,
          "total_sales": {
            "value": 8615.500122070312
          }
        },
        {
          "key_as_string": "2023-10-01T00:00:00.000Z",
          "key": 1696118400000,
          "doc_count": 6,
          "total_sales": {
            "value": 12106.000183105469
          }
        },
        {
          "key_as_string": "2023-11-01T00:00:00.000Z",
          "key": 1698796800000,
          "doc_count": 6,
          "total_sales": {
            "value": 8176.529968261719
          }
        },
        {
          "key_as_string": "2023-12-01T00:00:00.000Z",
          "key": 1701388800000,
          "doc_count": 9,
          "total_sales": {
            "value": 12509.620300292969
          }
        }
      ]
    }
  }
}
```

##### 找出平均订单金额最高的前3个客户。

代码：

``` json
GET /ecommerce/_search
{
  "aggs":{ //代表聚合名称
    "customer_id_group":{ //名称，随意起名
      "terms":{ //分组：对什么分组，由下面的“field”决定
        "field": "customer_id", //分组字段
        "size":3,
        "order":{
          "total_amount_avg":"desc"
        }
      },
      "aggs":{ //对产品类别分完组之后再进行聚合操作
        "total_amount_avg":{ //表示平均订单金额
          "avg":{ //对...求平均
            "field":"total_amount" //求平均字段为total_amount：总金额
          }
        }
      }
    }
  },
  "size":0 //表示只要统计结果，其它原始数据不显示
}
```

结果：

``` json
{
  "took": 4,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 100,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "customer_id_group": {
      "doc_count_error_upper_bound": -1,
      "sum_other_doc_count": 97,
      "buckets": [
        {
          "key": "C977",
          "doc_count": 1,
          "total_amount_avg": {
            "value": 4886.10009765625
          }
        },
        {
          "key": "C543",
          "doc_count": 1,
          "total_amount_avg": {
            "value": 4858.9501953125
          }
        },
        {
          "key": "C165",
          "doc_count": 1,
          "total_amount_avg": {
            "value": 4816.60009765625
          }
        }
      ]
    }
  }
}
```

##### 计算每个年龄段（18-30，31-50，51+）的客户数量。

代码：

``` json
GET /ecommerce/_search
{
  "aggs": {
    "customer_age_range": {
      "range": {
        "field":"customer_age",
        "ranges": [
          {"from": 18, "to":30},
          {"from": 31, "to":50},
          {"from": 51}
        ]
      }
    }
  },
  "size":0 //表示只要统计结果，其它原始数据不显示
}
```

结果：

``` json
{
  "took": 11,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 100,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "customer_age_range": {
      "buckets": [
        {
          "key": "18.0-30.0",
          "from": 18,
          "to": 30,
          "doc_count": 24
        },
        {
          "key": "31.0-50.0",
          "from": 31,
          "to": 50,
          "doc_count": 31
        },
        {
          "key": "51.0-*",
          "from": 51,
          "doc_count": 42
        }
      ]
    }
  }
}
```

##### 计算每个产品类别的平均单价。

代码：

``` json
GET /ecommerce/_search
{
  "aggs": {
    "product_category_group": {
      "terms": {
        "field": "product_category"
      },
      "aggs": {
        "price_avg": {
          "avg": {
            "field": "price"
          }
        }
      }
    }
  },
  "size":0 //表示只要统计结果，其它原始数据不显示
}
```

结果：

``` json
{
  "took": 2,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 100,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "product_category_group": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": "Jewelry",
          "doc_count": 14,
          "price_avg": {
            "value": 537.6807218279157
          }
        },
        {
          "key": "Electronics",
          "doc_count": 13,
          "price_avg": {
            "value": 484.44461763822113
          }
        },
        {
          "key": "Home Appliances",
          "doc_count": 13,
          "price_avg": {
            "value": 645.6961549612192
          }
        },
        {
          "key": "Beauty",
          "doc_count": 11,
          "price_avg": {
            "value": 701.0781749378551
          }
        },
        {
          "key": "Furniture",
          "doc_count": 10,
          "price_avg": {
            "value": 518.5519981384277
          }
        },
        {
          "key": "Books",
          "doc_count": 9,
          "price_avg": {
            "value": 524.0977762010363
          }
        },
        {
          "key": "Groceries",
          "doc_count": 9,
          "price_avg": {
            "value": 566.4644444783529
          }
        },
        {
          "key": "Fashion",
          "doc_count": 8,
          "price_avg": {
            "value": 369.4774992465973
          }
        },
        {
          "key": "Toys",
          "doc_count": 7,
          "price_avg": {
            "value": 485.97999899727955
          }
        },
        {
          "key": "Sports",
          "doc_count": 6,
          "price_avg": {
            "value": 483.9566599527995
          }
        }
      ]
    }
  }
}
```

##### 找出订单数量最多的前5个城市。

代码：

``` json
GET /ecommerce/_search
{
  "aggs": {
    "customer_city_group": {
      "terms": {
        "field": "customer_city",
        "size": 5, 
        "order": {
          "total_payment_method": "desc"
        }
      },
      "aggs": {
        "total_payment_method": {
          "value_count": {
            "field": "payment_method"
          }
        }
      }
    }
  },
  "size":0 //表示只要统计结果，其它原始数据不显示
}
```

结果：

``` json
{
  "took": 9,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 100,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "customer_city_group": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 41,
      "buckets": [
        {
          "key": "Los Angeles",
          "doc_count": 13,
          "total_payment_method": {
            "value": 13
          }
        },
        {
          "key": "San Diego",
          "doc_count": 13,
          "total_payment_method": {
            "value": 13
          }
        },
        {
          "key": "San Jose",
          "doc_count": 12,
          "total_payment_method": {
            "value": 12
          }
        },
        {
          "key": "Philadelphia",
          "doc_count": 11,
          "total_payment_method": {
            "value": 11
          }
        },
        {
          "key": "New York",
          "doc_count": 10,
          "total_payment_method": {
            "value": 10
          }
        }
      ]
    }
  }
}
```

##### 计算每个季度的平均订单金额。

代码：

``` json
GET /ecommerce/_search
{
  "aggs": {
    "order_date_range": {
      "date_histogram": {
        "field":"order_date",
        "calendar_interval": "quarter",
        "format": "yyyy-MM"
      },
      "aggs": {
        "total_amount_avg": {
          "avg": {
            "field": "total_amount"
          }
        }
      }
    }
  },
  "size":0 //表示只要统计结果，其它原始数据不显示
}
```

结果：

``` json
{
  "took": 32,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 100,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "order_date_range": {
      "buckets": [
        {
          "key_as_string": "2023-01",
          "key": 1672531200000,
          "doc_count": 33,
          "total_amount_avg": {
            "value": 1854.9030364065459
          }
        },
        {
          "key_as_string": "2023-04",
          "key": 1680307200000,
          "doc_count": 28,
          "total_amount_avg": {
            "value": 1330.724638257708
          }
        },
        {
          "key_as_string": "2023-07",
          "key": 1688169600000,
          "doc_count": 18,
          "total_amount_avg": {
            "value": 1658.9655813641018
          }
        },
        {
          "key_as_string": "2023-10",
          "key": 1696118400000,
          "doc_count": 21,
          "total_amount_avg": {
            "value": 1561.530973888579
          }
        }
      ]
    }
  }
}
```

##### 统计每个产品类别中的商品数量。

代码：

``` json
GET /ecommerce/_search
{
  "aggs": {
    "product_category_group": {
      "terms": {
        "field":"product_category"
      },
      "aggs": {
        "tproduct_id_number": {
          "value_count": {
            "field": "product_id"
          }
        }
      }
    }
  },
  "size":0 //表示只要统计结果，其它原始数据不显示
}
```

结果：

``` json
{
  "took": 14,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 100,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "product_category_group": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": "Jewelry",
          "doc_count": 14,
          "tproduct_id_number": {
            "value": 14
          }
        },
        {
          "key": "Electronics",
          "doc_count": 13,
          "tproduct_id_number": {
            "value": 13
          }
        },
        {
          "key": "Home Appliances",
          "doc_count": 13,
          "tproduct_id_number": {
            "value": 13
          }
        },
        {
          "key": "Beauty",
          "doc_count": 11,
          "tproduct_id_number": {
            "value": 11
          }
        },
        {
          "key": "Furniture",
          "doc_count": 10,
          "tproduct_id_number": {
            "value": 10
          }
        },
        {
          "key": "Books",
          "doc_count": 9,
          "tproduct_id_number": {
            "value": 9
          }
        },
        {
          "key": "Groceries",
          "doc_count": 9,
          "tproduct_id_number": {
            "value": 9
          }
        },
        {
          "key": "Fashion",
          "doc_count": 8,
          "tproduct_id_number": {
            "value": 8
          }
        },
        {
          "key": "Toys",
          "doc_count": 7,
          "tproduct_id_number": {
            "value": 7
          }
        },
        {
          "key": "Sports",
          "doc_count": 6,
          "tproduct_id_number": {
            "value": 6
          }
        }
      ]
    }
  }
}
```

##### 计算男性和女性客户的平均订单金额

代码：

``` json
GET /ecommerce/_search
{
  "aggs": {
    "customer_gender_group": {
      "terms": {
        "field":"customer_gender"
      },
      "aggs": {
        "total_amount_avg": {
          "avg": {
            "field": "total_amount"
          }
        }
      }
    }
  },
  "size":0 //表示只要统计结果，其它原始数据不显示
}
```

结果：

``` json
{
  "took": 3,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 100,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "customer_gender_group": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": "male",
          "doc_count": 55,
          "total_amount_avg": {
            "value": 1798.5043728915127
          }
        },
        {
          "key": "female",
          "doc_count": 45,
          "total_amount_avg": {
            "value": 1382.397343995836
          }
        }
      ]
    }
  }
}
```

##### 找出总销售额最高的前10个日期。

代码：

``` json
GET /ecommerce/_search
{
  "aggs":{ //代表聚合名称
    "top10_order_date":{ //名称，随意起名
      "terms":{ //分组：对什么分组，由下面的“field”决定
        "field": "order_date", //分组字段
        "size":10,
        "order": {
          "total_sales": "desc" //降序分组
        }
      },
      "aggs": { //求总销量
        "total_sales": {
          "sum": {
            "field":"total_amount"
          }
        }
      }
    }
  },
  "size":0 //表示只要统计结果，其它原始数据不显示
}
```

结果：

``` json
{
  "took": 82,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 100,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "top10_order_date": {
      "doc_count_error_upper_bound": -1,
      "sum_other_doc_count": 86,
      "buckets": [
        {
          "key": 1682726400000,
          "key_as_string": "2023-04-29T00:00:00.000Z",
          "doc_count": 2,
          "total_sales": {
            "value": 5951.77001953125
          }
        },
        {
          "key": 1677628800000,
          "key_as_string": "2023-03-01T00:00:00.000Z",
          "doc_count": 2,
          "total_sales": {
            "value": 5457.640075683594
          }
        },
        {
          "key": 1703980800000,
          "key_as_string": "2023-12-31T00:00:00.000Z",
          "doc_count": 2,
          "total_sales": {
            "value": 5177.3402099609375
          }
        },
        {
          "key": 1694736000000,
          "key_as_string": "2023-09-15T00:00:00.000Z",
          "doc_count": 1,
          "total_sales": {
            "value": 4886.10009765625
          }
        },
        {
          "key": 1696291200000,
          "key_as_string": "2023-10-03T00:00:00.000Z",
          "doc_count": 1,
          "total_sales": {
            "value": 4858.9501953125
          }
        },
        {
          "key": 1678665600000,
          "key_as_string": "2023-03-13T00:00:00.000Z",
          "doc_count": 1,
          "total_sales": {
            "value": 4592.9501953125
          }
        },
        {
          "key": 1691107200000,
          "key_as_string": "2023-08-04T00:00:00.000Z",
          "doc_count": 1,
          "total_sales": {
            "value": 4131.9501953125
          }
        },
        {
          "key": 1677283200000,
          "key_as_string": "2023-02-25T00:00:00.000Z",
          "doc_count": 2,
          "total_sales": {
            "value": 3872.6699829101562
          }
        },
        {
          "key": 1681171200000,
          "key_as_string": "2023-04-11T00:00:00.000Z",
          "doc_count": 1,
          "total_sales": {
            "value": 3790.89990234375
          }
        },
        {
          "key": 1674777600000,
          "key_as_string": "2023-01-27T00:00:00.000Z",
          "doc_count": 1,
          "total_sales": {
            "value": 3714.25
          }
        }
      ]
    }
  }
}
```

##### 计算每个季度销售额最高的产品类别。

代码：

``` json
GET /ecommerce/_search
{
  "aggs": {
    "order_date_group": {
      "date_histogram": {
        "field":"order_date",
        "calendar_interval": "quarter",
        "format": "yyyy-MM"
      },
      "aggs": {
        "top_sales_category": {
          "terms": {
            "field": "product_category",
            "size": 1,
            "order": {
              "total_sales": "desc"
            }
          },
          "aggs": {
            "total_sales": {
              "sum": {
                "field": "total_amount"
              }
            }
          }
        }
      }
    }
  },
  "size":0 //表示只要统计结果，其它原始数据不显示
}
```

结果：

``` json
{
  "took": 56,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 100,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "order_date_group": {
      "buckets": [
        {
          "key_as_string": "2023-01",
          "key": 1672531200000,
          "doc_count": 33,
          "top_sales_category": {
            "doc_count_error_upper_bound": 0,
            "sum_other_doc_count": 25,
            "buckets": [
              {
                "key": "Jewelry",
                "doc_count": 8,
                "total_sales": {
                  "value": 11757.280212402344
                }
              }
            ]
          }
        },
        {
          "key_as_string": "2023-04",
          "key": 1680307200000,
          "doc_count": 28,
          "top_sales_category": {
            "doc_count_error_upper_bound": 0,
            "sum_other_doc_count": 24,
            "buckets": [
              {
                "key": "Home Appliances",
                "doc_count": 4,
                "total_sales": {
                  "value": 6474.219955444336
                }
              }
            ]
          }
        },
        {
          "key_as_string": "2023-07",
          "key": 1688169600000,
          "doc_count": 18,
          "top_sales_category": {
            "doc_count_error_upper_bound": 0,
            "sum_other_doc_count": 15,
            "buckets": [
              {
                "key": "Home Appliances",
                "doc_count": 3,
                "total_sales": {
                  "value": 6066.1199951171875
                }
              }
            ]
          }
        },
        {
          "key_as_string": "2023-10",
          "key": 1696118400000,
          "doc_count": 21,
          "top_sales_category": {
            "doc_count_error_upper_bound": 0,
            "sum_other_doc_count": 17,
            "buckets": [
              {
                "key": "Beauty",
                "doc_count": 4,
                "total_sales": {
                  "value": 9416.89013671875
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

##### 

##### 



