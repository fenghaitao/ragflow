# Keyword Search

<cite>
**Referenced Files in This Document**
- [rag/nlp/query.py](file://rag/nlp/query.py)
- [rag/nlp/synonym.py](file://rag/nlp/synonym.py)
- [rag/nlp/rag_tokenizer.py](file://rag/nlp/rag_tokenizer.py)
- [rag/app/naive.py](file://rag/app/naive.py)
- [rag/utils/infinity_conn.py](file://rag/utils/infinity_conn.py)
- [rag/utils/es_conn.py](file://rag/utils/es_conn.py)
- [rag/utils/opensearch_conn.py](file://rag/utils/opensearch_conn.py)
- [rag/utils/doc_store_conn.py](file://rag/utils/doc_store_conn.py)
- [agent/tools/retrieval.py](file://agent/tools/retrieval.py)
- [web/src/components/metadata-filter/index.tsx](file://web/src/components/metadata-filter/index.tsx)
- [rag/prompts/meta_filter.md](file://rag/prompts/meta_filter.md)
- [rag/utils/ob_conn.py](file://rag/utils/ob_conn.py)
</cite>

## Table of Contents
1. [Introduction](#introduction)
2. [Architecture Overview](#architecture-overview)
3. [Core Components](#core-components)
4. [Exact and Fuzzy Matching Algorithms](#exact-and-fuzzy-matching-algorithms)
5. [Tokenization and Stemming](#tokenization-and-stemming)
6. [Synonym Expansion](#synonym-expansion)
7. [Naive Retrieval Application](#naive-retrieval-application)
8. [Configuration Parameters](#configuration-parameters)
9. [Domain Model for Keyword-Based Retrieval](#domain-model-for-keyword-based-retrieval)
10. [Metadata Filtering Enhancement](#metadata-filtering-enhancement)
11. [Performance Optimization](#performance-optimization)
12. [Precision vs Recall Trade-offs](#precision-vs-recall-trade-offs)
13. [When to Use Keyword Search](#when-to-use-keyword-search)
14. [Troubleshooting Guide](#troubleshooting-guide)
15. [Conclusion](#conclusion)

## Introduction

RAGFlow's keyword search recall method provides a robust foundation for retrieving relevant documents through precise and flexible matching mechanisms. The system implements sophisticated algorithms for exact and fuzzy keyword matching, incorporating advanced tokenization, stemming, and synonym expansion techniques. This comprehensive approach ensures optimal retrieval performance across diverse query types and dataset characteristics.

The keyword search system serves as a critical component in RAGFlow's multi-modal retrieval architecture, complementing vector-based similarity search with traditional text matching capabilities. It enables users to leverage both semantic understanding and precise keyword matching for comprehensive information retrieval.

## Architecture Overview

The keyword search architecture in RAGFlow follows a layered design that separates concerns between query processing, matching algorithms, and database interactions. The system integrates seamlessly with various document stores and retrieval engines.

```mermaid
graph TB
subgraph "Query Processing Layer"
QP[FulltextQueryer]
TK[Tokenizer]
SN[Synonym Expander]
end
subgraph "Matching Engine"
EM[Exact Matcher]
FM[Fuzzy Matcher]
PM[Proximity Matcher]
end
subgraph "Storage Layer"
ES[Elasticsearch]
OS[OpenSearch]
INF[Infinity]
OB[OceanBase]
end
subgraph "Application Layer"
NA[Naive Application]
RT[Retrieval Tool]
MF[Metadata Filter]
end
QP --> EM
QP --> FM
QP --> PM
TK --> QP
SN --> QP
EM --> ES
FM --> OS
PM --> INF
ES --> OB
RT --> QP
NA --> QP
MF --> ES
```

**Diagram sources**
- [rag/nlp/query.py](file://rag/nlp/query.py#L26-L40)
- [rag/utils/es_conn.py](file://rag/utils/es_conn.py#L190-L200)
- [rag/utils/infinity_conn.py](file://rag/utils/infinity_conn.py#L400-L430)

## Core Components

### FulltextQueryer Class

The `FulltextQueryer` serves as the central orchestrator for keyword search operations, implementing sophisticated query processing algorithms that handle both exact and fuzzy matching scenarios.

```mermaid
classDiagram
class FulltextQueryer {
+TermWeightDealer tw
+SynonymDealer syn
+list query_fields
+question(txt, tbl, min_match) MatchTextExpr
+paragraph(content_tks, keywords, keywords_topn) MatchTextExpr
+hybrid_similarity(avec, bvecs, atks, btkss, tkweight, vtweight) tuple
+token_similarity(atks, btkss) list
+similarity(qtwt, dtwt) float
+sub_special_char(line) string
+is_chinese(line) bool
+rmWWW(txt) string
+add_space_between_eng_zh(txt) string
}
class TermWeightDealer {
+weights(tokens, preprocess) list
+split(text) list
+pretoken(token, preprocess) list
+token_merge(tokens) list
}
class SynonymDealer {
+dictionary dict
+redis Redis
+lookup_num int
+load_tm float
+lookup(token, topn) list
+load() void
}
FulltextQueryer --> TermWeightDealer
FulltextQueryer --> SynonymDealer
```

**Diagram sources**
- [rag/nlp/query.py](file://rag/nlp/query.py#L26-L40)
- [rag/nlp/synonym.py](file://rag/nlp/synonym.py#L26-L48)

**Section sources**
- [rag/nlp/query.py](file://rag/nlp/query.py#L26-L280)
- [rag/nlp/synonym.py](file://rag/nlp/synonym.py#L26-L102)

### Tokenizer Integration

The system utilizes a specialized tokenizer that adapts its behavior based on the underlying document engine, providing consistent tokenization across different storage backends.

**Section sources**
- [rag/nlp/rag_tokenizer.py](file://rag/nlp/rag_tokenizer.py#L21-L43)

## Exact and Fuzzy Matching Algorithms

### Exact Matching Implementation

The exact matching algorithm focuses on precise term matching with configurable tolerance levels. The system employs sophisticated preprocessing techniques to normalize input text and optimize matching performance.

```mermaid
flowchart TD
Start([Input Query]) --> Preprocess[Text Preprocessing]
Preprocess --> Tokenize[Tokenization]
Tokenize --> WeightCalc[Weight Calculation]
WeightCalc --> SynLookup[Synonym Lookup]
SynLookup --> BuildQuery[Build Query Expression]
BuildQuery --> ApplyBoost[Apply Boost Factors]
ApplyBoost --> Optimize[Optimize Query]
Optimize --> Execute[Execute Search]
Execute --> End([Results])
Preprocess --> Normalize[Normalize Special Characters]
Preprocess --> SpaceAdd[Add Space Between Eng/Ch]
Preprocess --> WWWRemove[Remove WWW Patterns]
WeightCalc --> IDF[Calculate IDF Scores]
WeightCalc --> TF[Calculate TF Scores]
WeightCalc --> NER[Named Entity Recognition]
WeightCalc --> POS[Part-of-Speech Tagging]
```

**Diagram sources**
- [rag/nlp/query.py](file://rag/nlp/query.py#L85-L132)

### Fuzzy Matching with Proximity Scoring

The fuzzy matching system implements proximity-based scoring that considers word order and spatial relationships between terms. This approach enhances recall for queries with minor variations or typographical errors.

**Section sources**
- [rag/nlp/query.py](file://rag/nlp/query.py#L134-L218)

## Tokenization and Stemming

### Advanced Tokenization Pipeline

The tokenization process incorporates multiple stages to handle diverse linguistic patterns and improve matching accuracy across different languages and domains.

```mermaid
sequenceDiagram
participant Input as Query Input
participant Normalizer as Text Normalizer
participant Tokenizer as RagTokenizer
participant Stemmer as Fine-grained Tokenizer
participant Analyzer as Term Analyzer
Input->>Normalizer : Raw Query Text
Normalizer->>Normalizer : Remove WWW Patterns
Normalizer->>Normalizer : Add Space Between Eng/Ch
Normalizer->>Normalizer : Normalize Special Characters
Normalizer->>Tokenizer : Normalized Text
Tokenizer->>Tokenizer : Basic Tokenization
Tokenizer->>Stemmer : Tokenized Tokens
Stemmer->>Stemmer : Fine-grained Analysis
Stemmer->>Analyzer : Stemmed Tokens
Analyzer->>Analyzer : Calculate Frequencies
Analyzer->>Analyzer : Compute Weights
Analyzer->>Input : Weighted Token List
```

**Diagram sources**
- [rag/nlp/rag_tokenizer.py](file://rag/nlp/rag_tokenizer.py#L21-L43)
- [rag/nlp/query.py](file://rag/nlp/query.py#L141-L190)

### Stemming and Morphological Analysis

The stemming process applies morphological analysis to reduce words to their base forms while preserving semantic meaning and enabling broader matching coverage.

**Section sources**
- [rag/nlp/rag_tokenizer.py](file://rag/nlp/rag_tokenizer.py#L29-L33)

## Synonym Expansion

### Multi-Level Synonym Resolution

The synonym expansion system operates on multiple levels, combining custom dictionaries with external resources to provide comprehensive synonym coverage.

```mermaid
flowchart TD
Query[Input Token] --> DictCheck{Custom Dictionary?}
DictCheck --> |Found| ReturnDict[Return Dictionary Synonyms]
DictCheck --> |Not Found| WNCheck{WordNet Available?}
WNCheck --> |Yes| WNLookup[WordNet Synonym Lookup]
WNCheck --> |No| Empty[Empty Result]
WNLookup --> WNFilter[Filter and Rank Synonyms]
WNFilter --> ReturnWN[Return WordNet Synonyms]
ReturnDict --> Combine[Combine Results]
ReturnWN --> Combine
Empty --> Combine
Combine --> TopN[Limit to Top N]
TopN --> Final[Final Synonym List]
```

**Diagram sources**
- [rag/nlp/synonym.py](file://rag/nlp/synonym.py#L71-L96)

### Dynamic Synonym Loading

The system supports real-time synonym updates through Redis integration, enabling dynamic expansion of synonym dictionaries without system restarts.

**Section sources**
- [rag/nlp/synonym.py](file://rag/nlp/synonym.py#L49-L70)

## Naive Retrieval Application

### Architecture and Workflow

The naive retrieval application provides a streamlined interface for keyword-based document retrieval, integrating seamlessly with the broader RAGFlow ecosystem.

```mermaid
sequenceDiagram
participant Client as Client Application
participant Naive as Naive App
participant Queryer as FulltextQueryer
participant Store as Document Store
participant Filter as Metadata Filter
Client->>Naive : search(query, params)
Naive->>Naive : validate_parameters()
Naive->>Queryer : process_query(query)
Queryer->>Queryer : tokenize_and_weight()
Queryer->>Queryer : expand_synonyms()
Queryer->>Queryer : build_expression()
Queryer->>Naive : MatchTextExpr
Naive->>Filter : apply_metadata_filter()
Filter->>Filter : evaluate_conditions()
Filter->>Naive : filtered_results
Naive->>Store : execute_search(expr)
Store->>Naive : raw_results
Naive->>Naive : post_process_results()
Naive->>Client : formatted_response
```

**Diagram sources**
- [rag/app/naive.py](file://rag/app/naive.py#L605-L700)
- [agent/tools/retrieval.py](file://agent/tools/retrieval.py#L32-L68)

### Parameter Configuration

The naive retrieval application supports extensive configuration options for fine-tuning search behavior and performance characteristics.

**Section sources**
- [rag/app/naive.py](file://rag/app/naive.py#L605-L700)
- [agent/tools/retrieval.py](file://agent/tools/retrieval.py#L32-L68)

## Configuration Parameters

### Minimum Match Requirements

The system implements configurable minimum match thresholds that balance precision and recall based on query characteristics and user preferences.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `minimum_should_match` | float | 0.6 | Percentage of query terms that must match |
| `keywords_topn` | int | 30 | Maximum number of keywords to process |
| `synonym_topn` | int | 8 | Maximum synonyms per term |
| `boost_factor` | float | 1.0 | Overall query boost multiplier |

### Boost Factors and Scoring

The scoring system applies multiple boost factors to optimize retrieval quality for different term types and contexts.

| Boost Factor | Purpose | Typical Value |
|--------------|---------|---------------|
| Term Frequency | Rare terms receive higher boost | 1.0-5.0 |
| Synonym Expansion | Expanded terms receive reduced boost | 0.2 |
| Phrase Matching | Exact phrases receive higher boost | 2.0 |
| Field Weighting | Different fields have varying importance | 1.0-30.0 |

### Proximity Scoring Configuration

Proximity scoring considers word order and spatial relationships between query terms to improve relevance ranking.

**Section sources**
- [rag/nlp/query.py](file://rag/nlp/query.py#L113-L126)
- [rag/utils/infinity_conn.py](file://rag/utils/infinity_conn.py#L414-L418)

## Domain Model for Keyword-Based Retrieval

### Query Expression Model

The system defines a comprehensive query expression model that supports multiple matching strategies and scoring mechanisms.

```mermaid
classDiagram
class MatchTextExpr {
+list fields
+string matching_text
+int topn
+dict extra_options
+__init__(fields, matching_text, topn, extra_options)
}
class MatchDenseExpr {
+string vector_column_name
+VEC embedding_data
+string embedding_data_type
+string distance_type
+int topn
+dict extra_options
+__init__(vector_column_name, embedding_data, embedding_data_type, distance_type, topn, extra_options)
}
class FusionExpr {
+string method
+int topn
+dict fusion_params
+__init__(method, topn, fusion_params)
}
class OrderByExpr {
+list fields
+asc(field) OrderByExpr
+desc(field) OrderByExpr
}
MatchExpr <|-- MatchTextExpr
MatchExpr <|-- MatchDenseExpr
MatchExpr <|-- FusionExpr
```

**Diagram sources**
- [rag/utils/doc_store_conn.py](file://rag/utils/doc_store_conn.py#L59-L128)

### Field Weighting Schema

The domain model supports sophisticated field weighting that prioritizes different document components based on their relevance to the query.

**Section sources**
- [rag/utils/doc_store_conn.py](file://rag/utils/doc_store_conn.py#L59-L128)

## Metadata Filtering Enhancement

### Advanced Filtering Capabilities

The metadata filtering system provides powerful capabilities for refining search results based on document attributes and contextual information.

```mermaid
flowchart TD
Query[User Query] --> MetaGen[Metadata Filter Generator]
MetaGen --> AttrExtract[Extract Filterable Attributes]
AttrExtract --> CondBuild[Build Filter Conditions]
CondBuild --> LogicSelect[Select Logical Operators]
LogicSelect --> ExecPlan[Execution Plan]
ExecPlan --> Store[Document Store]
Store --> Filtered[Filtered Results]
AttrExtract --> DateInfer[Date Range Inference]
AttrExtract --> ValueMatch[Value Matching]
AttrExtract --> Negation[Negation Handling]
LogicSelect --> AndLogic[AND Logic]
LogicSelect --> OrLogic[OR Logic]
AndLogic --> Combined[Combined Filters]
OrLogic --> Combined
Combined --> ExecPlan
```

**Diagram sources**
- [rag/prompts/meta_filter.md](file://rag/prompts/meta_filter.md#L1-L67)
- [web/src/components/metadata-filter/index.tsx](file://web/src/components/metadata-filter/index.tsx#L1-L50)

### Operator Support Matrix

The system supports a comprehensive set of operators for precise metadata filtering.

| Operator Category | Operators | Description |
|------------------|-----------|-------------|
| Equality | `=`, `≠` | Exact value matching |
| Containment | `contains`, `not contains` | Substring matching |
| Ordering | `>`, `<`, `≥`, `≤` | Numeric/date comparisons |
| Pattern | `start with`, `end with` | String pattern matching |
| Existence | `empty`, `not empty` | Presence/absence checking |
| Enumeration | `in`, `not in` | Value set membership |

**Section sources**
- [rag/prompts/meta_filter.md](file://rag/prompts/meta_filter.md#L20-L35)
- [web/src/components/metadata-filter/index.tsx](file://web/src/components/metadata-filter/index.tsx#L15-L30)

## Performance Optimization

### Large Dataset Handling

The keyword search system implements several optimization strategies for handling large-scale datasets efficiently.

```mermaid
flowchart TD
LargeDataset[Large Dataset] --> Partition[Data Partitioning]
Partition --> Index[Pre-built Indices]
Index --> Cache[Query Result Caching]
Cache --> Parallel[Parallel Processing]
Parallel --> Batch[Batch Operations]
Batch --> Memory[Memory Management]
Memory --> Optimize[Performance Optimization]
Partition --> Sharding[Shard Distribution]
Index --> FieldIndex[Field-specific Indexing]
Cache --> TTL[TTL Management]
Parallel --> ThreadPool[Thread Pool Management]
Batch --> Buffer[Buffer Management]
Memory --> GC[Garbage Collection]
```

### Special Character Handling

The system implements comprehensive special character handling to ensure reliable matching across diverse text formats.

**Section sources**
- [rag/nlp/query.py](file://rag/nlp/query.py#L40-L43)
- [rag/utils/es_conn.py](file://rag/utils/es_conn.py#L194-L197)

## Precision vs Recall Trade-offs

### Balancing Strategies

The keyword search system provides multiple mechanisms for balancing precision and recall based on specific use case requirements.

```mermaid
graph LR
subgraph "Precision Focus"
P1[Strict Term Matching]
P2[High Boost Factors]
P3[Minimal Synonyms]
P4[Exact Phrases]
end
subgraph "Recall Focus"
R1[Fuzzy Matching]
R2[Extensive Synonyms]
R3[Proximity Scoring]
R4[Broad Term Variants]
end
subgraph "Balanced Approach"
B1[Configurable Thresholds]
B2[Adaptive Scoring]
B3[Hybrid Methods]
B4[User Feedback]
end
P1 --> B1
R1 --> B1
P2 --> B2
R2 --> B2
P3 --> B3
R3 --> B3
P4 --> B4
R4 --> B4
```

### Adaptive Threshold Configuration

The system supports adaptive threshold configuration that adjusts based on query characteristics and historical performance data.

**Section sources**
- [rag/nlp/query.py](file://rag/nlp/query.py#L278-L279)

## When to Use Keyword Search

### Query Characteristic Analysis

Different query types benefit from different search approaches, and the system provides guidance for optimal selection.

| Query Type | Recommended Method | Rationale |
|------------|-------------------|-----------|
| Exact Terms | Keyword Search | Precise matching required |
| Technical Terms | Hybrid Approach | Balance precision and recall |
| Natural Language | Vector + Keyword | Semantic understanding needed |
| Short Queries | Keyword Heavy | Limited context for vectors |
| Long Queries | Balanced Approach | Mixed precision/recall needs |

### Decision Framework

```mermaid
flowchart TD
Query[User Query] --> Length{Query Length}
Length --> |Short (< 5 words)| KeywordOnly[Keyword Search Only]
Length --> |Medium (5-15 words)| Hybrid[Hybrid Approach]
Length --> |Long (> 15 words)| VectorPlus[Vector + Keyword]
KeywordOnly --> ExactMatch[Focus on Exact Terms]
Hybrid --> Balanced[Balance Precision/Recall]
VectorPlus --> Semantic[Emphasize Semantics]
ExactMatch --> Results1[High Precision Results]
Balanced --> Results2[Optimal Results]
Semantic --> Results3[Comprehensive Results]
```

**Section sources**
- [agent/tools/retrieval.py](file://agent/tools/retrieval.py#L54-L56)

## Troubleshooting Guide

### Common Issues and Solutions

#### Special Character Problems

**Issue**: Special characters causing search failures or unexpected results
**Solution**: The system automatically escapes special characters using the `sub_special_char` method, which handles common Elasticsearch/OpenSearch reserved characters.

**Section sources**
- [rag/nlp/query.py](file://rag/nlp/query.py#L40-L43)

#### Performance Degradation

**Issue**: Slow query performance with large datasets
**Solution**: Implement proper indexing strategies, consider query result caching, and optimize field weights for frequently searched content.

#### Synonym Expansion Issues

**Issue**: Missing or incorrect synonym expansions
**Solution**: Verify custom synonym dictionary configuration and ensure WordNet resources are properly installed for English language support.

**Section sources**
- [rag/nlp/synonym.py](file://rag/nlp/synonym.py#L71-L96)

#### Metadata Filter Conflicts

**Issue**: Metadata filters not applying correctly
**Solution**: Check filter syntax and ensure metadata field names match exactly with indexed field names.

**Section sources**
- [rag/utils/ob_conn.py](file://rag/utils/ob_conn.py#L238-L264)

## Conclusion

RAGFlow's keyword search recall method provides a comprehensive and flexible foundation for document retrieval that balances precision and recall across diverse use cases. The system's sophisticated algorithms for exact and fuzzy matching, combined with advanced tokenization, stemming, and synonym expansion capabilities, enable highly effective information retrieval.

The modular architecture supports seamless integration with various document stores while maintaining consistent performance characteristics across different deployment environments. The extensive configuration options allow fine-tuning for specific domain requirements, ensuring optimal results for both development and production scenarios.

Key strengths of the system include its ability to handle complex queries with multiple matching strategies, robust metadata filtering capabilities, and efficient performance optimization for large-scale deployments. The combination of traditional keyword matching with modern semantic understanding creates a powerful retrieval system that serves a wide range of applications and user needs.

Future enhancements may include machine learning-based query understanding, improved multilingual support, and advanced ranking algorithms that incorporate user feedback and contextual information for even better retrieval performance.