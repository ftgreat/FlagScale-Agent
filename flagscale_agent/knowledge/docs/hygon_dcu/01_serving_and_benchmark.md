# 海光 DCU 服务部署与 Benchmark 事实

来源: auto_e2e 框架 @ commit f36f6ff (2026-06-22, 海光 vllm-plugin-FL 吞吐优化)。

## 固定服务参数（scripts/plugin_serv.sh，不可改）

- 端口 8001；dtype=bfloat16；tp=2；gpu-memory-utilization=0.925；max-model-len=262144
- 模型: /baai/ldwang/models/Qwen/Qwen3.6-27B（只读）
- 启动方式: `bash scripts/plugin_serv.sh <实验目录>/`（读该目录 env.sh 的 export 变量）
- 就绪标志: 端口 8001 可连或日志出现 "Uvicorn running"
- 停止: `pkill -f "vllm serve"`，并需清理残留 VLLM::Worker 进程

## Benchmark 方法（scripts/benchmark_perf.py）

- 默认用例: (4096 输入, 1024 输出, 64 并发, 256 请求)；每用例 4 轮，丢第 1 轮 warmup，后 3 轮平均
- 请求失败的用例跳过 summary；输出 benchmark_perf_raw.json（各轮原始）+ benchmark_perf.json（汇总）
- 提取指标: **Total tok/s（主指标）**、Req/s、Output tok/s、TTFT (Mean/Median/P99)、TPOT、ITL
- 判读规则: 波动 >10% 视为未确认，须重跑

## 精度验证（scripts/plugin_cli.sh）

- 发送 chat 请求: `"Type \"I love Qwen3.5\" backwards"`，temperature=0.6
- 结果落盘 `<实验目录>/test_case_accuracy.json`，人工/Agent 判读输出正确性
- 仅作冒烟级门禁；更强精度对齐用技能 infer-precision-check

## 缓存清理路径（实验前必做）

`~/.triton/cache` 与 `~/.flaggems/` —— 算子/Triton 编译缓存，不清理会污染算子改动测量。
（注：auto_e2e 的 task_draft.md 中 "~/.trition" 为笔误，正确拼写为 ~/.triton）
