# 1.2 SHELL

[TOC]

## 判断是否为空

```shell
if [ -n "$APP_ID" ]; then
	echo "is not empty"
fi

if [ -z "$APP_ID" ]; then
	echo "is empty"
fi
```

## 获取参数

```shell
#!/bin/bash
echo $1 $2

# 获取当前进程id
echo $!

# 当前脚本的文件名
echo $0
```

## 数组循环

```shell
for type in ${TYPES[@]}; do
    TYPES_STR="$TYPES_STR $type"
done

# 获取数组元素
echo "数组的元素为: ${my_array[*]}"
echo "数组的元素为: ${my_array[@]}"

# 获取数组长度
echo "数组元素个数为: ${#my_array[*]}"
echo "数组元素个数为: ${#my_array[@]}"

```

## 基本循环

```shell
for ((i = 0; i < ${#TYPES[@]}; i++)); do
	echo ${TYPES[i]}
done
```

## 文件判断

```shell
# 文件不存在
if [ ! -f "$file" ]; then
 touch "$file"
fi
# 文件夹不存在
if [ ! -d "$folder"]; then
 mkdir "$folder"
fi
```

## 数字大小判断

```shell
if [ $num1 -gt $num2 ] ; then
    echo "$num1 > $num2"
else
    echo "$num1 < $num2  或者 $num1 = $num2"
fi
```

## 获取函数返回值

```shell
#!/bin/bash

funWithReturn(){
    echo "这个函数会对输入的两个数字进行相加运算..."
    echo "输入第一个数字: "
    read aNum
    echo "输入第二个数字: "
    read anotherNum
    echo "两个数字分别为 $aNum 和 $anotherNum !"
    return $(($aNum+$anotherNum))
}
funWithReturn
echo "输入的两个数字之和为 $? !"
```



## 字符串匹配

**==比较**

使用bash检查字符串是否以某些字符开头可以使用==比较

```shell
[[ $str == h* ]]
```

示例

```shell
str="hello"
if [[ $str == h* ]];
then
 echo 'yes'
fi
```

有两个地方需要注意：

1. h*不需要使用引号括起来，使用引号括起来是直接做相等比较
2. 比较语句使用双中括号括起来，而不是使用单中括号

**=~正则比较**

如果使用Bash的正则

```shell
str="hello"
if [[ "$str" =~ ^he.* ]]; then
    echo "yes"
fi
```

使用正则匹配字符串的开头字符需要注意：

- `he*`：不要使用`he*`，这里的`*`号表示`e`字符`0`到多个，即`h`，以及`heeee`都是测试通过的
- `he.*`：这里只允许包含`he`的字符串通过测试
- `^he.*`：这个表示是以`he`开头的字符串通过检测

## 命令找不到（command not found）

```shell
export PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin
source /etc/profile
# 或者
/bin/vi  /etc/profile
```

## 文件内容删除（sed）

```shell
FILE="/usr/local/test.path"
CONTENT="/usr/local/test.txt"
# 将"/"替换为"\/"
CONTENT_REG=${CONTENT//\//\\/}
# 适用sed删除
sed -i ${CONTENT_REG}/d ${FILE}
```

## $ 变量含义

linux中shell变量$#,$@,$0,$1,$2的含义解释: 

```shell
$$ 
# Shell本身的PID（ProcessID） 

$! 
# Shell最后运行的后台Process的PID 

$? 
# 最后运行的命令的结束代码（返回值） 

$- 
# 使用Set命令设定的Flag一览

$* 
# 所有参数列表。如"$*"用「"」括起来的情况、以"$1 $2 … $n"的形式输出所有参数。 

$@ 
# 所有参数列表。如"$@"用「"」括起来的情况、以"$1" "$2" … "$n" 的形式输出所有参数。 

$# 
# 添加到Shell的参数个数 

$0 
# Shell本身的文件名 

$1～$n 
# 添加到Shell的各参数值
```
