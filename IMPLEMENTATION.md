# Multi-Language Document Intelligence Pipeline — Implementation

## 1. Complete Project Structure

```
doc-intelligence/
├── api/                          # API endpoints
│   ├── __init__.py
│   ├── app.py                    # FastAPI application entrypoint
│   ├── routes/
│   │   ├── __init__.py
│   │   ├── parse.py              # POST /parse — single document
│   │   ├── extract.py            # POST /extract — structured extraction with schema
│   │   ├── batch.py              # POST /batch — batch processing
│   │   ├── agentic.py            # POST /agentic/parse — query-driven extraction
│   │   ├── health.py             # GET /health — system health
│   │   └── feedback.py           # POST /feedback — HITL corrections
│   ├── middleware/
│   │   ├── rate_limit.py
│   │   ├── auth.py
│   │   └── monitoring.py
│   └── schemas/
│       ├── request.py            # Pydantic request models
│       └── response.py           # Pydantic response models
│
├── pipeline/                     # Core pipeline modules
│   ├── __init__.py
│   ├── orchestrator.py           # Pipeline orchestrator (state machine)
│   ├── ingestion/
│   │   ├── __init__.py
│   │   ├── loader.py             # DocumentLoaderFactory
│   │   ├── formats/
│   │   │   ├── pdf_handler.py    # PyMuPDF + pdfplumber
│   │   │   ├── docx_handler.py   # python-docx
│   │   │   ├── pptx_handler.py   # python-pptx
│   │   │   ├── image_handler.py  # Pillow + OpenCV
│   │   │   └── html_handler.py   # BeautifulSoup
│   │   └── metadata.py           # Metadata extraction
│   ├── preprocessing/
│   │   ├── __init__.py
│   │   ├── deskew.py             # Deskew correction
│   │   ├── denoise.py            # Background noise removal
│   │   ├── binarize.py           # Adaptive binarization
│   │   ├── dpi_normalize.py      # DPI normalization
│   │   └── page_segment.py       # Page segmentation
│   ├── ocr/
│   │   ├── __init__.py
│   │   ├── base.py               # Abstract OCR engine
│   │   ├── router.py             # Language-aware model router
│   │   ├── script_detector.py    # Script detection (UniHD)
│   │   ├── engines/
│   │   │   ├── mistral_ocr4.py   # Mistral OCR 4 wrapper
│   │   │   ├── paddle_ocr.py     # PaddleOCR wrapper
│   │   │   ├── easy_ocr.py       # EasyOCR wrapper
│   │   │   └── tesseract.py      # Tesseract 5 wrapper
│   │   └── fusion.py             # Hybrid fusion engine
│   ├── layout/
│   │   ├── __init__.py
│   │   ├── detector.py           # Region detection (RT-DETR)
│   │   ├── classifier.py         # Block type classification
│   │   ├── reading_order.py      # Reading order reconstruction
│   │   ├── hierarchy.py          # Document hierarchical tree builder
│   │   └── hocr.py               # hOCR format converter
│   ├── agentic/
│   │   ├── __init__.py
│   │   ├── controller.py         # AgenticOCR controller
│   │   ├── agent.py              # VLM agent with tool-use
│   │   ├── tools.py              # image_zoom_and_ocr_tool implementation
│   │   ├── evidence.py           # Evidence extraction & assembly
│   │   └── integration.py        # RAG integration protocol
│   ├── structuring/
│   │   ├── __init__.py
│   │   ├── schema.py             # JSON schema builder
│   │   ├── markdown_builder.py   # Markdown output
│   │   ├── confidence.py         # Confidence scoring engine
│   │   ├── table_parser.py       # Table structure extraction
│   │   └── formula_parser.py     # LaTeX formula extraction
│   ├── chunking/
│   │   ├── __init__.py
│   │   ├── semantic.py           # Semantic chunking
│   │   ├── hierarchical.py       # MultiDocFusion hierarchical chunking
│   │   ├── dshp_llm.py           # DSHP-LLM header tree parser
│   │   ├── dfs_grouper.py        # DFS-based grouping
│   │   └── context.py            # Chunk contextualization
│   ├── validation/
│   │   ├── __init__.py
│   │   ├── quality_gate.py       # Confidence-gated validation
│   │   ├── structural.py         # Structural integrity metrics
│   │   ├── hitl_queue.py         # Human-in-the-loop review queue
│   │   └── feedback_loop.py      # Feedback-driven improvement
│   └── rag/
│       ├── __init__.py
│       ├── embedder.py           # Multi-lingual embedding
│       ├── indexer.py            # Vector DB indexer
│       ├── retriever.py          # Hybrid retriever (dense + sparse)
│       ├── reranker.py           # Cross-encoder reranker
│       └── generator.py          # LLM generator interface
│
├── utils/                        # Shared utilities
│   ├── __init__.py
│   ├── config.py                 # YAML/ENV configuration
│   ├── logging.py                # Structured logging
│   ├── metrics.py                # Prometheus metrics
│   ├── cache.py                  # LRU + disk cache
│   ├── retry.py                  # Exponential backoff
│   ├── bbox.py                   # Bounding box utilities
│   └── language.py               # Language/script constants
│
├── models/                       # ML model weights registry
│   └── registry.json             # Model version manifest
│
├── output/                       # Output converters
│   ├── __init__.py
│   ├── json_converter.py
│   ├── markdown_converter.py
│   ├── csv_converter.py
│   └── parquet_converter.py
│
├── tests/                        # Test suite
│   ├── unit/
│   │   ├── test_ocr_router.py
│   │   ├── test_script_detector.py
│   │   ├── test_layout.py
│   │   └── test_chunking.py
│   ├── integration/
│   │   ├── test_pipeline.py
│   │   ├── test_batch.py
│   │   └── test_rag.py
│   └── fixtures/
│       ├── sample_pdfs/
│       └── indic_scripts/
│
├── config/
│   ├── default.yaml              # Default configuration
│   ├── production.yaml           # Production overrides
│   └── engines.yaml              # OCR engine registry
│
├── docker/                       # Containerization
│   ├── Dockerfile.api
│   ├── Dockerfile.worker
│   ├── docker-compose.yml
│   └── .dockerignore
│
├── scripts/                      # Infrastructure scripts
│   ├── deploy.sh
│   ├── migrate_index.sh
│   └── seed_db.sh
│
├── docs/                         # Documentation
│   ├── ARCHITECTURE.md
│   ├── IMPLEMENTATION.md
│   ├── FLOW.md
│   ├── API.md
│   └── RAG_INTEGRATION.md
│
├── requirements.txt
├── pyproject.toml
├── Makefile
└── README.md
```

