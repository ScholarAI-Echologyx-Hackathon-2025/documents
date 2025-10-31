# Extractor Service API Documentation

## Overview

The Extractor Service is a sophisticated PDF content extraction system designed for academic papers. It provides comprehensive extraction capabilities using multiple methods with intelligent fallback strategies.

**Base URL:** `http://localhost:8084`  
**Framework:** FastAPI  
**Content Type:** `application/json`

---

## Features

### Multi-Method Extraction Pipeline
- **GROBID**: Text structure and metadata extraction
- **PDFFigures2**: Figure detection and extraction
- **Table Transformer**: Advanced table recognition
- **Nougat OCR**: Mathematical content extraction
- **Tesseract OCR**: Scanned document processing
- **Computer Vision**: Image and diagram extraction

### Content Types Extracted
- Hierarchical text structure (title, sections, paragraphs)
- Figures with captions and metadata
- Tables with structure and content
- Mathematical equations (LaTeX format)
- Code blocks and algorithms
- References and citations
- Named entities and metadata

---

## API Endpoints

### 1. Extract from B2 URL

**Endpoint:** `POST /extract/b2-url`

**Description:** Extract content from a PDF stored in Backblaze B2 storage.

**Request Body:**
```json
{
  "url": "https://s3.us-west-004.backblazeb2.com/bucket/paper.pdf",
  "config": {
    "extraction_methods": ["grobid", "pdfplumber", "ocr"],
    "include_figures": true,
    "include_tables": true,
    "include_equations": true,
    "quality_threshold": 0.8
  }
}
```

**Response (200 - Success):**
```json
{
  "paper_id": "unique-paper-id",
  "metadata": {
    "title": "Research Paper Title",
    "authors": ["Author 1", "Author 2"],
    "abstract": "Paper abstract...",
    "doi": "10.1000/182",
    "publication_date": "2024-01-15",
    "venue": "Conference/Journal Name"
  },
  "content": {
    "sections": [
      {
        "title": "Introduction",
        "content": "Section content...",
        "subsections": []
      }
    ],
    "figures": [
      {
        "id": "fig1",
        "caption": "Figure caption",
        "url": "https://b2-url/extracted-figure-1.png",
        "page": 1,
        "bbox": [100, 200, 300, 400]
      }
    ],
    "tables": [
      {
        "id": "table1",
        "caption": "Table caption",
        "data": [["Header1", "Header2"], ["Data1", "Data2"]],
        "page": 2
      }
    ],
    "equations": [
      {
        "latex": "E = mc^2",
        "page": 3,
        "bbox": [150, 250, 350, 280]
      }
    ],
    "references": [
      {
        "id": "ref1",
        "title": "Referenced Paper",
        "authors": ["Ref Author"],
        "year": 2023
      }
    ]
  },
  "extraction_quality": {
    "overall_score": 0.95,
    "text_coverage": 0.98,
    "figure_detection": 0.92,
    "table_extraction": 0.89,
    "equation_recognition": 0.94
  },
  "processing_time": 45.2
}
```

### 2. Extract from File Upload

**Endpoint:** `POST /extract/upload`

**Description:** Extract content from an uploaded PDF file.

**Request:** Multipart form data with PDF file

**Response:** Same structure as B2 URL extraction

### 3. Get Extraction Status

**Endpoint:** `GET /extract/status/{task_id}`

**Description:** Check the status of an ongoing extraction task.

**Response:**
```json
{
  "task_id": "task-uuid",
  "status": "processing|completed|failed",
  "progress": 0.75,
  "estimated_completion": "2024-01-15T10:30:00Z",
  "current_stage": "figure_extraction",
  "error_message": null
}
```

### 4. Configure Extraction Pipeline

**Endpoint:** `POST /config/pipeline`

**Description:** Configure extraction methods and parameters.

**Request Body:**
```json
{
  "config_name": "academic_paper_config",
  "methods": {
    "grobid": {
      "enabled": true,
      "consolidate_citations": true,
      "include_raw_affiliations": false
    },
    "pdfplumber": {
      "enabled": true,
      "table_detection_threshold": 0.8
    },
    "ocr": {
      "enabled": true,
      "languages": ["eng"],
      "detect_scanned": true
    }
  },
  "quality_settings": {
    "minimum_confidence": 0.7,
    "fallback_methods": true,
    "deduplication": true
  }
}
```

### 5. Health Check

**Endpoint:** `GET /health`

**Description:** Check service health and component status.

