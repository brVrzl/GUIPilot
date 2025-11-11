# RQ2：流程不一致性实验

该实验复现论文中的流程不一致性评估。脚本会按照人工录制的交互序列回放界面，检测真实界面是否与原设计保持一致，并在发现异常时调用 VLM 助手尝试恢复流程。

## 数据准备

数据集目录需要以应用包名划分子目录，子目录下包含若干 `process_*` 录制文件夹。每个 `process_*` 目录应至少包含：

- `record.json`：由录制工具导出的流程描述；
- `*.jpg` / `*.xml`：每一步对应的截图与布局；
- （可选）`real/`：若要离线重放，还需提供真实屏幕截图 `real/<step>.jpg`。

示例结构（仓库内 `datasets/rq2/smoke_app`）：

```
datasets/rq2/smoke_app/
  process_1/
    1.jpg
    1.xml
    2.jpg
    2.xml
    record.json
    real/
      1.jpg
      2.jpg
```

`record.json` 中允许写入 `inconsistency_index` 字段（0-based），用于指定哪一步应该出现异常，便于在持续集成中保持确定性。

## 运行方式

### 1. 交互模式（需要真实设备）

```
conda activate guipilot
python experiments/rq2_flow_inconsistency/main.py \
  --dataset /path/to/rq2_dataset \
  --mode interactive \
  --output-dir runs/rq2/full
```

脚本会连接默认的 uiautomator2 设备，逐步执行交互，过程中需要人工确认：

- 对齐屏幕、在设定步骤故意执行错误操作；
- 审核 VLM 助手给出的恢复建议。

### 2. 离线重放模式（推荐用于 CI）

```
python experiments/rq2_flow_inconsistency/main.py \
  --dataset datasets/rq2/smoke_app \
  --mode replay \
  --use-layout \
  --skip-agent \
  --limit 1 \
  --output-dir runs/rq2/smoke
```

关键参数说明：

- `--mode replay`：不连接真机，从 `real/` 目录读取真实截图；
- `--use-layout`：直接使用录制时的 XML 构建控件信息，避免联网调用检测服务；
- `--skip-agent`：跳过 VLM 辅助阶段，加快烟测速度；
- `--limit`：只回放部分流程，控制执行时间。

脚本输出 `results.csv`（记录得分、耗时、恢复情况）与 `visualize/`（对比图像）。

## GitHub Actions 烟测

工作流文件 `/.github/workflows/rq2-smoke.yml` 会在 CI 中执行离线重放模式，验证依赖与脚本逻辑是否正确。可通过 `workflow_dispatch` 手动触发进行快速回归。