## 2. API Endpoint Design

### Endpoint Specification

| Method | Path | Description | Request | Response |
|--------|------|-------------|---------|----------|
| POST | `/parse` | Single document OCR + layout parsing | `DocumentInput` | `ParsedDocument` |
| POST | `/extract` | Structured extraction with JSON schema | `ExtractInput` | `ExtractedFields` |
| POST | `/batch` | Batch document processing (async) | `BatchInput` | `BatchJob` |
| GET | `/batch/{job_id}` | Batch job status & progress | Job ID | `BatchStatus` |
| POST | `/agentic/parse` | Query-driven AgenticOCR extraction | `AgenticInput` | `AgenticOutput` |
| GET | `/health` | System health, engine status | — | `HealthStatus` |
| POST | `/feedback` | Human correction for HITL loop | `FeedbackInput` | `FeedbackResult` |

### Request/Response Schemas

```python
# POST /parse
class DocumentInput(BaseModel):
    document: str | UploadFile  # URL, base64, or file upload
    options: ParseOptions = ParseOptions()
    
class ParseOptions(BaseModel):
    include_blocks: bool = True
    include_images: bool = False
    include_confidence: bool = True
    output_format: Literal["json", "markdown", "hocr"] = "json"
    page_range: str | None = None           # "0,2-4" or null for all
    language_hint: str | None = None        # Override auto-detection
    table_format: Literal["markdown", "html", "json"] = "markdown"
    detach_headers_footers: bool = True

# POST /extract
class ExtractInput(BaseModel):
    document: str | UploadFile
    schema: dict  # JSON Schema defining fields to extract
    prompt: str | None = None  # Optional annotation prompt
    include_blocks: bool = False

# POST /batch
class BatchInput(BaseModel):
    documents: list[str | UploadFile]
    options: ParseOptions
    callback_url: str | None = None
    priority: Literal["low", "normal", "high"] = "normal"

# POST /agentic/parse
class AgenticInput(BaseModel):
    query: str
    document: str | UploadFile
    page_range: str | None = None
    mode: Literal["auto", "region", "element", "image"] = "auto"

class AgenticOutput(BaseModel):
    query: str
    evidence: list[ExtractedEvidence]
    token_count: int
    total_tokens_saved: int | None  # vs full-page parse
```

