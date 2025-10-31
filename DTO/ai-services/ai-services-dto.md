# AI Services DTO Documentation

## Overview

This document defines the Data Transfer Objects (DTOs) used for communication between the AI services (Extractor and Paper Search) and the main ScholarAI platform microservices.

---

## Common DTOs

### ApiResponse<T>

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
@JsonInclude(JsonInclude.Include.NON_NULL)
public class ApiResponse<T> {
    
    @JsonFormat(pattern = "dd-MM-yyyy HH:mm:ss")
    private LocalDateTime timestamp;
    
    private int status;
    private String message;
    private T data;
    private ErrorDetails error;
    private String requestId;
    
    @Data
    @Builder
    public static class ErrorDetails {
        private String code;
        private String message;
        private List<String> details;
        private String suggestion;
        private boolean retryPossible;
    }
}
```

### PaginatedResponse<T>

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class PaginatedResponse<T> {
    
    private List<T> items;
    private long totalCount;
    private int page;
    private int pageSize;
    private int totalPages;
    private boolean hasNext;
    private boolean hasPrevious;
    
    private SearchMetadata metadata;
    
    @Data
    @Builder
    public static class SearchMetadata {
        private String query;
        private double searchTime;
        private List<String> sourcesUsed;
        private Map<String, Object> aggregations;
    }
}
```

---

## Extractor Service DTOs

### ExtractionRequestDto

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
@Valid
public class ExtractionRequestDto {
    
    @NotBlank(message = "PDF URL is required")
    @URL(message = "Invalid URL format")
    private String url;
    
    @Valid
    private ExtractionConfigDto config;
    
    private String paperId;
    
    @Pattern(regexp = "low|normal|high", message = "Priority must be low, normal, or high")
    private String priority = "normal";
    
    @URL(message = "Invalid callback URL format")
    private String callbackUrl;
    
    @Data
    @Builder
    @NoArgsConstructor
    @AllArgsConstructor
    public static class ExtractionConfigDto {
        
        @Builder.Default
        private List<String> extractionMethods = Arrays.asList("grobid", "pdfplumber");
        
        @Pattern(regexp = "fast|standard|comprehensive|custom")
        private String extractionLevel = "standard";
        
        @Builder.Default
        private boolean includeFigures = true;
        
        @Builder.Default
        private boolean includeTables = true;
        
        @Builder.Default
        private boolean includeEquations = true;
        
        @Builder.Default
        private boolean includeCodeBlocks = true;
        
        @DecimalMin(value = "0.0")
        @DecimalMax(value = "1.0")
        private double qualityThreshold = 0.7;
        
        @Builder.Default
        private boolean enableOcr = false;
        
        @Builder.Default
        private List<String> ocrLanguages = Arrays.asList("eng");
        
        @Builder.Default
        private boolean deduplication = true;
        
