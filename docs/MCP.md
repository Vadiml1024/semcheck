# MCP (Model Context Protocol) Tools/Resources Documentation

## Overview

Semcheck supports MCP (Model Context Protocol) Tools/Resources pattern, which allows external MCP clients (like VS Code, Claude Desktop) to use semcheck as a tool for semantic code analysis. Instead of semcheck using its own LLM, the external client's LLM uses semcheck's analysis capabilities through the MCP protocol.

## Architecture

In this pattern:
- **External MCP Client**: VS Code, Claude Desktop, or other MCP-compatible clients
- **External LLM**: The client's LLM (e.g., Claude, GPT-4) processes the semantic analysis
- **Semcheck MCP Server**: Provides tools and resources for code analysis
- **No Direct LLM Usage**: Semcheck doesn't make direct LLM calls; it provides data and analysis capabilities

## Prerequisites

### For Process-based Connections (Recommended)
- **Semcheck Installation**: Ensure `semcheck` binary is installed and accessible in your PATH
- **Configuration File**: Create a `semcheck-mcp.yaml` configuration file with MCP enabled
- **MCP Client**: Install a compatible MCP client (VS Code extension, Claude Desktop, etc.)

### For TCP Connections
- **Running Server**: Start semcheck MCP server manually before connecting
- **Client Support**: Verify your MCP client supports TCP connections
- **Network Access**: Ensure client can reach the server address and port

## Configuration

### MCP Server Configuration

Enable MCP mode by adding the following to your `semcheck.yaml` file:

```yaml
version: "1.0"
# Note: provider and model are not used in MCP mode
# The external client's LLM will be used instead
provider: ollama  # This is ignored in MCP mode
model: llama3.2   # This is ignored in MCP mode
timeout: 30

# MCP Configuration
mcp:
  enabled: true
  address: localhost  # Address to bind the server to
  port: 8080         # Port to listen on

rules:
  - name: example-rule
    description: Example rule for semantic analysis
    enabled: true
    files:
      include:
        - "*.go"
      exclude:
        - "*_test.go"
    specs:
      - path: "specs/example.md"
    fail_on: "error"
```

## Usage

### Running MCP Server

To start semcheck in MCP server mode:

```bash
semcheck -config semcheck-mcp.yaml -mcp-server
```

This will:
1. Load the configuration
2. Start a TCP server that accepts MCP protocol requests
3. Expose tools and resources for external MCP clients
4. Allow external clients to use their LLM for semantic analysis

### Using with External MCP Clients

External MCP clients can connect to semcheck and use the provided tools. There are two main approaches:

#### Method 1: Process-based (Recommended)
The MCP client starts the semcheck process directly:

**VS Code with MCP Extension:**
```json
{
  "mcp.servers": {
    "semcheck": {
      "command": "semcheck",
      "args": ["-config", "semcheck-mcp.yaml", "-mcp-server"]
    }
  }
}
```

**Claude Desktop:**
```json
{
  "mcpServers": {
    "semcheck": {
      "command": "semcheck",
      "args": ["-config", "semcheck-mcp.yaml", "-mcp-server"]
    }
  }
}
```

#### Method 2: TCP Connection
Connect to an already running semcheck MCP server. Note that TCP connection support varies by MCP client implementation.

**Prerequisites:**
1. Start semcheck MCP server manually:
   ```bash
   semcheck -config semcheck-mcp.yaml -mcp-server
   ```

2. Configure your MCP client to connect to the running server (client-specific configuration required).

**Important Notes:**
- Not all MCP clients support direct TCP connections
- Process-based approach is more widely supported and recommended
- Refer to your MCP client's documentation for TCP connection syntax
- Ensure the semcheck binary is in your PATH for process-based connections

#### Client Compatibility

