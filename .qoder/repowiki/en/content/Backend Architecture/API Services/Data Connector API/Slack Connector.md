# Slack Connector API Documentation

<cite>
**Referenced Files in This Document**
- [slack_connector.py](file://common/data_source/slack_connector.py)
- [interfaces.py](file://common/data_source/interfaces.py)
- [models.py](file://common/data_source/models.py)
- [utils.py](file://common/data_source/utils.py)
- [config.py](file://common/data_source/config.py)
- [exceptions.py](file://common/data_source/exceptions.py)
- [connector_app.py](file://api/apps/connector_app.py)
- [__init__.py](file://common/data_source/__init__.py)
</cite>

## Table of Contents
1. [Introduction](#introduction)
2. [Architecture Overview](#architecture-overview)
3. [OAuth 2.0 Authentication Flow](#oauth-20-authentication-flow)
4. [Configuration Options](#configuration-options)
5. [Core Components](#core-components)
6. [Message Retrieval and Processing](#message-retrieval-and-processing)
7. [Endpoint Documentation](#endpoint-documentation)
8. [Error Handling and Troubleshooting](#error-handling-and-troubleshooting)
9. [Performance Considerations](#performance-considerations)
10. [Integration Examples](#integration-examples)

## Introduction

The Slack Connector API provides seamless integration with Slack workspaces to import messages, files, and channel content into knowledge bases for contextual retrieval. Built on the Slack SDK, this connector enables organizations to leverage their Slack conversations as searchable knowledge sources while maintaining thread context and user metadata.

The connector supports comprehensive workspace connectivity testing, flexible channel selection (public/private), date range filtering, user filtering, and file type inclusion. It transforms Slack messages into searchable content with timestamps, user metadata, and thread preservation capabilities.

## Architecture Overview

The Slack connector follows a modular architecture that separates concerns between authentication, data retrieval, processing, and integration:

```mermaid
graph TB
subgraph "Client Layer"
UI[Web Interface]
API[REST API Endpoints]
end
subgraph "Connector Layer"
SC[SlackConnector]
CC[CredentialsProvider]
TC[TextCleaner]
end
subgraph "Data Processing"
MR[Message Retrieval]
TP[Thread Processing]
FP[Filter Processing]
DT[Document Transformation]
end
subgraph "Slack API"
AT[Auth Test]
CL[Channels List]
CH[Conversations History]
CR[Conversations Replies]
CJ[Conversations Join]
end
subgraph "Knowledge Base"
KB[Knowledge Base Storage]
IDX[Index Service]
RET[Retrieval Service]
end
UI --> API
API --> SC
SC --> CC
SC --> TC
SC --> MR
MR --> TP
TP --> FP
FP --> DT
DT --> KB
KB --> IDX
IDX --> RET
SC --> AT
SC --> CL
SC --> CH
SC --> CR
SC --> CJ
```

**Diagram sources**
- [slack_connector.py](file://common/data_source/slack_connector.py#L465-L670)
- [interfaces.py](file://common/data_source/interfaces.py#L56-L102)

**Section sources**
- [slack_connector.py](file://common/data_source/slack_connector.py#L465-L490)
- [interfaces.py](file://common/data_source/interfaces.py#L56-L102)

## OAuth 2.0 Authentication Flow

The Slack connector implements OAuth 2.0 authentication with bot token usage for secure workspace access. The authentication process involves several key steps:

### Bot Token Configuration

Slack applications require bot tokens for accessing workspace data. The connector supports two types of clients:

1. **Standard Client**: Full-featured client for comprehensive operations
2. **Fast Client**: Lightweight client for quick validation and basic operations

### Required Scopes

The Slack app must be configured with appropriate OAuth scopes:

| Scope | Purpose | Required |
|-------|---------|----------|
| `channels:read` | Access public channel messages | Yes |
| `groups:read` | Access private channel messages | Conditional |
| `users:read` | Access user information | Yes |
| `chat:write` | Post messages (optional) | No |

### Authentication Validation

The connector performs comprehensive authentication validation:

```mermaid
sequenceDiagram
participant Client as "SlackConnector"
participant API as "Slack API"
participant Validator as "Settings Validator"
Client->>API : auth.test()
API-->>Client : Auth response
Client->>Validator : Validate auth response
Validator->>API : conversations.list(limit=1)
API-->>Validator : Channels list
Validator->>Validator : Check for errors
Validator-->>Client : Validation result
Note over Client,Validator : Handles rate limits, missing scopes,<br/>and authentication failures
```

**Diagram sources**
- [slack_connector.py](file://common/data_source/slack_connector.py#L585-L642)

**Section sources**
- [slack_connector.py](file://common/data_source/slack_connector.py#L580-L642)

## Configuration Options

The Slack connector provides extensive configuration options for flexible workspace integration:

### Channel Selection

Configure which channels to include in the synchronization:

```python
# Channel selection examples
channels = ["general", "announcements", "support"]
channel_regex_enabled = True  # Enable regex pattern matching
```

### Date Range Filtering

Filter messages by timestamp ranges:

```python
# Date range configuration
start_timestamp = "1609459200"  # January 1, 2021
end_timestamp = "1640995200"    # January 1, 2022
```

### Message Filtering

Customize message filtering logic:

```python
# Custom message filter function
def custom_filter(message: MessageType) -> SlackMessageFilterReason | None:
    # Filter by user, content type, or custom criteria
    if message.get("bot_id"):
        return SlackMessageFilterReason.BOT
    return None
```

### Performance Configuration

Optimize connector performance:

| Parameter | Default | Description |
|-----------|---------|-------------|
| `batch_size` | 2 | Documents per batch |
| `num_threads` | 4 | Concurrent processing threads |
| `timeout_secs` | 1740 | Operation timeout in seconds |
| `max_retries` | 7 | Retry attempts for failed operations |

**Section sources**
- [slack_connector.py](file://common/data_source/slack_connector.py#L472-L490)
- [config.py](file://common/data_source/config.py#L89-L101)

## Core Components

### SlackConnector Class

The main connector class implements multiple interfaces for comprehensive functionality:

```mermaid
classDiagram
class SlackConnector {
+channels : list[str] | None
+channel_regex_enabled : bool
+batch_size : int
+num_threads : int
+client : WebClient | None
+fast_client : WebClient | None
+text_cleaner : SlackTextCleaner | None
+user_cache : dict[str, BasicExpertInfo | None]
+credentials_provider : Any
+set_credentials_provider(credentials_provider)
+validate_connector_settings()
+retrieve_all_slim_docs_perm_sync()
+load_from_checkpoint()
}
class SlimConnectorWithPermSync {
<<interface>>
+retrieve_all_slim_docs_perm_sync()
}
class CredentialsConnector {
<<interface>>
+load_credentials()
+set_credentials_provider()
}
class CheckpointedConnectorWithPermSync {
<<interface>>
+load_from_checkpoint()
+build_dummy_checkpoint()
+validate_checkpoint_json()
}
SlackConnector --|> SlimConnectorWithPermSync
SlackConnector --|> CredentialsConnector
SlackConnector --|> CheckpointedConnectorWithPermSync
```

**Diagram sources**
- [slack_connector.py](file://common/data_source/slack_connector.py#L465-L490)
- [interfaces.py](file://common/data_source/interfaces.py#L56-L102)

### Message Processing Pipeline

The connector processes messages through a sophisticated pipeline:

```mermaid
flowchart TD
Start([Message Received]) --> Join{Channel Member?}
Join --> |No| JoinChannel[Join Channel]
JoinChannel --> Process[Process Message]
Join --> |Yes| Process
Process --> Thread{Has Thread?}
Thread --> |Yes| GetThread[Get Thread Messages]
Thread --> |No| SingleMsg[Single Message]
GetThread --> Filter[Apply Filters]
SingleMsg --> Filter
Filter --> HasFilter{Filtered Out?}
HasFilter --> |Yes| Skip[Skip Message]
HasFilter --> |No| Transform[Transform to Document]
Transform --> Store[Store in Knowledge Base]
Skip --> End([End])
Store --> End
```

**Diagram sources**
- [slack_connector.py](file://common/data_source/slack_connector.py#L319-L421)

**Section sources**
- [slack_connector.py](file://common/data_source/slack_connector.py#L465-L670)

## Message Retrieval and Processing

### Channel Discovery

The connector automatically discovers available channels with intelligent fallback:

```mermaid
flowchart TD
Start([Discover Channels]) --> TryBoth[Try Public & Private]
TryBoth --> PublicOnly{Public Channels Only?}
PublicOnly --> |Yes| LogWarning[Log Warning Message]
PublicOnly --> |No| Success[Success]
LogWarning --> Fallback[Fallback to Public Only]
Fallback --> Success
Success --> Filter[Filter Selected Channels]
Filter --> End([Channel List])
```

**Diagram sources**
- [slack_connector.py](file://common/data_source/slack_connector.py#L90-L122)

### Thread Context Preservation

The connector maintains thread context by retrieving complete conversation threads:

```python
# Thread processing example
def get_thread(client: WebClient, channel_id: str, thread_id: str) -> ThreadType:
    """Retrieve complete thread conversation"""
    threads: list[MessageType] = []
    for result in make_paginated_slack_api_call(
        client.conversations_replies, 
        channel=channel_id, 
        ts=thread_id
    ):
        threads.extend(result["messages"])
    return threads
```

### Content Transformation

Messages are transformed into searchable documents with metadata:

| Field | Description | Example |
|-------|-------------|---------|
| `id` | Unique document identifier | `C012AB3CD__1609459200.000000` |
| `semantic_identifier` | Human-readable title | `John Doe in #general: Discussion about...` |
| `sections` | Message content with links | `[TextSection(link, text)]` |
| `primary_owners` | Thread participants | `[BasicExpertInfo(display_name="John")]` |
| `metadata` | Channel and timestamp info | `{"Channel": "general", "Timestamp": "..."}` |

**Section sources**
- [slack_connector.py](file://common/data_source/slack_connector.py#L155-L232)
- [slack_connector.py](file://common/data_source/slack_connector.py#L319-L421)

## Endpoint Documentation

### Create Connector

Create a new Slack connector instance:

**Endpoint:** `POST /api/connector/set`

**Request Body:**
```json
{
    "name": "Slack Workspace Connector",
    "source": "slack",
    "config": {
        "channels": ["general", "announcements"],
        "channel_regex_enabled": false,
        "batch_size": 2,
        "num_threads": 4
    },
    "refresh_freq": 30,
    "prune_freq": 720,
    "timeout_secs": 1740
}
```

**Response:**
```json
{
    "ret_code": 0,
    "ret_msg": "success",
    "data": {
        "id": "uuid-string",
        "name": "Slack Workspace Connector",
        "source": "slack",
        "config": {...},
        "status": "SCHEDULE"
    }
}
```

### Test Workspace Connectivity

Validate connector settings and workspace access:

**Endpoint:** `POST /api/connector/{connector_id}/validate`

**Response:**
```json
{
    "ret_code": 0,
    "ret_msg": "success",
    "data": true
}
```

### Initiate Message Synchronization

Start the synchronization process:

**Endpoint:** `POST /api/connector/{connector_id}/sync`

**Response:**
```json
{
    "ret_code": 0,
    "ret_msg": "success",
    "data": {
        "task_id": "uuid-string",
        "status": "RUNNING"
    }
}
```

### List Connector Logs

Monitor synchronization progress:

**Endpoint:** `GET /api/connector/{connector_id}/logs`

**Parameters:**
- `page`: Page number (default: 1)
- `page_size`: Items per page (default: 15)

**Response:**
```json
{
    "ret_code": 0,
    "ret_msg": "success",
    "data": {
        "total": 100,
        "logs": [...]
    }
}
```

**Section sources**
- [connector_app.py](file://api/apps/connector_app.py#L38-L118)

## Error Handling and Troubleshooting

### Common Error Types

The connector handles various error scenarios with specific exception types:

```mermaid
graph TD
Error[Connector Error] --> MissingCreds[ConnectorMissingCredentialError]
Error --> ValidationError[ConnectorValidationError]
Error --> ExpiredCreds[CredentialExpiredError]
Error --> InsufficientPerms[InsufficientPermissionsError]
Error --> Unexpected[UnexpectedValidationError]
Error --> RateLimit[RateLimitTriedTooManyTimesError]
MissingCreds --> Solution1[Provide valid credentials]
ValidationError --> Solution2[Check connector configuration]
ExpiredCreds --> Solution3[Refresh authentication token]
InsufficientPerms --> Solution4[Grant required OAuth scopes]
Unexpected --> Solution5[Contact support]
RateLimit --> Solution6[Implement exponential backoff]
```

**Diagram sources**
- [exceptions.py](file://common/data_source/exceptions.py#L4-L30)

### Rate Limiting

The connector implements robust rate limiting:

| Scenario | Action | Backoff Strategy |
|----------|--------|------------------|
| API Rate Limit | Exponential backoff | 2^n seconds (max 60s) |
| Too Many Requests | Retry with delay | 10-second delay |
| Temporary Failure | Linear backoff | Incremental delays |

### Troubleshooting Guide

#### Authentication Issues

1. **Invalid Bot Token**
   - Verify token format and validity
   - Check OAuth scope permissions
   - Ensure token hasn't expired

2. **Missing Permissions**
   - Add `channels:read` scope
   - Add `groups:read` for private channels
   - Verify user role permissions

#### Network Issues

1. **Connection Timeouts**
   - Increase timeout settings
   - Check network connectivity
   - Verify firewall settings

2. **Rate Limit Exceeded**
   - Reduce batch size
   - Increase retry delays
   - Monitor API usage

**Section sources**
- [slack_connector.py](file://common/data_source/slack_connector.py#L612-L642)
- [exceptions.py](file://common/data_source/exceptions.py#L4-L30)

## Performance Considerations

### Optimization Strategies

The connector implements several performance optimizations:

1. **Parallel Processing**: Uses configurable thread pools for concurrent message processing
2. **Batch Operations**: Processes messages in configurable batches to optimize API usage
3. **Caching**: Maintains user information cache to reduce API calls
4. **Pagination**: Handles large datasets with efficient pagination

### Memory Management

| Component | Memory Usage | Optimization |
|-----------|--------------|--------------|
| Message Cache | O(n) where n = messages | LRU eviction policy |
| User Cache | O(u) where u = users | Periodic cleanup |
| Thread Cache | O(t) where t = threads | Lazy loading |
| Document Queue | O(b) where b = batch_size | Streaming processing |

### Scalability Limits

| Resource | Limit | Recommendation |
|----------|-------|----------------|
| API Calls | 20,000 per hour | Monitor usage closely |
| Message Size | 250KB per message | Implement truncation |
| Thread Depth | Unlimited | Set reasonable limits |
| Channel Count | 10,000 per workspace | Filter selectively |

**Section sources**
- [config.py](file://common/data_source/config.py#L89-L101)
- [slack_connector.py](file://common/data_source/slack_connector.py#L472-L490)

## Integration Examples

### Basic Connector Setup

```python
from common.data_source.slack_connector import SlackConnector

# Initialize connector
connector = SlackConnector(
    channels=["general", "announcements"],
    channel_regex_enabled=False,
    batch_size=2,
    num_threads=4
)

# Set credentials
credentials = {
    "slack_bot_token": "xoxb-your-bot-token"
}

class SimpleCredentialsProvider:
    def get_credentials(self):
        return credentials

connector.set_credentials_provider(SimpleCredentialsProvider())

# Validate configuration
connector.validate_connector_settings()
```

### Advanced Configuration

```python
import re
from datetime import datetime, timedelta

# Custom message filter
def advanced_filter(message: MessageType) -> SlackMessageFilterReason | None:
    # Filter bots and system messages
    if message.get("bot_id") or message.get("subtype") in ["channel_join", "channel_leave"]:
        return SlackMessageFilterReason.BOT
    
    # Filter old messages
    timestamp = float(message.get("ts", 0))
    cutoff_date = datetime.now() - timedelta(days=30)
    if datetime.fromtimestamp(timestamp).replace(tzinfo=None) < cutoff_date:
        return SlackMessageFilterReason.OLD
    
    return None

# Configure connector with custom filter
connector = SlackConnector(
    channels=["team-discussions"],
    channel_regex_enabled=True,
    batch_size=10,
    num_threads=8
)

# Apply custom filter
connector.message_filter = advanced_filter
```

### Large Workspace Handling

```python
# Efficient processing for large workspaces
connector = SlackConnector(
    channels=None,  # Process all channels
    channel_regex_enabled=True,
    batch_size=20,  # Larger batches
    num_threads=16  # More threads
)

# Implement checkpointing for long-running operations
checkpoint = connector.build_dummy_checkpoint()
try:
    for document_batch in connector.load_from_checkpoint(
        start=earliest_timestamp,
        end=current_timestamp,
        checkpoint=checkpoint
    ):
        # Process document batch
        process_documents(document_batch)
except Exception as e:
    # Save checkpoint for recovery
    save_checkpoint(connector.validate_checkpoint_json(checkpoint))
```

**Section sources**
- [slack_connector.py](file://common/data_source/slack_connector.py#L645-L670)