        @Builder.Default
        private boolean fallbackMethods = true;
    }
}
```

### ExtractionResultDto

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
@JsonInclude(JsonInclude.Include.NON_NULL)
public class ExtractionResultDto {
    
    private String paperId;
    private PaperMetadataDto metadata;
    private ExtractedContentDto content;
    private ExtractionQualityDto extractionQuality;
    private double processingTime;
    private List<String> extractionMethodsUsed;
    private Map<String, String> b2Urls;
    
    @JsonFormat(pattern = "dd-MM-yyyy HH:mm:ss")
    private LocalDateTime createdAt;
    
    private String version = "1.0";
    
    @Data
    @Builder
    @NoArgsConstructor
    @AllArgsConstructor
    public static class PaperMetadataDto {
        private String title;
        private List<AuthorDto> authors;
        private String abstractText;
        private String doi;
        private String arxivId;
        
        @JsonFormat(pattern = "dd-MM-yyyy")
        private LocalDate publicationDate;
        
        private String venue;
        private String venueType;
        private String language = "en";
        private int pageCount;
        private List<String> keywords;
        
        @Data
        @Builder
        @NoArgsConstructor
        @AllArgsConstructor
        public static class AuthorDto {
            private String name;
            private String affiliation;
            private String email;
            private String orcid;
        }
    }
    
    @Data
    @Builder
    @NoArgsConstructor
    @AllArgsConstructor
    public static class ExtractedContentDto {
        private List<SectionDto> sections;
        private List<FigureDto> figures;
        private List<TableDto> tables;
        private List<EquationDto> equations;
        private List<CodeBlockDto> codeBlocks;
        private List<ReferenceDto> references;
        private List<NamedEntityDto> namedEntities;
        
        @Data
        @Builder
        @NoArgsConstructor
        @AllArgsConstructor
        public static class SectionDto {
            private String id;
            private String title;
            private String content;
            private int level;
            private int pageStart;
            private int pageEnd;
            private List<SectionDto> subsections;
            private BoundingBoxDto bbox;
        }
        
        @Data
        @Builder
        @NoArgsConstructor
        @AllArgsConstructor
        public static class FigureDto {
            private String id;
            private String caption;
            private String url;
            private int page;
            private BoundingBoxDto bbox;
            private String fileType;
            private long fileSize;
            private int width;
            private int height;
            private String extractionMethod;
            private double confidenceScore;
        }
        
        @Data
        @Builder
        @NoArgsConstructor
        @AllArgsConstructor
        public static class TableDto {
            private String id;
            private String caption;
            private List<List<String>> data;
            private List<String> headers;
            private int page;
            private BoundingBoxDto bbox;
            private int rowCount;
            private int colCount;
            private boolean hasHeader;
            private String extractionMethod;
            private double confidenceScore;
            private String csvUrl;
        }
        
        @Data
        @Builder
        @NoArgsConstructor
        @AllArgsConstructor
        public static class EquationDto {
            private String id;
            private String latex;
            private int page;
            private BoundingBoxDto bbox;
            private String equationType;
            private double confidenceScore;
            private String imageUrl;
        }
        
        @Data
        @Builder
        @NoArgsConstructor
        @AllArgsConstructor
        public static class CodeBlockDto {
            private String id;
            private String content;
            private String language;
            private int page;
            private BoundingBoxDto bbox;
            private int lineCount;
            private double confidenceScore;
        }
        
        @Data
        @Builder
        @NoArgsConstructor
        @AllArgsConstructor
        public static class ReferenceDto {
            private String id;
            private String title;
            private List<String> authors;
            private String venue;
            private Integer year;
            private String doi;
            private String url;
            private int page;
            private String rawText;
        }
        
        @Data
        @Builder
        @NoArgsConstructor
        @AllArgsConstructor
        public static class NamedEntityDto {
            private String text;
            private String label;
            private int startChar;
            private int endChar;
            private double confidenceScore;
            private int page;
        }
    }
    
    @Data
    @Builder
    @NoArgsConstructor
    @AllArgsConstructor
    public static class ExtractionQualityDto {
        private double overallScore;
        private double textCoverage;
        private double figureDetection;
        private double tableExtraction;
        private double equationRecognition;
        private double referenceParsing;
        private double extractionCompleteness;
        private Map<String, Double> confidenceDistribution;
    }
    
    @Data
    @Builder
    @NoArgsConstructor
    @AllArgsConstructor
    public static class BoundingBoxDto {
        private double x1;
        private double y1;
        private double x2;
        private double y2;
        private int page;
    }
}
```

