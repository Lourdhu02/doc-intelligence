# Key Papers

## AgenticOCR (arXiv 2602.24134)
- Query-driven on-demand OCR for RAG; decouples retrieval from page-level chunking
- Uses "thinking with images" to selectively parse regions of interest
- 4B/8B variants based on Qwen3-VL; SFT + RL (GRPO) training
- Serves as "third building block" of visual RAG stack (alongside Embedding + Reranking)
- Code: https://github.com/OpenDataLab/AgenticOCR

## InduOCRBench (ACL 2026 Industry Track, arXiv 2605.00911)
- 570 PDFs / 3,402 pages across 11 challenge types + 1 Normal category
- Dual-track eval: OCR fidelity + RAG impact (2,071 QA pairs)
- Key finding: high OCR accuracy ≠ strong RAG performance (30pt gap on VisualStyle)
- Dataset: https://huggingface.co/datasets/qihoo360/InduOCRBench

## MultiDocFusion (EMNLP 2025 / arXiv 2604.12352)
- Hierarchical + multimodal chunking pipeline for long industrial documents
- Pipeline: vision-based region detection → OCR → DSHP-LLM (hierarchical tree) → DFS-based grouping
- +8-15% retrieval precision, +2-3% ANLS QA over baselines

## Mistral OCR 4 (June 2026)
- Bounding boxes, typed-block classification, per-word + per-page confidence scores
- 170 languages, self-hosted single-container deployment
- OmniDocBench: 93.07, OlmOCRBench: 85.20, 72% human-preference win rate
- $2-5/1k pages; integrated with Mistral Search Toolkit
- https://mistral.ai/news/ocr-4/
