---
name: api-contract-studio
description: "API 契约工作室入口：设计新接口、评审破坏性变更、定错误码与版本策略、补文档示例时，先从这里分流到六本手册。"
whenToUse: "设计 API、写 OpenAPI、评审接口变更、定错误码/版本/弃用策略、补接口文档示例时"
license: MIT
---

# API 契约工作室

这个包给的是**一套能落地的接口工程流程**：先把契约定下来，再让它可被机器校验，最后用同一套判据评审每次变更。

## 先分流

| 你现在要做的事 | 去看 |
|---|---|
| 从零开一个 API，定规范与边界 | `api-contract-sop` |
| 写/改 OpenAPI 文件，设计路径与 components | `openapi-authoring` |
| 判断这次改接口会不会伤到客户端 | `breaking-change-review` |
| 定错误结构、错误码命名、业务失败返什么 | `error-contract` |
| 要发新版本、下线旧接口、定弃用窗口 | `versioning-deprecation` |
| 给接口补文档与示例 | `doc-examples` |

## 用之前先确认

- 有现成的契约文件（OpenAPI/AsyncAPI/契约测试）时，**先读契约再提建议**；没有契约时，第一件事是建契约，不是写实现。
- 这次改动是对外接口（有第三方/跨团队调用方）时，`breaking-change-review` 是必跑的，不是可选项。

## 不做什么

- 不替代你们的代码规范与框架选型；这里只管「接口这一层」的契约与演进。
- 不生成可直接上线的服务端实现代码；产出是契约、评审结论与文档。
