# Mellum SDK

This project provides an SDK for code completion using the Mellum model through Ollama. It enables integration of JetBrains' Mellum code completion capabilities into various development environments and tools.

## Overview

Mellum SDK is a Kotlin-based library that connects to Ollama to provide Mellum-powered code completions. It includes components for:

- Generating code completions using the Mellum model
- Integrating with Language Server Protocol (LSP)

## Requirements

- **Ollama** server running locally or remotely
- **JDK 17+** (for building and running the SDK)
- **11+ GB of available RAM** (minimum required to run the Mellum model)

## Quick Start

### 1. Install Ollama

Download and install Ollama from [ollama.com](https://ollama.com/download).

### 2. Pull the Mellum Model

```bash
ollama pull JetBrains/Mellum-4b-sft-all
```

Available models:
- `JetBrains/Mellum-4b-sft-all` — General-purpose model (default)
- `JetBrains/Mellum-4b-sft-kotlin` — Kotlin-optimized model

### 3. Start Ollama Server

```bash
ollama serve
```

The server will start on `http://localhost:11434` by default.

### 4. Add Mellum SDK to Your Project

Add the dependency to your `build.gradle.kts`:

```kotlin
dependencies {
    implementation("ai.jetbrains.code.mellum.sdk:sdk:0.2.2")
}
```

## Usage

### Basic Code Completion

```kotlin
import ai.grazie.code.files.jvm.*
import ai.grazie.code.files.model.DocumentProvider.Position
import ai.jetbrains.code.mellum.sdk.MellumCompletionService
import ai.koog.rag.base.files.JVMFileSystemProvider
import java.nio.file.Paths

// Create the completion service
val service = MellumCompletionService(
    documentProvider = JVMDocumentProvider,
    languageProvider = JVMLanguageProvider,
    fileSystemProvider = JVMFileSystemProvider.ReadOnly,
    workspaceProvider = JVMWorkspaceProvider(emptySet())
)

// Get completion at a specific position
val file = Paths.get("/path/to/your/file.kt")
val position = Position(line = 10, character = 5) // 0-based line and character
val completion = service.getCompletion(file, position)

println("Completion: $completion")
```

### Custom Configuration

```kotlin
import ai.jetbrains.code.mellum.sdk.MellumCompletionService
import ai.jetbrains.code.mellum.sdk.ollama.OllamaClient

// Connect to a remote Ollama server
val ollamaClient = OllamaClient(url = "http://remote-server:11434")

// Use a specific model and token limit
val service = MellumCompletionService(
    documentProvider = JVMDocumentProvider,
    languageProvider = JVMLanguageProvider,
    fileSystemProvider = JVMFileSystemProvider.ReadOnly,
    workspaceProvider = JVMWorkspaceProvider(emptySet()),
    ollamaClient = ollamaClient,
    modelName = "JetBrains/Mellum-4b-sft-kotlin",
    tokenLimit = 4096 // Adjust based on your needs
)
```

### LSP Integration

Mellum SDK provides LSP (Language Server Protocol) support for editor integration.

```kotlin
import ai.jetbrains.code.mellum.sdk.lsp.MellumLspCompletionService
import ai.grazie.code.files.jvm.*
import ai.jetbrains.code.mellum.sdk.ollama.OllamaClient

// Create the LSP completion service
val lspService = MellumLspCompletionService(
    documentProvider = JVMDocumentProvider,
    languageProvider = JVMLanguageProvider,
    fileSystemProvider = JVMFileSystemProvider.ReadOnly,
    workspaceProvider = JVMWorkspaceProvider(emptySet()),
    ollamaClient = OllamaClient(),
    modelName = "JetBrains/Mellum-4b-sft-all"
)

// The service exposes two JSON-RPC methods:
// - mellum/completion: Returns formatted completion (with post-processing for VSCode)
// - mellum/rawCompletion: Returns raw completion without formatting
```

The LSP handler uses the `@JsonSegment("mellum")` annotation, so the methods are available as:
- `mellum/completion`
- `mellum/rawCompletion`

## Architecture

### Core Components

- **`MellumCompletionService`**: Main entry point for code completion. Collects context from the file and workspace, prepares the prompt, and executes completion via Ollama.

- **`OllamaClient`**: HTTP client for communicating with the Ollama API. Supports custom URLs for remote servers.

- **`OllamaCompletionExecutor`**: Executes completion requests using the Mellum model.

- **`MellumLspCompletionService`**: LSP wrapper around `MellumCompletionService` for editor integration.

### How It Works

1. **Context Collection**: The SDK analyzes the current file and workspace to collect relevant context (neighboring files, imports, etc.).

2. **Prompt Preparation**: Constructs a FIM (Fill-in-the-Middle) prompt with:
   - File path
   - Code before cursor (`<fim_prefix>`)
   - Code after cursor (`<fim_suffix>`)
   - Collected context from other files
   - Token limit enforcement (default: 8192 tokens)

3. **Completion Execution**: Sends the prompt to Ollama, which runs the Mellum model locally.

4. **Result Processing**: Returns the generated completion text.

## Configuration Options

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `url` (OllamaClient) | String | `http://localhost:11434` | Ollama server URL |
| `modelName` | String | `JetBrains/Mellum-4b-sft-all` | Model to use for completion |
| `tokenLimit` | Int | `8192` | Maximum tokens for the prompt |

## Building from Source

```bash
# Clone the repository
git clone https://github.com/JetBrains/mellum-sdk.git
cd mellum-sdk

# Build the project
./gradlew build

# Run tests (requires Ollama with Mellum model)
./gradlew jvmOllamaTest
```

## License

Mellum SDK is licensed under the [Apache 2.0 License](LICENSE.txt).
