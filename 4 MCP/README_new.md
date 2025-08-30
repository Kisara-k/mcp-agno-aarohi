# Model Context Protocol (MCP) Tutorial

## Overview

This tutorial introduces the **Model Context Protocol (MCP)**, a standardized way for AI agents to interact with external tools and services. MCP enables agents to access weather data, perform calculations, manage files, and much more through a unified interface.

## Learning Objectives

By completing this tutorial, you will:

- Understand the Model Context Protocol architecture
- Learn to set up and use multiple MCP servers
- Integrate weather and calculator services with AI agents
- Build LangGraph workflows with MCP tools
- Compare MCP with traditional API integrations

## What is Model Context Protocol (MCP)?

**MCP** is a protocol that standardizes how AI applications connect to external data sources and tools:

- **Standardized Interface**: Consistent way to access different services
- **Bidirectional Communication**: Tools can provide context and capabilities
- **Security**: Controlled access to external resources
- **Extensibility**: Easy to add new tools and services

### MCP Components:

1. **MCP Server**: Provides tools and resources (weather, calculator, etc.)
2. **MCP Client**: Consumes tools and resources (your AI agent)
3. **Transport Layer**: Communication between client and server (stdio, HTTP)

## Getting Started

### Prerequisites

```bash
pip install python-dotenv langchain-mcp-adapters langgraph "langchain[openai]" mcp
pip install mcp-server-calculator
```

### Environment Setup

1. **Create a `.env` file**:

   ```env
   OPENAI_API_KEY=your_openai_api_key_here
   OWM_API_KEY=your_openweathermap_api_key_here
   ```

