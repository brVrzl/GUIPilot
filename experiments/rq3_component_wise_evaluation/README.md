## RQ3：组件级评估脚本

`main.py` 用于复现实验三——对 GUIPilot 管线中关键组件（OCR、匹配器、后处理等）的消融评估。脚本复用 RQ1 的变异器，在统一数据集与指标上比较不同组合的表现，并输出 CSV 及可视化结果。

### 环境准备

与 RQ1、RQ2 相同：

1. 按仓库根目录说明创建 Conda 环境并安装依赖。
2. 准备包含 `*.jpg`（截图）与同名 `*.json`（LabelMe 标注）的数据集，或直接使用仓库自带的 `datasets/new`。

数据集路径可通过 `--dataset` 参数显式传入，或在 `.env` 中设置 `DATASET_PATH`。

- 默认不开启离线模型：如需本地检测器，设置 `ENABLE_LOCAL_DETECTOR=1`（可配合 `DETECTOR_WEIGHT_PATH`）；如需离线 OCR，设置 `ENABLE_PADDLEOCR=1`（支持 `PADDLEOCR_USE_GPU=1`）；也可通过 `DETECTOR_SERVICE_URL`、`OCR_SERVICE_URL` 切换到远程服务。

### 脚本用法

```bash
python experiments/rq3_component_wise_evaluation/main.py \
  --dataset datasets/new \
  --limit 2 \
  --pipelines guipilot_full,gvt_matcher \
  --output-dir runs/rq3/demo
```

常用参数：

- `--limit`：限制评估的屏幕数量（默认评估全部）。
- `--pipelines`：逗号分隔的预设组合，默认运行 `guipilot_full,guipilot_no_postprocess,gvt_matcher`。可选值：
  - `guipilot_full`：完整流程（OCR + GUIPilot matcher + 后处理过滤）。
  - `guipilot_no_postprocess`：关闭后处理，观察过滤策略的收益。
  - `guipilot_no_ocr`：跳过 OCR，评估文本识别组件的重要性。
  - `gvt_matcher`：改用 GVT matcher，对比匹配器差异。
- `--skip-visualize`：跳过可视化，加速烟测。
- `--output-dir`：输出目录，默认当前目录。
- `--seed`：设置随机种子，确保变异器重现性。

### 输出内容

- `summary.csv`：各组合的聚合指标（precision、recall、cls_precision、平均时延等）。
- `<pipeline>/evaluation.csv`：逐屏幕、逐变异器的详细记录。
- `<pipeline>/visualize/`：若未禁用可视化，包含配对/检测结果图。

### 烟测示例

CI 工作流参考命令：

```bash
python experiments/rq3_component_wise_evaluation/main.py \
  --dataset datasets/new \
  --limit 1 \
  --pipelines guipilot_full \
  --skip-visualize \
  --output-dir ci-artifacts/rq3
```

