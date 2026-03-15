# JQL

# JQL 语法速查表

## 一、基础查询运算符

### 1. **比较运算符**

| 运算符   | 描述             | 示例                                 |
| -------- | ---------------- | ------------------------------------ |
| `=`      | 等于             | `project = "APP"`                    |
| `!=`     | 不等于           | `status != "Done"`                   |
| `>`      | 大于             | `priority > "Medium"`                |
| `>=`     | 大于等于         | `votes >= 5`                         |
| `<`      | 小于             | `created < -30d`                     |
| `<=`     | 小于等于         | `due <= endOfWeek()`                 |
| `IN`     | 在列表中         | `status IN ("Open", "In Progress")`  |
| `NOT IN` | 不在列表中       | `assignee NOT IN ("user1", "user2")` |
| `~`      | 包含             | `summary ~ "登录"`                   |
| `!~`     | 不包含           | `description !~ "测试"`              |
| `IS`     | 是（用于空值）   | `assignee IS EMPTY`                  |
| `IS NOT` | 不是（用于空值） | `due IS NOT EMPTY`                   |

### 2. **逻辑运算符**

| 运算符 | 描述     | 示例                                           |
| ------ | -------- | ---------------------------------------------- |
| `AND`  | 与       | `status = "Open" AND assignee = currentUser()` |
| `OR`   | 或       | `type = Bug OR priority = "High"`              |
| `NOT`  | 非       | `NOT status = "Closed"`                        |
| `()`   | 括号分组 | `(A AND B) OR (C AND D)`                       |

## 二、常用字段查询语法

### 1. **文本字段搜索**

| 模式          | 语法                                    | 说明                   |
| ------------- | --------------------------------------- | ---------------------- |
| 单个关键字    | `summary ~ "bug"`                       | 包含"bug"              |
| 多关键字(AND) | `summary ~ "登录" AND summary ~ "失败"` | 同时包含"登录"和"失败" |
| 多关键字(OR)  | `summary ~ "登录" OR summary ~ "注册"`  | 包含"登录"或"注册"     |
| 排除关键字    | `summary !~ "测试"`                     | 不包含"测试"           |
| 完整短语      | `summary ~ "\"用户登录失败\""`          | 包含完整短语           |
| 通配符        | `summary ~ "bug*"`                      | 以"bug"开头            |
| 通配符结尾    | `summary ~ "*error"`                    | 以"error"结尾          |
| 通配符两边    | `summary ~ "*auth*"`                    | 包含"auth"             |

### 2. **日期字段搜索**

| 模式     | 语法                                 | 说明             |
| -------- | ------------------------------------ | ---------------- |
| 绝对日期 | `created >= "2024-01-01"`            | 2024年1月1日之后 |
| 相对日期 | `created >= -7d`                     | 最近7天内        |
| 时间函数 | `due <= endOfWeek()`                 | 本周结束前       |
| 日期范围 | `created >= -30d AND created <= -7d` | 7天前到30天前    |

## 三、JQL 内置函数

### 1. **日期函数**

| 函数             | 描述     | 示例                        |
| ---------------- | -------- | --------------------------- |
| `now()`          | 当前时间 | `created >= now()`          |
| `startOfDay()`   | 当天开始 | `created >= startOfDay()`   |
| `endOfDay()`     | 当天结束 | `due <= endOfDay()`         |
| `startOfWeek()`  | 本周开始 | `created >= startOfWeek()`  |
| `endOfWeek()`    | 本周结束 | `due <= endOfWeek()`        |
| `startOfMonth()` | 本月开始 | `created >= startOfMonth()` |
| `endOfMonth()`   | 本月结束 | `due <= endOfMonth()`       |
| `startOfYear()`  | 本年年初 | `created >= startOfYear()`  |
| `endOfYear()`    | 本年年末 | `due <= endOfYear()`        |

### 2. **用户函数**

