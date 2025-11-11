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

如需添加新依赖，请同步更新相关环境文件并记录原因，以保持不同平台配置的可维护性。

> 环境创建后需手动执行 `conda activate <环境名>`（默认 macOS/Windows 为 `guipilot`，Linux GPU 为 `guipilot-gpu`）才能使用。

## CI 依赖检测

`.github/workflows/env-matrix-check.yml` 使用 GitHub Actions 在 macOS、Linux、Windows 三个平台上执行如下验证：

- 运行 `python scripts/setup_env.py --platform <target> --name guipilot-ci`；
- 在新环境中完成 pip 依赖安装并执行简单导入测试；
- 流水线结束后删除临时环境。

每次推送或 PR 都会触发该流程，可以及早发现不同平台的依赖缺失或版本冲突。

## 实验烟测

为确保关键实验脚本在 CI 中可复现，仓库提供以下工作流：

- `rq1-smoke.yml`：运行 RQ1 屏幕不一致性离线烟测（限制样本数，跳过可视化）。
- `rq2-smoke.yml`：运行 RQ2 流程不一致性离线烟测（使用录制布局、跳过 VLM 阶段）。

可在 GitHub Actions 页面手动触发 `workflow_dispatch`，或通过推送与 PR 自动获知依赖或脚本回归。

