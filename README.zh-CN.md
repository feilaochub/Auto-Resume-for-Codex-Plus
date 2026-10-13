# Codex Plus 自动续跑技能

[English](README.md)

额度恢复后，让 Codex 自动继续未完成的工作。

## 为什么做这个技能

为 Codex Plus 用户减少等待额度恢复的麻烦：5 小时额度耗尽后，不用再记住重置时间，也不用守在电脑前等着点“继续”。提前启用自动续跑，让 Codex 在额度恢复后的定时唤醒中接着完成工作。

## 原理

利用 Codex 原生定时任务，提前为当前对话设置重复预约。定时唤醒后检查任务状态和可用额度，再从原对话、原项目目录继续工作；任务完成或用户取消后停用预约。

默认每 60 分钟重试；客户端支持时，优先在预计额度恢复的 5 分钟后首次唤醒。额度信息不可读时，按周期尝试续跑。

## 项目特色

- **只需一个文件**：全部续跑规则都在 `SKILL.md` 中，复制即可安装。
- **无需额外依赖**：利用 Codex 自身能力，无需额外安装 MCP、守护进程或其他工具。
- **回到原任务继续**：保留原对话上下文，在原项目目录接着工作。

## 使用方式

### 让 AI 安装

直接把下面这段话发给 Codex，让它读取仓库并安装：

```text
读取这个仓库：https://github.com/feilaochub/Auto-Resume-for-Codex-Plus
按照仓库说明，将 auto-resume-for-codex-plus 技能安装到我的个人技能目录。
已有同名技能时先保留原文件，安装后告诉我如何启用。
```

### 手动安装

将仓库中的 `auto-resume-for-codex-plus` 文件夹复制到以下任一位置：

| 安装范围 | 目标文件路径 |
| --- | --- |
| 当前项目 | `<项目目录>/.agents/skills/auto-resume-for-codex-plus/SKILL.md` |
| macOS 个人安装 | `~/.agents/skills/auto-resume-for-codex-plus/SKILL.md` |
| Windows 个人安装 | `%USERPROFILE%\.agents\skills\auto-resume-for-codex-plus\SKILL.md` |

### 启用与关闭

安装后，在 Codex 桌面版的项目对话中明确要完成的任务，再直接说：

```text
开启自动续跑
```

查看状态：

```text
$auto-resume-for-codex-plus 查看状态
```

关闭自动续跑：

```text
$auto-resume-for-codex-plus 关闭自动续跑
```

## 注意事项

- 本项目仅适用于 Codex 桌面版，主要面向 Plus 用户，不适用于其他 AI 助手。
- 保持电脑开机、Codex 应用运行，且项目目录可访问。
- 默认重试间隔为 60 分钟，额度恢复后不一定立即续跑。默认授权有效期为 14 天，可在启用时指定其他期限；无原生到期功能时，需等下一次成功唤醒才会检查到期并停用。
- 续跑会直接操作原项目目录。手动继续工作前，建议先关闭自动续跑，减少并发修改风险。
