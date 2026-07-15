# Multi-Language Document Intelligence Pipeline — Flows

## 1. Document Processing Pipeline (Upload → Validate → Output)

```mermaid
flowchart TB
    subgraph UPLOAD["Upload & Ingestion"]
        U1[Upload Endpoint] --> U2{Format Detection}
        U2 -->|PDF| U3[PyMuPDF Loader]
        U2 -->|DOC/DOCX| U4[python-docx Loader]
        U2 -->|PPT/PPTX| U5[python-pptx Loader]
        U2 -->|PNG/JPEG/TIFF| U6[Pillow Loader]
        U2 -->|HTML| U7[BeautifulSoup Loader]
        U3 & U4 & U5 & U6 & U7 --> META[Metadata Extraction]
        META --> U8{Is Scanned?}
        U8 -->|Digital PDF| U9[PDF Plumber Table Extraction]
        U8 -->|Scanned/Image| U10[Convert Page to Image]
        U9 & U10 --> U11[Page-wise Envelope Assembly]
    end

    subgraph PREPROC["Preprocessing Pipeline"]
        P1[Deskew: Hough Transform] --> P2{Skew > 2deg?}
        P2 -->|Yes| P3[Rotate Correction]
        P2 -->|No| P4[DPI Normalize -> 300 DPI]
        P3 --> P4
        P4 --> P5[Adaptive Denoise]
        P5 --> P6{Document Type?}
        P6 -->|Scanned| P7[Sauvola Binarization]
        P6 -->|Digital| P8[Skip Binarization]
        P7 & P8 --> P9[Page Segmentation via RT-DETR]
    end

    subgraph OCR_STAGE["Multi-Engine OCR Routing"]
        O1[Script Detection UniHD] --> O2{Language Profile}
        O2 -->|English only| O3[Mistral OCR 4]
        O2 -->|Indic Language| O4[PaddleOCR v3.5]
        O2 -->|Mixed English+Indic| O5[Hybrid Fusion Engine]
        O2 -->|Low-Resource Script| O6[EasyOCR]
        O3 & O4 & O5 & O6 --> O7[OCR Results: Text + BBoxes + Confidence]
        O7 --> O8{Per-Block Confidence Check}
        O8 -->|All > 0.9| O9[Pass to Layout Stage]
        O8 -->|Any < 0.7| O10[Fallback Engine Re-route]
        O10 --> O7
        O8 -->|0.7-0.9| O11[Flag for QA]
        O9 & O11 --> O12[Region Merger]
        O12 --> O13[OCR Block Assembly]
    end

    subgraph LAYOUT["Layout-Aware Parsing"]
        L1[RT-DETR Layout Detector] --> L2[Region Bounding Boxes]
        L2 --> L3[Block Classification: 17 Types]
        L3 --> L4[Geometric Containment Tree]
        L4 --> L5[Reading Order: Multi-Column Sort]
        L5 --> L6[Header/Footer Detection & Strip]
        L6 --> L7[Cross-Page Table Merging]
        L7 --> L8[Annotated Layout]
    end

    subgraph STRUCTURE["Structured Output Generation"]
        S1[Block Assembly] --> S2[Table Structure Parsing]
        S1 --> S3[Formula LaTeX Conversion]
        S1 --> S4[Figure Caption Extraction]
        S2 & S3 & S4 --> S5[Markdown Builder]
        S5 --> S6[Parallel Converters]
        S6 --> S7[JSON Export]
        S6 --> S8[Markdown Export]
        S6 --> S9[Parquet Export]
        S6 --> S10[hOCR Export]
    end

    subgraph VALIDATE["Quality Assurance Gates"]
        V1[Word Confidence Check] --> V2{Threshold}
        V2 -->|> 0.9| V3[Auto-Approve]
        V2 -->|0.7-0.9| V4[Human Review Queue]
        V2 -->|< 0.7| V5[Re-route to Fallback]
        V5 --> V6[PaddleOCR / EasyOCR / Tesseract]
        V6 --> V7{Conf Improved?}
        V7 -->|Yes| V8[Merge Result]
        V7 -->|No| V9[Mark Manual Needed]
        V4 --> V10[Human Review Interface]
        V10 --> V11{Decision}
        V11 -->|Approve| V8
        V11 -->|Edit| V12[Apply Correction]
        V11 -->|Reject| V5
        V12 --> V8
        V3 & V8 & V9 --> V13[Final Validated Output]
    end

    subgraph CHUNK["Semantic Chunking for RAG"]
        W1[Validated Document] --> W2[Header Extraction]
        W2 --> W3[DSHP-LLM Header Tree Builder]
        W3 --> W4[Document Hierarchical Tree]
        W4 --> W5[DFS-Based Grouping]
        W5 --> W6{Block Type}
        W6 -->|Text| W7[Merge till Section Boundary]
        W6 -->|Table < 5KB| W8[Atomic Table Chunk]
        W6 -->|Table > 5KB| W9[Row-Split with Header Repeat]
        W6 -->|Figure+Caption| W10[Paired Chunk]
        W7 & W8 & W9 & W10 --> W11[Context Enrichment]
        W11 --> W12[Ready for Embedding]
    end

    UPLOAD --> PREPROC --> OCR_STAGE --> LAYOUT --> STRUCTURE --> VALIDATE
    LAYOUT --> CHUNK
    VALIDATE --> CHUNK
```

