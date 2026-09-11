# auto_e2e → FlagScale-Agent 迁移对照表

目的: 后续维护者理解两框架构件对应关系；auto_e2e 不再直接演进，以本仓库为准。

| auto_e2e 构件 | 去向 |
|---|---|
| CLAUDE.md（13 条宪法） | skill infer-hygon-optimize 的 Stage 0 + Critical Rules |
| .claude/commands/optimize.md（9 步循环） | skill infer-hygon-optimize 主体 |
| .claude/commands/benchmark.md | 并入主 skill Measure 阶段（命令见 knowledge 01） |
| .claude/commands/log-experiment.md | 主 skill §实验记录；LESSONS.md → memory |
| .claude/agents/profiler.md（148 行） | skill infer-hygon-profile |
| .claude/agents/research.md（病理清单） | 主 skill §Research 触发 + 病理清单；方法论复用技能 debug-strategy |
| scripts/*.sh + *.py | 任务资产，保持原样；参数/口径事实 → knowledge 01/02 |
| experiments/exp_N 产物纪律 | workspace-layout 技能的实验隔离规则 |
| summary.md 索引 | 任务资产 exp 体系内继续用 |
| LESSONS.md 跨实验教训 | FlagScale-Agent memory（pitfall/hygon/、insight/hygon/） |
| 精度验证方法 | 技能 infer-precision-check（冒烟级用 plugin_cli.sh） |
| 实验隔离/分支纪律 | train-reproduce 技能的 IMMUTABLE/ADAPTABLE 思想 |

## 迁移裁剪说明

- FlagScale-Agent 无子 agent（subagent）机制：profiler/research 从独立上下文 agent
  降级为 skill 内"阶段 + 触发条件 + 磁盘产物契约"。干净上下文诊断的意图由
  "诊断时先重新读磁盘产物、不依赖对话记忆"规则近似保留。
- run.sh/filter_progress.py 为 Claude Code CLI 专属启动器，不迁移。
- hipprof 为海光工具链；换芯片需替换 02 号文档与 infer-hygon-profile 中的工具命令。
