# MCP-Atlas 启动指南

## 1. 准备镜像

```bash
docker pull ghcr.io/scaleapi/mcp-atlas:1.2.5
docker tag ghcr.io/scaleapi/mcp-atlas:1.2.5 agent-environment:latest
```

## 2. 配置 `.env`

```bash
cp env.template .env
```

**必填**：

```bash
# 推理模型（被评测对象）
LLM_API_KEY=<your-key>
LLM_BASE_URL=<your-proxy>          # 例: https://your-proxy/v1

# 评分模型（Judge）
EVAL_LLM_API_KEY=<your-key>        # 可同 LLM_API_KEY
EVAL_LLM_BASE_URL=<your-proxy>     # 可同 LLM_BASE_URL
EVAL_LLM_MODEL=openai/GLM-5.0_H20_141_cp
```

**按任务配 MCP key**：根据任务用到的 server，在 `.env` 里填对应的 key（对照 `env.template` 注释即可）。改完需重启 Terminal A 的 docker。

## 3. 准备任务文件

把任务 CSV 放在 `services/mcp_eval/` 目录下（示例：`mcp_weather_tasks.csv`）。

## 4. 启动服务（两个终端）

> 以下命令均在项目根目录执行（即本文件所在目录）。

**Terminal A — MCP 工具沙箱**（端口 1984）

```bash
make run-docker
```

等待 `Uvicorn running on http://0.0.0.0:1984`。

**Terminal B — Agent 服务**（端口 3000）

```bash
make run-mcp-completion
```

等待 `Uvicorn running on http://0.0.0.0:3000`。

## 5. 跑任务 + 打分（Terminal C）

```bash
cd services/mcp_eval

# 跑任务
uv run python mcp_completion_script.py \
  --model "openai/GLM-5.0_H20_141_cp" \
  --input "mcp_weather_tasks.csv" \
  --output "weather_4_tasks_results.csv" \
  --concurrency 1

# 打分
uv run mcp_evals_scores.py \
  --input-file="completion_results/weather_4_tasks_results.csv" \
  --model-label="glm_weather_4_tasks" \
  --evaluator-model "openai/GLM-5.0_H20_141_cp"
```

结果文件：

- 任务输出：`completion_results/weather_4_tasks_results.csv`
- 评分结果：`evaluation_results/scored_glm_weather_4_tasks.csv` + 汇总 CSV + 直方图 PNG