## 2. Language-Aware Model Routing Flow

```mermaid
flowchart TD
    INPUT[Input Page Image] --> SCR[Script Detection Module UniHD]
    
    subgraph SCR["Script Detection"]
        S1[Global Page Analysis] --> S2[Script Family Classification]
        S2 --> S3[Per-Region Fine Classification]
        S3 --> S4[Confidence-Weighted Voting]
        S4 --> S5[Language Profile: Dominant + Secondary]
    end
    
    SCR --> DEC{Primary Language}
    
    DEC -->|Latin/English| R1{Layout Complexity}
    R1 -->|Simple| R1A[Mistral OCR 4]
    R1 -->|Complex Multi-Col| R1B[Mistral OCR 4 + Doc AI]
    
    DEC -->|Devanagari| R2{Sub-Language}
    R2 -->|Hindi/Marathi| R2A[PaddleOCR Hindi Model + PP-OCRv6]
    R2 -->|Sanskrit| R2B[PaddleOCR Devanagari -> EasyOCR]
    
    DEC -->|Dravidian| R3{Specific Script}
    R3 -->|Tamil| R3A[PaddleOCR Tamil Model]
    R3 -->|Telugu| R3B[PaddleOCR Telugu Model]
    R3 -->|Kannada| R3C[PaddleOCR Kannada Model]
    R3 -->|Malayalam| R3D[Mistral OCR 4 or PaddleOCR]
    
    DEC -->|Bengali Group| R4{Sub-Language}
    R4 -->|Bengali| R4A[PaddleOCR Bengali Model]
    R4 -->|Assamese| R4B[PaddleOCR Bengali Model shared script]
    
    DEC -->|Gujarati| R5A[Mistral OCR 4]
    DEC -->|Gurumukhi| R5B[EasyOCR -> Tesseract 5]
    DEC -->|Oriya| R5C[EasyOCR -> Tesseract 5]
    DEC -->|Urdu| R5D[Mistral OCR 4 Nastaliq]
    
    DEC -->|Mixed Script| HYB[Hybrid Fusion Engine]
    
    subgraph HYB["Hybrid Fusion"]
        H1[Mistral OCR 4 Run] --> H3[IoU Alignment of Regions]
        H2[PaddleOCR Run] --> H3
        H3 --> H4[Per-Region Confidence Compare]
        H4 --> H5{Both > 0.7?}
        H5 -->|No| H6[EasyOCR Tiebreak]
        H5 -->|Yes| H7[Select Higher Confidence]
        H6 --> H8[Best of 3 Merge]
        H7 --> H9[Region-Level Merge]
        H8 --> H9
    end
    
    R1A & R1B & R2A & R2B & R3A & R3B & R3C & R3D & R4A & R4B & R5A & R5B & R5C & R5D --> OUT[Output Blocks with Engine Attribution]
    HYB --> OUT
    
    OUT --> CONF{Per-Block Check}
    CONF -->|> 0.95| FINAL[Accept]
    CONF -->|0.8-0.95| ACCEPT[Accept with Flag]
    CONF -->|< 0.8| FALLBACK[Fallback Pipeline]
    FALLBACK --> F1[Try Next Available Engine]
    F1 --> F2[Re-OCR]
    F2 --> F3{Conf Improved?}
    F3 -->|Yes| MERGE[Merge Best Confidence]
    F3 -->|No after 3 engines| HITL[Human Review]
    MERGE & ACCEPT --> FINAL
```

