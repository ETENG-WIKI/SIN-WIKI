## 安装elasticSearch

前提：安装java（至少是1.7），并配置环境变量

首先，下载压缩包：

```
wget https://download.elastic.co/elasticsearch/release/org/elasticsearch/distribution/tar/elasticsearch/2.3.1/elasticsearch-2.3.1.tar.gz
```

第二步，解压：

```
tar -xzvf elasticsearch-2.3.1.tar.gz
```

第三步，删除压缩包：

```
rm elasticsearch-2.3.1.tar.gz
```

第四步，移动目录到/opt/local：

```
mv elasticsearch-2.3.1 /opt/local
cd /opt/local/elasticsearch-2.3.1
```

第五步，启动elasticSearch：

```
bin/elasticsearch -Des.insecure.allow.root=true
```

第六步，设置开机启动：

用vi修改/etc/rc.d/rc.local这个文件，在文件中添加

```
/opt/local/elasticsearch-2.3.1/bin/elasticsearch -Des.insecure.allow.root=true
```

至此，elasticSearch安装完成。

## 分词插件的安装与配置

第一步，到elasticSearch的安装目录:

cd /opt/local/elasticsearch-2.3.1

第二步，安装插件:

```
./bin/plugin install http://maven.nlpcn.org/org/ansj/elasticsearch-analysis-ansj/2.3.1/elasticsearch-analysis-ansj-2.3.1-release.zip
```

第三步，设置ansj作为默认分词器:

```
cd /opt/local/elasticsearch-2.3.1/config
vi elasticsearch.yml
```

加入以下配置

```

#默认分词器,索引
index.analysis.analyzer.default.type: index_ansj

#默认分词器,查询
index.analysis.analyzer.default_search.type: query_ansj

```

# 配置索引库

第一步 建立索引，并配置type（相当于数据表）

```
/**
  * curl生成配置
  * -d 后面是配置json
  * 配置json DEMO:
  * (wtd_worktrend为type，
  *  index配置为not_analyzed时，不进行分词)
  *  {
  *    "mappings": {
  *      "wtd_worktrend": {
  *        "properties": {
  *          "customer_name": {
  *            "type": "string"
  *          },
  *          "is_pub": {
  *            "type": "long"
  *          },
  *          "region_area": {
  *            "type": "string"
  *          },
  *          "region_name": {
  *            "type": "string"
  *          },
  *          "spl_flag": {
  *            "type": "long"
  *          },
  *          "wtd_address": {
  *            "type": "string"
  *          },
  *          "wtd_content": {
  *            "type": "string"
  *          },
  *          "wtd_name": {
  *            "type": "string"
  *          },
  *          "wtd_time": {
  *            "type": "string"
  *          },
  *          "from_code": {
  *            "type": "string",
  *            "index": "not_analyzed"
  *          },
  *          "corp_id": {
  *            "type": "string",
  *            "index": "not_analyzed"
  *          },
  *          "user_id": {
  *            "type": "string",
  *            "index": "not_analyzed"
  *          },
  *          "cus_id": {
  *            "type": "string",
  *            "index": "not_analyzed"
  *          }
  *        }
  *      }
  *    }
  *  }
  *
  * http://localhost:9200/villagesale villagesale为索引名称
  */
curl -H 'Content-Type: application/json' -X PUT -d '{"mappings":{"wtd_worktrend":{"properties":{"customer_name":{"type":"string"},"is_pub":{"type":"long"},"region_area":{"type":"string"},"region_name":{"type":"string"},"spl_flag":{"type":"long"},"wtd_address":{"type":"string"},"wtd_content":{"type":"string"},"wtd_name":{"type":"string"},"wtd_time":{"type":"string"},"from_code":{"type":"string","index":"not_analyzed"},"corp_id":{"type":"string","index":"not_analyzed"},"user_id":{"type":"string","index":"not_analyzed"},"cus_id":{"type":"string","index":"not_analyzed"}}}}}' http://localhost:9200/villagesale
```
返回数据为下面的结果时，则配置成功。
```
{"acknowledged":true}
```

第二步 导入数据，请参考elasticSearch数据导入。

以上出错的解决办法：
删除索引库，然后重新走步骤一和二：
```
curl -X DELETE 'http://localhost:9200/villagesale'
```
返回数据为下面的结果时，则删除成功。
```
{"acknowledged":true}
```

## elastic-jdbc
第一步，获取压缩包：

```
wget http://xbib.org/repository/org/xbib/elasticsearch/importer/elasticsearch-jdbc/2.3.1.0/elasticsearch-jdbc-2.3.1.0-dist.zip
```

第二步，解压：

```
unzip elasticsearch-jdbc-2.3.1.0-dist.zip
```

第三步，删除压缩包文件：

```
rm elasticsearch-jdbc-2.3.1.0-dist.zip
```

第四步，移动文件到对应目录下：

```
mv elasticsearch-jdbc-2.3.1.0 /opt/local
```

第五步，新建一个sh文件：

