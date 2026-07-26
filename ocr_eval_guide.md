# OCR / 文档解析模型评测指南

> 整理日期：2026-07-26
> 面向场景：ESG 数据 agent 的文档解析模块选型（图片/扫描件 PDF → 结构化 Markdown）

本分支包含三套 OCR 评测数据，分工明确，建议配合使用。

## 一、三套数据集的分工

| 目录 | 来源 | 规模 | Ground Truth | 定位 |
|---|---|---|---|---|
| [`vidore_esg_reports_v2/`](./vidore_esg_reports_v2) | HF `vidore/esg_reports_v2` | 5 份报告 / 254 页 / 107 MB | 无 | **领域回归**：真实 ESG 报告，人工审查 |
| [`omnidocbench_en/`](./omnidocbench_en) | HF `opendatalab/OmniDocBench` | 219 页 / 389 MB | **有** | **定量评分**：可与公开榜单对表 |
| [`ocrbench_v2_sample/`](./ocrbench_v2_sample) | HF `ling99/OCRBench_v2` | 310 条 / 50 MB | **有**（问答式） | **通用 VLM 比较**：不用于解析模块选型 |

### 建议的评测流程

1. **先跑 `omnidocbench_en/`** —— 有 GT，口径与官方一致，能算出 Edit Distance / TEDS / CDM 并直接对照下方榜单分数，确认你部署的版本表现是否符合预期。
2. **再跑 `vidore_esg_reports_v2/`** —— 无 GT，用于验证在真实 ESG 版式上的实际效果（这是榜单分数无法替代的）。人工审查时**重点看表格输出**。
3. **`ocrbench_v2_sample/` 单独用途** —— 仅在为 agent 挑选通用 VLM（图表问答、复杂版面理解等兜底能力）时使用，见第三节说明。

## 二、OmniDocBench v1.6_full 榜单成绩（选型主依据）

> 数据来源：https://github.com/opendatalab/OmniDocBench
> Overall = ((1 − 文本编辑距离)×100 + 表格TEDS + 公式CDM) / 3

| 模型版本 | 发布时间 | Overall ↑ | 文本编辑距离 ↓ | 表格 TEDS ↑ | 公式 CDM ↑ | 备注 |
|---|---|---|---|---|---|---|
| PaddleOCR-VL-1.6 | 2026-05 | **96.34** | 0.033 | 94.76 | 97.53 | Paddle 现旗舰 |
| MinerU2.5-Pro-2604-1.2B | 2026-04 | **95.75** | 0.036 | 93.42 | 97.45 | MinerU 现旗舰（mineru 3.1+ 默认） |
| PaddleOCR-VL-1.5 | 2026-01 | 94.93 | 0.038 | 91.67 | 96.89 | 与 1.6 架构兼容可直接替换 |
| PaddleOCR-VL（初版 0.9B） | 2025-10 | 94.18 | 0.040 | 90.65 | 95.91 | |
| MinerU2.5-2509-1.2B | 2025-09 | 93.04 | 0.045 | 87.88 | 95.77 | mineru 软件 2.x 默认 |
| （参考）dots.ocr | 2025 | 90.77 | 0.048 | 87.18 | 89.95 | 非本次对比对象 |
| DeepSeek-OCR-2 | 2026-01 | 90.25 | 0.050 | 83.89 | 91.84 | DeepSeek 现旗舰 |

### 三点结论

1. **代际差 > 家族差。** 两家旗舰（PaddleOCR-VL-1.6 与 MinerU2.5-Pro）只差 0.6 分，而同一家新旧版本能差 2~3 分以上。**先确认公司实际部署的版本**，比纠结选哪家更重要。
2. **表格是分差放大器。** TableTEDS 跨度 83.89 ~ 94.76，远大于文本编辑距离的差异。ESG 报告以 KPI 表格为主，这一项直接决定实际效果。
3. **DeepSeek-OCR 系不适合当扫描件主解析器。** 表格 TEDS 比第一名低约 11 分；在 olmOCR-Bench 上其 "Old scans"（老扫描件）子项仅 33.3 分。它的价值在"光学压缩、低 token 成本喂 LLM"的前置压缩层，而非高精度解析。

