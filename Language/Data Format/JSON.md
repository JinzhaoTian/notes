JSON（JavaScript Object Notation）是一种轻量级的数据交换格式，易于人阅读和编写，同时也易于机器解析和生成。它基于 JavaScript 的一个子集，但独立于编程语言，被广泛用于 Web 应用和 API 的数据传输。

## 主要特点

1. **键值对结构**：数据以`键: 值`的形式组织，类似字典或哈希表。
2. **支持的数据类型**：
    - 字符串（必须用双引号`"`）
    - 数字（整数或浮点数）
    - 布尔值（`true`/`false`）
    - 数组（`[]`，有序列表）
    - 对象（`{}`，键值对的集合）
    - `null`（空值）
3. **独立于语言**：虽然源自 JavaScript，但几乎所有编程语言都支持 JSON 解析和生成。


## JSONSchema

> 参考：[官网](http://json-schema.org/)

JSON Schema定义了一套词汇和规则，这套词汇和规则用来定义 JSON 元数据，且元数据也是通过 JSON 数据形式表达的。JSON 元数据定义了 JSON 数据需要满足的规范，规范包括成员、结构、类型、约束等。

对于 API 来说，需要定义什么样的请求合法，即什么样的 JSON 对于 API 来说是合法的输入。这个规范可以通过 JSON Schema 来描述，对应的 JSON Schema 如下：

```json
{ 
    "type": "object",
    "properties": {
        "city": { "type": "string" },
        "number": { "type": "number" },
        "user": { 
            "type": "object",
            "properties": {
                "name" : {"type": "string"},
                "age" : {"type": "number"}
            }                       
        }
    }
}
```



### 类型关键字

`type`关键字，这个关键字定义了 JSON 数据需要满足的类型要求。如：`{"type" : "object"}`，`{"type" : "string"}`，

`{"type" : "number"}`，`{"type" : "integer"}`，`{"type" : "array"}`，`{"type" : "boolean"}`，`{"type" : "null"}`。



### 简单类型

包括字符串、数值、布尔、空值几种基本类型。

* **字符串**：Json合法的字符串：`"Today is a good day."`，对应的Json Schema：`{"type": "string"}`。
  
  * 字符串长度关键字： minLength, maxLength，如：
  
    ```json
    {
        "type" : "string",
        "minLength" : 2,
        "maxLength" : 3,
    }
    ```
  
  * 正则表达式关键字： pattern，如：
  
    ```json
    {
        "type" : "string",
        "pattern" : "^(\\([0-9]{3}\\))?[0-9]{3}-[0-9]{4}$",
    }
    ```
  
  * 字符串格式关键字： format，如：
  
    ```json
    {
       "type" : "string",
       "format" : "date",
    }
    ```
  
* **数值**：Json Schema数值类型包括"number"和"integer"。number合法的数值可以是：`2`, `0.1`，对应的Json Schema为：`{"type": "number"}`。integer则只能是整数。"number"和"integer"的类型特定参数相同，可以限制倍数、范围。

  * 数值满足倍数关键字： multipleOf，要求数值必须某个特定值的整数倍，如：

    ```json
    {
        "type" : "number",
        "multipleOf" : 10,
    }
    ```

  * 数值范围关键字： minimum最小值, maximum最大值, exclusiveMinimum开区间最小值, exclusiveMaximum开区间最大值。如：要求数值在[0, 100)范围内

    ```json
    {
        "type" : "number",
        "minimum": 0,
        "exclusiveMaximum": 100
    }
    ```

* **布尔**：布尔类型没有额外的类型特定参数。

* **空值**：null类型没有额外的类型特定参数。



### 复合类型

复合类型可以通过Nest的方式构建复杂的数据结构，包括数组、对象。

* 数组：Json数组合法数据如：`[1, 2, 3]`,`[1, "abc", {"name" : "alex"}]`, `[]`, 对应的Json Schema为：`{"type": "array"}`

  * 数组成员类型关键字： items,可以要求数组内每个成员都是某种类型，如：

    ```json
    {
        "type": "array",
        "items": {
            "type": "number"
        }
    }
    ```


    关键字items还可以对应一个数组，这时Json数组内的元素必须与Json Schema内items数组内的每个Schema按位置一一匹配。

    ```json
    {
        "type": "array",
        "items": [
        {
            "type": "number"
        },
        {
            "type": "string"
        }]
    }
    ```

    可以匹配为：`[1, "abc"]`

  * 数组允许额外成员关键字： additionalItems，当**使用了items关键字，并且items关键字对应的是Schema数组**，这个限制才起作用。关键字additionalItems规定Json数组内的元素，除了一一匹配items数组内的Schema外，是否还允许多余的元组。当additionalItems为true时，允许额外元素。 

    ```json
    {
        "type": "array",
        "items": [
        {
            "type": "number"
        },
        {
            "type": "string"
        }],
        "additionalItems" : true
    }
    ```

    可匹配的为：`[1, "abc", "x"]`

  *  数组元素个数关键字： minItems, maxItems,可以限制数组内元素的个数

    ```json
    {
        "type": "array",
        "items": {
            "type": "number"
        },
        "minItems" : 5,
        "maxItems" : 10
    }
    ```

  *  数组内元素必须唯一关键字： uniqueItems,规定数组内的元素是否必须唯一。

    ```json
    {
        "type": "array",
        "items": {
            "type": "number"
        },
        "uniqueItems" : true
    }
    ```

    

* 对象：Json对象是最常见的Json数据类型，合法的数据可以是`{"name":"Froid", "age":26, "address":{"city":"New York", "country":"USA"} }`,类型限制Schema是`{"type" : "object"}`.

  * 成员的Schema关键字：properties,规定对象各成原所应遵循的Schema

    ```json
    { 
        "type": "object",     
        "properties": {      
            "name": {"type" : "string"},
            "age" : {"type" : "integer"},
            "address" : {
                "type" : "object",
                "properties" : {
                    "city" : {"type" : "string"},
                    "country" : {"type" : "string"}
                }
            }
        }
    }
    ```

    可匹配的合法的data是`{ "name":"Froid", "age":26, "address":{ "city":"New York", "country":"USA" } }`。properties关键字的内容是一个key/value结构的字典，其key对应Json数据中的key，其value是一个嵌套的Json Schema。表示Json数据中key对应的值所应遵守的Json Schema。在上面的例子中，"name"对应的Schema是`{"type" : "string"}`，表示"name"的值必须是一个字符串。在Json数据中，对象可以嵌套，同样在Json Schema中也可以嵌套。如"address"字段，在Json Schema中它的内容是一个嵌套的object类型的Json Schema。

  * 批量定义成员关键字：patternProperties，与properties一样，但是key通过正则表达式匹配属性名。

    ```json
    {
        "type": "object",
        "patternProperties": {
            "^S_": { "type": "string" },
            "^I_": { "type": "integer" }
        }
    }
    ```

    可以匹配为：`{"S_1" : "abc"}`

  * 必须出现的成员关键字：required，规定哪些对象成员是必须的。

    ```json
    { 
        "type": "object",     
        "properties": {      
            "name": {"type" : "string"},
            "age" : {"type" : "integer"},
        },
        "required" : ["name"]
    }
    ```

    "name"成员是必须的，因此合法的数据可以是：`{"name" : "mary", "age" : 26}`

  *  成员依赖关系关键字：dependencies，规定某些成员的依赖成员，不能在依赖成员缺席的情况下单独出现，属于数据完整性方面的约束。

    ```json
    {
        "type": "object",
        "dependencies": {
            "credit_card": ["billing_address"]
        }
    }
    ```

    如果有"credit_card"属性，则"billing_address" 属性不能缺席。

  * 允许额外属性关键字：additionaProperties，规定object类型是否允许出现**不在properties中规定的**属性，只能取true/false。

    ```json
    { 
        "type": "object",     
        "properties": {      
            "name": {"type" : "string"},
            "age" : {"type" : "integer"},
        },
        "required" : ["name"],
        "additionalProperties" : false
    }
    ```

    

  * 属性个数的限制关键字：minProperties, maxProperties，规定最少、最多有几个属性成员

    ```json
    {
        "type": "object",
        "minProperties": 2,
        "maxProperties": 3
    }
    ```

    

  * 



### 逻辑组合

关键字：allOf, anyOf, oneOf, not。从关键字名字可以看出其含义，满足所有、满足任意、满足一个。前三个关键字的使用形式是一致的，以allOf为例说明其形式。

```json
{
    "allOf" : [
        Schema1,
        Schema2,
        ...
    ]
}
```

其中，"allOf"的内容是一个数组，数组内的成员都是内嵌的Json Schema。上例Schema1、Schema2都是内嵌的Json Schema。整个Schema表示当前Json数据，需要同时满足Schema1、Schema2。

### 复杂结构

包括定义和引用。可以将相同的结构定义成一个“类型”，需要使用该“类型”时，可以通过其路径或id来引用。

### 通用关键字

* enum：关键字：enum，可以在任何json schema中出现，其value是一个list，表示json数据的取值只能是list中的某个。

  ```json
  {
      "type": "string",
      "enum": ["red", "amber", "green"]
  }
  ```

* metadata：关键字：title，description，default，example，只作为描述作用，不影响对数据的校验。

  ```json
  {
      "title" : "Match anything",
      "description" : "This is a schema that matches anything.",
      "default" : "Default value",
      "examples" : [
          "Anything",
          4035
      ]
  }
  ```



