## Key Projects and their Architectures

### Agentic RAG Application | 2025
- Developed Agentic RAG application using Claude Opus model with
    - AWS Bedrock Agent powering the planning, reasoning, and reflection loops, and conversation memory, and 
    - AWS OpenSearch for vector database knowledgebase, and semantic retrieval 
- Extended the RAG architecture with MCP-tool execution: Integrated the Agent with OpenAPI-based Bedrock Action Groups, MCP tool execution to enable secure, production-grade tool-augmented inference

**How the Agentic RAG Application Works**:

```mermaid
flowchart TD
  CH[(Conversation<br>History)]
  A[User Query]
  D[(Source <br>Knowledge Documents)]
  P[["Planning/<br>Orchestration-<br>(portion of agents.md)"]]
  T1[["External Integration <br>(portion of agents.md)"]]
  V[(Vector DB)]
  M[(Memory)]
  S[/Tool 1/]
  C[/Tool 2🚧/]
  R[["Reasoning/Reflection -<br>(portion of agents.md)"]]
  E[[LLM composes Answer]]
  F[Final Output]
  J{{Is answer complete<br>or confident?}}
  CH --> P
  A --> P -.-> T1
  D --chunking & embedding--> V
  T1 --prompt+query+history<br><br>chunking & embedding--> V
  T1 --decide<br>to search<br>in memory--> M
  T1 --decide<br>to invoke tool--> S
  T1 --....--> C
  V --Similarity computation:<br>Relevant source Docs--> R
  M --> R
  S --> R
  C --> R
  R --> J
  J -- No --> P
  J -- Yes --> E --> F
```

**How the MCP is Integrated in the Agentic RAG Application**

```mermaid
sequenceDiagram
    autonumber

    %% =====================================================
    %% Experience Layer
    %% =====================================================
    participant U as User
    participant PP as AI Agent<br><br>(Bedrock Agentic AI Application)<br>Planning and Orchestration Engine

    %% =====================================================
    %% Model Layer
    %% =====================================================
    participant BR as LLM Model<br><br>(Bedrock Claude Opus 4 Model)

    %% =====================================================
    %% Tooling Layer
    %% =====================================================
    participant OA as OpenAPI Tool(s) Schema<br><br>(Bedrock Action Group)
    participant MC as MCP Client<br><br>(Tool Adapter Lambda)
    participant MS as MCP Server<br><br>(RDS DB READ Query Tool)

    %% =====================================================
    %% Data Layer
    %% =====================================================
    participant DB as External Tool<br><br>(Amazon RDS)

    %% -----------------------------------------------------
    %% Tool Discovery (Model-side)
    %% -----------------------------------------------------
    Note over PP,OA: 0) Tool Discovery
    OA ->> PP: OpenAPI Tools Schema sent to Application

    %% =====================================================
    %% 1) Agent Reasoning and Planning
    %% =====================================================
    
    U->>PP: Natural language query

    Note over PP,BR: 1) Agent Reasoning and Planning

    PP->>BR: Prompt + instructions + OpenAPI tool schema
    BR-->>PP: Tool selection decision<br/>(tool name + arguments)

    %% =====================================================
    %% 2) Tool Invocation (Control Plane → Data Plane)
    %% =====================================================
    Note over PP,MC: 2) Tool Invocation<br/>Bedrock Action Group Event

    PP->>MC: Action Group Event<br/>(OpenAPI-based tool + args)

    %% -----------------------------------------------------
    %% Schema Translation Boundary
    %% -----------------------------------------------------
    Note over MC: MCP Client responsibility:<br/>Convert OpenAPI / Action Group event<br/>→ MCP JSON tool schema

    %% =====================================================
    %% 3) MCP Tool Execution
    %% =====================================================
    Note over MC,MS:3) MCP Tool Execution

    MC->>MS: call_tool(tool, MCP_JSON_args)

    %% =====================================================
    %% 4) Data Access
    %% =====================================================
    Note over MS,DB:4) PostgreSQL READ Operation

    MS->>DB: SQL SELECT / READ query
    DB-->>MS: Structured query result

    %% =====================================================
    %% 5) Tool Result Return
    %% =====================================================
    MS-->>MC: MCP tool result (JSON)
    MC-->>PP: Normalized tool response

    %% =====================================================
    %% 6) Final Response Generation
    %% =====================================================
    Note over PP,BR: 6) Final Response Generation

    PP->>BR: Tool result + conversation history
    BR-->>PP: Final response (streamed)

    PP-->>U: Streamed response to user
```