### ExtractionStatusDto

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class ExtractionStatusDto {
    
    private String taskId;
    
    @Pattern(regexp = "pending|processing|completed|failed")
    private String status;
    
    @DecimalMin("0.0")
    @DecimalMax("1.0")
    private double progress;
    
    @JsonFormat(pattern = "dd-MM-yyyy HH:mm:ss")
    private LocalDateTime estimatedCompletion;
    
    private String currentStage;
    private String errorMessage;
    private String resultUrl;
    
    @JsonFormat(pattern = "dd-MM-yyyy HH:mm:ss")
    private LocalDateTime createdAt;
    
    @JsonFormat(pattern = "dd-MM-yyyy HH:mm:ss")
    private LocalDateTime updatedAt;
}
```

---

## Paper Search Service DTOs

### PaperSearchRequestDto

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
@Valid
public class PaperSearchRequestDto {
    
    @NotBlank(message = "Search query is required")
    @Size(min = 2, max = 500, message = "Query must be between 2 and 500 characters")
    private String query;
    
    @Valid
    private SearchFiltersDto filters;
    
    @Valid
    private AIRefinementConfigDto aiRefinement;
    
    @Valid
    private PDFCollectionConfigDto pdfCollection;
    
    @Builder.Default
    private boolean includeCitations = false;
    
    @Builder.Default
    private boolean includeMetrics = true;
    
    @Builder.Default
    private boolean deduplicate = true;
    
    @Data
    @Builder
    @NoArgsConstructor
    @AllArgsConstructor
    public static class SearchFiltersDto {
        
        @Builder.Default
        private List<String> sources = Arrays.asList("arxiv", "semantic_scholar", "pubmed");
        
        private DateRangeDto dateRange;
        private List<String> publicationTypes;
        private List<String> fieldsOfStudy;
        private List<String> venues;
        private List<String> authors;
        
        @Min(0)
        private Integer minCitations;
        
        @Min(0)
        private Integer maxCitations;
        
        @Builder.Default
        private List<String> languages = Arrays.asList("en");
        
        private Boolean hasPdf;
        
        @Min(1)
        @Max(1000)
        @Builder.Default
        private int maxResultsPerSource = 100;
        
        @Data
        @Builder
        @NoArgsConstructor
        @AllArgsConstructor
        public static class DateRangeDto {
            
            @JsonFormat(pattern = "dd-MM-yyyy")
            private LocalDate start;
            
            @JsonFormat(pattern = "dd-MM-yyyy")
            private LocalDate end;
        }
    }
    
    @Data
    @Builder
    @NoArgsConstructor
    @AllArgsConstructor
    public static class AIRefinementConfigDto {
        
        @Builder.Default
        private boolean enabled = true;
        
        @Builder.Default
        private boolean expandQuery = true;
        
        @Builder.Default
        private boolean suggestRelatedTerms = true;
        
        @Builder.Default
        private boolean semanticExpansion = true;
        
        @Builder.Default
        private boolean fieldSpecificTerms = true;
    }
    
    @Data
    @Builder
    @NoArgsConstructor
    @AllArgsConstructor
    public static class PDFCollectionConfigDto {
        
        @Builder.Default
        private boolean enabled = true;
        
        @Pattern(regexp = "speed|coverage|balanced")
        @Builder.Default
        private String priority = "balanced";
        
        @Builder.Default
        private List<String> sources = Arrays.asList("direct", "publisher_api", "preprint_servers");
        
        @Min(1)
        @Max(100)
        @Builder.Default
        private int maxFileSize = 50; // MB
        
        @Min(5)
        @Max(300)
        @Builder.Default
        private int timeout = 30; // seconds
    }
}
```

