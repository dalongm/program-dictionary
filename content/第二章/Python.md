# 第二章 Python

## 离线安装库

```shell
# 1. 查看安装的包
pip list

# 2. 生成依赖列表
pip freeze >requirements.txt

# 3. 下载依赖文件的包到本地
pip download -d .\packages -r .\requirements.txt --trusted-host mirrors.aliyun.com

# 4. 离线安装
pip install --no-index --find-links .\packages -r .\requirements.txt

pip install numpy-1.16.2-cp36-cp36m-manylinux1_x86_64.whl

# 5. 安装 tar.gz
tar -zxvf xxx.tar.gz
cd xxx
python setup.py install
```

## str bytes互转

```python
# bytes object  
 b = b"example"  
  
 # str object  
 s = "example"  
  
 # str to bytes  
 bytes(s, encoding = "utf8")  
  
 # bytes to str  
 str(b, encoding = "utf-8")  
  
 # an alternative method  
 # str to bytes  
 str.encode(s)  
  
 # bytes to str  
 bytes.decode(b)  
```

## Conda

### 强制卸载

```shell
conda remove --force name
```

### 离线安装

```shell
conda install --use-local pytorch-1.0.0-py2.7_cuda9.0.176_cudnn7.4.1_1.tar.bz2
```



# 