```
cd /opt/local/elasticsearch-jdbc-2.3.1.0/bin

touch jdbc.sh

chmod +x jdbc.sh
```

第六步，修改sh文件(以下配置仅供参考，连接以及数据库等配置需按照情况修改)（如有报错，请升级java和javac到最新版本）：

```
#!/bin/bash

JDBC_IMPORTER_HOME="/opt/local/elasticsearch-jdbc-2.3.1.0"

bin=$JDBC_IMPORTER_HOME/bin

lib=$JDBC_IMPORTER_HOME/lib

echo $bin

echo $lib

echo '{

 "type" : "jdbc",

 "jdbc" : {

 //增量更新：设置初始时间

 "metrics" : {

   "lastexecutionstart" : "1990-01-01T00:00:00.000Z",

 "lastexecutionend" : "1990-01-01T00:00:00.000Z",

 "counter" : 0//任务运行次数

 },

 //定时执行：0秒、每分钟、每小时、每天、每月、每星期

 "schedule" : "0 0-59 0-23 ? * *",

 "url" : "jdbc:mysql://192.168.0.50:3306/villagesale",

 "statefile" : "statefile.json",//状态文件，记住最后一次同步时间

 "user" : "****",

 "password" : "****",

 "sql" : [

 {

 //主键作为index _id

 "statement" : "select *, wtd_id as _id from wtd_worktrend where ts > ?",

 "parameter" : [ "$metrics.lastexecutionstart" ]

 },

 {

  //记住每次同步日志:每个表

 "statement" : "insert into elasticsearch_jdbc values(‘wtd_worktrend’,?,?,?,?,?,?,?,?,?)",

 "parameter" : [

 "$job",//同一个表的同步次数

 "$now",

 "$state",

 "$metrics.counter",

 "$lastrowcount",//每次同步的记录条数

 "$lastexceptiondate",

 "$lastexception",

 "$metrics.lastexecutionstart",

 "$metrics.lastexecutionend"

 ]

 }

 ],

 "treat_binary_as_string" : true,

 "elasticsearch" : {

 "cluster" : "elasticsearch",

 "host" : "localhost",

 "port" : 9300

 },

 "index" : "villagesale",//对应mysql的数据库名

 "type" : "wtd_worktrend"//对应mysql的表名

  }

}' | java \

-cp "${lib}/\*" \

-Dlog4j.configurationFile=${bin}/log4j2.xml \

org.xbib.tools.Runner \

org.xbib.tools.JDBCImporter

```

第七步，运行sh文件：

```
./jdbc.sh
```

现在，就可以自动同步数据库。可以根据需求将此脚本加入开机启动项。

第八步，加入开机启动项：

```
vi /etc/rc.d/rc.local
```

追加代码

```
/opt/local/elasticsearch-jdbc-2.3.1.0/bin/marking-jdbc.sh

```

## restful API导入
方式一 新增单条纪录  
方法： POST  
路径：域名/index名称/type名称/文档ID  
Body：JSONObject（对象里不要有_id的字段，否则报错）

方式二 批量更新纪录  
待补充

## bool查询
Par1 概念介绍  
bool查询，常用的有三种：  
must 且查询，匹配程度是count大小依据  
should 或查询，匹配程度是count大小依据  
filter 且查询，匹配程度不作为count大小依据

或查询，至少匹配一个条件：  
minimum_should_match 设置为1

完整示例：
```
{
  "query":{
    "bool": {
      "must": [
        {
          "match": {
            "field1": "content1"
          }
        }
      ],
      "filter": [
        {
          "match": {
            "field2": "content2"
          }
        }
      ],
      "should": [
        {
          "match": {
            "field3": "content3"
          }
        }
      ],
      "minimum_should_match": 1
    }
  }
}
```

## 分页
开始位置 from  
分页大小 size

完整示例：
```
//从0～9
{
  "from": 0,
  "size": 10
}
```

## 排序
字段排序：
```
{
  "field": {
    "order": "asc"
  }
}
```

count排序：_score

完整示例：
```
//根据field和score排序
{
  "sort": [
    {
      "field": {
        "order": "asc"
      }
    },
    "_score"
  ]
}
```

## score初始化
```
{
  "track_scores": 1
}
```

## 展示数据配置
如果不配置，默认是全部字段
```
{
  "_source": [
    "field1",
    "field2"
  ]
}
```

## 开始搜索
方法：POST  
路径：域名/index名称/type名称/_search?pretty  
数据：JSONObject  
数据示例：
```
{
  "query":{
    "bool": {
      "must": [
        {
          "match": {
            "field1": "content1"
          }
        }
      ],
      "filter": [
        {
          "match": {
            "field2": "content2"
          }
        }
      ],
      "should": [
        {
          "match": {
            "field3": "content3"
          }
        }
      ],
      "minimum_should_match": 1
    }
  },
  "from": 0,
  "size": 10,
  "sort": [
    {
      "field": {
        "order": "asc"
      }
    },
    "_score"
  ],
  "track_scores": 1,
  "_source": [
    "field1",
    "field2"
  ]
}
```