2. **Get Required API Keys**:
   - **OpenAI**: [platform.openai.com/api-keys](https://platform.openai.com/api-keys)
   - **OpenWeatherMap**: [openweathermap.org/api](https://openweathermap.org/api)

### Project Structure

```
4 MCP/
├── 1.py                        # Main MCP client implementation
└── README.md                   # This tutorial
```

## Setting Up MCP Servers

### 1. Weather MCP Server Setup

**Step 1: Install Go**

- Download from: [go.dev/dl](https://go.dev/dl/)
- Follow installation instructions for your OS

**Step 2: Clone and Build Weather Server**

```bash
git clone https://github.com/mschneider82/mcp-openweather.git
cd mcp-openweather
go build -o mcp-weather
```

**Step 3: Get OpenWeatherMap API Key**

1. Visit [openweathermap.org/api](https://openweathermap.org/api)
2. Create a free account
3. Copy your API key
4. Add to `.env` file as `OWM_API_KEY`

### 2. Calculator MCP Server Setup

**Installation**:

```bash
pip install mcp-server-calculator
```

**No additional configuration required** - calculator server is ready to use.

## How to Run

### Option 1: Single Server (Weather Only)

```bash
# Terminal 1: Start weather server
./mcp-weather

# Terminal 2: Run client
python 1.py
```

### Option 2: Multiple Servers (Weather + Calculator)

```bash
# Terminal 1: Weather server
./mcp-weather

# Terminal 2: Calculator server
python -m mcp_server_calculator

# Terminal 3: Client
python 1.py
```

## Implementation Details

### 1. MCP Client Configuration

```python
from langchain_mcp_adapters.client import MultiServerMCPClient

client = MultiServerMCPClient({
    "weather": {
        "transport": "stdio",
        "command": "E:/langraph_mcp-demo/mcp-openweather/mcp-weather.exe",
        "args": [],
        "env": {"OWM_API_KEY": owm_key}
    },
    "calculator": {
        "transport": "stdio",
        "command": "python",
        "args": ["-m", "mcp_server_calculator"]
    }
})
```

### 2. Tool Integration with LangGraph

```python
from langgraph.graph import StateGraph, MessagesState, START
from langgraph.prebuilt import ToolNode, tools_condition

# Get tools from MCP servers
tools = await client.get_tools()

# Bind tools to language model
model = ChatOpenAI(model="gpt-4o-mini", api_key=openai_key)
model_with_tools = model.bind_tools(tools)

# Create LangGraph workflow
def call_model(state: MessagesState):
    response = model_with_tools.invoke(state["messages"])
    return {"messages": response}

# Build graph
graph_builder = StateGraph(MessagesState)
graph_builder.add_node("model", call_model)
graph_builder.add_node("tools", ToolNode(tools))
graph_builder.add_edge(START, "model")
graph_builder.add_conditional_edges("model", tools_condition)
graph_builder.add_edge("tools", "model")
```

### 3. Interactive Agent Execution

```python
graph = graph_builder.compile()

while True:
    user_input = input("User: ")
    if user_input.lower() in ["quit", "exit", "q"]:
        break

    for chunk in graph.stream(
        {"messages": [HumanMessage(content=user_input)]},
        stream_mode="values"
    ):
        chunk["messages"][-1].pretty_print()
```

## Key Concepts

### 1. **Transport Protocols**

**STDIO Transport**:

```python
"transport": "stdio",
"command": "python",
"args": ["-m", "mcp_server_calculator"]
```

**HTTP Transport**:

```python
"transport": "http",
"url": "http://localhost:8080/mcp"
```

### 2. **Server Configuration**

**Environment Variables**:

```python
"env": {"OWM_API_KEY": owm_key}
```

**Command Arguments**:

```python
"args": ["--config", "config.json", "--verbose"]
```

### 3. **Tool Discovery**

```python
# Automatically discover available tools
tools = await client.get_tools()

# List tool capabilities
for tool in tools:
    print(f"Tool: {tool.name}")
    print(f"Description: {tool.description}")
    print(f"Parameters: {tool.parameters}")
```

## Available MCP Servers

### Official Servers:

- **Weather**: OpenWeatherMap integration
- **Calculator**: Mathematical computations
- **File System**: File operations
- **Database**: SQL query execution
- **Git**: Repository management

### Community Servers:

- **Web Search**: Google, Bing search
- **Email**: Send/receive emails
- **Calendar**: Event management
- **Social Media**: Twitter, LinkedIn integration
- **Cloud Services**: AWS, Azure tools

**Explore more**: [github.com/modelcontextprotocol/servers](https://github.com/modelcontextprotocol/servers)

## Example Interactions

### 1. **Weather Queries**

```
User: What's the weather in New York?
Assistant: I'll check the current weather in New York for you.

[Weather tool called]

The current weather in New York is 72°F with clear skies and light winds from the southwest at 8 mph.
```

### 2. **Mathematical Calculations**

```
User: Calculate the square root of 144
Assistant: I'll calculate that for you.

[Calculator tool called]

The square root of 144 is 12.
```

### 3. **Combined Tool Usage**

```
User: What's the weather in London and calculate 25% of 200?
Assistant: I'll get the weather information for London and perform that calculation.

[Weather tool called]
[Calculator tool called]

The current weather in London is 18°C with partly cloudy skies.
25% of 200 equals 50.
```

## Advanced Features

### 1. **Error Handling**

```python
try:
    tools = await client.get_tools()
except Exception as e:
    print(f"Failed to connect to MCP server: {e}")
    # Fallback behavior
```

### 2. **Resource Management**

```python
# Proper cleanup
try:
    # Use MCP client
    pass
finally:
    await client.close()
```

### 3. **Custom Tool Configuration**

```python
# Configure tool-specific parameters
weather_tool = WeatherTool(
    units="metric",
    language="en",
    timeout=30
)
```

## Best Practices

### 1. **Server Management**

- **Keep servers running** during agent execution
- **Monitor server health** and restart if needed
- **Use process managers** for production deployments
- **Implement graceful shutdowns**

### 2. **Error Handling**

- **Check server availability** before tool calls
- **Implement retry logic** for transient failures
- **Provide fallback options** when tools fail
- **Log errors** for debugging

### 3. **Security**

- **Validate API keys** before starting servers
- **Use environment variables** for sensitive data
- **Implement access controls** for production
- **Monitor resource usage**

### 4. **Performance**

- **Cache frequent tool results**
- **Implement connection pooling**
- **Use async operations** where possible
- **Monitor response times**

## Monitoring and Debugging

### 1. **Server Status Monitoring**

```python
async def check_server_health():
    try:
        tools = await client.get_tools()
        return len(tools) > 0
    except:
        return False
```

### 2. **Tool Usage Analytics**

```python
tool_usage_stats = {
    "weather": {"calls": 0, "errors": 0},
    "calculator": {"calls": 0, "errors": 0}
}

def track_tool_usage(tool_name, success=True):
    tool_usage_stats[tool_name]["calls"] += 1
    if not success:
        tool_usage_stats[tool_name]["errors"] += 1
```

## Exercises

### Beginner

1. Set up a single MCP server (calculator)
2. Create a basic client to use the calculator
3. Add error handling for server failures

### Intermediate

1. Configure multiple MCP servers simultaneously
2. Build a conversational agent using MCP tools
3. Implement tool usage logging and monitoring

### Advanced

1. Create a custom MCP server for a specific domain
2. Build a multi-agent system with shared MCP tools
3. Implement dynamic server discovery and configuration

## Next Steps

After mastering basic MCP usage, explore:

1. **Custom MCP Servers** (see `5 Custom MCP Server/`)
2. **MCP with Agno Integration** (see `7 MCP with Agno/`)
3. **Advanced MCP patterns and architectures**
4. **Production MCP deployments**

## Further Reading

- [MCP Specification](https://spec.modelcontextprotocol.io/)
- [MCP Server Repository](https://github.com/modelcontextprotocol/servers)
- [LangChain MCP Adapters](https://python.langchain.com/docs/integrations/tools/mcp)
- [Building Custom MCP Servers](https://modelcontextprotocol.io/docs/building-servers)

## Video Tutorial

**Hindi Tutorial**: https://youtu.be/_KB2146fFYs

## Troubleshooting

### Common Issues:

1. **Server Connection Failures**:

   ```bash
   # Check if server is running
   ps aux | grep mcp-weather

   # Restart server if needed
   ./mcp-weather
   ```

2. **API Key Errors**:

   ```python
   # Verify API key is loaded
   import os
   print(os.getenv("OWM_API_KEY"))
   ```

3. **Tool Discovery Issues**:

   ```python
   # Debug tool discovery
   try:
       tools = await client.get_tools()
       print(f"Found {len(tools)} tools")
   except Exception as e:
       print(f"Tool discovery failed: {e}")
   ```

4. **Performance Issues**:
   ```python
   # Add timeouts to tool calls
   tools = await asyncio.wait_for(client.get_tools(), timeout=30)
   ```

---

**Note**: The Model Context Protocol represents a significant advancement in AI agent architecture, providing a standardized way to integrate external tools and services. This tutorial provides the foundation for building powerful, tool-enabled AI agents that can interact with the real world through MCP servers.
