**JSON（JavaScript Object Notation）** 是一种轻量级的数据交换格式，易于人阅读和编写，也易于机器解析和生成。它基于 JavaScript 语法，但独立于编程语言，几乎所有现代编程语言都支持 JSON。

### **JSON 核心结构**

#### 1. **键值对（Key-Value Pairs）**

```
"key": value
```

- 键：**必须是双引号包裹的字符串**（单引号无效）
- 值：可以是字符串、数字、布尔值、数组、对象或 `null`

#### 2. **数据类型**

| 类型   | 示例                               | 说明                       |
| :----- | :--------------------------------- | :------------------------- |
| 字符串 | `"name": "Alice"`                  | 双引号包裹                 |
| 数字   | `"age": 25`                        | 整数/浮点数，**不加引号**  |
| 布尔值 | `"isStudent": true`                | `true` 或 `false`（小写）  |
| 空值   | `"address": null`                  | `null`（小写）             |
| 数组   | `"hobbies": ["reading", "coding"]` | 有序列表，方括号 `[]` 包裹 |
| 对象   | `{ "person": { "name": "Bob" } }`  | 无序集合，花括号 `{}` 包裹 |

------

### **JSON 完整示例**

```
{
  "name": "张三",
  "age": 30,
  "isEmployed": true,
  "skills": ["Java", "Python", "SQL"],
  "contact": {
    "email": "zhangsan@example.com",
    "phone": "+86 13800138000"
  },
  "projects": [
    {
      "id": 1,
      "title": "Web App",
      "completed": true
    },
    {
      "id": 2,
      "title": "Mobile App",
      "completed": false
    }
  ],
  "address": null
}
```

------

### **JSON 语法规则**

1. **双引号强制**：键和字符串值必须使用 `" "`（单引号无效）

   ```
   ❌ 错误：{'key': 'value'}  
   ✅ 正确：{"key": "value"}
   ```

2. **无注释**：JSON 标准不支持注释（部分解析器扩展支持 `//` 或 `/* */`，但非标准）

3. **结尾无逗号**：最后一个元素后不能有逗号

   ```
   ❌ 错误：["A", "B",]  
   ✅ 正确：["A", "B"]
   ```

4. **严格数据类型**：`true`/`false`/`null` 必须小写

------

### **JSON 应用场景**

1. **API 数据传输**（如 RESTful API）
2. **配置文件**（如 `package.json`、`tsconfig.json`）
3. **NoSQL 数据库**（如 MongoDB 文档存储）
4. **前后端数据交换**

------

### **编程语言操作示例**

#### Python 解析 JSON

```
import json

# JSON字符串 → Python字典
json_str = '{"name": "Alice", "age": 25}'
data = json.loads(json_str)
print(data["name"])  # 输出: Alice

# Python字典 → JSON字符串
new_data = {"job": "Engineer", "active": True}
json_str = json.dumps(new_data)
```

#### JavaScript 解析 JSON

```
// JSON字符串 → JS对象
const jsonStr = '{"name": "Bob", "score": 95}';
const obj = JSON.parse(jsonStr);
console.log(obj.score);  // 输出: 95

// JS对象 → JSON字符串
const newObj = { id: 101, valid: false };
const jsonStr = JSON.stringify(newObj);
```