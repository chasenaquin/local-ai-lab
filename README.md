# local-ai-lab

# Final(ish) Version 1.0

## High Level Topology

```
[Local Machine/Docker Host (Still determining between one or multiple hosts.]
├── Ollama (Qwen3 models: 8b/14b/30b-MoE) (Possibly smaller models for sumarization if needed)
│   └── OpenAI-compatible API endpoint (http://localhost:11434)
├── Open WebUI → Chat interface to query llm, open web, and agents manually
├── Supabase → Vector DB for RAG (embeddings from notes/emails)
├── LangGraph/LangChain Agents (Python app/container) 
│   ├── Core Agent: "Secretary" (reasoning + tool router)
│   ├── Specialized Sub-Agents (CrewAI-style roles):
│   │   - Email Analyst (Outlook)
│   │   - Chat Summarizer (Teams)
│   │   - Task Manager (Jira)
│   │   - Knowledge Curator (Obsidian/GitHub/Filesystem) (other tools later like sharepoint & interal tooling)
│   └── Tools:
│       - Microsoft Graph API (emails/calendars/Teams channels)
│       - Jira REST API (issues/search)
│       - GitPython (commit/push to Obsidian repo)
│       - Filesystem read/write
│       - Vector search (Supabase)
├── Scheduler (cron/Python scripts)
│   ├── Daily/weekly runs: Fetch → Analyze → Summarize → Write to Obsidian
│   └── Triggers: Webhooks (if possible) or polling
└── Output: Obsidian Vault (synced to GitHub via Obsidian-Git plugin) & quite obviously Open Web UI.
```


```mermaid

graph TD
    subgraph Local Machine / Docker Host
        OLLAMA[Ollama<br/>Qwen3:8b/14b/30b Models<br/>OpenAI-compatible API]
        WEBUI[Open WebUI<br/>Manual Chat & Agent Testing]
        SUPABASE[Supabase<br/>Vector DB for RAG<br/>Obsidian notes, past summaries]
        AGENT[LangGraph / LangChain Agents<br/>Core Secretary Agent<br/>+ Specialized Sub-Agents]
        SCHEDULER[Scheduler<br/>Cron / Python Scripts<br/>Daily/Weekly Triggers]
    end

    subgraph Agents & Tools
        AGENT --> TOOL_OUTLOOK[Outlook Tool<br/>Microsoft Graph API<br/>Read/Summarize Emails]
        AGENT --> TOOL_TEAMS[Teams Tool<br/>Microsoft Graph API<br/>Read/Summarize Chats/Channels]
        AGENT --> TOOL_JIRA[Jira Tool<br/>REST API<br/>Issues, Updates, Tasks]
        AGENT --> TOOL_OBSIDIAN[Obsidian Tool<br/>GitPython<br/>Commit/Push Markdown]
        AGENT --> TOOL_FS[Filesystem Tool<br/>Read/Write Specific Folders]
        AGENT --> TOOL_VECTOR[Vector Search Tool<br/>Supabase RAG Retrieval]
    end

    subgraph External Services
        MS_GRAPH[Microsoft Graph<br/>Outlook + Teams]
        JIRA[Jira Cloud/Server]
        GITHUB[GitHub Repo<br/>Obsidian Vault Backup]
        OBSIDIAN_APP[Obsidian App<br/>Local Vault<br/>+ Obsidian-Git Plugin]
        LOCAL_FS[Local Filesystem<br/>Selected Folders]
    end

    %% Data Flows
    SCHEDULER -->|Daily/Weekly Trigger| AGENT
    WEBUI -->|Manual Query| AGENT

    AGENT -->|Fetch Emails/Chats| MS_GRAPH
    MS_GRAPH -->|Raw Data| AGENT
    AGENT -->|Fetch Issues| JIRA
    JIRA -->|Raw Data| AGENT

    AGENT -->|RAG Query| SUPABASE
    SUPABASE -->|Relevant Context| AGENT

    AGENT -->|Read Files| LOCAL_FS
    LOCAL_FS -->|Content| AGENT
    AGENT -->|Write Summaries/Reports| LOCAL_FS

    AGENT -->|Generate Markdown<br/>Commit & Push| TOOL_OBSIDIAN
    TOOL_OBSIDIAN -->|Auto-commit via Obsidian-Git| OBSIDIAN_APP
    OBSIDIAN_APP <-->|Sync| GITHUB

    %% Styling
    classDef core fill:#1e40af,stroke:#1e40af,color:#fff;
    classDef tool fill:#059669,stroke:#059669,color:#fff;
    classDef external fill:#7c3aed,stroke:#7c3aed,color:#fff;
    classDef storage fill:#dc2626,stroke:#dc2626,color:#fff;

    class OLLAMA,WEBUI,AGENT,SCHEDULER core;
    class TOOL_OUTLOOK,TOOL_TEAMS,TOOL_JIRA,TOOL_OBSIDIAN,TOOL_FS,TOOL_VECTOR tool;
    class MS_GRAPH,JIRA,GITHUB external;
    class SUPABASE,LOCAL_FS,OBSIDIAN_APP storage;
```

## Core Secretary Agent
```mermaid
graph TD
    TRIGGERS[Triggers<br/>Scheduler Cron<br/>Open WebUI Manual Query<br/>Future: Voice/Webhooks] --> CORE[Core Secretary Agent<br/>Supervisor - Qwen3:14b<br/>Planning & Orchestration]

    CORE --> EA[Email Analyst Agent]
    CORE --> CS[Chat Summarizer Agent]
    CORE --> TM[Task Manager Agent]
    CORE --> KC[Knowledge Curator Agent]

    EA --> CORE
    CS --> CORE
    TM --> CORE
    KC --> CORE

    CORE -->|Final Approval & Synthesis| KC
    KC --> OUTPUT[Output<br/>Obsidian Markdown<br/>Git Commit & Push]

    CORE --> TOOL_RAG[Vector RAG Tool<br/>Personal Knowledge Context]
    TOOL_RAG --> CORE

    style CORE fill:#8b5cf6,stroke:#fff,color:#fff
```


