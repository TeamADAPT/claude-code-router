# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Development Commands
-   **Build the project**:
    ```bash
    npm run build
    ```
    This runs `scripts/build.js` which:
    - Builds CLI with esbuild (`src/cli.ts` → `dist/cli.js`)
    - Copies tiktoken WASM file to dist
    - Builds UI in `ui/` directory
    - Copies UI build artifacts to dist

-   **Release a new version**:
    ```bash
    npm run release
    ```

### CLI Commands (ccr)
-   **Start the router server**:
    ```bash
    ccr start
    ```
-   **Stop the router server**:
    ```bash
    ccr stop
    ```
-   **Restart the router server**:
    ```bash
    ccr restart
    ```
-   **Check the server status**:
    ```bash
    ccr status
    ```
-   **Run Claude Code through the router**:
    ```bash
    ccr code "<your prompt>"
    ```
-   **Open web UI**:
    ```bash
    ccr ui
    ```
-   **Status line integration**:
    ```bash
    ccr statusline
    ```

## Architecture

This project is a TypeScript-based router for Claude Code requests that enables routing to different LLM providers based on configurable rules.

### Core Components

-   **CLI Entry Point** (`src/cli.ts`): Main command handler that processes `ccr` commands (start, stop, code, ui, etc.)
-   **Server Factory** (`src/server.ts`): Creates the Fastify server instance with API endpoints and static file serving
-   **Service Runner** (`src/index.ts`): Initializes configuration, logging, and starts the server with hooks
-   **Router Logic** (`src/utils/router.ts`): Core routing logic that selects providers/models based on:
    - Token count (automatic long context switching)
    - Request type (background, thinking, web search)
    - Custom router scripts
    - Subagent model specifications

### Configuration System

-   **Config Location**: `~/.claude-code-router/config.json`
-   **Example Config**: `config.example.json` in repository root
-   **Environment Variable Interpolation**: Supports `$VAR_NAME` and `${VAR_NAME}` syntax
-   **Config Structure**:
    - `Providers`: Array of LLM provider configurations
    - `Router`: Routing rules for different scenarios
    - `transformers`: Custom transformer plugins
    - `CUSTOM_ROUTER_PATH`: Path to custom JavaScript router

### Routing Scenarios

The router automatically selects models based on:
-   `default`: General tasks
-   `background`: Background tasks (e.g., claude-3-5-haiku requests)
-   `think`: Reasoning-heavy tasks (when `thinking` field present)
-   `longContext`: Long context requests (>60K tokens by default)
-   `webSearch`: Web search tool usage
-   Custom routing via JavaScript files

### Build System

-   **Build Tool**: esbuild for fast TypeScript compilation
-   **Bundle Target**: Single CLI executable with bundled dependencies
-   **UI Integration**: React-based web UI built separately and served statically
-   **WASM Handling**: Copies tiktoken WASM file for token counting

### Dependencies

-   **Core Framework**: `@musistudio/llms` (Fastify-based LLM server framework)
-   **Token Counting**: `tiktoken` for accurate token calculation
-   **Process Management**: PID file tracking for service lifecycle
-   **Logging**: `pino-rotating-file-stream` for structured logging

### Web UI

Located in `ui/` directory with its own build system:
-   **Framework**: React with TypeScript
-   **Build Output**: Single HTML file with inlined assets
-   **Features**: Configuration management, provider setup, transformer configuration

### Authentication & Security

-   **API Key Authentication**: Optional APIKEY in config for request authentication
-   **Host Restrictions**: Forces localhost when no API key set
-   **Access Levels**: Different permission levels for UI operations

### Custom Extensions

-   **Custom Routers**: JavaScript files for complex routing logic
-   **Transformers**: Plugin system for request/response modification
-   **Subagent Routing**: Special model selection for subagent tasks via `<CCR-SUBAGENT-MODEL>` tags