### FastAPI Application Entrypoint

```python
# api/app.py
from fastapi import FastAPI
from api.routes import parse, extract, batch, agentic, health, feedback

app = FastAPI(title="DocIntelligence Pipeline", version="2.0.0")

app.include_router(parse.router, prefix="/v1")
app.include_router(extract.router, prefix="/v1")
app.include_router(batch.router, prefix="/v1")
app.include_router(agentic.router, prefix="/v1")
app.include_router(health.router, prefix="/v1")
app.include_router(feedback.router, prefix="/v1")
```

## 3. OCR Engine Abstraction Layer

### Abstract Base Engine

```python
# pipeline/ocr/base.py
from abc import ABC, abstractmethod
from dataclasses import dataclass, field
from typing import Optional

@dataclass
class OCRBlock:
    block_id: str
    block_type: str  # title, text, table, equation, figure, etc.
    bbox: dict       # {"x1": 0.1, "y1": 0.2, "x2": 0.9, "y2": 0.8}
    text: str
    confidence: float
    word_confidences: list[dict] = field(default_factory=list)
    engine: str = ""
    script: str = ""
    language: str = ""
    metadata: dict = field(default_factory=dict)

@dataclass
class OCRPage:
    page_num: int
    dimensions: dict  # {"width": 2480, "height": 3508, "dpi": 300}
    blocks: list[OCRBlock]
    markdown: str = ""
    images: list[dict] = field(default_factory=list)
    page_confidence: float = 0.0
    engine: str = ""
    language: str = ""
    metadata: dict = field(default_factory=dict)

class BaseOCREngine(ABC):
    @abstractmethod
    async def process_page(self, image: Image.Image, 
                           language: str | None = None,
                           options: dict | None = None) -> OCRPage:
        ...

    @abstractmethod
    async def process_document(self, document_path: str,
                                options: dict | None = None) -> list[OCRPage]:
        ...
    
    @property
    @abstractmethod
    def name(self) -> str: ...
    
    @property
    @abstractmethod
    def supported_languages(self) -> list[str]: ...
    
    @property
    @abstractmethod
    def supported_scripts(self) -> list[str]: ...
```

### Mistral OCR 4 Engine Wrapper

```python
# pipeline/ocr/engines/mistral_ocr4.py
import httpx
from pipeline.ocr.base import BaseOCREngine, OCRPage, OCRBlock

class MistralOCR4Engine(BaseOCREngine):
    def __init__(self, api_key: str, endpoint: str = "https://api.mistral.ai/v1/ocr"):
        self.client = httpx.AsyncClient(
            headers={"Authorization": f"Bearer {api_key}"},
            timeout=120.0
        )
        self.endpoint = endpoint
    
    @property
    def name(self) -> str: return "mistral_ocr_4"
    
    @property
    def supported_languages(self) -> list[str]:
        return ["en", "hi", "bn", "gu", "ta", "ml", "kn", "te", "ur",
                "fr", "de", "es", "zh", "ja", "ar", "ru", ...]
    
    async def process_page(self, image, language=None, options=None):
        # Convert PIL image to base64
        # Call /v1/ocr with include_blocks=True
        # Parse response into OCRPage
        # Map Mistral block types to canonical types
        response = await self.client.post(self.endpoint, json={
            "document": {"image": base64_image},
            "model": "mistral-ocr-4-0",
            "include_blocks": True,
            "confidence_level": "word"
        })
        return self._parse_response(response.json(), page_num)
```

