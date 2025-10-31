# Paper Search Service API Documentation

## Overview

The Paper Search Service provides comprehensive academic paper search capabilities across multiple databases with AI-powered query refinement, PDF collection, and intelligent deduplication.

**Base URL:** `http://localhost:8085`  
**Framework:** FastAPI  
**Content Type:** `application/json`

---

## Features

### Multi-Source Search Integration
- **arXiv**: Preprint server search
- **Semantic Scholar**: Comprehensive academic database
- **PubMed/PMC**: Medical and life sciences
- **CrossRef**: DOI and citation metadata
- **OpenAlex**: Open academic database

### Advanced Capabilities
- **AI-powered query refinement** and expansion
- **Aggressive PDF collection** from multiple sources
- **Intelligent deduplication** across databases
- **Metadata enrichment** and validation
- **B2 cloud storage** integration
- **RabbitMQ messaging** for async processing

---

## API Endpoints

### 1. Multi-Source Paper Search

**Endpoint:** `POST /search/papers`

**Description:** Search for academic papers across multiple databases with AI query refinement.

**Request Body:**
```json
{
  "query": "machine learning neural networks",
  "filters": {
    "sources": ["arxiv", "semantic_scholar", "pubmed"],
    "date_range": {
      "start": "2020-01-01",
      "end": "2024-12-31"
    },
    "publication_types": ["journal", "conference"],
    "fields_of_study": ["Computer Science", "Medicine"],
    "min_citations": 10,
    "max_results_per_source": 50
  },
  "ai_refinement": {
    "enabled": true,
    "expand_query": true,
    "suggest_related_terms": true
  },
  "pdf_collection": {
    "enabled": true,
    "priority": "high_impact_first"
  }
}
```

**Response (200 - Success):**
```json
{
  "search_id": "search-uuid",
  "original_query": "machine learning neural networks",
  "refined_queries": [
    "machine learning neural networks",
    "artificial neural networks deep learning",
    "neural network architectures training"
  ],
  "results": {
    "total_papers": 127,
    "unique_papers": 89,
    "sources_searched": ["arxiv", "semantic_scholar", "pubmed"],
    "search_time": 12.4,
    "papers": [
      {
        "id": "paper-uuid",
        "title": "Deep Neural Networks for Computer Vision",
        "authors": [
          {
            "name": "John Doe",
            "affiliation": "Stanford University",
            "orcid": "0000-0000-0000-0000"
          }
        ],
        "abstract": "This paper presents...",
        "metadata": {
          "doi": "10.1000/182",
          "arxiv_id": "2024.01234",
          "pmid": "12345678",
          "publication_date": "2024-01-15",
          "venue": "ICLR 2024",
          "venue_type": "conference",
          "fields_of_study": ["Computer Science", "Machine Learning"]
        },
        "metrics": {
          "citation_count": 45,
          "citation_velocity": 2.3,
          "h_index": 12,
          "altmetric_score": 78,
          "influence_score": 8.7
        },
        "pdf_info": {
          "available": true,
          "url": "https://arxiv.org/pdf/2024.01234.pdf",
          "b2_url": "https://s3.us-west-004.backblazeb2.com/bucket/papers/paper-uuid.pdf",
          "file_size": 2.4,
          "pages": 12,
          "extraction_status": "completed"
        },
        "relevance_score": 0.94,
        "quality_score": 0.87,
        "source": "arxiv",
        "discovered_at": "2024-01-15T10:30:00Z"
      }
    ]
  },
  "aggregations": {
    "by_year": {"2024": 45, "2023": 32, "2022": 12},
    "by_venue": {"ICLR": 15, "NeurIPS": 12, "Nature": 8},
    "by_field": {"Computer Science": 67, "Medicine": 22},
    "by_source": {"arxiv": 45, "semantic_scholar": 32, "pubmed": 12}
  },
  "suggestions": {
    "related_queries": [
      "transformer neural networks",
      "convolutional neural networks",
      "recurrent neural networks"
    ],
    "trending_topics": [
      "attention mechanisms",
      "self-supervised learning",
      "neural architecture search"
    ]
  }
}
```

### 2. Search by DOI

**Endpoint:** `GET /search/doi/{doi}`

**Description:** Retrieve comprehensive paper information by DOI.

**Path Parameters:**
- `doi`: The DOI identifier (URL encoded)

**Response:**
```json
{
  "paper": {
    "id": "paper-uuid",
    "doi": "10.1000/182",
    "title": "Paper Title",
    "authors": [...],
    "metadata": {...},
    "full_text_available": true,
    "pdf_url": "https://b2-url/paper.pdf",
    "extraction_completed": true
  },
  "enrichment": {
    "citation_graph": {
      "cited_by_count": 45,
      "references_count": 32,
      "citation_velocity": 2.3
    },
    "impact_metrics": {
      "h_index": 12,
      "field_citation_ratio": 1.8,
      "relative_citation_ratio": 2.1
    }
  }
}
```

