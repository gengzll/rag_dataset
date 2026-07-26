# OCRBench v2 采样子集（210 条 / 50 MB）

从 [OCRBench v2](https://arxiv.org/abs/2501.00321)（NeurIPS 2025）中流式抽取的 **210 条样本**，覆盖 18 种任务类型，每类 12 条。

> ⚠️ **用途提醒**：OCRBench v2 评测的是**通用多模态大模型（LMM）**的文字能力，不是文档解析管线。MinerU / PaddleOCR-VL / DeepSeek-OCR **均不在其榜单上**。本目录供 agent 选型时比较通用 VLM 使用，**不适用于 OCR 解析模块的选型**（那个请用 `omnidocbench_en/`）。详见 `esg_ocr_test/ocr_model_benchmark_summary.md` 第二节。

## 目录结构

```
ocrbench_v2_sample/
  annotations.json    # 210 条标注（问题、答案、任务类型、bbox 等）
  images/             # 210 张图像（.jpg / .png）
```

## 为什么是"采样"而非完整子集

OCRBench v2 在 HuggingFace 上以打包 parquet 形式发布，无法按文件挑选：

| 仓库 | 总大小 | 分片 |
|---|---|---|
| `ling99/OCRBench_v2`（本采样来源） | 943 MB | 3 片，最小 219 MB |
| `lmms-lab/OCRBench-v2` | 4.9 GB | 11 片，最小 143 MB |

因此采用流式读取（range 请求），按 `type` 分层采样、每类最多 12 条，累计约 50 MB 即停。实际网络传输量约等于采样体积，而非全量 943 MB。

注意：直接取前 N 行会全是 `rico` 手机截图（数据集按来源排序），所以必须分层采样才有代表性。

## 任务类型分布

| 类型 | 条数 | 类型 | 条数 |
|---|---|---|---|
| APP agent en | 12 | document classification en | 12 |
| ASCII art classification en | 12 | document parsing cn | 12 |
| key information extraction cn | 12 | document parsing en | 12 |
| key information extraction en | 12 | formula recognition cn | 12 |
| key information mapping en | 12 | formula recognition en | 12 |
| VQA with position en | 12 | handwritten answer extraction cn | 12 |
| chart parsing en | 12 | math QA en | 12 |
| cognition VQA cn | 12 | cognition VQA en | 12 |
| diagram QA en | 12 | full-page OCR cn | 6 |

其中与文档解析相关的是 `document parsing`、`chart parsing`、`key information extraction`、`full-page OCR`；其余为 App 界面、ASCII 艺术、数学问答等场景。

## 标注格式

`annotations.json` 为数组，每条含：

```json
{
  "id": 0,
  "image": "images/0_APP_agent_en.jpeg",
  "dataset_name": "rico",
  "type": "APP agent en",
  "question": "What is the wrong answer 2?",
  "answers": ["enabled", "on"],
  "eval": null,
  "bbox": null,
  "bbox_list": null,
  "content": null,
  "width": 1080,
  "height": 1920
}
```

- `question` / `answers` — 问答对形式的 GT（`answers` 是可接受答案列表）
- `eval` — 指定该条用哪种评分方式
- `bbox` / `bbox_list` — 定位类任务的坐标 GT（用 IoU 评分）
- `content` — 解析类任务的结构化 GT（用 TEDS 评分）

## 评分方式

按任务类型分指标：解析类（图→HTML/Markdown/JSON）用 TEDS，定位类用 IoU，信息抽取用 F1，识别/问答类用字符串匹配；各任务得分平均为总分（满分 100）。榜单现状（2026-03）：英文榜首约 68 分，中文榜首约 64 分，远未饱和。

## 来源

- 论文：https://arxiv.org/abs/2501.00321
- 榜单：https://99franklin.github.io/ocrbench_v2/
- 数据：https://huggingface.co/datasets/ling99/OCRBench_v2
- 采样脚本：本仓库外的 `download_ocrbench_v2_sample.py`（分层采样，可调 `TARGET_BYTES` / `PER_TYPE_CAP`）
