## RQ4：案例研究脚本

`main.py` 用于复现实验四的案例研究：对比移动应用的 Mockup 与真实实现，以检查界面一致性并验证流程动作。

### 数据集结构

数据集根目录需包含若干流程目录，每个流程目录至少包括：

```
<dataset_root>/
  <app_name>/
    process_x/
      implementation/
        process.json
        1.jpg
        ...
      mockup/
        1.png
        ...
```

- `implementation/process.json`：按顺序记录真实流程的屏幕与动作：
  ```json
  [
    {
      "screen": "1.jpg",
      "mock_actions": ["点击登录按钮"],
      "actions": [
        {"action": "click", "bounds": [100, 200, 300, 260]}
      ]
    }
  ]
  ```
- `mockup/` 与 `implementation/` 中的截图文件名一一对应（Mockup 通常为 `*.png`，实现为 `*.jpg`）。

可通过命令行参数 `--dataset` 指定路径，或在 `.env` 中设置 `DATASET_PATH`。

### 脚本用法

```bash
python experiments/rq4_case_study/main.py \
  --dataset /path/to/case-study \
  --output-dir runs/rq4/report \
  --process-limit 2 \
  --screen-limit 3
```

常用参数：

- `--process-limit`：限制评估的流程数量。
- `--screen-limit`：限制每个流程评估的屏幕数量。
- `--skip-agent`：跳过 VLM agent 行为推理（CI 必备）。
- `--max-agent-trials`：启用 agent 时的最大重试次数。
- `--skip-visualize`：关闭可视化图片输出。
- `--include-branches`：默认会忽略文件名包含 `branch` 的屏幕，开启后强制评估。
- `--use-demo-data`：生成内置演示数据（无需外部下载），主要用于烟测或快速验证。
- `--allow-empty`：若筛选后无可评估屏幕，不抛出错误，直接退出。

### 输出内容

- `summary.csv`：逐屏幕记录匹配得分、检测到的不一致数量、动作验证结果等。
- `process_*/screen_*/report.json`：每个屏幕的完整检测/匹配结果及动作尝试日志。
- `process_*/screen_*/*.jpg`：若未关闭可视化，保存配对结果、标注等辅助图片。

### 烟测示例

CI 工作流使用的命令：

```bash
python experiments/rq4_case_study/main.py \
  --use-demo-data \
  --skip-agent \
  --allow-empty \
  --skip-visualize \
  --output-dir ci-artifacts/rq4
```

该命令在生成的演示数据上完成最小化流程，用于验证脚本依赖与基础逻辑。*** End Patch

