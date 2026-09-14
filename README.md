# TianGong AI LangGraph Server

## Retirement notice / 项目归档说明

This project was retired from active Tiangong AI workspace development on
2026-09-14. Its source and GitHub history are retained for reference.

- Organization: [tiangong-ai](https://github.com/tiangong-ai)
- Final repository: [tiangong-ai/langgraph-server](https://github.com/tiangong-ai/langgraph-server)
- Active workspace: [tiangong-ai/workspace-suite](https://github.com/tiangong-ai/workspace-suite)

The repository is being removed from the workspace's submodules and active
delivery catalog, then archived after that integration completes. No further
feature development or routine maintenance is planned here. The installation
and deployment instructions below are historical reference, not a maintained
deployment recommendation. No replacement service is designated by this notice.

Source retirement does not shut down an existing deployment. The recorded
LangGraph runtime also serves LCA consumers, including `lca_ai_suggestion`;
its host has carried LCA worker workloads. AWS resources, domains, certificates,
LangSmith, containers, data and credentials require a separate operational
handoff and remain unchanged by this source retirement.

本项目于 2026-09-14 退出天工 AI workspace 的持续开发，保留源码与 GitHub
历史供查阅。新组织为 [tiangong-ai](https://github.com/tiangong-ai)，本仓库的
最终地址为 [tiangong-ai/langgraph-server](https://github.com/tiangong-ai/langgraph-server)。
完成 workspace 子模块和交付目录移除后，本仓库归档，不再开展功能开发和日常维护。
下方安装与部署说明仅作历史参考，本公告没有指定替代服务。

源码归档不代表线上服务已停用。已有部署涉及 AI/LCA 共用运行环境及
`lca_ai_suggestion`，宿主机还承载过 LCA worker。AWS、域名、证书、LangSmith、
线上容器、数据和凭据保留现状，后续通过独立运维交接处理。

## Historical documentation / 历史文档

## Install dependencies

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.0/install.sh | bash

nvm install
nvm alias default 20
nvm use

npm install
```

## Self deployment with Docker (LangGraph + LangSmith tracing)

Official references:
- https://docs.langchain.com/langgraph-platform/application-structure
- https://docs.langchain.com/langsmith/deploy-standalone-server
- https://docs.langchain.com/langsmith/trace-with-langgraph

### 1) Prepare env file

```bash
cp .env.self-hosted.example .env
```

Fill required secrets in `.env`, at least:
- `OPENAI_API_KEY`
- `OPENAI_CHAT_MODEL`
- `OPENAI_CHAT_MODEL_MINI`
- `LANGSMITH_API_KEY` (for tracing)

Enable LangSmith logging:
- `LANGSMITH_TRACING=true`
- `LANGSMITH_PROJECT=tiangong-ai-langgraph-server`
- `LANGSMITH_ENDPOINT=https://api.smith.langchain.com` (default SaaS endpoint)

### 2) Build LangGraph API image

```bash
npx @langchain/langgraph-cli@latest build -t tiangong-langgraph-server:local
```

### 3) Configure Nginx API key

Create local key file from template:

```bash
cp nginx/langgraph-auth-key.conf.example nginx/langgraph-auth-key.conf
```

Then edit `nginx/langgraph-auth-key.conf`:

```nginx
map $http_x_api_key $langgraph_is_authorized {
  default 0;
  "your-strong-api-key" 1;
  # "your-second-key" 1;
}
```

### 4) Start services

```bash
docker compose -f docker-compose.self-hosted.yml up -d
```

If you need local Neo4j in the same stack:

```bash
docker compose -f docker-compose.self-hosted.yml --profile neo4j up -d
```

### 5) Verify server

```bash
curl http://localhost:8123/ok
# -> should return 401 without key

curl -H "X-API-Key: your-strong-api-key" http://localhost:8123/ok
# -> should return 200
```

Open LangSmith and check traces under your `LANGSMITH_PROJECT`.

## Local development server

```bash
npx @langchain/langgraph-cli@latest dev
```

## Background scripts (optional)

```bash
nohup node dist/multi_agents/kg_textbooks.js > kg_textbook.log 2>&1 &
tmux new -d -s neo4j_import 'node dist/multi_agents/kg_textbooks.js > kg_textbook.log 2>&1'
tmux kill-session -t neo4j_import
```

## Test prototype

```bash
npx ts-node src/prototype/structured_output.ts
```