**VS Code MCP Extension:**
- Supports process-based connections
- Configuration goes in VS Code settings or `.vscode/settings.json`
- Refer to: [VS Code MCP documentation](https://github.com/modelcontextprotocol/vscode-mcp)

**Claude Desktop:**
- Supports process-based connections
- Configuration goes in Claude Desktop's MCP settings
- Refer to: [Claude Desktop MCP documentation](https://github.com/anthropics/claude-desktop-mcp)

**Other MCP Clients:**
- Check your client's documentation for supported connection methods
- Process-based connections are generally more compatible
- TCP connections may require client-specific configuration syntax

## MCP Tools

Semcheck provides the following MCP tools:

### 1. analyze_code
Analyzes code files against specifications for semantic issues.

**Parameters:**
- `files` (array): List of file paths to analyze
- `specs` (array, optional): List of specification file paths
- `rule_name` (string, optional): Specific rule name to apply

**Example:**
```json
{
  "name": "analyze_code",
  "arguments": {
    "files": ["main.go", "utils.go"],
    "rule_name": "example-rule"
  }
}
```

### 2. list_rules
Lists all available semantic checking rules.

**Example:**
```json
{
  "name": "list_rules",
  "arguments": {}
}
```

### 3. get_rule_details
Gets detailed information about a specific rule.

**Parameters:**
- `rule_name` (string): Name of the rule to get details for

**Example:**
```json
{
  "name": "get_rule_details",
  "arguments": {
    "rule_name": "example-rule"
  }
}
```

### 4. match_files
Matches files against configured rules and returns matching pairs.

**Parameters:**
- `files` (array): List of file paths to match

**Example:**
```json
{
  "name": "match_files",
  "arguments": {
    "files": ["main.go", "spec.md"]
  }
}
```

## MCP Resources

Semcheck provides the following MCP resources:

### 1. Configuration
- **URI**: `config://semcheck.yaml`
- **Description**: Current semcheck configuration
- **Type**: JSON

### 2. Specification Files
- **URI**: `spec://<path>`
- **Description**: Specification files for rules
- **Type**: Markdown

**Example:**
```
spec://specs/example.md
```

## MCP Protocol

### Tools List Request
```json
{
  "id": "1",
  "method": "tools/list",
  "params": {}
}
```

### Tools List Response
```json
{
  "id": "1",
  "result": {
    "tools": [
      {
        "name": "analyze_code",
        "description": "Analyze code files against specifications",
        "inputSchema": {
          "type": "object",
          "properties": {
            "files": {
              "type": "array",
              "items": {"type": "string"}
            }
          }
        }
      }
    ]
  }
}
```

### Tool Call Request
```json
{
  "id": "2",
  "method": "tools/call",
  "params": {
    "name": "analyze_code",
    "arguments": {
      "files": ["main.go"]
    }
  }
}
```

### Tool Call Response
```json
{
  "id": "2",
  "result": {
    "content": [
      {
        "type": "text",
        "text": "Analysis results in JSON format..."
      }
    ]
  }
}
```

### Resources List Request
```json
{
  "id": "3",
  "method": "resources/list",
  "params": {}
}
```

### Resources List Response
```json
{
  "id": "3",
  "result": {
    "resources": [
      {
        "uri": "config://semcheck.yaml",
        "name": "Semcheck Configuration",
        "description": "Current semcheck configuration"
      }
    ]
  }
}
```

### Resource Read Request
```json
{
  "id": "4",
  "method": "resources/read",
  "params": {
    "uri": "config://semcheck.yaml"
  }
}
```

### Resource Read Response
```json
{
  "id": "4",
  "result": {
    "contents": [
      {
        "type": "text",
        "text": "Configuration data..."
      }
    ]
  }
}
```

## Architecture

### Components

1. **MCP Server** (`internal/mcp/server.go`):
   - Accepts TCP connections
   - Handles MCP protocol requests
   - Manages concurrent connections
   - Supports tools/list, tools/call, resources/list, resources/read methods

2. **Tools/Resources Handler** (`internal/mcp/tools.go`):
   - Implements MCP tools for semantic analysis
   - Provides MCP resources (config, specifications)
   - Handles tool calls and resource reads
   - No direct LLM integration (external client provides LLM)

3. **Configuration**: Extended config system with MCP settings

### Tools/Resources Pattern

The MCP Tools/Resources pattern allows external clients to:

1. **List Tools**: Discover available semantic analysis tools
2. **Call Tools**: Execute semantic analysis functions
3. **List Resources**: Discover available configuration and specification files
4. **Read Resources**: Access configuration and specification content

The external client's LLM uses these tools and resources to perform semantic analysis with its own reasoning capabilities.

## Configuration Options

### MCP Section

- `enabled` (bool): Enable/disable MCP functionality
- `address` (string): Server bind address (default: "localhost")
- `port` (int): Server port (default: 8080)

### Provider Integration

In MCP mode:
- **Tools/Resources mode**: Provider and model settings are ignored
- **External LLM**: The connecting MCP client provides the LLM
- **No Direct API Calls**: Semcheck doesn't make direct LLM API calls

## Testing

### Unit Tests

Run MCP unit tests:

```bash
go test ./internal/mcp/...
```

### Integration Tests

Test MCP server startup and tool/resource functionality:

```bash
go test ./internal/mcp/... -v
```

### Manual Testing

1. Start MCP server:
   ```bash
   semcheck -config semcheck-mcp.yaml -mcp-server
   ```

2. Test with an MCP client (e.g., using netcat):
   ```bash
   echo '{"id":"1","method":"tools/list","params":{}}' | nc localhost 8080
   ```

3. Test tool call:
   ```bash
   echo '{"id":"2","method":"tools/call","params":{"name":"list_rules","arguments":{}}}' | nc localhost 8080
   ```

## Use Cases

### 1. IDE Integration

Connect VS Code or other IDEs to semcheck for real-time semantic analysis:

```json
{
  "mcp.servers": {
    "semcheck": {
      "command": "semcheck",
      "args": ["-config", "semcheck-mcp.yaml", "-mcp-server"]
    }
  }
}
```

### 2. Claude Desktop Integration

Use Claude Desktop with semcheck tools:

```json
{
  "mcpServers": {
    "semcheck": {
      "command": "semcheck",
      "args": ["-config", "semcheck-mcp.yaml", "-mcp-server"]
    }
  }
}
```

### 3. Custom MCP Clients

Create custom MCP clients that integrate with:
- CI/CD systems
- Code review tools
- Automated testing frameworks
- Custom development workflows

### 4. Multi-Language Support

External MCP clients can use their LLM to analyze code in multiple languages while leveraging semcheck's configuration and rule system.

## Example Usage

### External Client Workflow

1. **Connect**: External client connects to semcheck MCP server
2. **Discover Tools**: Client calls `tools/list` to see available analysis tools
3. **Read Resources**: Client calls `resources/read` to get specifications
4. **Analyze Code**: Client calls `analyze_code` tool with file paths
5. **Process Results**: Client's LLM processes the analysis results using its own reasoning

### Sample Tool Call Flow

```javascript
// External client code
const client = new MCPClient('localhost', 8080);

// List available tools
const tools = await client.call('tools/list', {});

// Get rule details
const ruleDetails = await client.call('tools/call', {
  name: 'get_rule_details',
  arguments: { rule_name: 'example-rule' }
});

// Analyze code
const results = await client.call('tools/call', {
  name: 'analyze_code',
  arguments: { files: ['main.go'] }
});

// The client's LLM then processes these results
```

## Error Handling

The MCP implementation includes comprehensive error handling:

- **Connection errors**: Proper TCP connection management
- **Protocol errors**: Structured error responses following MCP standard
- **Tool errors**: Detailed error messages for tool call failures
- **Resource errors**: Clear error messages for resource access issues
- **Input validation**: All tool parameters are validated

## Security Considerations

- **Network Security**: MCP servers bind to localhost by default
- **Authentication**: No built-in authentication (add firewall rules or proxy)
- **Input Validation**: All tool inputs are validated before processing
- **File Access**: Resource access is restricted to configured paths

## Troubleshooting

### Common Issues

1. **Connection Refused**:
   - Ensure MCP server is running (for TCP connections)
   - Check address and port configuration
   - Verify firewall settings

2. **Command Not Found**:
   - Ensure `semcheck` binary is in your PATH
   - Use absolute path to semcheck if needed
   - Verify semcheck is properly installed

3. **Configuration Errors**:
   - Check MCP client's configuration syntax
   - Ensure config file path is correct
   - Verify semcheck-mcp.yaml exists and is valid

4. **Unknown Tool/Resource**:
   - Verify tool/resource names using `tools/list` or `resources/list`
   - Check spelling and case sensitivity

5. **Invalid Parameters**:
   - Verify parameter types and required fields
   - Check tool input schema using `tools/list`

6. **File Not Found**:
   - Ensure file paths are correct and accessible
   - Check working directory and relative paths

### Debug Mode

Enable debug logging by setting environment variables:

```bash
export DEBUG=1
semcheck -config semcheck-mcp.yaml -mcp-server
```

## Future Enhancements

Potential future improvements:

1. **Authentication**: Add token-based authentication
2. **TLS Support**: Encrypt MCP connections
3. **Metrics**: Add prometheus metrics for monitoring
4. **Streaming**: Support streaming responses for large analyses
5. **Batch Operations**: Add batch processing tools
6. **Custom Tools**: Allow configuration of custom analysis tools