# MCP Server

<cite>
**Referenced Files in This Document**
- [mcp/server/server.py](file://mcp/server/server.py)
- [mcp/client/client.py](file://mcp/client/client.py)
- [api/apps/mcp_server_app.py](file://api/apps/mcp_server_app.py)
- [api/db/services/mcp_server_service.py](file://api/db/services/mcp_server_service.py)
- [api/db/db_models.py](file://api/db/db_models.py)
- [common/mcp_tool_call_conn.py](file://common/mcp_tool_call_conn.py)
- [common/constants.py](file://common/constants.py)
- [agent/component/agent_with_tools.py](file://agent/component/agent_with_tools.py)
- [web/src/interfaces/database/mcp.ts](file://web/src/interfaces/database/mcp.ts)
- [web/src/interfaces/database/mcp-server.ts](file://web/src/interfaces/database/mcp-server.ts)
- [web/src/services/mcp-server-service.ts](file://web/src/services/mcp-server-service.ts)
</cite>

## Table of Contents
1. [Introduction](#introduction)
2. [Architecture Overview](#architecture-overview)
3. [Core Components](#core-components)
4. [Server Implementation](#server-implementation)
5. [Client Integration](#client-integration)
6. [Agent System Integration](#agent-system-integration)
7. [Configuration and Deployment](#configuration-and-deployment)
8. [Security Considerations](#security-considerations)
9. [Performance and Best Practices](#performance-and-best-practices)
10. [Troubleshooting](#troubleshooting)
11. [Conclusion](#conclusion)

## Introduction

The Model Context Protocol (MCP) server implementation in RAGFlow provides a standardized way to expose tools and services through a protocol that enables function calling with Large Language Models (LLMs). This system allows LLMs to discover, invoke, and interact with external tools and services seamlessly, enhancing their capabilities through external integrations.

The MCP server acts as a bridge between the RAGFlow knowledge base system and LLMs, enabling sophisticated agent workflows where tools can be dynamically discovered, configured, and invoked based on user requests and context.

## Architecture Overview

The MCP system follows a layered architecture that separates concerns between tool discovery, invocation, and management:

```mermaid
graph TB
subgraph "Client Layer"
LLM[Large Language Model]
Agent[Agent System]
end
subgraph "MCP Server Layer"
Server[MCP Server]
Router[Request Router]
Auth[Authentication]
end
subgraph "Tool Management Layer"
ToolRegistry[Tool Registry]
ToolDiscovery[Tool Discovery]
ToolInvocation[Tool Invocation]
end
subgraph "Backend Services"
RAGFlow[RAGFlow Backend]
KB[Knowledge Bases]
Tools[External Tools]
end
LLM --> Server
Agent --> Server
Server --> Router
Router --> Auth
Router --> ToolRegistry
ToolRegistry --> ToolDiscovery
ToolRegistry --> ToolInvocation
ToolInvocation --> RAGFlow
RAGFlow --> KB
RAGFlow --> Tools
```

**Diagram sources**
- [mcp/server/server.py](file://mcp/server/server.py#L326-L327)
- [agent/component/agent_with_tools.py](file://agent/component/agent_with_tools.py#L99-L104)

## Core Components

### MCP Server Core

The MCP server implementation consists of several key components that work together to provide tool discovery and invocation capabilities:

```mermaid
classDiagram
class RAGFlowConnector {
+string base_url
+string version
+dict authorization_header
+list_datasets(page, page_size, orderby, desc, id, name) string
+retrieval(dataset_ids, document_ids, question, page, page_size, similarity_threshold, vector_similarity_weight, top_k, rerank_id, keyword, force_refresh) list
+bind_api_key(api_key) void
+_post(path, json, stream, files) Response
+_get(path, params, json) Response
}
class RAGFlowCtx {
+RAGFlowConnector conn
+__init__(connector)
}
class MCPToolCallSession {
+MCPServer _mcp_server
+dict _server_variables
+Queue _queue
+bool _close
+get_tools(timeout) list
+tool_call(name, arguments, timeout) string
+close() void
}
class MCPServer {
+string id
+string name
+string tenant_id
+string url
+string server_type
+string description
+dict variables
+dict headers
}
RAGFlowConnector --> RAGFlowCtx : "used by"
MCPToolCallSession --> MCPServer : "manages"
MCPServer --> RAGFlowConnector : "configured with"
```

**Diagram sources**
- [mcp/server/server.py](file://mcp/server/server.py#L58-L307)
- [common/mcp_tool_call_conn.py](file://common/mcp_tool_call_conn.py#L42-L325)
- [api/db/db_models.py](file://api/db/db_models.py#L959-L968)

**Section sources**
- [mcp/server/server.py](file://mcp/server/server.py#L58-L307)
- [common/mcp_tool_call_conn.py](file://common/mcp_tool_call_conn.py#L42-L325)

### Tool Registration and Discovery

The MCP server implements a sophisticated tool registration and discovery mechanism that allows LLMs to understand available tools and their capabilities:

```mermaid
sequenceDiagram
participant Client as MCP Client
participant Server as MCP Server
participant Connector as RAGFlow Connector
participant Backend as RAGFlow Backend
Client->>Server : list_tools()
Server->>Connector : list_datasets()
Connector->>Backend : GET /api/v1/datasets
Backend-->>Connector : Dataset list
Connector-->>Server : Formatted dataset description
Server-->>Client : Tool definition with dataset information
Note over Client,Backend : Tool discovery complete
Client->>Server : call_tool("ragflow_retrieval", args)
Server->>Connector : retrieval(dataset_ids, document_ids, question)
Connector->>Backend : POST /api/v1/retrieval
Backend-->>Connector : Search results
Connector-->>Server : Formatted response
Server-->>Client : Tool execution result
```

**Diagram sources**
- [mcp/server/server.py](file://mcp/server/server.py#L365-L494)
- [common/mcp_tool_call_conn.py](file://common/mcp_tool_call_conn.py#L183-L220)

**Section sources**
- [mcp/server/server.py](file://mcp/server/server.py#L365-L494)

## Server Implementation

### Launch Modes and Configuration

The MCP server supports two primary launch modes to accommodate different deployment scenarios:

#### Self-Host Mode
Self-host mode runs the MCP server for a single tenant with a predefined API key, suitable for isolated deployments where security isolation is paramount.

#### Host Mode  
Host mode operates as a multi-tenant server where clients must provide authentication credentials, enabling shared infrastructure with tenant isolation.

```mermaid
flowchart TD
Start([MCP Server Startup]) --> CheckMode{Launch Mode?}
CheckMode --> |Self-Host| ValidateKey{API Key Provided?}
CheckMode --> |Host| EnableMultiTenant[Enable Multi-Tenant Mode]
ValidateKey --> |Yes| BindKey[Bind API Key]
ValidateKey --> |No| Error[Throw UsageError]
BindKey --> SetupTransports[Setup Transports]
EnableMultiTenant --> SetupTransports
SetupTransports --> CheckSSE{SSE Enabled?}
SetupTransports --> CheckStreamable{Streamable HTTP Enabled?}
CheckSSE --> |Yes| EnableSSE[Enable SSE Transport]
CheckSSE --> |No| SkipSSE[Skip SSE Transport]
CheckStreamable --> |Yes| EnableStreamable[Enable Streamable HTTP Transport]
CheckStreamable --> |No| SkipStreamable[Skip Streamable HTTP Transport]
EnableSSE --> StartServer[Uvicorn Server Start]
SkipSSE --> StartServer
EnableStreamable --> StartServer
SkipStreamable --> StartServer
StartServer --> Ready[Server Ready]
Error --> Exit([Exit])
Ready --> End([Running])
```

**Diagram sources**
- [mcp/server/server.py](file://mcp/server/server.py#L626-L686)

**Section sources**
- [mcp/server/server.py](file://mcp/server/server.py#L38-L46)
- [mcp/server/server.py](file://mcp/server/server.py#L626-L686)

### Transport Protocols

The MCP server supports multiple transport protocols to ensure compatibility with different client implementations:

#### Server-Sent Events (SSE)
Legacy SSE transport provides real-time communication through persistent HTTP connections, ideal for browser-based clients and environments where WebSocket support is limited.

#### Streamable HTTP
Modern Streamable HTTP transport offers improved performance and reliability through HTTP/2 multiplexing and efficient streaming capabilities.

**Section sources**
- [mcp/server/server.py](file://mcp/server/server.py#L48-L56)
- [mcp/server/server.py](file://mcp/server/server.py#L496-L581)

### Request Routing and Authentication

The server implements comprehensive request routing and authentication mechanisms:

```mermaid
sequenceDiagram
participant Client as MCP Client
participant Middleware as Auth Middleware
participant Server as MCP Server
participant Handler as Request Handler
Client->>Middleware : HTTP Request
Middleware->>Middleware : Extract Authorization Header
Middleware->>Middleware : Validate Token
alt Valid Token
Middleware->>Server : Forward Request
Server->>Handler : Process Request
Handler-->>Server : Response
Server-->>Middleware : Response
Middleware-->>Client : Response
else Invalid Token
Middleware-->>Client : 401 Unauthorized
end
```

**Diagram sources**
- [mcp/server/server.py](file://mcp/server/server.py#L496-L581)

**Section sources**
- [mcp/server/server.py](file://mcp/server/server.py#L496-L581)

## Client Integration

### Client Implementation

The MCP client provides a streamlined interface for connecting to MCP servers and invoking tools:

```mermaid
classDiagram
class MCPClient {
+string server_url
+dict headers
+async initialize() void
+async list_tools() list
+async call_tool(name, arguments) Response
}
class ClientSession {
+AsyncIterator streams
+async initialize() void
+async list_tools() ListToolsResult
+async call_tool(name, arguments) CallToolResult
}
class SSEClient {
+string url
+dict headers
+async connect() AsyncIterator
}
class StreamableHTTPClient {
+string url
+dict headers
+async connect() tuple
}
MCPClient --> ClientSession : "uses"
ClientSession --> SSEClient : "may use"
ClientSession --> StreamableHTTPClient : "may use"
```

**Diagram sources**
- [mcp/client/client.py](file://mcp/client/client.py#L22-L47)

**Section sources**
- [mcp/client/client.py](file://mcp/client/client.py#L22-L47)

### Tool Invocation Workflow

The client follows a standardized workflow for tool discovery and invocation:

```mermaid
flowchart TD
Connect[Connect to MCP Server] --> InitSession[Initialize Client Session]
InitSession --> ListTools[List Tools]
ListTools --> SelectTool{Tool Selected?}
SelectTool --> |Yes| PrepareArgs[Prepare Arguments]
SelectTool --> |No| ListTools
PrepareArgs --> CallTool[Call Tool]
CallTool --> ProcessResult[Process Result]
ProcessResult --> FormatOutput[Format Output]
FormatOutput --> Complete[Complete]
CallTool --> HandleError{Error?}
HandleError --> |Yes| LogError[Log Error]
HandleError --> |No| ProcessResult
LogError --> Retry{Retry?}
Retry --> |Yes| CallTool
Retry --> |No| Complete
```

**Diagram sources**
- [mcp/client/client.py](file://mcp/client/client.py#L22-L47)

**Section sources**
- [mcp/client/client.py](file://mcp/client/client.py#L22-L47)

## Agent System Integration

### Agent-MCP Integration

The RAGFlow agent system seamlessly integrates with MCP servers to provide enhanced tool capabilities:

```mermaid
classDiagram
class AgentWithTools {
+dict tools
+list tool_meta
+MCPToolCallSession toolcall_session
+LLMBundle chat_mdl
+__init__(canvas, id, param)
+_invoke(**kwargs) string
+_load_tool_obj(cpn) object
}
class MCPToolCallSession {
+MCPServer _mcp_server
+dict _server_variables
+get_tools(timeout) list
+tool_call(name, arguments, timeout) string
}
class ToolParamBase {
+list tools
+list mcp
}
class AgentParam {
+ToolMeta meta
+string function_name
+list tools
+list mcp
+int max_rounds
+string description
}
AgentWithTools --> MCPToolCallSession : "manages"
AgentWithTools --> ToolParamBase : "inherits"
AgentParam --> ToolParamBase : "extends"
```

**Diagram sources**
- [agent/component/agent_with_tools.py](file://agent/component/agent_with_tools.py#L38-L106)

**Section sources**
- [agent/component/agent_with_tools.py](file://agent/component/agent_with_tools.py#L38-L106)

### Tool Discovery in Agents

Agents dynamically discover and integrate MCP tools during initialization:

```mermaid
sequenceDiagram
participant Agent as Agent Component
participant Service as MCP Server Service
participant Session as Tool Call Session
participant MCP as MCP Server
Agent->>Service : Get MCP Server by ID
Service-->>Agent : MCP Server Configuration
Agent->>Session : Create Tool Call Session
Session->>MCP : Initialize Connection
MCP-->>Session : Connection Established
Session->>MCP : List Available Tools
MCP-->>Session : Tool Definitions
Session-->>Agent : Tool Metadata
Agent->>Agent : Register Tools
Agent->>Agent : Bind Tools to LLM
```

**Diagram sources**
- [agent/component/agent_with_tools.py](file://agent/component/agent_with_tools.py#L99-L104)

**Section sources**
- [agent/component/agent_with_tools.py](file://agent/component/agent_with_tools.py#L99-L104)

## Configuration and Deployment

### Environment Variables

The MCP server supports extensive configuration through environment variables:

| Variable | Description | Default | Options |
|----------|-------------|---------|---------|
| `RAGFLOW_MCP_BASE_URL` | RAGFlow backend API URL | `http://127.0.0.1:9380` | Any valid URL |
| `RAGFLOW_MCP_HOST` | Server host address | `127.0.0.1` | IP address or hostname |
| `RAGFLOW_MCP_PORT` | Server port | `9382` | Port number (1-65535) |
| `RAGFLOW_MCP_LAUNCH_MODE` | Launch mode | `self-host` | `self-host`, `host` |
| `RAGFLOW_MCP_HOST_API_KEY` | API key for self-host mode | `""` | Any string |
| `RAGFLOW_MCP_TRANSPORT_SSE_ENABLED` | Enable SSE transport | `True` | `True`, `False` |
| `RAGFLOW_MCP_TRANSPORT_STREAMABLE_ENABLED` | Enable Streamable HTTP | `True` | `True`, `False` |
| `RAGFLOW_MCP_JSON_RESPONSE` | Enable JSON responses | `True` | `True`, `False` |

**Section sources**
- [mcp/server/server.py](file://mcp/server/server.py#L48-L56)
- [mcp/server/server.py](file://mcp/server/server.py#L626-L630)

### Production Deployment

For production deployments, consider the following configuration:

```bash
# Production configuration example
export RAGFLOW_MCP_BASE_URL="https://api.ragflow.com"
export RAGFLOW_MCP_HOST="0.0.0.0"
export RAGFLOW_MCP_PORT="9382"
export RAGFLOW_MCP_LAUNCH_MODE="host"
export RAGFLOW_MCP_HOST_API_KEY=""
export RAGFLOW_MCP_TRANSPORT_SSE_ENABLED="false"
export RAGFLOW_MCP_TRANSPORT_STREAMABLE_ENABLED="true"
export RAGFLOW_MCP_JSON_RESPONSE="true"
```

**Section sources**
- [mcp/server/server.py](file://mcp/server/server.py#L685-L716)

## Security Considerations

### Authentication and Authorization

The MCP server implements multiple security layers:

#### API Key Authentication
For self-host mode, API keys provide basic authentication for server access.

#### Bearer Token Support
Host mode supports OAuth 2.1 compliant Bearer tokens for enhanced security.

#### Request Validation
All requests undergo validation for proper authentication and authorization.

```mermaid
flowchart TD
Request[Incoming Request] --> ExtractAuth[Extract Auth Header]
ExtractAuth --> ValidateToken{Valid Token?}
ValidateToken --> |Yes| CheckPermissions[Check Permissions]
ValidateToken --> |No| Reject[Reject Request]
CheckPermissions --> HasPermission{Has Permission?}
HasPermission --> |Yes| ProcessRequest[Process Request]
HasPermission --> |No| Reject
ProcessRequest --> Success[Return Response]
Reject --> Unauthorized[401 Unauthorized]
```

**Diagram sources**
- [mcp/server/server.py](file://mcp/server/server.py#L329-L362)

**Section sources**
- [mcp/server/server.py](file://mcp/server/server.py#L329-L362)

### Rate Limiting and Throttling

The system implements several mechanisms to prevent abuse:

- **Timeout Controls**: Configurable timeouts for tool calls and server connections
- **Connection Pooling**: Efficient connection management for multiple concurrent requests
- **Resource Limits**: Built-in limits on memory and processing resources

**Section sources**
- [common/mcp_tool_call_conn.py](file://common/mcp_tool_call_conn.py#L153-L167)

## Performance and Best Practices

### Caching Strategies

The MCP server implements intelligent caching to improve performance:

#### Dataset Metadata Caching
Dataset information is cached with TTL (Time-To-Live) to reduce redundant API calls.

#### Document Metadata Caching
Document-level metadata is cached separately with individual TTL management.

#### Connection Pooling
Persistent connections are maintained for efficient tool invocation.

```mermaid
graph LR
subgraph "Cache Layers"
DatasetCache[Dataset Metadata Cache<br/>TTL: 300s<br/>Max: 32 entries]
DocCache[Document Metadata Cache<br/>TTL: 300s<br/>Max: 128 entries]
end
subgraph "Cache Operations"
GetCache[Get Cached Data]
CheckTTL{Valid TTL?}
UpdateCache[Update Cache]
EvictOld[Evict Old Entries]
end
DatasetCache --> GetCache
DocCache --> GetCache
GetCache --> CheckTTL
CheckTTL --> |Yes| ReturnCached[Return Cached Data]
CheckTTL --> |No| FetchNew[Fetch New Data]
FetchNew --> UpdateCache
UpdateCache --> EvictOld
EvictOld --> StoreData[Store in Cache]
```

**Diagram sources**
- [mcp/server/server.py](file://mcp/server/server.py#L58-L121)

**Section sources**
- [mcp/server/server.py](file://mcp/server/server.py#L58-L121)

### Error Handling and Resilience

The system implements comprehensive error handling:

#### Connection Failures
Automatic retry mechanisms with exponential backoff for transient failures.

#### Timeout Management
Configurable timeouts prevent hanging connections and resource exhaustion.

#### Graceful Degradation
Fallback mechanisms ensure system stability during partial failures.

**Section sources**
- [common/mcp_tool_call_conn.py](file://common/mcp_tool_call_conn.py#L70-L114)

### Best Practices for Deployment

1. **Environment Separation**: Use separate environments for development, staging, and production
2. **Monitoring**: Implement comprehensive logging and monitoring for operational visibility
3. **Scaling**: Consider horizontal scaling for high-traffic scenarios
4. **Backup**: Regular backups of MCP server configurations and tool definitions
5. **Testing**: Comprehensive testing of tool integrations before production deployment

## Troubleshooting

### Common Issues and Solutions

#### Connection Refused
**Symptoms**: Client cannot connect to MCP server
**Causes**: Server not running, incorrect host/port configuration
**Solutions**: Verify server status, check firewall settings, validate configuration

#### Authentication Failures
**Symptoms**: 401 Unauthorized responses
**Causes**: Invalid API key, missing authorization header
**Solutions**: Verify credentials, check header format, validate token expiration

#### Tool Discovery Failures
**Symptoms**: Tools not appearing in LLM responses
**Causes**: Network connectivity, server configuration issues
**Solutions**: Check network connectivity, verify server logs, validate tool definitions

#### Timeout Errors
**Symptoms**: Requests timing out
**Causes**: Network latency, server overload, inefficient tool implementations
**Solutions**: Increase timeout values, optimize tool performance, check server resources

### Debugging Tools

#### Logging Configuration
Enable detailed logging for troubleshooting:

```python
import logging
logging.basicConfig(level=logging.DEBUG)
```

#### Health Checks
Monitor server health through built-in endpoints and metrics.

#### Performance Monitoring
Track response times, error rates, and resource utilization.

**Section sources**
- [common/mcp_tool_call_conn.py](file://common/mcp_tool_call_conn.py#L70-L114)

## Conclusion

The RAGFlow MCP server implementation provides a robust, scalable solution for integrating external tools with LLMs through a standardized protocol. The system's architecture supports multiple deployment scenarios, from isolated self-hosted installations to multi-tenant production environments.

Key strengths of the implementation include:

- **Flexibility**: Support for multiple transport protocols and deployment modes
- **Scalability**: Efficient caching and connection management for high-performance operation
- **Security**: Comprehensive authentication and authorization mechanisms
- **Integration**: Seamless integration with the RAGFlow agent system and broader ecosystem
- **Reliability**: Robust error handling and resilience mechanisms

The MCP server serves as a critical component in enabling sophisticated AI agent workflows, allowing LLMs to leverage external knowledge bases, tools, and services through a standardized, secure, and performant interface.

Future enhancements could include expanded transport protocol support, enhanced monitoring capabilities, and additional security features to further strengthen the platform's capabilities and reliability.