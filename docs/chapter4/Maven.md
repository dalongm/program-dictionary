# 4.2 Maven

[TOC]

## 忽略测试用例

```shell
# 不执行测试用例，也不编译测试用例类
mvn package -Dmaven.test.skip=true
# 不执行测试用例，但编译测试用例类生成相应的class文件至target/test-classes下
mvn package -DskipTests
```

## 刷新本地缓存

```shell
# mvn源更新后，刷新本地缓存
mvn clean package -U
```

## 

