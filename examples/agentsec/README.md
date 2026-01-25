# agentsec Examples

Comprehensive examples demonstrating how to secure AI agents with **Cisco AI Defense** using the `aidefense.runtime.agentsec` module.

## Quick Start

```bash
# 1. Configure credentials
cp .env.example .env
# Edit .env with your AI Defense API key and provider credentials

# 2. Run a simple example
cd 1-simple && poetry install && poetry run python basic_protection.py

# 3. Or run an agent framework example
cd 2-agent-frameworks/strands-agent && poetry install && ./scripts/run.sh --openai

# 4. Run all tests from the repo root
cd /path/to/ai-defense-python-sdk
./scripts/run-unit-tests.sh           # 996 unit tests
./scripts/run-integration-tests.sh    # Full integration tests
```

---

## Overview

| Category | Description | Examples |
|----------|-------------|----------|
| **1-simple/** | Standalone examples for core features | 7 examples |
| **2-agent-frameworks/** | Agent frameworks with MCP tools | 6 frameworks |
| **3-agent-runtimes/** | Cloud deployment with AI Defense | 2 runtimes, 6 modes |

---

## Core Concepts

### Integration Modes

agentsec supports two ways to integrate with Cisco AI Defense:

| Mode | How It Works | When to Use |
|------|--------------|-------------|
| **API Mode** (default) | SDK inspects requests via AI Defense API, then calls LLM directly | Most deployments |
| **Gateway Mode** | SDK routes all traffic through AI Defense Gateway proxy | Centralized policy, caching |

Set via environment variable:
```bash
AGENTSEC_LLM_INTEGRATION_MODE=api      # or "gateway"
AGENTSEC_MCP_INTEGRATION_MODE=api      # MCP can use different mode than LLM
```

### Supported LLM Providers

agentsec automatically patches these LLM client libraries:

| Provider | Package | Patched Methods |
|----------|---------|-----------------|
| **OpenAI** | `openai` | `chat.completions.create()` |
| **Azure OpenAI** | `openai` | `chat.completions.create()` (with Azure endpoint) |
| **AWS Bedrock** | `boto3` | `converse()`, `converse_stream()` |
| **Google Vertex AI** | `google-cloud-aiplatform` | `ChatVertexAI`, `generate_content()` |
| **Google GenAI** | `google-genai` | `generate_content()`, `generate_content_async()` |
| **MCP** | `mcp` | `ClientSession.call_tool()`, `get_prompt()`, `read_resource()` |

### Protection Coverage

agentsec inspects both **requests** (user prompts) and **responses** (LLM outputs):

| Inspection | What It Checks | When It Happens |
|------------|----------------|-----------------|
| **LLM Request** | Prompt injection, jailbreak, PII in prompts | Before LLM call |
| **LLM Response** | Sensitive data leakage, harmful content | After LLM response |
| **MCP Request** | Tool call arguments, prompt injection | Before MCP tool call |
| **MCP Response** | Tool response content, data leakage | After MCP tool response |

### Inspection Modes

| Mode | Behavior | Use Case |
|------|----------|----------|
| `off` | No inspection | Disabled |
| `on_monitor` | Inspect & log, never block | Testing, observability |
| `on_enforce` | Inspect & block violations | Production |

Set via environment variable:
```bash
AGENTSEC_API_MODE_LLM=on_enforce   # or "on_monitor", "off"
AGENTSEC_API_MODE_MCP=on_enforce   # MCP tool inspection mode
```

---

## Directory Structure

```
ai-defense-python-sdk/
├── scripts/
│   ├── run-unit-tests.sh           # Run all 996 unit tests
│   └── run-integration-tests.sh    # Run integration tests
│
└── examples/agentsec/
    ├── .env.example                # Template - copy to .env
    ├── README.md                   # This file
    │
    ├── 1-simple/                   # Standalone examples
    │   ├── basic_protection.py     # Minimal setup
    │   ├── openai_example.py       # OpenAI client
    │   ├── streaming_example.py    # Streaming responses
    │   ├── mcp_example.py          # MCP tool inspection
    │   ├── gateway_mode_example.py # Gateway mode
    │   ├── skip_inspection_example.py  # Per-call exclusion
    │   ├── simple_strands_bedrock.py   # Strands + Bedrock
    │   └── tests/
    │       ├── unit/               # Unit tests
    │       └── integration/        # Integration tests
    │
    ├── 2-agent-frameworks/         # Agent framework examples
    │   ├── strands-agent/          # AWS Strands SDK
    │   ├── langgraph-agent/        # LangGraph
    │   ├── langchain-agent/        # LangChain
    │   ├── crewai-agent/           # CrewAI
    │   ├── autogen-agent/          # AutoGen
    │   ├── openai-agent/           # OpenAI Agents SDK
    │   └── _shared/                # Shared provider configs & tests
    │
    └── 3-agent-runtimes/           # Cloud runtime examples
        ├── amazon-bedrock-agentcore/   # AWS AgentCore
        │   ├── direct-deploy/      # Direct code (boto3 SDK)
        │   ├── container-deploy/   # Docker container
        │   ├── lambda-deploy/      # AWS Lambda
        │   └── tests/              # Unit & integration tests
        └── gcp-vertex-ai-agent-engine/ # GCP Vertex AI
            ├── agent-engine-deploy/    # Managed Agent Engine
            ├── cloud-run-deploy/       # Cloud Run
            ├── gke-deploy/             # GKE Kubernetes
            └── tests/              # Unit & integration tests
```

---

## 1. Simple Examples

Standalone examples demonstrating core agentsec features without agent frameworks.

| Example | Description | Request | Response |
|---------|-------------|:-------:|:--------:|
| `basic_protection.py` | Minimal setup - modes, patched clients | ✅ | ✅ |
| `openai_example.py` | OpenAI client with automatic inspection | ✅ | ✅ |
| `streaming_example.py` | Streaming responses with chunk inspection | ✅ | ✅ |
| `mcp_example.py` | MCP tool call inspection (pre & post) | ✅ | ✅ |
| `gateway_mode_example.py` | Gateway mode configuration | ✅ | ✅ |
| `skip_inspection_example.py` | Per-call exclusion with context manager | ✅ | ✅ |
| `simple_strands_bedrock.py` | Strands agent with Bedrock Claude | ✅ | ✅ |

### Run Examples

```bash
cd 1-simple
poetry install  # First time only

# Run individual examples
poetry run python basic_protection.py
poetry run python openai_example.py
poetry run python streaming_example.py
poetry run python mcp_example.py
poetry run python gateway_mode_example.py
poetry run python skip_inspection_example.py
poetry run python simple_strands_bedrock.py

# Run integration tests (from repo root)
cd /path/to/ai-defense-python-sdk
./scripts/run-integration-tests.sh --simple
```

---

## 2. Agent Frameworks

Agent framework examples with MCP tool support and multi-provider configuration.

### Supported Frameworks

| Framework | Package | Description |
|-----------|---------|-------------|
| **Strands** | `strands-agents` | AWS Strands SDK with native tool support |
| **LangGraph** | `langgraph` | LangChain's state graph pattern |
| **LangChain** | `langchain` | LCEL chains with tool calling |
| **CrewAI** | `crewai` | Multi-agent crew orchestration |
| **AutoGen** | `ag2` | Microsoft's multi-agent framework |
| **OpenAI Agents** | `openai` | OpenAI's native agent SDK |

### Provider Support

All frameworks support multiple LLM providers:

| Provider | Flag | Auth Methods | Config File |
|----------|------|--------------|-------------|
| **OpenAI** | `--openai` | API key | `config-openai.yaml` |
| **AWS Bedrock** | `--bedrock` | default, profile, session_token, iam_role | `config-bedrock.yaml` |
| **Azure OpenAI** | `--azure` | api_key, managed_identity, cli | `config-azure.yaml` |
| **GCP Vertex AI** | `--vertex` | adc, service_account | `config-vertex.yaml` |

### Run Examples

```bash
cd 2-agent-frameworks

# Run with specific provider
./strands-agent/scripts/run.sh --openai
./strands-agent/scripts/run.sh --bedrock
./langgraph-agent/scripts/run.sh --azure
./crewai-agent/scripts/run.sh --vertex

# Run integration tests (from repo root)
cd /path/to/ai-defense-python-sdk
./scripts/run-integration-tests.sh --agents                # All frameworks, all providers
./scripts/run-integration-tests.sh strands langgraph       # Specific frameworks
./scripts/run-integration-tests.sh --api                   # API mode only
./scripts/run-integration-tests.sh --gateway               # Gateway mode only
./scripts/run-integration-tests.sh strands --gateway       # Specific framework + mode
```

### Protection Coverage

All agent frameworks support both request and response inspection:

- **LLM Calls**: Every `chat.completions.create()` / `converse()` / `generate_content()` is inspected
- **MCP Tool Calls**: Tool request and response payloads are inspected (when MCP is used)
- **Streaming**: Response chunks are buffered and inspected at completion

---

## 3. Agent Runtimes

Cloud deployment examples with full AI Defense protection.

### AWS Bedrock AgentCore

Three deployment modes for AWS AgentCore agents:

| Mode | Description | Protection | Request | Response |
|------|-------------|------------|:-------:|:--------:|
| **Direct Deploy** | Agent runs in AgentCore, client calls via boto3 SDK | Client-side | ✅ | ✅ |
| **Container Deploy** | Docker container on AgentCore | Server-side | ✅ | ✅ |
| **Lambda Deploy** | AWS Lambda function | Server-side | ✅ | ✅ |

> **Note**: Direct Deploy uses the **boto3 SDK directly** (not the AgentCore CLI) to ensure full response inspection. The CLI bypasses the patched client path for responses.

```bash
cd 3-agent-runtimes/amazon-bedrock-agentcore
poetry install

# Direct Deploy (boto3 SDK)
./direct-deploy/scripts/deploy.sh
poetry run python direct-deploy/test_with_protection.py "What is 5+5?"

# Container Deploy
./container-deploy/scripts/deploy.sh
./container-deploy/scripts/invoke.sh "What is 5+5?"

# Lambda Deploy
./lambda-deploy/scripts/deploy.sh
./lambda-deploy/scripts/invoke.sh "What is 5+5?"

# Run all integration tests (6 tests: 3 deploy modes x 2 integration modes)
./tests/integration/test-all-modes.sh --verbose
```

### GCP Vertex AI Agent Engine

Three deployment modes for GCP Vertex AI agents:

| Mode | Description | Protection | Request | Response |
|------|-------------|------------|:-------:|:--------:|
| **Agent Engine** | Google's managed agent service | Server-side | ✅ | ✅ |
| **Cloud Run** | Serverless containers | Server-side | ✅ | ✅ |
| **GKE** | Kubernetes deployment | Server-side | ✅ | ✅ |

**Supported Google AI SDKs:**

| SDK | Package | Status | Environment Variable |
|-----|---------|--------|----------------------|
| **vertexai** | `google-cloud-aiplatform` | Legacy (default) | `GOOGLE_AI_SDK=vertexai` |
| **google-genai** | `google-genai` | Modern (recommended) | `GOOGLE_AI_SDK=google_genai` |

```bash
cd 3-agent-runtimes/gcp-vertex-ai-agent-engine
poetry install

# Choose SDK (optional, defaults to vertexai)
export GOOGLE_AI_SDK=google_genai  # Use modern SDK

# Agent Engine (local test)
./agent-engine-deploy/scripts/deploy.sh test

# Cloud Run (deploy to GCP)
./cloud-run-deploy/scripts/deploy.sh
./cloud-run-deploy/scripts/invoke.sh "Check service health"
./cloud-run-deploy/scripts/cleanup.sh  # Clean up

# GKE (deploy to GCP)
./gke-deploy/scripts/deploy.sh setup   # First time: create cluster
./gke-deploy/scripts/deploy.sh
./gke-deploy/scripts/invoke.sh "Check service health"
./gke-deploy/scripts/cleanup.sh        # Clean up

# Run all integration tests (6 tests: 3 deploy modes x 2 integration modes)
./tests/integration/test-all-modes.sh --verbose
```

### Runtime Integration Tests

```bash
# Run from repo root
cd /path/to/ai-defense-python-sdk

# Run all runtime tests (24 total: 8 per runtime x 2 modes + MCP)
./scripts/run-integration-tests.sh --runtimes                      # All runtimes, all modes
./scripts/run-integration-tests.sh amazon-bedrock-agentcore        # Specific runtime
./scripts/run-integration-tests.sh gcp-vertex-ai-agent-engine      # Specific runtime
./scripts/run-integration-tests.sh amazon-bedrock-agentcore --gateway  # Specific mode
```

---

## Configuration

### Environment Variables

Copy `.env.example` to `.env` and configure:

#### AI Defense API Mode (Required for API integration)

| Variable | Required | Description |
|----------|:--------:|-------------|
| `AI_DEFENSE_API_MODE_LLM_ENDPOINT` | Yes | AI Defense API endpoint for LLM inspection |
| `AI_DEFENSE_API_MODE_LLM_API_KEY` | Yes | AI Defense API key for LLM inspection |
| `AI_DEFENSE_API_MODE_MCP_ENDPOINT` | For MCP | AI Defense API endpoint for MCP inspection |
| `AI_DEFENSE_API_MODE_MCP_API_KEY` | For MCP | AI Defense API key for MCP inspection |
| `AGENTSEC_API_MODE_LLM` | No | LLM mode: `on_enforce`, `on_monitor`, `off` (default: `on_monitor`) |
| `AGENTSEC_API_MODE_MCP` | No | MCP mode: `on_enforce`, `on_monitor`, `off` (default: `on_monitor`) |
| `AGENTSEC_LLM_INTEGRATION_MODE` | No | LLM integration: `api`, `gateway` (default: `api`) |
| `AGENTSEC_MCP_INTEGRATION_MODE` | No | MCP integration: `api`, `gateway` (default: `api`) |

#### AI Defense Gateway Mode (For Gateway integration)

| Variable | When Required | Description |
|----------|:-------------:|-------------|
| `AGENTSEC_OPENAI_GATEWAY_URL` | Gateway + OpenAI | OpenAI gateway URL |
| `AGENTSEC_OPENAI_GATEWAY_API_KEY` | Gateway + OpenAI | OpenAI gateway API key |
| `AGENTSEC_AZURE_OPENAI_GATEWAY_URL` | Gateway + Azure | Azure OpenAI gateway URL |
| `AGENTSEC_AZURE_OPENAI_GATEWAY_API_KEY` | Gateway + Azure | Azure OpenAI gateway API key |
| `AGENTSEC_BEDROCK_GATEWAY_URL` | Gateway + Bedrock | Bedrock gateway URL (uses AWS Sig V4 auth) |
| `AGENTSEC_VERTEXAI_GATEWAY_URL` | Gateway + Vertex | Vertex AI gateway URL (uses ADC OAuth2) |
| `AGENTSEC_MCP_GATEWAY_URL` | Gateway + MCP | MCP gateway URL |
| `AGENTSEC_MCP_GATEWAY_API_KEY` | Gateway + MCP | MCP gateway API key |

### Provider Credentials

<details>
<summary><strong>OpenAI</strong></summary>

```bash
OPENAI_API_KEY=sk-your-openai-api-key
```

</details>

<details>
<summary><strong>Azure OpenAI</strong></summary>

```bash
AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
AZURE_OPENAI_API_KEY=your-azure-openai-key
```

</details>

<details>
<summary><strong>AWS Bedrock</strong></summary>

```bash
# Option 1: Access keys
AWS_ACCESS_KEY_ID=your-access-key
AWS_SECRET_ACCESS_KEY=your-secret-key
AWS_REGION=us-east-1

# Option 2: Profile (recommended)
AWS_PROFILE=your-profile
AWS_REGION=us-east-1

# Option 3: SSO (interactive)
aws sso login
```

</details>

<details>
<summary><strong>GCP Vertex AI / Google GenAI</strong></summary>

```bash
GOOGLE_CLOUD_PROJECT=your-gcp-project-id
GOOGLE_CLOUD_LOCATION=us-central1

# Authenticate via gcloud CLI (ADC)
gcloud auth application-default login

# Choose SDK (optional)
GOOGLE_AI_SDK=google_genai  # Modern SDK (recommended)
# or
GOOGLE_AI_SDK=vertexai      # Legacy SDK (default)
```

</details>

### YAML Config (Agent Frameworks)

Each agent framework has config files in `config/`:

```yaml
# config/config-bedrock.yaml
provider: bedrock
bedrock:
  model_id: anthropic.claude-3-haiku-20240307-v1:0
  region: us-east-1
  auth:
    method: default  # or: profile, session_token, iam_role
```

Select config via flag:
```bash
./scripts/run.sh --bedrock   # Uses config/config-bedrock.yaml
./scripts/run.sh --azure     # Uses config/config-azure.yaml
./scripts/run.sh --vertex    # Uses config/config-vertex.yaml
./scripts/run.sh --openai    # Uses config/config-openai.yaml
```

---

## Testing

### Test Summary

| Category | Test Type | Test Count | What It Validates |
|----------|-----------|:----------:|-------------------|
| **Core SDK** | Unit | ~600 | Patching, inspection, decisions, config |
| **Simple Examples** | Unit | ~70 | Example file structure, syntax |
| **Simple Examples** | Integration | 14 | 7 examples x 2 modes (API + Gateway) |
| **Agent Frameworks** | Unit | ~200 | Agent setup, provider configs |
| **Agent Frameworks** | Integration | 48 | 6 frameworks x 4 providers x 2 modes |
| **AgentCore** | Unit | ~60 | Deploy scripts, protection setup |
| **AgentCore** | Integration | 8 | 4 deploy modes x 2 integration modes |
| **Vertex AI** | Unit | ~50 | Deploy scripts, SDK selection |
| **Vertex AI** | Integration | 16 | 4 deploy modes x 2 integration modes x 2 (LLM + MCP) |

**Total: 996 unit tests**

### Run Tests

```bash
# From project root
cd /path/to/ai-defense-python-sdk

# All unit tests (996 tests)
./scripts/run-unit-tests.sh

# All integration tests
./scripts/run-integration-tests.sh

# Specific test categories
./scripts/run-integration-tests.sh --simple      # Simple examples only
./scripts/run-integration-tests.sh --agents      # Agent frameworks only
./scripts/run-integration-tests.sh --runtimes    # Agent runtimes only

# Specific mode
./scripts/run-integration-tests.sh --api         # API mode only
./scripts/run-integration-tests.sh --gateway     # Gateway mode only

# Specific framework/runtime
./scripts/run-integration-tests.sh strands       # Strands agent only
./scripts/run-integration-tests.sh amazon-bedrock-agentcore  # AgentCore only
```

---

## Integration Pattern

The standard pattern for integrating agentsec:

```python
# 1. Load environment variables FIRST
from pathlib import Path
from dotenv import load_dotenv
load_dotenv(Path(__file__).parent.parent / ".env")

# 2. Import and enable protection BEFORE importing LLM clients
from aidefense.runtime import agentsec
agentsec.protect(
    api_mode_llm="on_enforce",     # LLM inspection mode
    api_mode_mcp="on_enforce",     # MCP tool inspection mode
)

# 3. Import and use LLM clients normally - they're now protected
from openai import OpenAI
client = OpenAI()

# 4. All calls are inspected by AI Defense
response = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[{"role": "user", "content": "Hello!"}]
)
```

### Gateway Mode Pattern

```python
from aidefense.runtime import agentsec
agentsec.protect(
    llm_integration_mode="gateway",   # Use gateway instead of API
    mcp_integration_mode="gateway",   # MCP through gateway too
)
```

### Skip Inspection Pattern

```python
from aidefense.runtime.agentsec import skip_inspection, no_inspection

# Context manager - skip specific calls
with skip_inspection():
    response = client.chat.completions.create(...)

# Decorator - skip entire function
@no_inspection()
def my_function():
    response = client.chat.completions.create(...)
```

**Key Rule**: Always call `agentsec.protect()` **before** importing LLM client libraries.

---

## Troubleshooting

### Common Issues

| Issue | Solution |
|-------|----------|
| `ModuleNotFoundError: No module named 'aidefense'` | Run `poetry install` in the example directory |
| `ModuleNotFoundError: No module named 'dotenv'` | Run `poetry add python-dotenv` or `pip install python-dotenv` |
| AWS auth fails | Run `aws sts get-caller-identity` to verify credentials |
| Azure 401 Unauthorized | Check endpoint format: `https://your-resource.openai.azure.com` |
| GCP auth fails | Run `gcloud auth application-default login` |
| OpenAI 401 Unauthorized | Verify key at https://platform.openai.com/api-keys |
| `SecurityPolicyError` raised | Expected in enforce mode when content violates policies |
| `[BLOCKED] Prompt Injection` | AI Defense detected prompt injection - this is working correctly |
| No inspection happening | Ensure `agentsec.protect()` is called BEFORE importing LLM clients |
| MCP tool calls not inspected | Ensure `mcp` package is installed and `AGENTSEC_API_MODE_MCP` is set |
| Poetry version error | Remove `package-mode = false` from pyproject.toml if using older Poetry |

### Debug Logging

Enable detailed logs to see what agentsec is doing:

```bash
export AGENTSEC_LOG_LEVEL=DEBUG
```

You'll see output like:

```
╔══════════════════════════════════════════════════════════════
║ [PATCHED] LLM CALL: gpt-4o-mini
║ Operation: OpenAI.chat.completions.create | Mode: on_enforce | Integration: api
╚══════════════════════════════════════════════════════════════
[aidefense.runtime.agentsec.patchers.openai] DEBUG: Request inspection (1 messages)
[aidefense.runtime.agentsec.inspectors.llm] DEBUG: AI Defense response: {'action': 'allow', ...}
[aidefense.runtime.agentsec.patchers.openai] DEBUG: Response inspection (response: 42 chars)
```

For MCP tool calls:
```
[aidefense.runtime.agentsec.patchers.mcp] INFO: MCP client patched successfully
[aidefense.runtime.agentsec.inspectors.mcp] DEBUG: MCP Request inspection executed
[aidefense.runtime.agentsec.inspectors.mcp] DEBUG: MCP Response inspection executed
```

### Getting Help

- Check the individual example READMEs for detailed setup instructions
- Review the main [README.md](../../README.md) for SDK configuration
- Enable DEBUG logging to trace inspection flow
- Run `./scripts/run-unit-tests.sh` to verify SDK installation