| 函数             | 描述         | 示例                                  |
| ---------------- | ------------ | ------------------------------------- |
| `currentUser()`  | 当前用户     | `assignee = currentUser()`            |
| `membersOf()`    | 组成员       | `assignee IN membersOf("developers")` |
| `currentLogin()` | 当前登录用户 | `reporter = currentLogin()`           |

### 3. **其他函数**

| 函数                      | 描述           | 示例                                           |
| ------------------------- | -------------- | ---------------------------------------------- |
| `latestReleasedVersion()` | 最新发布版本   | `fixVersion = latestReleasedVersion()`         |
| `projectsLeadByUser()`    | 用户负责的项目 | `project IN projectsLeadByUser(currentUser())` |

## 四、多关键字组合模式

| 需求                 | JQL 语法                                                    | 示例                                                         |
| -------------------- | ----------------------------------------------------------- | ------------------------------------------------------------ |
| **必须全部包含**     | `text ~ "A" AND text ~ "B" AND text ~ "C"`                  | `summary ~ "登录" AND summary ~ "失败" AND summary ~ "Android"` |
| **包含任意一个**     | `text ~ "A" OR text ~ "B" OR text ~ "C"`                    | `text ~ "崩溃" OR text ~ "错误" OR text ~ "异常"`            |
| **包含A和B，排除C**  | `text ~ "A" AND text ~ "B" AND text !~ "C"`                 | `text ~ "支付" AND text ~ "功能" AND text !~ "测试"`         |
| **(A或B) 且 (C或D)** | `(text ~ "A" OR text ~ "B") AND (text ~ "C" OR text ~ "D")` | `(text ~ "登录" OR text ~ "注册") AND (text ~ "失败" OR text ~ "错误")` |
| **精确短语+关键字**  | `text ~ "\"完整短语\"" AND text ~ "关键字"`                 | `summary ~ "\"用户登录失败\"" AND description ~ "Android"`   |

## 五、字段别名速查

| JQL 字段          | 常见别名 | 示例                         |
| ----------------- | -------- | ---------------------------- |
| `project`         | 项目     | `project = "APP"`            |
| `type`            | 类型     | `type = Bug`                 |
| `status`          | 状态     | `status = "In Progress"`     |
| `assignee`        | 经办人   | `assignee = currentUser()`   |
| `reporter`        | 报告人   | `reporter = "john.doe"`      |
| `priority`        | 优先级   | `priority = "High"`          |
| `resolution`      | 解决结果 | `resolution = "Fixed"`       |
| `created`         | 创建时间 | `created >= -7d`             |
| `updated`         | 更新时间 | `updated >= -1d`             |
| `due`             | 到期时间 | `due IS NOT EMPTY`           |
| `labels`          | 标签     | `labels = "urgent"`          |
| `component`       | 组件     | `component = "API"`          |
| `fixVersion`      | 修复版本 | `fixVersion = "v1.2.0"`      |
| `affectedVersion` | 影响版本 | `affectedVersion = "v1.1.0"` |

## 六、常用查询模板

### 1. **我的待办事项**

```
assignee = currentUser() AND 
status NOT IN ("Done", "Closed") AND 
type IN (Bug, Task, Story)
ORDER BY priority DESC, created DESC
```

### 2. **紧急 Bug 搜索**

```
project = "APP" AND 
type = Bug AND 
priority = "High" AND 
status NOT IN ("Done", "Closed") AND 
text ~ "崩溃" AND created >= -7d
ORDER BY created DESC
```

### 3. **本周到期任务**

```
assignee = currentUser() AND 
due IS NOT EMPTY AND 
due <= endOfWeek() AND 
status NOT IN ("Done", "Closed")
ORDER BY due ASC
```

### 4. **需要我评审的**

```
status = "In Review" AND 
reporter != currentUser() AND 
text ~ "review" AND 
project IN projectsLeadByUser(currentUser())
ORDER BY created DESC
```

## 七、排序和分页

### 1. **排序语法**

