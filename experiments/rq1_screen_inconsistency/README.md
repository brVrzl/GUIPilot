# RQ1：界面不一致性检测实验

本实验复现论文中针对界面级不一致性的评估。脚本会对真实界面截图应用多种扰动（插入、删除、文本/颜色修改、组件交换等），并统计匹配与检测模块的表现。

## 数据准备

1. 数据集结构需包含 `*.jpg` 截图与同名 `*.json` 标注（四点坐标 + 组件类型）。示例可参考仓库内的 `datasets/new/`。
2. 若使用自定义数据集，请确保：
   - 目录下图片文件名为数字（如 `1.jpg`、`6.jpg`），脚本会按照数字升序遍历；
   - JSON 标注与图片同名位于同一目录。

可以通过两种方式指定数据路径：

- 设置环境变量 `DATASET_PATH` 指向数据根目录；
- 运行脚本时使用 `--dataset <path>` 覆盖该路径。

## 运行方式

```bash
conda activate guipilot  # 或 guipilot-gpu
python experiments/rq1_screen_inconsistency/main.py \
  --dataset datasets/new \
  --output-dir runs/rq1/full
```

脚本默认遍历数据集中所有符合规则的截图，并在输出目录下生成：

- `evaluation.csv`：记录每个扰动、匹配器与检测器组合的指标；
- `visualize/`：保存配对可视化结果与预测详情（可通过 `--skip-visualize` 禁用）。

## 快速验证（推荐用于 CI）

```bash
python experiments/rq1_screen_inconsistency/main.py \
  --dataset datasets/new \
  --limit 2 \
  --skip-visualize \
  --output-dir runs/rq1/smoke
```

`--limit` 参数会限制脚本处理的截图数量，便于在持续集成中快速确认依赖和管线是否正常。

