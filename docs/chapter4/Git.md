# 4.1 Git

[TOC]



## 安装Git

### 下载

```shell
wget https://www.kernel.org/pub/software/scm/git/git-2.11.1.tar.gz
```

### 安装依赖

```shell
yum groupinstall "Development Tools"
yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel perl-CPAN perl-devel perl-ExtUtils-Embed
```

### 解压安装

```shell
tar -zvxf git-2.11.1.tar.gz
cd git-2.11.1
# 指向ssl安装位置
./configure --with-openssl=/usr/local/openssl
make install

rpm -e git --nodeps
ln -s /usr/local/bin/git /usr/bin/git

git version
```

## 基本操作

```bash
# 更新远程仓库链接
git remote set-url origin xxxx

# 推送指定分支到远端
git pull <远程主机名> <远程分支名>:<本地分支名>

# 拉取指定分支到本地
git push <远程主机名> <本地分支名>:<远程分支名>

# 取消commit但保留add
git reset --soft HEAD^

# 取消commit和add
git reset HEAD^

# 删除commit和add代码，恢复到上次的提交状态
git reset HEAD^
```

## 子模块

```bash
# 添加子模块
git submodule add ${子模块仓库地址} ${子模块名称}

# 拉取子模块
# 方法一：正常的使用git clone命令，然后再使用 git submodule init 和git submodule update来获取子模块
# 初始化子模块(只添加子模块版本的指针，文件夹是空的)
git submodule init ${子模块名称}

# 更新子模块
git submodule update ${子模块名称}

# 方法二：在使用git clone命令时，加上–recurse-submodules或–recursive 这样的递归参数
git clone --recursive ${主仓库地址}
```



## .修改默认编辑器

以sublime为例，需将`sublime_text.exe`的路径添加到环境变量中。

```shell
git config --global core.editor sublime_text.exe
```

## 修改已commit的注释信息

```shell
git commit --amend
```

## 查看用户名和邮箱地址

```shell
git config user.name
git config user.email
```

## 修改用户名和邮箱地址

```shell
git config --global user.name "username"
git config --global user.email "email"
```

## 文件换行格式

```shell
# 提交时转换为LF，检出时不转换
git config --global core.autocrlf input
# 拒绝提交包含混合换行符的文件
git config --global core.safecrlf true
```

## 查看所有的提交记录

```shell
git log
```

## 查看最新的commit

```shell
git show
```

## 查看指定commit hashID的所有修改

```shell
git show commitId
```

## 查看某次commit中具体某个文件的修改

```shell
git show commitId fileName
```

## 回到与远程仓库一致处

```shell
git fetch --all  
git reset --hard origin/develop
git pull
```

##  标签

```shell
# 新建一个标签
git tag <tagname>

# 可以指定标签信息
git tag -a v0.1 -m "version 0.1 released"
# 查看所有标签
git tag
# 推送标签
git push origin v1.5

# 删除标签
git tag -d v1.4-lw

# 删除远程标签
git push origin :refs/tags/v1.4-lw

# 检出标签
git checkout 2.0.0
```

## 修改 comment

```shell
git commit --amend
```

## 清除本地的新增文件（未add）

```shell
git clean -df
```

## 常见问题

### fatal: write error: Broken pipe

```shell
git config http.postBuffer 104857600
```


### 中文乱码处理

```shell
# 注释：该命令表示提交命令的时候使用utf-8编码集提交
git config --global i18n.commitencoding utf-8
# 注释：该命令表示日志输出时使用utf-8编码集显示
git config --global i18n.logoutputencoding utf-8
# 注释：设置LESS字符集为utf-8
export LESSCHARSET=utf-8
```

## Git LFS

### 介绍

> Git 大文件存储（Large File Storage，简称LFS）目的是更好地把大型二进制文件，比如音频文件、数据集、图像和视频等集成到 Git 的工作流中。我们知道，Git 存储二进制效率不高，因为它会压缩并存储二进制文件的所有完整版本，随着版本的不断增长以及二进制文件越来越多，这种存储方案并不是最优方案。而 LFS 处理大型二进制文件的方式是用文本指针替换它们，这些文本指针实际上是包含二进制文件信息的文本文件。文本指针存储在 Git 中，而大文件本身通过HTTPS托管在Git LFS服务器上。 

### 安装

```shell
git lfs install

	Updated git hooks.
    Git LFS initialized.
```

### LFS追踪文件

```shell
git lfs track "*.zip"
git lfs track "Anaconda3-4.4.0-Linux-x86_64.sh"
```

执行命令后，将会在项目中生成`.gitattributes`文件，该文件保存文件的追踪记录，需要将该文件推送到远程仓库当中。

### 查看追踪规则

```shell
git lfs track

    Listing tracked patterns
        *.iso (.gitattributes)
        grafana-6.4.3-1.x86_64.rpm (.gitattributes)
    Listing excluded patterns
```

### 提交&推送

与git基本操作一致，使用”git add“，”git commit“，”git push“命令。

```shell
# 添加
git add Anaconda3-4.4.0-Linux-x86_64.sh
# 提交
git commit -m "添加大文件 Anaconda3-4.4.0-Linux-x86_64.sh"
# 推送
git push
```

### 查看追踪列表

```shell
git lfs ls-files

194faa7784 * grafana-6.4.3-1.x86_64.rpm
```

### 克隆&拉取

与git基本操作一致，使用`git clone`,`git pull`命令。

```shell
# 克隆
git clone git@gitlab.example.com:group/project.git
# 拉取
git lfs fetch origin master
```

## 可执行权限

```bash
# 查看文件filename的权限
git ls-files --stage filename
> 100644 7365f0cb8734bd2031ddf800e90 0       filename

# 增加文件filename的可执行权限
git update-index --chmod +x filename

# 再次查看文件filename的权限
git ls-files --stage filename
> 100755 7365f0cb8734bd2031ddf800e90 0       filename

# 忽略权限更改
# 当前仓库
git config core.filemode false
# 全局
git config --global core.fileMode false
```



[^1]:  https://docs.gitlab.com/ee/administration/lfs/manage_large_binaries_with_git_lfs.html 