# Scalekit OAuth Example

This example demonstrates how to use the Scalekit OAuth provider with FastMCP servers.

## Overview

The Scalekit OAuth provider enables authentication using Scalekit's OAuth 0 and OpenID Connect services. It provides enterprise-grade authentication with support for SSO connections and user management. FastMCP acts as a protected resource server that validates access tokens issued by Scalekit's authorization server.

## Setup

### 1. Scalekit Configuration

**Get Your Credentials**
1. Visit the [Scalekit Dashboard](https://app.scalekit.com/)
2. Go to **Developers** → **Settings**
3. Copy the following values:
   - Environment URL
   - Client ID  
   - Client Secret

**Register Your MCP Server**
1. Navigate to **MCP Servers** section
2. Click **Create New Server**
3. Fill in your MCP server details
4. Save and note the **Resource ID** (e.g., `res_123`)

### 2. Set Environment Variables

```bash
# Required Scalekit credentials
export SCALEKIT_ENVIRONMENT_URL="https://your-env.scalekit.com"
export SCALEKIT_CLIENT_ID="sk_123"
export SCALEKIT_RESOURCE_ID="res_456"
export MCP_URL="http://localhost:8000/mcp"
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

The server will start on `http://localhost:8000/mcp` with Scalekit OAuth authentication enabled.

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

- **Universal Authentication Support**: Works with any OAuth, OIDC, SAML connection and can even co-exist with your existing auth system
- **JWT Validation**: Validates tokens using Scalekit's public keys
- **Token Caching**: Reuses tokens across sessions
- **Resource Server Pattern**: Follows OAuth 2.1 resource server specifications
- **Error Handling**: Graceful handling of auth failures and token expiration

## Troubleshooting

### Common Issues

1. **"Token validation failed"**: Inspect the token on your server in debug mode and perform JWT decode to see if any value in the token mismatches with what you have saved in your environment variables
2. **MCP Inspector CORS issues**: If you are using mcp-inspector and face CORS issues, go to your Scalekit dashboard, add your MCP server URL under Authentication -> Redirect URLs -> Allowed callback URLs -> Add URL (add your MCP inspector URL here http://localhost:6274/)
3. **Browser doesn't open**: Check firewall settings for localhost

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
- Monitor Scalekit's Auth logs for unusual activity
- Set appropriate token expiry time
- Validate token scopes and claims in your application

## Next Steps

- Implement role-based access control
- Add custom scopes and claims validation
- Set up multi-organization support
- Get auth-visibility with Scalekit's Auth logs