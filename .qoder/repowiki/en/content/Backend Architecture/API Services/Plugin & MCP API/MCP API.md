# MCP API

<cite>
**Referenced Files in This Document**   
- [mcp_server_app.py](file://api/apps/mcp_server_app.py)
- [mcp_server_service.py](file://api/db/services/mcp_server_service.py)
- [mcp_tool_call_conn.py](file://common/mcp_tool_call_conn.py)
- [api_utils.py](file://api/utils/api_utils.py)
- [constants.py](file://common/constants.py)
- [db_models.py](file://api/db/db_models.py)
- [mcp.ts](file://web/src/interfaces/database/mcp.ts)
</cite>

## Table of Contents
1. [Introduction](#introduction)
2. [MCP Server Management](#mcp-server-management)
3. [Tool Discovery and Management](#tool-discovery-and-management)
4. [Tool Execution](#tool-execution)
5. [Request/Response Formats](#requestresponse-formats)
6. [Security Considerations](#security-considerations)
7. [API Endpoints](#api-endpoints)
8. [Examples](#examples)

## Introduction

The Model Context Protocol (MCP) server integration in RAGFlow enables communication between RAGFlow agents and external tools through a standardized protocol. This API documentation covers all endpoints in `mcp_server_app.py` for managing MCP servers, including server registration, configuration, health checking, and tool discovery.

MCP servers act as bridges between RAGFlow agents and external tools, allowing agents to discover and invoke tools from various services. The integration supports two transport types: Server-Sent Events (SSE) and Streamable HTTP, as defined by the `MCPServerType` enum. Each MCP server is associated with a tenant and can host multiple tools that agents can utilize in their workflows.

The MCP integration follows a client-server architecture where RAGFlow acts as the MCP client, connecting to external MCP servers that expose tools through the standardized MCP protocol. When an MCP server is configured, RAGFlow establishes a session to discover available tools and cache their specifications for efficient invocation.

**Section sources**
- [mcp_server_app.py](file://api/apps/mcp_server_app.py#L1-L443)
- [constants.py](file://common/constants.py#L138-L142)

## MCP Server Management

MCP server management endpoints allow users to create, update, delete, list, and test MCP server configurations. Each MCP server configuration includes essential information such as name, URL, server type, headers, and variables.

The MCP server lifecycle begins with registration through the `/create` endpoint, where users provide the server details. During creation, RAGFlow automatically connects to the MCP server to discover and cache available tools. The server configuration is tenant-scoped, ensuring isolation between different tenants in multi-tenant deployments.

When updating an MCP server configuration, the system re-validates the connection and refreshes the tool cache to ensure the tool list remains current. The `/test_mcp` endpoint allows users to validate MCP server connectivity and tool availability without persisting the configuration, serving as a pre-flight check before server registration.

**Section sources**
- [mcp_server_app.py](file://api/apps/mcp_server_app.py#L71-L265)
- [mcp_server_service.py](file://api/db/services/mcp_server_service.py#L22-L92)

## Tool Discovery and Management

Tool discovery is a core functionality of the MCP integration, enabling RAGFlow agents to identify and utilize external tools. The tool discovery process involves connecting to the MCP server, retrieving tool specifications, and caching them for subsequent use.

The `/list_tools` endpoint retrieves tool specifications from one or more MCP servers. For each server, the system establishes a connection using the configured transport type (SSE or Streamable HTTP) and requests the list of available tools. The tool specifications include the tool name, description, input schema, and enabled status.

Tool caching is implemented to optimize performance and reduce network overhead. When an MCP server is created or updated, the system automatically discovers available tools and stores their specifications in the server's variables field. This cached information allows for quick tool listing without requiring real-time connections to external servers.

The `/cache_tools` endpoint allows manual updating of the tool cache, enabling users to refresh tool specifications or modify tool configurations such as enabling/disabling specific tools.

**Section sources**
- [mcp_server_app.py](file://api/apps/mcp_server_app.py#L299-L400)
- [mcp_tool_call_conn.py](file://common/mcp_tool_call_conn.py#L42-L249)
- [api_utils.py](file://api/utils/api_utils.py#L641-L668)

## Tool Execution

Tool execution allows RAGFlow agents to invoke external tools through the MCP server integration. The execution process involves establishing a session with the MCP server, sending the tool call request with appropriate arguments, and handling the response.

The `/test_tool` endpoint provides a way to execute a specific tool with given arguments. This endpoint is useful for testing tool functionality and validating input parameters before integrating the tool into agent workflows. The execution process respects the timeout configuration and handles various error conditions such as connection failures, authentication errors, and tool execution errors.

The MCP tool call session manages the connection lifecycle, handling protocol-specific details for SSE and Streamable HTTP transports. For SSE transport, the system establishes a persistent connection to receive tool responses as server-sent events. For Streamable HTTP transport, the system uses HTTP streaming to receive responses incrementally.

Error handling during tool execution includes timeout detection, connection failure recovery, and proper propagation of error messages from the MCP server to the client. The system also handles cases where the tool returns an error response, ensuring that the error information is properly formatted and returned to the caller.

**Section sources**
- [mcp_server_app.py](file://api/apps/mcp_server_app.py#L346-L376)
- [mcp_tool_call_conn.py](file://common/mcp_tool_call_conn.py#L206-L218)

## Request/Response Formats

The MCP API uses standardized request and response formats for consistency and ease of integration. All endpoints follow a common response structure with code, message, and data fields.

### MCP Server Object
The MCP server object represents a configured MCP server instance with the following properties:

| Field | Type | Description |
|-------|------|-------------|
| id | string | Unique identifier for the MCP server |
| name | string | User-defined name for the MCP server |
| url | string | Base URL of the MCP server |
| server_type | string | Transport type (sse or streamable-http) |
| create_date | string | Creation timestamp in ISO format |
| update_date | string | Last update timestamp in ISO format |
| variables | object | Configuration variables including tools cache |
| headers | object | HTTP headers to include in requests to the MCP server |

### Tool Object
The tool object represents a discoverable tool on an MCP server with the following properties:

| Field | Type | Description |
|-------|------|-------------|
| name | string | Unique name of the tool |
| description | string | Description of the tool's functionality |
| enabled | boolean | Whether the tool is enabled for use |
| inputSchema | object | JSON Schema defining the tool's input parameters |

### Input Schema
The input schema follows JSON Schema specification to define the structure and validation rules for tool input parameters:

| Field | Type | Description |
|-------|------|-------------|
| type | string | Type of the input (object, string, number, etc.) |
| title | string | Title of the schema |
| properties | object | Map of property names to their schema definitions |
| required | array | List of required property names |

**Section sources**
- [mcp.ts](file://web/src/interfaces/database/mcp.ts#L1-L60)
- [mcp_server_app.py](file://api/apps/mcp_server_app.py#L1-L443)

## Security Considerations

The MCP server integration implements several security measures to protect against unauthorized access and ensure secure communication with external tools.

Authentication is handled through the headers configuration, allowing users to specify authorization tokens and other authentication credentials. The system supports template variables in headers, enabling dynamic credential injection based on runtime context.

All MCP server operations are tenant-scoped, ensuring that users can only access and manage MCP servers within their tenant. The API enforces authentication and authorization checks on all endpoints, preventing unauthorized access to MCP server configurations and tool execution.

The system implements timeout controls for all MCP server operations, preventing long-running requests from consuming excessive resources. Connection pooling and session management help optimize resource usage while maintaining security boundaries between different MCP server connections.

When executing tool calls, the system validates input parameters against the tool's input schema to prevent injection attacks and ensure data integrity. Error messages are sanitized to avoid leaking sensitive information about the internal system or the MCP server configuration.

**Section sources**
- [mcp_server_app.py](file://api/apps/mcp_server_app.py#L73-L442)
- [mcp_tool_call_conn.py](file://common/mcp_tool_call_conn.py#L59-L113)

## API Endpoints

### MCP Server Management Endpoints

```mermaid
flowchart TD
A["MCP Server Management Endpoints"] --> B["/list"]
A --> C["/detail"]
A --> D["/create"]
A --> E["/update"]
A --> F["/rm"]
A --> G["/import"]
A --> H["/export"]
B --> I["List MCP servers with pagination"]
C --> J["Get detailed information about a specific MCP server"]
D --> K["Create a new MCP server configuration"]
E --> L["Update an existing MCP server configuration"]
F --> M["Remove one or more MCP servers"]
G --> N["Import multiple MCP server configurations"]
H --> O["Export MCP server configurations"]
```

**Diagram sources **
- [mcp_server_app.py](file://api/apps/mcp_server_app.py#L30-L297)

### Tool Management Endpoints

```mermaid
flowchart TD
A["Tool Management Endpoints"] --> B["/list_tools"]
A --> C["/test_tool"]
A --> D["/cache_tools"]
A --> E["/test_mcp"]
B --> F["Retrieve available tools from MCP servers"]
C --> G["Execute a specific tool with arguments"]
D --> H["Update the tool cache for an MCP server"]
E --> I["Test MCP server connectivity and tool discovery"]
```

**Diagram sources **
- [mcp_server_app.py](file://api/apps/mcp_server_app.py#L299-L402)

## Examples

### Creating an MCP Server

```json
POST /mcp_server/create
{
  "name": "github-tools",
  "url": "https://mcp.github.com/v1",
  "server_type": "sse",
  "headers": {
    "Authorization": "Bearer {{authorization_token}}"
  },
  "variables": {
    "authorization_token": "ghp_XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
  }
}
```

### Listing Available Tools

```json
POST /mcp_server/list_tools
{
  "mcp_ids": ["github-tools"]
}
```

Response:
```json
{
  "code": 0,
  "message": "Success",
  "data": {
    "github-tools": [
      {
        "name": "search_repositories",
        "description": "Search GitHub repositories by query",
        "enabled": true,
        "inputSchema": {
          "type": "object",
          "properties": {
            "query": {
              "type": "string",
              "description": "Search query"
            },
            "limit": {
              "type": "integer",
              "description": "Maximum number of results"
            }
          },
          "required": ["query"]
        }
      }
    ]
  }
}
```

### Executing a Tool

```json
POST /mcp_server/test_tool
{
  "mcp_id": "github-tools",
  "tool_name": "search_repositories",
  "arguments": {
    "query": "machine learning",
    "limit": 5
  }
}
```

**Section sources**
- [mcp_server_app.py](file://api/apps/mcp_server_app.py#L71-L442)
- [mcp_tool_call_conn.py](file://common/mcp_tool_call_conn.py#L42-L249)