# Datasets

## InduOCRBench
- 570 PDFs, 3,402 pages, 11 challenge types + Normal
- 12 industries; 2,071 QA pairs for RAG eval; ACL 2026 Industry Track
- Hybrid Markdown annotations (MD + HTML tables + LaTeX); 98% annotation accuracy
- Key: measures OCR→RAG gap, not just character accuracy
- https://huggingface.co/datasets/qihoo360/InduOCRBench

## OmniDocBench
- End-to-end document parsing benchmark (text, tables, formulas, reading order)
- Used as primary public benchmark by Mistral OCR, PaddleOCR-VL, etc.
- SOTA scores: PaddleOCR-VL 92.9%, Hunyuan-OCR 94.1%, Mistral OCR 4 93.1%
- https://github.com/OpenDataLab/OmniDocBench

## Indian Language OCR Datasets
- **Sarvam Indic OCR Bench**: Benchmark for 22 Indian languages
- **BornoDrishti** (EACL 2026): Bangla OCR across diverse documents
- **IIT CFILT datasets**: Devanagari, Bengali, Tamil annotated corpora
- **TDIL (Technology Development for Indian Languages)**: Government OCR datasets
- **Bhashini project**: Crowdsourced Indian language NLP/OCR data
- **Marathi Legal Documents** (arXiv 2512.18004): 6 OCR-MT pipeline combinations evaluated

## Other Benchmarks
- **OlmOCRBench**: Allen AI benchmark; Mistral OCR 4 scores 85.20 (#3)
- **ParseBench**: General document parsing eval; Mistral OCR 4 scores 60.7 (#6)
- **FinRAGBench-V**: Financial document RAG benchmark
- **SlideVQA**: Slide-based visual QA dataset
