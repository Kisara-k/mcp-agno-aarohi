# Custom MCP Server Tutorial with LangGraph and Streamlit

## Overview

This tutorial demonstrates how to **build custom MCP servers** and integrate them with AI agents using LangGraph and Streamlit. You'll learn to create your own tools, deploy them as MCP servers, and build interactive web applications that use these tools.

## Learning Objectives

By completing this tutorial, you will:

- Build custom MCP servers from scratch
- Create mathematical and utility tools as MCP services
- Integrate custom MCP servers with LangGraph workflows
- Develop interactive web applications with Streamlit
- Deploy and manage custom MCP server infrastructure

## What is a Custom MCP Server?

A **Custom MCP Server** allows you to:

- **Create domain-specific tools** tailored to your needs
- **Expose APIs** as standardized MCP tools
- **Integrate legacy systems** with modern AI agents
- **Build specialized services** for specific use cases

### Server Components:

1. **Tool Definitions**: Functions that agents can call
2. **Transport Layer**: How the server communicates (stdio, HTTP)
3. **Resource Management**: Handling concurrent requests
4. **Error Handling**: Graceful failure management

## Getting Started

### Prerequisites

```bash
pip install python-dotenv langchain-mcp-adapters langgraph "langchain[openai]" mcp streamlit
```

### Environment Setup

1. **Create a `.env` file**:

   ```env
   OPENAI_API_KEY=your_openai_api_key_here
   ```

