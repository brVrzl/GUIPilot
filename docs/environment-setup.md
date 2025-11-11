# 环境配置指南

## 快速开始

使用统一脚本自动选择平台并创建环境：

```bash
python scripts/setup_env.py
```

> 提示：脚本会优先使用 `CONDA_EXE`（由 Conda/Miniforge 设置），否则自动在 `PATH` 中查找 `conda`，可直接在 Windows、macOS、Linux 上运行。

脚本会检测当前操作系统并对应选择：

- macOS → `envs/environment-macos.yml`
- Linux GPU → `envs/environment-linux-gpu.yml`
- Windows → `envs/environment-windows.yml`

环境创建/更新成功后，脚本会自动在对应 Conda 环境内运行 `pip install -r requirements-pip.txt`（Linux GPU 会额外安装 `requirements-pip-gpu.txt`）。

检测到同名环境已存在时，脚本会自动改用 `conda env update`。

若需要手动指定，可加入 `--update` 参数强制同步：

```bash
python scripts/setup_env.py --update
```

## 指定平台或环境名称

- 指定平台（跨系统或需要 GPU 版本）：

  ```bash
  python scripts/setup_env.py --platform linux-gpu
  ```

- 指定自定义环境名称：

  ```bash
  python scripts/setup_env.py --platform windows --name guipilot-win
  ```

## 文件说明

- `envs/environment-macos.yml`：macOS/Apple Silicon 适用，CPU 计算环境。
- `envs/environment-linux-gpu.yml`：Linux + NVIDIA GPU 环境，额外包含 CUDA 相关依赖。
- `envs/environment-windows.yml`：Windows x86_64 环境，默认 CPU 模式。
- `requirements-pip.txt`：跨平台通用的 pip 依赖。
- `requirements-pip-gpu.txt`：Linux GPU 专用的 CUDA 扩展依赖。
- `requirements-pip-windows.txt`：Windows 平台的精简 pip 依赖（针对 protobuf 与 paddlepaddle 的版本约束）。

如需添加新依赖，请同步更新相关环境文件并记录原因，以保持不同平台配置的可维护性。

> 环境创建后需手动执行 `conda activate <环境名>`（默认 macOS/Windows 为 `guipilot`，Linux GPU 为 `guipilot-gpu`）才能使用。

## CI 流程

`.github/workflows/env-matrix-check.yml` 会在 macOS、Linux（可选 Windows）三个 runner 上执行统一烟测：

1. `python scripts/setup_env.py --platform <target> --name guipilot-ci` 创建/更新 Conda 环境；
2. 在新环境中运行依赖探测（关键包导入）；
3. 依次执行四个实验脚本的精简版测试：
   - RQ1：`experiments/rq1_screen_inconsistency/main.py --limit 1 --skip-visualize`
   - RQ2：`experiments/rq2_flow_inconsistency/main.py --mode replay --skip-agent --limit 1`
   - RQ3：`experiments/rq3_component_wise_evaluation/main.py --pipelines guipilot_full --limit 1 --skip-visualize`
   - RQ4：`experiments/rq4_case_study/main.py --use-demo-data --skip-agent --allow-empty --skip-visualize`
4. 清理临时环境，避免污染 runner。

所有推送与 PR 默认触发该流程，确保跨平台依赖与脚本回归能被及早发现。若需恢复 Windows 节点，可解注工作流中的相关矩阵配置。*** End Patch

