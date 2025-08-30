# Agno Memory and Tools Tutorial

## Overview

This tutorial demonstrates building **intelligent agents with memory and tools** using the Agno framework. You'll learn to create sophisticated AI agents that can remember past conversations, use external tools, and maintain persistent context across sessions.

## Learning Objectives

By completing this tutorial, you will:

- Understand agent memory systems and their importance
- Learn to integrate tools with AI agents
- Implement persistent memory with SQLite
- Create custom tools for specific tasks
- Build agents that remember user preferences and context
- Master conversation flow and context management

## What is Agent Memory?

**Agent Memory** enables AI agents to:

- **Remember** past conversations and interactions
- **Learn** from user preferences and patterns
- **Maintain context** across multiple sessions
- **Personalize** responses based on history
- **Store** and retrieve relevant information efficiently

## Getting Started

### Prerequisites

```bash
pip install -U agno
pip install python-dotenv
```

### Environment Setup

1. **Create a `.env` file**:

   ```env
   OPENAI_API_KEY=your_openai_api_key_here
   GOOGLE_API_KEY=your_google_api_key_here
   SERPER_API_KEY=your_serper_api_key_here
   ```

2. **Get Required API Keys**:
   - **OpenAI**: [platform.openai.com/api-keys](https://platform.openai.com/api-keys)
   - **Google**: [ai.google.dev/gemini-api/docs/api-key](https://ai.google.dev/gemini-api/docs/api-key)
   - **Serper (Google Search)**: [serper.dev](https://serper.dev/)

### Project Structure

```
3 Agno Memory/
├── L-2_agno_agenst_tools_memory.ipynb    # Main tutorial notebook
└── README.md                             # This tutorial
```

## How to Run

```bash
jupyter notebook L-2_agno_agenst_tools_memory.ipynb
```

## Implementation Components

### 1. Basic Agent with Memory

```python
from agno.agent import Agent
from agno.models.openai import OpenAIChat
from agno.memory import ConversationMemory

agent = Agent(
    model=OpenAIChat(id="gpt-4"),
    memory=ConversationMemory(),
    description="Helpful assistant with memory"
)
```

### 2. Tool Integration

**Google Search Tool**:

```python
from agno.tools.google import GoogleSearchTool

search_agent = Agent(
    model=OpenAIChat(id="gpt-4"),
    tools=[GoogleSearchTool()],
    description="Agent with web search capabilities"
)
```

**Python Code Execution**:

```python
from agno.tools.python import PythonTool

code_agent = Agent(
    model=OpenAIChat(id="gpt-4"),
    tools=[PythonTool()],
    description="Agent that can execute Python code"
)
```

### 3. Persistent Memory with SQLite

```python
from agno.memory.db import SQLiteMemory

persistent_agent = Agent(
    model=OpenAIChat(id="gpt-4"),
    memory=SQLiteMemory(
        db_file="agent_memory.db",
        table_name="conversations"
    ),
    description="Agent with persistent memory"
)
```

### 4. Custom Tool Creation

```python
from agno.tools.base import Tool

class WeatherTool(Tool):
    def __init__(self):
        super().__init__(
            name="get_weather",
            description="Get current weather for a location"
        )

    def run(self, location: str) -> str:
        # Your weather API implementation
        return f"Weather in {location}: Sunny, 72°F"

weather_agent = Agent(
    model=OpenAIChat(id="gpt-4"),
    tools=[WeatherTool()],
    description="Agent with weather information"
)
```

## Key Features Demonstrated

### 1. **Memory Types**

**Short-term Memory**:

- Conversation buffer
- Recent interaction context
- Session-based storage

**Long-term Memory**:

- Persistent database storage
- User preference learning
- Cross-session continuity

**Working Memory**:

- Active task context
- Tool usage history
- Current goal tracking

### 2. **Tool Categories**

**Information Retrieval**:

- Web search capabilities
- Database queries
- API integrations

**Content Creation**:

- Code generation and execution
- Document creation
- Data visualization

**Communication**:

- Email sending
- Notification systems
- Social media posting

### 3. **Memory Operations**

**Storage**:

```python
# Automatic conversation storage
agent.memory.add_message("user", "My name is John")
agent.memory.add_message("assistant", "Nice to meet you, John!")
```

**Retrieval**:

```python
# Context-aware retrieval
relevant_context = agent.memory.get_relevant_context("What's my name?")
```

**Search**:

```python
# Semantic search through memory
results = agent.memory.search("previous Python code examples")
```

## Memory Architecture

### 1. **Conversation Memory**

```python
class ConversationMemory:
    def __init__(self, max_messages=100):
        self.messages = []
        self.max_messages = max_messages

    def add_message(self, role, content):
        self.messages.append({"role": role, "content": content})
        if len(self.messages) > self.max_messages:
            self.messages.pop(0)  # Remove oldest
```

### 2. **Database Memory**

```python
class SQLiteMemory:
    def __init__(self, db_file="memory.db"):
        self.db = sqlite3.connect(db_file)
        self.create_tables()

    def store_conversation(self, user_id, conversation):
        # Store conversation with timestamps and metadata
        pass

    def retrieve_context(self, user_id, query):
        # Retrieve relevant conversation history
        pass
```

### 3. **Vector Memory**

```python
class VectorMemory:
    def __init__(self, embedding_model):
        self.embeddings = embedding_model
        self.vector_store = ChromaDB()

    def store_with_embedding(self, text, metadata):
        # Store text with vector embeddings for semantic search
        pass

    def semantic_search(self, query, k=5):
        # Find semantically similar memories
        pass
```

## Practical Examples

### 1. **Personal Assistant with Memory**

```python
personal_assistant = Agent(
    model=OpenAIChat(id="gpt-4"),
    memory=SQLiteMemory("personal_assistant.db"),
    tools=[GoogleSearchTool(), PythonTool()],
    description="Personal assistant that remembers your preferences"
)

# First conversation
personal_assistant.run("I love Italian food and work as a data scientist")

# Later conversation (agent remembers)
personal_assistant.run("Recommend a restaurant for dinner")
# Agent will suggest Italian restaurants and consider your profession
```

### 2. **Learning Agent**

```python
learning_agent = Agent(
    model=OpenAIChat(id="gpt-4"),
    memory=VectorMemory(embedding_model="text-embedding-ada-002"),
    description="Agent that learns from interactions"
)

# Agent builds knowledge over time
learning_agent.run("Python list comprehensions are efficient")
learning_agent.run("I prefer functional programming style")

# Later questions benefit from learned context
learning_agent.run("Help me write efficient Python code")
```

### 3. **Project Assistant**

```python
project_assistant = Agent(
    model=OpenAIChat(id="gpt-4"),
    memory=SQLiteMemory("project_memory.db"),
    tools=[PythonTool(), GoogleSearchTool()],
    description="Assistant for software development projects"
)

# Track project progress and decisions
project_assistant.run("We decided to use FastAPI for the backend")
project_assistant.run("The database schema uses PostgreSQL")

# Later queries have full project context
project_assistant.run("How should we implement user authentication?")
```

## Advanced Memory Strategies

### 1. **Hierarchical Memory**

```python
class HierarchicalMemory:
    def __init__(self):
        self.short_term = ConversationMemory(max_messages=20)
        self.medium_term = ConversationMemory(max_messages=200)
        self.long_term = SQLiteMemory("long_term.db")

    def consolidate_memory(self):
        # Move important information from short to long-term
        important_messages = self.identify_important_messages()
        self.long_term.store_messages(important_messages)
```

### 2. **Contextual Memory**

```python
class ContextualMemory:
    def __init__(self):
        self.contexts = {
            "work": SQLiteMemory("work_context.db"),
            "personal": SQLiteMemory("personal_context.db"),
            "projects": SQLiteMemory("projects_context.db")
        }

    def get_context_memory(self, context_type):
        return self.contexts.get(context_type, self.contexts["personal"])
```

### 3. **Adaptive Memory**

```python
class AdaptiveMemory:
    def __init__(self):
        self.memory_importance = {}
        self.access_frequency = {}

    def update_importance(self, memory_id, importance_score):
        # Adjust memory importance based on usage
        self.memory_importance[memory_id] = importance_score

    def prune_memory(self):
        # Remove less important memories to save space
        pass
```

## Best Practices

### 1. **Memory Management**

- **Regular pruning** of old, irrelevant memories
- **Importance scoring** for memory retention
- **Context separation** for different domains
- **Privacy protection** for sensitive information

### 2. **Tool Integration**

- **Error handling** for tool failures
- **Rate limiting** for API calls
- **Caching** of frequent tool results
- **Security** for external tool access

### 3. **Performance Optimization**

- **Lazy loading** of memory content
- **Batch operations** for database queries
- **Memory compression** for storage efficiency
- **Indexed search** for fast retrieval

## Monitoring and Analytics

### 1. **Memory Usage Metrics**

```python
class MemoryMetrics:
    def __init__(self, memory):
        self.memory = memory

    def get_usage_stats(self):
        return {
            "total_memories": self.memory.count(),
            "average_memory_age": self.memory.avg_age(),
            "most_accessed": self.memory.top_accessed(10),
            "memory_size": self.memory.get_size()
        }
```

### 2. **Tool Performance Tracking**

```python
class ToolMetrics:
    def __init__(self):
        self.tool_usage = {}
        self.tool_performance = {}

    def track_tool_call(self, tool_name, duration, success):
        # Track tool usage and performance
        pass
```

## Exercises

### Beginner

1. Create a basic agent with conversation memory
2. Add a simple tool (calculator, weather)
3. Implement memory search functionality

### Intermediate

1. Build an agent with persistent SQLite memory
2. Create custom tools for specific tasks
3. Implement context-aware memory retrieval

### Advanced

1. Design a hierarchical memory system
2. Create an adaptive memory with importance scoring
3. Build a multi-agent system with shared memory

## Next Steps

After mastering memory and tools, explore:

1. **MCP Integration** (see `4 MCP/`)
2. **Custom MCP Servers** (see `5 Custom MCP Server/`)
3. **Multi-agent systems**
4. **Advanced agent architectures**

## Further Reading

- [Agno Memory Documentation](https://docs.agno.ai/memory)
- [Agent Tool Development Guide](https://docs.agno.ai/tools)
- [Memory Systems in AI](https://en.wikipedia.org/wiki/Memory_system)
- [Agent Architecture Patterns](https://arxiv.org/abs/2308.11432)

## Troubleshooting

### Common Issues:

1. **Memory Database Locks**:

   ```python
   # Use connection pooling
   memory = SQLiteMemory(db_file="memory.db", check_same_thread=False)
   ```

2. **Tool Timeout Errors**:

   ```python
   # Add timeout handling
   tool = GoogleSearchTool(timeout=30)
   ```

3. **Memory Size Growth**:
   ```python
   # Implement automatic pruning
   memory.set_auto_prune(max_memories=10000)
   ```

## Use Cases

### Business Applications:

- **Customer Service**: Remember customer history and preferences
- **Sales Assistants**: Track leads and conversation history
- **Technical Support**: Maintain context across support sessions

### Personal Applications:

- **Personal Assistants**: Learn user preferences and habits
- **Learning Companions**: Track learning progress and adapt
- **Project Management**: Maintain project context and decisions

### Research Applications:

- **Research Assistants**: Accumulate domain knowledge
- **Data Analysis**: Remember analysis patterns and insights
- **Experiment Tracking**: Maintain experimental context

---

**Note**: Memory and tool integration transforms simple chatbots into intelligent agents capable of complex reasoning, learning, and task execution. This tutorial provides the foundation for building production-ready intelligent systems that can adapt and improve over time.