### Coverage Heatmap

```mermaid
quadrantChart
    title OCR Engine Coverage by Language
    x-axis "Low Coverage" --> "High Coverage"
    y-axis "Low Accuracy" --> "High Accuracy"
    quadrant-1 "Primary Choice"
    quadrant-2 "Secondary Choice"
    quadrant-3 "Fallback"
    quadrant-4 "Avoid"
    "English-Mistral OCR 4": [0.92, 0.95]
    "Hindi-PaddleOCR": [0.85, 0.90]
    "Tamil-PaddleOCR": [0.80, 0.88]
    "Bengali-PaddleOCR": [0.78, 0.85]
    "Gujarati-Mistral OCR 4": [0.75, 0.87]
    "Malayalam-Mistral OCR 4": [0.70, 0.82]
    "Urdu-Mistral OCR 4": [0.72, 0.80]
    "Punjabi-EasyOCR": [0.55, 0.65]
    "Oriya-EasyOCR": [0.45, 0.55]
    "Mixed English+Hindi-Hybrid": [0.90, 0.88]
```

## 3. AgenticOCR Query-Driven Extraction Flow

```mermaid
sequenceDiagram
    participant User as User / App
    participant QR as Query Router
    participant Ret as Page Retriever
    participant AG as AgenticOCR Agent
    participant VLM as VLM Qwen3-VL
    participant Tool as Zoom & OCR Tool
    participant Gen as Generator LLM

    User->>QR: Query: What was Q3 revenue growth?
    QR->>QR: Classify Query Complexity
    
    alt Simple Factual
        QR->>Ret: Direct Retrieval from Index
        Ret-->>User: Chunk-Level Answer
    else Complex Multi-Step
        QR->>QR: Expand: Q3 revenue growth, quarterly financial results 2025, revenue breakdown
        QR->>Ret: Multi-Query Hybrid Retrieval
        Ret->>AG: Top-5 Retrieved Pages images + text
    end

    AG->>VLM: Analyze Page Layouts + Query Semantics
    VLM-->>AG: Regions of Interest Identified

    loop For Each Region
        AG->>Tool: image_zoom_and_ocr_tool image, bbox, tau
        
        alt tau = region complex
            Tool->>Tool: Full Layout Analysis on Zoomed Region
            Tool->>Tool: Fine-Grained OCR Sub-Elements
        else tau = element table/text/equation
            Tool->>Tool: Direct Element Parsing
        else tau = image visual only
            Tool->>Tool: Return Cropped Patch No OCR
        end
        
        Tool-->>AG: Structured Evidence bbox, type, text, conf
    end

    AG->>AG: Format Interleaved Text-Image Evidence
    AG->>Gen: Evidence + Reduced-Resolution Page Screenshots
    Gen->>Gen: Generate Answer with Source Citations
    Gen-->>User: Q3 revenue growth was 25% driven by...
    Gen-->>User: Evidence: Page 5 Table: Revenue Quarterly

    Note over AG,Gen: AgenticOCR parsed 37% of page area<br/>vs full-page baseline 14.5K -> 9.8K tokens
```

### AgenticOCR Integration Protocol

```python
class AgenticOCRMiddleware:
    def __init__(self, model_path: str):
        self.agent = AgenticOCRModel(model_path)
        self.tool = ImageZoomAndOCRTool()
    
    async def process(self, query: str, pages: list) -> AgenticOutput:
        evidence = []
        tokens_saved = 0
        
        for page in pages:
            decision = await self.agent.analyze(page, query)
            if not decision.needs_parsing:
                tokens_saved += page.estimated_tokens
                continue
            
            for region in decision.regions_of_interest:
                mode = self._select_mode(region.type)
                result = await self.tool.execute(
                    image=page.image, bbox=region.bbox, mode=mode
                )
                evidence.append(result)
                tokens_saved += region.area_ratio * page.estimated_tokens
        
        assembled = self._assemble(evidence, pages)
        return AgenticOutput(
            query=query, evidence=assembled,
            tokens_used=sum(e.tokens for e in assembled),
            tokens_saved=tokens_saved
        )
    
    def _select_mode(self, region_type: str) -> str:
        return {"table": "element", "equation": "element",
                "figure": "image", "text": "region",
                "complex_layout": "region"}.get(region_type, "region")
```

