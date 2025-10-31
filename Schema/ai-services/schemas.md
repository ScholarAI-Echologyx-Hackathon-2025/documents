# AI Services Schema Documentation

## Overview

This document contains schema definitions for the AI services in the ScholarAI platform, including the Extractor Service and Paper Search Service data models.

---

## Extractor Service Schemas

### ExtractionRequest

```python
from pydantic import BaseModel, HttpUrl
from typing import List, Optional, Dict, Any
from enum import Enum

class ExtractionMethod(str, Enum):
    GROBID = "grobid"
    PDFPLUMBER = "pdfplumber"  
    PDFFIGURES2 = "pdffigures2"
    NOUGAT = "nougat"
    TESSERACT = "tesseract"
    OCR = "ocr"

class ExtractionLevel(str, Enum):
    FAST = "fast"
    STANDARD = "standard"
    COMPREHENSIVE = "comprehensive"
    CUSTOM = "custom"

class ExtractionConfig(BaseModel):
    extraction_methods: List[ExtractionMethod] = ["grobid", "pdfplumber"]
    extraction_level: ExtractionLevel = ExtractionLevel.STANDARD
    include_figures: bool = True
    include_tables: bool = True
    include_equations: bool = True
    include_code_blocks: bool = True
    quality_threshold: float = 0.7
    enable_ocr: bool = False
    ocr_languages: List[str] = ["eng"]
    deduplication: bool = True
    fallback_methods: bool = True

class ExtractionRequest(BaseModel):
    url: HttpUrl
    config: ExtractionConfig = ExtractionConfig()
    paper_id: Optional[str] = None
    priority: str = "normal"  # "low", "normal", "high"
    callback_url: Optional[HttpUrl] = None
```

### ExtractionResult

```python
from datetime import datetime
from typing import List, Dict, Any, Optional

class BoundingBox(BaseModel):
    x1: float
    y1: float 
    x2: float
    y2: float
    page: int

class Author(BaseModel):
    name: str
    affiliation: Optional[str] = None
    email: Optional[str] = None
    orcid: Optional[str] = None

class PaperMetadata(BaseModel):
    title: str
    authors: List[Author]
    abstract: Optional[str] = None
    doi: Optional[str] = None
    arxiv_id: Optional[str] = None
    publication_date: Optional[datetime] = None
    venue: Optional[str] = None
    venue_type: Optional[str] = None  # "journal", "conference", "preprint"
    language: str = "en"
    page_count: int
    keywords: List[str] = []

class Section(BaseModel):
    id: str
    title: str
    content: str
    level: int  # 1 = main section, 2 = subsection, etc.
    page_start: int
    page_end: int
    subsections: List["Section"] = []
    bbox: Optional[BoundingBox] = None

class Figure(BaseModel):
    id: str
    caption: str
    url: HttpUrl  # B2 storage URL
    page: int
    bbox: BoundingBox
    file_type: str  # "png", "jpg", "svg"
    file_size: int  # bytes
    width: int
    height: int
    extraction_method: str
    confidence_score: float

class Table(BaseModel):
    id: str
    caption: str
    data: List[List[str]]  # 2D array of cell values
    headers: List[str]
    page: int
    bbox: BoundingBox
    row_count: int
    col_count: int
    has_header: bool
    extraction_method: str
    confidence_score: float
    csv_url: Optional[HttpUrl] = None  # B2 URL to CSV file

class Equation(BaseModel):
    id: str
    latex: str
    page: int
    bbox: BoundingBox
    equation_type: str  # "inline", "display", "numbered"
    confidence_score: float
    image_url: Optional[HttpUrl] = None  # Rendered equation image

class CodeBlock(BaseModel):
    id: str
    content: str
    language: Optional[str] = None  # "python", "java", "pseudocode"
    page: int
    bbox: BoundingBox
    line_count: int
    confidence_score: float

class Reference(BaseModel):
    id: str
    title: Optional[str] = None
    authors: List[str] = []
    venue: Optional[str] = None
    year: Optional[int] = None
    doi: Optional[str] = None
    url: Optional[HttpUrl] = None
    page: int
    raw_text: str

class NamedEntity(BaseModel):
    text: str
    label: str  # "PERSON", "ORG", "CHEMICAL", "DISEASE", etc.
    start_char: int
    end_char: int
    confidence_score: float
    page: int

class ExtractionQuality(BaseModel):
    overall_score: float
    text_coverage: float
    figure_detection: float
    table_extraction: float
    equation_recognition: float
    reference_parsing: float
    extraction_completeness: float
    confidence_distribution: Dict[str, float]

class ExtractedContent(BaseModel):
    sections: List[Section] = []
    figures: List[Figure] = []
    tables: List[Table] = []
    equations: List[Equation] = []
    code_blocks: List[CodeBlock] = []
    references: List[Reference] = []
    named_entities: List[NamedEntity] = []

class ExtractionResult(BaseModel):
    paper_id: str
    metadata: PaperMetadata
    content: ExtractedContent
    extraction_quality: ExtractionQuality
    processing_time: float  # seconds
    extraction_methods_used: List[str]
    b2_urls: Dict[str, HttpUrl]  # asset_type -> B2 URL mapping
    created_at: datetime
    version: str = "1.0"
```

