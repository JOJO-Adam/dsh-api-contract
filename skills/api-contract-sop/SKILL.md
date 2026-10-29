---
name: api-contract-sop
description: "API contract standard operating procedure: contract-first rationale, the seven decisions to lock early, drift causes, the compatible-evolution-first principle, and this studio's mechanically verified breaking-change detection."
version: "1.0.0"
display_name: "API 契约 SOP"
display_name_en: "API Contract SOP"
description_zh: "API 契约标准作业流程：契约先行的真实价值、必须早定的七件事、契约漂移的三种成因、兼容演进优先原则，以及本套技能机械化验证的破坏性变更检测。"
description_en: "API contract SOP: contract-first rationale, seven decisions to lock early, drift causes, compatibility-first, verified breaking-change detection."
whenToUse: "写 API 契约、定接口规范、契约先行、接口评审、纠结文档还是代码先写时"
---

# API 契约 SOP

本套技能的**唯一判断依据**。

## 1 · 契约先行的真实价值

不是"文档好看"，而是三点：
1. 前端可在后端写完前基于契约开发 / Mock
2. **歧义在设计阶段暴露**，而不是联调时吵架
3. 契约可被工具校验 → 能自动发现实现与契约不一致

## 2 · 必须早定的七件事（定错就是以后的破坏性变更）

1. 命名风格（snake_case / camelCase，全项目统一）
2. 资源用复数还是单数
3. 分页方案（offset / cursor，参数名统一）
4. 时间格式（统一 ISO 8601 带时区）
5. ID 类型（字符串还是数字）
6. 列表返回是裸数组还是 `{items, total}`
7. 版本策略（路径 / Header）

## 3 · 契约漂移的三种成因

| 成因 | 说明 |
|---|---|
| 代码先改、文档后补 | 补的时候漏了 |
| 多人手工维护同一份文档 | 并行改动合并冲突 |
| 运行时行为与文档本就不一致 | 文档写可选、实现是必填 |

**根治办法**：文档**由契约生成**，不手工维护；契约纳入 CI 做一致性检查。

## 4 · 兼容演进优先

**默认不升版本**，用兼容方式达成：
- 改字段名 → 新增字段 + 旧字段标 `deprecated` + 一个周期后删
- 加能力 → 新增可选字段 / 新端点
- 只有**确实无法兼容**时才升 `/v2`

> 频繁升版本会让维护成本指数上升。

## 5 · 破坏性变更可以机械判定（已实证）

用两版 OpenAPI spec 做结构对比，就能分类：

```
🔴 BREAKING  [参数类型] `id`: string → integer，客户端传值可能被拒
🔴 BREAKING  [字段类型] `id`: string → integer，客户端反序列化可能失败
🔴 BREAKING  [字段删除] 响应字段 `email` 消失，客户端读取会得到 undefined/None
🔴 BREAKING  [必填化] 响应 `nickname` 新增为 required，客户端可能没处理空值
🟢 ok        [新增字段] `nickname`
🟢 ok        [新增字段] `createdAt`
```

**6 项变更 → 4 破坏 / 2 兼容。** 说明这类判定不需要靠直觉，对比结构即可。

## 6 · 四块交付

1. 判定/诊断（基于用户真实 spec，不猜）
2. **依据**（机制层面：会让客户端坏在哪一步）
3. 具体改动（字段级 YAML）
4. 影响面与迁移建议

## 7 · 红线

- 不说「应该兼容」而不给机制解释
- 不把「新增响应字段」咬定 100% 安全（严格校验 schema 的客户端会失败）
- 不承诺「这样改一定不影响老客户端」
- 不让业务失败返回 HTTP 200
- 不让错误信息泄露 SQL / 堆栈 / 表名
- 示例不用 `"string"` 占位符
