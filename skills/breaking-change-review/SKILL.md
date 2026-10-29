---
name: breaking-change-review
description: "Breaking change review playbook: the full classification table with the exact client failure mechanism for each change type, the unknown-field caveat, and a verified detection output."
version: "1.0.0"
display_name: "破坏性变更审查"
display_name_en: "Breaking Change Review"
description_zh: "破坏性变更审查手册：完整分类表（每类都给出客户端具体坏在哪）、未知字段的兼容性例外，以及一次真实的检测输出。"
description_en: "Breaking change review: classification table with exact client failure mechanism, unknown-field caveat, verified detection output."
whenToUse: "评审破坏性变更、判断这次改接口会不会伤到客户端、上线前做兼容性检查时"
---

# 破坏性变更审查手册

## 一 · 判定表（机制层面，不靠感觉）

| 变更 | 判定 | 客户端具体坏在哪 |
|---|---|---|
| 删除响应字段 | 🔴 | 读该字段的代码得到 undefined/null，可能 NPE |
| 字段类型变化 | 🔴 | 反序列化失败，或后续方法调用崩（`.toUpperCase()` on number） |
| 新增必填请求参数 | 🔴 | 老客户端不传 → 400 |
| 新增必填响应字段 | 🔴 | 客户端无处理逻辑，空值可能崩 |
| 删除路径/方法 | 🔴 | 404 / 405 |
| 参数可选→必填 | 🔴 | 老调用缺参 → 400 |
| 删除枚举值 | 🔴 | 客户端 switch 缺分支 → 落到 default 或抛错 |
| 收紧校验（maxLength 变小、正则变严） | 🔴 | 原合法请求被拒 |
| 新增可选请求参数 | 🟢 | 老客户端不传，服务端用默认 |
| 新增响应字段 | 🟡 | **通常**兼容，但严格 schema 校验的客户端会失败 |
| 参数必填→可选 | 🟢 | 放宽 |
| 放宽校验 | 🟢 | — |
| 新增路径/方法 | 🟢 | 老客户端不会调用 |

## 二 · 那个 🟡 必须解释清楚

**「新增响应字段」不是绝对安全。**

会失败的客户端类型：
- 严格反序列化（如 Jackson `FAIL_ON_UNKNOWN_PROPERTIES=true`）
- 对响应做 JSON Schema 严格校验（`additionalProperties: false`）
- 生成的强类型客户端且 schema 校验严格

**正确表述**：
> 「新增字段通常兼容，因为多数客户端忽略未知字段；但如果你的客户端开启了严格模式或做了 schema 校验，需要先确认。」

**错误表述**：
> ❌ 「加个字段肯定没事」

## 三 · 实测检测输出（真跑过 · 正反双向验证）

### 正向：8 类变更 → 全部命中

构造一份含 8 类变更的 spec 对（v1→v2：参数类型改动、字段删除、字段必填化、收紧 maxLength、路径删除、新增可选参数、新增方法、新增字段），检测输出：

```
检出 9 项：破坏性 6 · 兼容 3

🔴 BREAKING  [参数类型] GET /users/{id} `id`: string → integer，客户端传值可能被拒
🔴 BREAKING  [字段类型] GET /users/{id} `id`: string → integer，反序列化可能失败
🔴 BREAKING  [收紧校验] GET /users/{id} `name` maxLength 50 → 10，原合法值会被拒
🔴 BREAKING  [字段删除] GET /users/{id} 响应字段 `email` 消失，客户端读取得到 undefined/None
🔴 BREAKING  [必填化] GET /users/{id} 响应 `nickname` 新增为 required，客户端可能没处理空值
🔴 BREAKING  [路径删除] /orders/{id} 整个接口消失，调用方 404
🟢 ok        [新增可选参数] GET /users/{id} `verbose`
🟢 ok        [新增字段] GET /users/{id} `nickname`（未知字段多数客户端可忽略，严格模式除外）
🟢 ok        [新增方法] POST /items
```

（`id` 类型变更报了两次——参数侧和响应字段侧各一次，**这是正确行为**，因为两边客户端都可能坏。）

### 反向：确认不误报

| 测试 | 预期 | 实测 |
|---|---|---|
| v1 与 v1 自比 | 0 检出 | ✅ 0 项，exit=0 |
| 只加可选参数 + 新路径 | 破坏性 0，放行 | ✅ 破坏性 0，exit=0 |

**这两条比正向更重要**：一个只会报"破坏性"的检测器等于噪音，**能正确放行兼容变更**才有用。

### 结论
判定表**可以完整落地成可执行检查**（对比两份 spec 的 paths → methods → parameters → response schema → properties/required/enum/maxLength），且经正反双向验证：不漏报、不误报、退出码可用于 CI 拦截。


## 四 · 审查输出格式

```
## 结论
破坏性 N 项 / 兼容 M 项

## 逐项
| 变更 | 判定 | 机制说明 |

## 影响面
哪类客户端、坏在哪一步

## 建议
兼容做法 或 版本升级 + 公告期
```

## 五 · 红线

- 不给「应该没事」「一般兼容」这类无机制的解释
- 不把「新增响应字段」咬定 100% 安全
- 判定必须基于用户真实 spec；没材料只能说通用规则，不能假装看过
- 判为破坏性后必须给迁移路径