2. **Get OpenAI API Key**:
   - Visit [platform.openai.com/api-keys](https://platform.openai.com/api-keys)
   - Create a new API key
   - Add it to your `.env` file

### Project Structure

```
5 Custom MCP Server/
├── custom_mcp_server.py        # Custom MCP server implementation
├── mcp_client_langgraph.py     # LangGraph client
├── web_app.py                  # Streamlit web interface
└── README.md                   # This tutorial
```

## How to Run

### Option 1: Command Line Interface

```bash
# Terminal 1: Start custom MCP server
python custom_mcp_server.py

# Terminal 2: Run LangGraph client
python mcp_client_langgraph.py
```

### Option 2: Web Interface

```bash
# Terminal 1: Start custom MCP server
python custom_mcp_server.py

# Terminal 2: Launch Streamlit app
streamlit run web_app.py
```

## Implementation Details

### 1. Custom MCP Server Implementation

**Server Definition**:

```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("Math")  # Server name
```

**Tool Creation**:

```python
@mcp.tool()
def add(a: int, b: int) -> int:
    """Add two numbers"""
    return a + b

@mcp.tool()
def multiply(a: int, b: int) -> int:
    """Multiply two numbers"""
    return a * b

@mcp.tool()
def divide(a: float, b: float) -> float:
    """Divide a by b. Raises error if b is zero."""
    if b == 0:
        raise ValueError("Division by zero is not allowed.")
    return a / b

@mcp.tool()
def square_root(x: float) -> float:
    """Return the square root of x."""
    if x < 0:
        raise ValueError("Cannot take square root of a negative number.")
    return math.sqrt(x)

@mcp.tool()
def factorial(n: int) -> int:
    """Return factorial of n."""
    if n < 0:
        raise ValueError("Factorial is not defined for negative numbers.")
    return math.factorial(n)
```

**Server Deployment**:

```python
if __name__ == "__main__":
    # Option 1: STDIO transport
    # mcp.run(transport="stdio")

    # Option 2: HTTP transport
    mcp.run(transport="streamable-http")
```

### 2. LangGraph Client Integration

**Client Configuration**:

```python
import asyncio
from langchain_mcp_adapters.client import MCPClient
from langchain_openai import ChatOpenAI
from langgraph.graph import StateGraph, MessagesState

async def main():
    # Connect to custom MCP server
    client = MCPClient(
        transport="stdio",
        command="python",
        args=["custom_mcp_server.py"]
    )

    # Get available tools
    tools = await client.get_tools()

    # Bind tools to language model
    model = ChatOpenAI(model="gpt-4o-mini")
    model_with_tools = model.bind_tools(tools)
```

**Graph Workflow**:

```python
def call_model(state: MessagesState):
    response = model_with_tools.invoke(state["messages"])
    return {"messages": response}

# Build workflow graph
graph_builder = StateGraph(MessagesState)
graph_builder.add_node("model", call_model)
graph_builder.add_node("tools", ToolNode(tools))
graph_builder.add_edge(START, "model")
graph_builder.add_conditional_edges("model", tools_condition)
graph_builder.add_edge("tools", "model")

graph = graph_builder.compile()
```

### 3. Streamlit Web Application

**App Structure**:

```python
import streamlit as st
import asyncio
from langchain_mcp_adapters.client import MCPClient

st.title("Custom MCP Server Demo")
st.write("Chat with an AI agent that can perform mathematical calculations!")

# Initialize session state
if "messages" not in st.session_state:
    st.session_state.messages = []

# Display chat history
for message in st.session_state.messages:
    with st.chat_message(message["role"]):
        st.markdown(message["content"])

# Chat input
if prompt := st.chat_input("Ask me to perform calculations..."):
    # Add user message
    st.session_state.messages.append({"role": "user", "content": prompt})

    # Process with MCP agent
    response = await process_with_mcp_agent(prompt)

    # Add assistant response
    st.session_state.messages.append({"role": "assistant", "content": response})

    # Rerun to update display
    st.rerun()
```

## Key Features

### 1. **Mathematical Operations**

The custom MCP server provides:

- **Addition**: Add two numbers
- **Multiplication**: Multiply two numbers
- **Division**: Divide with zero-check protection
- **Square Root**: Calculate square roots with negative number protection
- **Factorial**: Calculate factorials with input validation

### 2. **Error Handling**

**Input Validation**:

```python
def divide(a: float, b: float) -> float:
    if b == 0:
        raise ValueError("Division by zero is not allowed.")
    return a / b

def square_root(x: float) -> float:
    if x < 0:
        raise ValueError("Cannot take square root of a negative number.")
    return math.sqrt(x)
```

**Graceful Failure**:

```python
try:
    result = tool_function(*args)
    return {"success": True, "result": result}
except Exception as e:
    return {"success": False, "error": str(e)}
```

### 3. **Transport Options**

**STDIO Transport** (for local development):

```python
mcp.run(transport="stdio")
```

**HTTP Transport** (for remote access):

```python
mcp.run(transport="streamable-http", port=8080)
```

## Example Interactions

### 1. **Basic Calculations**

```
User: What is 15 + 27?
Agent: I'll calculate that for you.
[add tool called with a=15, b=27]
Result: 15 + 27 = 42
```

### 2. **Complex Operations**

```
User: Calculate the square root of 144 and then multiply by 5
Agent: I'll perform these calculations step by step.
[square_root tool called with x=144]
[multiply tool called with a=12, b=5]
Result: The square root of 144 is 12, and 12 × 5 = 60
```

### 3. **Error Handling**

```
User: What is 10 divided by 0?
Agent: I'll attempt that calculation.
[divide tool called with a=10, b=0]
Error: Division by zero is not allowed.
I cannot divide by zero as it's mathematically undefined.
```

## Advanced Server Features

### 1. **Resource Management**

```python
from mcp.server.fastmcp import FastMCP
import asyncio
from concurrent.futures import ThreadPoolExecutor

class AdvancedMCPServer:
    def __init__(self):
        self.mcp = FastMCP("Advanced Math")
        self.executor = ThreadPoolExecutor(max_workers=10)

    @mcp.tool()
    async def heavy_calculation(self, n: int) -> int:
        """Perform computationally intensive calculation."""
        loop = asyncio.get_event_loop()
        result = await loop.run_in_executor(
            self.executor,
            self._compute_intensive_task,
            n
        )
        return result
```

### 2. **State Management**

```python
class StatefulMCPServer:
    def __init__(self):
        self.mcp = FastMCP("Stateful Calculator")
        self.memory = {}  # Simple in-memory storage

    @mcp.tool()
    def store_value(self, name: str, value: float) -> str:
        """Store a value with a name for later use."""
        self.memory[name] = value
        return f"Stored {value} as '{name}'"

    @mcp.tool()
    def recall_value(self, name: str) -> float:
        """Recall a previously stored value."""
        if name not in self.memory:
            raise ValueError(f"No value stored with name '{name}'")
        return self.memory[name]
```

### 3. **Data Validation**

```python
from pydantic import BaseModel, Field

class CalculationRequest(BaseModel):
    a: float = Field(..., description="First number")
    b: float = Field(..., description="Second number")
    operation: str = Field(..., regex="^(add|subtract|multiply|divide)$")

@mcp.tool()
def validated_calculation(request: CalculationRequest) -> float:
    """Perform calculation with validated input."""
    if request.operation == "add":
        return request.a + request.b
    elif request.operation == "divide" and request.b == 0:
        raise ValueError("Division by zero")
    # ... other operations
```

## Building Domain-Specific Servers

### 1. **File Operations Server**

```python
@mcp.tool()
def read_file(filepath: str) -> str:
    """Read contents of a file."""
    try:
        with open(filepath, 'r') as f:
            return f.read()
    except FileNotFoundError:
        raise ValueError(f"File not found: {filepath}")

@mcp.tool()
def write_file(filepath: str, content: str) -> str:
    """Write content to a file."""
    with open(filepath, 'w') as f:
        f.write(content)
    return f"Successfully wrote to {filepath}"
```

### 2. **API Integration Server**

```python
import requests

@mcp.tool()
def fetch_weather(city: str) -> dict:
    """Fetch weather data for a city."""
    api_key = os.getenv("WEATHER_API_KEY")
    url = f"http://api.openweathermap.org/data/2.5/weather"
    params = {"q": city, "appid": api_key, "units": "metric"}

    response = requests.get(url, params=params)
    response.raise_for_status()
    return response.json()
```

### 3. **Database Operations Server**

```python
import sqlite3

@mcp.tool()
def query_database(sql: str) -> list:
    """Execute a SQL query and return results."""
    conn = sqlite3.connect("database.db")
    cursor = conn.cursor()
    cursor.execute(sql)
    results = cursor.fetchall()
    conn.close()
    return results
```

## Deployment Strategies

### 1. **Local Development**

```python
# STDIO for local testing
mcp.run(transport="stdio")
```

### 2. **Network Deployment**

```python
# HTTP for remote access
mcp.run(transport="streamable-http", host="0.0.0.0", port=8080)
```

### 3. **Production Deployment**

```python
# With proper error handling and logging
import logging

logging.basicConfig(level=logging.INFO)

try:
    mcp.run(transport="streamable-http", host="0.0.0.0", port=8080)
except Exception as e:
    logging.error(f"Server failed to start: {e}")
```

## Best Practices

### 1. **Error Handling**

- **Validate all inputs** before processing
- **Provide meaningful error messages**
- **Log errors** for debugging
- **Gracefully handle** edge cases

### 2. **Performance**

- **Use async operations** for I/O bound tasks
- **Implement connection pooling** for databases
- **Cache frequently used results**
- **Set appropriate timeouts**

### 3. **Security**

- **Validate and sanitize** all inputs
- **Use environment variables** for secrets
- **Implement rate limiting**
- **Add authentication** for sensitive operations

### 4. **Documentation**

- **Document all tools** with clear descriptions
- **Provide usage examples**
- **Specify parameter types** and constraints
- **Include error scenarios**

## Exercises

### Beginner

1. Create a simple string manipulation MCP server
2. Build a unit conversion tool server
3. Implement basic data validation

### Intermediate

1. Create a file system operations server
2. Build a database query server
3. Implement stateful operations with memory

### Advanced

1. Create a multi-service MCP server
2. Build a server with real-time data feeds
3. Implement a distributed MCP server architecture

## Next Steps

After mastering custom MCP servers, explore:

1. **MCP Server Cloud Deployment** (see `6 MCP Server of Cloud/`)
2. **MCP with Agno Integration** (see `7 MCP with Agno/`)
3. **Production MCP server management**
4. **Advanced MCP server patterns**

## Further Reading

- [FastMCP Documentation](https://fastmcp.dev/)
- [MCP Server Development Guide](https://modelcontextprotocol.io/docs/building-servers)
- [LangGraph Integration Patterns](https://python.langchain.com/docs/langgraph)
- [Streamlit Application Development](https://docs.streamlit.io/)

## Video Tutorial

**Hindi Tutorial**: https://youtu.be/AlmdrYSDemI

## Troubleshooting

### Common Issues:

1. **Server Startup Failures**:

   ```python
   # Check port availability
   import socket
   sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
   result = sock.connect_ex(('localhost', 8080))
   if result == 0:
       print("Port is in use")
   ```

2. **Tool Discovery Issues**:

   ```python
   # Verify server is responding
   try:
       tools = await client.get_tools()
       print(f"Found {len(tools)} tools")
   except Exception as e:
       print(f"Server connection failed: {e}")
   ```

3. **Performance Problems**:

   ```python
   # Add timeout to operations
   import asyncio

   @mcp.tool()
   async def slow_operation(data: str) -> str:
       try:
           result = await asyncio.wait_for(process_data(data), timeout=30)
           return result
       except asyncio.TimeoutError:
           raise ValueError("Operation timed out")
   ```

---

**Note**: Building custom MCP servers opens up unlimited possibilities for extending AI agents with domain-specific capabilities. This tutorial provides the foundation for creating production-ready, scalable MCP servers that can transform how AI agents interact with your systems and data.
