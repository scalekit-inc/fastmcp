# Scalekit OAuth Example

This example demonstrates how to use the Scalekit OAuth provider with FastMCP servers.

## Overview

The Scalekit OAuth provider enables authentication using Scalekit's OAuth 2.0 and OpenID Connect services. It provides enterprise-grade authentication with support for SSO connections and user management. FastMCP acts as a protected resource server that validates access tokens issued by Scalekit's authorization server.

## Setup

### 1. Scalekit Configuration

1. **Create a Scalekit Environment**:
   - Go to [Scalekit Admin Portal](https://scalekit.com/admin)
   - Create a new environment or use an existing one
   - Note your environment URL (e.g., `https://your-env.scalekit.com`)

2. **Create an OAuth Application**:
   - In your Scalekit environment, navigate to Applications
   - Create a new OAuth application
   - Configure redirect URIs for your OAuth flow
   - For this example: `http://localhost:8000/auth/callback`
   - Copy your `Client ID`

3. **Create a Resource Server**:
   - Navigate to Resources section in Scalekit Admin
   - Create a new Resource with appropriate scopes
   - Note the `Resource ID`

4. **Set up SSO Connection** (optional for enterprise SSO):
   - Go to Connections section
   - Set up your SSO connection (SAML, OIDC, or OAuth provider)
   - Configure the connection for your organization

### 2. Set Environment Variables

```bash
# Required Scalekit credentials
export SCALEKIT_ENVIRONMENT_URL="https://your-env.scalekit.com"
export SCALEKIT_CLIENT_ID="sk_123"
export SCALEKIT_RESOURCE_ID="res_456"
export SCALEKIT_MCP_URL="http://localhost:8000/mcp"
```

### 3. Install Dependencies

```bash
cd /path/to/fastmcp
uv sync
```

## Running the Example

### Start the Server

```bash
# From this directory
uv run python server.py
```

The server will start on `http://localhost:8000` with Scalekit OAuth authentication enabled.

### Test with Client

In another terminal:

```bash
# From this directory
uv run python client.py
```

The client will:
1. Attempt to connect to the server
2. Detect that OAuth authentication is required
3. Open a browser for Scalekit authentication
4. Complete the OAuth flow and connect to the server
5. Demonstrate calling authenticated tools

## How It Works

### Authentication Flow

1. **Client Request**: Client attempts to connect to FastMCP server
2. **Auth Challenge**: Server responds with `401 Unauthorized` and `WWW-Authenticate` header
3. **OAuth Discovery**: Client discovers OAuth endpoints from server metadata
4. **Authorization**: Client redirects user to Scalekit for authentication
5. **Callback**: Scalekit redirects back with authorization code
6. **Token Exchange**: Client exchanges code for access token
7. **API Calls**: Client uses access token for authenticated MCP requests

### Server Components

- **ScalekitProvider**: Validates tokens using Scalekit's JWT verification
- **Protected Resources**: MCP tools and resources require valid Scalekit tokens
- **OAuth Metadata**: Server advertises Scalekit as authorization server

### Client Components

- **OAuth Client**: Handles browser-based OAuth flow
- **Token Storage**: Caches tokens for future use
- **Automatic Auth**: Transparently handles authentication

## Key Features

- **Enterprise SSO**: Works with any Scalekit SSO connection
- **JWT Validation**: Validates tokens using Scalekit's public keys
- **Token Caching**: Reuses tokens across sessions
- **Resource Server Pattern**: Follows OAuth 2.0 resource server specifications
- **Error Handling**: Graceful handling of auth failures and token expiration

## Troubleshooting

### Common Issues

1. **"Invalid client" error**: Check CLIENT_ID and RESOURCE_ID
2. **"Token validation failed"**: Check ENVIRONMENT_URL and token scope
3. **"Redirect URI mismatch"**: Ensure redirect URL matches Scalekit settings
4. **Browser doesn't open**: Check firewall settings for localhost

### Debug Mode

Enable debug logging:

```python
import logging
logging.basicConfig(level=logging.DEBUG)
```

### Token Inspection

Check cached tokens:

```bash
ls ~/.fastmcp/oauth-mcp-client-cache/
```

Clear token cache:

```python
from fastmcp.client.auth.oauth import FileTokenStorage
FileTokenStorage.clear_all()
```

## Security Notes

- Use HTTPS in production
- Rotate client secrets regularly
- Monitor Scalekit logs for unusual activity
- Set appropriate token expiration times
- Validate token scopes and claims in your application

## Next Steps

- Explore Scalekit Directory Sync for user provisioning
- Set up multi-organization support
- Implement role-based access control
- Add custom scopes and claims validation
- Integrate with Scalekit's audit logs