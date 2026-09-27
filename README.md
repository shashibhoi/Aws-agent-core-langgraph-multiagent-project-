# Aws-agent-core-langgraph-multiagent-project
-# Multi\-Agent Trading Analyst — Amazon Bedrock AgentCore

A production\-style starter project for a multi\-agent trading research application using:

- LangGraph for orchestration
- Amazon Bedrock for LLM reasoning
- Amazon Bedrock AgentCore Runtime for hosting
- Optional yfinance market\-data tool
- Technical\-analysis and risk tools
- Optional MCP integration point
- CloudWatch/AgentCore observability through the AgentCore deployment
- Python 3\.13

> Important: This project is for research/simulation. It does not place trades or provide personalized financial advice.

## Architecture

```text
Client
  |
  v
AgentCore Runtime
  |
  v
LangGraph Supervisor
  |
  +--> Market Research Agent ----> Market Data Tool
  |
  +--> Technical Analysis Agent -> Indicators Tool
  |
  +--> Risk Agent ---------------> Risk Metrics
  |
  +--> Decision Agent
  |
  v
Structured Trading Research Report
```

## Project layout

```text
multi-agent-trading-agentcore/
├── agentcore/
│   └── agentcore.json
├── app/
│   └── TradingResearchAgent/
│       ├── main.py
│       ├── graph.py
│       ├── state.py
│       ├── config.py
│       ├── models.py
│       ├── agents/
│       │   ├── market_agent.py
│       │   ├── technical_agent.py
│       │   ├── risk_agent.py
│       │   └── decision_agent.py
│       ├── tools/
│       │   ├── market_data.py
│       │   ├── technical.py
│       │   └── risk.py
│       └── prompts.py
├── infra/
│   └── trading-agent-policy.json
├── scripts/
│   ├── local_test.py
│   └── package_zip.sh
├── tests/
│   └── test_graph.py
├── .env.example
├── .gitignore
├── Dockerfile
├── pyproject.toml
└── requirements.txt
```

## Prerequisites

- AWS account
- AWS CLI configured
- Node\.js 20\+
- Python 3\.13
- AgentCore CLI

Install the current AgentCore CLI:

```bash
npm install -g @aws/agentcore
agentcore --version
```

Configure AWS:

```bash
aws configure
aws sts get-caller-identity
```

Make sure the selected Bedrock model is enabled/available in your AWS account and region\.

## Local setup

```bash
python3.13 -m venv .venv
source .venv/bin/activate

pip install -r requirements.txt

cp .env.example .env
```

For a real Bedrock run, set:

```text
AWS_REGION=ap-south-1
BEDROCK_MODEL_ID=global.anthropic.claude-sonnet-4-5-20250929-v1:0
USE_MOCK_LLM=false
```

For a local graph smoke test without Bedrock credentials:

```text
USE_MOCK_LLM=true
```

Run:

```bash
python -m app.TradingResearchAgent.main
```

The AgentCore app listens on port 8080\.

Test:

```bash
curl -X POST http://localhost:8080/invocations \
  -H "Content-Type: application/json" \
  -d '{"symbol":"AAPL","query":"Analyze AAPL for research purposes"}'
```

## AgentCore local development

From the project root:

```bash
agentcore dev --no-browser
```

Then invoke:

```bash
curl -X POST http://localhost:8080/invocations \
  -H "Content-Type: application/json" \
  -d '{"symbol":"AAPL","query":"Analyze AAPL using market, technical and risk agents"}'
```

## Deploy with AgentCore CLI

The project includes `agentcore/agentcore.json`\.

Preview:

```bash
agentcore deploy --dry-run
```

Deploy:

```bash
agentcore deploy
```

Check:

```bash
agentcore status
```

Invoke:

```bash
agentcore invoke --prompt "Analyze AAPL using market, technical and risk agents"
```

Depending on the installed CLI version, the CLI may ask for deployment\-target/account configuration on first use\. Follow the generated project prompts and keep the runtime configuration in `agentcore/agentcore.json`\.

## Direct ZIP deployment

AgentCore Runtime supports direct Python ZIP deployment\. The current runtime limit is 250 MB zipped / 750 MB unzipped\.

For native dependencies, build for the AgentCore Linux/arm64 environment rather than copying arbitrary local binaries\.

The included script is a simple packaging helper:

```bash
bash scripts/package_zip.sh
```

For production dependency packaging, prefer the AgentCore CLI CodeZip build or AWS’s documented `uv pip install` target\-platform workflow\.

## IAM

The runtime execution role needs permissions appropriate to your model and tools\. `infra/trading-agent-policy.json` contains the minimal example policy for invoking Bedrock models and writing CloudWatch logs\.

Do not put AWS access keys in `.env` or source code when running on AgentCore\. AgentCore should use an IAM execution role\.

## Optional market data

By default the project can use `yfinance` for research data\. In production, replace this with your approved market\-data provider/API and store credentials in a proper secret/credential mechanism\.

No order\-placement capability is included intentionally\.

## MCP extension

The code has a clean tools boundary so the market\-data and portfolio tools can later be exposed through MCP or an AgentCore Gateway\. Keep the agent responsible for reasoning and the tool server responsible for deterministic external operations\.

## Output

The decision agent returns a structured research report:

- symbol
- research summary
- market analysis
- technical analysis
- risk analysis
- scenario
- confidence
- rationale
- risk flags

The system does not execute a BUY/SELL order\.

## Production checklist

- Replace demo/retail market\-data source with an approved provider
- Use an authenticated AgentCore endpoint
- Store secrets outside source control
- Add AgentCore Memory only if conversation/user state is needed
- Add AgentCore Gateway/MCP for enterprise tools
- Enable observability and CloudWatch transaction search
- Add evaluation datasets and regression tests
- Add rate limits and cost controls
- Add human approval before any future order\-execution capability
