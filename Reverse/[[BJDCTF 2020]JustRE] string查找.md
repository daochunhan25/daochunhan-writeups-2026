> Problem: [[BJDCTF 2020]JustRE](https://www.nssctf.cn/problem/703)

## Reverse

#c #语言逆向 #逆向技术

# 字符串查找法

## 文件信息

* **文件**: RE2.exe (32位Windows GUI程序)
* **赛事**: BJDCTF 2020

## 解题步骤

### 1. 字符串提取

```
rabin2 -z RE2.exe | grep -i bjd
```

![3889b930d0.jpg](/files/2026/1/18/3889b930d0.jpg)

发现flag中包含两个整型参数

分析vaddr虚拟地址0x407030

## 2. 定位引用代码

```
r2 -A RE2.exe
[0x00401462]> axt 0x407030
```

![0186a26b35.jpg](/files/2026/1/18/0186a26b35.jpg)

字符串在地址0x4013aa被引用

### 3. 分析汇编代码

查看引用字符串地址代码

![98a70c6c8d.jpg](/files/2026/1/18/98a70c6c8d.jpg)

发现`fcn.004013aa` 函数分析不完整 查看更前的代码

```
[0x004013aa]> pd -20 @0x4013aa
```

![6b04336018.jpg](/files/2026/1/18/6b04336018.jpg)

综上 得到push两个参数0 0x4e1f=19999 flagstrBJD

### 4. 构造flag

* 格式字符串：`BJD{%d%d2069a45792d233ac}`
* 参数1: 19999 (0x4e1f)
* 参数2: 0
* **Flag**: `NSSCTF{1999902069a45792d233ac}`