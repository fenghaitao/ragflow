# Agent Tools System Documentation

<cite>
**Referenced Files in This Document**
- [agent/tools/__init__.py](file://agent/tools/__init__.py)
- [agent/tools/base.py](file://agent/tools/base.py)
- [agent/tools/tavily.py](file://agent/tools/tavily.py)
- [agent/tools/exesql.py](file://agent/tools/exesql.py)
- [agent/tools/code_exec.py](file://agent/tools/code_exec.py)
- [agent/tools/crawler.py](file://agent/tools/crawler.py)
- [agent/tools/github.py](file://agent/tools/github.py)
- [agent/tools/google.py](file://agent/tools/google.py)
- [agent/tools/retrieval.py](file://agent/tools/retrieval.py)
- [agent/tools/wikipedia.py](file://agent/tools/wikipedia.py)
- [agent/tools/pubmed.py](file://agent/tools/pubmed.py)
- [agent/test/dsl_examples/exesql.json](file://agent/test/dsl_examples/exesql.json)
- [agent/test/dsl_examples/tavily_and_generate.json](file://agent/test/dsl_examples/tavily_and_generate.json)
- [sandbox/executor_manager/core/container.py](file://sandbox/executor_manager/core/container.py)
- [sandbox/executor_manager/services/execution.py](file://sandbox/executor_manager/services/execution.py)
- [sandbox/tests/sandbox_security_tests_full.py](file://sandbox/tests/sandbox_security_tests_full.py)
- [common/data_source/utils.py](file://common/data_source/utils.py)
</cite>

## Table of Contents
1. [Introduction](#introduction)
2. [System Architecture](#system-architecture)
3. [Base Class Architecture](#base-class-architecture)
4. [Core Tool Implementations](#core-tool-implementations)
5. [Security and Safety Mechanisms](#security-and-safety-mechanisms)
6. [Configuration and Authentication](#configuration-and-authentication)
7. [Tool Composition and Workflows](#tool-composition-and-workflows)
8. [Best Practices and Guidelines](#best-practices-and-guidelines)
9. [Extension Patterns](#extension-patterns)
10. [Troubleshooting and Debugging](#troubleshooting-and-debugging)

## Introduction

The Agent Tools System is a comprehensive framework that enables AI agents to interact with external services, databases, and execute code in secure, controlled environments. This system provides a unified interface for various tools including web search, database queries, code execution, and data retrieval, all built around a robust base architecture that ensures security, reliability, and extensibility.

The system supports multiple programming languages, implements sophisticated error handling and retry mechanisms, and provides seamless integration with third-party APIs and services. Each tool follows a standardized pattern while maintaining flexibility for specific use cases.

## System Architecture

The agent tools system follows a modular architecture with clear separation of concerns:

```mermaid
graph TB
subgraph "Agent Tools System"
Base[Base Classes]
Tools[Tool Implementations]
Config[Configuration Management]
Security[Security Layer]
end
subgraph "External Services"
WebAPIs[Web APIs]
Databases[Database Systems]
CodeSandbox[Code Execution Sandbox]
ThirdParty[Third-party Services]
end
subgraph "Agent Workflow"
Canvas[Canvas Engine]
Components[Component System]
DSL[DSL Processing]
end
Base --> Tools
Tools --> WebAPIs
Tools --> Databases
Tools --> CodeSandbox
Tools --> ThirdParty
Canvas --> Components
Components --> DSL
DSL --> Tools
Security --> Tools
Config --> Tools
```

**Diagram sources**
- [agent/tools/base.py](file://agent/tools/base.py#L1-L176)
- [agent/tools/__init__.py](file://agent/tools/__init__.py#L1-L48)

### Key Architectural Principles

1. **Modularity**: Each tool is implemented as a separate module with clear interfaces
2. **Extensibility**: New tools can be added following established patterns
3. **Security**: All external calls are sandboxed and monitored
4. **Reliability**: Built-in retry mechanisms and error handling
5. **Standardization**: Consistent API patterns across all tools

**Section sources**
- [agent/tools/base.py](file://agent/tools/base.py#L1-L176)
- [agent/tools/__init__.py](file://agent/tools/__init__.py#L1-L48)

## Base Class Architecture

The foundation of the agent tools system is built on two primary base classes that provide common functionality and enforce consistent patterns across all tool implementations.

### ToolParamBase Class

The `ToolParamBase` class serves as the parameter definition and validation layer for all tools:

```mermaid
classDiagram
class ToolParamBase {
+dict meta
+check() void
+get_input_form() dict
+get_meta() dict
-_init_inputs() void
-_init_attr_by_meta() void
}
class ToolMeta {
+string name
+string displayName
+string description
+string displayDescription
+dict parameters
}
class ToolParameter {
+string type
+string description
+string displayDescription
+list enum
+bool required
}
ToolParamBase --> ToolMeta : "defines"
ToolMeta --> ToolParameter : "contains"
```

**Diagram sources**
- [agent/tools/base.py](file://agent/tools/base.py#L29-L43)
- [agent/tools/base.py](file://agent/tools/base.py#L65-L111)

### ToolBase Class

The `ToolBase` class provides the execution framework and common functionality:

```mermaid
classDiagram
class ToolBase {
+Canvas _canvas
+string _id
+ComponentParamBase _param
+invoke(**kwargs) Any
+get_meta() dict
+thoughts() string
-_invoke(**kwargs) Any
-_retrieve_chunks() void
}
class ComponentBase {
+set_output(key, value) void
+output(key) Any
+check_if_canceled() bool
+get_input() dict
+string_format() string
}
ToolBase --|> ComponentBase
ToolBase --> ToolParamBase : "uses"
```

**Diagram sources**
- [agent/tools/base.py](file://agent/tools/base.py#L114-L176)

### Core Features of Base Classes

1. **Parameter Validation**: Automatic validation of input parameters
2. **Output Management**: Standardized output handling and formatting
3. **Error Handling**: Consistent error reporting and logging
4. **Cancellation Support**: Graceful task cancellation
5. **Reference Management**: Automatic reference tracking for knowledge bases

**Section sources**
- [agent/tools/base.py](file://agent/tools/base.py#L114-L176)

## Core Tool Implementations

The system includes numerous specialized tools, each designed for specific use cases while following the common base architecture.

### Web Search Tools

#### Tavily Search Tool

The Tavily tool provides optimized search capabilities for Large Language Models:

```mermaid
sequenceDiagram
participant Agent as Agent
participant Tavily as TavilyTool
participant API as Tavily API
participant Canvas as Canvas
Agent->>Tavily : invoke(query, topic, domains)
Tavily->>Tavily : validate_parameters()
Tavily->>API : search(query, options)
API-->>Tavily : results[]
Tavily->>Tavily : _retrieve_chunks(results)
Tavily->>Canvas : add_reference(chunks)
Tavily-->>Agent : formalized_content
```

**Diagram sources**
- [agent/tools/tavily.py](file://agent/tools/tavily.py#L101-L154)

Key features:
- Optimized for LLM consumption
- Supports news and general search topics
- Domain filtering and exclusion
- Advanced content extraction
- Built-in retry mechanisms

#### Google Search Tool

Provides comprehensive Google search functionality:

```mermaid
flowchart TD
Start([Google Search Initiated]) --> ValidateParams["Validate Parameters<br/>- API Key<br/>- Country/Language<br/>- Query"]
ValidateParams --> ConstructQuery["Construct Search Query<br/>- Add API parameters<br/>- Handle pagination"]
ConstructQuery --> CallAPI["Call SerpApi"]
CallAPI --> ParseResults["Parse Organic Results"]
ParseResults --> FormatOutput["Format for Canvas<br/>- Title extraction<br/>- URL linking<br/>- Content summarization"]
FormatOutput --> End([Return Formatted Content])
CallAPI --> |Error| RetryLogic["Retry Logic<br/>- Max retries<br/>- Backoff delay"]
RetryLogic --> CallAPI
```

**Diagram sources**
- [agent/tools/google.py](file://agent/tools/google.py#L116-L173)

### Database Tools

#### ExeSQL Tool

The ExeSQL tool provides secure database query execution:

```mermaid
flowchart TD
InputValidation["Input Validation<br/>- SQL syntax<br/>- Parameter binding<br/>- Security checks"] --> ParseDBType["Parse Database Type<br/>- MySQL/MariaDB<br/>- PostgreSQL<br/>- MSSQL<br/>- Trino<br/>- IBM DB2"]
ParseDBType --> EstablishConnection["Establish Connection<br/>- Credential validation<br/>- Connection pooling"]
EstablishConnection --> ProcessSQL["Process SQL Queries<br/>- Split multiple statements<br/>- Handle variables<br/>- Apply limits"]
ProcessSQL --> ExecuteQueries["Execute Queries<br/>- Execute each statement<br/>- Handle errors<br/>- Collect results"]
ExecuteQueries --> FormatResults["Format Results<br/>- Convert to DataFrame<br/>- Apply date formatting<br/>- Generate markdown"]
FormatResults --> OutputResults["Output Results<br/>- JSON format<br/>- Markdown format<br/>- Reference tracking"]
```

**Diagram sources**
- [agent/tools/exesql.py](file://agent/tools/exesql.py#L79-L276)

Security features:
- Database name and password validation
- Connection string sanitization
- Query result size limits
- Support for multiple database types

**Section sources**
- [agent/tools/tavily.py](file://agent/tools/tavily.py#L1-L252)
- [agent/tools/google.py](file://agent/tools/google.py#L1-L173)
- [agent/tools/exesql.py](file://agent/tools/exesql.py#L1-L276)

### Code Execution Tools

#### Code Exec Tool

The Code Exec tool provides secure code execution in isolated sandboxes:

```mermaid
sequenceDiagram
participant Agent as Agent
participant CodeExec as CodeExec Tool
participant Sandbox as Sandbox Service
participant Container as Container Runtime
Agent->>CodeExec : invoke(lang, script, args)
CodeExec->>CodeExec : encode_code(script)
CodeExec->>CodeExec : create_execution_request()
CodeExec->>Sandbox : POST /run (base64 code)
Sandbox->>Container : create_container()
Container->>Container : execute_code()
Container-->>Sandbox : stdout/stderr
Sandbox-->>CodeExec : execution_result
CodeExec->>CodeExec : deserialize_stdout()
CodeExec->>CodeExec : populate_outputs()
CodeExec-->>Agent : formatted_output
```

**Diagram sources**
- [agent/tools/code_exec.py](file://agent/tools/code_exec.py#L126-L192)

Security measures:
- Base64 encoding for code transmission
- Isolated container execution
- Resource limits (CPU, memory)
- Seccomp profile enforcement
- Timeout protection

**Section sources**
- [agent/tools/code_exec.py](file://agent/tools/code_exec.py#L1-L344)

### Information Retrieval Tools

#### Retrieval Tool

The Retrieval tool provides access to internal knowledge bases:

```mermaid
flowchart TD
QueryProcessing["Query Processing<br/>- Variable substitution<br/>- Text formatting<br/>- Metadata filtering"] --> KBSelection["Knowledge Base Selection<br/>- ID resolution<br/>- Tenant isolation<br/>- Access validation"]
KBSelection --> EmbeddingModel["Embedding Model<br/>- Vector search<br/>- Similarity calculation<br/>- Threshold filtering"]
EmbeddingModel --> Reranking["Reranking<br/>- Relevance scoring<br/>- Cross-language support<br/>- Knowledge graph enhancement"]
Reranking --> Formatting["Content Formatting<br/>- Chunk aggregation<br/>- Reference generation<br/>- Knowledge base prompt"]
Formatting --> OutputGeneration["Output Generation<br/>- Formalized content<br/>- JSON structure<br/>- Reference tracking"]
```

**Diagram sources**
- [agent/tools/retrieval.py](file://agent/tools/retrieval.py#L80-L245)

Advanced features:
- Multi-tenant support
- Cross-language search
- Knowledge graph integration
- Metadata filtering
- Table of contents enhancement

**Section sources**
- [agent/tools/retrieval.py](file://agent/tools/retrieval.py#L1-L251)

## Security and Safety Mechanisms

The agent tools system implements multiple layers of security to protect against malicious code execution and unauthorized access.

### Sandbox Architecture

The code execution sandbox provides isolation through containerization:

```mermaid
graph TB
subgraph "Host System"
subgraph "Sandbox Manager"
Executor[Executor Manager]
Container[Container Orchestration]
Security[Security Policies]
end
subgraph "Container Runtime"
subgraph "Python Sandbox"
PythonImage[Python Base Image]
PythonCode[Python Code Execution]
end
subgraph "Node.js Sandbox"
NodeImage[Node.js Base Image]
NodeCode[JavaScript Code Execution]
end
end
end
Executor --> Container
Container --> Security
Container --> PythonImage
Container --> NodeImage
PythonImage --> PythonCode
NodeImage --> NodeCode
```

**Diagram sources**
- [sandbox/executor_manager/core/container.py](file://sandbox/executor_manager/core/container.py#L102-L130)

### Security Features

1. **Container Isolation**: Each execution runs in a separate container
2. **Resource Limits**: CPU and memory constraints prevent resource exhaustion
3. **Seccomp Profiles**: System call filtering prevents dangerous operations
4. **Network Isolation**: No outbound network access by default
5. **File System Restrictions**: Read-only filesystem with temporary scratch space

### Code Security Validation

```mermaid
flowchart TD
CodeSubmission["Code Submission"] --> Base64Decode["Base64 Decode"]
Base64Decode --> SecurityAnalysis["Security Analysis<br/>- Pattern matching<br/>- Dangerous imports<br/>- System calls"]
SecurityAnalysis --> |Safe| ContainerLaunch["Launch Container"]
SecurityAnalysis --> |Unsafe| RejectExecution["Reject Execution<br/>- Return security error<br/>- Log violations"]
ContainerLaunch --> ExecuteCode["Execute Code"]
ExecuteCode --> MonitorExecution["Monitor Execution<br/>- Resource usage<br/>- Time limits<br/>- System calls"]
MonitorExecution --> CleanupResources["Cleanup Resources"]
```

**Diagram sources**
- [sandbox/executor_manager/services/execution.py](file://sandbox/executor_manager/services/execution.py#L236-L265)

**Section sources**
- [sandbox/executor_manager/core/container.py](file://sandbox/executor_manager/core/container.py#L102-L130)
- [sandbox/executor_manager/services/execution.py](file://sandbox/executor_manager/services/execution.py#L236-L265)
- [sandbox/tests/sandbox_security_tests_full.py](file://sandbox/tests/sandbox_security_tests_full.py#L381-L415)

## Configuration and Authentication

Each tool requires specific configuration parameters for authentication and operational settings.

### API Key Management

Most tools require API keys for external service access:

| Tool | Configuration Parameter | Purpose | Security Notes |
|------|------------------------|---------|----------------|
| Tavily | `api_key` | Search API authentication | Store securely, rotate regularly |
| Google | `api_key` | SerpApi authentication | Rate-limited, monitor usage |
| GitHub | None required | Public repository access | No authentication needed |
| PubMed | `email` | Entrez API identification | Must be valid email address |

### Database Configuration

ExeSQL tool requires database connection parameters:

| Parameter | Type | Description | Security Notes |
|-----------|------|-------------|----------------|
| `db_type` | string | Database type (mysql/postgres/mssql/etc.) | Validate against allowed types |
| `host` | string | Database server hostname | Use internal networks when possible |
| `port` | integer | Database server port | Default ports should be configurable |
| `database` | string | Target database name | Validate against allowed databases |
| `username` | string | Database username | Use dedicated service accounts |
| `password` | string | Database password | Encrypt in storage, never log |

### Environment Variables

Critical configuration is managed through environment variables:

- `COMPONENT_EXEC_TIMEOUT`: Default timeout for tool execution (seconds)
- `SANDBOX_HOST`: Host address for code execution sandbox
- `SANDBOX_MAX_MEMORY`: Memory limit for sandbox containers
- `SANDBOX_ENABLE_SECCOMP`: Enable seccomp security policies

**Section sources**
- [agent/tools/tavily.py](file://agent/tools/tavily.py#L78-L86)
- [agent/tools/google.py](file://agent/tools/google.py#L58-L61)
- [agent/tools/exesql.py](file://agent/tools/exesql.py#L47-L54)

## Tool Composition and Workflows

The agent tools system supports complex workflows through tool composition and chaining.

### Basic Tool Chain

```mermaid
sequenceDiagram
participant User as User Query
participant Agent as Agent
participant Tool1 as Search Tool
participant Tool2 as Analysis Tool
participant Tool3 as Output Tool
User->>Agent : Initial query
Agent->>Tool1 : Execute search
Tool1-->>Agent : Search results
Agent->>Tool2 : Process results
Tool2-->>Agent : Analysis results
Agent->>Tool3 : Format output
Tool3-->>Agent : Final response
Agent-->>User : Complete answer
```

### Multi-Tool Workflows

Complex scenarios often require multiple tools working together:

```mermaid
graph TD
Start([User Query]) --> Decision{Tool Needed?}
Decision --> |Web Search| WebSearch[Web Search Tool]
Decision --> |Database Query| Database[Database Tool]
Decision --> |Code Execution| CodeExec[Code Execution Tool]
Decision --> |Internal Retrieval| Retrieval[Retrieval Tool]
WebSearch --> Combine[Combine Results]
Database --> Combine
CodeExec --> Combine
Retrieval --> Combine
Combine --> Analysis[Analysis & Synthesis]
Analysis --> Output[Generate Response]
Output --> End([Final Answer])
```

### Template-Based Workflows

The system includes pre-built workflow templates:

1. **Deep Research**: Multi-stage research with multiple search tools
2. **SEO Blog Generation**: Content creation workflow with research and analysis
3. **Market Analysis**: Business intelligence workflow combining data sources
4. **Customer Support**: Multi-channel support with knowledge base integration

**Section sources**
- [agent/test/dsl_examples/tavily_and_generate.json](file://agent/test/dsl_examples/tavily_and_generate.json#L1-L55)
- [agent/test/dsl_examples/exesql.json](file://agent/test/dsl_examples/exesql.json#L1-L44)

## Best Practices and Guidelines

### Error Handling

Implement robust error handling patterns:

1. **Retry Logic**: Use exponential backoff for transient failures
2. **Graceful Degradation**: Provide fallback responses when tools fail
3. **Clear Error Messages**: Human-readable error descriptions
4. **Logging**: Comprehensive logging for debugging and monitoring

### Rate Limiting

Implement appropriate rate limiting:

```mermaid
flowchart TD
Request[Incoming Request] --> CheckRate["Check Rate Limits<br/>- Per-minute limits<br/>- Burst capacity<br/>- User quotas"]
CheckRate --> |Within Limits| ProcessRequest["Process Request"]
CheckRate --> |Over Limit| QueueRequest["Queue Request<br/>- Wait for slot<br/>- Return queued status"]
ProcessRequest --> UpdateMetrics["Update Metrics<br/>- Request count<br/>- Response times<br/>- Error rates"]
QueueRequest --> ProcessRequest
UpdateMetrics --> ReturnResponse["Return Response"]
```

### Performance Optimization

1. **Caching**: Cache frequently accessed data
2. **Connection Pooling**: Reuse database connections
3. **Async Operations**: Use asynchronous processing where possible
4. **Resource Management**: Proper cleanup of resources

### Security Guidelines

1. **Input Validation**: Sanitize all user inputs
2. **Access Control**: Implement proper authorization
3. **Audit Logging**: Log all tool executions
4. **Principle of Least Privilege**: Use minimal required permissions

**Section sources**
- [common/data_source/utils.py](file://common/data_source/utils.py#L113-L250)

## Extension Patterns

Creating new tools follows established patterns that ensure consistency and security.

### Tool Creation Template

```mermaid
classDiagram
class NewToolParam {
+dict meta
+check() void
+get_input_form() dict
+__init__()
}
class NewTool {
+string component_name
+_invoke(**kwargs) Any
+thoughts() string
}
class ToolBase {
<<abstract>>
+invoke(**kwargs) Any
+get_meta() dict
}
NewTool --|> ToolBase
NewTool --> NewToolParam : "uses"
```

### Implementation Steps

1. **Define Parameters**: Create parameter class inheriting from `ToolParamBase`
2. **Implement Tool**: Create tool class inheriting from `ToolBase`
3. **Add Validation**: Implement `check()` method for parameter validation
4. **Implement Invocation**: Override `_invoke()` method with tool logic
5. **Add Documentation**: Include comprehensive docstrings
6. **Add Tests**: Create unit tests for the new tool

### Integration with Existing Tools

New tools integrate seamlessly with existing infrastructure:

```mermaid
sequenceDiagram
participant Canvas as Canvas Engine
participant ToolRegistry as Tool Registry
participant NewTool as New Tool
participant ExternalAPI as External API
Canvas->>ToolRegistry : Register new tool
ToolRegistry->>ToolRegistry : Load tool dynamically
Canvas->>NewTool : invoke(parameters)
NewTool->>ExternalAPI : External call
ExternalAPI-->>NewTool : Response
NewTool-->>Canvas : Formatted result
```

**Section sources**
- [agent/tools/__init__.py](file://agent/tools/__init__.py#L25-L48)

## Troubleshooting and Debugging

### Common Issues and Solutions

#### Authentication Failures

**Problem**: API authentication errors
**Solution**: Verify API keys are correctly configured and not expired

#### Rate Limiting

**Problem**: Requests being throttled
**Solution**: Implement exponential backoff and reduce request frequency

#### Timeout Errors

**Problem**: Long-running operations timing out
**Solution**: Increase timeout values or optimize tool implementation

#### Sandbox Issues

**Problem**: Code execution failing in sandbox
**Solution**: Check container logs and verify security policies

### Debugging Tools

1. **Logging**: Enable debug logging for detailed execution traces
2. **Monitoring**: Use metrics to track tool performance
3. **Testing**: Run security tests to validate sandbox integrity
4. **Validation**: Use parameter validation to catch configuration errors early

### Monitoring and Observability

Key metrics to monitor:

- Tool execution times
- Success/failure rates
- Resource utilization
- Error patterns
- API usage statistics

**Section sources**
- [sandbox/tests/sandbox_security_tests_full.py](file://sandbox/tests/sandbox_security_tests_full.py#L381-L415)