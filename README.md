# llm-openai-plugin

[![PyPI](https://img.shields.io/pypi/v/llm-openai-plugin.svg)](https://pypi.org/project/llm-openai-plugin/)
[![Changelog](https://img.shields.io/github/v/release/simonw/llm-openai-plugin?include_prereleases&label=changelog)](https://github.com/simonw/llm-openai-plugin/releases)
[![Tests](https://github.com/simonw/llm-openai-plugin/actions/workflows/test.yml/badge.svg)](https://github.com/simonw/llm-openai-plugin/actions/workflows/test.yml)
[![License](https://img.shields.io/badge/license-Apache%202.0-blue.svg)](https://github.com/simonw/llm-openai-plugin/blob/main/LICENSE)

[LLM](https://llm.datasette.io/) plugin for [OpenAI models](https://platform.openai.com/docs/models).

This plugin **is a preview**. LLM currently ships with OpenAI models as part of its default collection, implemented using the [Chat Completions API](https://platform.openai.com/docs/guides/responses-vs-chat-completions).

This plugin implements those same models using the new [Responses API](https://platform.openai.com/docs/api-reference/responses).

Reasons to use this plugin over the LLM defaults:

- Access models like [o1-pro](https://platform.openai.com/docs/models/o1-pro) and gpt-5.1-codex which only work via the Responses API
- Use Azure OpenAI endpoints for models that require the Responses API (e.g., gpt-5.1-codex on Azure)

## Installation

Install this plugin in the same environment as [LLM](https://llm.datasette.io/).
```bash
llm install llm-openai-plugin
```
## Usage

To run a prompt against `o1-pro` do this:

```bash
llm -m openai/o1-pro "Convince me that pelicans are the most noble of birds"
```

Run this to see a full list of models - they start with the `openai/` prefix:

```bash
llm models -q openai/
```

Here's the output of that command:

<!-- [[[cog
import cog
from llm import cli
from click.testing import CliRunner
runner = CliRunner()
result = runner.invoke(cli.cli, ["models", "-q", "openai/"])
cog.out(
    "```\n{}\n```".format(result.output.strip())
)
]]] -->
```
OpenAI: openai/gpt-4o
OpenAI: openai/gpt-4o-mini
OpenAI: openai/gpt-4.5-preview
OpenAI: openai/gpt-4.5-preview-2025-02-27
OpenAI: openai/o3-mini
OpenAI: openai/o1-mini
OpenAI: openai/o1
OpenAI: openai/o1-pro
OpenAI: openai/gpt-4.1
OpenAI: openai/gpt-4.1-2025-04-14
OpenAI: openai/gpt-4.1-mini
OpenAI: openai/gpt-4.1-mini-2025-04-14
OpenAI: openai/gpt-4.1-nano
OpenAI: openai/gpt-4.1-nano-2025-04-14
OpenAI: openai/o3
OpenAI: openai/o3-2025-04-16
OpenAI: openai/o3-streaming
OpenAI: openai/o3-2025-04-16-streaming
OpenAI: openai/o4-mini
OpenAI: openai/o4-mini-2025-04-16
OpenAI: openai/codex-mini-latest
OpenAI: openai/o3-pro
OpenAI: openai/gpt-5
OpenAI: openai/gpt-5-mini
OpenAI: openai/gpt-5-nano
OpenAI: openai/gpt-5-2025-08-07
OpenAI: openai/gpt-5-mini-2025-08-07
OpenAI: openai/gpt-5-nano-2025-08-07
OpenAI: openai/gpt-5-codex
OpenAI: openai/gpt-5-pro
OpenAI: openai/gpt-5-pro-2025-10-06
OpenAI: openai/gpt-5.1
OpenAI: openai/gpt-5.1-chat-latest
OpenAI: openai/gpt-5.1-codex
OpenAI: openai/gpt-5.1-codex-mini
OpenAI: openai/gpt-5.1-codex-max
OpenAI: openai/gpt-5.2
OpenAI: openai/gpt-5.2-chat-latest
OpenAI: openai/gpt-5.2-pro
```
<!-- [[[end]]] -->
Add `--options` to see a full list of options that can be provided to each model.

The `o3-streaming` model ID exists because o3 currently requires a verified organization in order to support streaming. If you have a verified organization you can use `o3-streaming` - everyone else should use `o3`.

### Reasoning options

Models with reasoning capabilities (o1, o3, gpt-5, etc.) support additional options:

```bash
llm -m openai/o3-mini -o reasoning_effort low "What is 2+2?"
llm -m openai/o3-mini -o reasoning_effort high "Solve this complex problem"
llm -m openai/o3-mini -o reasoning_summary concise "Explain quantum computing"
```

Available `reasoning_effort` values: `minimal`, `low`, `medium`, `high`

Available `reasoning_summary` values: `auto`, `concise`, `detailed`

## Configuring Azure and Custom Endpoints

You can configure additional models that use the Responses API by creating a YAML configuration file. This is useful for Azure OpenAI deployments or local Responses API-compatible servers.

Create `~/.config/io.datasette.llm/extra-responses-models.yaml` (on macOS: `~/Library/Application Support/io.datasette.llm/extra-responses-models.yaml`):

```yaml
# Azure OpenAI example
- model_id: azure-codex
  model_name: gpt-5.1-codex
  api_base: "https://my-resource.openai.azure.com/openai/v1/"
  api_key_name: azure-openai
  vision: true
  reasoning: true
  aliases:
    - codex

# Local server example (no authentication)
- model_id: local-model
  model_name: my-local-model
  api_base: "http://localhost:8080/v1/"
  vision: false
  reasoning: false
```

Then set your API key:

```bash
llm keys set azure-openai
# Paste your Azure OpenAI API key
```

Now you can use the model:

```bash
llm -m azure-codex "Write a function to sort a list"
llm -m azure-codex -o reasoning_effort high "Solve this problem"
```

### Configuration options

| Key | Required | Description |
|-----|----------|-------------|
| `model_id` | Yes | Unique identifier for the model in LLM |
| `model_name` | Yes | Model/deployment name sent to the API |
| `api_base` | Yes | Base URL for the API endpoint |
| `api_key_name` | No | Key alias from `llm keys` (omit for no auth) |
| `aliases` | No | Alternative names for the model |
| `headers` | No | Custom HTTP headers (dict) |
| `vision` | No | Enable image/PDF attachments (default: false) |
| `reasoning` | No | Enable reasoning options (default: false) |
| `can_stream` | No | Enable streaming (default: true) |
| `supports_schema` | No | Enable JSON schema output (default: true) |

## Development

To set up this plugin locally, first checkout the code. Then create a new virtual environment:
```bash
cd llm-openai-plugin
python -m venv venv
source venv/bin/activate
```
Now install the dependencies and test dependencies:
```bash
llm install -e '.[test]'
```
To run the tests:
```bash
python -m pytest
```

This project uses [pytest-recording](https://github.com/kiwicom/pytest-recording) to record OpenAI API responses for the tests, and [syrupy](https://github.com/syrupy-project/syrupy) to capture snapshots of their results.

If you add a new test that calls the API you can capture the API response and snapshot like this:
```bash
PYTEST_OPENAI_API_KEY="$(llm keys get openai)" pytest --record-mode once --snapshot-update
```
Then review the new snapshots in `tests/__snapshots__/` to make sure they look correct.
