> Problem: [[BJDCTF 2020]JustRE](https://www.nssctf.cn/problem/703)

## Reverse

#c #语言逆向 #逆向技术

## 文件信息

* **文件名**: RE2.exe
* **类型**: 32位Windows GUI程序

![90befe2ad9.jpg](/files/2026/1/18/90befe2ad9.jpg)

## 解题步骤

调用链概括

1. **`entry`** → 程序入口
2. **`FUN_00401000` (main)** → 主函数
3. **`FUN_00401150`** → 创建主窗口
4. **`FUN_004011c0`** → 主窗口过程
5. **`FUN_00401350`** → 对话框过程

#### 1.程序入口分析

使用Ghidra分析`entry`函数 其调用链

```
entry (0x401462) → FUN_00401000 (main)
```

![e6bef18e67.jpg](/files/2026/1/18/e6bef18e67.jpg)

确认主函数00401000（传递HINSTANCE 赋值最后被退出函数调用）

#### 2.主函数FUN_00401000

![8ce5ed4822.jpg](/files/2026/1/18/8ce5ed4822.jpg)

注册窗口004010c0

![cdf5c654f3.jpg](/files/2026/1/18/cdf5c654f3.jpg)

**窗口过程函数是 `LAB_004011c0`**（也就是 `FUN_004011c0`）

![a6614a4004.jpg](/files/2026/1/18/a6614a4004.jpg)

找到对话框地址00401350

![d59bf496d5.jpg](/files/2026/1/18/d59bf496d5.jpg)

flag中间含有两个整型参数

函数逻辑大概是cnt==19999 输出flag

分析一下汇编

![c924c4bd91.jpg](/files/2026/1/18/c924c4bd91.jpg)

刚好PUSH两个整数参数 0和19999

### 得到flag

直接填入两个整数尝试flag 得出

```
NSSCTF{1999902069a45792d233ac}
```