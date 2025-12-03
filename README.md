# DORA System – Repository & Technology Overview

The DORA platform is built as a set of modular repositories, aligned with the system architecture.  
Each repository has a clearly defined responsibility, technology stack, and dependency surface.

## DORA: Documentation Orchestration & Review Assistant

**Tag line:** Your intelligent partner in documentation.

---

## 1. `cloudsteak/dora-mcp-server`

Link: https://github.com/cloudsteak/dora-mcp-server

### **Purpose**
Central MCP integration server exposing `/mcp/...` endpoints and routing tool-calls to internal agents.

### **Responsibilities**
- Serve MCP capabilities, resources, tools, and tool-call endpoints  
- Act as a router between the DORA backend and internal agents  
- Hide agent implementation details behind a standardized MCP model  

### **Dependencies**
- DORA Backend (consumer of MCP)
- DORA Agents (internal services called by the MCP)

### **Technologies**
- Python 3.13  
- FastAPI  
- Uvicorn  
- httpx / aiohttp (for internal calls)  
- Docker  
- Helm chart (Kubernetes Deployment + ClusterIP)

---

## 2. `cloudsteak/dora-backend`  

Link: https://github.com/cloudsteak/dora-backend

### **Purpose**
Core intelligence layer of the platform, coordinating LLM interactions, RAG, and MCP client logic.

### **Responsibilities**
- Communicate with LLM providers (OpenAI, Gemini, Azure OpenAI, etc.)  
- Implement tool/function calling  
- Call MCP server `/mcp/...` endpoints  
- Integrate RAG (vector DB, document retrieval)  
- Provide streaming API endpoint to the frontend  
- Manage conversation state and agent orchestration  

### **Dependencies**
- MCP Server  
- DORA Frontend  
- Vector DB (optional for RAG)

### **Technologies**
- Python 3.13  
- FastAPI  
- httpx (async streaming)  
- LLM SDKs  
- Redis / Qdrant (optional RAG)  
- Docker  
- Helm chart

---

## 3. `cloudsteak/dora-frontend`

Link: https://github.com/cloudsteak/dora-frontend

### **Purpose**
The user-facing web-based chat interface.  
Runs fully in Python using NiceGUI to avoid NodeJS build chains.

### **Responsibilities**
- Provide a modern chat UI  
- Send user messages to the backend  
- Receive and display streamed responses  
- Act as the ONLY publicly exposed component  

### **Dependencies**
- DORA Backend (REST / streaming API)

### **Technologies**
- Python 3.x  
- NiceGUI (Vue 3 + Tailwind internally, but Python-only development)  
- httpx (streamed responses)  
- Docker  
- Helm chart  
- Kubernetes Ingress for public access  

---

## 4. `cloudsteak/dora-agents`

Link: https://github.com/cloudsteak/dora-agents

### **Purpose**
Collection of all internal microservice “agent” implementations consumed by the MCP server.

### **Responsibilities**
- Each folder contains an isolated agent microservice  
- Provide specialized functionality (diagram generation, documentation writing, code analysis, etc.)  
- Act as internal-only services reachable only via the MCP server  

### **Dependencies**
- MCP Server (caller)  

### **Technologies**
- Python 3.x  
- FastAPI (typical for each agent)  
- Specialized libraries (Mermaid renderer, Markdown tools, code parsers, etc.)  
- Docker per agent  
- Helm chart per agent  
- ClusterIP networking only (no Ingress)  

**Example folder structure:**

agents/
mermaid-agent/
documentation-agent/
code-analyzer-agent/


---

## 5. `cloudsteak/dora-gitops`

Link: https://github.com/cloudsteak/dora-gitops

### **Purpose**
Central GitOps repository for deploying the entire DORA platform via ArgoCD.

### **Responsibilities**
- Define ArgoCD Applications or ApplicationSets  
- Reference Helm charts from each repository  
- Maintain environment-specific values (`dev`, `prod`)  
- Act as the declarative source of truth for production deployments  

### **Dependencies**
- ArgoCD  
- All other DORA repositories  

### **Technologies**
- Kubernetes manifests  
- ArgoCD  
- Helm  
- Kustomize (optional)  

---

# Summary Table

| Repository | Purpose | Depends on | Technologies |
|-----------|----------|------------|--------------|
| **dora-mcp-server** | MCP central server | Backend, Agents | Python, FastAPI, Docker, Helm |
| **dora-backend** | LLM + RAG + MCP client | MCP server, Frontend | Python, FastAPI, httpx, Docker |
| **dora-frontend** | Chat UI | Backend | Python, NiceGUI, Docker |
| **dora-agents** | Internal agent microservices | MCP server | Python, FastAPI, Docker, Helm |
| **dora-gitops** | ArgoCD GitOps orchestration | All repos | ArgoCD, Helm, K8s |

