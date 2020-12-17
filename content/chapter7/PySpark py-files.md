# spark-submit提交PySpark任务py-files参数

## 目的

将pyspark任务提交到Spark on Yarn集群时，可能遇到依赖缺失的情况，使用py-files参数可以传依赖文件到集群中，解决依赖缺失的问题。

### 个别依赖py文件

只有几个依赖文件时，可以使用“,”分隔方式指定。

```
spark-submit \
  --master yarn \
  --deploy-mode cluster \
  --py-files dep1.py,dep2.py \
  spark-app.py
```

### 子模块

当依赖存在子模块时，可以将依赖打成一个zip包，比如项目目录结构为：

```
my-spark-app
│  config.py
│  spark-app.py
│
└─sub_module
        sub_test1.py
        sub_test2.py
        __init__.py
```

使用以下命令将依赖“config.py,sub_module”打到同一个压缩包中：

```bash
cd xxx/my-spark-app
zip -r -q dependence.zip config.py sub_module
```

然后使用提交命令：

```
spark-submit \
  --master yarn \
  --deploy-mode cluster \
  --py-files denpendence.zip \
  spark-app.py
```

### 整个环境

如果想把本地开发环境依赖整体打包可以使用命令：

```bash
cd xxx/python3.6/site-packages
zip -r -q dependence.zip *
```

然后使用提交命令：

```
spark-submit \
  --master yarn \
  --deploy-mode cluster \
  --py-files denpendence.zip \
  spark-app.py
```

然而经测试denpendence.zip中的**egg依赖模块**是无法引用的，可以单独将**egg依赖模块**提出来，添加在py-files参数后面：

```
spark-submit \
  --master yarn \
  --deploy-mode cluster \
  --py-files denpendence.zip,hdfs-2.5.8-py3.6.egg \
  spark-app.py
```

## 参考资料

[python - I can't seem to get --py-files on Spark to work - Stack Overflow](https://stackoverflow.com/questions/36461054/i-cant-seem-to-get-py-files-on-spark-to-work)