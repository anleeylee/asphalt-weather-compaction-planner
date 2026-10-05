# Asphalt Weather & Compaction Planner（沥青天气与压实规划工具）

**基于模型估算 HMA 冷却/压实窗口，用于沥青铺装规划** —— 文档化的集总热模型，不是拍脑袋。

[![Python 3.12](https://img.shields.io/badge/Python-3.12-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![CLI](https://img.shields.io/badge/CLI-command--line-blue)](#使用)
[![Platform: Windows](https://img.shields.io/badge/Platform-Windows-0078D6)](https://www.microsoft.com/windows)

**[English](./README.md)** | 简体中文

---

## 解决什么问题

热拌沥青（HMA）从出料到终压之间窗口很有限——气温、风速、基层温度和摊铺厚度都会缩短或延长它，判断失误就意味着压实失败和返工。本工具在气温、风速、面层/基层温度、摊铺厚度与出料温度上运行**文档化、带版本的集总热模型（ADI-THM-v1.0）**，估算**起压/止压窗口与冷却曲线**。

AI 只解释模型输出、绝不替代热模型；工具也绝不说"已批准施工"——供应商/机构的限制优先于规划默认值。

## 核心功能

- **HMA 冷却/压实窗口** ——基于集总热模型的起压/止压估算
- **模型版本文档化** ——ADI-THM-v1.0，每次运行可复现
- **输入参数** ——气温、风速、面层/基层温度、摊铺厚度、出料温度
- **冷却曲线输出** ——铺装规划所需的温度-时间曲线
- **诚实的边界** ——绝不说"已批准施工"；供应商/机构限制优先
- **AI 边界** ——AI 解释，热模型决策

## 安装

```bash
pip install -r requirements.txt
```

## 使用

```bash
python -m scripts.s08_weather_compaction_planner --project ./project --air-temp 60 --wind 5 --lift-in 3
```

加 `--help` 查看全部选项。本工具家族统一 CLI 约定：`--project <路径> --input <路径> --output <路径> --format json|csv|md|xlsx|pdf --config <路径> --verbose --dry-run`。

## 输出

- `thermal_run.json` ——输入快照、模型版本、窗口、假设、限制、风险因子
- `thermal_summary.md` ——规划摘要

## 工作原理

- **标准生命周期** ——`发现 → 摄取 → 提取 → 归一化 → 校验 → 计算 → 输出 → 审计`
- **确定性热模型** ——文档化集总模型，带版本、可复现
- **AI 解释而非替代** ——窗口由模型输出决定，不由 AI 决定

## 质量保证

- 每条记录做确定性 schema 校验；
- 黄金样例 fixtures 与方向检查（`tests/`）；
- 复核队列：低置信度、比例不明与冲突进入 `review_queue.json` / 摘要——绝不静默修正；
- 审计日志：每次运行将 `run_id`、耗时、输入哈希、引擎版本与输出写入 `<project>/audit/`。

## 测试

```bash
python -m pytest -q
```

## 安全与隐私

本地文件默认留在本地。API 密钥存放在环境变量（`ASPHALTCOSTS_API_KEY`、`ADI_AI_API_KEY`）——绝不写入源码。派生文件写入 `working/` 或 `output/`；来源文件永不修改。

## 许可证

MIT —— 见 [LICENSE](LICENSE)。属 [Asphalt Desktop Intelligence](https://github.com/anleeylee/asphalt-desktop-intelligence) 工具集的一部分。

## 计算引擎

[**AsphaltCosts.com**](https://asphaltcosts.com/) 是确定性计算层：面积 → 压实体积 → 净吨数 → 订购吨数（损耗只计一次）→ 车次 → 材料成本，内置有出处的规划默认值（FHWA 密度 145 lb/ft³，损耗率与车容量可编辑）。本工具把测量并校验后的输入喂给该引擎（或其带标签的本地镜像 `asphaltcosts-web-engine/1.0-mirror`），绝不重写公式。
