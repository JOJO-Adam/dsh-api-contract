---
name: versioning-deprecation
description: "Versioning and deprecation playbook: compatible-evolution-first, when a new version is genuinely unavoidable, three version placement options, and the four-step deprecation process with real timeline input."
version: "1.0.0"
display_name: "版本与弃用"
display_name_en: "Versioning & Deprecation Playbook"
description_zh: "版本与弃用手册：兼容演进优先、什么时候才真需要新版本、三种版本位置方案，以及四步弃用流程与弃用窗口的确定依据。"
description_en: "Versioning and deprecation: compatibility-first, when a new version is unavoidable, version placement, four-step deprecation."
whenToUse: "要发新版 API、下线旧接口、给客户定弃用窗口时"
---

# 版本与弃用手册

## 一 · 兼容演进优先

**默认不升版本。** 绝大多数需求可以用兼容方式达成：

| 需求 | 兼容做法 |
|---|---|
| 改字段名 | 新增字段 + 旧字段标 deprecated + 一个周期后删 |
| 加能力 | 新增可选字段 / 新端点 |
| 换 ID 策略 | 新增字段并存，逐步迁移 |
| 调整枚举 | 新增值（不删旧值） |

**频繁升版本会让维护成本指数上升** —— 每个版本都要独立测试、部署、文档。

## 二 · 什么时候才真的需要新版本

必须同时满足三条：
1. 改动确实是破坏性的（见 `breaking-change-review`）
2. **无法通过兼容方式达成**
3. 有资源维护两套

**反例**：只是想改字段名 → 不需要 /v2。

## 三 · 版本放哪

| 方式 | 优点 | 缺点 |
|---|---|---|
| 路径 `/v1/users` | 直观、工具支持好、易缓存 | URL 带版本 |
| Header `Accept: ...vnd.x.v1+json` | URL 干净 | 调试麻烦、易被忽略、缓存复杂 |
| Query `?version=1` | 简单 | 不规范、易漏 |

**推荐路径版本**（最不容易出错），除非团队已有强约定。

## 四 · 四步弃用

**① 标注**
```yaml
/users/{id}:
  get:
    deprecated: true
    description: |
      已废弃，请迁移到 /v2/users/{id}。
      计划下线：YYYY-MM-DD。迁移指南见 xxx。
```

**② 告警响应头**
```http
Deprecation: true
Sunset: Wed, 31 Dec 2026 23:59:59 GMT
Link: </v2/users/{id}>; rel="successor-version"
```

**③ 公告** —— 正式通知使用方，不是只写在文档里

**④ 下线** —— 日期到了删，且**提前再提醒一次**

## 五 · 弃用窗口怎么定

**没有万能数字，先看两件事**：

1. **实际调用量** —— 必须统计。还有人大量在用就下线 = 事故
2. **使用方类型**
   - 内部服务：1-2 个迭代
   - 对外合作方：要确认对方已排期
   - 公开 API：通常数月

**关键**：不要假设"应该没人用了"。

## 六 · 交付格式

```
判断（是否需要升版本 + 为什么不能兼容）
策略（兼容演进 / 升版本 / 并行弃用）
具体做法（契约改动 + 响应头 + 时间表）
风险（谁受影响、怎么确认已迁移）
```

## 七 · 红线

- 不轻易建议升版本
- 弃用必须给替代方案 + 下线日期
- 不假设"没人用旧接口"，要用户先统计
- 不承诺"下线不影响任何人"
