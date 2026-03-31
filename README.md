# pluto-skills

Pluto 的个人 Claude Code 插件仓库，用于存放自定义 Skills、Commands 和 Agents。

## 仓库结构

```
pluto-skills/
├── .claude-plugin/
│   └── plugin.json          # 插件清单
├── skills/                  # Skills（推荐方式）
│   └── <skill-name>/
│       ├── SKILL.md         # 必需 - 主指令文件
│       ├── reference.md     # 可选 - 参考资料
│       └── scripts/         # 可选 - 辅助脚本
├── commands/                # Commands（简单指令）
│   └── <command-name>.md
├── agents/                  # Agents（子代理）
│   └── <agent-name>.md
└── README.md
```

## 如何添加 Skill

Skill 是推荐的扩展方式，支持子目录、辅助文件和更多高级功能。

### 1. 创建 Skill 目录

```bash
mkdir skills/my-skill
```

### 2. 创建 SKILL.md

在 `skills/my-skill/SKILL.md` 中编写：

```markdown
---
name: my-skill
description: 简要描述这个 skill 做什么、何时触发（描述要具体，Claude 会根据它判断是否调用）
# --- 可选字段 ---
# user-invocable: true          # 用户可通过 /my-skill 调用（默认 true）
# disable-model-invocation: true # 禁止 Claude 自动调用，仅用户可触发
# allowed-tools: Read, Grep      # Skill 激活时自动授权的工具
# model: opus                    # 覆盖当前会话模型
# effort: high                   # 推理深度：low, medium, high, max
# context: fork                  # 在隔离子代理中运行
# agent: Explore                 # 子代理类型
# argument-hint: "[filename]"    # 自动补全提示
# paths: "src/**/*.ts"           # 触发路径 glob
---

这里写 Skill 的详细指令，支持 Markdown。

可使用的变量：
- `$ARGUMENTS` — 用户传入的参数
- `$0`, `$1`, ... — 按位置的参数
- `${CLAUDE_PLUGIN_ROOT}` — 插件根目录路径

可用动态注入：
- `` !`shell-command` `` — 在 Skill 加载时执行 shell 命令并注入输出
```

### 3. 添加辅助文件（可选）

```
skills/my-skill/
├── SKILL.md
├── reference.md      # 额外参考文档，SKILL.md 中可引用
├── examples.md       # 使用示例
└── scripts/
    └── helper.sh     # 辅助脚本
```

### Skill 示例

```markdown
---
name: review-code
description: Multi-dimensional code review. Use when asked to "review code" or "代码审查".
allowed-tools: Read, Grep, Glob
---

请对指定代码进行多维度审查，包括：
1. 正确性
2. 可读性
3. 性能
4. 安全性

审查目标：$ARGUMENTS
```

## 如何添加 Command

Command 是更简单的方式，每个 Command 就是一个 `.md` 文件。

### 在 `commands/` 中创建文件

例如 `commands/deploy.md`：

```markdown
---
name: deploy
description: 部署项目到生产环境
allowed-tools: Bash
argument-hint: "[env]"
---

执行以下部署流程：
1. 运行测试
2. 构建项目
3. 部署到 $ARGUMENTS 环境
```

用户通过 `/deploy production` 调用。

### Skill vs Command 的区别

| | Skill | Command |
|---|---|---|
| 位置 | `skills/<name>/SKILL.md` | `commands/<name>.md` |
| 支持子目录 | 是 | 否 |
| 辅助文件 | 支持 reference.md 等 | 不支持 |
| 推荐场景 | 复杂指令、需要参考资料 | 简单的一次性指令 |

## 如何添加 Agent

Agent 是专门的子代理定义。

### 在 `agents/` 中创建文件

例如 `agents/security-reviewer.md`：

```markdown
---
name: security-reviewer
description: 安全审查专家，审查代码中的安全漏洞
model: sonnet
effort: medium
maxTurns: 20
---

你是一个安全审查专家。请检查代码中的：
- SQL 注入
- XSS
- CSRF
- 敏感信息泄露
- OWASP Top 10 漏洞
```

## 如何使用这个插件

### 方式一：通过 Marketplace 安装（推荐）

在 Claude Code 中执行：

```bash
# 1. 添加 marketplace
/plugin marketplace add sinpor/pluto-skills

# 2. 安装插件
/plugin install pluto-skills@pluto-skills
```

### 方式二：通过项目配置安装

在目标项目的 `.claude/settings.json` 中添加：

```json
{
  "extraKnownMarketplaces": {
    "pluto-skills": {
      "source": {
        "source": "github",
        "repo": "sinpor/pluto-skills"
      }
    }
  },
  "enabledPlugins": {
    "pluto-skills@pluto-skills": true
  }
}
```

### 方式三：本地开发调试

```bash
claude --plugin-dir /path/to/pluto-skills "your prompt"
```