### PaddleOCR Engine Wrapper

```python
# pipeline/ocr/engines/paddle_ocr.py
from paddleocr import PaddleOCR
from pipeline.ocr.base import BaseOCREngine

class PaddleOCREngine(BaseOCREngine):
    def __init__(self, lang_map: dict):
        self.instances = {
            script: PaddleOCR(use_angle_cls=True, lang=code)
            for script, code in lang_map.items()
        }
        # Indic-specific: 'en', 'hi', 'ta', 'te', 'bn', 'gu', 'ml', 'kn'
    
    async def process_page(self, image, language=None, options=None):
        # Use PP-OCRv6 pipeline:
        # 1. Text detection (DB++) → bboxes
        # 2. Angle classification → orientation
        # 3. Text recognition (SVTR) → text + confidence
        # If language is Indic, use PP-OCRv6 Indic model
        result = self.instances.get(language, self.instances['en']).ocr(image, cls=True)
        return self._paddle_to_ocrpage(result)
```

### Language-Aware Model Router

```python
# pipeline/ocr/router.py
class OCRRouter:
    def __init__(self, config: dict):
        self.engines: dict[str, BaseOCREngine] = {
            "mistral_ocr_4": MistralOCR4Engine(config["mistral"]),
            "paddle_ocr": PaddleOCREngine(config["paddle"]["lang_map"]),
            "easy_ocr": EasyOCREngine(config["easyocr"]),
            "tesseract": TesseractEngine(config["tesseract"]),
        }
        self.script_detector = ScriptDetector(config["script_detector"])
        self.fusion_engine = HybridFusionEngine()
    
    async def route_page(self, image: Image.Image, 
                          hint: str | None = None) -> OCRPage:
        # Step 1: Detect script(s) on page
        scripts = self.script_detector.detect(image)
        
        # Step 2: Select engine(s) based on dominant script
        if hint:
            dominant = hint
        else:
            dominant = scripts[0].script if scripts else "latin"
        
        routing_key = self._routing_key(dominant)
        
        if routing_key == "hybrid":
            return await self._hybrid_route(image, scripts)
        
        engine = self.engines[routing_key]
        page = await engine.process_page(image, dominant)
        page.engine = engine.name
        
        return page
    
    def _routing_key(self, script: str) -> str:
        # Determine routing strategy from script
        high_quality_scripts = {"latin", "devanagari", "bengali", 
                                "gujarati", "tamil", "malayalam"}
        if script in high_quality_scripts:
            if script == "latin":
                return "mistral_ocr_4"
            return "paddle_ocr"
        medium_quality = {"telugu", "kannada", "gurumukhi"}
        if script in medium_quality:
            return "paddle_ocr"  # PaddleOCR has better Indic coverage
        return "easy_ocr"  # Fallback for low-resource
    
    async def _hybrid_route(self, image, scripts):
        # Run Mistral OCR 4 + PaddleOCR in parallel
        mistral_task = self.engines["mistral_ocr_4"].process_page(image)
        paddle_task = self.engines["paddle_ocr"].process_page(image)
        mistral_page, paddle_page = await asyncio.gather(mistral_task, paddle_task)
        return self.fusion_engine.fuse(mistral_page, paddle_page)
```

## 4. Document Preprocessing Pipeline

### Preprocessing Steps

