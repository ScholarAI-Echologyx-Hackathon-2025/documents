# Paper Search Agent DTOs (Async + RabbitMQ Enabled)

This document defines the data transfer objects (DTOs) used in the asynchronous paper search workflow. Web search requests are now published as messages to a RabbitMQ queue, and results are returned or stored via a job processing pipeline.

## 🔁 Async Architecture

- **Client** → sends a `WebSearchInputDTO` via `POST /api/search/web` (async)
- **Controller** → publishes a `WebSearchJobRequestDTO` to RabbitMQ
- **Consumer Worker** → consumes from queue, performs search, and returns a `WebSearchJobResultDTO`

## Overview

The paper search system uses the following DTOs:
- **WebSearchInputDTO**: Used for web search requests with domain, keywords, batch size, and date range
- **DateRangeDTO**: Used to specify date ranges for search queries
- **WebSearchJobRequestDTO**: RabbitMQ job request payload for async processing
- **WebSearchJobResultDTO**: Result DTO returned after processing the job
- **PaperResultDTO**: Represents a single paper result

## 📥 Request DTOs

## ✅ WebSearchInputDTO

Used for web search requests with comprehensive search parameters.

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class WebSearchInputDTO {

    @NotBlank(message = "Domain must not be blank")
    private String domain;

    @NotNull(message = "Keywords list must not be null")
    private List<@NotBlank(message = "Keyword must not be blank") String> keywords;

    @Min(value = 1, message = "Batch size must be at least 1")
    @Max(value = 100, message = "Batch size cannot exceed 100")
    private int batchSize;

    @Valid
    @NotNull(message = "Date range is required")
    private DateRangeDTO dateRange;
}
```

### Fields

| Field | Type | Required | Validation | Description |
|-------|------|----------|------------|-------------|
| `domain` | String | Yes | `@NotBlank` | The research domain or field to search within (e.g., "computer science", "machine learning", "quantum computing") |
| `keywords` | List<String> | Yes | `@NotNull`, each element `@NotBlank` | List of search keywords for finding relevant papers |
| `batchSize` | int | Yes | `@Min(1)`, `@Max(100)` | Number of results to retrieve per batch (1-100) |
| `dateRange` | DateRangeDTO | Yes | `@Valid`, `@NotNull` | Date range for filtering search results |

### Validation Rules

- **Domain**: Must not be blank and should specify a valid search domain
- **Keywords**: Must be a non-null list where each keyword is not blank
- **Batch Size**: Must be between 1 and 100 (inclusive)
- **Date Range**: Must be a valid DateRangeDTO object

### Common Research Domain Values

- `computer science`: Computer Science and Information Technology
- `machine learning`: Machine Learning and Artificial Intelligence
- `quantum computing`: Quantum Computing and Quantum Information
- `data science`: Data Science and Analytics
- `cybersecurity`: Cybersecurity and Information Security
- `software engineering`: Software Engineering and Development
- `bioinformatics`: Bioinformatics and Computational Biology
- `robotics`: Robotics and Automation

## ✅ DateRangeDTO

Used to specify date ranges for filtering search results.

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class DateRangeDTO {

    @NotNull(message = "Start date must be provided")
    @PastOrPresent(message = "Start date cannot be in the future")
    private java.time.LocalDate startDate;

    @NotNull(message = "End date must be provided")
    @PastOrPresent(message = "End date cannot be in the future")
    private LocalDate endDate;
}
```

### Fields

| Field | Type | Required | Validation | Description |
|-------|------|----------|------------|-------------|
| `startDate` | LocalDate | Yes | `@NotNull`, `@PastOrPresent` | Start date for the search range (inclusive) |
| `endDate` | LocalDate | Yes | `@NotNull`, `@PastOrPresent` | End date for the search range (inclusive) |

### Validation Rules

- **Start Date**: Must be provided and cannot be in the future
- **End Date**: Must be provided and cannot be in the future
- **Date Logic**: End date should typically be after or equal to start date (business logic validation)

