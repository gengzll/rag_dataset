# rag_dataset

Small QA datasets for RAG evaluation, mirrored here for environments where
HuggingFace is unreachable (GitHub-only networks).

| Dir | File | Size | Contents | Source |
|---|---|---|---|---|
| `wikipedia-mini/` | `passages.parquet` | 780 KB | ~3,200 Wikipedia passages (retrieval corpus) | [rag-datasets/rag-mini-wikipedia](https://huggingface.co/datasets/rag-datasets/rag-mini-wikipedia) |
| `wikipedia-mini/` | `test.parquet` | 56 KB | ~918 question–answer pairs | same |
| `hotpot-qa/` | `validation-00000-of-00001.parquet` | 27 MB | HotpotQA distractor validation split (~7.4k multi-hop QA, each with 2 gold + 8 distractor paragraphs) | [hotpotqa/hotpot_qa](https://huggingface.co/datasets/hotpotqa/hotpot_qa) |

Licenses follow the upstream datasets (CC BY-SA 4.0 for HotpotQA; see the
source pages). Intended for evaluation/testing use with
[RAGdx](https://github.com/gengzll/RAGdx) — convert with
`scripts/prepare_dataset.py --from-file <parquet>`.