```python
# pipeline/preprocessing/processor.py
class DocumentPreprocessor:
    def __init__(self):
        self.deskewer = DeskewCorrector(max_angle=15.0)
        self.denoiser = BackgroundDenoiser(strength="adaptive")
        self.binarizer = AdaptiveBinarizer(method="sauvola")
        self.dpi_normalizer = DPINormalizer(target_dpi=300)
        self.segmenter = PageSegmenter()
    
    async def preprocess(self, page_image: Image.Image) -> Image.Image:
        # 1. Deskew: detect skew angle via Hough transform, rotate
        if self.deskewer.needs_correction(page_image):
            page_image = await self.deskewer.correct(page_image)
        
        # 2. DPI normalize to 300 DPI for consistent OCR quality
        page_image = self.dpi_normalizer.normalize(page_image)
        
        # 3. Denoise: remove background noise (especially scanned docs)
        page_image = await self.denoiser.denoise(page_image)
        
        # 4. Binarize if needed (scanned docs with poor contrast)
        if self.binarizer.should_binarize(page_image):
            page_image = await self.binarizer.binarize(page_image)
        
        return page_image
```

### Page Segmentation and Layout Analysis

```python
# pipeline/preprocessing/page_segment.py
class PageSegmenter:
    """
    Uses RT-DETR (heron-layout) or equivalent to detect:
    - Text regions, tables, figures, equations, headers, footers
    Returns list of bounding boxes with type labels
    """
    async def segment(self, page_image: Image.Image) -> list[LayoutRegion]:
        # Run object detection model
        detections = await self.model.predict(page_image)
        regions = [LayoutRegion(
            bbox=d.bbox,
            block_type=d.label,
            confidence=d.score
        ) for d in detections]
        return regions
```

## 5. Block Classification Schema

### Canonical Block Types (Engine-Agnostic)

```python
# pipeline/layout/classifier.py
class BlockType(str, Enum):
    TITLE = "title"               # Document/section titles
    TEXT = "text"                 # Body paragraphs
    TABLE = "table"               # Tabular data (grid/borderless)
    FIGURE = "figure"             # Images, diagrams, charts
    EQUATION = "equation"         # Mathematical formulas
    LIST = "list"                 # Bulleted/numbered lists
    CAPTION = "caption"           # Figure/table captions
    HEADER = "header"             # Running page headers
    FOOTER = "footer"             # Page footers
    SIGNATURE = "signature"       # Handwritten/digital signatures
    CODE = "code"                 # Code blocks
    REFERENCE = "reference"       # Citations, bibliography
    ASIDE = "aside_text"          # Sidebars, margin notes
    CHECKBOX = "checkbox"         # Form checkboxes
    FORM_FIELD = "form_field"     # Form label-value pairs
    STAMP = "stamp"               # Official stamps
    BARCODE = "barcode"           # QR codes, barcodes
    DECORATED_TEXT = "decorated_text"  # VisualStyle: strikethrough, colored, highlighted

class LayoutRegion:
    block_id: str
    block_type: BlockType
    bbox: BoundingBox
    confidence: float
    attributes: dict  # E.g., {"has_border": true, "num_rows": 5, "num_cols": 3}
    children: list[LayoutRegion]  # For hierarchical nesting
```

### Block Type Mapping Between Engines

```python
ENGINE_BLOCK_MAPPING = {
    "mistral_ocr_4": {
        "title": BlockType.TITLE, "text": BlockType.TEXT,
        "table": BlockType.TABLE, "figure": BlockType.FIGURE,
        "equation": BlockType.EQUATION, "caption": BlockType.CAPTION,
        "header": BlockType.HEADER, "footer": BlockType.FOOTER,
        "signature": BlockType.SIGNATURE, "code": BlockType.CODE,
        "references": BlockType.REFERENCE, "aside_text": BlockType.ASIDE,
        "list": BlockType.LIST,
    },
    "paddle_ocr": {
        "text": BlockType.TEXT, "title": BlockType.TITLE,
        "table": BlockType.TABLE, "figure": BlockType.FIGURE,
        "equation": BlockType.EQUATION, "footer": BlockType.FOOTER,
        "header": BlockType.HEADER, "reference": BlockType.REFERENCE,
    },
}
```

## 6. Structured Extraction Pipeline

### Pipeline Orchestrator (State Machine)

