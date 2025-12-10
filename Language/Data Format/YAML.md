YAML（YAML Ain't Markup Language）是一种人类友好的数据序列化标准，主要用于配置文件和数据交换，其**设计目标**是易读易写，比 JSON 和 XML 更简洁。

## 核心语法

1. **基本结构**：
	- 使用缩进表示层次关系（空格，非 Tab）
	- 注释以 `#` 开头
	- 文件通常以 `.yaml` 或 `.yml` 结尾
2. **基本数据类型**：
	- 字符串：`"Hello"`，或不用引号
	- 整数：`42`
	- 浮点数：`3.14`
	- 布尔值：`true`
	- null：`null`，或 `~`
3. **集合类型**：
```yaml
# 对象/字典
person:
  name: "John"
  age: 30
```

```yaml
# 数组/列表
colors:
  - red
  - green
  - blue
```

```yaml
# 内联写法
person: {name: "John", age: 30}
colors: [red, green, blue]
```

4. **多行字符串**：
```yaml
description: |
  这是多行文本
  保留换行符

folded_text: >
  折叠多行文本
  合并为单行空格
```

5. **特殊功能**：
```yaml
# 锚点与别名（避免重复）
defaults: &defaults
  adapter: postgres
  host: localhost

development:
  <<: *defaults  # 合并锚点内容
  database: dev_db

# 类型强制转换
str_as_int: !!str 123  # 转为字符串
is_true: !!bool "yes"  # 转为布尔值
```

6. **文档分隔**：
```yaml
# 单个文件包含多个文档
---
document1: "第一个文档"
...
---
document2: "第二个文档"
```

## 注意事项

1. **缩进必须一致**（通常2个空格）
2. **字符串引号可选**，但特殊字符需引号
3. **键值对用冒号+空格**分隔
4. **列表项用短横线+空格**开头
5. YAML 1.2 是 JSON 的超集