### PaperSearchResponseDto

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class PaperSearchResponseDto {
    
    private String searchId;
    private String originalQuery;
    private List<String> refinedQueries;
    private SearchResultsDto results;
    private SearchAggregationsDto aggregations;
    private SearchSuggestionsDto suggestions;
    
    @Builder.Default
    private String processingStatus = "completed";
    
    @JsonFormat(pattern = "dd-MM-yyyy HH:mm:ss")
    private LocalDateTime createdAt;
    
    @Data
    @Builder
    @NoArgsConstructor
    @AllArgsConstructor
    public static class SearchResultsDto {
        private int totalPapers;
        private int uniquePapers;
        private List<String> sourcesSearched;
        private double searchTime;
        private List<PaperDto> papers;
    }
    
    @Data
    @Builder
    @NoArgsConstructor
    @AllArgsConstructor
    public static class PaperDto {
        private String id;
        private String title;
        private List<AuthorInfoDto> authors;
        private String abstractText;
        private Map<String, Object> metadata;
        
        @JsonFormat(pattern = "dd-MM-yyyy")
        private LocalDate publicationDate;
        
        private String venue;
        private String venueType;
        private List<String> fieldsOfStudy;
        private PaperMetricsDto metrics;
        private PDFInfoDto pdfInfo;
        private double relevanceScore;
        private double qualityScore;
        private String source;
        private String sourceUrl;
        
        @JsonFormat(pattern = "dd-MM-yyyy HH:mm:ss")
        private LocalDateTime discoveredAt;
        
        @Data
        @Builder
        @NoArgsConstructor
        @AllArgsConstructor
        public static class AuthorInfoDto {
            private String name;
            private String affiliation;
            private String orcid;
            private Integer hIndex;
            private Integer citationCount;
            private String url;
        }
        
        @Data
        @Builder
        @NoArgsConstructor
        @AllArgsConstructor
        public static class PaperMetricsDto {
            @Builder.Default
            private int citationCount = 0;
            
            @Builder.Default
            private double citationVelocity = 0.0;
            
            private Integer hIndex;
            private Double altmetricScore;
            private Double influenceScore;
            private Double fieldCitationRatio;
            private Double relativeCitationRatio;
            private Integer downloads;
            private Integer views;
        }
        
        @Data
        @Builder
        @NoArgsConstructor
        @AllArgsConstructor
        public static class PDFInfoDto {
            private boolean available;
            private String url;
            private String b2Url;
            private Double fileSize; // MB
            private Integer pages;
            
            @Builder.Default
            private String extractionStatus = "pending";
            
            private String extractionUrl;
        }
    }
    
    @Data
    @Builder
    @NoArgsConstructor
    @AllArgsConstructor
    public static class SearchAggregationsDto {
        private Map<String, Integer> byYear;
        private Map<String, Integer> byVenue;
        private Map<String, Integer> byField;
        private Map<String, Integer> bySource;
        private Map<String, Integer> byAuthor;
        private Map<String, Integer> byPublicationType;
    }
    
    @Data
    @Builder
    @NoArgsConstructor
    @AllArgsConstructor
    public static class SearchSuggestionsDto {
        private List<String> relatedQueries;
        private List<String> trendingTopics;
        private List<String> suggestedAuthors;
        private List<String> suggestedVenues;
        private List<String> queryRefinements;
    }
}
```

### CitationNetworkDto

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class CitationNetworkDto {
    
    private NetworkDto network;
    private InsightsDto insights;
    private List<ResearchClusterDto> clusters;
    private List<CitationPathDto> citationPaths;
    private Map<String, Object> temporalEvolution;
    private Map<String, Double> influencePropagation;
    
    @Data
    @Builder
    @NoArgsConstructor
    @AllArgsConstructor
    public static class NetworkDto {
        private List<CitationNodeDto> nodes;
        private List<CitationEdgeDto> edges;
        
        @Data
        @Builder
        @NoArgsConstructor
        @AllArgsConstructor
        public static class CitationNodeDto {
            private String id;
            private String title;
            private List<String> authors;
            private int year;
            private int citationCount;
            private String venue;
            private double centralityScore;
            private double influenceScore;
            private String nodeType; // "paper", "author", "venue"
        }
        
        @Data
        @Builder
        @NoArgsConstructor
        @AllArgsConstructor
        public static class CitationEdgeDto {
            private String source;
            private String target;
            private String relationship; // "cites", "cited_by", "co_cited"
            private double weight;
            private double confidence;
            
            @JsonFormat(pattern = "dd-MM-yyyy HH:mm:ss")
            private LocalDateTime createdAt;
        }
    }
    
    @Data
    @Builder
    @NoArgsConstructor
    @AllArgsConstructor
    public static class InsightsDto {
        private List<String> keyPapers;
        private List<ResearchClusterDto> researchClusters;
        private List<CitationPathDto> citationPaths;
    }
    
    @Data
    @Builder
    @NoArgsConstructor
    @AllArgsConstructor
    public static class ResearchClusterDto {
        private String id;
        private String topic;
        private List<String> keywords;
        private List<String> papers; // paper IDs
        private double influenceScore;
        private double growthRate;
        private List<String> keyAuthors;
        private List<String> representativeVenues;
    }
    
    @Data
    @Builder
    @NoArgsConstructor
    @AllArgsConstructor
    public static class CitationPathDto {
        private String fromPaper;
        private String toPaper;
        private List<String> path; // sequence of paper IDs
        private int pathLength;
        private double pathStrength;
        private String researchFlow; // "forward", "backward", "bidirectional"
    }
}
```

---

## Message Queue DTOs

