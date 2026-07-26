# vidore/esg_reports_v2 — ESG 报告图片 PDF 子集

源自 HuggingFace 数据集 [`vidore/esg_reports_v2`](https://huggingface.co/datasets/vidore/esg_reports_v2)（ViDoRe Benchmark v2 的 corpus 子集）。

用于验证 OCR / 文档解析模型（MinerU、PaddleOCR-VL、DeepSeek-OCR、Databricks ai_parse_document 等）在真实 ESG 文档上的解析准确率。

## 文件清单

| 文件 | 公司 | 页数 | 大小 |
|---|---|---|---|
| `restaurant_brands_international_2023.pdf` | Restaurant Brands International（汉堡王母公司） | 51 | 20.4 MB |
| `mcdonalds_2023_2024.pdf` | McDonald's | 78 | 40.0 MB |
| `loungers_2023.pdf` | Loungers（英国餐饮连锁） | 26 | 9.1 MB |
| `texas_restaurants_2023.pdf` | Texas Roadhouse | 40 | 14.8 MB |
| `wendys_2023.pdf` | Wendy's | 59 | 22.8 MB |

共 5 份报告、254 页，均为餐饮/快消行业公司的可持续发展（ESG/CSR）报告。

## 关键特性

- **纯图片 PDF，无文字层**：每一页都是位图图像，`Ctrl+F` 搜不到任何文字。喂给解析模型时强制走真正的 OCR 路径，不会被 PDF 内嵌文本"作弊"。
- **真实 ESG 版式**：包含 ESG 文档的典型难点——无框/彩色 KPI 表格、多栏排版、图表混排、脚注、带单位的指标数据。
- **干净渲染，非扫描退化**：页面由数字版 PDF 渲染而来，无倾斜/噪声/光照问题。适合第一轮"模型本身识别能力"评测；如需模拟扫描件，可对页面叠加旋转、噪声、JPEG 压缩后再测。

## 数据来源与生成方式

- 来源：HuggingFace 数据集 [`vidore/esg_reports_v2`](https://huggingface.co/datasets/vidore/esg_reports_v2)（ViDoRe Benchmark v2 的 corpus 子集，30 份报告、1538 页图像）。
- 生成：流式读取该数据集 `corpus/test` split 的前 5 份完整文档，按页序（corpus-id）排列，每份合并为一个纯图片 PDF。
- 原报告均为相关公司公开发布的 ESG/CSR 报告，本目录仅用于 OCR 模型评测研究。

## 使用建议

1. 无标注数据，评估方式为人工审查模型输出（Markdown/JSON）与原页面的一致性。
2. 重点检查**表格输出**（TEDS 意义上的结构还原）——这是各模型在 OmniDocBench 上拉开差距的主要子项，也是 ESG 文档的核心内容。
3. 定量分数请配合带 ground truth 的 [`omnidocbench_en/`](../omnidocbench_en)，本数据集负责领域真实性回归。各模型版本的公开基准成绩与评测流程建议见仓库根目录 [`ocr_eval_guide.md`](../ocr_eval_guide.md)。
