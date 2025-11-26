# Local RAG Chatbot with n8n

A self-hosted Docker Compose setup for building intelligent RAG (Retrieval-Augmented Generation) chatbots using n8n, Ollama, and Qdrant - completely local and private.

![n8n RAG Chatbot Architecture](https://img.shields.io/badge/n8n-RAG%20Chatbot-orange)

## 🎯 What's This?

This project provides a complete, local AI infrastructure for building RAG chatbots that can:
- Answer questions based on your documents
- Keep all data private (no cloud APIs)
- Run entirely on your own hardware
- Scale from CPU to GPU setups

### Components Included

✅ **[n8n](https://n8n.io/)** - Low-code workflow automation platform with AI capabilities

✅ **[Ollama](https://ollama.com/)** - Run large language models (LLMs) locally

✅ **[Qdrant](https://qdrant.tech/)** - Vector database for semantic search and embeddings

✅ **[PostgreSQL](https://www.postgresql.org/)** - Reliable database for n8n data persistence

### What You Can Build

⭐️ **Document Q&A Systems** - Chat with your PDFs, docs, and knowledge bases

⭐️ **Private Customer Support Bots** - No data leaves your infrastructure

⭐️ **Internal Knowledge Assistants** - Help employees find information instantly

⭐️ **Research Assistants** - Analyze and summarize documents locally

## 📋 Prerequisites

- Docker and Docker Compose installed
- At least 8GB RAM (16GB recommended)
- 20GB free disk space (for models)
- For GPU: NVIDIA GPU with Docker GPU support or AMD GPU with ROCm

## 🚀 Installation

### 1. Clone and Configure

- Clone this repository
- Create .env file based on .env example

**Important:** Edit the `.env` file and update these values:
```bash
POSTGRES_USER=user  
POSTGRES_PASSWORD=password
POSTGRES_DB=n8n
N8N_ENCRYPTION_KEY=key
N8N_USER_MANAGEMENT_JWT_SECRET=jwt_secret
N8N_DEFAULT_BINARY_DATA_MODE=filesystem
```

### 2. Choose Your Setup

#### 🖥️ CPU-Only (Mac, Windows, Linux without GPU)

```bash
docker compose --profile cpu up -d
```

#### 🎮 NVIDIA GPU

```bash
docker compose --profile gpu-nvidia up -d
```

> **Note:** First time GPU users should follow the [Ollama Docker GPU setup guide](https://github.com/ollama/ollama/blob/main/docs/docker.md).

#### 🔴 AMD GPU (Linux only)

```bash
docker compose --profile gpu-amd up -d
```

#### 🍎 Mac with Apple Silicon

Ollama can't access the GPU inside Docker on Mac. Two options:

**Option 1: Run everything on CPU**
```bash
docker compose --profile cpu up -d
```

**Option 2: Run Ollama natively (faster)**

1. Install Ollama from [ollama.com](https://ollama.com)
2. Pull models locally:
   ```bash
   ollama pull llama3.2
   ollama pull nomic-embed-text
   ```
3. Update `.env`:
   ```bash
   OLLAMA_HOST=host.docker.internal:11434
   ```
4. Start containers:
   ```bash
   docker compose up -d
   ```
5. After startup, update Ollama credentials in n8n:
   - Go to http://localhost:5678/home/credentials
   - Click "Local Ollama service"
   - Change base URL to `http://host.docker.internal:11434/`

### 3. Verify Installation

```bash
# Check all services are running
docker compose ps

# View logs
docker compose logs -f

# Wait for model downloads to complete (first time only)
docker logs ollama-pull-llama
docker logs ollama-pull-embeddings
```

## 🎮 Getting Started

### First Time Setup

1. **Access n8n**: Open http://localhost:5678 in your browser
2. **Create your account**: Set up your n8n credentials (first-time only)
3. **Import demo workflows**: The setup includes pre-configured workflows in `/n8n/demo-data/workflows`

### Building Your First RAG Chatbot

1. **Prepare your documents**: Place files in the `./shared` folder
2. **Access the folder in n8n**: Use path `/data/shared` in file nodes
3. **Create a workflow** with these components:
   - **Document Loader**: Read files from `/data/shared`
   - **Text Splitter**: Break documents into chunks
   - **Embeddings**: Use Ollama with `nomic-embed-text`
   - **Vector Store**: Store in Qdrant
   - **AI Agent**: Answer questions using the stored knowledge

### Key Configurations

#### Ollama Credentials in n8n

- **Base URL**: `http://ollama:11434` (or `http://ollama-cpu:11434`)
- **API Key**: Leave empty (no authentication needed)

#### Qdrant Credentials

- **Host**: `http://qdrant:6333`
- **API Key**: Leave empty (default setup has no authentication)

## 📁 Directory Structure

```
.
├── docker-compose.yml       # Main orchestration file
├── .env                     # Environment variables (passwords, keys)
├── .env.example            # Template for environment variables
├── shared/                 # Shared folder accessible at /data/shared in n8n
├── n8n/
│   └── demo-data/
│       ├── credentials/    # Pre-configured credentials
│       └── workflows/      # Demo workflows
└── README.md
```

## 🔧 Configuration

### Available Models

By default, the setup downloads:
- **llama3.2** - Main language model
- **nomic-embed-text** - Embedding model for vector search

To use different models, modify the docker-compose.yml:

```yaml
command:
  - "-c"
  - "sleep 3; ollama pull <your-model-name>"
```

### Accessing Local Files

The `./shared` folder is mounted to `/data/shared` inside n8n. Use this path in nodes that read/write files:

- **Read/Write Files from Disk**
- **Local File Trigger**
- **Execute Command**

Example:
```
Local path:  ./shared/documents/my-file.pdf
n8n path:    /data/shared/documents/my-file.pdf
```

## 🔄 Upgrading

### Stop current services
```bash
docker compose down
```

### Pull latest images
```bash
docker compose pull
```

### Restart with your profile
```bash
# CPU
docker compose --profile cpu up -d

# NVIDIA GPU
docker compose --profile gpu-nvidia up -d

# AMD GPU
docker compose --profile gpu-amd up -d
```

## 🐛 Troubleshooting

### n8n Can't Connect to Ollama

**Problem**: Error "Cannot connect to Ollama"

**Solution**: Use the Docker service name, not `localhost`:
- ✅ Correct: `http://ollama:11434`
- ❌ Wrong: `http://localhost:11434`

### Ollama Not Starting

**Problem**: Ollama container not found

**Solution**: Make sure you started with a profile:
```bash
docker compose --profile cpu up -d
```

### Models Not Downloaded

**Problem**: Ollama says model not found

**Solution**: Check download logs:
```bash
docker logs ollama-pull-llama
docker logs ollama-pull-embeddings
```

Wait for downloads to complete (can take 5-10 minutes first time).

### Out of Memory

**Problem**: Services crash or slow performance

**Solution**: 
- Increase Docker memory limit (Docker Desktop → Settings → Resources)
- Use smaller models (e.g., `llama3.2:1b` instead of default)
- Close other applications

## 📚 Learn More

### n8n AI Documentation
- [AI Agents Tutorial](https://docs.n8n.io/advanced-ai/intro-tutorial/)
- [LangChain in n8n](https://docs.n8n.io/advanced-ai/langchain/langchain-n8n/)
- [Vector Databases Explained](https://docs.n8n.io/advanced-ai/examples/understand-vector-databases/)

### Example Workflows
- [Chat with PDF Documents](https://n8n.io/workflows/2165-chat-with-pdf-docs-using-ai-quoting-sources/)
- [AI Agent Chat](https://n8n.io/workflows/1954-ai-agent-chat/)
- [Document Q&A with Citations](https://n8n.io/workflows/2026-ai-chat-with-any-data-source-using-the-n8n-workflow-tool/)

## 🔒 Security Notes

- Change default passwords in `.env` file
- Keep `.env` file out of version control (already in `.gitignore`)
- This setup has no authentication on Ollama/Qdrant (safe for local use)
- For production, add authentication and use HTTPS

## 📊 Resource Usage

### Minimum Requirements
- **CPU**: 4 cores
- **RAM**: 8GB
- **Storage**: 20GB

### Recommended for Best Performance
- **CPU**: 8+ cores or GPU
- **RAM**: 16GB+
- **Storage**: 50GB SSD

## 💬 Support

- **n8n Community**: [community.n8n.io](https://community.n8n.io/)
- **Ollama Discord**: [discord.gg/ollama](https://discord.gg/ollama)
- **Qdrant Discord**: [qdrant.to/discord](https://qdrant.to/discord)

## 📜 License

This project is open-source. See individual component licenses:
- n8n: [n8n License](https://github.com/n8n-io/n8n/blob/master/LICENSE.md)
- Ollama: MIT License
- Qdrant: Apache 2.0 License

## 🙏 Credits

Based on the [n8n Self-hosted AI Starter Kit](https://github.com/n8n-io/self-hosted-ai-starter-kit)

---

**Happy Building! 🚀**