```python
# pipeline/orchestrator.py
from enum import Enum, auto

class PipelineState(Enum):
    INIT = auto()
    LOADING = auto()
    PREPROCESSING = auto()
    SCRIPT_DETECTION = auto()
    OCR = auto()
    LAYOUT_ANALYSIS = auto()
    BLOCK_CLASSIFICATION = auto()
    READING_ORDER = auto()
    VALIDATION = auto()
    STRUCTURING = auto()
    CHUNKING = auto()
    COMPLETE = auto()
    FAILED = auto()

class PipelineOrchestrator:
    def __init__(self, config):
        self.preprocessor = DocumentPreprocessor()
        self.ocr_router = OCRRouter(config)
        self.layout_analyzer = LayoutAnalyzer(config)
        self.structurer = StructureBuilder()
        self.validator = QualityGate(config)
        self.chunker = HierarchicalChunker(config)
        self.state = PipelineState.INIT
        self.metrics = PipelineMetrics()
    
    async def process_document(self, input: DocumentInput) -> ParsedDocument:
        # Load document → list of page images with metadata
        self.state = PipelineState.LOADING
        pages = await DocumentLoaderFactory.create(input.document).load()
        
        for i, page in enumerate(pages):
            # Preprocess
            self.state = PipelineState.PREPROCESSING
            processed = await self.preprocessor.preprocess(page.image)
            
            # Script detection
            self.state = PipelineState.SCRIPT_DETECTION
            scripts = await ScriptDetector().detect(processed)
            
            # OCR
            self.state = PipelineState.OCR
            ocr_page = await self.ocr_router.route_page(processed)
            
            # Layout analysis + block classification
            self.state = PipelineState.LAYOUT_ANALYSIS
            layout_regions = await self.layout_analyzer.analyze(processed)
            
            # Merge OCR blocks with layout regions
            annotated = self._merge(ocr_page.blocks, layout_regions)
            
            # Reading order
            self.state = PipelineState.READING_ORDER
            ordered_blocks = ReadingOrderReconstructor().sort(annotated)
            
            # Validation
            self.state = PipelineState.VALIDATION
            validated, review_needed = await self.validator.check(ordered_blocks)
            
            # Structure building
            self.state = PipelineState.STRUCTURING
            structured_page = self.structurer.build(ordered_blocks, ocr_page)
            
            pages[i] = structured_page
        
        # Chunking
        self.state = PipelineState.CHUNKING
        chunks = await self.chunker.chunk(pages)
        
        self.state = PipelineState.COMPLETE
        return ParsedDocument(pages=pages, chunks=chunks)
```

## 7. Batch Processing Architecture

### Queue-Based Worker Architecture

```mermaid
flowchart TB
    API[POST /batch] --> QUEUE[Job Queue: Redis/SQS]
    QUEUE --> DISPATCH[Job Dispatcher]
    DISPATCH -->|Job 1| W1[Worker 1: GPU]
    DISPATCH -->|Job 2| W2[Worker 2: GPU]
    DISPATCH -->|Job N| WN[Worker N: CPU]
    W1 --> PROGRESS[Progress Store: Redis]
    W2 --> PROGRESS
    WN --> PROGRESS
    W1 --> RESULT[(Result Store: S3)]
    W2 --> RESULT
    WN --> RESULT
    PROGRESS --> POLL[GET /batch/{id}]
    RESULT --> CALLBACK[Callback URL]
```

### Batch Job Lifecycle

```python
class BatchJob:
    job_id: str
    status: JobStatus  # QUEUED → PROCESSING → COMPLETED / FAILED / PARTIALLY_COMPLETED
    total_documents: int
    processed_count: int
    failed_count: int
    progress_pct: float
    estimated_remaining_sec: int
    results: list[JobResult]
    errors: list[JobError]
    created_at: datetime
    completed_at: datetime | None

class JobDispatcher:
    def __init__(self, queue, workers, storage):
        self.queue = queue
        self.workers = workers
        self.storage = storage
    
    async def create_job(self, batch_input: BatchInput) -> BatchJob:
        job = BatchJob(job_id=uuid4(), total_documents=len(batch_input.documents))
        for doc in batch_input.documents:
            task = ProcessingTask(job_id=job.job_id, document=doc, options=batch_input.options)
            await self.queue.enqueue(task, priority=batch_input.priority)
        return job
    
    async def process_task(self, task: ProcessingTask):
        try:
            result = await orchestrator.process_document(task.document, task.options)
            await self.storage.store(task.job_id, task.task_id, result)
            await self._update_progress(task.job_id, increment=1)
        except Exception as e:
            await self._record_error(task.job_id, task.task_id, str(e))
```

