# AI Services Integration Guide

## Overview

This guide provides comprehensive instructions for integrating the Extractor and Paper Search AI services with the main ScholarAI platform. It covers deployment, configuration, testing, and monitoring of the integrated system.

---

## 🚀 Quick Integration Setup

### Prerequisites

Before integrating AI services, ensure you have:

- **Meta repository** cloned with all submodules
- **Docker and Docker Compose** installed and operational
- **Platform services** running (User Service, API Gateway, etc.)
- **RabbitMQ and PostgreSQL** instances available
- **Backblaze B2 storage** configured with credentials
- **Python 3.11+** environment for AI services

### Integration Architecture

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   Frontend      │    │   API Gateway    │    │  Microservices  │
│   (Port 3000)   │◄──►│   (Port 8989)    │◄──►│  (8081-8083)    │
└─────────────────┘    └──────────────────┘    └─────────────────┘
                                │
                                ▼
                       ┌──────────────────┐
                       │   RabbitMQ       │
                       │   (Port 5672)    │
                       └──────────────────┘
                                │
                    ┌───────────┴───────────┐
                    ▼                       ▼
           ┌─────────────────┐    ┌─────────────────┐
           │ Extractor       │    │ Paper Search    │
           │ Service         │    │ Service         │
           │ (Port 8084)     │    │ (Port 8085)     │
           └─────────────────┘    └─────────────────┘
                    │                       │
                    └───────────┬───────────┘
                                ▼
                       ┌──────────────────┐
                       │ Backblaze B2     │
                       │ Storage          │
                       └──────────────────┘
```

---

## 📦 Step 1: Add AI Services as Submodules

### Update Meta Repository

Add the AI services to your meta repository structure:

```bash
cd /path/to/meta

# Add extractor service as submodule
git submodule add https://github.com/extractor.git AI-Services/extractor

# Add paper-search service as submodule  
git submodule add https://github.com/paper-search.git AI-Services/paper-search

# Update .gitmodules file
git add .gitmodules AI-Services/
git commit -m "Add AI services as submodules"
git push origin main
```

### Updated Meta Repository Structure

```
meta/
├── Frontend/                    # → frontend submodule
├── Microservices/
│   ├── api-gateway/
│   ├── user-service/
│   ├── project-service/
│   ├── notification-service/
│   └── service-registry/
├── AI-Services/                 # → New AI services directory
│   ├── extractor/              # → extractor submodule
│   └── paper-search/           # → paper-search submodule
├── Docker/
├── Scripts/
└── .gitmodules
```

---

## 🐳 Step 2: Docker Integration

### Update Docker Compose Configuration

Add AI services to your Docker orchestration:

**File: `Docker/ai-services.yml`**

```yaml
services:
  # AI Service - PDF Extractor
  extractor-service:
    build:
      context: ../AI-Services/extractor
      dockerfile: Dockerfile
    container_name: scholar-extractor
    restart: unless-stopped
    ports:
      - "8084:8000"
    environment:
      - ENVIRONMENT=docker
      - RABBITMQ_URL=amqp://rabbitmq:5672
      - B2_APPLICATION_KEY_ID=${B2_APPLICATION_KEY_ID}
      - B2_APPLICATION_KEY=${B2_APPLICATION_KEY}
      - B2_BUCKET_NAME=${B2_BUCKET_NAME}
      - POSTGRES_URL=postgresql://extractor_user:${EXTRACTOR_DB_PASSWORD}@extractor-postgres:5432/extractorDB
      - REDIS_URL=redis://redis:6379/2
    volumes:
      - extractor-temp:/app/temp
      - extractor-models:/app/models
    networks:
      - scholarai-network
    depends_on:
      - rabbitmq
      - redis
      - extractor-postgres
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 120s

  # AI Service - Paper Search
  paper-search-service:
    build:
      context: ../AI-Services/paper-search
      dockerfile: Dockerfile
    container_name: scholar-paper-search
    restart: unless-stopped
    ports:
      - "8085:8000"
    environment:
      - ENVIRONMENT=docker
      - RABBITMQ_URL=amqp://rabbitmq:5672
      - B2_APPLICATION_KEY_ID=${B2_APPLICATION_KEY_ID}
      - B2_APPLICATION_KEY=${B2_APPLICATION_KEY}
      - B2_BUCKET_NAME=${B2_BUCKET_NAME}
      - POSTGRES_URL=postgresql://search_user:${SEARCH_DB_PASSWORD}@search-postgres:5432/searchDB
      - REDIS_URL=redis://redis:6379/3
      - SEMANTIC_SCHOLAR_API_KEY=${SEMANTIC_SCHOLAR_API_KEY}
      - ARXIV_API_DELAY=3
    volumes:
      - search-cache:/app/cache
      - search-models:/app/models
    networks:
      - scholarai-network
    depends_on:
      - rabbitmq
      - redis
      - search-postgres
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 90s

  # Database for Extractor Service
  extractor-postgres:
    image: 'postgres:17-alpine'
    container_name: extractor-db
    restart: always
    environment:
      POSTGRES_USER: extractor_user
      POSTGRES_PASSWORD: ${EXTRACTOR_DB_PASSWORD}
      POSTGRES_DB: extractorDB
    ports:
      - "5435:5432"
    volumes:
      - extractor-postgres-data:/var/lib/postgresql/data
      - ../AI-Services/extractor/scripts/init.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - scholarai-network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U extractor_user -d extractorDB"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Database for Paper Search Service
  search-postgres:
    image: 'postgres:17-alpine'
    container_name: search-db
    restart: always
    environment:
      POSTGRES_USER: search_user
      POSTGRES_PASSWORD: ${SEARCH_DB_PASSWORD}
      POSTGRES_DB: searchDB
    ports:
      - "5436:5432"
    volumes:
      - search-postgres-data:/var/lib/postgresql/data
      - ../AI-Services/paper-search/scripts/init.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - scholarai-network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U search_user -d searchDB"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  extractor-postgres-data:
  search-postgres-data:
  extractor-temp:
  extractor-models:
  search-cache:
  search-models:

