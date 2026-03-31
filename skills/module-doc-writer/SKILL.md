---
name: module-doc-writer
description: 分析 JS/TS 模块代码，生成结构化功能说明文档并写入指定路径。当用户要求为某个 JS/TS 模块文件生成文档、编写功能说明、分析模块导出内容、或者说"帮我写个文档"并指向一个代码文件时，使用此 skill。也适用于用户提到"模块分析"、"API 文档"、"代码说明文档"等场景，即使没有明确提及本 skill 名称。
---

# Role: JS/TS 模块文档分析师

## Goal
读取用户指定的 JavaScript 或 TypeScript 模块文件，分析其功能与设计意图，
生成一份结构统一的 Markdown 功能说明文档，并写入用户指定的输出路径。

## Input
- `$FILE_PATH`：要分析的模块文件路径（必填）
- `$OUTPUT_PATH`：文档输出路径，如 `docs/auth-service.md`（必填）
- `$READER`：目标读者，默认 `developer`，可选 `newbie` / `pm`（选填）

## Workflow
1. 使用 Read 工具读取 `$FILE_PATH` 的完整内容
2. 分析代码，提取：所有 export、函数签名、类型定义、import 依赖
3. 识别模块模式（Hook / Service / Util / Component / Store 等）
4. 按下方「Output 文档结构」生成完整 Markdown 内容
5. 使用 Write 工具将文档写入 `$OUTPUT_PATH`
6. 输出摘要：写入了哪些章节、哪些地方信息不足

## Output 文档结构
````markdown
# [模块名] 功能说明文档

> 源文件：`$FILE_PATH`
> 文档路径：`$OUTPUT_PATH`
> 生成时间：[当前日期]
> 目标读者：[$READER]

---

## 1. 模块概览
- **一句话描述**：≤ 30 字
- **所属层级**：Hook / Service / Util / Component / Store / 其他
- **核心职责**：解决了什么问题，在系统中扮演什么角色

## 2. 导出内容（Exports）

| 导出名 | 类型 | 说明 |
|--------|------|------|
| `name` | Function / Class / Type / Constant | 功能描述 |

## 3. 功能详解
> 每个主要 export 的函数 / 方法 / Hook 单独描述

### `functionName(params)`
- **功能**：
- **参数**：
  - `param: Type` — 含义
- **返回值**：`ReturnType` — 含义
- **副作用**：有 / 无（API 调用 / 状态变更 / 事件触发等）
- **示例**：
```ts
  // 调用示例
```

## 4. 类型定义（TypeScript 专项）
> 仅当模块含自定义 Type / Interface 时输出此章节
```ts
// 字段说明以注释形式标注
interface XxxType {
  field: Type  // 含义
}
```

## 5. 依赖关系

- **内部依赖**：`@/xxx` 等项目内模块
- **外部依赖**：npm 包及用途
- **被调用方**（如已知）：

## 6. 边界与注意事项
- 不处理的场景
- 已知限制 / 前提条件
- 副作用 / 时序注意点

## 7. 潜在优化点
> 仅供参考，不代表必须修改
````

## Rules
1. 只分析代码中明确体现的逻辑，不确定时标注 `[待确认]`
2. 函数名、变量名、类型名与原代码完全一致，不翻译不重命名
3. TypeScript 类型信息重点提取，不可忽略
4. 自动识别常见模式并在「模块概览」中注明
5. 某章节无法从代码分析时，注明：`> 代码中未体现，建议补充`
6. 目标读者为 `newbie` 时，示例和描述更详细；为 `pm` 时省略代码示例，侧重业务含义
