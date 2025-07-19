# MCP Server - Modular Model Context Protocol Implementation

[![TypeScript](https://img.shields.io/badge/TypeScript-5.8.3-blue.svg)](https://www.typescriptlang.org/)
[![Node.js](https://img.shields.io/badge/Node.js-20+-green.svg)](https://nodejs.org/)
[![Express](https://img.shields.io/badge/Express-5.1.0-lightgrey.svg)](https://expressjs.com/)
[![MCP SDK](https://img.shields.io/badge/MCP_SDK-1.16.0-orange.svg)](https://modelcontextprotocol.io/)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

A modular, extensible implementation of the [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) server built with TypeScript, Express.js, and a clean architectural design. This project provides a solid foundation for building MCP servers with custom tools and resources.

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [Architecture](#architecture)
- [Usage Examples](#usage-examples)
- [API Documentation](#api-documentation)
- [Development](#development)
- [Configuration](#configuration)
- [Troubleshooting](#troubleshooting)
- [Deployment](#deployment)
- [Contributing](#contributing)
- [Resources](#resources)

## Overview

The Model Context Protocol (MCP) is an open protocol that enables seamless integration between LLM applications and external data sources and tools. This implementation provides:

- **Modular Architecture**: Clean separation of concerns with dependency injection
- **HTTP Transport**: RESTful API with session management
- **Extensible Design**: Easy-to-use base classes for tools and resources
- **Type Safety**: Full TypeScript support with comprehensive type definitions
- **Production Ready**: Built-in error handling, logging, and validation
- **Developer Experience**: Hot reload, linting, and formatting tools

### Key Features

- ✅ **Tools**: Custom functions that AI models can execute
- ✅ **Resources**: Context and data sources for AI models
- ✅ **HTTP Transport**: Standard HTTP/JSON API endpoints
- ✅ **Session Management**: Stateful connections with automatic cleanup
- ✅ **Registry Pattern**: Dynamic registration of tools and resources
- ✅ **Configuration**: Flexible server configuration options
- ✅ **Validation**: Input validation with Zod schemas
- ✅ **Error Handling**: Comprehensive error handling and reporting
- ✅ **Logging**: Structured logging for debugging and monitoring

## Prerequisites

Before you begin, ensure you have the following installed on your system:

- **Node.js**: Version 20 or higher
- **Package Manager**: npm, yarn, or pnpm (pnpm recommended)
- **TypeScript**: Knowledge of TypeScript is recommended
- **Git**: For version control and cloning the repository

### System Requirements

- **OS**: Linux, macOS, or Windows
- **Memory**: Minimum 512MB RAM
- **Storage**: At least 100MB free space
- **Network**: Internet connection for installing dependencies

## Installation

### 1. Clone the Repository

```bash
git clone <repository-url>
cd generic-mcp
```

### 2. Install Dependencies

Using pnpm (recommended):
```bash
pnpm install
```

Using npm:
```bash
npm install
```

Using yarn:
```bash
yarn install
```

### 3. Build the Project

```bash
pnpm build
```

### 4. Verify Installation

```bash
pnpm check
```

This command runs type checking and linting to ensure everything is set up correctly.

## Quick Start

### 1. Start the Development Server

```bash
pnpm dev
```

The server will start on `http://localhost:3000` by default.

### 2. Test the Health Endpoint

```bash
curl http://localhost:3000/health
```

You should see a response like:
```json
{
  "status": "healthy",
  "name": "mcp-server",
  "version": "1.0.0",
  "timestamp": "2025-07-19T12:00:00.000Z",
  "sessions": 0
}
```

### 3. Start an MCP Session

```bash
curl -X POST http://localhost:3000/mcp \
    -H "Content-Type: application/json" \
    -H "Accept: application/json, text/event-stream" \
    -d '{
        "jsonrpc": "2.0",
        "id": 1,
        "method": "initialize",
        "params": {
            "protocolVersion": "2024-11-05",
            "capabilities": {},
            "clientInfo": {
                "name": "test-client",
                "version": "1.0.0"
            }
        }
    }'
```

### 4. List Available Tools

```bash
curl -X POST http://localhost:3000/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -H "MCP-Session-ID: <session-id-from-initialize>" \
  -d '{
    "jsonrpc": "2.0",
    "id": 2,
    "method": "tools/list"
  }'
```

## Architecture

### Modular Design

The project follows a modular architecture with clear separation of concerns:

```
src/
├── app.ts                    # Main application setup
├── index.ts                  # Entry point
├── config/
│   └── server.config.ts      # Server configuration
├── http/
│   ├── middleware/           # Express middleware
│   ├── routes/              # HTTP route handlers
│   └── utils/               # HTTP utilities
├── resources/
│   ├── base-resource.ts     # Base class for resources
│   ├── registry.ts          # Resource registry
│   └── examples/            # Example implementations
├── server/
│   └── mcp-server.factory.ts # MCP server factory
├── tools/
│   ├── base-tool.ts         # Base class for tools
│   ├── registry.ts          # Tool registry
│   └── examples/            # Example implementations
├── transport/
│   └── session-manager.ts   # Session management
├── types/
│   └── index.ts             # Type definitions
└── utils/
    ├── errors.ts            # Error handling
    └── logger.ts            # Logging utilities
```

### Core Components

#### 1. **Application Layer** (`app.ts`)
The main application class that orchestrates all components:
- Express.js setup and middleware configuration
- Dependency injection and service registration
- Route configuration and error handling
- Graceful shutdown handling

#### 2. **Tools System** (`tools/`)
- **BaseTool**: Abstract base class for implementing custom tools
- **ToolRegistry**: Manages tool registration and discovery
- **Examples**: Reference implementations (AddTool)

#### 3. **Resources System** (`resources/`)
- **BaseResource**: Abstract base class for implementing custom resources
- **ResourceRegistry**: Manages resource registration and discovery
- **Examples**: Reference implementations (GreetingResource)

#### 4. **Transport Layer** (`transport/`)
- **SessionTransportManager**: Manages HTTP sessions and MCP connections
- **Session lifecycle**: Creation, management, and cleanup
- **Protocol handling**: JSON-RPC message processing

#### 5. **HTTP Layer** (`http/`)
- **Middleware**: Request logging, CORS, validation
- **Routes**: MCP endpoint handlers
- **Utilities**: Request/response helpers

### Dependency Injection

The application uses constructor-based dependency injection:

```typescript
interface ServerDependencies {
  toolRegistry: ToolRegistry;
  resourceRegistry: ResourceRegistry;
  sessionManager: SessionTransportManager;
}
```

This pattern enables:
- **Testability**: Easy mocking and unit testing
- **Flexibility**: Runtime configuration of dependencies
- **Maintainability**: Clear dependency relationships

### Registry Pattern

Both tools and resources use a registry pattern for dynamic registration:

```typescript
// Tool registration
const toolRegistry = new ToolRegistryImpl();
toolRegistry.registerToolClass(new AddTool());

// Resource registration
const resourceRegistry = new ResourceRegistryImpl();
resourceRegistry.registerResourceClass(new GreetingResource());
```

Benefits:
- **Extensibility**: Add new tools/resources without modifying core code
- **Discovery**: Automatic enumeration of available capabilities
- **Validation**: Type-safe registration with runtime checks

## Usage Examples

### Creating a Custom Tool

Tools are functions that AI models can execute. Here's how to create a custom tool:

#### 1. Create the Tool Class

```typescript
// src/tools/examples/weather-tool.ts
import { BaseTool } from "@tools/base-tool.js";
import { ValidationError } from "@types";
import { z } from "zod";

export class WeatherTool extends BaseTool {
  readonly name = "get_weather";
  readonly title = "Weather Information";
  readonly description = "Get current weather for a location";
  readonly inputSchema = {
    location: z.string().describe("City name or coordinates"),
    units: z.enum(["celsius", "fahrenheit"]).optional().describe("Temperature units")
  };

  async execute(params: Record<string, unknown>): Promise<{
    content: Array<{ type: "text"; text: string }>;
  }> {
    this.validateParams(params);

    const { location, units = "celsius" } = params as {
      location: string;
      units?: "celsius" | "fahrenheit";
    };

    // Validate location parameter
    if (!location || location.trim().length === 0) {
      throw new ValidationError("Location parameter is required");
    }

    // Your weather API integration here
    const weatherData = await this.fetchWeatherData(location, units);
    
    const result = `Weather in ${location}: ${weatherData.temperature}°${units === "celsius" ? "C" : "F"}, ${weatherData.description}`;
    
    return this.createTextResponse(result);
  }

  private async fetchWeatherData(location: string, units: string) {
    // Implement your weather API call here
    // This is a mock implementation
    return {
      temperature: 22,
      description: "Sunny"
    };
  }
}
```

#### 2. Register the Tool

```typescript
// In src/app.ts, modify the createDependencies method:
private createDependencies(): ServerDependencies {
  const toolRegistry = new ToolRegistryImpl();
  const resourceRegistry = new ResourceRegistryImpl();

  // Register default tools
  toolRegistry.registerToolClass(new AddTool());
  toolRegistry.registerToolClass(new WeatherTool()); // Add your tool

  // Register default resources
  resourceRegistry.registerResourceClass(new GreetingResource());

  logger.info(
    `Registered ${toolRegistry.getCount()} tools and ${resourceRegistry.getCount()} resources`,
  );

  return {
    toolRegistry,
    resourceRegistry,
    sessionManager: this.sessionManager,
  };
}
```

### Creating a Custom Resource

Resources provide context and data to AI models. Here's how to create a custom resource:

#### 1. Create the Resource Class

```typescript
// src/resources/examples/database-resource.ts
import { BaseResource } from "@resources/base-resource.js";
import { ValidationError } from "@types";

export class DatabaseResource extends BaseResource {
  readonly name = "database";
  readonly title = "Database Query Resource";
  readonly description = "Execute database queries";
  readonly uriTemplate = "database://{table}/{id?}";
  readonly mimeType = "application/json";

  async read(
    uri: URL,
    params: Record<string, string>,
  ): Promise<{
    contents: Array<{ uri: string; text: string }>;
  }> {
    this.validateParams(params, ["table"]);

    const { table, id } = params;

    // Basic validation
    if (!table || table.trim().length === 0) {
      throw new ValidationError("Table parameter is required");
    }

    // Sanitize table name (basic SQL injection prevention)
    const sanitizedTable = table.replace(/[^a-zA-Z0-9_]/g, "");
    
    let result: any;
    
    if (id) {
      // Fetch specific record
      result = await this.fetchRecord(sanitizedTable, id);
    } else {
      // Fetch all records
      result = await this.fetchTable(sanitizedTable);
    }

    return this.createContentResponse(
      uri.href,
      JSON.stringify(result, null, 2)
    );
  }

  private async fetchRecord(table: string, id: string) {
    // Implement your database query here
    // This is a mock implementation
    return {
      id: id,
      table: table,
      data: { name: "Sample Record" }
    };
  }

  private async fetchTable(table: string) {
    // Implement your database query here
    // This is a mock implementation
    return [
      { id: "1", name: "Record 1" },
      { id: "2", name: "Record 2" }
    ];
  }
}
```

#### 2. Register the Resource

```typescript
// In src/app.ts, modify the createDependencies method:
private createDependencies(): ServerDependencies {
  const toolRegistry = new ToolRegistryImpl();
  const resourceRegistry = new ResourceRegistryImpl();

  // Register default tools
  toolRegistry.registerToolClass(new AddTool());

  // Register default resources
  resourceRegistry.registerResourceClass(new GreetingResource());
  resourceRegistry.registerResourceClass(new DatabaseResource()); // Add your resource

  logger.info(
    `Registered ${toolRegistry.getCount()} tools and ${resourceRegistry.getCount()} resources`,
  );

  return {
    toolRegistry,
    resourceRegistry,
    sessionManager: this.sessionManager,
  };
}
```

### Tool and Resource Registration Process

The registration process follows these steps:

1. **Instantiation**: Create instances of your tool/resource classes
2. **Registration**: Add them to the appropriate registry
3. **Discovery**: The MCP server automatically exposes them via the protocol
4. **Validation**: Input parameters are validated using Zod schemas
5. **Execution**: Tools are executed or resources are read based on client requests

#### Dynamic Registration

You can also register tools and resources dynamically at runtime:

```typescript
// During application startup or based on configuration
const toolRegistry = app.getDependencies().toolRegistry;
const resourceRegistry = app.getDependencies().resourceRegistry;

// Conditionally register tools based on environment
if (process.env.ENABLE_WEATHER_TOOL === 'true') {
  toolRegistry.registerToolClass(new WeatherTool());
}

// Register resources based on database availability
if (isDatabaseAvailable()) {
  resourceRegistry.registerResourceClass(new DatabaseResource());
}
```

## API Documentation

The MCP server exposes the following HTTP endpoints:

### Health Check

**GET** `/health`

Returns server health status and basic information.

**Response:**
```json
{
  "status": "healthy",
  "name": "mcp-server",
  "version": "1.0.0",
  "timestamp": "2025-07-19T12:00:00.000Z",
  "sessions": 0
}
```

### MCP Protocol Endpoints

All MCP protocol communication happens through these endpoints:

#### Initialize Session

**POST** `/mcp`

Initialize a new MCP session.

**Request:**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "initialize",
  "params": {
    "protocolVersion": "2024-11-05",
    "capabilities": {},
    "clientInfo": {
      "name": "client-name",
      "version": "1.0.0"
    }
  }
}
```

**Response:**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "protocolVersion": "2024-11-05",
    "capabilities": {
      "tools": {},
      "resources": {}
    },
    "serverInfo": {
      "name": "mcp-server",
      "version": "1.0.0"
    }
  }
}
```

#### List Tools

**POST** `/mcp`

List available tools in the current session.

**Headers:**
- `X-MCP-Session-ID`: Session ID from initialize response

**Request:**
```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "method": "tools/list"
}
```

**Response:**
```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "result": {
    "tools": [
      {
        "name": "add",
        "description": "Add two numbers",
        "inputSchema": {
          "type": "object",
          "properties": {
            "a": { "type": "string", "description": "First number" },
            "b": { "type": "string", "description": "Second number" }
          },
          "required": ["a", "b"]
        }
      }
    ]
  }
}
```

#### Call Tool

**POST** `/mcp`

Execute a specific tool.

**Headers:**
- `X-MCP-Session-ID`: Session ID from initialize response

**Request:**
```json
{
  "jsonrpc": "2.0",
  "id": 3,
  "method": "tools/call",
  "params": {
    "name": "add",
    "arguments": {
      "a": "5",
      "b": "3"
    }
  }
}
```

**Response:**
```json
{
  "jsonrpc": "2.0",
  "id": 3,
  "result": {
    "content": [
      {
        "type": "text",
        "text": "8"
      }
    ]
  }
}
```

#### List Resources

**POST** `/mcp`

List available resources in the current session.

**Headers:**
- `X-MCP-Session-ID`: Session ID from initialize response

**Request:**
```json
{
  "jsonrpc": "2.0",
  "id": 4,
  "method": "resources/list"
}
```

**Response:**
```json
{
  "jsonrpc": "2.0",
  "id": 4,
  "result": {
    "resources": [
      {
        "name": "greeting",
        "description": "Dynamic greeting generator",
        "uri": "greeting://{name}",
        "mimeType": "text/plain"
      }
    ]
  }
}
```

#### Read Resource

**POST** `/mcp`

Read a specific resource.

**Headers:**
- `X-MCP-Session-ID`: Session ID from initialize response

**Request:**
```json
{
  "jsonrpc": "2.0",
  "id": 5,
  "method": "resources/read",
  "params": {
    "uri": "greeting://Alice"
  }
}
```

**Response:**
```json
{
  "jsonrpc": "2.0",
  "id": 5,
  "result": {
    "contents": [
      {
        "uri": "greeting://Alice",
        "mimeType": "text/plain",
        "text": "Hello, Alice!"
      }
    ]
  }
}
```

#### Close Session

**DELETE** `/mcp`

Close an MCP session and clean up resources.

**Headers:**
- `X-MCP-Session-ID`: Session ID to close

**Response:**
```json
{
  "jsonrpc": "2.0",
  "result": {
    "message": "Session closed successfully"
  }
}
```

### Error Responses

The server returns standard JSON-RPC error responses:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "error": {
    "code": -32600,
    "message": "Invalid Request",
    "data": "Additional error details"
  }
}
```

Common error codes:
- `-32700`: Parse error
- `-32600`: Invalid Request
- `-32601`: Method not found
- `-32602`: Invalid params
- `-32603`: Internal error

## Development

### Available Scripts

The project includes several npm/pnpm scripts for development:

```bash
# Development
pnpm dev          # Start development server with hot reload
pnpm build        # Build the project for production
pnpm start        # Start the production server

# Code Quality
pnpm check        # Run type checking and linting
pnpm lint         # Run Biome linting
pnpm format       # Format code with Biome
pnpm type-check   # Run TypeScript type checking

# Git Hooks
pnpm prepare      # Install git hooks (Husky)
```

### Development Environment Setup

1. **Install Dependencies**
   ```bash
   pnpm install
   ```

2. **Set Up Git Hooks**
   ```bash
   pnpm prepare
   ```

3. **Start Development Server**
   ```bash
   pnpm dev
   ```

4. **Run Quality Checks**
   ```bash
   pnpm check
   ```

### Code Quality Tools

The project uses several tools to maintain code quality:

#### Biome
[Biome](https://biomejs.dev/) provides fast linting and formatting:
- **Configuration**: `biome.json`
- **Features**: ESLint-like linting, Prettier-like formatting
- **Speed**: Extremely fast performance
- **Integration**: VS Code extension available

#### TypeScript
Full TypeScript support with strict type checking:
- **Configuration**: `tsconfig.json`
- **Path Aliases**: Convenient imports (e.g., `@tools/*`, `@resources/*`)
- **Build Output**: `dist/` directory with declaration files
- **Source Maps**: Enabled for debugging

#### Commitlint
Enforces conventional commit messages:
- **Configuration**: `commitlint.config.js`
- **Format**: `type(scope): description`
- **Examples**: 
  - `feat: add weather tool`
  - `fix: resolve session cleanup issue`
  - `docs: update README installation section`

#### Husky
Git hooks for automated quality checks:
- **Pre-commit**: Runs linting and formatting
- **Commit-msg**: Validates commit message format

### Testing and Validation

#### Running Tests

The project uses the unified `check` command for validation:

```bash
pnpm check
```

This command:
1. Runs TypeScript type checking (`tsc --noEmit`)
2. Executes Biome linting on the `src` directory
3. Reports any errors or warnings

#### Manual Testing

For manual testing of the MCP server:

1. **Start the server**:
   ```bash
   pnpm dev
   ```

2. **Test health endpoint**:
   ```bash
   curl http://localhost:3000/health
   ```

3. **Test MCP session**:
   ```bash
   # Initialize session
   curl -X POST http://localhost:3000/mcp \
     -H "Content-Type: application/json" \
     -d '{
       "jsonrpc": "2.0",
       "id": 1,
       "method": "initialize",
       "params": {
         "protocolVersion": "2024-11-05",
         "capabilities": {},
         "clientInfo": {"name": "test", "version": "1.0.0"}
       }
     }'
   ```

#### Debugging

The project includes comprehensive logging:

```typescript
import { logger } from "@utils/logger.js";

// Log levels: debug, info, warn, error
logger.info("Server started successfully");
logger.error("Failed to process request", { error });
```

To enable debug logging:
```bash
DEBUG=* pnpm dev
```

## Configuration

### Server Configuration

The server can be configured through the `ServerConfig` interface:

```typescript
// src/config/server.config.ts
export interface ServerConfig {
  name: string;                              // Server name for MCP identification
  version: string;                           // Server version
  port: number;                             // HTTP port to listen on
  enableDnsRebindingProtection: boolean;    // DNS rebinding protection
  allowedHosts: string[];                   // Allowed hosts for DNS protection
  sessionIdGenerator: () => string;         // Session ID generator function
}
```

### Default Configuration

```typescript
export const defaultServerConfig: ServerConfig = {
  name: "mcp-server",
  version: "1.0.0",
  port: Number(process.env.PORT) || 3000,
  enableDnsRebindingProtection: false,
  allowedHosts: ["127.0.0.1"],
  sessionIdGenerator: () => randomUUID(),
};
```

### Environment Variables

You can customize the configuration using environment variables:

```bash
# Server configuration
PORT=8080                                 # Server port (default: 3000)

# Development
DEBUG=*                                   # Enable debug logging
NODE_ENV=production                       # Environment mode
```

### Custom Configuration

To use custom configuration:

```typescript
// Custom configuration
const customConfig: ServerConfig = {
  name: "my-mcp-server",
  version: "2.0.0",
  port: 8080,
  enableDnsRebindingProtection: true,
  allowedHosts: ["localhost", "127.0.0.1", "example.com"],
  sessionIdGenerator: () => `session-${Date.now()}`,
};

// Create application with custom config
const app = new McpApplication(customConfig);
```

### Path Aliases

The project uses TypeScript path aliases for clean imports:

```typescript
// Instead of relative imports
import { BaseTool } from "../../tools/base-tool.js";

// Use clean aliases
import { BaseTool } from "@tools/base-tool.js";
```

Available aliases:
- `@config/*` → `./src/config/*`
- `@http/*` → `./src/http/*`
- `@resources/*` → `./src/resources/*`
- `@server/*` → `./src/server/*`
- `@tools/*` → `./src/tools/*`
- `@transport/*` → `./src/transport/*`
- `@types/*` → `./src/types/*`
- `@types` → `./src/types/index.ts`
- `@utils/*` → `./src/utils/*`

## Troubleshooting

### Common Issues

#### 1. Port Already in Use

**Error**: `EADDRINUSE: address already in use :::3000`

**Solutions**:
- Change the port: `PORT=8080 pnpm dev`
- Kill the process using the port: `kill -9 $(lsof -ti:3000)`
- Use a different port in your configuration

#### 2. Module Not Found Errors

**Error**: `Cannot find module '@tools/base-tool.js'`

**Solutions**:
- Ensure you've built the project: `pnpm build`
- Check TypeScript path aliases in `tsconfig.json`
- Verify the file extension is `.js` in import statements

#### 3. TypeScript Compilation Errors

**Error**: Various TypeScript errors during `pnpm check`

**Solutions**:
- Check for missing type definitions: `pnpm install @types/node`
- Verify TypeScript configuration in `tsconfig.json`
- Ensure all imports use correct file extensions

#### 4. Session Management Issues

**Error**: `Session not found` or session-related errors

**Solutions**:
- Ensure you're sending the `X-MCP-Session-ID` header
- Check that the session ID is from a recent `initialize` call
- Verify session timeout settings

#### 5. JSON-RPC Protocol Errors

**Error**: Invalid JSON-RPC requests

**Solutions**:
- Ensure requests include `jsonrpc: "2.0"`
- Include a unique `id` field for each request
- Use correct `method` names (e.g., `tools/list`, `resources/read`)

### Debug Mode

Enable debug logging to troubleshoot issues:

```bash
DEBUG=* pnpm dev
```

This will show detailed logs for:
- HTTP requests and responses
- Session creation and management
- Tool and resource execution
- Error handling

### Checking Server Health

Always start troubleshooting by checking the health endpoint:

```bash
curl http://localhost:3000/health
```

A healthy response should look like:
```json
{
  "status": "healthy",
  "name": "mcp-server",
  "version": "1.0.0",
  "timestamp": "2025-07-19T12:00:00.000Z",
  "sessions": 0
}
```

### Validation Errors

When creating custom tools or resources, common validation issues include:

- **Missing required parameters**: Ensure all required fields are provided
- **Invalid parameter types**: Check Zod schema definitions
- **Empty or invalid values**: Implement proper validation in your classes

### Performance Issues

If you experience performance problems:

1. **Check session count**: Monitor active sessions via health endpoint
2. **Review tool complexity**: Ensure tools don't perform long-running operations
3. **Monitor memory usage**: Use Node.js profiling tools
4. **Optimize resource queries**: Cache frequently accessed data

## Deployment

### Production Build

1. **Build the project**:
   ```bash
   pnpm build
   ```

2. **Start production server**:
   ```bash
   pnpm start
   ```

### Docker Deployment

Create a `Dockerfile`:

```dockerfile
FROM node:20-alpine

WORKDIR /app

# Copy package files
COPY package.json pnpm-lock.yaml ./

# Install dependencies
RUN npm install -g pnpm && pnpm install --frozen-lockfile

# Copy source code
COPY . .

# Build the application
RUN pnpm build

# Expose port
EXPOSE 3000

# Start the server
CMD ["pnpm", "start"]
```

Build and run:
```bash
docker build -t mcp-server .
docker run -p 3000:3000 mcp-server
```

### Environment Variables for Production

```bash
# Required
NODE_ENV=production
PORT=3000

# Optional
DEBUG=                                    # Disable debug logging
LOG_LEVEL=info                           # Set log level
```

### Performance and Scaling Guidelines

#### Memory Management

- **Session Cleanup**: The server automatically cleans up inactive sessions
- **Resource Pooling**: Consider implementing connection pooling for database resources
- **Memory Monitoring**: Use tools like `clinic.js` for memory profiling

#### Horizontal Scaling

- **Stateless Design**: The server is designed to be stateless (sessions are in-memory)
- **Load Balancing**: Use sticky sessions or external session storage for multiple instances
- **Health Checks**: The `/health` endpoint supports load balancer health checks

#### Performance Optimization

- **Tool Execution**: Keep tool execution time under 30 seconds
- **Resource Caching**: Implement caching for frequently accessed resources
- **Async Operations**: Use async/await for all I/O operations

#### Monitoring

Set up monitoring for:
- **Response times**: Track endpoint performance
- **Error rates**: Monitor failed requests
- **Session count**: Watch for session leaks
- **Resource usage**: CPU and memory consumption

### Security Considerations

#### DNS Rebinding Protection

Enable DNS rebinding protection in production:

```typescript
const config: ServerConfig = {
  enableDnsRebindingProtection: true,
  allowedHosts: ["your-domain.com", "api.your-domain.com"],
  // ... other config
};
```

#### Input Validation

- All tool and resource inputs are validated using Zod schemas
- Implement additional sanitization for database queries
- Use parameterized queries to prevent SQL injection

#### HTTPS in Production

Use a reverse proxy (nginx, Cloudflare) to terminate SSL:

```nginx
server {
    listen 443 ssl;
    server_name your-domain.com;
    
    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;
    
    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

## Contributing

We welcome contributions to improve the MCP server implementation!

### Development Workflow

1. **Fork the repository**
2. **Create a feature branch**:
   ```bash
   git checkout -b feature/new-tool
   ```

3. **Make your changes**:
   - Follow the existing code style
   - Add tests for new functionality
   - Update documentation as needed

4. **Run quality checks**:
   ```bash
   pnpm check
   ```

5. **Commit your changes**:
   ```bash
   git commit -m "feat: add weather tool"
   ```

6. **Push and create a Pull Request**

### Code Standards

#### TypeScript Guidelines

- Use strict TypeScript settings
- Prefer interfaces over types for object shapes
- Use async/await over Promises
- Implement proper error handling

#### Naming Conventions

- **Classes**: PascalCase (e.g., `WeatherTool`, `DatabaseResource`)
- **Files**: kebab-case (e.g., `weather-tool.ts`, `database-resource.ts`)
- **Variables**: camelCase (e.g., `sessionId`, `toolRegistry`)
- **Constants**: UPPER_SNAKE_CASE (e.g., `DEFAULT_PORT`, `MAX_SESSIONS`)

#### Code Organization

- Keep files focused on a single responsibility
- Use the established directory structure
- Implement base classes for common patterns
- Use dependency injection for testability

#### Documentation

- Add JSDoc comments for public methods
- Include usage examples in complex code
- Update README for new features
- Document configuration options

### Pull Request Guidelines

- Include a clear description of changes
- Reference related issues
- Ensure all checks pass
- Add tests for new functionality
- Update documentation

### Reporting Issues

When reporting issues, please include:

- Node.js and package manager versions
- Steps to reproduce the issue
- Expected vs actual behavior
- Relevant error messages or logs
- Configuration details (if applicable)

## Resources

### Model Context Protocol

- **Official Specification**: [https://spec.modelcontextprotocol.io/](https://spec.modelcontextprotocol.io/)
- **GitHub Repository**: [https://github.com/modelcontextprotocol](https://github.com/modelcontextprotocol)
- **Community Resources**: [https://modelcontextprotocol.io/](https://modelcontextprotocol.io/)

### Technology Stack

- **Node.js Documentation**: [https://nodejs.org/docs/](https://nodejs.org/docs/)
- **TypeScript Handbook**: [https://www.typescriptlang.org/docs/](https://www.typescriptlang.org/docs/)
- **Express.js Guide**: [https://expressjs.com/](https://expressjs.com/)
- **Zod Documentation**: [https://zod.dev/](https://zod.dev/)

### Development Tools

- **Biome Linter**: [https://biomejs.dev/](https://biomejs.dev/)
- **JSON-RPC Specification**: [https://www.jsonrpc.org/](https://www.jsonrpc.org/)
- **pnpm Documentation**: [https://pnpm.io/](https://pnpm.io/)

### Related Projects

- **MCP SDK**: [https://github.com/modelcontextprotocol/typescript-sdk](https://github.com/modelcontextprotocol/typescript-sdk)
- **Language Server Protocol**: [https://microsoft.github.io/language-server-protocol/](https://microsoft.github.io/language-server-protocol/)

---

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

## Acknowledgments

- The Model Context Protocol team for creating the MCP specification
- The TypeScript and Node.js communities for excellent tooling
- Contributors to the open-source packages used in this project

---

*For more information, questions, or support, please open an issue on the GitHub repository.*
