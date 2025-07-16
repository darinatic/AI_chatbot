# Agentic AI Chatbot Workflow Diagram

## Overall System Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          STREAMLIT UI LAYER                            │
├─────────────────────────────────────────────────────────────────────────┤
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐      │
│  │   LLM Selection │    │ Usecase Selection│    │  API Key Input  │      │
│  │   - Groq        │    │ - Basic Chatbot │    │ - GROQ_API_KEY  │      │
│  │   - OpenAI      │    │ - Chatbot+Web   │    │ - TAVILY_API_KEY│      │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘      │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                          MAIN APPLICATION                              │
│                        (main.py - Entry Point)                         │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                          LLM CONFIGURATION                             │
│                           (GroqLLM Class)                              │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  • Validates API keys                                          │   │
│  │  • Initializes selected LLM model                              │   │
│  │  • Returns configured model instance                           │   │
│  └─────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                          GRAPH BUILDER                                 │
│                       (GraphBuilder Class)                             │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  Decides graph structure based on usecase:                     │   │
│  │                                                                 │   │
│  │  IF "Basic Chatbot" → basic_chatbot_build_graph()              │   │
│  │  IF "Chatbot With Web" → chatbot_with_tools_build_graph()      │   │
│  └─────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                          ┌─────────┴─────────┐
                          │                   │
                          ▼                   ▼
```

## Basic Chatbot Workflow

```
┌─────────────────────────────────────────────────────────────────────────┐
│                       BASIC CHATBOT FLOW                               │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                          GRAPH STRUCTURE                               │
│                                                                         │
│    START ──────► CHATBOT NODE ──────► END                              │
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  BasicChatbotNode.process():                                    │   │
│  │  • Receives user message in state                               │   │
│  │  • Invokes LLM with message                                     │   │
│  │  • Returns AI response                                          │   │
│  └─────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                        DISPLAY RESULT                                  │
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  • Stream graph events                                          │   │
│  │  • Display user message                                         │   │
│  │  • Display AI response                                          │   │
│  └─────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────┘
```

## Chatbot with Web Search Workflow

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    CHATBOT WITH WEB SEARCH FLOW                        │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                          GRAPH STRUCTURE                               │
│                                                                         │
│           START ──────► CHATBOT NODE ──────► TOOLS NODE                │
│                              │                    │                     │
│                              │                    │                     │
│                              └────────────────────┘                     │
│                                                   │                     │
│                                                   ▼                     │
│                                                 END                     │
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  CHATBOT NODE (ChatbotWithToolNode):                            │   │
│  │  • Receives user message                                        │   │
│  │  • LLM decides if tools are needed                              │   │
│  │  • Returns response OR tool calls                               │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  TOOLS NODE (Tavily Search):                                    │   │
│  │  • Executes search queries                                      │   │
│  │  • Returns search results                                       │   │
│  │  • Flows back to chatbot node                                   │   │
│  └─────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                        DISPLAY RESULT                                  │
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  • Process all message types:                                   │   │
│  │    - HumanMessage: Display user input                           │   │
│  │    - ToolMessage: Display search results                        │   │
│  │    - AIMessage: Display final AI response                       │   │
│  └─────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────┘
```

## Detailed Component Flow

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        COMPONENT INTERACTION                           │
└─────────────────────────────────────────────────────────────────────────┘

   User Input
       │
       ▼
┌─────────────────┐
│  LoadStreamlitUI│  ─────► UI Configuration
│                 │         • LLM Selection
│                 │         • Usecase Selection  
│                 │         • API Key Validation
└─────────────────┘
       │
       ▼
┌─────────────────┐
│   main.py       │  ─────► Application Entry Point
│                 │         • Load UI
│                 │         • Handle User Input
│                 │         • Configure LLM
│                 │         • Build Graph
│                 │         • Display Results
└─────────────────┘
       │
       ▼
┌─────────────────┐
│   GroqLLM       │  ─────► LLM Configuration
│                 │         • Validate API Keys
│                 │         • Initialize Model
│                 │         • Return LLM Instance
└─────────────────┘
       │
       ▼
┌─────────────────┐
│  GraphBuilder   │  ─────► Graph Construction
│                 │         • Select Graph Type
│                 │         • Add Nodes
│                 │         • Define Edges
│                 │         • Compile Graph
└─────────────────┘
       │
       ▼
┌─────────────────┐
│  Graph Execution│  ─────► Message Processing
│                 │         • State Management
│                 │         • Node Execution
│                 │         • Tool Integration
│                 │         • Response Generation
└─────────────────┘
       │
       ▼
┌─────────────────┐
│DisplayResultUI  │  ─────► Result Presentation
│                 │         • Format Messages
│                 │         • Stream Output
│                 │         • Handle Tool Results
└─────────────────┘
```

## State Management Flow

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           STATE FLOW                                   │
└─────────────────────────────────────────────────────────────────────────┘

Initial State: {"messages": [user_message]}
       │
       ▼
┌─────────────────┐
│  Graph Process  │
│                 │
│  State Updates: │
│  • Add AI messages
│  • Add tool calls
│  • Add tool results
│  • Maintain context
└─────────────────┘
       │
       ▼
Final State: {"messages": [user_msg, ai_msg, tool_msg, final_ai_msg]}
       │
       ▼
┌─────────────────┐
│  UI Display     │
│                 │
│  Renders each   │
│  message type   │
│  appropriately  │
└─────────────────┘
```

## Tool Integration Details

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         TOOL INTEGRATION                               │
└─────────────────────────────────────────────────────────────────────────┘

┌─────────────────┐
│  get_tools()    │  ─────► Returns [TavilySearchResults(max_results=2)]
└─────────────────┘
       │
       ▼
┌─────────────────┐
│create_tool_node │  ─────► Creates ToolNode for graph integration
└─────────────────┘
       │
       ▼
┌─────────────────┐
│  tools_condition│  ─────► LangGraph built-in conditional logic
│                 │         • Checks if LLM wants to use tools
│                 │         • Routes to tool node or END
└─────────────────┘
```

This workflow demonstrates a sophisticated AI chatbot system with modular architecture, supporting both basic conversation and web-enhanced responses through a graph-based approach using LangGraph and Streamlit.