## 📨 RabbitMQ Job DTOs

## 🟦 WebSearchJobRequestDTO

Payload to be published to RabbitMQ for async search jobs.

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class WebSearchJobRequestDTO {

    @NotNull
    private UUID jobId;

    @NotNull
    @Valid
    private WebSearchInputDTO searchInput;

    @NotNull
    private String callbackUrl; // Optional: For async result push
}
```

### Fields

| Field | Type | Required | Validation | Description |
|-------|------|----------|------------|-------------|
| `jobId` | UUID | Yes | `@NotNull` | Unique identifier for the search job |
| `searchInput` | WebSearchInputDTO | Yes | `@NotNull`, `@Valid` | The search parameters and criteria |
| `callbackUrl` | String | Yes | `@NotNull` | URL for pushing results asynchronously (optional) |

### Validation Rules

- **Job ID**: Must be a valid UUID and cannot be null
- **Search Input**: Must be a valid WebSearchInputDTO object
- **Callback URL**: Must be provided for async result delivery

## 🟩 WebSearchJobResultDTO

Result DTO returned after processing the job.

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class WebSearchJobResultDTO {

    @NotNull
    private UUID jobId;

    @NotNull
    private List<PaperResultDTO> results;

    private LocalDateTime processedAt;

    private String source; // "arxiv.org", "ieee.org", etc.

    private int totalFound;
}
```

### Fields

| Field | Type | Required | Validation | Description |
|-------|------|----------|------------|-------------|
| `jobId` | UUID | Yes | `@NotNull` | Unique identifier matching the original job request |
| `results` | List<PaperResultDTO> | Yes | `@NotNull` | List of found papers matching the search criteria |
| `processedAt` | LocalDateTime | No | - | Timestamp when the job was processed |
| `source` | String | No | - | Source platform where the search was performed (e.g., "arxiv.org", "ieee.org") |
| `totalFound` | int | No | - | Total number of papers found (may be more than returned) |

### Validation Rules

- **Job ID**: Must match the original job request ID
- **Results**: Must be a non-null list (can be empty if no results found)

## 📄 PaperResultDTO

Represents a single paper result with comprehensive details.

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class PaperResultDTO {

    private String title;

    private String abstractText;

    @Valid
    private List<AuthorDTO> authors;

    private String publicationDate;

    @Valid
    private PaperIdentifiersDTO identifiers;

    private String source;

    private String paperUrl;

    private String pdfUrl;

    private String downloadUrl;

    private boolean isOpenAccess;

    private String venueName;

    private String publisher;

    private List<String> publicationTypes;

    private Integer citationCount;

    private Integer referenceCount;
}
```

### Fields

| Field | Type | Required | Validation | Description |
|-------|------|----------|------------|-------------|
| `title` | String | No | - | Title of the research paper |
| `abstractText` | String | No | - | Abstract or summary of the paper |
| `authors` | List<AuthorDTO> | No | `@Valid` | List of authors with detailed information |
| `publicationDate` | String | No | - | Publication date of the paper (ISO format) |
| `identifiers` | PaperIdentifiersDTO | No | `@Valid` | Various identifiers for the paper |
| `source` | String | No | - | Source platform where paper was retrieved (e.g., "arxiv", "pubmed") |
| `paperUrl` | String | No | - | Link to the webpage that contains the paper |
| `pdfUrl` | String | No | - | Direct PDF link that opens in PDF viewer |
| `downloadUrl` | String | No | - | Our storage URL for direct download |
| `isOpenAccess` | boolean | No | - | Whether the paper is open access |
| `venueName` | String | No | - | Name of the venue/conference/journal |
| `publisher` | String | No | - | Publisher of the paper |
| `publicationTypes` | List<String> | No | - | Types of publication (e.g., "Journal", "Conference") |
| `citationCount` | Integer | No | - | Number of citations (if available) |
| `referenceCount` | Integer | No | - | Number of references in the paper |

## 👤 AuthorDTO

Represents detailed information about a paper author.

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class AuthorDTO {

    private String name;

    private String authorId;

    private String orcid;

    private String affiliation;

    private String email;

    private String researchInterests;

    private String hIndex;

    private String profileUrl;
}
```