| 语法                  | 描述         | 示例                                   |
| --------------------- | ------------ | -------------------------------------- |
| `ORDER BY field`      | 升序排列     | `ORDER BY created`                     |
| `ORDER BY field DESC` | 降序排列     | `ORDER BY priority DESC`               |
| 多字段排序            | 多个字段排序 | `ORDER BY priority DESC, created DESC` |

### 2. **常用排序字段**

| 排序字段   | 用途          | 示例                     |
| ---------- | ------------- | ------------------------ |
| `created`  | 按创建时间    | `ORDER BY created DESC`  |
| `updated`  | 按更新时间    | `ORDER BY updated DESC`  |
| `priority` | 按优先级      | `ORDER BY priority DESC` |
| `due`      | 按到期时间    | `ORDER BY due ASC`       |
| `votes`    | 按投票数      | `ORDER BY votes DESC`    |
| `key`      | 按 Issue 编号 | `ORDER BY key DESC`      |

### 3. **分页控制**

| 参数         | 描述     | 示例 URL         |
| ------------ | -------- | ---------------- |
| `startAt`    | 起始位置 | `&startAt=50`    |
| `maxResults` | 每页数量 | `&maxResults=25` |

## 八、性能优化提示

| 建议                 | 好示例                               | 差示例                                          |
| -------------------- | ------------------------------------ | ----------------------------------------------- |
| **先过滤项目**       | `project = "APP" AND text ~ "bug"`   | `text ~ "bug" AND project = "APP"`              |
| **使用索引字段**     | `created >= -7d AND text ~ "紧急"`   | `text ~ "紧急" AND created >= -7d`              |
| **避免\*通配符开头** | `summary ~ "bug*"`                   | `summary ~ "*bug"`                              |
| **限制日期范围**     | `created >= -30d AND text ~ "error"` | `text ~ "error"`                                |
| **使用IN替代多个OR** | `status IN ("Open", "In Progress")`  | `status = "Open" OR status = "In Progress"`     |
| **避免过多OR**       | 使用其他字段过滤                     | `text ~ "a" OR text ~ "b" OR ... OR text ~ "z"` |

## 九、错误排查

| 错误信息                                     | 原因                  | 解决方案                 |
| -------------------------------------------- | --------------------- | ------------------------ |
| `The value 'xxx' does not exist`             | 字段值不存在          | 检查字段值拼写和大小写   |
| `Field 'xxx' cannot be used in this context` | 字段不允许在JQL中使用 | 使用其他字段或联系管理员 |
| `The operator '~' is not supported`          | 字段不支持文本搜索    | 使用其他字段或`CONTAINS` |
| `Cannot parse date 'xxx'`                    | 日期格式错误          | 使用标准格式`YYYY-MM-DD` |
| `Too many results`                           | 结果过多              | 添加更多过滤条件或分页   |

## 十、快速参考卡片

### **基础格式**：

```
字段 运算符 值 [AND|OR 字段 运算符 值...] [ORDER BY 字段 [DESC]]
```

### **多关键字搜索**：

```
summary ~ "A" AND summary ~ "B" AND summary !~ "C"
```

### **组合查询**：

```
project = "APP" AND 
status NOT IN ("Done", "Closed") AND 
(assignee = currentUser() OR reporter = currentUser()) AND 
created >= -30d
ORDER BY priority DESC, created DESC
```

### **保存为过滤器**：

```
-- 命名建议：我的_未完成_高优先级任务
assignee = currentUser() AND 
status NOT IN ("Done", "Closed") AND 
priority = "High"
```

------

**记忆口诀**：

- 

  等于`=`，包含`~`，否定加`!`

- 

  `AND`且，`OR`或，括号定顺序

- 

  日期用函数，用户用`currentUser()`

- 

  排序加`ORDER BY`，分页加`startAt`

- 

  通配符用`*`，短语用`"引号"`

掌握这些核心语法，您就可以高效使用JQL进行问题搜索和过滤了！