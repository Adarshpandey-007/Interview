# 🎯 FloatChat / OceanGPT — Complete Interview Preparation Guide

> **Use this document to prepare for any interview questions about your FloatChat project.**
> Every section is written in simple, explainable language so you can confidently describe what you built, why you built it, and how it works.

---

## 📋 Table of Contents

1. [Project Overview — The 30-Second Pitch](#1-project-overview--the-30-second-pitch)
2. [Problem Statement & Motivation](#2-problem-statement--motivation)
3. [System Architecture — The Big Picture](#3-system-architecture--the-big-picture)
4. [Frontend Technologies (Next.js + React)](#4-frontend-technologies-nextjs--react)
5. [Backend Technologies (Python + FastMCP)](#5-backend-technologies-python--fastmcp)
6. [AI/ML & LLM Integration (Gemini + Agentic AI)](#6-aiml--llm-integration-gemini--agentic-ai)
7. [Model Context Protocol (MCP) — Deep Dive](#7-model-context-protocol-mcp--deep-dive)
8. [RAG (Retrieval-Augmented Generation) & Vector Search](#8-rag-retrieval-augmented-generation--vector-search)
9. [Database Layer (PostgreSQL + PostGIS + ChromaDB)](#9-database-layer-postgresql--postgis--chromadb)
10. [Data Ingestion Pipeline (NetCDF Processing)](#10-data-ingestion-pipeline-netcdf-processing)
11. [State Management (Zustand)](#11-state-management-zustand)
12. [Data Visualization (Leaflet Maps + Plotly Charts)](#12-data-visualization-leaflet-maps--plotly-charts)
13. [API Design & Route Architecture](#13-api-design--route-architecture)
14. [LLMOps — API Key Rotation & Rate Limiting](#14-llmops--api-key-rotation--rate-limiting)
15. [Multi-Agent Architecture (Specialist Agents)](#15-multi-agent-architecture-specialist-agents)
16. [Intent Routing & NLP Classification](#16-intent-routing--nlp-classification)
17. [Error Handling & Resilience](#17-error-handling--resilience)
18. [Testing Strategy (Jest + React Testing Library)](#18-testing-strategy-jest--react-testing-library)
19. [UI/UX Design System & Styling](#19-uiux-design-system--styling)
20. [DevOps & Deployment (Docker + Vercel)](#20-devops--deployment-docker--vercel)
21. [Security Considerations](#21-security-considerations)
22. [Most Likely Interview Questions & Answers](#22-most-likely-interview-questions--answers)
23. [Technical Deep-Dive Questions](#23-technical-deep-dive-questions)
24. [Behavioral / Scenario Questions](#24-behavioral--scenario-questions)

---

## 1. Project Overview — The 30-Second Pitch

> **"FloatChat is a Multi-Agent Conversational Ocean Data Explorer that I built for the Smart India Hackathon (SIH) Problem Statement 25040. It lets users ask natural language questions about the ocean — like 'What's the temperature near the Maldives?' — and it automatically fetches real-time data from ARGO floats worldwide, plots it on maps and charts, and gives actionable insights. The backend uses a Python MCP Server with Gemini AI for intelligent tool-calling, and the frontend is built with Next.js 14 with interactive Leaflet maps, Plotly depth plots, and a beautiful ocean-themed glassmorphic UI."**

### Key Points to Mention:
- **Full-stack project** — Next.js 14 frontend + Python backend
- **AI-powered** — Google Gemini LLM with tool-calling (agentic AI)
- **Real data** — ARGO float ocean data from Argovis API, OceanOPS, NOAA ERDDAP
- **Production-grade** — Error boundaries, rate limiting, API key rotation, 42 tests passing
- **SIH Problem Statement 25040** — Conversational Ocean Data Explorer

---

## 2. Problem Statement & Motivation

### What is SIH 25040?
The Smart India Hackathon (SIH) Problem Statement 25040 asks teams to build a **Conversational Ocean Data Explorer** that can:
- Ingest binary ocean data files (NetCDF format from ARGO floats)
- Store them in a relational + vector database
- Let users query the data using natural language
- Visualize the data with maps, charts, and tables
- Use AI agents to provide intelligent analysis

### What are ARGO Floats?
> **Simple explanation:** "ARGO floats are small robotic sensors that float in the ocean, diving up to 2000 meters deep. They measure temperature, salinity, and pressure at different depths. There are about 4000 of them worldwide, and they surface every 10 days to transmit data via satellite. Each float has a unique WMO ID (like a license plate number)."

### Why is this important?
- Climate change monitoring
- Ocean current tracking
- Fish migration patterns
- Maritime safety and navigation
- Coastal development planning

---

## 3. System Architecture — The Big Picture

```
┌─────────────────────────────────────────────────┐
│              USER (Web Browser)                   │
│  Next.js 14 App Router + TailwindCSS + Zustand    │
└────────────┬────────────────────────┬─────────────┘
             │ HTTP POST              │ UI Updates
             │ (JSON)                 │ (React State)
┌────────────▼────────────────────────▼─────────────┐
│         Next.js API Routes (Server-Side)           │
│  /api/query — Main LLM + MCP orchestration         │
│  /api/floats, /api/profiles — Data endpoints        │
│  Rate Limiter + API Key Rotation                    │
└────────────┬────────────────────────┬──────────────┘
             │ SSE Transport           │ PostgreSQL
             │ (MCP Protocol)          │ (pg driver)
┌────────────▼──────────────┐   ┌─────▼───────────┐
│  Python FastMCP Server    │   │  PostgreSQL +    │
│  ├ get_ocean_profile()    │   │  PostGIS DB      │
│  ├ search_ocean_area()    │   │  (Docker)        │
│  ├ check_float_health()   │   └──────────────────┘
│  ├ search_erddap()        │
│  └ system_health_check()  │
└─────┬──────┬──────┬───────┘
      │      │      │
┌─────▼──┐ ┌▼────┐ ┌▼──────┐
│Argovis │ │Ocean│ │ERDDAP │
│  API   │ │OPS  │ │ NOAA  │
└────────┘ └─────┘ └───────┘
```

### How to explain this flow:
1. **User types a question** in the chat interface (e.g., "Show me floats near India")
2. **Next.js API route** receives the request, applies rate limiting, and connects to the Python MCP server
3. **MCP Client** (in Next.js) sends the query to the Python MCP Server over **SSE (Server-Sent Events)**
4. **Gemini LLM** decides which tools to call (e.g., `search_ocean_area`)
5. **Python MCP Server** executes the tool, fetching real data from Argovis/OceanOPS/ERDDAP APIs
6. **Tool results** are fed back to Gemini for synthesis into a human-readable answer
7. **Response** flows back to the frontend with visualization commands (center map, switch tab, etc.)
8. **React/Zustand** updates the UI — map centers, chart loads, table populates

---

## 4. Frontend Technologies (Next.js + React)

### Next.js 14 (App Router)

> **What is Next.js?**
> "Next.js is a React framework that gives us server-side rendering, API routes, file-based routing, and optimized production builds out of the box. I used version 14 with the **App Router** — the newest routing paradigm that uses the `app/` directory instead of the older `pages/` directory."

#### Key Next.js features used:

| Feature | Where Used | Why |
|---------|-----------|-----|
| **App Router** | `src/app/` directory | Modern file-based routing with layouts |
| **Server Components** | Page components (e.g., `page.tsx`) | Rendered on server for better performance |
| **Client Components** | `"use client"` directive | Interactive components like ChatPanel, MapPanel |
| **API Routes** | `src/app/api/query/route.ts` | Serverless backend endpoints |
| **Dynamic Imports** | `next/dynamic` for Leaflet & Plotly | Avoid SSR issues with browser-only libraries |
| **Error Boundaries** | `error.tsx`, `global-error.tsx` | Graceful error handling at route level |
| **Loading States** | `loading.tsx` | Skeleton screens while pages load |
| **Suspense** | `<Suspense>` wrapper | Streaming and lazy loading |
| **Metadata** | Layout exports | SEO optimization |

#### Why App Router over Pages Router?
> "The App Router is the recommended way in Next.js 14. It supports React Server Components which reduce the JavaScript sent to the browser, has built-in layouts that persist across navigation, and has streaming support with Suspense. It also makes the routing more intuitive — each folder in `app/` is a route."

### React 18

#### Key React features used:
- **Hooks**: `useState`, `useEffect`, `useRef`, `useMemo` for state and lifecycle management
- **Suspense**: Lazy loading of heavy components
- **`dangerouslySetInnerHTML`**: Rendering markdown content from LLM responses (with `marked` library)
- **`AbortController`**: Cancelling pending API requests when component unmounts

### TypeScript
> "I used TypeScript throughout the entire project for type safety. It catches bugs at compile time and makes the codebase self-documenting. For example, I defined interfaces like `ChatMessage`, `RealProfile`, `FloatSummary` to ensure data consistency across components."

Key TypeScript patterns used:
- **Interfaces** for data shapes (`ChatMessage`, `RealProfile`, `Profile`)
- **Generic types** (e.g., `nearestPoint<T extends { lat: number; lon: number }>`)
- **Union types** for states (`'map' | 'plot' | 'table' | 'none'`)
- **Type guards** and null checks

---

## 5. Backend Technologies (Python + FastMCP)

### Python MCP Server (`backend/mcp_server.py`)

> **What is the backend?**
> "The backend is a Python server built using FastMCP that exposes ocean data tools via the Model Context Protocol. It acts as a bridge between the AI model and real-world ocean data APIs. When Gemini decides it needs data, it calls one of our tools, and the Python server fetches that data from external APIs like Argovis."

### Python Libraries Used:

| Library | Purpose | Interview Explanation |
|---------|---------|----------------------|
| `mcp.server.fastmcp` | MCP server framework | "This is the official MCP SDK for Python. It lets me define tools as simple Python functions decorated with `@mcp.tool()`, and it handles the protocol communication automatically." |
| `requests` | HTTP client | "For calling external ocean data APIs (Argovis, OceanOPS, ERDDAP)" |
| `python-dotenv` | Environment variables | "Loads API keys from `.env` files so we don't hardcode secrets" |
| `chromadb` | Vector database | "Stores document embeddings for semantic search over ARGO metadata" |
| `pandas` | Data processing | "Used in the ingestion pipeline for processing NetCDF data arrays" |
| `langchain` | LLM orchestration | "Provides abstractions for chaining LLM calls with tool results" |

### MCP Tools Defined:

```python
@mcp.tool()
def get_ocean_profile(wmo_id: str, variables: str = "temperature,salinity") -> str:
    """Fetch vertical ocean profiles for a specific ARGO float by WMO ID."""
    # Calls Argovis API, parses response, returns JSON summary

@mcp.tool()
def search_ocean_area(lat_min, lat_max, lon_min, lon_max, variables) -> str:
    """Search for ARGO float profiles within a geographic bounding box."""
    # Converts coordinates to Argovis box format, groups by float

@mcp.tool()
def check_float_health(wmo_id: str) -> str:
    """Check operational status and hardware health of an ARGO float."""
    # Calls OceanOPS API for deployment info, sensor status

@mcp.tool()
def search_erddap(variables, dataset_id, time_min, lat/lon bounds) -> str:
    """Fallback query to NOAA ERDDAP for bulk ocean observations."""
    # Builds constraint URL, queries ERDDAP tabledap

@mcp.tool()
def system_health_check() -> str:
    """Verify if the MCP server is online and configured."""
    # Returns server status and API key configuration
```

---

## 6. AI/ML & LLM Integration (Gemini + Agentic AI)

### Google Gemini 2.5 Flash

> **What LLM do you use and why?**
> "I use Google Gemini 2.5 Flash because it has native function-calling support, which is essential for our agentic architecture. When a user asks 'What's the temperature near India?', Gemini doesn't just guess — it recognizes it needs data and calls our `search_ocean_area` tool with the right coordinates. It's also faster and cheaper than GPT-4 for our use case."

### What is Agentic AI?

> "Agentic AI means the LLM doesn't just generate text — it can take **actions**. In our project, Gemini acts as an agent that can:
> 1. **Reason** about what data is needed to answer a question
> 2. **Plan** which tools to call and in what order
> 3. **Execute** tools via MCP (function calling)
> 4. **Synthesize** the results into a human-readable answer
> 5. **Control the UI** — automatically switching tabs, centering maps, loading profiles"

### Tool-Calling Flow (Step by Step):

```
User: "Show me floats near the Maldives"
                    ↓
Gemini thinks: "This is a location query. I need to call search_ocean_area 
               with Maldives coordinates (lat 2-8, lon 71-76)"
                    ↓
Gemini outputs: functionCall { name: "search_ocean_area", 
                               args: { lat_min: 2, lat_max: 8, 
                                       lon_min: 71, lon_max: 76 } }
                    ↓
Our code executes the tool via MCP → Python server calls Argovis API
                    ↓
Tool result: JSON with 15 floats found in the area
                    ↓
Result fed back to Gemini for synthesis
                    ↓
Gemini outputs: "I found 15 ARGO floats near the Maldives. The closest one 
                 is float 2901234 at 4.5°N, 73.2°E, last observed on..."
```

### Multi-Turn Tool Execution Loop:
> "The system supports up to 3 iterations of tool calling. This means Gemini can call one tool, look at the results, decide it needs more information, and call another tool. For example, it might first `search_ocean_area` to find floats, then `get_ocean_profile` to get detailed data for the closest float."

```typescript
// From route.ts — the tool execution loop
let iterations = 0;
const MAX_ITERATIONS = 3;

while (iterations < MAX_ITERATIONS && mcpClient) {
  const calls = response.functionCalls?.() || [];
  if (calls.length === 0) break;  // No more tools needed
  
  iterations++;
  // Execute each tool via MCP
  for (const call of calls) {
    const mcpResult = await mcpClient.callTool({
      name: call.name,
      arguments: call.args,
    });
    // Collect results...
  }
  // Feed results back to Gemini
  result = await chat.sendMessage(functionResponses);
  response = result.response;
}
```

### System Prompt Design:
> "The system prompt is crucial for agent behavior. I designed it to make Gemini behave as an 'Ocean Advisor' with specific instructions like:
> - Use layman terms to explain scientific data
> - Structure answers as 'Finding → Impact → Recommendation'
> - Proactively use tools (don't just guess)
> - If no data found, provide regional estimates and clearly mark them"

---

## 7. Model Context Protocol (MCP) — Deep Dive

### What is MCP?

> "The Model Context Protocol (MCP) is an open standard created by Anthropic that standardizes how AI applications connect to external data sources and tools. Think of it like USB for AI — just like USB provides a standard way to connect any device to any computer, MCP provides a standard way to connect any AI model to any data source."

### Key MCP Concepts:

| Concept | Explanation |
|---------|-------------|
| **MCP Server** | "The Python backend that hosts the tools. It registers functions like `get_ocean_profile` and makes them available via the MCP protocol." |
| **MCP Client** | "The Next.js API route that connects to the MCP Server and invokes tools on behalf of the LLM." |
| **Tools** | "Functions that the LLM can call. Each tool has a name, description, and parameter schema — similar to REST API endpoints." |
| **SSE Transport** | "Server-Sent Events — the communication protocol between the MCP Client and Server. It's a persistent HTTP connection where the server pushes events to the client." |
| **Function Declarations** | "JSON schemas describing the tools that get sent to Gemini so it knows what tools are available and what parameters they expect." |

### How MCP Works in Our Project:

```
Step 1: Next.js connects to Python MCP Server over SSE
        transport = new SSEClientTransport(new URL("http://127.0.0.1:8000/sse"))
        mcpClient = new Client({ name: "floatchat-nextjs", version: "1.0.0" })
        await mcpClient.connect(transport)

Step 2: Next.js fetches the list of available tools
        const toolsResp = await mcpClient.listTools()
        // Returns: [get_ocean_profile, search_ocean_area, check_float_health, ...]

Step 3: Convert MCP tool schemas to Gemini format
        // MCP uses JSON Schema format, Gemini uses its own format
        const geminiDeclarations = mcpToolsToGeminiDeclarations(mcpTools)

Step 4: Gemini decides to call a tool → Next.js forwards it to MCP
        const mcpResult = await mcpClient.callTool({
          name: "search_ocean_area",
          arguments: { lat_min: 2, lat_max: 8, lon_min: 71, lon_max: 76 }
        })

Step 5: Python MCP Server executes the tool and returns result
```

### Why MCP Instead of Direct API Calls?

> "MCP provides several advantages:
> 1. **Decoupling** — The LLM doesn't know about HTTP endpoints, it just knows tool names and parameters. If we change the API, we only update the MCP server.
> 2. **Standardization** — Any MCP-compatible model can use our tools without code changes.
> 3. **Security** — The MCP Server controls what the LLM can and cannot do. Tools are read-only by design.
> 4. **Composability** — We can add new tools just by adding `@mcp.tool()` decorated functions."

---

## 8. RAG (Retrieval-Augmented Generation) & Vector Search

### What is RAG?

> "RAG stands for Retrieval-Augmented Generation. Instead of relying only on what the LLM was trained on, we first **retrieve** relevant data from our database, then **augment** the LLM's prompt with that data, and finally **generate** a more accurate answer. This is crucial for ocean data because Gemini wasn't trained on real-time ARGO float measurements."

### How RAG Works in FloatChat:

```
Traditional LLM:
    User → LLM → Answer (based only on training data, may be outdated/wrong)

RAG in FloatChat:
    User → LLM decides to call tools → MCP fetches real data → 
    LLM synthesizes answer with real data → Accurate answer
```

### Vector Search with ChromaDB

> **What is ChromaDB?**
> "ChromaDB is an open-source vector database. Instead of storing data as rows and columns (like SQL), it stores data as mathematical vectors (embeddings). This allows us to do **semantic search** — finding data by meaning rather than exact keyword matches."

#### How We Use ChromaDB:
1. **Ingestion**: Parse NetCDF files → create text descriptions of each ARGO float profile
2. **Embedding**: Convert descriptions to numerical vectors
3. **Indexing**: Store in ChromaDB collection `argo_metadata` with cosine similarity
4. **Search**: When user asks "warm waters near Arabian Sea", find profiles semantically matching that description

```python
# Example: Ingesting into ChromaDB (from ingest_argo.py)
collection.add(
    documents=[profile_description],     # Text summary
    metadatas=[{"wmo_id": "4901283", "lat": 12.5, "lon": 73.2}],
    ids=[profile_id]
)

# Example: Searching ChromaDB
results = collection.query(
    query_texts=["warm waters with high salinity"],
    n_results=5
)
```

### Text-Based Ranking Fallback (`toolExecutor.ts`):

> "When ChromaDB is not available (it runs as a separate service), I built a fallback ranking system in TypeScript that scores profiles based on keyword matching. For example, if the user searches for 'Arabian Sea', profiles between 0-30°N and 50-80°E get a higher score."

---

## 9. Database Layer (PostgreSQL + PostGIS + ChromaDB)

### PostgreSQL with PostGIS Extension

> **What is PostGIS?**
> "PostGIS is a spatial extension for PostgreSQL that adds support for geographic objects. It lets us store coordinates as geometric points and run spatial queries like 'find all floats within 100km of this location' directly in SQL."

### Database Schema:

```sql
-- Floats table: stores each ARGO float device
floats(
    id SERIAL PRIMARY KEY,
    wmo_id VARCHAR UNIQUE,       -- e.g., '4901283'
    launch_date TIMESTAMP,
    last_observation TIMESTAMP,
    geom GEOMETRY(POINT, 4326),  -- PostGIS spatial point
    metadata_json JSONB          -- Flexible metadata storage
)

-- Profiles table: each measurement cycle of a float
profiles(
    id SERIAL PRIMARY KEY,
    float_id INTEGER REFERENCES floats(id),
    cycle_number INTEGER,
    timestamp TIMESTAMP,
    latitude DOUBLE PRECISION,
    longitude DOUBLE PRECISION,
    location_geom GEOMETRY(POINT, 4326),
    min_depth DOUBLE PRECISION,
    max_depth DOUBLE PRECISION,
    qc_status VARCHAR            -- Quality Control flag
)

-- Measurements table: depth-indexed readings
measurements(
    id SERIAL PRIMARY KEY,
    profile_id INTEGER REFERENCES profiles(id),
    depth DOUBLE PRECISION,
    temperature DOUBLE PRECISION,
    salinity DOUBLE PRECISION
)

-- Profile statistics (pre-computed aggregates)
profile_stats(
    profile_id INTEGER PRIMARY KEY,
    mean_temp DOUBLE PRECISION,
    mean_salinity DOUBLE PRECISION,
    surface_temp DOUBLE PRECISION,
    mixed_layer_depth DOUBLE PRECISION
)
```

### Spatial Indexing:

```sql
-- GIST index on geometry columns for fast spatial queries
CREATE INDEX idx_floats_geom ON floats USING GIST(geom);
CREATE INDEX idx_profiles_location ON profiles USING GIST(location_geom);

-- B-Tree indexes on timestamps for temporal queries
CREATE INDEX idx_profiles_timestamp ON profiles(timestamp);
```

### PostGIS Spatial Queries:

```sql
-- Find nearest floats using KNN distance operator (<->)
SELECT f.wmo_id, p.latitude, p.longitude,
  ROUND(
    (ST_Distance(
      ST_SetSRID(ST_MakePoint(p.longitude, p.latitude), 4326)::geography,
      ST_SetSRID(ST_MakePoint($2, $1), 4326)::geography
    ) / 1000.0)::numeric, 1
  ) AS dist_km
FROM floats f
JOIN profiles p ON p.float_id = f.id
ORDER BY ... <-> ST_SetSRID(ST_MakePoint($2, $1), 4326)::geography
LIMIT 5;
```

> **How to explain this:** "The `<->` operator is PostGIS's KNN (K-Nearest Neighbor) distance operator. Combined with the GIST spatial index, it efficiently finds the 5 closest floats to any given coordinate without scanning every row. The `ST_Distance` function calculates the actual distance in meters using the geographic (spheroidal) model, which I then convert to kilometers."

### Docker Compose for Database:

```yaml
services:
  db:
    image: postgis/postgis:16-3.4      # PostgreSQL 16 + PostGIS 3.4
    environment:
      POSTGRES_DB: floatchat
      POSTGRES_USER: floatchat
    ports:
      - '5433:5432'                     # Expose on non-default port
    healthcheck:
      test: ['CMD-SHELL', 'pg_isready -U floatchat -d floatchat']

  chroma:
    image: chromadb/chroma:latest       # ChromaDB vector store
    ports:
      - '8000:8000'
```

---

## 10. Data Ingestion Pipeline (NetCDF Processing)

### What is NetCDF?

> "NetCDF (Network Common Data Form) is a binary file format used by scientists to store multi-dimensional array data. ARGO float data comes in `.nc` files that contain arrays of temperature, salinity, and pressure at different depths. We need to parse these binary files and load them into our database."

### Ingestion Pipeline Steps:

```
1. Read .nc file with Python netCDF4 library
        ↓
2. Extract variables: PRES (pressure), TEMP (temperature), PSAL (salinity)
        ↓
3. Filter invalid/masked coordinates (QC filtering)
        ↓
4. Generate text descriptions for ChromaDB indexing
        ↓
5. Store embeddings in ChromaDB (argo_metadata collection)
        ↓
6. Map data to PostgreSQL relational schema
        ↓
7. Insert into floats → profiles → measurements tables
```

### Code Flow (ingest_argo.py):

```python
import netCDF4 as nc
import pandas as pd
import chromadb

# Parse binary NetCDF file
dataset = nc.Dataset("argo_float_4901283.nc")

# Extract measurement arrays
pressure = dataset.variables['PRES'][:]    # Depth levels
temperature = dataset.variables['TEMP'][:]  # Temperature readings
salinity = dataset.variables['PSAL'][:]     # Salinity readings
latitude = dataset.variables['LATITUDE'][:]
longitude = dataset.variables['LONGITUDE'][:]

# QC Filter: Remove masked/invalid values
valid_mask = ~temperature.mask & ~salinity.mask

# Store in ChromaDB for semantic search
client = chromadb.PersistentClient(path="./chroma_data")
collection = client.get_or_create_collection("argo_metadata")
collection.add(
    documents=[description],
    metadatas=[{"wmo_id": wmo_id, "lat": lat, "lon": lon}],
    ids=[profile_id]
)

# Store in PostgreSQL for structured queries
cursor.execute("INSERT INTO floats (wmo_id, ...) VALUES (%s, ...)", ...)
cursor.execute("INSERT INTO profiles (float_id, ...) VALUES (%s, ...)", ...)
cursor.execute("INSERT INTO measurements (profile_id, depth, temp, sal) VALUES (%s, %s, %s, %s)", ...)
```

---

## 11. State Management (Zustand)

### What is Zustand?

> "Zustand is a lightweight state management library for React. I chose it over Redux because it's simpler (no boilerplate), has a smaller bundle size, and works perfectly with React hooks. Our entire app state — chat messages, active tab, map coordinates, loaded profiles — is managed in a single Zustand store."

### Our Zustand Store Structure:

```typescript
interface ChatState {
  // Chat state
  messages: ChatMessage[];           // All conversation messages
  
  // Visualization state
  activeTab: 'map' | 'plot' | 'table' | 'none';  // Which tab is showing
  focusedFloatId?: string | null;    // Currently selected float
  targetCenter?: { lat: number; lon: number } | null;  // Map center
  
  // Real profile data
  realProfiles: {
    floatId?: string;
    loading: boolean;
    error?: string | null;
    profiles?: RealProfile[];
    selectedCycle?: number;          // Which dive cycle to display
    selectedCycles: number[];        // Overlay comparison cycles
  };
  
  // Actions
  addMessage: (m) => void;
  setActiveTab: (t) => void;
  setFocus: (id, lat, lon) => void;
  executeVisualizationCommand: (cmd) => void;
  reset: () => void;
}
```

### Why Zustand Over Redux?

| Feature | Redux | Zustand (Our Choice) |
|---------|-------|---------------------|
| Boilerplate | Actions, reducers, store config | Just one `create()` call |
| Bundle size | ~7KB | ~1KB |
| TypeScript | Requires extra setup | Built-in support |
| Middleware | Separate packages | Built-in (persist, devtools) |
| Learning curve | Steep | Minimal |

### Key Pattern — Visualization Commands:

> "One of the coolest features is that Gemini can control the UI. When it returns data, it also sends `visualizationCommands` like 'center the map at coordinates X,Y' or 'switch to the plot tab'. Our Zustand store processes these commands:"

```typescript
executeVisualizationCommand: (cmd) => set((state) => {
  if (cmd.action === 'switch_tab') {
    return { activeTab: cmd.tab };
  } else if (cmd.action === 'center_map') {
    return { activeTab: 'map', targetCenter: { lat: cmd.lat, lon: cmd.lon } };
  } else if (cmd.action === 'load_profile') {
    return { activeTab: 'plot', focusedFloatId: cmd.float_id };
  }
})
```

---

## 12. Data Visualization (Leaflet Maps + Plotly Charts)

### Leaflet Maps (`react-leaflet`)

> **What is Leaflet?**
> "Leaflet is a lightweight, open-source JavaScript library for interactive maps. I use `react-leaflet` for the React wrapper. Our map shows ARGO float positions as markers, dynamically centers on queried locations, and supports popups with float details."

#### Key Implementation Details:
- **Dynamic Import**: `next/dynamic` with `ssr: false` because Leaflet needs the browser `window` object
- **AbortController**: API fetch for float positions is cancelled when component unmounts
- **Focused Marker**: Different icon style for the currently selected float
- **Auto-centering**: Map automatically pans to coordinates from LLM responses

```typescript
// Dynamic import to avoid SSR issues
const MapContainer = dynamic(
  () => import('react-leaflet').then(m => m.MapContainer), 
  { ssr: false }
);

// Auto-center on LLM-driven coordinates
useEffect(() => {
  if (targetCenter && map) {
    map.setView([targetCenter.lat, targetCenter.lon], 6, { animate: true });
  }
}, [targetCenter, map]);
```

### Plotly Depth Plots (`react-plotly.js`)

> **What kind of charts?**
> "I create oceanographic depth plots where the Y-axis is inverted (depth increases downward, like a real ocean cross-section). Temperature and salinity are plotted side by side on dual X-axes, showing how they change with depth."

#### Key Features:
- **Inverted Y-axis**: `yaxis: { autorange: 'reversed' }` — depth 0m at top, 2000m at bottom
- **Dual X-axes**: Temperature on left (`xaxis`), Salinity on right (`xaxis2`)
- **Cycle Overlay**: Compare multiple dive cycles on the same plot with dotted lines
- **Transparent Background**: Matches the dark ocean theme

```typescript
layout={{
  yaxis: { title: 'Depth (m)', autorange: 'reversed' },  // Ocean standard!
  xaxis: { title: 'Temperature (°C)', domain: [0, 0.48] },
  xaxis2: { title: 'Salinity (PSU)', domain: [0.52, 1], anchor: 'y' },
  paper_bgcolor: 'rgba(0,0,0,0)',   // Transparent to show theme
  plot_bgcolor: 'rgba(0,0,0,0)',
}}
```

---

## 13. API Design & Route Architecture

### Next.js API Routes:

| Route | Method | Purpose |
|-------|--------|---------|
| `/api/query` | POST | **Main endpoint** — receives user question, orchestrates LLM + MCP + tool execution |
| `/api/floats` | GET | Returns list of all ARGO floats for map display |
| `/api/profiles` | GET | Returns profile data for a specific float |
| `/api/health` | GET | System health check |
| `/api/stats` | GET | Dashboard statistics |
| `/api/real` | GET | Fetch real-time data from Argovis API |
| `/api/upload` | POST | Handle NetCDF/CSV file uploads |
| `/api/legal` | POST | Maritime legal agent queries |
| `/api/planner` | POST | Coastal planner agent queries |
| `/api/dashboard` | GET | Admin dashboard data |

### Main Query Route (`/api/query/route.ts`) — Step by Step:

```typescript
export async function POST(request: Request) {
  // 1. Parse request body
  const { text, history } = await request.json();
  
  // 2. Rate limiting check
  const rate = consume(ip);
  if (!rate.allowed) return NextResponse.json({ error: 'Rate limit exceeded' }, { status: 429 });
  
  // 3. Connect to Python MCP Server over SSE
  const transport = new SSEClientTransport(new URL(MCP_URL));
  const mcpClient = new Client({ name: "floatchat-nextjs", version: "1.0.0" });
  await mcpClient.connect(transport);
  
  // 4. List available MCP tools
  const mcpTools = (await mcpClient.listTools()).tools;
  
  // 5. Convert MCP tools to Gemini function declarations
  const geminiDeclarations = mcpToolsToGeminiDeclarations(mcpTools);
  
  // 6. Call Gemini with tools (with API key rotation)
  for (let attempt = 0; attempt < apiKeys.length; attempt++) {
    const model = genAI.getGenerativeModel({ model: MODEL, tools: [...] });
    const chat = model.startChat({ history });
    let result = await chat.sendMessage(fullPrompt);
    
    // 7. Tool execution loop (max 3 iterations)
    while (iterations < 3) {
      const calls = response.functionCalls();
      if (calls.length === 0) break;
      // Execute tools via MCP, feed results back to Gemini
    }
    
    // 8. Extract visualization commands from tool results
    // (coordinates for map centering, etc.)
  }
  
  // 9. Return response with message, tools used, viz commands
  return NextResponse.json({ intent, message, toolsUsed, visualizationCommands });
}
```

---

## 14. LLMOps — API Key Rotation & Rate Limiting

### Sequential API Key Rotation

> **What is this and why?**
> "Gemini's free tier has rate limits (e.g., 15 requests per minute per key). During a live demo, hitting this limit would crash the app. So I implemented a **sequential rotation system** with 5 API keys. When one key gets a 429 (Too Many Requests) error, the system automatically switches to the next key without any user interruption."

```typescript
// Global index survives across requests (in-process)
declare global { var __geminiKeyIndex: number; }

const apiKeys = [
  process.env.GEMINI_API_KEY,
  process.env.GEMINI_API_KEY_2,
  process.env.GEMINI_API_KEY_3,
  process.env.GEMINI_API_KEY_4,
  process.env.GEMINI_API_KEY_5
].filter(Boolean);

// Try each key, rotating on 429 errors
for (let attempt = 0; attempt < apiKeys.length; attempt++) {
  const keyIndex = (globalThis.__geminiKeyIndex + attempt) % apiKeys.length;
  try {
    const genAI = new GoogleGenerativeAI(apiKeys[keyIndex]);
    // ... make the call ...
    break; // Success! Stay on this key
  } catch (err) {
    if (err.message.includes('429') && attempt < apiKeys.length - 1) {
      globalThis.__geminiKeyIndex = (keyIndex + 1) % apiKeys.length;
      continue; // Try next key
    }
    throw err; // All keys exhausted or non-429 error
  }
}
```

### Token Bucket Rate Limiter

> "On the frontend side, I implemented a token bucket rate limiter to prevent abuse. Each IP address gets 30 tokens (requests) per minute. This protects the LLM API from excessive usage."

```typescript
const capacity = 30;          // 30 requests
const intervalMs = 60_000;    // per minute

export function consume(ip: string): { allowed: boolean; remaining: number; resetMs: number } {
  let bucket = buckets.get(ip);
  if (!bucket) bucket = { tokens: capacity, lastRefill: Date.now() };
  
  refill(bucket);  // Add tokens based on elapsed time
  
  if (bucket.tokens <= 0) {
    return { allowed: false, remaining: 0, resetMs: ... };
  }
  
  bucket.tokens -= 1;
  return { allowed: true, remaining: bucket.tokens, resetMs: ... };
}
```

---

## 15. Multi-Agent Architecture (Specialist Agents)

### Three Specialist Agents:

#### 1. Main Ocean Explorer (`/app`)
- Full conversational interface with MCP + Gemini
- Map, Plot, and Table visualization tabs
- Chat history with markdown rendering

#### 2. Coastal Planner Agent (`/planner`)

> "The Coastal Planner is a multi-step wizard + AI chat system for maritime project planning. It has three components:"

| Component | Purpose |
|-----------|---------|
| **ProjectWizard** | Multi-step form: captures coordinates, project type (port, fish farm, etc.), and scale |
| **PlannerChat** | AI chat where Gemini roleplays as a "Nature Advocate" to critique environmental impacts |
| **ImpactAssessment** | Auto-generates structured impact reports evaluating marine effects |

#### 3. Maritime Legal Agent (`/legal`)

> "The Legal Agent is a regulatory compliance assistant for maritime activities:"

| Component | Purpose |
|-----------|---------|
| **JurisdictionSelector** | Switch between legal contexts (EEZ, territorial waters, continental shelf) |
| **LegalChat** | AI answers questions about CRZ regulations, international treaties, and zoning laws |

### Why Multi-Agent?
> "Different domains need different expertise. A single general-purpose agent would give shallow answers. By creating specialized agents with domain-specific system prompts, each agent gives deeper, more accurate answers. The Legal Agent knows about UNCLOS and CRZ regulations, while the Planner Agent knows about environmental impact assessment frameworks."

---

## 16. Intent Routing & NLP Classification

### Client-Side Intent Router

> "Before sending a query to the LLM, I classify the user's intent using regex patterns to determine which visualization tab should activate:"

```typescript
const MAP_REGEX = /(nearest|closest|map|location|where)/i;
const PLOT_REGEX = /(salinity|temperature|profile|depth|section)/i;
const TABLE_REGEX = /(summary|table|list|stats|statistics)/i;

export function classifyIntent(text: string): Intent {
  if (MAP_REGEX.test(text)) return 'map';
  if (PLOT_REGEX.test(text)) return 'plot';
  if (TABLE_REGEX.test(text)) return 'table';
  return 'unknown';
}
```

### Coordinate Extraction

> "I also extract geographic coordinates from natural language using regex:"

```typescript
export function extractLatLon(text: string): { lat?: number; lon?: number } {
  // Matches patterns like "12.5N 73.2E" or "12°N, 73°E"
  const regex = /(\d{1,2}(?:\.\d+)?)[°\s]?([NS])[,\s]+(\d{1,3}(?:\.\d+)?)[°\s]?([EW])/i;
  const match = text.match(regex);
  // Convert S/W to negative values
  if (match[2].toUpperCase() === 'S') lat = -lat;
  if (match[4].toUpperCase() === 'W') lon = -lon;
}
```

### Haversine Distance Calculation

> "To find the nearest float to a given point, I use the Haversine formula which calculates the great-circle distance between two points on a sphere (Earth):"

```typescript
export function haversineKm(lat1: number, lon1: number, lat2: number, lon2: number): number {
  const R = 6371; // Earth's radius in km
  const dLat = toRad(lat2 - lat1);
  const dLon = toRad(lon2 - lon1);
  const a = Math.sin(dLat/2) ** 2 + 
            Math.cos(toRad(lat1)) * Math.cos(toRad(lat2)) * Math.sin(dLon/2) ** 2;
  const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));
  return R * c;
}
```

---

## 17. Error Handling & Resilience

### Multi-Layer Error Handling:

```
Layer 1: Global Error Boundary (global-error.tsx)
   └── Catches any unhandled error in the entire app
   └── Shows a recovery UI with "Try Again" button

Layer 2: Route-Level Error Boundary (error.tsx)
   └── Catches errors within specific routes
   └── Shows contextual error messages

Layer 3: Component-Level Error Handling
   └── AbortController for cancelled requests
   └── try/catch in async operations
   └── Loading/error/empty states in every component

Layer 4: API-Level Error Handling
   └── Rate limiting (429 responses)
   └── API key rotation on failures
   └── Timeout management (5s connect, 25s read)
   └── Graceful MCP disconnection
```

### AbortController Pattern:

> "I use `AbortController` to prevent stale state updates. When a user switches tabs while data is loading, the pending request is cancelled so it doesn't update the wrong component:"

```typescript
useEffect(() => {
  const controller = new AbortController();
  
  fetch('/api/floats', { signal: controller.signal })
    .then(r => r.json())
    .then(data => setFloats(data.floats))
    .catch(error => {
      if (error?.name === 'AbortError') return; // Intentionally cancelled
      setLoadError('Map data could not be loaded.');
    });

  return () => controller.abort(); // Cleanup on unmount
}, []);
```

---

## 18. Testing Strategy (Jest + React Testing Library)

### Test Suite Overview:

> "I have 42 tests across 12 test suites, covering unit tests, component tests, and API route tests."

| Test File | What It Tests |
|-----------|---------------|
| `adminStatCard.test.tsx` | Admin panel stat card rendering |
| `adminPage.test.tsx` | Dashboard loading and data hydration |
| `docsIndexPage.test.tsx` | Markdown document index page |
| `docsPresence.test.ts` | Documentation file existence |
| `uploadValidator.test.ts` | File upload validation rules (NetCDF, CSV) |
| `intentRouter.test.ts` | Intent classification correctness |
| `haversine.test.ts` | Haversine distance calculation accuracy |
| `rateLimit.test.ts` | Token bucket rate limiter behavior |
| `queryRoute.llm.test.ts` | LLM query API endpoint |
| `queryRoute.fallback.test.ts` | Fallback when LLM unavailable |
| `realProfilesFetch.test.ts` | Real-time profile data fetching |

### Testing Setup:

```typescript
// jest.setup.ts
import '@testing-library/jest-dom';

// Polyfill fetch for Node.js test environment
import fetch, { Request, Response } from 'cross-fetch';
global.fetch = fetch;
global.Request = Request;
global.Response = Response;

// Polyfill Response.json() for Node 18
if (!Response.json) {
  Response.json = (data, init) => 
    new Response(JSON.stringify(data), { ...init, headers: { 'content-type': 'application/json' } });
}
```

### SQL Injection Prevention Testing:

```typescript
// uploadValidator.test.ts
test('blocks mutation SQL keywords', () => {
  const dangerous = ['INSERT INTO', 'DROP TABLE', 'DELETE FROM', 'ALTER TABLE'];
  dangerous.forEach(keyword => {
    expect(executeQueryArgoSql(keyword)).toContain('Error: Only SELECT queries');
  });
});
```

---

## 19. UI/UX Design System & Styling

### Design Philosophy

> "I created a **glassmorphic, ocean-themed dark mode UI** that feels premium and immersive. The design system is built on custom Tailwind tokens — not generic colors."

### Custom Design Tokens:

```javascript
colors: {
  floatchat: {
    primary: '#051014',       // Deep midnight blue base
    secondary: '#07242C',     // Slightly lighter for panels
    accent: '#00F0FF',        // Neon cyan for primary actions
    accentCoral: '#FF4545',   // Energetic coral for warnings
    bg: '#030B0E',            // Very dark background
    panel: 'rgba(7, 36, 44, 0.45)',  // Semi-transparent panels
    ink: '#E0F2FE',           // Light cyan text
    border: 'rgba(0, 240, 255, 0.15)',
    success: '#00FF9D',
    warning: '#FFB800',
    danger: '#FF4545',
  }
}
```

### Custom Animations:

```javascript
keyframes: {
  'float-slow': { '50%': { transform: 'translateY(-8px)' } },      // Floating effect
  'pulse-glow': { '50%': { boxShadow: '0 0 25px rgba(0, 240, 255, 0.5)' } },  // Glow pulse
  'shimmer': { '100%': { backgroundPosition: '1000px 0' } },       // Loading shimmer
}
```

### Accessibility Features:
- **Skip-to-main-content** link
- **Focus-visible** rings on all interactive elements
- **Reduced-motion** detection (disables animations for `prefers-reduced-motion`)
- **ARIA labels** on dropzones and interactive regions
- **Keyboard navigation** support (Enter/Space to trigger actions)
- **High contrast** text over dark backgrounds

### Typography:
- **Font**: Inter (loaded from Google Fonts)
- **Markdown rendering**: Using `marked` library with Tailwind Typography plugin (`prose` classes)

---

## 20. DevOps & Deployment (Docker + Vercel)

### Docker Compose:
```yaml
services:
  db: postgis/postgis:16-3.4    # PostgreSQL + PostGIS
  chroma: chromadb/chroma        # Vector database
```

### Vercel Deployment:
```json
{
  "framework": "nextjs",
  "functions": {
    "api/**/*.js": { "maxDuration": 10 }  // 10-second API timeout
  }
}
```

### Environment Variables:
| Variable | Purpose |
|----------|---------|
| `GEMINI_API_KEY` through `_5` | 5 API keys for rotation |
| `DATABASE_URL` | PostgreSQL connection string |
| `MCP_SERVER_URL` | Python MCP Server endpoint |
| `ARGOVIS_API_KEY` | External ocean data API key |
| `ARGO_DATA_DIR` | Path to NetCDF data files |

---

## 21. Security Considerations

| Concern | Implementation |
|---------|---------------|
| **SQL Injection** | Forbidden keywords list blocks INSERT/DROP/ALTER/DELETE |
| **Row Limit** | Auto-appends `LIMIT 50` to prevent data exfiltration |
| **Rate Limiting** | Token bucket (30 req/min per IP) prevents API abuse |
| **API Key Security** | Keys in `.env.local` (gitignored), never exposed to client |
| **Input Validation** | Zod schema validation on API inputs |
| **CORS** | Next.js default CORS policy (same-origin) |
| **Error Masking** | Detailed errors logged server-side, generic messages to client |

---

## 22. Most Likely Interview Questions & Answers

### Q1: "Tell me about your project."
> **Answer:** "FloatChat is a Multi-Agent Conversational Ocean Data Explorer built for the Smart India Hackathon. It lets users ask natural language questions about the ocean and gets real-time answers using ARGO float data. I built the full stack — a Next.js 14 frontend with interactive maps and charts, a Python MCP Server backend with 5 tool functions, and integrated Google Gemini for intelligent tool-calling. The system processes real ocean data from APIs like Argovis and OceanOPS, stores it in PostgreSQL with PostGIS for spatial queries, and uses ChromaDB for semantic vector search."

### Q2: "What was the most challenging part?"
> **Answer:** "The most challenging part was building the MCP tool-calling loop. I had to convert MCP tool schemas to Gemini function declarations (different formats), handle multi-turn tool execution where the LLM might call 3 different tools in sequence, feed results back to the model, and extract visualization commands from the tool results — all while handling timeouts, rate limits, and connection failures gracefully. I also had to solve a critical infinite render loop bug in the BubbleBackground component where a `useEffect` dependency on an array reference was causing React to re-render infinitely."

### Q3: "Why did you choose these specific technologies?"
> **Answer:**
> - **Next.js 14**: App Router for modern server components, API routes for serverless backend, built-in optimizations
> - **Python for MCP**: The MCP SDK is best supported in Python, and oceanographic libraries (netCDF4, numpy) are Python-native
> - **Gemini 2.5 Flash**: Native function-calling support, fast response times, free tier for prototyping
> - **Zustand over Redux**: 1KB vs 7KB, zero boilerplate, perfect for our single-store pattern
> - **PostgreSQL + PostGIS**: Industry standard for spatial data, GIST indexing for KNN queries
> - **ChromaDB**: Lightweight, embeddable vector store perfect for our PoC scale

### Q4: "How does the LLM know which tool to call?"
> **Answer:** "When we connect to the MCP Server, we get back a list of all available tools with their names, descriptions, and parameter schemas. We convert these into Gemini's function declaration format and pass them when creating the model. Gemini's training includes function-calling patterns — it reads the tool descriptions, understands the user's intent, and decides which tool(s) to call. For example, if the user says 'nearest float at 12N 73E', Gemini recognizes this as a spatial query and calls `search_ocean_area` with the right coordinates."

### Q5: "What is MCP and why is it important?"
> **Answer:** "MCP stands for Model Context Protocol — it's an open standard created by Anthropic that acts as a universal connector between AI models and external data sources. Think of it like USB for AI. In our project, the Python server exposes ocean data tools via MCP, and the Next.js frontend consumes them via an MCP client. The key benefit is **decoupling** — the LLM doesn't need to know about HTTP endpoints or API formats, it just calls named functions. If we change our data source from Argovis to another API, we only update the MCP server — no changes to the LLM or frontend."

### Q6: "How do you handle API rate limits?"
> **Answer:** "I implemented two layers of protection:
> 1. **Frontend rate limiter**: Token bucket algorithm — each IP gets 30 requests per minute. When exhausted, the API returns 429.
> 2. **LLM key rotation**: I have 5 Gemini API keys. When one hits the rate limit (429 error), the system automatically tries the next key. A global index tracks which key to use, rotating round-robin. This ensures the demo never crashes during presentations."

### Q7: "Explain your database schema design."
> **Answer:** "I designed a normalized relational schema with three main tables forming a hierarchy: `floats` (the device) → `profiles` (each measurement cycle) → `measurements` (depth-indexed readings). I added PostGIS geometry columns with GIST spatial indexes for fast nearest-neighbor queries. There's also a `profile_stats` table with pre-computed aggregates (mean temperature, surface temperature, mixed layer depth) to avoid expensive calculations at query time. For semantic search, I use ChromaDB alongside PostgreSQL — they complement each other."

### Q8: "How does the frontend react to LLM responses?"
> **Answer:** "The LLM response includes `visualizationCommands` — instructions for the UI. For example, `{ action: 'center_map', lat: 4.5, lon: 73.2 }` tells the map to pan to those coordinates. The Zustand store processes these commands and React re-renders the affected components. I also classify the intent from the query text (map/plot/table) to switch the active tab. This creates a seamless experience where asking 'show me floats near India' automatically switches to the map tab and centers on India."

### Q9: "What testing approach did you use?"
> **Answer:** "I used Jest with React Testing Library for 42 tests across 12 suites. I tested:
> - **Unit tests**: Haversine distance calculation, intent classification, rate limiter behavior
> - **Component tests**: Admin dashboard rendering, stat card display, document page loading
> - **API route tests**: Query endpoint with and without LLM, fallback behavior
> - **Validation tests**: File upload validator for NetCDF/CSV formats, SQL injection prevention
> I also had to polyfill `fetch`, `Request`, and `Response` for the Node.js test environment since Next.js API routes depend on Web APIs."

### Q10: "What would you improve if you had more time?"
> **Answer:**
> - Add Redis for distributed rate limiting (current in-memory limiter resets on server restart)
> - Implement streaming responses with Server-Sent Events for real-time LLM output
> - Add user authentication and session management
> - Build a CI/CD pipeline with automated testing and deployment
> - Add Sentry for production error monitoring
> - Implement WebSocket for real-time float position updates
> - Add data export (CSV/NetCDF) for analysis results

---

## 23. Technical Deep-Dive Questions

### Q: "Explain SSE (Server-Sent Events) and how MCP uses it."
> "SSE is a one-way communication protocol where the server pushes events to the client over a persistent HTTP connection. Unlike WebSockets (bidirectional), SSE is simpler and works over regular HTTP. In our project, the MCP Client in Next.js connects to the Python MCP Server at `http://127.0.0.1:8000/sse`. The server sends tool results as SSE events — each event has a type and data payload. This is more efficient than polling because the connection stays open."

### Q: "What is the Haversine formula and why do you need it?"
> "The Haversine formula calculates the shortest distance between two points on a sphere (great-circle distance). It accounts for the Earth's curvature, unlike simple Euclidean distance which would be inaccurate for large distances. I use it in two places: (1) the TypeScript `geo.ts` utility for client-side nearest-float calculations, and (2) as a fallback in `toolExecutor.ts` when PostGIS's `ST_Distance` is unavailable."

### Q: "How does dynamic importing help with SSR?"
> "Leaflet and Plotly require the `window` object (browser DOM) to function. In Next.js, pages are first rendered on the server (SSR) where `window` doesn't exist — this causes crashes. By using `next/dynamic` with `{ ssr: false }`, I tell Next.js to skip server rendering for these components and load them only on the client side. The `loading` option provides a placeholder while the component loads."

### Q: "Explain the tool schema conversion between MCP and Gemini."
> "MCP tools use standard JSON Schema format (with `type: 'string'`, `type: 'number'`), while Gemini uses its own format with an enum-like type system (`SchemaType.STRING`, `SchemaType.NUMBER`). My `mcpToolsToGeminiDeclarations` function iterates through each MCP tool's `inputSchema.properties`, maps the types, and constructs Gemini-compatible `functionDeclarations`. This conversion happens at runtime, so any new MCP tools are automatically available to Gemini."

### Q: "What is cosine similarity and how does ChromaDB use it?"
> "Cosine similarity measures how similar two vectors are by calculating the cosine of the angle between them. A value of 1 means identical direction (very similar), 0 means perpendicular (unrelated). ChromaDB converts text into numerical vectors (embeddings) and uses cosine similarity to find documents that are semantically similar to a search query. When a user asks about 'warm waters', ChromaDB finds profiles whose descriptions are closest in meaning."

### Q: "Explain `dangerouslySetInnerHTML` — why is it 'dangerous' and why did you use it?"
> "It's called 'dangerous' because it injects raw HTML into the DOM, which can enable XSS (Cross-Site Scripting) attacks if the content comes from untrusted sources. I use it to render markdown from the LLM responses — the `marked` library converts markdown to HTML, which I then inject. This is relatively safe because the content comes from our own Gemini API (not user-generated), and we control the system prompt. In production, I would add a sanitization library like DOMPurify."

---

## 24. Behavioral / Scenario Questions

### Q: "How did you handle a situation where something wasn't working?"
> "During development, I encountered a critical bug where ALL buttons on every page stopped working. After investigation, I found that the `BubbleBackground` component had an infinite render loop caused by a `useEffect` dependency on an array literal `sizeRange` which created a new reference every render. This cascaded to crash React's reconciler. I fixed it by tracking the primitive values `sizeRange[0]` and `sizeRange[1]` instead of the array reference. I also added the fix to the progress report and wrote a test to prevent regression."

### Q: "How do you plan and organize your development?"
> "I followed a structured approach with documented sessions:
> 1. **Session 1**: Built the design system, layouts, and core UI
> 2. **Session 2**: Fixed critical bugs, added testing infrastructure (42 tests)
> 3. **Session 3**: Presentation stabilization — resilience, abort controllers, error states
> 4. **Session 4**: SIH alignment — identified gaps, implemented MCP integration
> Each session had clear objectives, and I maintained a `PROGRESS_REPORT.md` documenting every fix, bug, and improvement."

### Q: "How did you ensure code quality?"
> "Multiple layers:
> - **TypeScript**: Catches type errors at compile time
> - **ESLint**: Enforces coding standards
> - **Jest tests**: 42 automated tests cover critical paths
> - **Error boundaries**: Prevent crashes from cascading
> - **Code reviews**: I reviewed my own code against the SIH requirements checklist
> - **Documentation**: README, DESIGN_SYSTEM.md, PROGRESS_REPORT.md, API contracts"

### Q: "What did you learn from this project?"
> "Several things:
> 1. **MCP Protocol**: Learned how AI models communicate with external tools using the standardized Model Context Protocol
> 2. **Agentic AI patterns**: How to build a reasoning loop where the LLM plans and executes multi-step tool chains
> 3. **Spatial databases**: PostGIS spatial indexing, GIST indexes, and geographic distance calculations
> 4. **Production resilience**: The importance of error boundaries, abort controllers, and API key rotation for real-world deployments
> 5. **Oceanography domain**: Learned about ARGO floats, temperature-salinity profiles, and ocean data standards"

---

## 📌 Quick Reference Card — Technologies at a Glance

| Category | Technology | Version |
|----------|-----------|---------|
| **Frontend Framework** | Next.js (App Router) | 14.2 |
| **UI Library** | React | 18.3.1 |
| **Language** | TypeScript | 5.4.5 |
| **Styling** | TailwindCSS | 3.4.4 |
| **State Management** | Zustand | 4.5.2 |
| **Maps** | Leaflet + react-leaflet | 1.9.4 / 4.2.1 |
| **Charts** | Plotly.js + react-plotly.js | 2.35.2 / 2.6.0 |
| **Markdown** | marked | 13.0.2 |
| **Schema Validation** | Zod | 3.23.8 |
| **LLM** | Google Gemini 2.5 Flash | — |
| **LLM SDK** | @google/generative-ai | 0.24.1 |
| **MCP SDK (JS)** | @modelcontextprotocol/sdk | 1.29.0 |
| **MCP SDK (Python)** | mcp (FastMCP) | ≥1.2.1 |
| **Vector DB** | ChromaDB | ≥0.4.24 |
| **Database** | PostgreSQL + PostGIS | 16 / 3.4 |
| **DB Client** | pg (node-postgres) | 8.12.0 |
| **Python** | Python | 3.10+ |
| **HTTP Client (Python)** | requests | ≥2.31.0 |
| **Data Processing** | pandas | ≥2.2.0 |
| **LLM Chain** | LangChain | ≥0.1.13 |
| **Testing** | Jest + React Testing Library | 29.7 / 14.3.1 |
| **Linting** | ESLint | 8.57 |
| **Containerization** | Docker Compose | 3.9 |
| **Deployment** | Vercel | — |

---

## 🎤 Final Tips for the Interview

1. **Start with the big picture** — describe the project as a whole before diving into specifics
2. **Use analogies** — "MCP is like USB for AI", "Zustand is like a simplified Redux"
3. **Show awareness of trade-offs** — "I chose Zustand over Redux because..."
4. **Mention real challenges** — the infinite render loop bug shows debugging skills
5. **Know your numbers** — 42 tests, 12 suites, 5 API keys, 5 MCP tools, 3 agent types
6. **Be ready to code** — know how Haversine formula works, how token bucket works
7. **Emphasize production readiness** — error boundaries, rate limiting, abort controllers
8. **Connect to business value** — climate monitoring, maritime safety, coastal planning

> **Good luck with your interview! 🚀**
