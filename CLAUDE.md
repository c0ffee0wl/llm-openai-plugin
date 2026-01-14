# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Plugin Does

This is an LLM plugin that implements OpenAI models using the **Responses API** (not the Chat Completions API). The key difference: models like o1-pro and gpt-5.1-codex only work via the Responses API.

Models are registered with the `openai/` prefix (e.g., `openai/gpt-4o-mini`, `openai/o3`).

## Commands

```bash
# Install with test dependencies (using uv)
uv pip install -e ".[test]"

# Run all tests
uv run pytest tests/ -v

# Run a single test
uv run pytest tests/test_openai.py::test_plugin_is_installed -v

# Record new API responses for tests (requires real API key)
PYTEST_OPENAI_API_KEY="$(llm keys get openai)" uv run pytest --record-mode once --snapshot-update
```

## Architecture

**Single-module plugin**: All code is in `llm_openai.py` (~650 lines).

### Key Classes

- `_SharedResponses` - Base class with shared logic for building API requests, handling responses, and managing tool calls
- `ResponsesModel(_SharedResponses, KeyModel)` - Sync model implementation
- `AsyncResponsesModel(_SharedResponses, AsyncKeyModel)` - Async model implementation

### Model Registration

`register_models()` is the plugin entry point (via `[project.entry-points.llm]` in pyproject.toml). It:
1. Registers hardcoded OpenAI models with their capabilities (vision, reasoning, streaming)
2. Loads additional models from `extra-responses-models.yaml` for Azure/custom endpoints

### Options System

Options are Pydantic models combined via `combine_options()`:
- `BaseOptions` - max_output_tokens, temperature, top_p, store, truncation
- `VisionOptions` - image_detail (only for vision-capable models)
- `ReasoningOptions` - reasoning_effort, reasoning_summary (only for reasoning models)

### API Flow

1. `execute()` calls `get_client()` to create OpenAI client (supports custom base_url for Azure)
2. `_build_kwargs()` constructs the API request from prompt, options, tools, and schema
3. `client.responses.create()` is called (this is the Responses API, not chat.completions)
4. Streaming events are handled by `_handle_event()`, tool calls buffered in `_tc_buf`

## Testing

Tests use pytest-recording to capture real API responses in `tests/cassettes/`. Snapshots are stored in `tests/__snapshots__/`.
