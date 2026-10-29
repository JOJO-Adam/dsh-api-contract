---
name: openapi-authoring
description: "OpenAPI authoring playbook: resource-oriented path design, the reusable component pattern, request and response conventions, and the anti-patterns that quietly become future breaking changes."
version: "1.0.0"
display_name: "OpenAPI 编写"
display_name_en: "OpenAPI Authoring Playbook"
description_zh: "OpenAPI 编写手册：资源导向的路径设计、可复用 components 模式、请求与响应约定，以及那些会悄悄变成未来破坏性变更的反模式。"
description_en: "OpenAPI authoring: resource-oriented paths, reusable components, request/response conventions, anti-patterns that become breaking changes."
whenToUse: "写 OpenAPI / swagger、设计路径与 components、定请求响应约定时"
---

# OpenAPI 编写手册

## 一 · 路径设计：以资源为中心

```
✅ GET /orders/{id} · GET /orders · POST /orders · PATCH /orders/{id} · DELETE /orders/{id}
❌ GET /getOrderById · POST /createOrder · POST /updateStatus
```

无法用资源表达的动作 → 子资源或动作端点（`/orders/{id}/cancel`），但要少而一致。

## 二 · 组件复用（避免五处定义同一实体）

```yaml
components:
  schemas:
    User:
      type: object
      required: [id, name]
      properties:
        id: { type: string, example: "u_123" }
        name: { type: string, example: "张三" }
        email: { type: string, format: email, nullable: true, example: "zhangsan@example.com" }
    Error:
      type: object
      required: [code, message]
      properties:
        code: { type: string, example: "ORDER_NOT_FOUND" }
        message: { type: string, example: "订单不存在" }
        traceId: { type: string }
```

所有接口 `$ref`，不复制粘贴。

## 三 · 请求/响应三条约定

**① 列表不返回裸数组**
```yaml
# ❌ 以后想加分页/总数就是破坏性变更
schema: { type: array, items: { $ref: '#/components/schemas/Order' } }
# ✅ 留了扩展位
schema:
  type: object
  properties:
    items: { type: array, items: { $ref: '#/components/schemas/Order' } }
    total: { type: integer }
    nextCursor: { type: string, nullable: true }
```

**② 不暴露内部实现**
不要返回数据库主键、内部状态枚举、内部服务名、堆栈。

**③ 必填要真实**
文档写可选、实现里必填 = 契约谎言，比缺文档更糟。

## 四 · 会变成未来破坏性变更的反模式

| 反模式 | 为什么以后会痛 |
|---|---|
| 返回裸数组 | 加分页就是破坏性 |
| 用 `status: 1/2/3` 数字码而不给枚举 | 无法推断含义，改语义没人知道 |
| 返回内部主键当业务 ID | 换 ID 策略就全崩 |
| 时间戳用 number | 时区/精度问题，换格式破坏性 |
| 布尔字段不用 `is/has` 前缀 | 语义歧义 |
| 错误格式随手写 | 各接口不一致，客户端难统一 |

## 五 · 命名与格式清单

- [ ] 全项目一种命名风格
- [ ] 时间统一 ISO 8601 带时区
- [ ] 金额带单位或币种（`amountCents` 优于 `amount`）
- [ ] 枚举值不用中文/拼音
- [ ] 布尔用 `is_/has_/can_`
- [ ] 资源名用复数

## 六 · 示例要真

```yaml
# ❌
name: { type: string, example: "string" }
# ✅
name: { type: string, example: "张三" }
```

详见 `doc-examples`。

## 七 · 红线

- 不返回裸数组
- 不设计会暴露内部实现的字段
- 不给 `example: "string"` 这类占位符
- 命名风格必须一致，发现不一致要指出
