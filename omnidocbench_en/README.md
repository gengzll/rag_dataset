# OmniDocBench 英文子集（带 Ground Truth）

从 [OmniDocBench](https://github.com/opendatalab/OmniDocBench)（CVPR 2025，MinerU / PaddleOCR-VL / DeepSeek-OCR 等模型刷榜用的主基准）中筛出的 **219 页英文子集**，带完整 ground truth，可直接计算 Edit Distance / TEDS / CDM。

全量数据集为 1651 页、1.55 GB；本子集仅 219 页、约 90 MB，按 ESG 报告的实际难点筛选。

## 目录结构

```
omnidocbench_en/
  OmniDocBench_english_subset.json   # 219 页的全部 ground truth（2.4 MB）
  images/                            # 219 张页面图像（.jpg / .png，约 1653×2206）
```

## 筛选口径

只保留 `language` 为 `english`（198 页）或 `en_ch_mixed`（21 页）的页面，并从三个角度采样：

| 维度 | 说明 | 为什么选它 |
|---|---|---|
| 表格难点 | 无框线 / 少框线 / 跨行跨列 / 横向表 / 含公式表 | ESG 报告核心是 KPI 表；这也是各模型分差最大的子项 |
| 扫描退化 | `fuzzy_scan`(8) + `geometric_deformation`(2) | 对应扫描件场景的鲁棒性 |
| 复杂版式 | 杂志类页面（84 页） | 多栏 + 彩色背景 + 图文混排，视觉上最接近 ESG 报告设计 |

**注意**：OmniDocBench 的 `research_report`（研报）子类共 132 页，其中 126 页为简体中文、仅 1 页英文，因此不适用于英文 ESG 场景，未纳入本子集。

## 内容分布

- **语言**：english 198 / en_ch_mixed 21
- **文档类型**：magazine 84、academic_literature 60、exam_paper 24、colorful_textbook 21、book 12、PPT2PDF 10、newspaper 8
- **特殊属性**（一页可有多个）：table_horizontal 136、table_fewer_line 71、table_full_line 54、table_span 46、table_with_formula 38、colorful_backgroud 21、watermark 13、table_omission_line 11、fuzzy_scan 8、table_wireless_line 6、table_with_img 5、table_veritical 5、geometric_deformation 2

## 标注格式

JSON 为数组，每个元素代表一页，含两部分：

**`page_info`** — 页面级属性（筛选子集用）：

```json
{"data_source": "academic_literature", "language": "english",
 "layout": "double_column",
 "special_issue": ["table_fewer_line", "table_horizontal"],
 "subset": "v1.5"}
```

**`layout_dets`** — 版面块数组，每块含 `category_type`（text_block / title / table / figure / header / footer 等）、`poly`（四角点坐标，8 个数）、`order`（阅读顺序）。不同类型的 GT 字段不同：

- 文本块 → `text`（文本 GT）
- 表格块 → `html`（结构化 GT，算 TEDS 用；单元格内公式为 LaTeX）
- 公式块 → `latex`（算 CDM 用）
- 标记为 `ignore` / `abandon` 的块（页眉页脚等）评测时排除

## 使用方式

评测代码在 [OmniDocBench 官方仓库](https://github.com/opendatalab/OmniDocBench)，把模型输出的 Markdown 与本 JSON 对齐后计算：文本 Edit Distance、表格 TEDS、公式 CDM。

因为口径与官方一致，结果可直接对照公开榜单分数（见同仓库 `vidore_esg_reports_v2/ocr_model_benchmark_summary.md`）。

## 与 `vidore_esg_reports_v2/` 的分工

| 数据集 | Ground Truth | 用途 |
|---|---|---|
| `omnidocbench_en/`（本目录） | 有 | 定量评分，可与公开榜单对表 |
| `vidore_esg_reports_v2/` | 无 | 真实 ESG 领域回归，人工审查 |

## 来源

- 原数据集：https://huggingface.co/datasets/opendatalab/OmniDocBench
- 评测代码：https://github.com/opendatalab/OmniDocBench
- 原始许可与引用要求以上游仓库为准，本目录仅为评测用的子集副本。
