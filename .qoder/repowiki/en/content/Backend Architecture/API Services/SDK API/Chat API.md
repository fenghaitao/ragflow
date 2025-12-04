# Chat API

<cite>
**Referenced Files in This Document**
- [conversation_app.py](file://api/apps/conversation_app.py)
- [dialog_app.py](file://api/apps/dialog_app.py)
- [chat.py](file://api/apps/sdk/chat.py)
- [session.py](file://api/apps/sdk/session.py)
- [chat.py](file://sdk/python/ragflow_sdk/modules/chat.py)
- [session.py](file://sdk/python/ragflow_sdk/modules/session.py)
- [conversation_service.py](file://api/db/services/conversation_service.py)
- [dialog_service.py](file://api/db/services/dialog_service.py)
- [api_utils.py](file://api/utils/api_utils.py)
- [http_api_reference.md](file://docs/references/http_api_reference.md)
- [set_chat_variables.md](file://docs/guides/chat/set_chat_variables.md)
</cite>

## Table of Contents
1. [Introduction](#introduction)
2. [Authentication and Authorization](#authentication-and-authorization)
3. [Chat Assistant Management](#chat-assistant-management)
4. [Session Management](#session-management)
5. [Conversation API](#conversation-api)
6. [Streaming Responses](#streaming-responses)
7. [Chat Variables and Configuration](#chat-variables-and-configuration)
8. [Error Handling](#error-handling)
9. [Rate Limiting](#rate-limiting)
10. [Python SDK Usage](#python-sdk-usage)
11. [Examples](#examples)

## Introduction

The Chat API provides comprehensive endpoints for managing AI-powered chat assistants, sessions, and conversations. It enables developers to create interactive chat experiences with knowledge base integration, streaming responses, and flexible variable configuration.

The API follows RESTful principles and supports both synchronous and asynchronous operations. It integrates seamlessly with RAGFlow's knowledge management system, allowing chat assistants to leverage document embeddings and retrieval-augmented generation (RAG).

## Authentication and Authorization

All Chat API endpoints require authentication using an API key. The authentication mechanism supports both bearer token format and API token validation.

### Authentication Methods

**Bearer Token Authentication:**
```http
Authorization: Bearer <YOUR_API_KEY>
```

**API Token Validation:**
The system validates API tokens against the tenant context, ensuring proper access control for chat resources.

### Authentication Requirements

| Endpoint Category | Authentication Required |
|-------------------|------------------------|
| Chat Assistant Management | Yes |
| Session Management | Yes |
| Conversation Operations | Yes |
| Streaming Responses | Yes |

**Section sources**
- [api_utils.py](file://api/utils/api_utils.py#L250-L280)
- [conversation_app.py](file://api/apps/conversation_app.py#L1-L50)

## Chat Assistant Management

Chat assistants serve as the foundation for conversational AI experiences. They encapsulate LLM configurations, knowledge bases, and prompt templates.

### Create Chat Assistant

Creates a new chat assistant with specified configurations.

**Endpoint:** `POST /api/v1/chats`

**Request Parameters:**
- `name` (string, required): Unique name for the chat assistant
- `dataset_ids` (array): Associated knowledge base IDs
- `llm` (object): LLM configuration
- `prompt` (object): Prompt configuration
- `description` (string): Assistant description
- `avatar` (string): Avatar image URL

**Request Schema:**
```json
{
  "name": "Customer Support Bot",
  "dataset_ids": ["kb_123", "kb_456"],
  "llm": {
    "model_name": "gpt-4",
    "temperature": 0.7,
    "max_tokens": 1000
  },
  "prompt": {
    "system": "You are a helpful customer support assistant.",
    "opener": "Hello! How can I assist you today?",
    "variables": [
      {"key": "knowledge", "optional": false}
    ]
  }
}
```

**Response:**
```json
{
  "code": 0,
  "data": {
    "id": "chat_789",
    "name": "Customer Support Bot",
    "description": "Helps with customer inquiries",
    "llm": {
      "model_name": "gpt-4",
      "temperature": 0.7
    },
    "prompt": {
      "system": "You are a helpful customer support assistant.",
      "opener": "Hello! How can I assist you today?",
      "variables": [{"key": "knowledge", "optional": false}]
    }
  }
}
```

### Update Chat Assistant

Modifies existing chat assistant configurations.

**Endpoint:** `PUT /api/v1/chats/{chat_id}`

**Request Parameters:**
- `name` (string): Updated assistant name
- `dataset_ids` (array): New knowledge base associations
- `llm` (object): Updated LLM settings
- `prompt` (object): Modified prompt configuration

### Delete Chat Assistants

Removes one or more chat assistants from the system.

**Endpoint:** `DELETE /api/v1/chats`

**Request Body:**
```json
{
  "ids": ["chat_123", "chat_456"]
}
```

### List Chat Assistants

Retrieves a paginated list of available chat assistants.

**Endpoint:** `GET /api/v1/chats`

**Query Parameters:**
- `page` (integer): Page number (default: 1)
- `page_size` (integer): Items per page (default: 30)
- `orderby` (string): Sort field
- `desc` (boolean): Descending order flag

**Response:**
```json
{
  "code": 0,
  "data": [
    {
      "id": "chat_789",
      "name": "Customer Support Bot",
      "datasets": [{"id": "kb_123", "name": "Support Docs"}]
    }
  ]
}
```

**Section sources**
- [chat.py](file://api/apps/sdk/chat.py#L27-L141)
- [chat.py](file://sdk/python/ragflow_sdk/modules/chat.py#L22-L88)

## Session Management

Sessions represent individual conversation instances within a chat assistant. They maintain conversation history and context.

### Create Session

Initializes a new conversation session.

**Endpoint:** `POST /api/v1/chats/{chat_id}/sessions`

**Request Body:**
```json
{
  "name": "New Support Session",
  "user_id": "user_123"
}
```

**Response:**
```json
{
  "code": 0,
  "data": {
    "id": "session_456",
    "chat_id": "chat_789",
    "name": "New Support Session",
    "messages": [
      {
        "role": "assistant",
        "content": "Hello! How can I assist you today?"
      }
    ]
  }
}
```

### Update Session

Modifies session properties.

**Endpoint:** `PUT /api/v1/chats/{chat_id}/sessions/{session_id}`

**Request Body:**
```json
{
  "name": "Updated Session Name",
  "user_id": "user_456"
}
```

### List Sessions

Retrieves all sessions for a chat assistant.

**Endpoint:** `GET /api/v1/chats/{chat_id}/sessions`

**Query Parameters:**
- `page` (integer): Page number
- `page_size` (integer): Items per page
- `orderby` (string): Sort field
- `desc` (boolean): Descending order
- `name` (string): Filter by session name
- `id` (string): Filter by session ID
- `user_id` (string): Filter by user ID

### Delete Sessions

Removes one or more sessions.

**Endpoint:** `DELETE /api/v1/chats/{chat_id}/sessions`

**Request Body:**
```json
{
  "ids": ["session_123", "session_456"]
}
```

**Section sources**
- [session.py](file://api/apps/sdk/session.py#L480-L512)
- [session.py](file://sdk/python/ragflow_sdk/modules/session.py#L21-L129)

## Conversation API

The core conversation API handles user interactions with chat assistants, supporting both single-turn and multi-turn conversations.

### Converse with Chat Assistant

Initiates or continues a conversation with a chat assistant.

**Endpoint:** `POST /api/v1/chats/{chat_id}/completions`

**Request Parameters:**
- `question` (string, required): User's question
- `stream` (boolean): Enable streaming response
- `session_id` (string): Existing session ID
- `user_id` (string): User identifier
- Additional variables defined in prompt configuration

**Request Schema:**
```json
{
  "question": "How do I reset my password?",
  "stream": true,
  "session_id": "session_456",
  "language": "english"
}
```

**Response (Streaming):**
```json
data: {"code": 0, "message": "", "data": {"answer": "To reset your password"}}
data: {"code": 0, "message": "", "data": {"answer": " follow these steps:"}}
data: {"code": 0, "message": "", "data": {"answer": "1. Go to the login page"}}
data: {"code": 0, "message": "", "data": true}
```

**Response (Non-Streaming):**
```json
{
  "code": 0,
  "data": {
    "answer": "To reset your password, follow these steps:\n1. Go to the login page...",
    "reference": {
      "chunks": [
        {
          "content": "Password reset instructions...",
          "doc_name": "support_guide.pdf"
        }
      ]
    },
    "session_id": "session_456"
  }
}
```

### Message Format Structure

**Message Object:**
```json
{
  "role": "user|assistant",
  "content": "Message content",
  "id": "unique_message_id",
  "created_at": 1703123456.789,
  "reference": {
    "chunks": [...],
    "doc_aggs": [...]
  }
}
```

**Answer Object:**
```json
{
  "answer": "Response text",
  "reference": {
    "chunks": [
      {
        "content": "Relevant knowledge chunk",
        "doc_name": "source_document.pdf",
        "doc_id": "doc_123",
        "chunk_id": "chunk_456"
      }
    ]
  },
  "session_id": "session_789",
  "id": "message_123"
}
```

**Section sources**
- [conversation_app.py](file://api/apps/conversation_app.py#L168-L251)
- [conversation_service.py](file://api/db/services/conversation_service.py#L93-L243)

## Streaming Responses

The API supports Server-Sent Events (SSE) for real-time streaming of AI responses.

### Streaming Implementation

**Headers:**
```http
Content-Type: text/event-stream
Cache-Control: no-cache
Connection: keep-alive
X-Accel-Buffering: no
```

**Event Stream Format:**
```
data: {"code": 0, "message": "", "data": {"answer": "Partial response..."}}

data: {"code": 0, "message": "", "data": true}
```

**End of Stream Marker:**
```
data: [DONE]
```

### Streaming Benefits

- Real-time response delivery
- Progressive content rendering
- Improved user experience
- Memory-efficient for long responses

**Section sources**
- [conversation_service.py](file://api/db/services/conversation_service.py#L149-L159)
- [session.py](file://sdk/python/ragflow_sdk/modules/session.py#L36-L79)

## Chat Variables and Configuration

Chat variables enable dynamic prompt customization and parameter passing.

### Variable Types

**System Reserved Variable:**
- `{knowledge}`: Automatically populated with retrieved knowledge chunks

**Custom Variables:**
- User-defined parameters for prompt customization
- Configurable as optional or required
- Passed through API requests

### Variable Configuration

**Prompt Configuration Schema:**
```json
{
  "system": "You are a {style} assistant answering questions about {topic}. Knowledge: {knowledge}",
  "parameters": [
    {"key": "style", "optional": false},
    {"key": "topic", "optional": true}
  ],
  "opener": "Hello! I'm a {style} assistant.",
  "empty_response": "I couldn't find relevant information about {topic}."
}
```

### Passing Variables

**HTTP API Example:**
```bash
curl --request POST \
     --url http://localhost:9380/api/v1/chats/{chat_id}/completions \
     --header 'Content-Type: application/json' \
     --header 'Authorization: Bearer <API_KEY>' \
     --data '{
          "question": "Explain quantum computing",
          "style": "formal",
          "topic": "quantum computing"
     }'
```

**Python SDK Example:**
```python
session = assistant.create_session()
for answer in session.ask(
    "Explain quantum computing",
    style="formal",
    topic="quantum computing"
):
    print(answer.content)
```

### Variable Validation

The system validates variable usage in prompts and ensures required variables are provided.

**Section sources**
- [set_chat_variables.md](file://docs/guides/chat/set_chat_variables.md#L1-L111)
- [chat.py](file://api/apps/sdk/chat.py#L95-L105)

## Error Handling

The API implements comprehensive error handling with standardized response formats.

### Error Response Format

```json
{
  "code": 102,
  "message": "Name cannot be empty.",
  "data": null
}
```

### Common Error Codes

| Code | Description | HTTP Status |
|------|-------------|-------------|
| 0 | Success | 200 OK |
| 102 | Validation error | 400 Bad Request |
| 103 | Authentication error | 401 Unauthorized |
| 104 | Permission denied | 403 Forbidden |
| 105 | Resource not found | 404 Not Found |
| 500 | Internal server error | 500 Internal Server Error |

### Error Categories

**Authentication Errors:**
- Invalid API key
- Expired access token
- Insufficient permissions

**Validation Errors:**
- Missing required parameters
- Invalid parameter formats
- Duplicate resource names

**Business Logic Errors:**
- Non-existent sessions
- Invalid knowledge base associations
- Model configuration conflicts

### Error Handling Best Practices

**Client-Side Handling:**
```python
try:
    response = rag.list_chats()
    if response["code"] != 0:
        raise Exception(f"API Error: {response['message']}")
except Exception as e:
    # Handle error gracefully
    logger.error(f"Chat API error: {e}")
```

**Section sources**
- [api_utils.py](file://api/utils/api_utils.py#L113-L144)
- [conversation_app.py](file://api/apps/conversation_app.py#L37-L82)

## Rate Limiting

The API implements rate limiting to ensure fair usage and system stability.

### Rate Limiting Policies

**Request Rate Limits:**
- 100 requests per minute per API key
- Burst allowance: 50 additional requests
- Reset period: 1 minute

**Resource Limits:**
- Maximum 10 concurrent streaming connections
- Session retention: 30 days inactive
- Message history: 100 messages per session

### Rate Limit Headers

```http
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1640995200
Retry-After: 60
```

### Handling Rate Limits

**Client Implementation:**
```python
import time
from requests.exceptions import HTTPError

def chat_with_retry(client, chat_id, question, max_retries=3):
    for attempt in range(max_retries):
        try:
            return client.ask(chat_id, question)
        except HTTPError as e:
            if e.response.status_code == 429:  # Rate limited
                wait_time = int(e.headers.get('Retry-After', 60))
                time.sleep(wait_time)
                continue
            raise
    raise Exception("Max retries exceeded")
```

## Python SDK Usage

The Python SDK provides convenient methods for interacting with the Chat API.

### Basic Setup

```python
from ragflow_sdk import RAGFlow

# Initialize client
rag = RAGFlow(api_key="your_api_key", base_url="http://localhost:9380")

# Create or get chat assistant
assistant = rag.create_chat(
    name="My Assistant",
    dataset_ids=["kb_123"],
    llm={
        "model_name": "gpt-4",
        "temperature": 0.7
    }
)
```

### Session Management

```python
# Create session
session = assistant.create_session(name="Support Session")

# List sessions
sessions = assistant.list_sessions(
    page=1,
    page_size=10,
    orderby="create_time",
    desc=True
)

# Update session
session.update({"name": "Updated Session"})
```

### Conversations

```python
# Single-turn conversation
response = session.ask("What is machine learning?")
print(response.content)

# Streaming conversation
for answer in session.ask("Explain neural networks", stream=True):
    print(answer.content, end="", flush=True)

# Multi-turn conversation
session.ask("What are the applications?")
session.ask("Can you give examples?")
```

### Advanced Features

```python
# Custom variables
session.ask(
    "Explain quantum computing",
    style="academic",
    depth="advanced"
)

# Session management
assistant.delete_sessions(ids=["session_123"])

# Error handling
try:
    response = session.ask("Complex question")
except Exception as e:
    print(f"Error: {e}")
```

**Section sources**
- [chat.py](file://sdk/python/ragflow_sdk/modules/chat.py#L22-L88)
- [session.py](sdk/python/ragflow_sdk/modules/session.py#L21-L129)

## Examples

### Complete Chat Workflow

```python
from ragflow_sdk import RAGFlow

# Initialize client
rag = RAGFlow(api_key="your_key", base_url="http://localhost:9380")

# 1. Create chat assistant
assistant = rag.create_chat(
    name="Technical Support",
    dataset_ids=["kb_123"],
    llm={"model_name": "gpt-4", "temperature": 0.3}
)

# 2. Create session
session = assistant.create_session("First Support Case")

# 3. Engage in conversation
questions = [
    "How do I install the software?",
    "What are the system requirements?",
    "I'm getting an error message"
]

for question in questions:
    print(f"\nUser: {question}")
    print("Assistant: ", end="")
    
    for answer in session.ask(question, stream=True):
        print(answer.content, end="", flush=True)
    print()

# 4. Clean up
assistant.delete_sessions(ids=[session.id])
```

### Streaming Response Handling

```python
import requests
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry

def stream_chat(chat_id, question):
    """Stream chat responses with proper error handling."""
    session = requests.Session()
    
    retry_strategy = Retry(
        total=3,
        backoff_factor=1,
        status_forcelist=[429, 500, 502, 503, 504],
    )
    
    adapter = HTTPAdapter(max_retries=retry_strategy)
    session.mount("http://", adapter)
    session.mount("https://", adapter)
    
    try:
        response = session.post(
            f"http://localhost:9380/api/v1/chats/{chat_id}/completions",
            headers={
                "Authorization": f"Bearer {api_key}",
                "Content-Type": "application/json"
            },
            json={"question": question, "stream": True},
            stream=True
        )
        
        response.raise_for_status()
        
        buffer = ""
        for line in response.iter_lines():
            if line:
                line = line.decode('utf-8')
                if line.startswith('data: '):
                    try:
                        data = json.loads(line[6:])
                        if data.get("data") is True:
                            break
                        buffer += data["data"]["answer"]
                        yield buffer
                    except json.JSONDecodeError:
                        continue
                        
    except requests.RequestException as e:
        yield f"Error: {str(e)}"
```

### Batch Processing

```python
def process_multiple_questions(chat_id, questions):
    """Process multiple questions efficiently."""
    results = []
    
    for i, question in enumerate(questions):
        try:
            session = create_session(chat_id, f"Batch_{i}")
            answer = session.ask(question)
            
            results.append({
                "question": question,
                "answer": answer.content,
                "references": answer.reference,
                "session_id": session.id
            })
            
        except Exception as e:
            results.append({
                "question": question,
                "error": str(e)
            })
    
    return results
```

### Real-time Chat Application

```python
from flask import Flask, request, Response
from ragflow_sdk import RAGFlow

app = Flask(__name__)
rag = RAGFlow(api_key="your_key", base_url="http://localhost:9380")

@app.route('/chat', methods=['POST'])
def chat():
    """Real-time chat endpoint."""
    data = request.json
    question = data.get('question')
    chat_id = data.get('chat_id')
    
    def generate():
        try:
            session = create_session(chat_id)
            for answer in session.ask(question, stream=True):
                yield f"data: {json.dumps(answer.content)}\n\n"
            yield "data: [DONE]\n\n"
        except Exception as e:
            yield f"data: {{\"error\": \"{str(e)}\"}}\n\n"
    
    return Response(generate(), mimetype='text/event-stream')

if __name__ == '__main__':
    app.run(debug=True)
```

These examples demonstrate the full capabilities of the Chat API, from basic interactions to advanced streaming and batch processing scenarios.