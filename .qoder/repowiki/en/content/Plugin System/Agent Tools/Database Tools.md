# Database Tools

<cite>
**Referenced Files in This Document**
- [agent/tools/exesql.py](file://agent/tools/exesql.py)
- [common/connection_utils.py](file://common/connection_utils.py)
- [rag/utils/infinity_conn.py](file://rag/utils/infinity_conn.py)
- [rag/utils/ob_conn.py](file://rag/utils/ob_conn.py)
- [api/db/db_utils.py](file://api/db/db_utils.py)
- [api/db/db_models.py](file://api/db/db_models.py)
- [agent/templates/sql_assistant.json](file://agent/templates/sql_assistant.json)
- [conf/mapping.json](file://conf/mapping.json)
- [conf/os_mapping.json](file://conf/os_mapping.json)
- [api/apps/canvas_app.py](file://api/apps/canvas_app.py)
</cite>

## Table of Contents
1. [Introduction](#introduction)
2. [System Architecture](#system-architecture)
3. [Supported Database Types](#supported-database-types)
4. [ExeSQL Tool Implementation](#exesql-tool-implementation)
5. [Connection Configuration](#connection-configuration)
6. [Security Considerations](#security-considerations)
7. [Performance Optimization](#performance-optimization)
8. [Vector Database Integration](#vector-database-integration)
9. [Best Practices](#best-practices)
10. [Troubleshooting](#troubleshooting)

## Introduction

The RAGFlow system provides comprehensive database interaction capabilities through its agent framework, enabling seamless integration with various database backends including traditional relational databases, modern vector databases, and specialized systems like OceanBase. The primary interface for database operations is the ExeSQL tool, which serves as a powerful bridge between natural language queries and structured database interactions.

This documentation covers the complete database tool ecosystem, focusing on the ExeSQL component that enables SQL query execution across multiple database types, security mechanisms for safe database access, and performance optimization strategies for efficient agent workflows.

## System Architecture

The database interaction system follows a modular architecture designed for extensibility and reliability:

```mermaid
graph TB
subgraph "Agent Framework"
Agent[Agent Component]
ExeSQL[ExeSQL Tool]
Params[Parameter Management]
end
subgraph "Database Layer"
ConnUtils[Connection Utils]
PoolMgr[Connection Pooling]
RetryMech[Retry Mechanism]
end
subgraph "Database Backends"
MySQL[MySQL/MariaDB]
PostgreSQL[PostgreSQL]
MSSQL[Microsoft SQL Server]
Trino[Trino]
DB2[IBM DB2]
Infinity[Infinity Vector DB]
OceanBase[OceanBase]
end
subgraph "Security Layer"
Auth[Authentication]
Injection[SQL Injection Prevention]
Validation[Input Validation]
end
Agent --> ExeSQL
ExeSQL --> Params
ExeSQL --> ConnUtils
ConnUtils --> PoolMgr
PoolMgr --> RetryMech
ExeSQL --> MySQL
ExeSQL --> PostgreSQL
ExeSQL --> MSSQL
ExeSQL --> Trino
ExeSQL --> DB2
ExeSQL --> Infinity
ExeSQL --> OceanBase
ExeSQL --> Auth
Auth --> Injection
Injection --> Validation
```

**Diagram sources**
- [agent/tools/exesql.py](file://agent/tools/exesql.py#L28-L120)
- [common/connection_utils.py](file://common/connection_utils.py#L31-L103)

## Supported Database Types

The system supports a comprehensive range of database backends, each with specific configuration requirements and capabilities:

### Relational Databases

| Database Type | Driver/Library | Connection Method | Authentication |
|---------------|----------------|-------------------|----------------|
| MySQL | pymysql | Standard connection | Username/password |
| MariaDB | pymysql | Standard connection | Username/password |
| PostgreSQL | psycopg2 | Standard connection | Username/password |
| Microsoft SQL Server | pyodbc | ODBC connection string | Username/password |
| IBM DB2 | ibm_db | Connection string | Username/password |

### Specialized Databases

| Database Type | Purpose | Configuration |
|---------------|---------|---------------|
| Trino | Distributed SQL Query Engine | Catalog/schema separation, TLS support |
| Infinity | Vector Database | Connection pooling, index management |
| OceanBase | High-performance OLTP | Vector embeddings, hybrid search |

### Vector Database Support

The system includes specialized support for vector databases with advanced indexing and similarity search capabilities:

```mermaid
classDiagram
class InfinityConnection {
+ConnectionPool connPool
+string dbName
+createIdx(indexName, knowledgebaseId, vectorSize)
+search(selectFields, condition, matchExprs)
+insert(documents)
+update(condition, newValue)
+delete(condition)
}
class VectorIndex {
+string vector_name
+int dimension
+string metric
+string encoder
+create_hnsw_index()
}
class FullTextIndex {
+string field_name
+string analyzer
+create_ft_index()
}
InfinityConnection --> VectorIndex : creates
InfinityConnection --> FullTextIndex : creates
```

**Diagram sources**
- [rag/utils/infinity_conn.py](file://rag/utils/infinity_conn.py#L174-L308)

**Section sources**
- [agent/tools/exesql.py](file://agent/tools/exesql.py#L120-L188)
- [rag/utils/infinity_conn.py](file://rag/utils/infinity_conn.py#L174-L308)

## ExeSQL Tool Implementation

The ExeSQL tool serves as the primary interface for executing SQL queries across different database backends. It implements a sophisticated parameter management system and result handling mechanism.

### Core Architecture

```mermaid
classDiagram
class ExeSQLParam {
+string db_type
+string database
+string username
+string host
+int port
+string password
+int max_records
+check() void
+get_input_form() dict
}
class ExeSQL {
+string component_name
+_invoke(**kwargs) string
+thoughts() string
-convert_decimals(obj) object
}
class ToolBase {
+check_if_canceled() bool
+get_input_elements_from_text() dict
+set_input_value(key, value) void
+string_format(template, args) string
+set_output(key, value) void
+output(key) object
}
ExeSQL --|> ToolBase
ExeSQL --> ExeSQLParam : uses
```

**Diagram sources**
- [agent/tools/exesql.py](file://agent/tools/exesql.py#L28-L77)
- [agent/tools/exesql.py](file://agent/tools/exesql.py#L79-L276)

### Parameter Management

The ExeSQL tool implements comprehensive parameter validation and management:

```mermaid
flowchart TD
Start([Parameter Validation]) --> CheckDBType{Valid DB Type?}
CheckDBType --> |No| ValidationError[Validation Error]
CheckDBType --> |Yes| CheckDatabase{Database Name Empty?}
CheckDatabase --> |Yes| ValidationError
CheckDatabase --> |No| CheckUsername{Username Empty?}
CheckUsername --> |Yes| ValidationError
CheckUsername --> |No| CheckHost{Host Empty?}
CheckHost --> |Yes| ValidationError
CheckHost --> |No| CheckPort{Port Positive?}
CheckPort --> |No| ValidationError
CheckPort --> |Yes| CheckPassword{Trino Password?}
CheckPassword --> |Yes & Trino| ValidationError
CheckPassword --> |No| CheckMaxRecords{Max Records Positive?}
CheckMaxRecords --> |No| ValidationError
CheckMaxRecords --> |Yes| SecurityCheck{Security Check?}
SecurityCheck --> |RagFlow DB| SecurityError[Security Error]
SecurityCheck --> |Normal| Success[Validation Success]
ValidationError --> End([Validation Failed])
SecurityError --> End
Success --> End([Validation Complete])
```

**Diagram sources**
- [agent/tools/exesql.py](file://agent/tools/exesql.py#L55-L69)

### Query Execution Pipeline

The query execution process follows a structured pipeline:

```mermaid
sequenceDiagram
participant Agent as Agent Component
participant ExeSQL as ExeSQL Tool
participant DB as Database Backend
participant Pool as Connection Pool
participant Result as Result Handler
Agent->>ExeSQL : Invoke with SQL query
ExeSQL->>ExeSQL : Parse and validate parameters
ExeSQL->>ExeSQL : Extract variables from SQL
ExeSQL->>ExeSQL : Format SQL with parameters
ExeSQL->>Pool : Get database connection
Pool->>DB : Establish connection
DB-->>Pool : Connection established
Pool-->>ExeSQL : Return connection
loop For each SQL statement
ExeSQL->>DB : Execute SQL
DB-->>ExeSQL : Return results
ExeSQL->>ExeSQL : Process results
ExeSQL->>Result : Format output
end
ExeSQL->>Pool : Release connection
ExeSQL-->>Agent : Return formatted results
```

**Diagram sources**
- [agent/tools/exesql.py](file://agent/tools/exesql.py#L82-L272)

**Section sources**
- [agent/tools/exesql.py](file://agent/tools/exesql.py#L28-L276)

## Connection Configuration

Database connections are configured through a flexible parameter system that supports various authentication methods and connection options.

### Basic Configuration Parameters

| Parameter | Type | Description | Default Value |
|-----------|------|-------------|---------------|
| db_type | string | Database backend type | mysql |
| database | string | Target database name | Required |
| username | string | Database username | Required |
| host | string | Database hostname/IP | Required |
| port | integer | Database port number | 3306 (MySQL) |
| password | string | Database password | Required (except Trino) |
| max_records | integer | Maximum records to return | 1024 |

### Advanced Configuration Options

The system supports several advanced configuration options for optimal performance and security:

#### Connection Pooling

```mermaid
graph LR
subgraph "Connection Pool Management"
Pool[Connection Pool]
Acquire[Acquire Connection]
Release[Release Connection]
Health[Health Check]
end
subgraph "Pool Configuration"
MaxSize[Max Size: 32]
Recycle[Recycle: 3600s]
Ping[Pre-Ping: True]
end
Pool --> Acquire
Acquire --> Release
Release --> Health
Health --> Pool
Pool -.-> MaxSize
Pool -.-> Recycle
Pool -.-> Ping
```

**Diagram sources**
- [rag/utils/infinity_conn.py](file://rag/utils/infinity_conn.py#L180-L195)

#### Timeout and Retry Configuration

The connection utilities implement sophisticated timeout and retry mechanisms:

```mermaid
flowchart TD
Request[Database Request] --> TimeoutCheck{Timeout Configured?}
TimeoutCheck --> |Yes| Timer[Start Timer]
TimeoutCheck --> |No| Execute[Execute Query]
Timer --> Execute
Execute --> Success{Success?}
Success --> |Yes| Return[Return Results]
Success --> |No| RetryCheck{Retry Attempts Left?}
RetryCheck --> |Yes| Delay[Exponential Backoff]
RetryCheck --> |No| Error[Return Error]
Delay --> Execute
Return --> End([Complete])
Error --> End
```

**Diagram sources**
- [common/connection_utils.py](file://common/connection_utils.py#L31-L103)

**Section sources**
- [agent/tools/exesql.py](file://agent/tools/exesql.py#L47-L54)
- [common/connection_utils.py](file://common/connection_utils.py#L31-L103)

## Security Considerations

The database tools implement multiple layers of security to protect against common vulnerabilities and ensure safe database access.

### SQL Injection Prevention

The system employs several strategies to prevent SQL injection attacks:

#### Parameterized Queries

```mermaid
flowchart TD
Input[User Input] --> Parse[Parse Variables]
Parse --> Validate[Validate Types]
Validate --> Sanitize[Sanitize Values]
Sanitize --> Format[Format SQL]
Format --> Execute[Execute Query]
subgraph "Validation Rules"
TypeCheck[Type Checking]
LengthLimit[Length Limits]
PatternMatch[Pattern Matching]
end
Validate --> TypeCheck
Validate --> LengthLimit
Validate --> PatternMatch
```

**Diagram sources**
- [agent/tools/exesql.py](file://agent/tools/exesql.py#L104-L114)

#### Credential Management

The system implements secure credential handling:

| Security Feature | Implementation | Purpose |
|------------------|----------------|---------|
| Environment Variables | Support for TRINO_USE_TLS, COMPONENT_EXEC_TIMEOUT | Secure credential storage |
| Connection Validation | Pre-execution connection testing | Prevent unauthorized access |
| Database Restrictions | Block rag_flow database access | Security isolation |
| Password Validation | Minimum length requirements | Strong authentication |

### Access Control

```mermaid
graph TB
subgraph "Access Control Layer"
Auth[Authentication]
Authz[Authorization]
Audit[Audit Logging]
end
subgraph "Security Checks"
CredCheck[Credential Validation]
DBRestrict[Database Restrictions]
IPWhitelist[IP Whitelisting]
end
subgraph "Runtime Protection"
Timeout[Execution Timeout]
RateLimit[Rate Limiting]
ResourceLimit[Resource Limits]
end
Auth --> CredCheck
Authz --> DBRestrict
Audit --> IPWhitelist
CredCheck --> Timeout
DBRestrict --> RateLimit
IPWhitelist --> ResourceLimit
```

**Diagram sources**
- [agent/tools/exesql.py](file://agent/tools/exesql.py#L65-L69)

**Section sources**
- [agent/tools/exesql.py](file://agent/tools/exesql.py#L55-L69)
- [common/connection_utils.py](file://common/connection_utils.py#L31-L103)

## Performance Optimization

The database tools implement comprehensive performance optimization strategies to ensure efficient operation in agent workflows.

### Query Optimization Strategies

#### Connection Management

```mermaid
sequenceDiagram
participant Agent as Agent
participant Pool as Connection Pool
participant DB as Database
Agent->>Pool : Request connection
Pool->>Pool : Check available connections
alt Connection available
Pool-->>Agent : Return pooled connection
else No connection available
Pool->>DB : Create new connection
DB-->>Pool : New connection ready
Pool-->>Agent : Return new connection
end
Agent->>DB : Execute query
DB-->>Agent : Return results
Agent->>Pool : Release connection
Pool->>Pool : Return to pool
```

**Diagram sources**
- [rag/utils/infinity_conn.py](file://rag/utils/infinity_conn.py#L180-L195)

#### Result Pagination

The system implements intelligent result pagination to handle large datasets efficiently:

| Parameter | Purpose | Default | Range |
|-----------|---------|---------|-------|
| max_records | Maximum rows to return | 1024 | 1-1000000 |
| page_size | Records per page | 100 | 1-1000 |
| offset | Starting position | 0 | 0+ |

#### Batch Processing

For bulk operations, the system supports efficient batch processing:

```mermaid
flowchart TD
Start[Start Bulk Operation] --> Split[Split Data into Batches]
Split --> Batch1[Batch 1: 1000 records]
Split --> Batch2[Batch 2: 1000 records]
Split --> BatchN[Batch N: Remaining]
Batch1 --> Transaction1[Begin Transaction]
Batch2 --> Transaction2[Begin Transaction]
BatchN --> TransactionN[Begin Transaction]
Transaction1 --> Insert1[Insert Batch 1]
Transaction2 --> Insert2[Insert Batch 2]
TransactionN --> InsertN[Insert Batch N]
Insert1 --> Commit1[Commit Transaction]
Insert2 --> Commit2[Commit Transaction]
InsertN --> CommitN[Commit Transaction]
Commit1 --> Success[Success]
Commit2 --> Success
CommitN --> Success
```

**Diagram sources**
- [api/db/db_utils.py](file://api/db/db_utils.py#L26-L51)

### Performance Monitoring

The system includes built-in performance monitoring and metrics collection:

#### Connection Health Monitoring

```mermaid
graph TB
subgraph "Health Monitoring"
Monitor[Health Monitor]
Metrics[Performance Metrics]
Alerts[Alert System]
end
subgraph "Metrics Collection"
Latency[Response Latency]
Throughput[Query Throughput]
ErrorRate[Error Rate]
ConnectionCount[Active Connections]
end
Monitor --> Latency
Monitor --> Throughput
Monitor --> ErrorRate
Monitor --> ConnectionCount
Latency --> Metrics
Throughput --> Metrics
ErrorRate --> Metrics
ConnectionCount --> Metrics
Metrics --> Alerts
```

**Diagram sources**
- [rag/utils/infinity_conn.py](file://rag/utils/infinity_conn.py#L246-L257)

**Section sources**
- [api/db/db_utils.py](file://api/db/db_utils.py#L26-L51)
- [rag/utils/infinity_conn.py](file://rag/utils/infinity_conn.py#L174-L257)

## Vector Database Integration

The system provides comprehensive support for vector databases, particularly focusing on the Infinity vector database for advanced similarity search and embedding operations.

### Infinity Vector Database Features

#### Index Management

```mermaid
classDiagram
class VectorIndex {
+string index_name
+string vector_column
+string distance_metric
+int M
+int ef_construction
+string encoder
+create_index()
+drop_index()
+optimize_index()
}
class FullTextIndex {
+string field_name
+string analyzer
+string language
+create_index()
+drop_index()
}
class HybridSearch {
+vector_search()
+text_search()
+fusion_search()
}
VectorIndex --> HybridSearch : supports
FullTextIndex --> HybridSearch : supports
```

**Diagram sources**
- [rag/utils/infinity_conn.py](file://rag/utils/infinity_conn.py#L280-L306)

#### Embedding Operations

The system supports various embedding sizes and operations:

| Vector Dimension | Use Case | Performance | Memory Usage |
|------------------|----------|-------------|--------------|
| 512 | General purpose | Fast | Low |
| 768 | Enhanced accuracy | Medium | Medium |
| 1024 | High precision | Slow | High |
| 1536 | Large models | Very slow | Very high |

#### Search Capabilities

```mermaid
flowchart TD
Query[Search Query] --> Type{Query Type}
Type --> |Text| TextSearch[Full-text Search]
Type --> |Vector| VectorSearch[Similarity Search]
Type --> |Hybrid| HybridSearch[Combined Search]
TextSearch --> Rank[Rank Results]
VectorSearch --> Rank
HybridSearch --> Fusion[Fusion Ranking]
Fusion --> Rank
Rank --> Filter[Apply Filters]
Filter --> Paginate[Apply Pagination]
Paginate --> Return[Return Results]
```

**Diagram sources**
- [rag/utils/infinity_conn.py](file://rag/utils/infinity_conn.py#L333-L506)

**Section sources**
- [rag/utils/infinity_conn.py](file://rag/utils/infinity_conn.py#L174-L506)
- [conf/mapping.json](file://conf/mapping.json#L159-L200)

## Best Practices

### Database Configuration

1. **Connection Pooling**: Always use connection pooling for production environments
2. **Timeout Configuration**: Set appropriate timeouts based on query complexity
3. **Resource Limits**: Configure maximum record limits to prevent memory issues
4. **Monitoring**: Implement comprehensive monitoring for connection health

### Security Best Practices

1. **Credential Management**: Store credentials securely using environment variables
2. **Network Security**: Use encrypted connections (TLS/SSL) for sensitive data
3. **Access Control**: Implement proper authentication and authorization
4. **Audit Logging**: Enable comprehensive audit logging for security monitoring

### Performance Best Practices

1. **Query Optimization**: Use parameterized queries and avoid dynamic SQL construction
2. **Result Limiting**: Always set reasonable limits on returned records
3. **Batch Processing**: Use batch operations for bulk data manipulation
4. **Indexing**: Properly index frequently queried columns

### Agent Workflow Integration

1. **Error Handling**: Implement robust error handling for database failures
2. **Retry Logic**: Use exponential backoff for transient failures
3. **Timeout Management**: Set appropriate timeouts for agent execution
4. **Resource Cleanup**: Ensure proper cleanup of database connections

## Troubleshooting

### Common Issues and Solutions

#### Connection Problems

| Issue | Symptoms | Solution |
|-------|----------|----------|
| Connection Timeout | Queries hang indefinitely | Increase timeout values, check network connectivity |
| Authentication Failure | Access denied errors | Verify credentials, check user permissions |
| SSL/TLS Issues | Certificate errors | Configure proper SSL settings, update certificates |
| Pool Exhaustion | Connection refused errors | Increase pool size, optimize connection usage |

#### Performance Issues

```mermaid
flowchart TD
PerfIssue[Performance Issue] --> Diagnose{Diagnose Problem}
Diagnose --> SlowQuery[Slow Queries]
Diagnose --> HighLatency[High Latency]
Diagnose --> MemoryIssue[Memory Issues]
SlowQuery --> Optimize[Optimize Queries]
HighLatency --> Network[Check Network]
MemoryIssue --> Resources[Increase Resources]
Optimize --> Index[Add Indexes]
Optimize --> Query[Refactor Queries]
Network --> Firewall[Check Firewall]
Network --> DNS[Check DNS Resolution]
Resources --> RAM[Increase RAM]
Resources --> CPU[Scale CPU]
```

#### Debugging Tools

1. **Connection Testing**: Use built-in connection testing utilities
2. **Query Logging**: Enable detailed query logging for debugging
3. **Performance Profiling**: Monitor query execution times
4. **Resource Monitoring**: Track memory and CPU usage

### Error Codes and Messages

| Error Code | Description | Resolution |
|------------|-------------|------------|
| 2013 | Lost connection | Check network, increase timeout |
| 2006 | MySQL server gone | Restart MySQL, check resources |
| 57P01 | Admin shutdown | Wait for server restart |
| 57P03 | Cannot connect now | Check server status, verify credentials |

**Section sources**
- [agent/tools/exesql.py](file://agent/tools/exesql.py#L136-L168)
- [api/db/db_models.py](file://api/db/db_models.py#L242-L308)