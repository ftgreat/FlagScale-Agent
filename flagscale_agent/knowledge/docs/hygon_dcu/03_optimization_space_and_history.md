# 优化空间与历史实验结论

来源: auto_e2e @ f36f6ff（CLAUDE.md 任务定义 + ARCHITECTURE 决策树）。

## 优化对象（任务约束）

- 唯一指定算子: `fused_recurrent_gated_delta_rule_packed_decode_kernel`
- 可修改: /workspace/vllm-plugin-FL（editable）、/workspace/FlagGems（editable）
- 若算子不在可修改仓库，先把算子实现适配到可修改仓库
- 修改算子前必须根据调用关系确认其在当前模型推理中生效

## 三个优化层面

1. **推理框架 vllm-plugin-FL**: 调度策略、内存管理、batch 处理逻辑、KV cache 管理、
   prefill/decode 分离
2. **算子库 FlagGems**: 算子实现优化、算子融合、Triton kernel 海光调优、新算子注册
3. **环境变量 env.sh**: `VLLM_FL_FLAGOS_BLACKLIST` 控制算子走 FlagGems 或原生实现

## 基线与历史

- Baseline Total tok/s: 2836.29（exp_13_numwarps2 分支 + FlagGems master）
- 已知基线分支状态: vllm-plugin-FL @ exp_30_no_validation
- 历史实验索引: <工作区>/experiments/summary.md（Exp|Date|Description|Throughput|Δ%|Notes）
- 决策树: Δ≥5% 保留同方向；0-5% 确认非噪声；≤0 回退；5+ 实验 <5% → 触发诊断

## 历史结论存放约定

- 平台/工具事实 → 本 knowledge 组
- 单次实验结论 → experiments/exp_N/result.md（任务资产，不入 knowledge）
- 跨实验持久教训 → FlagScale-Agent memory（pitfall/hygon/、insight/hygon/）