**Response:**
```json
{
  "status": "healthy",
  "components": {
    "grobid": "running",
    "b2_storage": "connected",
    "rabbitmq": "connected",
    "ocr_engines": "ready"
  },
  "version": "1.0.0",
  "uptime": 3600
}
```

---

## RabbitMQ Integration

### Message Format

The service consumes extraction requests from RabbitMQ:

```json
{
  "message_id": "unique-message-id",
  "source": "paper-search-service",
  "task_type": "pdf_extraction",
  "payload": {
    "paper_id": "paper-uuid",
    "pdf_url": "https://b2-url/paper.pdf",
    "config": {
      "extraction_level": "full",
      "priority": "normal"
    }
  },
  "callback": {
    "service": "paper-search",
    "endpoint": "/extraction-complete",
    "method": "POST"
  }
}
```

### Queue Configuration

- **Exchange:** `extraction.direct`
- **Queue:** `pdf.extraction.requests`
- **Routing Key:** `pdf.extract`
- **Dead Letter Queue:** `pdf.extraction.failed`

---

## Configuration Options

### Extraction Levels

| Level | Description | Methods Used | Processing Time |
|:------|:------------|:-------------|:----------------|
| `fast` | Basic text extraction | GROBID only | ~10 seconds |
| `standard` | Complete extraction | GROBID + PDFPlumber | ~30 seconds |
| `comprehensive` | All methods | All extractors + OCR | ~60 seconds |
| `custom` | User-defined | Selected methods | Variable |

### Quality Thresholds

| Metric | Minimum | Good | Excellent |
|:-------|:--------|:-----|:----------|
| Text Coverage | 0.6 | 0.8 | 0.95 |
| Figure Detection | 0.5 | 0.7 | 0.9 |
| Table Extraction | 0.4 | 0.6 | 0.85 |
| Equation Recognition | 0.3 | 0.6 | 0.8 |

---

## Error Handling

### Common Error Codes

| Code | Status | Description |
|:-----|:-------|:------------|
| `INVALID_URL` | 400 | Invalid or inaccessible PDF URL |
| `EXTRACTION_FAILED` | 500 | Extraction pipeline failed |
| `UNSUPPORTED_FORMAT` | 400 | File format not supported |
| `TIMEOUT` | 408 | Extraction timeout exceeded |
| `STORAGE_ERROR` | 503 | B2 storage connection failed |

### Error Response Format

```json
{
  "error": {
    "code": "EXTRACTION_FAILED",
    "message": "PDF extraction failed at figure detection stage",
    "details": {
      "stage": "figure_extraction",
      "method": "pdfplumber",
      "reason": "Corrupted PDF structure"
    },
    "suggestion": "Try using OCR-based extraction method",
    "retry_possible": true
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

---

## Performance Metrics

### Processing Times (Average)

| Document Type | Pages | Fast Mode | Standard Mode | Comprehensive Mode |
|:--------------|:------|:----------|:--------------|:-------------------|
| Text-only Paper | 10 | 8s | 15s | 25s |
| Figure-rich Paper | 15 | 12s | 35s | 65s |
| Math-heavy Paper | 12 | 10s | 45s | 85s |
| Scanned Document | 20 | N/A | 60s | 120s |

### Resource Requirements

- **CPU**: 2-4 cores per extraction task
- **Memory**: 4-8 GB RAM per task
- **Storage**: 100-500 MB temporary space per document
- **Network**: High bandwidth for B2 downloads

---

## Integration Examples

### Python Client Example

```python
import requests
import json

# Extract from B2 URL
extraction_request = {
    "url": "https://b2-url/paper.pdf",
    "config": {
        "extraction_methods": ["grobid", "pdfplumber"],
        "include_figures": True,
        "quality_threshold": 0.8
    }
}

response = requests.post(
    "http://localhost:8084/extract/b2-url",
    json=extraction_request
)

if response.status_code == 200:
    result = response.json()
    print(f"Extracted {len(result['content']['sections'])} sections")
    print(f"Found {len(result['content']['figures'])} figures")
    print(f"Quality score: {result['extraction_quality']['overall_score']}")
```

### JavaScript Client Example

```javascript
const extractPDF = async (pdfUrl) => {
  const response = await fetch('http://localhost:8084/extract/b2-url', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      url: pdfUrl,
      config: {
        extraction_methods: ['grobid', 'pdfplumber'],
        include_figures: true,
        quality_threshold: 0.8
      }
    })
  });
  
  const result = await response.json();
  return result;
};
```

---

*For more detailed information about specific extraction methods and advanced configuration options, refer to the technical documentation in the extractor repository.*