### Fields

| Field | Type | Required | Validation | Description |
|-------|------|----------|------------|-------------|
| `name` | String | Yes | - | Full name of the author |
| `authorId` | String | No | - | Unique identifier for the author in the source system |
| `orcid` | String | No | - | ORCID identifier for the author |
| `affiliation` | String | No | - | Author's institutional affiliation |
| `email` | String | No | - | Author's email address (if publicly available) |
| `researchInterests` | String | No | - | Author's research interests and expertise |
| `hIndex` | String | No | - | Author's h-index (if available) |
| `profileUrl` | String | No | - | URL to author's profile page |

## 🆔 PaperIdentifiersDTO

Represents various identifiers for a paper that can be used as global unique identifiers.

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class PaperIdentifiersDTO {

    private String doi;

    private String arxivId;

    private String pubmedId;

    private String pubmedCentralId;

    private String corpusId;

    private String semanticScholarId;

    private String isbn;

    private String issn;
}
```

### Fields

| Field | Type | Required | Validation | Description |
|-------|------|----------|------------|-------------|
| `doi` | String | No | - | Digital Object Identifier |
| `arxivId` | String | No | - | arXiv identifier (e.g., "1706.03762") |
| `pubmedId` | String | No | - | PubMed identifier |
| `pubmedCentralId` | String | No | - | PubMed Central identifier |
| `corpusId` | String | No | - | Semantic Scholar Corpus ID |
| `semanticScholarId` | String | No | - | Semantic Scholar paper ID |
| `isbn` | String | No | - | International Standard Book Number |
| `issn` | String | No | - | International Standard Serial Number |

### Identifier Priority

The following order is recommended for using identifiers as global unique identifiers:
1. **DOI** - Most reliable and widely used
2. **arXiv ID** - For preprints and papers on arXiv
3. **PubMed ID** - For biomedical papers
4. **Corpus ID** - Semantic Scholar's internal identifier
5. **Semantic Scholar ID** - Semantic Scholar's paper identifier

## Request-Response Mapping

| Endpoint | Request DTO | Response DTO | Description |
|----------|-------------|--------------|-------------|
| `POST /api/search/web` | `WebSearchInputDTO` | `APIResponse<UUID>` | Initiate async search job, returns job ID |
| `GET /api/search/result/{jobId}` | - | `APIResponse<WebSearchJobResultDTO>` | Retrieve search results by job ID |

## 🛠 Suggested Flow

1. **POST /api/search/web**: accepts `WebSearchInputDTO`, generates jobId, publishes `WebSearchJobRequestDTO` to RabbitMQ
2. **Consumer Service**: listens to queue, performs search, builds `WebSearchJobResultDTO`
3. **Result Storage**: cache or DB by jobId, optionally push to callback
4. **GET /api/search/result/{jobId}**: retrieves result asynchronously

## Usage Examples

### 🧪 Example Job Message

**RabbitMQ JSON Payload:**
```json
{
  "jobId": "efed0cfa-189d-4c6d-b3e9-b122f7e239bd",
  "searchInput": {
    "domain": "quantum computing",
    "keywords": ["federated learning", "quantum optimization"],
    "batchSize": 30,
    "dateRange": {
      "startDate": "2022-01-01",
      "endDate": "2024-12-31"
    }
  },
  "callbackUrl": "https://example.com/api/search/result"
}
```

### Web Search Request
```json
{
  "domain": "machine learning",
  "keywords": [
    "deep learning",
    "neural networks",
    "transformer models"
  ],
  "batchSize": 20,
  "dateRange": {
    "startDate": "2023-01-01",
    "endDate": "2024-12-15"
  }
}
```

**Response** (202 Accepted):
```json
{
  "timestamp": "15-12-2024 14:30:25",
  "status": 202,
  "message": "Search job initiated successfully",
  "data": "efed0cfa-189d-4c6d-b3e9-b122f7e239bd"
}
```

**Result Retrieval** (GET /api/search/result/efed0cfa-189d-4c6d-b3e9-b122f7e239bd):
```json
{
  "timestamp": "15-12-2024 14:35:10",
  "status": 200,
  "message": "Search results retrieved successfully",
  "data": {
    "jobId": "efed0cfa-189d-4c6d-b3e9-b122f7e239bd",
    "results": [
      {
        "title": "Attention Is All You Need",
        "abstractText": "The dominant sequence transduction models are based on complex recurrent or convolutional neural networks...",
        "authors": [
          {
            "name": "Ashish Vaswani",
            "authorId": "vaswani_ashish",
            "orcid": "0000-0001-1234-5678",
            "affiliation": "Google Research",
            "email": "vaswani@google.com",
            "researchInterests": "Machine Learning, Natural Language Processing",
            "hIndex": "45",
            "profileUrl": "https://scholar.google.com/citations?user=vaswani_ashish"
          },
          {
            "name": "Noam Shazeer",
            "authorId": "shazeer_noam",
            "orcid": "0000-0002-2345-6789",
            "affiliation": "Google Research",
            "email": "shazeer@google.com",
            "researchInterests": "Machine Learning, Neural Networks",
            "hIndex": "38",
            "profileUrl": "https://scholar.google.com/citations?user=shazeer_noam"
          }
        ],
        "publicationDate": "2017-06-12",
        "identifiers": {
          "doi": "10.48550/arXiv.1706.03762",
          "arxivId": "1706.03762",
          "corpusId": "2153511463",
          "semanticScholarId": "2153511463"
        },
        "source": "arxiv",
        "paperUrl": "https://arxiv.org/abs/1706.03762",
        "pdfUrl": "https://arxiv.org/pdf/1706.03762.pdf",
        "downloadUrl": "https://storage.scholarai.com/papers/1706.03762.pdf",
        "isOpenAccess": true,
        "venueName": "Advances in Neural Information Processing Systems",
        "publisher": "Neural Information Processing Systems Foundation",
        "publicationTypes": ["Conference"],
        "citationCount": 45000,
        "referenceCount": 31
      }
    ],
    "processedAt": "2024-12-15T14:35:10",
    "source": "arxiv.org",
    "totalFound": 1
  }
}
```

### Date Range Examples

**Recent Papers (Last 6 months)**:
```json
{
  "startDate": "2024-06-15",
  "endDate": "2024-12-15"
}
```

**Specific Year Range**:
```json
{
  "startDate": "2020-01-01",
  "endDate": "2022-12-31"
}
```

**Current Year**:
```json
{
  "startDate": "2024-01-01",
  "endDate": "2024-12-15"
}
```

## Error Handling

### Validation Errors

**Invalid Domain** (400 Bad Request):
```json
{
  "timestamp": "2024-12-15T14:30:25",
  "status": 400,
  "code": "VALIDATION_ERROR",
  "message": "Invalid input data",
  "details": [
    "Domain must not be blank"
  ],
  "suggestion": "Please provide a valid research domain"
}
```

**Invalid Batch Size** (400 Bad Request):
```json
{
  "timestamp": "2024-12-15T14:30:25",
  "status": 400,
  "code": "VALIDATION_ERROR",
  "message": "Invalid input data",
  "details": [
    "Batch size cannot exceed 100"
  ],
  "suggestion": "Please provide a batch size between 1 and 100"
}
```

**Invalid Date Range** (400 Bad Request):
```json
{
  "timestamp": "2024-12-15T14:30:25",
  "status": 400,
  "code": "VALIDATION_ERROR",
  "message": "Invalid input data",
  "details": [
    "Start date cannot be in the future"
  ],
  "suggestion": "Please provide valid past or present dates"
}
```

**Job Not Found** (404 Not Found):
```json
{
  "timestamp": "2024-12-15T14:30:25",
  "status": 404,
  "code": "JOB_NOT_FOUND",
  "message": "Search job not found",
  "details": [
    "Job with ID efed0cfa-189d-4c6d-b3e9-b122f7e239bd does not exist"
  ],
  "suggestion": "Please check the job ID or wait for the job to complete"
}
```

**Job Still Processing** (202 Accepted):
```json
{
  "timestamp": "2024-12-15T14:30:25",
  "status": 202,
  "code": "JOB_PROCESSING",
  "message": "Search job is still being processed",
  "details": [
    "Job efed0cfa-189d-4c6d-b3e9-b122f7e239bd is currently being processed"
  ],
  "suggestion": "Please try again in a few moments"
}
```

## Best Practices

### For WebSearchInputDTO

1. **Domain Selection**:
   - Use specific research domains for targeted searches
   - Consider using multiple domains for comprehensive results
   - Validate domain relevance to your research area

2. **Keyword Strategy**:
   - Use specific, relevant keywords
   - Include synonyms and related terms
   - Avoid overly broad or narrow keyword sets

3. **Batch Size Optimization**:
   - Use smaller batch sizes (10-20) for faster initial results
   - Use larger batch sizes (50-100) for comprehensive searches
   - Consider pagination for large result sets

### For DateRangeDTO

1. **Date Range Selection**:
   - Use recent ranges for cutting-edge research
   - Use broader ranges for comprehensive literature reviews
   - Consider academic publication cycles

2. **Date Validation**:
   - Ensure end date is not before start date
   - Consider reasonable date ranges (not too broad or narrow)
   - Account for timezone differences if applicable

### For Async Workflow

1. **Job Management**:
   - Store job IDs for result retrieval
   - Implement proper job status tracking
   - Set appropriate job timeouts

2. **Error Handling**:
   - Handle RabbitMQ connection failures gracefully
   - Implement retry mechanisms for failed jobs
   - Provide clear error messages for job failures

3. **Performance Optimization**:
   - Use appropriate queue configurations
   - Implement job prioritization if needed
   - Monitor queue performance and scaling

## Dependencies

These DTOs use the following annotations from the Jakarta Validation API:
- `@NotBlank`: Ensures the field is not null, empty, or contains only whitespace
- `@NotNull`: Ensures the field is not null
- `@Min`: Validates minimum value for numeric fields
- `@Max`: Validates maximum value for numeric fields
- `@PastOrPresent`: Validates that the date is in the past or present
- `@Valid`: Triggers validation of nested objects
- `@Email`: Validates email format (for AuthorDTO.email)

And Lombok annotations for boilerplate code reduction:
- `@Data`: Generates getters, setters, toString, equals, and hashCode methods
- `@AllArgsConstructor`: Generates a constructor with all fields
- `@NoArgsConstructor`: Generates a no-argument constructor
- `@Builder`: Implements the Builder pattern for object creation

## Modular Design Benefits

The modular design provides several advantages:

1. **Separation of Concerns**: Each DTO has a specific responsibility
   - `PaperResultDTO`: Main paper information
   - `AuthorDTO`: Author-specific details
   - `PaperIdentifiersDTO`: Various identifiers for paper tracking

2. **Reusability**: Individual DTOs can be reused in other contexts
   - `AuthorDTO` can be used for author profiles
   - `PaperIdentifiersDTO` can be used for paper deduplication

3. **Extensibility**: Easy to add new fields without affecting other DTOs
   - New author fields can be added to `AuthorDTO`
   - New identifiers can be added to `PaperIdentifiersDTO`

4. **Validation**: Granular validation at each level
   - Author-specific validation in `AuthorDTO`
   - Identifier-specific validation in `PaperIdentifiersDTO`