## 4. Quality Assurance and HITL Review Flow

```mermaid
flowchart TD
    INPUT[Pipeline Output Document] --> QA_ENGINE{QA Engine Parallel Gates}
    
    subgraph QA_ENGINE["7 Parallel Quality Gates"]
        G1[Word Confidence Check]
        G2[Page Confidence Check]
        G3[Structural Integrity Score]
        G4[Reading Order Coherence]
        G5[Table Structure Validity]
        G6[Language Consistency]
        G7[Entity Preservation Audit]
    end
    
    G1 -->|Pass all| AGG[Aggregate Quality Score]
    G2 -->|Pass all| AGG
    G3 -->|Pass all| AGG
    G4 -->|Pass all| AGG
    G5 -->|Pass all| AGG
    G6 -->|Pass all| AGG
    G7 -->|Pass all| AGG
    
    G1 -->|Fail| G1F{Conf < 0.7?}
    G3 -->|Fail| G3F{Score < 0.8?}
    G4 -->|Fail| G4F{Order < 0.8?}
    G5 -->|Fail| G5F{Table < 0.8?}
    
    G1F -->|Yes| RT[Re-route OCR Engine]
    G3F -->|Yes| RT
    G4F -->|No| HITL_FLAG[Flag for HITL]
    G5F -->|No| HITL_FLAG
    G1F -->|No| HITL_FLAG
    
    AGG --> SCORE{Overall Score}
    SCORE -->|>= 0.90| AUTO[Auto-Approve]
    SCORE -->|0.70 - 0.89| HITL_POOL[Human Review Pool]
    SCORE -->|< 0.70| REJECT[Reject & Reprocess]
    
    REJECT --> RT
    RT --> RT1{Retry Engine}
    RT1 -->|PaddleOCR| RT2[Re-OCR Paddle]
    RT1 -->|EasyOCR| RT3[Re-OCR EasyOCR]
    RT1 -->|Tesseract+LLM| RT4[Re-OCR + LM Post-Correction]
    RT2 & RT3 & RT4 --> RT5{Quality Improved?}
    RT5 -->|Yes after 3 retries| AGG
    RT5 -->|No| MANUAL[Manual Transcription Required]
    
    HITL_POOL --> DASH
    
    subgraph DASH["Human Review Dashboard"]
        H1[Side-by-Side: Original + OCR]
        H1 --> H2[Confidence Heatmap Overlay]
        H2 --> H3[Block-Level Annotations]
        H3 --> H4{Reviewer Action}
        H4 -->|Approve| H5[Approved]
        H4 -->|Edit Text| H6[Inline Correction]
        H6 --> H5
        H4 -->|Reject & Reroute| RT
        H4 -->|Bad Original| H7[Log Quality Issue]
    end
    
    AUTO --> FINAL[Final Document Output]
    H5 --> FINAL
    MANUAL --> FINAL
    
    FINAL --> FB[(Correction DB)] --> TRAIN[Fine-Tune Calibration] -.-> QA_ENGINE
```

### QA Scoring Formula

```
Overall = 0.30 * word_conf_avg + 0.20 * structural_integrity
        + 0.15 * reading_order + 0.10 * table_validity
        + 0.10 * language_consistency + 0.10 * entity_preservation
        + 0.05 * layout_coverage

Auto-Approve:  Overall >= 0.90 AND no metric < 0.7
Human Review:  Overall >= 0.70 AND < 0.90
Reject:        Overall < 0.70 OR any metric < 0.5
```

## 5. Batch Processing Job Lifecycle

