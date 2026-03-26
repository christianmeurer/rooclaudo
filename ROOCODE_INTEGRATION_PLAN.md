# RooCode Integration Plan

## 1. Current Architecture Overview
The current application functions as a wrapper around the Claude Code CLI. The architecture relies on:
- **Claude Code Wrapper:** A CLI execution environment that launches Claude Code.
- **LiteLLM Proxy:** A proxy layer used to route and manage LLM API requests.
- **DigitalOcean API:** Integrations for managing backend resources or deployments.
In this setup, API keys, proxy endpoints, and other configurations are dynamically passed to the Claude Code CLI using standard environment variables during execution.

## 2. The Core Challenge
Migrating to RooCode introduces a fundamental environmental shift:
- **Environment Context:** RooCode is a Visual Studio Code (VS Code) extension, whereas Claude Code is a standalone CLI application.
- **Configuration Injection:** As a VS Code extension, RooCode does not inherit configuration via environment variables passed from an external CLI wrapper in the same straightforward manner. It relies on VS Code's internal settings and extension storage mechanisms.

## 3. Proposed Architectural Changes
To bridge the gap between a CLI-centric wrapper and a VS Code extension, the architecture must transition to a Daemon/Launcher model that directly manipulates VS Code's state.

- **Daemon/Launcher Mode:** The application will no longer simply spawn a child process. Instead, it will act as a launcher that prepares the environment and then opens VS Code.
- **`globalStorage` Modification:** To configure RooCode with the LiteLLM proxy and necessary API keys, the launcher must programmatically modify the VS Code extension's `globalStorage`. This involves locating the RooCode extension's state files (typically JSON or SQLite databases within the VS Code user data directory) and injecting the configuration directly before VS Code initializes.

## 4. Step-by-Step Implementation Guide

### Step 1: Daemonize the LiteLLM Proxy
- Refactor the current wrapper to start the LiteLLM proxy as an independent background daemon process.
- Ensure the daemon logs output appropriately and maintains a stable connection to the DigitalOcean API.

### Step 2: Implement the Configuration Injector
- Create a utility to locate the host machine's VS Code user data directory (handling differences across Windows, macOS, and Linux).
- Implement logic to safely read, parse, modify, and save the `globalStorage` specific to the RooCode extension.
- Inject the local LiteLLM proxy URL and any required authentication tokens into the extension's configuration schema.

### Step 3: Develop the VS Code Launcher
- Build a routine that executes the Configuration Injector to ensure `globalStorage` is up-to-date.
- Use system commands (e.g., `code .`) to launch VS Code in the target workspace after the storage has been successfully modified.

### Step 4: Manage Execution State
- Implement checks to determine if VS Code is already running.
- If VS Code is running, notify the user that a restart of the editor may be required to pick up `globalStorage` changes, or handle graceful termination and restart if permitted.

### Step 5: Validation and Testing
- Test the end-to-end flow: from executing the launcher to interacting with the RooCode extension in VS Code.
- Verify that RooCode successfully routes its requests through the local LiteLLM proxy and correctly interfaces with the DigitalOcean API infrastructure.
