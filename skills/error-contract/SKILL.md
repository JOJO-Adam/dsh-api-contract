---
name: error-contract
description: "Error contract playbook: the unified error shape, HTTP status versus business code separation, three anti-patterns including 200-for-business-failure, and stable error code naming."
version: "1.0.0"
display_name: "错误契约"
display_name_en: "Error Contract Playbook"
description_zh: "错误契约手册：统一错误结构、HTTP 状态码与业务错误码的分工、三种反模式（含业务失败返 200），以及稳定的错误码命名规范。"
description_en: "Error contract playbook: unified error shape, HTTP status vs business code, three anti-patterns, stable code naming."
whenToUse: "设计错误码、统一错误结构、纠结业务失败该返 200 还是 4xx 时"
---

# 错误契约手册

## 一 · 统一错误结构

```json
{
  "code": "ORDER_ALREADY_PAID",
  "message": "该订单已支付，不能重复支付",
  "traceId": "a1b2c3d4",
  "details": [{ "field": "orderId", "issue": "ALREADY_PAID" }]
}
```

| 字段 | 作用 | 稳定性 |
|---|---|---|
| `code` | 机器判断用 | **必须稳定**（改了就是破坏性） |
| `message` | 人看 | 可改、可国际化 |
| `traceId` | 对日志 | — |
| `details` | 字段级错误 | 结构稳定 |

🔴 **客户端只能判断 `code`，不能判断 `message` 文本** —— 文案一改客户端就崩，是最常见的隐性破坏。

## 二 · HTTP 状态码 vs 业务 code

| 场景 | HTTP | 业务 code |
|---|---|---|
| 参数格式错 | 400 | `VALIDATION_FAILED` |
| 未登录 | 401 | `UNAUTHORIZED` |
| 无权限 | 403 | `FORBIDDEN` |
| 资源不存在 | 404 | `NOT_FOUND` |
| 业务规则拒绝 | 409 / 422 | `INSUFFICIENT_BALANCE` |
| 限流 | 429 | `RATE_LIMITED` |
| 服务端异常 | 500 | `INTERNAL_ERROR` |

**分工**：HTTP 码说"请求层面发生了什么"，业务 code 说"业务上为什么不行"。**两层都要有。**

## 三 · 三种必须避免的反模式

### ① 业务失败返回 200
```json
❌ HTTP 200 {"success": false, "msg": "余额不足"}
```
后果：监控看不到失败率、网关重试策略失效、客户端要判断两层。
✅ 用 4xx + 业务 code。

### ② 各接口错误结构不同
`{error:"..."}` / `{code,msg}` / 纯字符串混用 → 客户端写不出统一处理。
✅ 全项目一个错误 schema，`$ref` 复用。

### ③ 泄露内部信息
```json
❌ {"message":"SQLSTATE[23000]: Integrity constraint violation: users.email"}
❌ {"message":"at com.foo.Bar.baz(Bar.java:42)"}
```
后果：暴露表结构与堆栈，安全隐患。
✅ 对外说人话，细节进日志（用 traceId 关联）。

## 四 · 错误码命名

```
<领域>_<原因>
ORDER_NOT_FOUND
ORDER_ALREADY_PAID
PAYMENT_INSUFFICIENT_BALANCE
AUTH_TOKEN_EXPIRED
```

- 全大写 + 下划线
- **一旦发布就是契约，不能改**
- 不复用同一 code 表达不同原因
- 不用纯数字码（除非团队已有强约定）

## 五 · 分页与列表的错误也要统一

列表接口的 `cursor` 失效、`page` 越界，也要走同一错误结构，不要返回空数组糊过去。

## 六 · 交付格式

```
统一错误 schema（YAML）
错误码清单（按领域分组）
各状态码适用场景
客户端该怎么判断（强调判断 code 不判断 message）
```

## 七 · 红线

- 不让业务失败返 200
- 不让错误泄露 SQL/堆栈/表名
- 不设计会变的错误码
- 强调客户端判断 code，不判断 message