### QueueMessageDto

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class QueueMessageDto {
    
    private MessageHeaderDto header;
    private Map<String, Object> payload;
    private CallbackInfoDto callback;
    
    @Data
    @Builder
    @NoArgsConstructor
    @AllArgsConstructor
    public static class MessageHeaderDto {
        private String messageId;
        private String correlationId;
        private String sourceService;
        private String destinationService;
        private String taskType;
        
        @Builder.Default
        private int priority = 5; // 1=lowest, 10=highest
        
        @Builder.Default
        private int retryCount = 0;
        
        @Builder.Default
        private int maxRetries = 3;
        
        @JsonFormat(pattern = "dd-MM-yyyy HH:mm:ss")
        private LocalDateTime timestamp;
        
        @JsonFormat(pattern = "dd-MM-yyyy HH:mm:ss")
        private LocalDateTime expiresAt;
    }
    
    @Data
    @Builder
    @NoArgsConstructor
    @AllArgsConstructor
    public static class CallbackInfoDto {
        private String service;
        private String endpoint;
        
        @Builder.Default
        private String method = "POST";
        
        private Map<String, String> headers;
    }
}
```

### ExtractionRequestMessageDto

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class ExtractionRequestMessageDto extends QueueMessageDto {
    
    // Payload structure for extraction requests
    @Data
    @Builder
    @NoArgsConstructor
    @AllArgsConstructor
    public static class ExtractionPayloadDto {
        private String paperId;
        private String pdfUrl;
        private Map<String, Object> extractionConfig;
        private String userId;
        private String projectId;
        private String priority;
    }
}
```

### SearchRequestMessageDto

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class SearchRequestMessageDto extends QueueMessageDto {
    
    // Payload structure for search requests
    @Data
    @Builder
    @NoArgsConstructor
    @AllArgsConstructor
    public static class SearchPayloadDto {
        private String userId;
        private String query;
        private Map<String, Object> filters;
        private Map<String, Object> aiRefinement;
        private Map<String, Object> pdfCollection;
        private String searchId;
    }
}
```

---

## Async Task DTOs

### AsyncTaskResponseDto

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class AsyncTaskResponseDto {
    
    private String taskId;
    
    @Pattern(regexp = "pending|processing|completed|failed")
    private String status;
    
    @DecimalMin("0.0")
    @DecimalMax("1.0")
    private double progress;
    
    @JsonFormat(pattern = "dd-MM-yyyy HH:mm:ss")
    private LocalDateTime estimatedCompletion;
    
    private String resultUrl;
    private String errorMessage;
    
    @JsonFormat(pattern = "dd-MM-yyyy HH:mm:ss")
    private LocalDateTime createdAt;
    
    @JsonFormat(pattern = "dd-MM-yyyy HH:mm:ss")
    private LocalDateTime updatedAt;
}
```

### BatchOperationDto

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class BatchOperationDto {
    
    private String batchId;
    private String operationType; // "extraction", "search", "analysis"
    private List<String> itemIds;
    private int totalItems;
    private int completedItems;
    private int failedItems;
    private double overallProgress;
    
    @JsonFormat(pattern = "dd-MM-yyyy HH:mm:ss")
    private LocalDateTime startedAt;
    
    @JsonFormat(pattern = "dd-MM-yyyy HH:mm:ss")
    private LocalDateTime estimatedCompletion;
    
    private List<BatchItemStatusDto> items;
    
    @Data
    @Builder
    @NoArgsConstructor
    @AllArgsConstructor
    public static class BatchItemStatusDto {
        private String itemId;
        private String status;
        private double progress;
        private String errorMessage;
        private String resultUrl;
    }
}
```

---

## Validation and Error DTOs

### ValidationErrorDto

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class ValidationErrorDto {
    
    private String field;
    private String rejectedValue;
    private String message;
    private String errorCode;
    
    @JsonFormat(pattern = "dd-MM-yyyy HH:mm:ss")
    private LocalDateTime timestamp;
}
```

### ServiceHealthDto

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class ServiceHealthDto {
    
    private String status; // "healthy", "degraded", "unhealthy"
    private Map<String, String> components;
    private String version;
    private long uptime; // seconds
    
    @JsonFormat(pattern = "dd-MM-yyyy HH:mm:ss")
    private LocalDateTime timestamp;
    
    private List<HealthCheckDto> checks;
    
    @Data
    @Builder
    @NoArgsConstructor
    @AllArgsConstructor
    public static class HealthCheckDto {
        private String name;
        private String status;
        private String message;
        private long responseTime; // milliseconds
    }
}
```

---

*These DTOs provide comprehensive data contracts for integration between the AI services and the main ScholarAI platform, ensuring type safety and clear communication protocols.*
