# skills

个人编写和维护的 AI Agent Skills 集合，收录日常使用中的工作流程、领域规则和辅助资源。

每个 Skill 使用独立目录，以 `SKILL.md` 作为入口，可按需单独使用。

## 目录

- [Skill 列表](#skill-列表)
- [快速开始](#快速开始)
- [启动免费铸造监测](#启动免费铸造监测)
- [关闭与卸载](#关闭与卸载)
- [项目结构](#项目结构)
- [添加 Skill](#添加-skill)
- [许可证](#许可证)

## Skill 列表

| Skill | 说明 |
| --- | --- |
| [free-mint-watch](free-mint-watch/SKILL.md) | 发现并核实免费铸造项目，支持去重与定时通知；不自动铸造或交易。 |

## 快速开始

### 1. 获取仓库

```bash
git clone https://github.com/microai-lab/skills.git
cd skills
```

### 2. 安装所需 Skill

以下以 macOS / Linux 上的 Codex 和 `free-mint-watch` 为例。在本仓库根目录执行：

```bash
mkdir -p "$HOME/.agents/skills"
cp -R -i ./free-mint-watch "$HOME/.agents/skills/"
```

如果目标目录已存在，先核对已安装版本，再决定是否覆盖。若同名技能已安装在其他技能目录中，更新原有安装，避免重复加载。

Codex 支持用户级 `~/.agents/skills/` 和项目级 `.agents/skills/` 目录；上面的命令采用用户级安装。安装后检查技能是否可选；未出现时重启 Codex。目录和加载方式见 [Codex 官方技能文档](https://learn.chatgpt.com/docs/build-skills)。

其他 AI Agent 请按对应工具的技能加载方式安装完整目录。保留参考资料、脚本和资源文件，避免只复制 `SKILL.md` 导致引用失效。跨平台能力要求见 [接入说明](free-mint-watch/references/compatibility.md)。

### 3. 使用 Skill

阅读对应的 `SKILL.md`，确认所需工具、配置和权限，再在 AI Agent 中指定技能名称和任务。例如：

```text
使用 $free-mint-watch 只检查一次当前可参与的免费铸造项目，核实费用和截止时间，按新增项目和已有项目分组列出，不创建定时任务。
```

部分 Skill 依赖外部工具或服务。定时运行需要宿主提供调度能力，复制 Skill 文件本身不会启动后台任务。

## 启动免费铸造监测

### 运行条件

单次检查需要实时搜索、公开网页或只读链上查询能力。定时监测还需要宿主提供调度器、持久状态和通知通道；监测阶段无需连接钱包。只有用户选择项目并明确同意准备铸造后，才进入独立的钱包流程。

### 开启定时监测

在支持定时任务的宿主中发送以下指令：

```text
使用 $free-mint-watch 开启定时监测，每10分钟检查一次，时区 Asia/Shanghai，全天运行。
仅在推荐名单或关键信息变化时通知；每次推送完整列出新增项目和仍可参与的已有项目。
检查时只使用公开数据，不连接钱包，不自动铸造或交易。
将配置与去重状态保存到当前工作区的 .monitoring/ 目录，并在定时任务中记录技能、配置和状态文件的绝对路径。
如果已有对应任务，请更新原任务，不重复创建。
完成后告知任务名称或 ID、生效时间、状态文件位置及通知通道；缺少能力时说明原因，不声称已经启动。
```

收到宿主确认任务已创建或更新后，才表示监测已启动。安装技能、发送上述文字或关闭聊天窗口本身，都不能作为定时任务状态的依据。

### 修改时间或恢复监测

```text
将已有的 free-mint-watch 监测改为每15分钟检查，仅北京时间09:00–23:00运行，保留去重记录和其他配置。
```

```text
恢复之前暂停的 free-mint-watch 定时监测，沿用原配置和去重记录，不新建重复任务。
```

### 查看运行状态

```text
查看 free-mint-watch 对应定时任务的启停状态、检查频率、最近一次运行结果和配置、状态文件位置，不创建新任务。
```

无变化时保持静默，因此没有通知不一定代表任务停止。每次实际推送都会包含“新增项目”和“已有项目”；已失效或待复核的旧项目单独说明。

## 关闭与卸载

### 停止当前单次检查

使用宿主的停止按钮中断当前执行。单次检查完成后自然结束，无需关闭常驻进程。

### 暂停定时监测

在管理该任务的宿主中发送：

```text
暂停 free-mint-watch 对应的定时监测任务，保留配置和去重状态；如本轮仍在执行，也停止当前检查。确认任务已暂停后告知结果。
```

也可以在宿主的定时任务管理界面找到对应任务并暂停。关闭聊天窗口、停止一轮检查或删除 Skill 文件，不能代替暂停调度任务。

### 永久关闭定时监测

```text
删除 free-mint-watch 对应的定时监测任务，保留配置和去重状态文件，不影响其他监测任务。完成后确认该任务已删除。
```

### 禁用或卸载 Skill

先暂停或删除对应定时任务，再处理技能文件。仅希望禁用 Codex 技能、保留安装文件时，可按 [官方禁用说明](https://learn.chatgpt.com/docs/build-skills#enable-or-disable-local-codex-skills) 在 `~/.codex/config.toml` 中设置：

```toml
[[skills.config]]
path = "/absolute/path/to/free-mint-watch/SKILL.md"
enabled = false
```

将 `path` 替换为实际安装文件的绝对路径；已有对应配置时修改原条目。保存后重启 Codex。重新启用时将 `enabled` 改为 `true` 并重启；这不会自动恢复定时任务。

彻底卸载时，删除实际安装位置的 `free-mint-watch` 目录；按上方命令安装的目录为 `~/.agents/skills/free-mint-watch/`。仓库中的源文件可以保留。

配置与历史记录独立保存在监测工作区的 `.monitoring/` 中，默认保留以便恢复后继续去重。需要从零开始时，在任务停止后另行清理该任务使用的 `free-mint-config.json` 和 `free-mint-state.json`。

## 项目结构

```text
skills/
├── README.md
├── LICENSE
└── free-mint-watch/
    ├── SKILL.md
    ├── agents/
    │   └── openai.yaml
    └── references/
```

各 Skill 可按需包含以下内容：

| 路径 | 用途 |
| --- | --- |
| `SKILL.md` | 技能名称、描述和使用说明，必需。 |
| `agents/` | 面向特定 Agent 的元数据或配置。 |
| `references/` | 执行任务时按需读取的参考资料。 |
| `scripts/` | 辅助脚本。 |
| `assets/` | 模板、图片等资源。 |

## 添加 Skill

1. 在仓库根目录创建独立文件夹，使用小写字母、数字和连字符命名，例如 `my-skill`。
2. 添加 `SKILL.md`，在 YAML frontmatter 中填写 `name` 和 `description`，在正文中说明适用场景和执行流程。
3. 按需添加参考资料、脚本或资源，使用相对路径引用，避免依赖个人机器上的绝对路径。
4. 检查引用和脚本是否可用，并更新本 README 中的 Skill 列表。

最小 `SKILL.md` 示例：

```markdown
---
name: my-skill
description: 说明这个技能的用途，以及何时使用。
---

# My Skill

描述任务目标、执行步骤和必要约束。
```

## 许可证

本项目采用 [MIT License](LICENSE)。