### Progress Tracking

```python
class ProgressTracker:
    def __init__(self, redis):
        self.redis = redis
    
    async def update(self, job_id: str, task_id: str, status: str):
        pipeline = self.redis.pipeline()
        pipeline.hincrby(f"job:{job_id}:stats", status, 1)
        pipeline.hset(f"job:{job_id}:tasks", task_id, status)
        await pipeline.execute()
    
    async def get_status(self, job_id: str) -> BatchJob:
        stats = await self.redis.hgetall(f"job:{job_id}:stats")
        total = int(stats.get("total", 0))
        processed = int(stats.get("completed", 0))
        failed = int(stats.get("failed", 0))
        return BatchJob(
            job_id=job_id,
            total_documents=total,
            processed_count=processed,
            failed_count=failed,
            progress_pct=((processed + failed) / total * 100) if total > 0 else 0
        )
```

### Error Handling Strategy

| Error Type | Retry Strategy | Max Retries | Escalation |
|-----------|---------------|-------------|------------|
| OCR engine timeout | Exponential backoff (1s, 4s, 16s) | 3 | Fallback to next engine |
| Document corruption | No retry, mark as failed | 0 | Log + alert |
| Rate limit (429) | Linear backoff + jitter | 5 | Queue rebalance |
| Network failure | Exponential backoff | 3 | Dead letter queue |
| GPU OOM | Auto-reduce batch size | 2 | Offload to CPU worker |
| Model loading failure | Immediate retry with reload | 2 | Health check trigger |

## 8. Output Format Converters

### Markdown Builder

```python
# pipeline/structuring/markdown_builder.py
class MarkdownBuilder:
    def build(self, page: ParsedPage) -> str:
        md_parts = []
        for block in page.ordered_blocks:
            if block.block_type == BlockType.TITLE:
                md_parts.append(self._render_title(block))
            elif block.block_type == BlockType.TABLE:
                md_parts.append(self._render_table(block))
            elif block.block_type == BlockType.EQUATION:
                md_parts.append(self._render_equation(block))
            elif block.block_type == BlockType.LIST:
                md_parts.append(self._render_list(block))
            elif block.block_type == BlockType.CODE:
                md_parts.append(self._render_code(block))
            elif block.block_type == BlockType.FIGURE:
                md_parts.append(self._render_figure(block))
            elif block.block_type in (BlockType.HEADER, BlockType.FOOTER):
                continue  # Strip headers/footers
            else:
                md_parts.append(block.text)
        return "\n\n".join(md_parts)
    
    def _render_table(self, block) -> str:
        # Preserve Markdown table with alignment
        # For large tables, add continuation markers
        return f"\n{block.text}\n"
    
    def _render_equation(self, block) -> str:
        # Wrap in $$ for block equations or $ for inline
        return f"$${block.text}$$\n"
```

### JSON Converter

```python
# output/json_converter.py
class JSONConverter:
    def convert(self, document: ParsedDocument, schema: dict | None = None) -> dict:
        if schema:
            return self._schema_conformant(document, schema)
        return self._full_export(document)
    
    def _full_export(self, doc) -> dict:
        return {
            "doc_id": doc.doc_id,
            "metadata": doc.metadata,
            "pages": [self._export_page(p) for p in doc.pages],
            "chunks": [self._export_chunk(c) for c in doc.chunks]
        }
```

### Parquet Converter (for large-scale analytics)

