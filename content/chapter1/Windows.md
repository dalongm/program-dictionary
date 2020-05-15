# Windows

## 目录树（tree）

```powershell
# 文件夹
tree
# 包含文件
tree /F
# 适用ASCII字符
tree /A
```

## 查看MD5

```bash
# 查看MD5值：
certutil -hashfile 文件名  MD5

# 查看 SHA1
certutil -hashfile 文件名  SHA1 

# 查看SHA256
certutil -hashfile 文件名  SHA256

# 支持的算法有：MD2 MD4 MD5 SHA1 SHA256 SHA384 SHA512
```


