I am trying to build an academic paper web search agent in fast api.
it will get an asynchronous request from spring boot.
we will use rabitmq
request will contain :
research domain: 
(optional) research topics:
(optional) description: 
(optional) date: any or a range:
Batch size (no of papers):

note: you can suggest a better request dto.
we have different clients and api like : arxiv, bioarxiv, core, sematic scholar, etc
now we have to think of a perfect agentic workflow here.
we will feed free LLM agent gemini these information like the request, what clients are available.
then gemini will provide optimized query for the web search for the paper and most importantly for that domain and topic or description which clients should we use. sometimes a client is more focused for a specific domain so using that client might not be a good idea. we need a decision like that.Think of some advance internal logic for the most relevant papers for that search parameters.
we need to retrieve paper metadata along side the actual paper.
{
  "paperId": "string", // Internal unique ID for the paper
  "title": "string",   // Full paper title
  "abstract": "string|null", // Paper abstract

  "authors": [ /* List of authors with rich metadata */ ],

  "publication": {
    "date": "2024-08-01", // Date of official publication or preprint upload
    "venueName": "NeurIPS", // Journal or conference name
    "publisher": "Springer Nature", // Publishing body
    "volume": "42", // Volume number (if applicable)
    "issue": "3", // Issue number
    "pages": "201-215", // Page range in the publication
    "publicationType": ["journal", "peer-reviewed"], // Tags for publication type
    "isOpenAccess": true, // True if publicly accessible without paywall
    "license": "CC-BY-4.0", // Reuse license
    "version": "published", // published | preprint | accepted
    "conferenceInstance": {
      "name": "NeurIPS",
      "location": "Vancouver, Canada",
      "startDate": "2024-12-02",
      "endDate": "2024-12-08"
    }
  },

  "identifiers": {
    "doi": "10.1145/12345678", // Digital Object Identifier
    "arxivId": "2406.01234", // arXiv identifier
    "pubmedId": "PMC1234567", // PubMed Central ID
    "semanticScholarId": "abcdef123", // Semantic Scholar internal ID
    "corpusId": 987654321 // S2 Corpus ID
  },

  "files": {
    "sourcePageUrl": "https://arxiv.org/abs/2406.01234", // Original paper info page
    "editorViewUrl": "https://scholarai.app/editor/abcdef123", // Internal viewer/editor
    "cloudPdfUrl": "https://storage.scholarai.app/papers/abcdef123.pdf", // PDF stored in our cloud
    "supplementaryUrls": [
      "https://host/code.zip",
      "https://host/dataset.tar.gz"
    ]
  },

  "citations": {
    "citationCount": 153, // Total citation count
    "influentialCitationCount": 19, // Highly influential citations
    "referenceCount": 42, // Number of references used in this paper
    "references": [ // (Optional) List of referenced papers
      {
        "title": "Prior Work on XYZ",
        "doi": "10.1145/111111",
        "authors": ["Alice A.", "Bob B."],
        "year": 2020
      }
    ],
    "citationsPerYear": {
      "2020": 5,
      "2021": 12,
      "2022": 35,
      "2023": 101
    }
  },

  "topics": {
    "fieldsOfStudy": ["Artificial Intelligence", "Natural Language Processing"], // S2-style domain tags
    "tags": ["transformers", "explainability", "zero-shot"], // Extracted topical keywords
    "methods": ["quantum circuits", "self-attention"], // Methods or models used
    "datasetsUsed": ["CIFAR-10", "SQuAD"] // Named datasets
  },

  "sourceInfo": {
    "source": "ArXiv", // Primary scraped source
    "sourceHost": "arxiv.org", // Domain name used for scraping
    "scrapedAt": "2025-07-03T21:00:00Z", // When scraped
    "lastUpdated": "2025-07-03T21:30:00Z", // When the data was last refreshed
    "language": "en" // Language of the paper
  }
}