```python
# output/parquet_converter.py
import pyarrow as pa
import pyarrow.parquet as pq

class ParquetConverter:
    SCHEMA = pa.schema([
        ("doc_id", pa.string()),
        ("page_num", pa.int32()),
        ("block_id", pa.string()),
        ("block_type", pa.string()),
        ("text", pa.string()),
        ("confidence", pa.float32()),
        ("engine", pa.string()),
        ("script", pa.string()),
        ("bbox_x1", pa.float32()),
        ("bbox_y1", pa.float32()),
        ("bbox_x2", pa.float32()),
        ("bbox_y2", pa.float32()),
        ("section_path", pa.string()),
        ("chunk_id", pa.string()),
    ])
    
    def convert(self, document: ParsedDocument) -> bytes:
        table = pa.Table.from_pylist(self._flatten(document), schema=self.SCHEMA)
        buf = pa.BufferOutputStream()
        pq.write_table(table, buf, compression="snappy")
        return buf.getvalue().to_pybytes()
```

## 9. Deployment and Scaling Strategy

### Container Architecture

```yaml
# docker/docker-compose.yml
version: "3.9"
services:
  api:
    build:
      context: ..
      dockerfile: docker/Dockerfile.api
    ports: ["8000:8000"]
    environment:
      - OCR_ENGINE=mistral,paddle,easyocr
      - GPU_MODE=cuda
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: 1
              capabilities: [gpu]
  
  worker:
    build:
      context: ..
      dockerfile: docker/Dockerfile.worker
    deploy:
      replicas: 4
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: 1
              capabilities: [gpu]
  
  redis:
    image: redis:7-alpine
    ports: ["6379:6379"]
  
  vector-db:
    image: pgvector/pgvector:pg16
    environment:
      POSTGRES_DB: docintel
      POSTGRES_PASSWORD: ${DB_PASSWORD}
```

### GPU Resource Allocation

| Component | GPU Requirement | Memory | Notes |
|-----------|----------------|--------|-------|
| Mistral OCR 4 | 1× NVIDIA A10G | 12 GB | Self-hosted container |
| PaddleOCR | 1× NVIDIA T4 | 4 GB | ONNX runtime |
| Layout Analyzer (RT-DETR) | 1× NVIDIA T4 | 4 GB | Batch inference |
| AgenticOCR (4B) | 1× NVIDIA A10G | 12 GB | VLM inference |
| Embedding (bge-m3) | CPU (or 1× T4) | 8 GB | ONNX quantization |
| DSHP-LLM | 1× NVIDIA T4 | 8 GB | QLoRA 4-bit |

### Auto-Scaling Policy

```yaml
# Horizontal Pod Autoscaler spec (Kubernetes)
metrics:
  - type: Custom
    custom:
      metric: queue_depth
      target:
        type: AverageValue
        averageValue: 50  # Scale up when 50+ jobs queued
  - type: Resource
    resource:
      name: nvidia_com/gpu_memory_used
      target:
        type: AverageUtilization
        averageUtilization: 80
behavior:
  scaleDown:
    stabilizationWindowSeconds: 300
    policies:
    - type: Percent
      value: 20
      periodSeconds: 60
  scaleUp:
    stabilizationWindowSeconds: 0
    policies:
    - type: Percent
      value: 100
      periodSeconds: 30
```

### Mistral OCR 4 Self-Hosted Deployment

```bash
# Mistral OCR 4 enterprise container
docker run -d \
  --name mistral-ocr-4 \
  --gpus all \
  -p 8080:8080 \
  -e MISTRAL_OCR_LICENSE_KEY=${LICENSE_KEY} \
  -v /data/models:/models \
  mistral/ocr-4:latest

# API integration
MISTRAL_OCR_URL=http://mistral-ocr-4:8080/v1/ocr
```

### Observability Stack

| Component | Tool | Metrics |
|-----------|------|---------|
| Metrics collection | Prometheus | Per-engine latency, throughput, error rate, confidence distribution |
| Visualization | Grafana | Pipeline health dashboard, per-language accuracy trends |
| Log aggregation | Loki/ELK | Structured JSON logs per document, trace IDs |
| Tracing | OpenTelemetry | End-to-end trace: ingestion → OCR → chunking → vector DB |
| Alerting | AlertManager | Engine down, queue depth > threshold, accuracy drop > 5% |