```mermaid
stateDiagram-v2
    [*] --> QUEUED: POST /batch
    
    QUEUED --> VALIDATING: Worker Dequeues
    VALIDATING --> QUEUED: Validation Fail Retry
    VALIDATING --> PROCESSING: Documents Valid
    
    PROCESSING --> PROCESSING: Per-Doc Progress Update every 5 docs
    
    PROCESSING --> COMPLETED: All Succeeded
    PROCESSING --> PARTIAL: Some Failed
    PROCESSING --> FAILED: All Failed
    PROCESSING --> CANCELLED: User Cancels
    
    COMPLETED --> NOTIFY: Callback Fired
    PARTIAL --> NOTIFY
    FAILED --> NOTIFY
    
    NOTIFY --> COMPLETED: 200 OK
    NOTIFY --> PARTIAL: Timeout Retry 3x
    NOTIFY --> [*]: Logged
    
    COMPLETED --> ARCHIVED: 30 Days TTL
    PARTIAL --> ARCHIVED
    FAILED --> ARCHIVED
    
    note right of PROCESSING: GPU Workers: Complex Docs<br/>CPU Workers: Simple Docs<br/>Progress: Redis HMSET
```

### Error Retry Matrix

| Error Type | Strategy | Max Retries | Fallback |
|-----------|----------|-------------|----------|
| OCR Timeout | Exponential Backoff 1s 4s 16s | 3 | Next Engine |
| Document Corrupt | No Retry | 0 | Alert |
| Rate Limit 429 | Linear + Jitter | 5 | Rebalance Queue |
| GPU OOM | Reduce Batch Size | 2 | CPU Worker |
| Network Failure | Exponential Backoff | 3 | Dead Letter Queue |

## 6. RAG Ingestion Pipeline (Chunk -> Embed -> Index)

```mermaid
flowchart LR
    subgraph CHUNK["Phase 1: Semantic Chunking"]
        C1[Validated Document] --> C2[MultiDocFusion Pipeline]
        C2 --> C3[DSHP-LLM: Hierarchical Tree]
        C3 --> C4[DFS-Based Grouping]
        C4 --> C5[Hierarchical Chunks]
        C5 --> C6[Enrichment Layer]
        C6 --> C7[Markdown Headers]
        C6 --> C8[Section Path Metadata]
        C6 --> C9[Page Range Tag]
        C6 --> C10[Language Tag]
        C6 --> C11[Block Type Composition]
    end
    
    subgraph EMBED["Phase 2: Embedding"]
        E1[Chunks] --> E2{Chunk Type}
        E2 -->|Text/Markdown| E3[bge-m3 Multilingual Embedding]
        E2 -->|Table| E4[Table-Aware Embedding + Header Repetition]
        E2 -->|Figure+Caption| E5[VLM Description + Caption Embed]
        E3 & E4 & E5 --> E6[1024-dim Dense Vector]
        E6 --> E7[BM25 Sparse Encoding]
        E7 --> E8[Normalize + Pool]
    end
    
    subgraph INDEX["Phase 3: Indexing"]
        I1[Dense Vectors] --> I2[Vector DB: pgvector / Pinecone]
        I1 --> I3[Metadata Index: doc_id section_path language]
        I2 --> I4[HNSW Index]
        I4 --> I6[Hybrid Index Ready]
        I3 --> I5[BM25 Inverted Index]
        I5 --> I6
    end
    
    subgraph QUERY["Phase 4: Retrieval & Generation"]
        Q1[User Query] --> Q2[Query Expansion x3]
        Q2 --> Q3[Dense Search]
        Q2 --> Q4[BM25 Search]
        Q3 & Q4 --> Q5[RRF Fusion]
        Q5 --> Q6[Metadata Filtering]
        Q6 --> Q7[Cross-Encoder Reranker]
        Q7 --> Q8[Top-K Chunks]
        Q8 --> Q9[MM Context Assembly]
        Q9 --> Q10[LLM Generator]
        Q10 --> Q11[Answer + Citations]
    end
    
    CHUNK --> EMBED --> INDEX
    INDEX -.-> QUERY
```

### MultiDocFusion Hierarchical Chunking Detail