networks:
  scholarai-network:
    external: true
```

### Update Environment Variables

**File: `Docker/.env`**

```bash
# Existing variables...

# AI Services Configuration
EXTRACTOR_DB_PASSWORD=extractor_secure_password
SEARCH_DB_PASSWORD=search_secure_password

# Backblaze B2 Configuration
B2_APPLICATION_KEY_ID=your_b2_key_id
B2_APPLICATION_KEY=your_b2_application_key
B2_BUCKET_NAME=scholarai-papers

# External API Keys
SEMANTIC_SCHOLAR_API_KEY=your_semantic_scholar_key
GOOGLE_API_KEY=your_google_api_key
```

---

## 🔧 Step 3: API Gateway Integration

### Add AI Service Routes

Update your API Gateway configuration to route requests to AI services:

**File: `Microservices/api-gateway/src/main/resources/application-docker.yml`**

```yaml
spring:
  cloud:
    gateway:
      routes:
        # Existing routes...
        
        # Extractor Service Routes
        - id: extractor-service
          uri: http://extractor-service:8000
          predicates:
            - Path=/api/v1/extract/**
          filters:
            - StripPrefix=3
            - name: Retry
              args:
                retries: 3
                statuses: SERVICE_UNAVAILABLE,INTERNAL_SERVER_ERROR
            - name: CircuitBreaker
              args:
                name: extractorCircuitBreaker
                fallbackUri: forward:/fallback/extractor
        
        # Paper Search Service Routes  
        - id: paper-search-service
          uri: http://paper-search-service:8000
          predicates:
            - Path=/api/v1/search/**
          filters:
            - StripPrefix=3
            - name: Retry
              args:
                retries: 3
                statuses: SERVICE_UNAVAILABLE,INTERNAL_SERVER_ERROR
            - name: CircuitBreaker
              args:
                name: searchCircuitBreaker
                fallbackUri: forward:/fallback/search

        # AI Services Health Check
        - id: ai-health-check
          uri: http://extractor-service:8000
          predicates:
            - Path=/api/v1/ai/health/**
          filters:
            - StripPrefix=4

resilience4j:
  circuitbreaker:
    instances:
      extractorCircuitBreaker:
        slidingWindowSize: 10
        failureRateThreshold: 50
        waitDurationInOpenState: 30s
      searchCircuitBreaker:
        slidingWindowSize: 10
        failureRateThreshold: 50
        waitDurationInOpenState: 30s
```

---

## 🔌 Step 4: RabbitMQ Queue Configuration

### Queue Setup for AI Services

**File: `Microservices/notification-service/src/main/resources/rabbitmq-config.yml`**

```yaml
rabbitmq:
  exchanges:
    - name: ai.services.direct
      type: direct
      durable: true
      
  queues:
    # Extractor Service Queues
    - name: extractor.requests
      durable: true
      arguments:
        x-dead-letter-exchange: ai.services.dlx
        x-dead-letter-routing-key: extractor.failed
        x-message-ttl: 1800000  # 30 minutes
        
    - name: extractor.results
      durable: true
      
    - name: extractor.failed
      durable: true
      
    # Paper Search Service Queues
    - name: search.requests
      durable: true
      arguments:
        x-dead-letter-exchange: ai.services.dlx
        x-dead-letter-routing-key: search.failed
        x-message-ttl: 900000   # 15 minutes
        
    - name: search.results
      durable: true
      
    - name: search.failed
      durable: true
      
    # Priority Queues
    - name: extractor.priority
      durable: true
      arguments:
        x-max-priority: 10
        
    - name: search.priority
      durable: true
      arguments:
        x-max-priority: 10

  bindings:
    - exchange: ai.services.direct
      queue: extractor.requests
      routing_key: extract.pdf
      
    - exchange: ai.services.direct
      queue: search.requests
      routing_key: search.papers
      
    - exchange: ai.services.direct
      queue: extractor.priority
      routing_key: extract.priority
      
    - exchange: ai.services.direct
      queue: search.priority
      routing_key: search.priority
```

---

## 💻 Step 5: Service Integration Code

### Project Service Integration

Add AI service integration to your Project Service:

**File: `Microservices/project-service/src/main/java/com/scholarai/project/service/PaperSearchService.java`**

```java
@Service
@Slf4j
public class PaperSearchService {
    
    @Autowired
    private RabbitTemplate rabbitTemplate;
    
    @Autowired
    private WebClient.Builder webClientBuilder;
    
    @Value("${ai.services.search.url}")
    private String searchServiceUrl;
    
    public Mono<PaperSearchResponseDto> searchPapers(PaperSearchRequestDto request) {
        return webClientBuilder.build()
            .post()
            .uri(searchServiceUrl + "/search/papers")
            .bodyValue(request)
            .retrieve()
            .bodyToMono(ApiResponse.class)
            .map(response -> (PaperSearchResponseDto) response.getData())
            .doOnSuccess(result -> log.info("Search completed: {} papers found", 
                result.getResults().getTotalPapers()))
            .doOnError(error -> log.error("Search failed", error));
    }
    
    public void requestPaperExtraction(String paperId, String pdfUrl) {
        ExtractionRequestMessage message = ExtractionRequestMessage.builder()
            .header(MessageHeader.builder()
                .messageId(UUID.randomUUID().toString())
                .sourceService("project-service")
                .destinationService("extractor-service")
                .taskType("pdf_extraction")
                .timestamp(LocalDateTime.now())
                .build())
            .payload(Map.of(
                "paperId", paperId,
                "pdfUrl", pdfUrl,
                "extractionConfig", Map.of(
                    "extractionLevel", "standard",
                    "includeFigures", true,
                    "includeTables", true
                )
            ))
            .build();
            
        rabbitTemplate.convertAndSend("ai.services.direct", "extract.pdf", message);
        log.info("Extraction request sent for paper: {}", paperId);
    }
    
    @RabbitListener(queues = "extractor.results")
    public void handleExtractionResult(ExtractionResultDto result) {
        log.info("Extraction completed for paper: {}", result.getPaperId());
        
        // Update paper status in database
        paperRepository.updateExtractionStatus(
            result.getPaperId(), 
            "completed", 
            result.getExtractionQuality().getOverallScore()
        );
        
        // Store extracted content
        extractedContentService.saveContent(result);
        
        // Notify user if required
        notificationService.sendExtractionCompleteNotification(result);
    }
}
```

### User Service Integration

Add search history and preferences to User Service:

**File: `Microservices/user-service/src/main/java/com/scholarai/user/service/SearchPreferencesService.java`**

```java
@Service
@Slf4j
public class SearchPreferencesService {
    
    @Autowired
    private SearchHistoryRepository searchHistoryRepository;
    
    @Autowired
    private UserPreferencesRepository userPreferencesRepository;
    
    public void recordSearchQuery(String userId, PaperSearchRequestDto request, 
                                 PaperSearchResponseDto response) {
        SearchHistory history = SearchHistory.builder()
            .id(UUID.randomUUID())
            .userId(UUID.fromString(userId))
            .query(request.getQuery())
            .filters(convertFiltersToJson(request.getFilters()))
            .resultsCount(response.getResults().getTotalPapers())
            .searchTime(response.getResults().getSearchTime())
            .createdAt(LocalDateTime.now())
            .build();
            
        searchHistoryRepository.save(history);
        log.info("Search history recorded for user: {}", userId);
    }
    
    public List<String> getSuggestedQueries(String userId) {
        List<SearchHistory> recentSearches = searchHistoryRepository
            .findTop10ByUserIdOrderByCreatedAtDesc(UUID.fromString(userId));
            
        return recentSearches.stream()
            .map(SearchHistory::getQuery)
            .distinct()
            .collect(Collectors.toList());
    }
    
    public UserSearchPreferences getSearchPreferences(String userId) {
        return userPreferencesRepository
            .findByUserId(UUID.fromString(userId))
            .map(this::convertToSearchPreferences)
            .orElse(UserSearchPreferences.getDefault());
    }
}
```

---

## 🧪 Step 6: Testing Integration

### Integration Test Setup

Create comprehensive integration tests:

**File: `tests/integration/ai-services-integration-test.py`**

```python
import pytest
import asyncio
import aiohttp
import json
from datetime import datetime

class TestAIServicesIntegration:
    
    @pytest.fixture
    async def setup_services(self):
        """Ensure all services are running and healthy"""
        services = [
            "http://localhost:8989",  # API Gateway
            "http://localhost:8084",  # Extractor Service
            "http://localhost:8085"   # Paper Search Service
        ]
        
        async with aiohttp.ClientSession() as session:
            for service_url in services:
                health_url = f"{service_url}/health"
                async with session.get(health_url) as response:
                    assert response.status == 200
                    
    async def test_paper_search_flow(self, setup_services):
        """Test complete paper search workflow"""
        search_request = {
            "query": "machine learning neural networks",
            "filters": {
                "sources": ["arxiv", "semantic_scholar"],
                "date_range": {
                    "start": "2022-01-01",
                    "end": "2024-12-31"
                },
                "max_results_per_source": 10
            },
            "ai_refinement": {"enabled": True},
            "pdf_collection": {"enabled": True}
        }
        
        async with aiohttp.ClientSession() as session:
            # Send search request through API Gateway
            async with session.post(
                "http://localhost:8989/api/v1/search/papers",
                json=search_request,
                headers={"Authorization": "Bearer test_token"}
            ) as response:
                assert response.status == 200
                result = await response.json()
                
                # Verify search results structure
                assert "search_id" in result["data"]
                assert "results" in result["data"]
                assert result["data"]["results"]["total_papers"] > 0
                
                # Verify AI refinement worked
                assert len(result["data"]["refined_queries"]) > 0
                
                return result["data"]
    
    async def test_pdf_extraction_flow(self, setup_services):
        """Test PDF extraction workflow"""
        extraction_request = {
            "url": "https://arxiv.org/pdf/2301.00001.pdf",
            "config": {
                "extraction_methods": ["grobid", "pdfplumber"],
                "include_figures": True,
                "include_tables": True,
                "quality_threshold": 0.7
            }
        }
        
        async with aiohttp.ClientSession() as session:
            # Send extraction request
            async with session.post(
                "http://localhost:8989/api/v1/extract/b2-url",
                json=extraction_request,
                headers={"Authorization": "Bearer test_token"}
            ) as response:
                assert response.status == 200
                result = await response.json()
                
                # Verify extraction results
                assert "paper_id" in result["data"]
                assert "metadata" in result["data"]
                assert "content" in result["data"]
                assert "extraction_quality" in result["data"]
                
                # Verify content extraction
                content = result["data"]["content"]
                assert len(content["sections"]) > 0
                assert result["data"]["extraction_quality"]["overall_score"] > 0.5
                
                return result["data"]
    
    async def test_integrated_search_and_extraction(self, setup_services):
        """Test integrated search and extraction workflow"""
        # First, search for papers
        search_data = await self.test_paper_search_flow(setup_services)
        
        # Get a paper with PDF available
        paper_with_pdf = None
        for paper in search_data["results"]["papers"]:
            if paper["pdf_info"]["available"]:
                paper_with_pdf = paper
                break
                
        assert paper_with_pdf is not None, "No papers with PDFs found"
        
        # Request extraction for the paper
        extraction_request = {
            "url": paper_with_pdf["pdf_info"]["url"],
            "config": {"extraction_level": "standard"}
        }
        
        async with aiohttp.ClientSession() as session:
            async with session.post(
                "http://localhost:8989/api/v1/extract/b2-url",
                json=extraction_request,
                headers={"Authorization": "Bearer test_token"}
            ) as response:
                assert response.status == 200
                
        # Verify the complete workflow
        print(f"✅ Successfully integrated search and extraction for paper: {paper_with_pdf['title']}")

if __name__ == "__main__":
    pytest.main([__file__, "-v"])
```

### Load Testing

**File: `tests/load/ai-services-load-test.py`**

```python
import asyncio
import aiohttp
import time
import statistics
from concurrent.futures import ThreadPoolExecutor

async def load_test_search_service():
    """Load test the paper search service"""
    concurrent_requests = 10
    total_requests = 100
    
    search_queries = [
        "machine learning",
        "neural networks", 
        "deep learning",
        "computer vision",
        "natural language processing"
    ]
    
    async def make_request(session, query):
        start_time = time.time()
        try:
            async with session.post(
                "http://localhost:8085/search/papers",
                json={
                    "query": query,
                    "filters": {"max_results_per_source": 5}
                }
            ) as response:
                await response.json()
                return time.time() - start_time, response.status
        except Exception as e:
            return time.time() - start_time, 500
    
    # Execute load test
    response_times = []
    status_codes = []
    
    async with aiohttp.ClientSession() as session:
        for batch in range(total_requests // concurrent_requests):
            tasks = []
            for i in range(concurrent_requests):
                query = search_queries[i % len(search_queries)]
                tasks.append(make_request(session, query))
            
            batch_results = await asyncio.gather(*tasks)
            for response_time, status_code in batch_results:
                response_times.append(response_time)
                status_codes.append(status_code)
    
    # Calculate statistics
    success_rate = sum(1 for status in status_codes if status == 200) / len(status_codes)
    avg_response_time = statistics.mean(response_times)
    p95_response_time = statistics.quantiles(response_times, n=20)[18]  # 95th percentile
    
    print(f"Load Test Results:")
    print(f"Total Requests: {total_requests}")
    print(f"Success Rate: {success_rate:.2%}")
    print(f"Average Response Time: {avg_response_time:.2f}s")
    print(f"95th Percentile Response Time: {p95_response_time:.2f}s")
    
    assert success_rate > 0.95, f"Success rate {success_rate:.2%} below threshold"
    assert avg_response_time < 10.0, f"Average response time {avg_response_time:.2f}s too high"

if __name__ == "__main__":
    asyncio.run(load_test_search_service())
```

---

## 📊 Step 7: Monitoring and Observability

### Metrics Configuration

**File: `Docker/monitoring.yml`**

```yaml
services:
  prometheus:
    image: prom/prometheus:latest
    container_name: scholar-prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./monitoring/prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
    networks:
      - scholarai-network

  grafana:
    image: grafana/grafana:latest
    container_name: scholar-grafana
    ports:
      - "3001:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana-data:/var/lib/grafana
      - ./monitoring/grafana/dashboards:/etc/grafana/provisioning/dashboards
      - ./monitoring/grafana/datasources:/etc/grafana/provisioning/datasources
    networks:
      - scholarai-network

volumes:
  prometheus-data:
  grafana-data:

networks:
  scholarai-network:
    external: true
```

### Prometheus Configuration

**File: `Docker/monitoring/prometheus.yml`**

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'api-gateway'
    static_configs:
      - targets: ['api-gateway:8989']
    metrics_path: '/actuator/prometheus'
    
  - job_name: 'user-service'
    static_configs:
      - targets: ['user-service:8081']
    metrics_path: '/actuator/prometheus'
    
  - job_name: 'extractor-service'
    static_configs:
      - targets: ['extractor-service:8000']
    metrics_path: '/metrics'
    
  - job_name: 'paper-search-service'
    static_configs:
      - targets: ['paper-search-service:8000']
    metrics_path: '/metrics'
    
  - job_name: 'rabbitmq'
    static_configs:
      - targets: ['rabbitmq:15692']
    metrics_path: '/metrics'

rule_files:
  - "alert_rules.yml"

alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - alertmanager:9093
```

---

## 🚀 Step 8: Deployment Scripts

### Complete Deployment Script

**File: `Scripts/deploy-ai-services.sh`**

```bash
#!/bin/bash

###############################################################################
# ScholarAI AI Services Deployment Script
###############################################################################

set -e

# Color definitions
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'

# Configuration
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
ROOT_DIR="$(dirname "$SCRIPT_DIR")"
ENV_FILE="$ROOT_DIR/Docker/.env"

print_header() {
    echo -e "${BLUE}╔══════════════════════════════════════════════════════════════╗${NC}"
    echo -e "${BLUE}║                 ScholarAI AI Services Deployment            ║${NC}"
    echo -e "${BLUE}╚══════════════════════════════════════════════════════════════╝${NC}"
    echo
}

check_prerequisites() {
    echo -e "${YELLOW}Checking prerequisites...${NC}"
    
    # Check Docker
    if ! command -v docker &> /dev/null; then
        echo -e "${RED}❌ Docker is not installed${NC}"
        exit 1
    fi
    
    # Check Docker Compose
    if ! command -v docker-compose &> /dev/null; then
        echo -e "${RED}❌ Docker Compose is not installed${NC}"
        exit 1
    fi
    
    # Check environment file
    if [[ ! -f "$ENV_FILE" ]]; then
        echo -e "${RED}❌ Environment file not found: $ENV_FILE${NC}"
        exit 1
    fi
    
    # Check submodules
    if [[ ! -d "$ROOT_DIR/AI-Services/extractor" ]]; then
        echo -e "${RED}❌ Extractor service submodule not found${NC}"
        echo -e "${YELLOW}Run: git submodule update --init --recursive${NC}"
        exit 1
    fi
    
    if [[ ! -d "$ROOT_DIR/AI-Services/paper-search" ]]; then
        echo -e "${RED}❌ Paper search service submodule not found${NC}"
        echo -e "${YELLOW}Run: git submodule update --init --recursive${NC}"
        exit 1
    fi
    
    echo -e "${GREEN}✅ All prerequisites satisfied${NC}"
}

deploy_infrastructure() {
    echo -e "${YELLOW}Deploying infrastructure services...${NC}"
    
    cd "$ROOT_DIR"
    
    # Start infrastructure
    docker-compose -f Docker/services.yml up -d
    
    # Wait for services to be healthy
    echo -e "${YELLOW}Waiting for infrastructure services to be healthy...${NC}"
    sleep 30
    
    # Check RabbitMQ
    until docker exec scholar-rabbitmq rabbitmq-diagnostics -q ping; do
        echo -e "${YELLOW}Waiting for RabbitMQ...${NC}"
        sleep 5
    done
    
    echo -e "${GREEN}✅ Infrastructure services deployed${NC}"
}

deploy_microservices() {
    echo -e "${YELLOW}Deploying microservices...${NC}"
    
    cd "$ROOT_DIR"
    
    # Start microservices
    docker-compose -f Docker/apps.yml up -d
    
    # Wait for services
    echo -e "${YELLOW}Waiting for microservices to be healthy...${NC}"
    sleep 60
    
    echo -e "${GREEN}✅ Microservices deployed${NC}"
}

deploy_ai_services() {
    echo -e "${YELLOW}Deploying AI services...${NC}"
    
    cd "$ROOT_DIR"
    
    # Build and start AI services
    docker-compose -f Docker/ai-services.yml up -d --build
    
    # Wait for AI services
    echo -e "${YELLOW}Waiting for AI services to be healthy...${NC}"
    sleep 120
    
    # Check extractor service health
    until curl -f http://localhost:8084/health > /dev/null 2>&1; do
        echo -e "${YELLOW}Waiting for Extractor Service...${NC}"
        sleep 10
    done
    
    # Check paper search service health
    until curl -f http://localhost:8085/health > /dev/null 2>&1; do
        echo -e "${YELLOW}Waiting for Paper Search Service...${NC}"
        sleep 10
    done
    
    echo -e "${GREEN}✅ AI services deployed${NC}"
}

run_integration_tests() {
    echo -e "${YELLOW}Running integration tests...${NC}"
    
    cd "$ROOT_DIR"
    
    # Install test dependencies
    python -m pip install pytest aiohttp
    
    # Run integration tests
    python tests/integration/ai-services-integration-test.py
    
    echo -e "${GREEN}✅ Integration tests passed${NC}"
}

deploy_monitoring() {
    echo -e "${YELLOW}Deploying monitoring stack...${NC}"
    
    cd "$ROOT_DIR"
    
    # Start monitoring services
    docker-compose -f Docker/monitoring.yml up -d
    
    echo -e "${GREEN}✅ Monitoring stack deployed${NC}"
    echo -e "${BLUE}📊 Grafana: http://localhost:3001 (admin/admin)${NC}"
    echo -e "${BLUE}📊 Prometheus: http://localhost:9090${NC}"
}

show_service_status() {
    echo -e "\n${BLUE}╔══════════════════════════════════════════════════════════════╗${NC}"
    echo -e "${BLUE}║                    Service Status                           ║${NC}"
    echo -e "${BLUE}╚══════════════════════════════════════════════════════════════╝${NC}"
    
    services=(
        "Frontend:http://localhost:3000"
        "API Gateway:http://localhost:8989"
        "User Service:http://localhost:8081/actuator/health"
        "Project Service:http://localhost:8083/actuator/health"
        "Notification Service:http://localhost:8082/actuator/health"
        "Service Registry:http://localhost:8761"
        "Extractor Service:http://localhost:8084/health"
        "Paper Search Service:http://localhost:8085/health"
        "RabbitMQ Management:http://localhost:15672"
    )
    
    for service in "${services[@]}"; do
        name=$(echo "$service" | cut -d: -f1)
        url=$(echo "$service" | cut -d: -f2-)
        
        if curl -f "$url" > /dev/null 2>&1; then
            echo -e "${GREEN}✅ $name${NC} - $url"
        else
            echo -e "${RED}❌ $name${NC} - $url"
        fi
    done
}

# Main deployment flow
main() {
    print_header
    
    case "${1:-deploy}" in
        "deploy")
            check_prerequisites
            deploy_infrastructure
            deploy_microservices
            deploy_ai_services
            deploy_monitoring
            run_integration_tests
            show_service_status
            
            echo -e "\n${GREEN}🎉 ScholarAI AI Services deployment completed successfully!${NC}"
            echo -e "${BLUE}🚀 Platform available at: http://localhost:3000${NC}"
            ;;
            
        "status")
            show_service_status
            ;;
            
        "test")
            run_integration_tests
            ;;
            
        "logs")
            service_name="${2:-all}"
            if [[ "$service_name" == "all" ]]; then
                docker-compose -f Docker/ai-services.yml logs -f
            else
                docker-compose -f Docker/ai-services.yml logs -f "$service_name"
            fi
            ;;
            
        "stop")
            echo -e "${YELLOW}Stopping all services...${NC}"
            docker-compose -f Docker/ai-services.yml down
            docker-compose -f Docker/apps.yml down
            docker-compose -f Docker/services.yml down
            docker-compose -f Docker/monitoring.yml down
            echo -e "${GREEN}✅ All services stopped${NC}"
            ;;
            
        *)
            echo "Usage: $0 {deploy|status|test|logs [service]|stop}"
            exit 1
            ;;
    esac
}

main "$@"
```

---

## 📝 Step 9: Documentation Updates

Update your documentation to reflect the AI services integration:

### README.md Updates

Add AI services information to your main README:

```markdown
### 🤖 AI Services (Python/FastAPI)
1. **Extractor Service** (Port 8084) - PDF content extraction
2. **Paper Search Service** (Port 8085) - Academic search & collection

### Quick Start with AI Services

```bash
# Clone with AI services
git clone --recursive https://github.com/meta.git
cd meta

# Deploy complete platform including AI services
./Scripts/deploy-ai-services.sh

# Access services
# Frontend: http://localhost:3000
# Extractor API: http://localhost:8084/docs
# Search API: http://localhost:8085/docs
```

### API Documentation Updates

Add API endpoints for AI services to your documentation:

```markdown
| Service | Base URL | Documentation |
|:---|:---|:---|
| Extractor Service | `http://localhost:8084` | [Extractor API](docs/API/extractor-service.md) |
| Paper Search Service | `http://localhost:8085` | [Search API](docs/API/paper-search-service.md) |
```

---

## 🎯 Integration Success Metrics

### Deployment Validation

- ✅ All services start without errors
- ✅ Health checks pass for all components
- ✅ RabbitMQ queues configured correctly
- ✅ Database connections established
- ✅ B2 storage connectivity verified

### Functional Validation

- ✅ Paper search returns results from multiple sources
- ✅ PDF extraction completes successfully
- ✅ Queue messages processed correctly
- ✅ API Gateway routes requests properly
- ✅ Frontend integrates with AI services

### Performance Validation

- ✅ Search response time < 5 seconds
- ✅ Extraction completion < 2 minutes
- ✅ System handles 10+ concurrent requests
- ✅ Memory usage within acceptable limits
- ✅ No memory leaks detected

---

## 🔧 Troubleshooting Guide

### Common Issues and Solutions

**Issue: AI Services fail to start**
```bash
# Check logs
docker-compose -f Docker/ai-services.yml logs

# Common causes:
# 1. Missing environment variables
# 2. Database connection issues
# 3. Insufficient memory
# 4. Port conflicts
```

**Issue: RabbitMQ connection failures**
```bash
# Check RabbitMQ status
docker exec scholar-rabbitmq rabbitmq-diagnostics -q ping

# Reset queues if needed
docker exec scholar-rabbitmq rabbitmqctl reset
```

**Issue: PDF extraction fails**
```bash
# Check extractor service logs
docker logs scholar-extractor

# Common causes:
# 1. Invalid PDF URL
# 2. B2 storage configuration
# 3. Insufficient disk space
# 4. OCR model download issues
```

**Issue: Search service timeout**
```bash
# Check external API limits
# Increase timeout configuration
# Monitor resource usage
docker stats scholar-paper-search
```

---

## 🔄 Maintenance and Updates

### Regular Maintenance Tasks

1. **Monitor service health** and resource usage
2. **Update AI models** and dependencies
3. **Clean up temporary files** and old extractions
4. **Backup databases** and configuration
5. **Review and optimize** queue configurations

### Update Procedure

```bash
# Update submodules
git submodule update --remote --merge

# Rebuild and restart AI services
docker-compose -f Docker/ai-services.yml down
docker-compose -f Docker/ai-services.yml up -d --build

# Run integration tests
./Scripts/deploy-ai-services.sh test
```

---

**🎉 Congratulations! You have successfully integrated the AI services with your ScholarAI platform. The system now provides comprehensive academic paper search and extraction capabilities powered by advanced AI technologies.**
