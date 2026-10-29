---
name: doc-examples
description: "Documentation and examples playbook: doc-from-spec principle, four levels of example quality, the damage done by placeholder values, and the six things examples must cover beyond happy path."
version: "1.0.0"
display_name: "文档与示例"
display_name_en: "Documentation & Examples Playbook"
description_zh: "文档与示例手册：文档从契约生成为原则、示例质量的四个层次、占位符的三个害处，以及示例之外必须写清的六件事。"
description_en: "Documentation and examples: doc-from-spec, four example quality levels, placeholder damage, six must-cover items."
whenToUse: "写接口文档、补示例、判断示例质量够不够买家看懂时"
---

# 文档与示例手册

## 一 · 原理：手写文档必然过期

> 「任何手工维护的文档，在代码变更的一瞬间就已经过期。」

**原则**：文档从契约生成；手写的只保留契约表达不了的部分（业务背景、迁移说明、设计取舍）。

## 二 · 示例的四个层次

| 层次 | 形态 | 价值 |
|---|---|---|
| ❌ 占位符 | `"name": "string"` | 近乎零 |
| 🟡 类型示例 | `"name": "张三"` | 能看懂含义 |
| 🟢 业务流程 | 登录 → 拿 token → 查订单 完整链路 | 新人能跑通 |
| 🟢🟢 可执行 | 直接 curl 复制即跑通 | 最高 |

**目标**：至少第三层，核心接口争取第四层。

## 三 · 占位符的三个害处

1. 看不出真实数据形态（长度/格式/边界）
2. 复制粘贴直接报错
3. 掩盖了「这个字段允许 null」这类关键信息

```yaml
# ❌
email: { type: string, example: "string" }
# ✅
email: { type: string, format: email, example: "zhangsan@example.com" }
```

## 四 · 可执行示例的心智

好示例必须**自带前置条件**：

```bash
# 1. 登录拿 token（前置步骤，不能跳过）
curl -X POST https://api.example.com/v1/auth/login \
  -H 'Content-Type: application/json' \
  -d '{"username":"demo","password":"<your-password>"}'
# → {"token":"eyJ...","expiresIn":3600}

# 2. 带 token 调用
curl https://api.example.com/v1/orders/123 \
  -H 'Authorization: Bearer <token>'
```

要点：
- 有前置步骤
- 有真实形态的响应（不是 `{"data":{}}`）
- **凭据必须占位**（`<your-password>`），不能写真值

## 五 · 示例之外必须写清的六件事

1. **认证方式**：怎么拿 token、放哪、过期怎么办
2. **错误长什么样**：统一错误结构（见 `error-contract`）
3. **分页怎么翻**：offset 还是 cursor，游标从哪来
4. **限流规则**：多少 QPS、超了返回什么
5. **幂等性**：哪些接口可重试、靠什么保证
6. **版本与弃用策略**

## 六 · 交付格式

```
示例（可直接跑）
前置条件（认证/环境）
常见错误与处理
一致性说明（示例是否由 spec 推导）
```

## 七 · 红线

- 不给 `"example": "string"` 占位符
- 示例凭据必须占位
- 不写跑不通的示例；无法验证要标「未实测」
- 不手工维护本可从 spec 生成的部分
