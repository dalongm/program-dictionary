# 5.2 ES

[TOC]

## 查看mapping

```shell
curl -XGET '127.0.0.1:9200/noah_app_access_20200101/_mapping/?pretty=true'
```

## 统计某个字段的数目

```url
http://host:9200/_sql?sql=SELECT count(distinct(resource_id)) as num FROM index_*
```

## 查看索引数据

**POST** 

```url
http://host:9200/index_*/_search?pretty
```

**Body**

```json
{
	"size":1000
}
```

## 删除索引

```shell
curl -XDELETE '127.0.0.1:9200/index_*'
```