### 如何确认公司在用的版本

- **MinerU**：`pip show mineru`。软件 2.x → 模型 MinerU2.5；软件 3.1+ → 模型 2.5-Pro；若配置为 pipeline 后端 = 传统管线（非 VLM，分数明显更低）
- **PaddleOCR**：`pip show paddleocr`。先区分用的是 PaddleOCR-VL 产线还是 PP-StructureV3 产线（差别巨大），再看加载的模型名（PaddleOCR-VL / -1.5 / -1.6）
- **DeepSeek-OCR**：看模型 ID（`deepseek-ai/DeepSeek-OCR` vs `DeepSeek-OCR-2`）

## 三、OCRBench v2（不用于解析模块选型）

> 数据来源：https://99franklin.github.io/ocrbench_v2/ · https://arxiv.org/abs/2501.00321

### 它是什么、怎么打分

- **评测对象**：通用多模态大模型（LMM），如 Qwen-VL、Gemini、GPT-4V 等，考察其"看图识文字"的综合能力。
- **数据构成**：中英双语，31 种场景（街景、手写、图表、表格、公式、文档等），10,000 条人工校验的**问答对**；另有 1,500 张人工标注图片的私有测试集（23 个任务）防过拟合。
- **任务形式**：单张图片 + 一个问题 → 短答案（QA 式），而非整页转录。
- **计算方式**：按任务类型分指标——解析类（图→HTML/Markdown/JSON）用 TEDS，定位类用 IoU，信息抽取用 F1，识别/问答类用字符串匹配；各任务得分平均为总分（满分 100）。
- **榜单现状（2026-03）**：英文榜第一 ~68 分，中文榜第一（Qwen2.5-VL-72B）~64 分——远未饱和，说明对 LMM 而言仍然很难。

### 为什么不适用于本次选型

1. **被测对象不同**：MinerU / PaddleOCR-VL / DeepSeek-OCR 均**不在其榜单上**——它测的是通用大模型，不是文档解析管线，无法用它横比我们的三个候选。
2. **任务形式不匹配**：我们的场景是"整本扫描 PDF → 结构化 Markdown"（整页转录），它是"单图 + 问题 → 短答案"，QA 得分无法换算成整页转录准确率。
3. **文档形态不匹配**：无多页扫描报告，也无 ESG/财报版式（无框 KPI 表、多栏、脚注）；31 个场景里大量是街景/手写等与我们无关的任务。本仓库采样的 310 条里 `document parsing en` 只有 15 条，也印证了这一点。
4. **分数口径不通**：其总分与 OmniDocBench 的 Edit Distance/TEDS 体系不可互换，无法和模型论文、公开榜单对表。

**适用场景**：为 agent 挑选通用 VLM 做兜底（图表问答、复杂版面理解）时，用它比较候选大模型的文字能力。

## 四、其他可选公开基准

三家模型都有公开成绩、可作交叉验证的另一个基准是 **olmOCR-Bench**（AI2，1400 份 PDF、7000+ 条判定式测试）：

| 模型 | 总分 | 备注 |
|---|---|---|
| PaddleOCR-VL（初版） | 80.0 | 三家中最高 |
| DeepSeek-OCR（初代） | 75.7 | **Old scans 子项仅 33.3** |
| MinerU 2.5.4 | 75.2 | |

注意榜上为较旧版本，且以英文为主。其余可选：**Real5-OmniDocBench**（OmniDocBench v1.5 的五种真实退化物理重建，专测扫描鲁棒性）、**ParseBench**（LlamaIndex，约 2000 页保险/金融/政府企业文档，面向 agent 工作流打分，与 ESG 场景较贴近）。

## 数据来源

- OmniDocBench 官方榜单与评测代码：https://github.com/opendatalab/OmniDocBench
- olmOCR 榜单：https://github.com/allenai/olmocr
- MinerU2.5-Pro 论文：https://arxiv.org/pdf/2604.04771
- PaddleOCR-VL-1.6 论文：https://arxiv.org/pdf/2606.03264
- DeepSeek-OCR 2 论文：https://arxiv.org/abs/2601.20552
