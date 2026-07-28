# rag_dataset

Evaluation datasets mirrored here for environments where HuggingFace is
unreachable (GitHub-only networks).

Two families live in this repo:

- **RAG QA datasets** — retrieval/QA evaluation, used with [RAGdx](https://github.com/gengzll/RAGdx).
- **OCR / document-parsing datasets** — for evaluating OCR models (MinerU,
  PaddleOCR-VL, DeepSeek-OCR, …) on image/scanned documents. See
  [`ocr_eval_guide.md`](./ocr_eval_guide.md) for benchmark scores, the recommended
  evaluation flow, and how to identify which model version you are running.

## RAG QA datasets

| Dir | File | Size | Contents | Source |
|---|---|---|---|---|
| `wikipedia-mini/` | `passages.parquet` | 780 KB | ~3,200 Wikipedia passages (retrieval corpus) | [rag-datasets/rag-mini-wikipedia](https://huggingface.co/datasets/rag-datasets/rag-mini-wikipedia) |
| `wikipedia-mini/` | `test.parquet` | 56 KB | ~918 question–answer pairs | same |
| `hotpot-qa/` | `validation-00000-of-00001.parquet` | 27 MB | HotpotQA distractor validation split (~7.4k multi-hop QA, each with 2 gold + 8 distractor paragraphs) | [hotpotqa/hotpot_qa](https://huggingface.co/datasets/hotpotqa/hotpot_qa) |

Convert with `scripts/prepare_dataset.py --from-file <parquet>` in RAGdx.

## OCR / document-parsing datasets

Available on the `esg-ocr-test-data` branch.

| Dir | Size | Ground truth | Purpose | Source |
|---|---|---|---|---|
| [`vidore_esg_reports_v2/`](./vidore_esg_reports_v2) | 5 PDFs / 254 pages / 107 MB | No | Domain regression on **real ESG reports** — image-only PDFs with no text layer, reviewed manually | [vidore/esg_reports_v2](https://huggingface.co/datasets/vidore/esg_reports_v2) |
| [`omnidocbench_en/`](./omnidocbench_en) | 219 pages / 389 MB | **Yes** | Quantitative scoring (Edit Distance / TEDS / CDM), directly comparable to the public leaderboard | [opendatalab/OmniDocBench](https://huggingface.co/datasets/opendatalab/OmniDocBench) |
| [`ocrbench_v2_sample/`](./ocrbench_v2_sample) | 310 samples / 50 MB | **Yes** (QA-style) | Comparing general-purpose VLMs — **not** for document-parsing model selection | [ling99/OCRBench_v2](https://huggingface.co/datasets/ling99/OCRBench_v2) |

All three are English-only subsets, sized down from much larger upstream datasets
(OmniDocBench is 1.55 GB / 1651 pages upstream; OCRBench v2 is 943 MB of packed
parquet). Each directory has its own README describing the sampling criteria and
annotation format.

Suggested flow: score on `omnidocbench_en/` first (has ground truth, comparable to
published numbers), then validate on `vidore_esg_reports_v2/` for real ESG layouts.
Details in [`ocr_eval_guide.md`](./ocr_eval_guide.md).

## Licenses

Licenses follow the upstream datasets (CC BY-SA 4.0 for HotpotQA; see each source
page). The OCR directories are evaluation-purpose subsets — cite and comply with
the upstream benchmarks' terms when publishing results.