---

## Paper Search Service Schemas

### SearchRequest

```python
from datetime import date
from typing import List, Optional, Dict, Any

class DateRange(BaseModel):
    start: Optional[date] = None
    end: Optional[date] = None

class SearchFilters(BaseModel):
    sources: List[str] = ["arxiv", "semantic_scholar", "pubmed"]
    date_range: Optional[DateRange] = None
    publication_types: List[str] = []  # "journal", "conference", "preprint"
    fields_of_study: List[str] = []
    venues: List[str] = []
    authors: List[str] = []
    min_citations: Optional[int] = None
    max_citations: Optional[int] = None
    languages: List[str] = ["en"]
    has_pdf: Optional[bool] = None
    max_results_per_source: int = 100

class AIRefinementConfig(BaseModel):
    enabled: bool = True
    expand_query: bool = True
    suggest_related_terms: bool = True
    semantic_expansion: bool = True
    field_specific_terms: bool = True

class PDFCollectionConfig(BaseModel):
    enabled: bool = True
    priority: str = "balanced"  # "speed", "coverage", "balanced"
    sources: List[str] = ["direct", "publisher_api", "preprint_servers"]
    max_file_size: int = 50  # MB
    timeout: int = 30  # seconds

class SearchRequest(BaseModel):
    query: str
    filters: SearchFilters = SearchFilters()
    ai_refinement: AIRefinementConfig = AIRefinementConfig()
    pdf_collection: PDFCollectionConfig = PDFCollectionConfig()
    include_citations: bool = False
    include_metrics: bool = True
    deduplicate: bool = True
```

### SearchResult

```python
class AuthorInfo(BaseModel):
    name: str
    affiliation: Optional[str] = None
    orcid: Optional[str] = None
    h_index: Optional[int] = None
    citation_count: Optional[int] = None
    url: Optional[HttpUrl] = None

class PaperMetrics(BaseModel):
    citation_count: int = 0
    citation_velocity: float = 0.0  # citations per month
    h_index: Optional[int] = None
    altmetric_score: Optional[float] = None
    influence_score: Optional[float] = None
    field_citation_ratio: Optional[float] = None
    relative_citation_ratio: Optional[float] = None
    downloads: Optional[int] = None
    views: Optional[int] = None

class PDFInfo(BaseModel):
    available: bool
    url: Optional[HttpUrl] = None
    b2_url: Optional[HttpUrl] = None
    file_size: Optional[float] = None  # MB
    pages: Optional[int] = None
    extraction_status: str = "pending"  # "pending", "processing", "completed", "failed"
    extraction_url: Optional[HttpUrl] = None

class Paper(BaseModel):
    id: str
    title: str
    authors: List[AuthorInfo]
    abstract: Optional[str] = None
    metadata: Dict[str, Any] = {}  # DOI, arXiv ID, PMID, etc.
    publication_date: Optional[date] = None
    venue: Optional[str] = None
    venue_type: Optional[str] = None
    fields_of_study: List[str] = []
    metrics: PaperMetrics = PaperMetrics()
    pdf_info: PDFInfo = PDFInfo()
    relevance_score: float
    quality_score: float
    source: str
    source_url: Optional[HttpUrl] = None
    discovered_at: datetime

class SearchAggregations(BaseModel):
    by_year: Dict[str, int] = {}
    by_venue: Dict[str, int] = {}
    by_field: Dict[str, int] = {}
    by_source: Dict[str, int] = {}
    by_author: Dict[str, int] = {}
    by_publication_type: Dict[str, int] = {}

class SearchSuggestions(BaseModel):
    related_queries: List[str] = []
    trending_topics: List[str] = []
    suggested_authors: List[str] = []
    suggested_venues: List[str] = []
    query_refinements: List[str] = []

class SearchResults(BaseModel):
    total_papers: int
    unique_papers: int
    sources_searched: List[str]
    search_time: float
    papers: List[Paper]

class SearchResponse(BaseModel):
    search_id: str
    original_query: str
    refined_queries: List[str]
    results: SearchResults
    aggregations: SearchAggregations
    suggestions: SearchSuggestions
    processing_status: str = "completed"
    created_at: datetime
```