### 3. Author-Based Search

**Endpoint:** `POST /search/author`

**Description:** Search for papers by specific authors.

**Request Body:**
```json
{
  "authors": [
    {
      "name": "John Doe",
      "orcid": "0000-0000-0000-0000",
      "affiliation": "Stanford University"
    }
  ],
  "include_collaborations": true,
  "date_range": {
    "start": "2020-01-01",
    "end": "2024-12-31"
  }
}
```

### 4. Citation Network Analysis

**Endpoint:** `POST /analysis/citations`

**Description:** Analyze citation networks and discover research paths.

**Request Body:**
```json
{
  "paper_ids": ["paper-uuid-1", "paper-uuid-2"],
  "analysis_depth": 3,
  "include_co_citations": true,
  "min_citation_threshold": 5
}
```

**Response:**
```json
{
  "network": {
    "nodes": [
      {
        "id": "paper-uuid",
        "title": "Paper Title",
        "year": 2024,
        "citation_count": 45,
        "centrality_score": 0.78
      }
    ],
    "edges": [
      {
        "source": "paper-uuid-1",
        "target": "paper-uuid-2",
        "relationship": "cites",
        "weight": 1.0
      }
    ]
  },
  "insights": {
    "key_papers": ["paper-uuid-3", "paper-uuid-7"],
    "research_clusters": [
      {
        "topic": "Neural Networks",
        "papers": ["uuid1", "uuid2"],
        "influence_score": 8.9
      }
    ],
    "citation_paths": [
      {
        "from": "paper-uuid-1",
        "to": "paper-uuid-5",
        "path": ["uuid1", "uuid3", "uuid5"],
        "path_strength": 0.85
      }
    ]
  }
}
```

### 5. Trending Topics Discovery

**Endpoint:** `GET /discover/trending`

**Description:** Discover trending research topics and emerging fields.

**Query Parameters:**
- `field`: Field of study filter
- `time_window`: Time window for trend analysis (7d, 30d, 90d, 1y)
- `min_paper_count`: Minimum papers for trend consideration

**Response:**
```json
{
  "trending_topics": [
    {
      "topic": "Large Language Models",
      "growth_rate": 2.4,
      "paper_count": 234,
      "key_terms": ["transformer", "attention", "pre-training"],
      "representative_papers": ["uuid1", "uuid2"],
      "trend_score": 0.89
    }
  ],
  "emerging_authors": [
    {
      "name": "Jane Smith",
      "growth_rate": 3.1,
      "recent_publications": 8,
      "impact_increase": 0.67
    }
  ],
  "hot_venues": [
    {
      "venue": "ICLR 2024",
      "topic_diversity": 0.78,
      "citation_velocity": 2.1,
      "breakthrough_papers": 12
    }
  ]
}
```

### 6. Bulk PDF Processing

**Endpoint:** `POST /processing/bulk-pdfs`

**Description:** Process multiple PDFs for content extraction and indexing.

**Request Body:**
```json
{
  "papers": [
    {
      "paper_id": "uuid1",
      "pdf_url": "https://arxiv.org/pdf/2024.01234.pdf",
      "priority": "high"
    }
  ],
  "extraction_config": {
    "level": "comprehensive",
    "include_figures": true,
    "include_tables": true
  },
  "callback_url": "https://api.scholarai.com/extraction-complete"
}
```

### 7. Search Analytics

**Endpoint:** `GET /analytics/search-stats`

**Description:** Get search analytics and usage statistics.

**Response:**
```json
{
  "period": "last_30_days",
  "statistics": {
    "total_searches": 1547,
    "unique_queries": 892,
    "papers_discovered": 15670,
    "pdfs_collected": 8934,
    "average_response_time": 8.2
  },
  "popular_queries": [
    {"query": "machine learning", "count": 87},
    {"query": "covid-19 treatment", "count": 64}
  ],
  "source_performance": {
    "arxiv": {"searches": 567, "success_rate": 0.95},
    "semantic_scholar": {"searches": 432, "success_rate": 0.92}
  }
}
```

---

## RabbitMQ Integration

### Message Processing

The service processes search requests via RabbitMQ for scalable async operations:

**Incoming Message Format:**
```json
{
  "message_id": "msg-uuid",
  "source_service": "frontend",
  "task_type": "paper_search",
  "payload": {
    "user_id": "user-uuid",
    "query": "machine learning",
    "filters": {...},
    "callback": {
      "service": "user-service",
      "endpoint": "/search-complete",
      "method": "POST"
    }
  },
  "priority": "normal",
  "timestamp": "2024-01-15T10:30:00Z"
}
```