```mermaid
flowchart TD
    DOC[Long Industrial Document] --> DP[Vision-Based Region Detection]
    DP --> OCR_ENG[OCR Text Extraction]
    OCR_ENG --> AL[Annotated Layout: bbox type text]
    AL --> HEADER_LIST[Extract Header List H1 H1.1 H1.1.1 H2]
    HEADER_LIST --> DSHP
    
    subgraph DSHP["DSHP-LLM Document Hierarchical Tree"]
        H1[Header List] --> H2[Assign Parent-Child IDs]
        H2 --> H3[Build Header Tree]
        H3 --> H4[Link General Nodes tables figures text]
        H4 --> H5[Complete Hierarchical Tree]
    end
    
    H5 --> DFS_TRAV
    
    subgraph DFS_TRAV["DFS-Based Grouping"]
        D1[Root Node] --> D2[Depth-First Traversal]
        D2 --> D3{Token Count Exceeds Max?}
        D3 -->|No| D4[Continue Aggregating]
        D3 -->|Yes| D5[Create Chunk Boundary]
        D4 --> D2
        D5 --> D2
        D5 --> D6[Hierarchical Chunks with Markdown Headers]
    end
    
    D6 --> C1[Chunk 1: Introduction<br/># 1. Introduction<br/>Text...]
    D6 --> C2[Chunk 2: Background<br/>## 1.1 Background<br/>Text...]
    D6 --> C3[Chunk 3: Methodology<br/>## 1.2 Methodology<br/>Text...<br/>|Table|]
    D6 --> C4[Chunk N: Results<br/># 2. Results<br/>Text...]
    
    C1 & C2 & C3 & C4 --> EMBED[Multilingual Embedding bge-m3]
    EMBED --> VDB[(Vector Database)]
```

### Chunking Rules

| Priority | Rule | Detail |
|----------|------|--------|
| 1 | Section Boundary | Always break at section heading transitions |
| 2 | Table Atomicity | Tables under 5KB / 50 rows remain intact |
| 3 | Large Table Split | > 50 rows split by 30-row windows repeat header |
| 4 | Figure-Caption | Never split figure from its caption |
| 5 | List Integrity | Multi-item lists together unless > 50 items |
| 6 | Equation Blocks | Block equations intact inline stay with paragraph |
| 7 | Token Cap | Soft 1024 hard 2048 section continuations |
| 8 | Overlap | 64-token overlap at section boundaries |
| 9 | Noise Removal | Strip headers footers page numbers |
| 10 | Context Injection | [Doc: title, Section: path] per chunk |

### Addressing the InduOCRBench Gap

```mermaid
flowchart LR
    subgraph STANDARD["Standard OCR-First RAG <br/> InduOCRBench shows 30pt gap"]
        A1[PDF] --> A2[OCR Only]
        A2 --> A3[Flat Text]
        A3 --> A4[Token Chunking]
        A4 --> A5[Embed]
        A5 --> A6[Retrieve]
        A6 --> A7[VisualStyle: 82.9% OCR -> 52.8% RAG]
    end
    
    subgraph OURS["Our Architecture <br/> Mitigating the Gap"]
        B1[PDF] --> B2[Layout Analysis First]
        B2 --> B3[Style-Preserving OCR]
        B3 --> B4[Block Classification with Decorated Text]
        B4 --> B5[DSHP-LLM Hierarchical Tree]
        B5 --> B6[Structure-Aware Chunking]
        B6 --> B7[Style Metadata Preserved]
        B7 --> B8[Embed with Style Context]
        B8 --> B9[AgenticOCR for Complex Queries]
    end
    
    STANDARD -.->|30pt gap| OURS
    
    subgraph MITIGATIONS["Key Mitigations"]
        M1[VisualStyle: Preserve strikethrough color as block attributes]
        M2[CrossPageTable: Merge via header matching across pages]
        M3[MultiColumn: Whitespace histogram before text extraction]
        M4[UltraWide: Panorama stitching of page segments]
        M5[HistoryBooks: Non-standard reading order via LLM layout reasoning]
    end
    
    OURS --> MITIGATIONS
```

This architecture directly addresses the InduOCRBench finding: **high OCR accuracy does not guarantee strong RAG performance**. Every stage from layout analysis through chunking is explicitly structure-aware, ensuring semantic fidelity is preserved for downstream retrieval.
