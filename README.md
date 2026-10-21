# cc-team

Claude Code 的多 Agent 团队流程：一个主控负责理解和拆分任务、按难度分派给不同模型的执行者、审查结果，再交给独立的对抗式审查员挑错，修改复审后交付。

## 角色

| 角色 | 文件 | 模型 / effort | 职责 |
|---|---|---|---|
| 主控 | `skills/cc-team/SKILL.md` | 主会话（建议 Opus / high） | 拆分、分派、审查、整合、交付 |
| 高难度执行者 | `agents/worker-opus.md` | Opus / medium | 方案设计、复杂代码、数值方法、严密写作 |
| 常规执行者 | `agents/worker-sonnet.md` | Sonnet / high | 按规格实现、测试、数据处理、资料整理 |
| 轻量执行者 | `agents/worker-haiku.md` | Haiku / low | 格式转换、按规则提取、简单检索 |
| 对抗式审查员 | `agents/critic.md` | Opus / high，只读 | 独立挑错，按阻断 / 重要 / 建议分级 |

## 流程

1. **理解与拆分**：复述目标、写出验收标准，计划写入 `PLAN.md`；大任务先让 critic 审查计划
2. **分派**：独立子任务并行，每份简报自成一体
3. **主控审查**：抽查产出文件，不合格的续接原执行者返工（最多 2 轮，之后升级）
4. **对抗审查**：critic 只拿到需求、验收标准和产出，看不到主控的推理
5. **处理意见**：逐条采纳或驳回；驳回阻断级意见须交给用户决定；对抗审查最多 2 轮
6. **交付**：结果、采纳 / 驳回的意见、剩余风险

## 安装

```bash
git clone https://github.com/GZTgituser/cc-team.git
mkdir -p ~/.claude/agents ~/.claude/skills
cp cc-team/agents/*.md ~/.claude/agents/
cp -r cc-team/skills/cc-team ~/.claude/skills/
```

Windows PowerShell：

```powershell
git clone https://github.com/GZTgituser/cc-team.git
New-Item -ItemType Directory -Force "$HOME\.claude\agents", "$HOME\.claude\skills" | Out-Null
Copy-Item cc-team\agents\*.md "$HOME\.claude\agents\"
Copy-Item -Recurse cc-team\skills\cc-team "$HOME\.claude\skills\"
```

如果 `~/.claude/agents/` 是第一次创建，需要重启 Claude Code。

## 建议的设置

在 `~/.claude/settings.json` 中禁止主控"分叉自己"，避免子任务绕开模型分级：

```json
{
  "permissions": {
    "deny": ["Agent(fork)"]
  }
}
```

不要开启 `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS`：开启后，被命名的子代理会以 Agent Teams 队员身份启动，而本流程依靠命名来续接执行者返工。

## 使用

```
/cc-team 把 data/ 下三组试验数据整理成对比图，并写一段分析
```

## 注意

- 一个任务的用量可能是单会话的数倍，简单任务直接在普通会话里做更划算。
- 主控和审查员同为 Opus，可能在同一处一起看漏；可把 `critic.md` 的 `model` 改成 `sonnet` 换个视角。
- 后台执行者需要权限时会在主会话里等待批准，无人值守时流程会停在那里。