**Outgoing Message Format:**
```json
{
  "message_id": "msg-uuid",
  "task_type": "pdf_extraction_request",
  "destination": "extractor-service",
  "payload": {
    "paper_id": "paper-uuid",
    "pdf_url": "https://b2-url/paper.pdf",
    "extraction_config": {
      "level": "standard",
      "priority": "normal"
    }
  }
}
```

### Queue Configuration

- **Exchange:** `paper.search.direct`
- **Queues:**
  - `search.requests` - Incoming search requests
  - `search.results` - Completed search results
  - `pdf.collection` - PDF collection tasks
  - `extraction.requests` - Extraction task requests

---

## Configuration Options

### Search Sources

| Source | API Limit | Coverage | Response Time |
|:-------|:----------|:---------|:--------------|
| arXiv | 1000/day | Preprints | ~2s |
| Semantic Scholar | 5000/day | 200M+ papers | ~3s |
| PubMed | 10000/day | Life sciences | ~4s |
| CrossRef | 100/min | DOI metadata | ~1s |
| OpenAlex | No limit | 200M+ works | ~2s |

### PDF Collection Strategies

| Strategy | Description | Success Rate | Speed |
|:---------|:------------|:-------------|:------|
| `direct_download` | Direct PDF URLs | 85% | Fast |
| `publisher_api` | Publisher APIs | 60% | Medium |
| `institutional_access` | University proxies | 95% | Slow |
| `preprint_servers` | arXiv, bioRxiv | 90% | Fast |

---

## Advanced Features

### AI Query Refinement

The service uses advanced NLP to improve search queries:

1. **Query Expansion**: Add synonyms and related terms
2. **Concept Extraction**: Identify key research concepts
3. **Intent Recognition**: Understand search intent
4. **Domain Adaptation**: Adjust queries by field of study

### Deduplication Algorithm

Papers are deduplicated using multiple signals:
- DOI matching (highest confidence)
- Title similarity (90%+ threshold)
- Author overlap + venue matching
- Abstract similarity (85%+ threshold)
- arXiv ID to published version linking

### Quality Scoring

Papers receive quality scores based on:
- Venue reputation (impact factor, h-index)
- Author reputation (h-index, citation count)
- Citation metrics (count, velocity, field-normalized)
- Content quality (abstract completeness, reference count)

---

## Error Handling

### Common Error Codes

| Code | Status | Description |
|:-----|:-------|:------------|
| `INVALID_QUERY` | 400 | Query format invalid or empty |
| `SOURCE_UNAVAILABLE` | 503 | External search source down |
| `RATE_LIMIT_EXCEEDED` | 429 | API rate limit exceeded |
| `NO_RESULTS_FOUND` | 404 | No papers match criteria |
| `PDF_COLLECTION_FAILED` | 502 | PDF download failed |

### Retry Logic

- **Source failures**: Automatic retry with exponential backoff
- **Rate limits**: Queue requests for retry after reset
- **Network errors**: Retry up to 3 times with different endpoints

---

## Performance Optimization

### Caching Strategy

- **Query Results**: 1-hour cache for identical queries
- **Paper Metadata**: 24-hour cache for enriched metadata
- **PDF URLs**: 7-day cache for download links
- **Author Profiles**: 7-day cache for author information

### Parallel Processing

- **Multi-source searches**: Parallel API calls
- **PDF downloads**: Concurrent download workers
- **Content extraction**: Async processing via RabbitMQ
- **Deduplication**: Background clustering algorithms

---

## Integration Examples

### Python Client

```python
import asyncio
import aiohttp

async def search_papers(query):
    async with aiohttp.ClientSession() as session:
        payload = {
            "query": query,
            "filters": {
                "sources": ["arxiv", "semantic_scholar"],
                "date_range": {"start": "2022-01-01", "end": "2024-12-31"}
            },
            "ai_refinement": {"enabled": True},
            "pdf_collection": {"enabled": True}
        }
        
        async with session.post(
            "http://localhost:8085/search/papers",
            json=payload
        ) as response:
            result = await response.json()
            return result

# Usage
results = asyncio.run(search_papers("neural networks"))
print(f"Found {results['results']['total_papers']} papers")
```

### Frontend Integration

```javascript
const searchPapers = async (query, filters = {}) => {
  const response = await fetch('http://localhost:8085/search/papers', {
    method: 'POST',
    headers: {'Content-Type': 'application/json'},
    body: JSON.stringify({
      query,
      filters: {
        sources: ['arxiv', 'semantic_scholar'],
        ...filters
      },
      ai_refinement: { enabled: true },
      pdf_collection: { enabled: true }
    })
  });
  
  return await response.json();
};

// Usage with real-time updates
const results = await searchPapers("machine learning");
console.log(`Search completed in ${results.results.search_time}s`);
```

---

*For detailed information about specific search algorithms, API rate limits, and advanced configuration options, refer to the paper-search service repository documentation.*
