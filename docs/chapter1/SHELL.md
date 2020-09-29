

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

if []; then
elif []; then
else
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

### 替换`JAVA_HOME`

```bash
PROFILE=/etc/profile
# 配置环境变量
# 不包含已有的JAVA_HOME
JAVA_HOME=/usr/java/jdk1.8.0_212-amd64
if [ $(grep -c "export JAVA_HOME=" $PROFILE) -eq 0 ]; then
    echo "export JAVA_HOME=${JAVA_HOME}" >>$PROFILE
    echo "export JRE_HOME=\${JAVA_HOME}/jre" >>$PROFILE
    echo "export CLASSPATH=.:\${JAVA_HOME}/lib:\${JRE_HOME}/lib" >>$PROFILE
    echo "export PATH=\${PATH}:\${JAVA_HOME}/bin" >>$PROFILE
else
	JAVA_HOME_REG=${JAVA_HOME//\//\\/}
    sed -i "s/^export JAVA_HOME.*\$/export JAVA_HOME=$JAVA_HOME_REG/g" $PROFILE
fi
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

### 特殊用法

```bash
${ }的一些特异功能

定义一个变量：
file=/dir1/dir2/dir3/my.file.txt
可以用${ }分别替换获得不同的值：
${file#*/} 拿掉第一个 / 及其左边的字符串：dir1/dir2/dir3/my.file.txt
${file##*/} 拿掉最后一个 / 及其左边的字符串：my.file.txt
${file#*.} 拿掉第一个 . 及其左边的字符串：file.txt
${file##*.} 拿掉最后一个 . 及其左边的字符串：txt
${file%/*} 拿掉最后一个 / 及其右边的字符串：/dir1/dir2/dir3
${file%%/*} 拿掉第一个 / 及其右边的字符串：(空值)
${file%.*} 拿掉最后一个 . 及其右边的字符串：/dir1/dir2/dir3/my.file
${file%%.*} 拿掉第一个 . 及其右边的字符串：/dir1/dir2/dir3/my
记忆的方法：
# 去掉左边(键盘上 # 在 $ 的左边)
% 去掉右边(在键盘上 % 在 $ 的右边)
单一符号是最小匹配，两个符号是最大匹配。
${file:0:5} 提取最左边的 5 个字节：/dir1
${file:5:5} 提取第 5 个字节右边的连续 5 个字节：/dir2
也可以对变量值里的字符串作替换：
${file/dir/path} 将第一个 dir 替换为 path：/path1/dir2/dir3/my.file.txt
${file//dir/path} 将全部 dir 替换为 path：/path1/path2/path3/my.file.txt
利用${ }还可针对不同的变量状态赋值(未设定、空值、非空值)： 
${file-my.file.txt} 若 $file 未设定，则使用 my.file.txt 作传回值。(空值及非空值时不作处理) 
${file:-my.file.txt} 若 $file 未设定或为空值，则使用 my.file.txt 作传回值。(非空值时不作处理)
${file+my.file.txt} 若 $file 设为空值或非空值，均使用 my.file.txt 作传回值。(未设定时不作处理)
${file:+my.file.txt} 若 $file 为非空值，则使用 my.file.txt 作传回值。(未设定及空值时不作处理)
${file=my.file.txt} 若 $file 未设定，则使用 my.file.txt 作传回值，同时将 $file 赋值为 my.file.txt。 (空值及非空值时不作处理)
${file:=my.file.txt} 若 $file 未设定或为空值，则使用 my.file.txt 作传回值，同时将 $file 赋值为 my.file.txt。 (非空值时不作处理)
${file?my.file.txt} ：若 $file 未设定，则将 my.file.txt 输出至 STDERR。(空值及非空值时不作处理)
${file:?my.file.txt} ：若 $file 未设定或为空值，则将 my.file.txt 输出至 STDERR。(非空值时不作处理)
以上的理解在于，一定要分清楚 unset 与 null 及 non-null 这三种赋值状态。
一般而言，与 null 有关，若不带 : 的话，null 不受影响，若带 : 则连 null 也受影响。
${#var} 可计算出变量值的长度：
${#file} 可得到 27，/dir1/dir2/dir3/my.file.txt 刚好是 27 个字节。
```

## 获取目录下的文件夹

```bash
ls -al $1 | awk '/^d/ {print $NF}'
```

## 获取目录下的文件

```bash
ls -al $1 | awk '/^-/ {print $NF}'
```

## Expect命令

> [Linux expect详解](https://www.jianshu.com/p/2fcdf764f464)

新建用户，免密自身脚本。

`main.sh`

```bash
#!/bin/bash
# main.sh

BASE_DIR=$(readlink -f $(dirname $0))
NAME="hello"

cd ${BASE_DIR}
adduser $NAME

echo "$NAME" | passwd --stdin $NAME

# 添加账户到root组
usermod -a -G root $NAME

su -l $NAME -c "expect ${BASE_DIR}/ssh.sh $NAME"
```

`ssh.sh`

```bash
#!/usr/bin/expect
# ssh.sh

# $argc 参数数据
if {$argc < 1} {
    puts "Usage:expect username"
    exit 1
}

set timeout 30
set host "localhost"

# $argv 0 第0个参数值
set username [lindex $argv 0]
set yes "yes"

spawn ssh-keygen
expect "*Enter file in which to save the key*" {send "\r"}
expect "*Enter passphrase*" {send "\r"}
expect "*Enter same passphrase*" {send "\r"}

# 添加公钥
exec bash -c "cat /home/$username/.ssh/id_rsa.pub >>/home/$username/.ssh/authorized_keys"

# 修改.ssh目录及文件权限
exec bash -c "chmod 700 /home/$username/.ssh/"
exec bash -c "chmod 600 /home/$username/.ssh/authorized_keys"

# 处理第一次登录确认提示
spawn ssh $username@$host
expect "*Are you sure you want*" {send "$yes\r exit\r"}
expect eof
```

## nohup

使用&后台运行程序：

- 结果会输出到终端
- 使用Ctrl + C发送SIGINT信号，程序免疫
- 关闭session发送SIGHUP信号，程序关闭

使用nohup运行程序：

- 结果默认会输出到nohup.out
- 使用Ctrl + C发送SIGINT信号，程序关闭
- 关闭session发送SIGHUP信号，程序免疫

平日线上经常使用nohup和&配合来启动程序：

- 同时免疫SIGINT和SIGHUP信号

同时，还有一个最佳实践：

- 不要将信息输出到终端标准输出，标准错误输出，而要用日志组件将信息记录到日志里