### CitationNetwork

```python
class CitationNode(BaseModel):
    id: str
    title: str
    authors: List[str]
    year: int
    citation_count: int
    venue: Optional[str] = None
    centrality_score: float
    influence_score: float
    node_type: str  # "paper", "author", "venue"

class CitationEdge(BaseModel):
    source: str
    target: str
    relationship: str  # "cites", "cited_by", "co_cited", "co_authored"
    weight: float
    confidence: float
    created_at: Optional[datetime] = None

class ResearchCluster(BaseModel):
    id: str
    topic: str
    keywords: List[str]
    papers: List[str]  # paper IDs
    influence_score: float
    growth_rate: float
    key_authors: List[str]
    representative_venues: List[str]

class CitationPath(BaseModel):
    from_paper: str
    to_paper: str
    path: List[str]  # sequence of paper IDs
    path_length: int
    path_strength: float
    research_flow: str  # "forward", "backward", "bidirectional"

class CitationNetworkAnalysis(BaseModel):
    network: Dict[str, Any]  # nodes and edges
    insights: Dict[str, Any]
    clusters: List[ResearchCluster]
    citation_paths: List[CitationPath]
    temporal_evolution: Dict[str, Any]
    influence_propagation: Dict[str, float]
```

---

## Database Schemas (PostgreSQL)

### Extracted Papers Table

```sql
CREATE TABLE extracted_papers (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    paper_id VARCHAR(255) UNIQUE NOT NULL,
    original_url TEXT NOT NULL,
    b2_storage_url TEXT,
    
    -- Metadata
    title TEXT,
    authors JSONB,
    abstract TEXT,
    doi VARCHAR(255),
    arxiv_id VARCHAR(50),
    publication_date DATE,
    venue VARCHAR(500),
    page_count INTEGER,
    
    -- Extraction Info
    extraction_methods TEXT[],
    extraction_quality JSONB,
    processing_time FLOAT,
    extraction_status VARCHAR(50) DEFAULT 'pending',
    
    -- Content (stored as JSONB for flexibility)
    sections JSONB,
    figures JSONB,
    tables JSONB,
    equations JSONB,
    references JSONB,
    
    -- Timestamps
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Indexes
    UNIQUE(paper_id),
    INDEX(doi),
    INDEX(arxiv_id),
    INDEX(extraction_status),
    INDEX(created_at)
);
```

### Search Cache Table

```sql
CREATE TABLE search_cache (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    query_hash VARCHAR(64) UNIQUE NOT NULL,
    original_query TEXT NOT NULL,
    refined_queries JSONB,
    filters JSONB,
    
    -- Results
    results JSONB NOT NULL,
    aggregations JSONB,
    suggestions JSONB,
    
    -- Metadata
    sources_searched TEXT[],
    total_papers INTEGER,
    search_time FLOAT,
    
    -- Cache Management
    expires_at TIMESTAMP NOT NULL,
    hit_count INTEGER DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Indexes
    UNIQUE(query_hash),
    INDEX(expires_at),
    INDEX(created_at)
);
```

### Paper Index Table

