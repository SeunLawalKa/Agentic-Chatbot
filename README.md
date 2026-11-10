# Agentic Chatbot Autonomous System

This project is a **multi-agent** AI chatbot designed to answer user queries on various topics. The system is built around a supervisor agent that analyzes incoming queries and routes them to the most suitable specialized agent for the task.

## Main Functions

* **Routing:** Classifies the user query topic (e.g., Official Gazete, News/General Knowledge, Travel Planning, Uploaded Document Query, Other) and triggers the relevant agent.
* **News and General Knowledge:** Includes a ReAct agent using web search (Tavily/DuckDuckGo) and Wikipedia tools for current events, definitions, and general knowledge questions.
* **Travel Planning:** Runs a complex sub-workflow (containing its own agents and tools) to generate detailed travel plans based on user-provided destination, dates, budget, etc., and can present the result as a PDF.
* **Agentic RAG (Document Querying):** Allows users to upload their own documents (PDF, TXT, DOCX, etc.) and ask questions about their content. This agent processes the uploaded document and generates answers using the RAG mechanism.
* **Fallback:** Provides a standard response for queries outside the defined scopes.

## Deficiencies
* Agent Memory
  
## Technologies Used

* **Programming Language:** Python
* **Orchestration & Agent Framework:** LangChain & LangGraph
* **User Interface:** Streamlit
* **Language Model (LLM):** Google Gemini (e.g., Gemini 1.5 Flash)
* **Embedding Models:** intfloat/multilingual-e5-large (classic RAG) & models/embedding-001 (Agentic RAG) 
* **Vector Database:** ChromaDB
* **External APIs:** Tavily/DuckDuckGo, Wikipedia, Google Serper, OpenWeatherMap, TomTom Maps, ExchangeRate-API
* **Containerization:** Docker & Docker Compose

## Architecture

## Project Structure
```
agentic-chatbot-project/
├── app/                      # Main application code
│   ├── agents/               # Core agent logic used as nodes in LangGraph
│   │   ├── __init__.py
│   │   ├── agentic_rag_agent.py   # Agent performing RAG with uploaded documents (LangGraph node)
│   │   ├── fallback_agent.py       # Agent for cases not fitting other categories (LangGraph node)
│   │   ├── news_agent.py           # Agent for general information/news queries (LangGraph node)
│   │   ├── official_gazete_agent.py   # official Gazete RAG agent (LangGraph node) - renamed from official_news_agent.py
│   │   ├── supervisor.py           # Agent that classifies and routes the query (LangGraph node)
│   │   └── travel_agent.py         # Agent processing travel queries (LangGraph node), calls TravelPlanningSystem
│   ├── core/                   # Core components (LLM, State)
│   │   ├── __init__.py
│   │   ├── llm.py               # Manages LLM instances (get_llm)
│   │   └── state.py             # Main AgentState definition (for LangGraph)
│   ├── storage/                # Data storage related (ChromaDB)
│   │   ├── __init__.py
│   │   └── database.py          # ChromaDB client, collection management, querying
│   ├── tools/                  # General tools used by main agents
│   │   ├── __init__.py
│   │   ├── external_apis.py     # Tools for Wikipedia, WebSearch (Tavily/DDG) (used by news_agent)
│   │   └── rag_tools.py         # retrieve_documents, format_context (used by official_gazete_agent)
│   ├── travel_system/          # Travel planning subsystem (its own workflow)
│   │   ├── __init__.py
│   │   ├── agents/             # Specialized agents within the travel system
│   │   │   ├── __init__.py
│   │   │   ├── coordinator_agent.py  # Agent compiling the travel plan
│   │   │   ├── date_budget_agent.py   # Agent processing dates and budget
│   │   │   └── destination_agent.py   # Agent researching destination info (city, weather, hotel links)
│   │   ├── tools/              # Tools specific to the travel system
│   │   │   ├── __init__.py
│   │   │   ├── budget_tools.py      # Currency rates, budget evaluation
│   │   │   ├── date_tools.py        # Date calculations
│   │   │   ├── destination_tools.py # City info, weather, hotel links, map URLs
│   │   │   └── parsing_tools.py     # Structuring the user query
│   │   ├── utils/              # Utility modules for the travel system
│   │   │   ├── __init__.py
│   │   │   └── pdf_saver.py        # Saving travel plan as PDF
│   │   └── workflow.py         # TravelPlanningSystem class (LangGraph workflow for travel)
│   ├── ui/                     # Streamlit user interface
│   │   ├── __init__.py
│   │   └── streamlit_app.py    # Main Streamlit app, calls graph.py
│   ├── utils/                  # General helper functions (embedding, text processing)
│   │   ├── __init__.py
│   │   ├── embedding.py         # Loading embedding model and creating embeddings (used by scripts and database)
│   │   └── text_processing.py   # Text splitting (used by process_data script)
│   ├── __init__.py
│   └── graph.py                # Main LangGraph workflow (AgentState, Supervisor, Agent Nodes)
├── assets/                   # Static files (e.g. fonts)
│   └── fonts/                # Font files for PDF generation
│       ├── DejaVuSansCondensed.ttf
│       └── DejaVuSansCondensed-Bold.ttf
├── configs/                  # Configuration files
│   ├── __init__.py
│   ├── agent_config.py       # Prompts for agents, categories, RAG settings, etc.
│   ├── api_config.py         # Logic for loading API keys (e.g. load_dotenv)
│   ├── app_config.py         # General settings for app/ (Chroma path, model names, etc.)
│   └── script_config.py      # Settings for scripts/ (URLs, paths, etc.)
├── data/                     # Application data (can be volume-mounted)
│   ├── embeddings/           # ChromaDB database files
│   │   └── chroma.sqlite3    # (and other ChromaDB files)
│   ├── processed/            # Processed and chunked data (.jsonl)
│   │   ├── haberler/          # News
│   │   │   └── haberler_processed.jsonl
│   │   └── official_gazete/      # official Gazete
│   │       └── official_gazete_processed.jsonl
│   └── raw/                  # Raw fetched data
│       ├── haberler/          # News
│       │   └── trt_haberler.json
│       └── official_gazete/      # official Gazete
│           └── aa_official_gazete_tum_haberler.json
├── plans/                    # Directory for saved travel plan PDFs
├── scripts/                  # Data processing and preparation scripts
│   ├── __init__.py
│   ├── del.py                # Optional script to clean the data/ directory
│   ├── generate_embeddings.py# Generates embeddings from processed data and writes to DB
│   ├── news_fetcher.py       # Fetches data from TRT Haber RSS
│   ├── process_data.py       # Processes raw data, splits into chunks
│   └── official_news_fetcher.py # Fetches official Gazete news from AA
├── .dockerignore             # Files to ignore in Docker build
├── .env                      # Environment variables (API keys, etc.) – should not be committed!
├── .gitignore                # Files not tracked by Git
├── chatbot-architecture.png  # Generated Mermaid architecture diagram
├── docker-compose.yml        # Docker Compose configuration
├── Dockerfile                # Dockerfile for main application
├── README.md                 # Project description
└── requirements.txt          # Python dependencies

 
```
------------
