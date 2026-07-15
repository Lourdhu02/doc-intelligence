# OCR & Document Parsing Tools

## Mistral OCR 4
- 170 languages, paragraph-level bboxes, typed-block classification, confidence scores
- Self-hosted Docker container; $2-5/1k pages; 93.07 OmniDocBench
- Best for: enterprise RAG, sensitive data, multilingual document workflows

## PaddleOCR (v3.5 / PP-OCRv6)
- 109 languages including Devanagari, Tamil, Telugu, Bengali; 2M param recognition model
- PaddleOCR-VL (0.9B VLM) for document parsing; SOTA on OmniDocBench
- Pipeline: text detection → angle classification → recognition; ONNX/CUDA 12 support
- Best for: cost-sensitive, self-hosted, Indian language OCR pipelines

## EasyOCR
- 80+ languages including Hindi, Tamil, Bengali, Telugu
- PyTorch-based; easy API but lower accuracy than PaddleOCR on Indic scripts
- Best for: quick prototyping, moderate accuracy needs

## Tesseract 5
- 100+ languages; LSTM-based; widely available (legacy tool)
- Poor accuracy on handwritten/mixed-script Indian documents (~60-70% on Devanagari)
- Best for: fallback, simple printed English/text-only documents

## Docling (IBM)
- PDF → structured Markdown/JSON with layout parsing, table extraction
- Deep learning-based document understanding; open-source
- Best for: document-level parsing pipeline integration

## Layout Parsers
- **LayoutLMv3**: Microsoft's layout-aware transformer for document understanding
- **PP-StructureV3**: PaddleOCR's layout + table + formula parser
- **Unstructured.io**: 30+ format support, open-source document parsing for RAG