```sql
CREATE TABLE paper_index (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    paper_id VARCHAR(255) UNIQUE NOT NULL,
    
    -- Basic Info
    title TEXT NOT NULL,
    authors JSONB,
    abstract TEXT,
    
    -- Identifiers
    doi VARCHAR(255),
    arxiv_id VARCHAR(50),
    pmid VARCHAR(20),
    
    -- Publication Info
    publication_date DATE,
    venue VARCHAR(500),
    venue_type VARCHAR(50),
    fields_of_study TEXT[],
    
    -- Metrics
    citation_count INTEGER DEFAULT 0,
    citation_velocity FLOAT DEFAULT 0,
    quality_score FLOAT,
    influence_score FLOAT,
    
    -- PDF Info
    pdf_available BOOLEAN DEFAULT FALSE,
    pdf_url TEXT,
    b2_url TEXT,
    extraction_completed BOOLEAN DEFAULT FALSE,
    
    -- Search Optimization
    search_vector TSVECTOR,
    
    -- Source Info
    source VARCHAR(100),
    source_url TEXT,
    last_updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Timestamps
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Indexes
    UNIQUE(paper_id),
    INDEX(doi),
    INDEX(arxiv_id),
    INDEX(pmid),
    INDEX(venue),
    INDEX(publication_date),
    INDEX(citation_count),
    INDEX(quality_score),
    INDEX(fields_of_study) USING GIN,
    INDEX(search_vector) USING GIN
);
```

---

## Message Queue Schemas

### RabbitMQ Message Format

```python
class MessageHeader(BaseModel):
    message_id: str
    correlation_id: Optional[str] = None
    source_service: str
    destination_service: str
    task_type: str
    priority: int = 5  # 1=lowest, 10=highest
    retry_count: int = 0
    max_retries: int = 3
    timestamp: datetime
    expires_at: Optional[datetime] = None

class CallbackInfo(BaseModel):
    service: str
    endpoint: str
    method: str = "POST"
    headers: Dict[str, str] = {}

class QueueMessage(BaseModel):
    header: MessageHeader
    payload: Dict[str, Any]
    callback: Optional[CallbackInfo] = None
    
    class Config:
        json_encoders = {
            datetime: lambda v: v.isoformat()
        }
```

### Extraction Request Message

```python
class ExtractionRequestMessage(BaseModel):
    header: MessageHeader
    payload: Dict[str, Any] = {
        "paper_id": "str",
        "pdf_url": "str", 
        "extraction_config": "dict",
        "user_id": "optional[str]",
        "project_id": "optional[str]"
    }
    callback: Optional[CallbackInfo] = None
```

### Search Request Message

```python
class SearchRequestMessage(BaseModel):
    header: MessageHeader
    payload: Dict[str, Any] = {
        "user_id": "str",
        "query": "str",
        "filters": "dict",
        "ai_refinement": "dict",
        "pdf_collection": "dict"
    }
    callback: Optional[CallbackInfo] = None
```

---

## API Response Wrappers

### Standard Response Format

```python
from typing import Generic, TypeVar, Optional

T = TypeVar('T')

class APIResponse(BaseModel, Generic[T]):
    timestamp: datetime
    status: int
    message: str
    data: Optional[T] = None
    error: Optional[Dict[str, Any]] = None
    request_id: Optional[str] = None

class PaginatedResponse(BaseModel, Generic[T]):
    items: List[T]
    total_count: int
    page: int
    page_size: int
    total_pages: int
    has_next: bool
    has_previous: bool

class AsyncTaskResponse(BaseModel):
    task_id: str
    status: str  # "pending", "processing", "completed", "failed"
    progress: float  # 0.0 to 1.0
    estimated_completion: Optional[datetime] = None
    result_url: Optional[HttpUrl] = None
    error_message: Optional[str] = None
```

---

## Configuration Schemas

### Service Configuration

```python
class DatabaseConfig(BaseModel):
    host: str
    port: int = 5432
    database: str
    username: str
    password: str
    pool_size: int = 10
    max_overflow: int = 20

class RedisConfig(BaseModel):
    host: str
    port: int = 6379
    password: Optional[str] = None
    database: int = 0
    connection_pool_size: int = 10

class RabbitMQConfig(BaseModel):
    host: str
    port: int = 5672
    username: str
    password: str
    virtual_host: str = "/"
    connection_timeout: int = 30

class B2Config(BaseModel):
    application_key_id: str
    application_key: str
    bucket_name: str
    endpoint_url: str
    region: str = "us-west-004"

class AIServiceConfig(BaseModel):
    database: DatabaseConfig
    redis: RedisConfig
    rabbitmq: RabbitMQConfig
    b2_storage: B2Config
    log_level: str = "INFO"
    debug: bool = False
    max_workers: int = 4
    request_timeout: int = 300
```

---

*This schema documentation provides the complete data model definitions for the AI services integration with the ScholarAI platform. All schemas are designed to be extensible and maintain backward compatibility.*
