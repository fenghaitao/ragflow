# Authentication API

<cite>
**Referenced Files in This Document**
- [api/apps/auth/oauth.py](file://api/apps/auth/oauth.py)
- [api/apps/auth/oidc.py](file://api/apps/auth/oidc.py)
- [api/apps/auth/github.py](file://api/apps/auth/github.py)
- [api/apps/auth/__init__.py](file://api/apps/auth/__init__.py)
- [api/apps/auth/README.md](file://api/apps/auth/README.md)
- [api/apps/user_app.py](file://api/apps/user_app.py)
- [admin/server/auth.py](file://admin/server/auth.py)
- [api/apps/__init__.py](file://api/apps/__init__.py)
- [api/db/services/user_service.py](file://api/db/services/user_service.py)
- [api/db/db_models.py](file://api/db/db_models.py)
- [api/utils/api_utils.py](file://api/utils/api_utils.py)
- [api/apps/system_app.py](file://api/apps/system_app.py)
- [api/apps/api_app.py](file://api/apps/api_app.py)
</cite>

## Table of Contents
1. [Introduction](#introduction)
2. [Authentication Architecture](#authentication-architecture)
3. [OAuth 2.0 Authentication](#oauth-20-authentication)
4. [OpenID Connect (OIDC) Authentication](#openid-connect-oidc-authentication)
5. [GitHub OAuth Authentication](#github-oauth-authentication)
6. [Token-Based Authentication](#token-based-authentication)
7. [Session Management](#session-management)
8. [API Key Authentication](#api-key-authentication)
9. [Security Considerations](#security-considerations)
10. [Error Handling](#error-handling)
11. [Integration Patterns](#integration-patterns)
12. [Configuration Requirements](#configuration-requirements)
13. [Troubleshooting Guide](#troubleshooting-guide)

## Introduction

RAGFlow's authentication system provides comprehensive support for multiple authentication methods including OAuth 2.0, OpenID Connect (OIDC), and GitHub OAuth. The system implements token-based authentication with session management, API key authentication, and robust security measures to protect against common vulnerabilities.

The authentication system is built around a unified architecture that supports multiple identity providers while maintaining consistent user experience and security standards across all authentication methods.

## Authentication Architecture

RAGFlow's authentication system follows a modular architecture with clear separation of concerns:

```mermaid
graph TB
subgraph "Authentication Layer"
A[OAuth Client Factory] --> B[OAuth Client]
A --> C[OIDC Client]
A --> D[GitHub Client]
B --> E[Token Exchange]
C --> F[ID Token Validation]
D --> G[User Info Retrieval]
E --> H[User Registration/Login]
F --> H
G --> H
end
subgraph "Session Management"
H --> I[Access Token Generation]
I --> J[Session Storage]
J --> K[Token Validation]
end
subgraph "API Services"
K --> L[Authenticated Routes]
M[API Key Auth] --> L
N[Admin Auth] --> L
end
subgraph "Database"
O[User Service]
P[API Token Service]
Q[User-Tenant Relations]
end
H --> O
I --> P
L --> Q
```

**Diagram sources**
- [api/apps/auth/__init__.py](file://api/apps/auth/__init__.py#L22-L40)
- [api/apps/user_app.py](file://api/apps/user_app.py#L161-L266)
- [api/apps/__init__.py](file://api/apps/__init__.py#L108-L144)

**Section sources**
- [api/apps/auth/__init__.py](file://api/apps/auth/__init__.py#L1-L40)
- [api/apps/user_app.py](file://api/apps/user_app.py#L1-L50)

## OAuth 2.0 Authentication

### Overview

OAuth 2.0 authentication provides support for third-party identity providers through the standard OAuth 2.0 protocol. The system supports dynamic configuration of OAuth providers and automatic user registration.

### Configuration

```python
# OAuth 2.0 Configuration Example
oauth_config = {
    "type": "oauth2",
    "client_id": "your_client_id",
    "client_secret": "your_client_secret",
    "authorization_url": "https://provider.com/oauth/authorize",
    "token_url": "https://provider.com/oauth/token",
    "userinfo_url": "https://provider.com/oauth/userinfo",
    "redirect_uri": "https://your-app.com/v1/user/oauth/callback/<channel>",
    "scope": "openid profile email"
}
```

### Authentication Flow

```mermaid
sequenceDiagram
participant Client as "Client Application"
participant RAGFlow as "RAGFlow Server"
participant Provider as "OAuth Provider"
Client->>RAGFlow : GET /v1/user/login/{channel}
RAGFlow->>RAGFlow : Generate state parameter
RAGFlow->>RAGFlow : Store state in session
RAGFlow->>Provider : Redirect to authorization URL
Provider->>Client : Authorization prompt
Client->>Provider : User grants permission
Provider->>RAGFlow : Callback with authorization code
RAGFlow->>Provider : Exchange code for access token
Provider->>RAGFlow : Return access token
RAGFlow->>Provider : Fetch user information
Provider->>RAGFlow : Return user profile
RAGFlow->>RAGFlow : Register or login user
RAGFlow->>Client : Redirect with auth token
```

**Diagram sources**
- [api/apps/user_app.py](file://api/apps/user_app.py#L161-L266)
- [api/apps/auth/oauth.py](file://api/apps/auth/oauth.py#L48-L85)

### Endpoint Specifications

#### GET /v1/user/login/{channel}

Initiates OAuth 2.0 authentication flow.

**Parameters:**
- `channel` (path): Authentication channel identifier

**Response:**
- Redirects to OAuth provider's authorization URL

#### GET /v1/user/oauth/callback/{channel}

Handles OAuth callback and completes authentication.

**Parameters:**
- `channel` (path): Authentication channel identifier
- `code` (query): Authorization code from OAuth provider
- `state` (query): State parameter for CSRF protection

**Success Response:**
- Redirects to client application with authentication token

**Error Responses:**
- `invalid_state`: State parameter mismatch
- `missing_code`: Authorization code missing
- `token_failed`: Failed to exchange code for token
- `email_missing`: User email not available

**Section sources**
- [api/apps/user_app.py](file://api/apps/user_app.py#L161-L266)
- [api/apps/auth/oauth.py](file://api/apps/auth/oauth.py#L1-L152)

## OpenID Connect (OIDC) Authentication

### Overview

OIDC extends OAuth 2.0 with identity layer functionality, providing standardized token validation and user information retrieval.

### Configuration

```python
# OIDC Configuration Example
oidc_config = {
    "type": "oidc",
    "issuer": "https://provider.com/oidc",
    "client_id": "your_client_id",
    "client_secret": "your_client_secret",
    "redirect_uri": "https://your-app.com/v1/user/oauth/callback/<channel>"
}
```

### ID Token Validation

OIDC clients automatically validate ID tokens using JSON Web Key Sets (JWKS):

```mermaid
flowchart TD
A[Receive ID Token] --> B[Decode JWT Header]
B --> C[Extract Signing Algorithm]
C --> D[Fetch JWKS from Provider]
D --> E[Find Matching Signing Key]
E --> F[Verify Signature]
F --> G[Validate Claims]
G --> H[Extract User Information]
F --> |Invalid Signature| I[Reject Token]
G --> |Invalid Claims| I
I --> J[Authentication Failure]
```

**Diagram sources**
- [api/apps/auth/oidc.py](file://api/apps/auth/oidc.py#L60-L85)

### Endpoint Specifications

#### GET /v1/user/login/{channel}

Initiates OIDC authentication flow.

**Parameters:**
- `channel` (path): OIDC provider channel identifier

**Response:**
- Redirects to OIDC provider's authorization endpoint

#### GET /v1/user/oauth/callback/{channel}

Handles OIDC callback and validates ID tokens.

**Parameters:**
- `channel` (path): OIDC provider channel identifier
- `code` (query): Authorization code from OIDC provider
- `state` (query): State parameter for CSRF protection

**Success Response:**
- Redirects to client application with authentication token

**Error Responses:**
- `invalid_state`: State parameter mismatch
- `missing_code`: Authorization code missing
- `token_failed`: Failed to exchange code for token
- `email_missing`: User email not available
- `invalid_id_token`: ID token validation failed

**Section sources**
- [api/apps/auth/oidc.py](file://api/apps/auth/oidc.py#L1-L108)
- [api/apps/user_app.py](file://api/apps/user_app.py#L161-L266)

## GitHub OAuth Authentication

### Overview

GitHub OAuth provides seamless integration with GitHub accounts for user authentication and authorization.

### Configuration

```python
# GitHub OAuth Configuration Example
github_config = {
    "type": "github",
    "client_id": "your_github_client_id",
    "client_secret": "your_github_client_secret",
    "redirect_uri": "https://your-app.com/v1/user/oauth/callback/<channel>"
}
```

### GitHub-Specific Features

GitHub OAuth has special handling for user information retrieval:

```mermaid
flowchart TD
A[GitHub OAuth Callback] --> B[Exchange Code for Access Token]
B --> C[Fetch User Profile]
C --> D[Fetch Email Addresses]
D --> E[Select Primary Email]
E --> F[Normalize User Information]
F --> G[Register/Login User]
C --> |API Error| H[Handle Error]
D --> |API Error| H
H --> I[Authentication Failure]
```

**Diagram sources**
- [api/apps/auth/github.py](file://api/apps/auth/github.py#L35-L89)
- [api/apps/user_app.py](file://api/apps/user_app.py#L290-L354)

### Endpoint Specifications

#### GET /v1/user/login/github

Initiates GitHub OAuth authentication flow.

**Response:**
- Redirects to GitHub's authorization URL

#### GET /v1/user/github_callback

Handles GitHub OAuth callback (deprecated, use `/oauth/callback/{channel}` instead).

**Parameters:**
- `code` (query): Authorization code from GitHub

**Success Response:**
- Redirects to client application with authentication token

**Error Responses:**
- `user:email not in scope`: Required email scope not granted
- Various API errors from GitHub

**Section sources**
- [api/apps/auth/github.py](file://api/apps/auth/github.py#L1-L89)
- [api/apps/user_app.py](file://api/apps/user_app.py#L269-L354)

## Token-Based Authentication

### Access Token Management

RAGFlow uses UUID-based access tokens for authentication:

```mermaid
classDiagram
class UserService {
+query(access_token, status) User[]
+filter_by_id(user_id) User
+query_user(email, password) User
+save(**kwargs) User
+update_user_password(user_id, new_password) void
}
class User {
+string id
+string email
+string access_token
+string password
+boolean is_active
+timestamp create_time
+timestamp update_time
}
class APIToken {
+string tenant_id
+string token
+string beta
+timestamp create_time
+timestamp update_time
}
UserService --> User : manages
User --> APIToken : authenticates
```

**Diagram sources**
- [api/db/services/user_service.py](file://api/db/services/user_service.py#L33-L102)
- [api/db/db_models.py](file://api/db/db_models.py#L1-L200)

### Token Validation Process

```mermaid
flowchart TD
A[Incoming Request] --> B[Extract Authorization Header]
B --> C[Parse Access Token]
C --> D{Token Format Valid?}
D --> |No| E[Reject - Invalid Format]
D --> |Yes| F{Token Length >= 32?}
F --> |No| E
F --> |Yes| G{Token Starts with "INVALID_"?}
G --> |Yes| H[Reject - Invalidated Token]
G --> |No| I[Query User by Token]
I --> J{User Found?}
J --> |No| K[Reject - Invalid Token]
J --> |Yes| L{User Active?}
L --> |No| M[Reject - Inactive User]
L --> |Yes| N[Authenticate User]
```

**Diagram sources**
- [api/apps/__init__.py](file://api/apps/__init__.py#L108-L144)
- [api/db/services/user_service.py](file://api/db/services/user_service.py#L46-L66)

### Authentication Decorators

RAGFlow provides several authentication decorators:

#### @login_required

Protects routes requiring user authentication:

```python
@manager.route('/protected', methods=['GET'])
@login_required
async def protected_route():
    return get_json_result(data={'message': 'Authenticated'})
```

#### @token_required

Validates API tokens for service-to-service authentication:

```python
@manager.route('/api/service', methods=['POST'])
@token_required
async def service_route(tenant_id=None):
    return get_json_result(data={'tenant_id': tenant_id})
```

**Section sources**
- [api/apps/__init__.py](file://api/apps/__init__.py#L147-L176)
- [api/utils/api_utils.py](file://api/utils/api_utils.py#L250-L280)

## Session Management

### Session Configuration

RAGFlow uses Redis-based session storage for OAuth state management:

```python
# Session Configuration
app.config["SESSION_TYPE"] = "redis"
app.config["SESSION_REDIS"] = settings.decrypt_database_config(name="redis")
app.config["SECRET_KEY"] = settings.SECRET_KEY
```

### OAuth State Protection

OAuth flows use state parameters to prevent CSRF attacks:

```mermaid
sequenceDiagram
participant Client as "Client"
participant RAGFlow as "RAGFlow"
participant Provider as "OAuth Provider"
Client->>RAGFlow : GET /login/{channel}
RAGFlow->>RAGFlow : Generate random state
RAGFlow->>RAGFlow : Store state in session
RAGFlow->>Provider : Redirect with state parameter
Provider->>Client : Authorization prompt
Client->>Provider : User grants permission
Provider->>RAGFlow : Callback with code + state
RAGFlow->>RAGFlow : Compare state with session
alt State matches
RAGFlow->>RAGFlow : Continue authentication
else State mismatch
RAGFlow->>Client : Error - invalid_state
end
```

**Diagram sources**
- [api/apps/user_app.py](file://api/apps/user_app.py#L185-L189)

### Session Security

- State parameters are randomly generated using secure UUID
- Sessions expire after configured timeout
- CSRF protection through state parameter validation
- Secure cookie configuration for production deployments

**Section sources**
- [api/apps/__init__.py](file://api/apps/__init__.py#L89-L98)
- [api/apps/user_app.py](file://api/apps/user_app.py#L168-L189)

## API Key Authentication

### Overview

API keys provide programmatic access to RAGFlow services with tenant isolation.

### API Token Management

```mermaid
classDiagram
class APITokenService {
+used(token) void
+delete_by_tenant_id(tenant_id) void
+query(**kwargs) APIToken[]
+save(**kwargs) APIToken
}
class APIToken {
+string tenant_id
+string token
+string beta
+timestamp create_time
+timestamp update_time
+string dialog_id
}
APITokenService --> APIToken : manages
```

**Diagram sources**
- [api/db/services/api_service.py](file://api/db/services/api_service.py#L25-L40)

### API Key Usage

API keys are included in the Authorization header:

```
Authorization: Bearer YOUR_API_KEY
```

### API Key Endpoints

#### POST /v1/api/new_token

Creates a new API token for the authenticated user.

**Request Body:**
```json
{
    "name": "My API Key"
}
```

**Response:**
```json
{
    "code": 200,
    "message": "success",
    "data": {
        "tenant_id": "tenant_id",
        "token": "generated_token",
        "beta": "beta_token_part",
        "create_time": "timestamp"
    }
}
```

#### GET /v1/api/token_list

Lists all API tokens for the authenticated user.

**Response:**
```json
{
    "code": 200,
    "message": "success",
    "data": [
        {
            "token": "token1",
            "name": "Key 1",
            "create_time": "timestamp"
        }
    ]
}
```

#### DELETE /v1/api/rm

Deletes one or more API tokens.

**Request Body:**
```json
{
    "tokens": ["token1", "token2"],
    "tenant_id": "tenant_id"
}
```

**Section sources**
- [api/apps/api_app.py](file://api/apps/api_app.py#L26-L82)
- [api/apps/system_app.py](file://api/apps/system_app.py#L210-L291)

## Security Considerations

### Token Security

#### Access Token Validation

RAGFlow implements multiple layers of token validation:

```mermaid
flowchart TD
A[Token Received] --> B[Format Validation]
B --> C{UUID Format?}
C --> |No| D[Reject - Invalid Format]
C --> |Yes| E{Length >= 32?}
E --> |No| D
E --> |Yes| F{Not invalidated?}
F --> |No| G[Reject - Invalidated]
F --> |Yes| H[Database Lookup]
H --> I{User Exists?}
I --> |No| J[Reject - Not Found]
I --> |Yes| K{User Active?}
K --> |No| L[Reject - Inactive]
K --> |Yes| M[Accept]
```

**Diagram sources**
- [api/db/services/user_service.py](file://api/db/services/user_service.py#L46-L66)

#### Token Expiration and Refresh

- Access tokens are UUID-based and don't have explicit expiration
- Tokens are invalidated on logout by prefixing with "INVALID_"
- API tokens can be managed through the token lifecycle API
- Regular cleanup of expired tokens recommended

### OAuth Security Measures

#### State Parameter Protection

- Random state generation using secure UUID
- State stored in session with expiration
- Strict comparison between request and session state
- Immediate rejection on state mismatch

#### Scope Validation

- OAuth providers require explicit scope declarations
- GitHub requires "user:email" scope for email access
- OIDC providers validate scopes against requested permissions

#### CSRF Protection

- State parameter prevents cross-site request forgery
- Session-based state storage
- Automatic state cleanup after successful authentication

### Password Security

#### Password Hashing

```python
# Password hashing using Werkzeug security
from werkzeug.security import generate_password_hash, check_password_hash

# Store hashed passwords
hashed_password = generate_password_hash(raw_password)

# Verify passwords
if check_password_hash(hashed_password, raw_input):
    # Authentication successful
```

#### Password Policies

- Minimum password length enforcement
- Strong password recommendations
- No password reuse policies
- Regular password rotation encouraged

### Data Protection

#### Encryption

- HTTPS required for all authentication flows
- Secret keys encrypted in configuration
- Database connections use SSL/TLS
- Session cookies use secure attributes

#### Data Privacy

- Minimal user data collection
- Explicit consent for data sharing
- User data retention policies
- GDPR compliance features

**Section sources**
- [api/db/services/user_service.py](file://api/db/services/user_service.py#L46-L66)
- [api/apps/user_app.py](file://api/apps/user_app.py#L95-L137)

## Error Handling

### Authentication Errors

RAGFlow provides comprehensive error handling for authentication failures:

#### Common Error Responses

```json
{
    "code": 401,
    "message": "Unauthorized",
    "data": null
}
```

```json
{
    "code": 403,
    "message": "Forbidden",
    "data": null
}
```

```json
{
    "code": 400,
    "message": "Bad Request",
    "data": null
}
```

#### OAuth-Specific Errors

| Error Code | Description | Resolution |
|------------|-------------|------------|
| `invalid_state` | State parameter mismatch | Retry authentication flow |
| `missing_code` | Authorization code missing | Check OAuth provider configuration |
| `token_failed` | Failed to exchange code for token | Verify client credentials |
| `email_missing` | User email not available | Check required scopes |
| `user_inactive` | User account is inactive | Contact administrator |

#### Token Validation Errors

| Error Code | Description | Resolution |
|------------|-------------|------------|
| `empty_token` | Empty access token | Provide valid token |
| `invalid_format` | Invalid token format | Use UUID format |
| `short_token` | Token too short (< 32 chars) | Generate new token |
| `invalidated_token` | Token has been invalidated | Re-authenticate |
| `not_found` | Token not found in database | Check token validity |

#### API Key Errors

| Error Code | Description | Resolution |
|------------|-------------|------------|
| `missing_auth` | Authorization header missing | Include Bearer token |
| `invalid_auth` | API key is invalid | Check token format |
| `insufficient_perms` | Insufficient permissions | Verify token scope |

### Error Response Format

All authentication endpoints return consistent error responses:

```json
{
    "code": 401,
    "message": "Authentication failed: {specific_error}",
    "data": null
}
```

### Logging and Monitoring

- All authentication attempts logged with timestamps
- Failed authentication attempts tracked
- Security events flagged for monitoring
- Rate limiting applied to prevent brute force attacks

**Section sources**
- [api/apps/user_app.py](file://api/apps/user_app.py#L95-L137)
- [api/apps/auth/oauth.py](file://api/apps/auth/oauth.py#L65-L85)

## Integration Patterns

### Third-Party Identity Provider Integration

#### Dynamic Provider Configuration

RAGFlow supports dynamic configuration of multiple identity providers:

```python
# Multi-provider configuration
OAUTH_CONFIG = {
    "google": {
        "type": "oidc",
        "issuer": "https://accounts.google.com",
        "client_id": "google_client_id",
        "client_secret": "google_client_secret",
        "redirect_uri": "https://app.com/v1/user/oauth/callback/google"
    },
    "azure": {
        "type": "oidc", 
        "issuer": "https://login.microsoftonline.com/common/v2.0",
        "client_id": "azure_client_id",
        "client_secret": "azure_client_secret",
        "redirect_uri": "https://app.com/v1/user/oauth/callback/azure"
    },
    "github": {
        "type": "github",
        "client_id": "github_client_id",
        "client_secret": "github_client_secret",
        "redirect_uri": "https://app.com/v1/user/oauth/callback/github"
    }
}
```

#### Provider Discovery

OIDC providers support automatic metadata discovery:

```mermaid
sequenceDiagram
participant Client as "RAGFlow"
participant Provider as "OIDC Provider"
Client->>Provider : GET /.well-known/openid-configuration
Provider->>Client : Return OIDC metadata
Client->>Client : Extract endpoints
Client->>Provider : Authorization request
Provider->>Client : Authorization response
```

**Diagram sources**
- [api/apps/auth/oidc.py](file://api/apps/auth/oidc.py#L46-L57)

### Single Sign-On (SSO) Integration

#### Enterprise SSO Support

RAGFlow integrates with enterprise SSO solutions:

- Microsoft Azure AD
- Google Workspace
- Okta
- Auth0
- Custom SAML/WS-Fed providers

#### SSO Configuration

```python
# Enterprise SSO configuration
enterprise_config = {
    "type": "oidc",
    "issuer": "https://enterprise-idp.com/oidc",
    "client_id": "enterprise_client_id",
    "client_secret": "enterprise_client_secret",
    "redirect_uri": "https://app.com/v1/user/oauth/callback/enterprise",
    "scope": "openid profile email groups"
}
```

### Mobile and Desktop Applications

#### Native App Integration

RAGFlow supports native mobile and desktop applications:

- OAuth 2.0 PKCE (Proof Key for Code Exchange)
- Device-specific token management
- Silent token refresh capabilities
- Biometric authentication integration

#### SDK Integration

```python
# Python SDK example
from ragflow_sdk import RAGFlowClient

client = RAGFlowClient(api_key="your_api_key")
result = client.chat("Hello, RAGFlow!")
```

**Section sources**
- [api/apps/auth/__init__.py](file://api/apps/auth/__init__.py#L29-L40)
- [api/apps/auth/README.md](file://api/apps/auth/README.md#L1-L64)

## Configuration Requirements

### Environment Variables

#### Basic Configuration

```bash
# Database Configuration
DATABASE_URL=mysql://user:pass@host:port/db
REDIS_URL=redis://localhost:6379

# Security Configuration  
SECRET_KEY=your_secret_key_here
JWT_SECRET=jwt_secret_key

# OAuth Configuration
OAUTH_GOOGLE_CLIENT_ID=google_client_id
OAUTH_GOOGLE_CLIENT_SECRET=google_client_secret
OAUTH_AZURE_CLIENT_ID=azure_client_id
OAUTH_AZURE_CLIENT_SECRET=azure_client_secret
GITHUB_CLIENT_ID=github_client_id
GITHUB_CLIENT_SECRET=github_client_secret
```

#### OAuth Provider Configuration

```yaml
# OAuth Configuration (service_conf.yaml)
oauth:
  google:
    type: oidc
    issuer: https://accounts.google.com
    client_id: ${OAUTH_GOOGLE_CLIENT_ID}
    client_secret: ${OAUTH_GOOGLE_CLIENT_SECRET}
    redirect_uri: https://app.com/v1/user/oauth/callback/google
  
  azure:
    type: oidc
    issuer: https://login.microsoftonline.com/common/v2.0
    client_id: ${OAUTH_AZURE_CLIENT_ID}
    client_secret: ${OAUTH_AZURE_CLIENT_SECRET}
    redirect_uri: https://app.com/v1/user/oauth/callback/azure
  
  github:
    type: github
    client_id: ${GITHUB_CLIENT_ID}
    client_secret: ${GITHUB_CLIENT_SECRET}
    redirect_uri: https://app.com/v1/user/oauth/callback/github
```

### Database Schema Requirements

#### User Table Structure

```sql
CREATE TABLE users (
    id VARCHAR(32) PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password VARCHAR(255),
    nickname VARCHAR(255),
    avatar TEXT,
    access_token VARCHAR(32),
    is_superuser BOOLEAN DEFAULT FALSE,
    is_active BOOLEAN DEFAULT TRUE,
    login_channel VARCHAR(50),
    last_login_time DATETIME,
    create_time BIGINT,
    update_time BIGINT,
    status VARCHAR(10) DEFAULT 'VALID'
);
```

#### API Token Table Structure

```sql
CREATE TABLE api_tokens (
    tenant_id VARCHAR(32) NOT NULL,
    token VARCHAR(255) NOT NULL,
    beta VARCHAR(32),
    create_time BIGINT,
    update_time BIGINT,
    dialog_id VARCHAR(32),
    PRIMARY KEY (tenant_id, token)
);
```

### Deployment Considerations

#### Production Configuration

- HTTPS required for all authentication endpoints
- Secure cookie configuration
- Proper CORS headers
- Rate limiting enabled
- Session timeout configured
- Database connection pooling

#### Scaling Considerations

- Redis cluster for session storage
- Database read replicas
- CDN for static assets
- Load balancer configuration
- Health checks for authentication services

**Section sources**
- [api/apps/auth/README.md](file://api/apps/auth/README.md#L17-L45)
- [api/db/db_models.py](file://api/db/db_models.py#L1-L200)

## Troubleshooting Guide

### Common Issues and Solutions

#### OAuth Authentication Failures

**Issue: "invalid_state" error**
- **Cause**: State parameter mismatch between request and session
- **Solution**: Clear browser cookies and retry authentication
- **Prevention**: Ensure session storage is working correctly

**Issue: "missing_code" error**
- **Cause**: OAuth provider didn't return authorization code
- **Solution**: Check OAuth provider configuration and redirect URI
- **Prevention**: Verify redirect URI matches provider settings

**Issue: "token_failed" error**
- **Cause**: Failed to exchange authorization code for access token
- **Solution**: Verify client credentials and network connectivity
- **Prevention**: Test OAuth provider connectivity regularly

#### Token Validation Issues

**Issue: "empty_token" error**
- **Cause**: Missing or empty Authorization header
- **Solution**: Include proper Bearer token in request headers
- **Prevention**: Implement proper token handling in client applications

**Issue: "invalid_format" error**
- **Cause**: Token doesn't match UUID format
- **Solution**: Regenerate token or check token format
- **Prevention**: Use UUID generation libraries consistently

**Issue: "invalidated_token" error**
- **Cause**: Token has been invalidated (logout occurred)
- **Solution**: Re-authenticate to obtain new token
- **Prevention**: Implement token refresh mechanisms

#### Database Connection Issues

**Issue: "User not found" errors**
- **Cause**: Database connection problems or corrupted data
- **Solution**: Check database connectivity and user records
- **Prevention**: Implement database health checks

**Issue: "Duplicate email" errors**
- **Cause**: Multiple users with same email address
- **Solution**: Clean up duplicate records or merge accounts
- **Prevention**: Implement email uniqueness validation

### Debugging Tools

#### Logging Configuration

Enable detailed logging for authentication issues:

```python
import logging

logging.basicConfig(level=logging.DEBUG)
logger = logging.getLogger('ragflow.auth')
```

#### Health Checks

Monitor authentication service health:

```python
# Health check endpoint
@manager.route('/health/auth', methods=['GET'])
async def auth_health():
    try:
        # Test database connectivity
        user = UserService.query(id='test')
        return get_json_result(data={'status': 'healthy'})
    except Exception as e:
        return get_json_result(data={'status': 'unhealthy', 'error': str(e)})
```

#### Monitoring Metrics

Track authentication metrics:

- Authentication success/failure rates
- Token validation performance
- OAuth provider response times
- Session creation/deletion rates

### Support Resources

#### Documentation Links

- OAuth 2.0 Specification: [https://tools.ietf.org/html/rfc6749](https://tools.ietf.org/html/rfc6749)
- OpenID Connect Specification: [https://openid.net/connect/](https://openid.net/connect/)
- GitHub OAuth Documentation: [https://docs.github.com/en/developers/apps/building-oauth-apps](https://docs.github.com/en/developers/apps/building-oauth-apps)

#### Community Support

- GitHub Issues: Report bugs and feature requests
- Documentation: Comprehensive guides and examples
- API Reference: Complete endpoint documentation
- Examples: Sample integrations and use cases

**Section sources**
- [api/apps/user_app.py](file://api/apps/user_app.py#L95-L137)
- [api/db/services/user_service.py](file://api/db/services/user_service.py#L46-L66)