# 海光 HIP Profiling 工具链事实

来源: auto_e2e @ f36f6ff（profiler agent + task_draft.md）。

## 工具链角色

- `scripts/run_hipprof.sh` — profiling 全流程编排（启动带 trace 的服务 → warmup →
  开 trace → 负载 → 关 trace → 停服 → 等保存）
- `scripts/hipprof_wrapper.sh` — 包装 hipprof 启动 trace 会话
- `scripts/dryrun_perf.py` — 发送 benchmark 同款请求但不记录结果（warmup/profiling 负载），
  支持 `--profile`
- `scripts/plugin_serv_prof.sh` — torch profiler（--profiler-config）变体启动，PyTorch 层分析用
- trace 会话控制: `hipprof --session-client hipprof_vllm --start|--stop`

## Trace 产物（生成于实验目录）

| 文件 | 内容 | 列/格式 |
|---|---|---|
| trace.hipkernel.csv | GPU kernel 耗时 | Name, Calls, TotalDurationNs, AverageNs, Percentage |
| trace.hiptrace.csv | HIP API 调用统计 | 同上 |
| trace.db | 详细 trace db | sqlite |
| trace_*.pftrace | timeline | Perfetto |

## Kernel 名称模式 → 功能类别映射（海光/ROCm 栈）

- Attention: `kernel_unified_attention_*`
- GEMM: `Cijk_*`（rocBLAS 矩阵乘）
- 通信: `ncclDevKernel_*`
- Triton 融合: `triton_*`
- 自定义: `delta_rule`、`conv1d`、`chunk_fwd`、`recompute_w_u`

## Host-side API 判读阈值（经验值，来自 auto_e2e 实测）

- hipEventSynchronize >50% → GPU 利用率低 / pipeline bubble
- hipModuleLaunchKernel 高频低耗 → 融合或 Graph capture 机会
- hipMemcpyWithStream/Async 异常高 → 数据搬运瓶颈

## 分析判例（实测参考）

Attention 18.1% vs GEMM 47.8% → 优先 GEMM；hipEventSynchronize 63.4% → bubble 问题。
