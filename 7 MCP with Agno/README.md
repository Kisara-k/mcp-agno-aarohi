# MCP Integration with Agno Tutorial

## Overview

This tutorial demonstrates the powerful integration of **Model Context Protocol (MCP) servers with the Agno framework**. You'll learn to combine the simplicity of Agno agents with the extensibility of MCP tools, creating sophisticated AI systems that can access external services seamlessly.

## Learning Objectives

By completing this tutorial, you will:

- Integrate MCP servers with Agno agents
- Use multiple MCP servers simultaneously with Agno
- Build custom MCP servers for Agno agents
- Deploy cloud-based MCP services for Agno
- Compare different integration patterns and architectures

## Why Combine MCP with Agno?

**Agno + MCP** provides the best of both worlds:

- **Agno's Simplicity**: Easy agent creation and management
- **MCP's Extensibility**: Standardized tool integration
- **Unified Interface**: Consistent API across all tools
- **Scalability**: Add new capabilities without code changes
- **Production Ready**: Battle-tested architecture patterns

### Integration Benefits:

- **Rapid Development**: Build complex agents in minutes
- **Tool Ecosystem**: Access to growing MCP server library
- **Maintainability**: Standardized tool interfaces
- **Flexibility**: Easy to swap and upgrade tools

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
   ```

2. **Get OpenAI API Key**:
   - Visit [platform.openai.com/api-keys](https://platform.openai.com/api-keys)
   - Create a new API key
   - Add it to your `.env` file

### Project Structure

```
7 MCP with Agno/
├── 4.py                        # Basic MCP + Agno integration
├── single_mcp_server.py        # Single MCP server with Agno
├── multipl_mcp_server.py       # Multiple MCP servers with Agno
├── multipl_mcp_custom.py       # Custom MCP servers with Agno
├── custom_mcp_server.py        # Custom MCP server implementation
└── README.md                   # This tutorial
```

## How to Run

### Option 1: Basic Integration

```bash
python 4.py
```

### Option 2: Single MCP Server

```bash
python single_mcp_server.py
```

### Option 3: Multiple MCP Servers

```bash
# Start your MCP servers first, then:
python multipl_mcp_server.py
```

### Option 4: Custom MCP Servers

```bash
# Terminal 1: Start custom MCP server
python custom_mcp_server.py

# Terminal 2: Run Agno client
python multipl_mcp_custom.py
```

## Implementation Patterns

### 1. **Basic MCP Integration with Agno**

**Simple Remote MCP Server**:

```python
import asyncio
from agno.agent import Agent
from agno.models.openai import OpenAIChat
from agno.tools.mcp import MultiMCPTools

async def run_agent(message: str):
    # Configure remote MCP server
    math_server_url = "http://31.97.233.200:8080/mcp"

    multi_mcp_tools = MultiMCPTools(
        commands=[],                                # No local stdio MCPs
        urls=[math_server_url],                     # Only remote MCP
        urls_transports=["streamable-http"],
    )

    # Connect to MCP servers
    await multi_mcp_tools.connect()

    # Create Agno agent with MCP tools
    agent = Agent(
        model=OpenAIChat(id="gpt-4o-mini"),
        tools=[multi_mcp_tools],
        instructions=["You are a helpful math assistant"],
        markdown=True,
    )

    # Use the agent
    agent.print_response(message, stream=True)

    # Cleanup
    await multi_mcp_tools.close()

# Run the agent
asyncio.run(run_agent("Calculate 25 * 8 + 100"))
```

### 2. **Single MCP Server Integration**

**Local Calculator Server**:

```python
from agno.agent import Agent
from agno.models.openai import OpenAIChat
from agno.tools.mcp import MCPTools

def create_calculator_agent():
    # Configure local MCP server
    mcp_tools = MCPTools(
        command="python",
        args=["-m", "mcp_server_calculator"]
    )

    # Create agent with calculator tools
    agent = Agent(
        model=OpenAIChat(id="gpt-4o-mini"),
        tools=[mcp_tools],
        description="Math expert with calculator capabilities",
        instructions=[
            "Use the calculator for all mathematical operations",
            "Show step-by-step calculations",
            "Verify your results"
        ]
    )

    return agent

# Interactive usage
agent = create_calculator_agent()
agent.print_response("What is the square root of 2025?", stream=True)
```

### 3. **Multiple MCP Servers**

**Calculator + Wikipedia Integration**:

```python
from agno.agent import Agent
from agno.models.openai import OpenAIChat
from agno.tools.mcp import MultiMCPTools

async def create_research_agent():
    multi_mcp_tools = MultiMCPTools(
        commands=[
            ["python", "-m", "mcp_server_calculator"],     # Calculator
            ["python", "-m", "mcp_server_wikipedia"]       # Wikipedia
        ]
    )

    await multi_mcp_tools.connect()

    agent = Agent(
        model=OpenAIChat(id="gpt-4o-mini"),
        tools=[multi_mcp_tools],
        description="Research assistant with calculation and Wikipedia access",
        instructions=[
            "Use Wikipedia for factual information",
            "Use calculator for mathematical operations",
            "Combine information from multiple sources",
            "Provide well-researched, accurate responses"
        ]
    )

    return agent, multi_mcp_tools

# Example usage
async def main():
    agent, tools = await create_research_agent()

    agent.print_response(
        "Research the population of Tokyo and calculate how many times larger it is than New York",
        stream=True
    )

    await tools.close()

asyncio.run(main())
```

### 4. **Custom MCP Server Integration**

**Custom File Operations Server**:

```python
# custom_file_server.py
from mcp.server.fastmcp import FastMCP
import os

mcp = FastMCP("FileOperations")

@mcp.tool()
def list_files(directory: str = ".") -> list:
    """List files in a directory"""
    try:
        return os.listdir(directory)
    except Exception as e:
        return [f"Error: {str(e)}"]

@mcp.tool()
def read_file(filepath: str) -> str:
    """Read contents of a file"""
    try:
        with open(filepath, 'r') as f:
            return f.read()
    except Exception as e:
        return f"Error reading file: {str(e)}"

@mcp.tool()
def write_file(filepath: str, content: str) -> str:
    """Write content to a file"""
    try:
        with open(filepath, 'w') as f:
            f.write(content)
        return f"Successfully wrote to {filepath}"
    except Exception as e:
        return f"Error writing file: {str(e)}"

if __name__ == "__main__":
    mcp.run(transport="stdio")
```

**Agno Client for Custom Server**:

```python
from agno.agent import Agent
from agno.models.openai import OpenAIChat
from agno.tools.mcp import MCPTools

def create_file_agent():
    # Connect to custom file operations server
    mcp_tools = MCPTools(
        command="python",
        args=["custom_file_server.py"]
    )

    agent = Agent(
        model=OpenAIChat(id="gpt-4o-mini"),
        tools=[mcp_tools],
        description="File management assistant",
        instructions=[
            "Help users manage their files safely",
            "Always confirm before writing or deleting files",
            "Provide clear feedback on file operations"
        ]
    )

    return agent

# Usage
agent = create_file_agent()
agent.print_response("List the files in the current directory", stream=True)
```

## Key Integration Patterns

### 1. **Local MCP Servers**

**STDIO Transport**:

```python
mcp_tools = MCPTools(
    command="python",
    args=["-m", "mcp_server_calculator"]
)
```

**Multiple Local Servers**:

```python
multi_mcp_tools = MultiMCPTools(
    commands=[
        ["python", "-m", "mcp_server_calculator"],
        ["python", "-m", "mcp_server_wikipedia"],
        ["node", "weather-server.js"]
    ]
)
```

### 2. **Remote MCP Servers**

**HTTP Transport**:

```python
multi_mcp_tools = MultiMCPTools(
    urls=["http://api.example.com:8080/mcp"],
    urls_transports=["streamable-http"]
)
```

**Mixed Local and Remote**:

```python
multi_mcp_tools = MultiMCPTools(
    commands=[["python", "-m", "mcp_server_calculator"]],  # Local
    urls=["http://remote-server:8080/mcp"],                # Remote
    urls_transports=["streamable-http"]
)
```

### 3. **Dynamic Server Discovery**

```python
async def discover_and_connect_mcps():
    available_servers = [
        {"name": "calculator", "command": ["python", "-m", "mcp_server_calculator"]},
        {"name": "weather", "url": "http://weather-api:8080/mcp"},
        {"name": "files", "command": ["python", "file_server.py"]}
    ]

    active_commands = []
    active_urls = []

    for server in available_servers:
        if await test_server_availability(server):
            if "command" in server:
                active_commands.append(server["command"])
            elif "url" in server:
                active_urls.append(server["url"])

    return MultiMCPTools(
        commands=active_commands,
        urls=active_urls,
        urls_transports=["streamable-http"] * len(active_urls)
    )
```

## Available MCP Servers for Agno

### Official Servers:

- **mcp-server-calculator**: Mathematical operations
- **mcp-server-wikipedia**: Wikipedia search and articles
- **mcp-server-filesystem**: File system operations
- **mcp-server-git**: Git repository management
- **mcp-server-postgres**: PostgreSQL database operations

### Community Servers:

- **mcp-server-weather**: Weather information
- **mcp-server-email**: Email sending/receiving
- **mcp-server-calendar**: Calendar operations
- **mcp-server-web-search**: Web search capabilities

**Explore more**: [github.com/modelcontextprotocol/servers](https://github.com/modelcontextprotocol/servers)

## Example Use Cases

### 1. **Research Assistant**

```python
async def create_research_assistant():
    tools = MultiMCPTools(
        commands=[
            ["python", "-m", "mcp_server_wikipedia"],
            ["python", "-m", "mcp_server_calculator"]
        ],
        urls=["http://web-search-server:8080/mcp"],
        urls_transports=["streamable-http"]
    )

    await tools.connect()

    agent = Agent(
        model=OpenAIChat(id="gpt-4"),
        tools=[tools],
        description="Comprehensive research assistant",
        instructions=[
            "Research topics thoroughly using multiple sources",
            "Calculate statistics and metrics when relevant",
            "Cross-reference information for accuracy",
            "Provide citations and sources"
        ]
    )

    return agent, tools

# Usage
agent, tools = await create_research_assistant()
agent.print_response(
    "Research the economic impact of renewable energy and calculate the projected growth rate",
    stream=True
)
```

### 2. **Development Assistant**

```python
def create_dev_assistant():
    tools = MultiMCPTools(
        commands=[
            ["python", "-m", "mcp_server_git"],
            ["python", "file_operations_server.py"],
            ["python", "-m", "mcp_server_calculator"]
        ]
    )

    agent = Agent(
        model=OpenAIChat(id="gpt-4"),
        tools=[tools],
        description="Software development assistant",
        instructions=[
            "Help with code review and file management",
            "Use git operations for version control tasks",
            "Calculate code metrics and statistics",
            "Maintain project organization"
        ]
    )

    return agent

# Usage
agent = create_dev_assistant()
agent.print_response("Check git status and list recent commits", stream=True)
```

### 3. **Data Analysis Assistant**

```python
async def create_data_analyst():
    tools = MultiMCPTools(
        commands=[
            ["python", "-m", "mcp_server_calculator"],
            ["python", "database_server.py"]
        ],
        urls=["http://data-api:8080/mcp"],
        urls_transports=["streamable-http"]
    )

    await tools.connect()

    agent = Agent(
        model=OpenAIChat(id="gpt-4"),
        tools=[tools],
        description="Data analysis specialist",
        instructions=[
            "Perform statistical calculations",
            "Query databases for insights",
            "Generate data summaries and reports",
            "Identify trends and patterns"
        ]
    )

    return agent, tools
```

## Best Practices

### 1. **Tool Management**

- **Test server availability** before connecting
- **Implement graceful fallbacks** when servers are unavailable
- **Monitor tool performance** and response times
- **Cache frequent tool results** when appropriate

### 2. **Error Handling**

```python
async def robust_agent_creation():
    try:
        tools = MultiMCPTools(commands=[["python", "-m", "mcp_server_calculator"]])
        await tools.connect()

        agent = Agent(
            model=OpenAIChat(id="gpt-4o-mini"),
            tools=[tools],
            description="Math assistant with fallback"
        )

        return agent, tools
    except Exception as e:
        # Fallback to agent without tools
        print(f"MCP connection failed: {e}")
        agent = Agent(
            model=OpenAIChat(id="gpt-4o-mini"),
            description="Basic assistant (no external tools)"
        )
        return agent, None
```

### 3. **Resource Management**

```python
async def safe_agent_usage():
    agent, tools = await create_agent_with_mcp()

    try:
        # Use the agent
        agent.print_response("Your query here", stream=True)
    finally:
        # Always cleanup
        if tools:
            await tools.close()
```

### 4. **Performance Optimization**

- **Use connection pooling** for frequently accessed servers
- **Implement caching** for expensive operations
- **Monitor memory usage** with multiple tools
- **Set appropriate timeouts** for tool operations

## Exercises

### Beginner

1. Create an Agno agent with a single MCP server
2. Build a basic calculator assistant using MCP tools
3. Implement error handling for MCP connection failures

### Intermediate

1. Integrate multiple MCP servers with one Agno agent
2. Create a custom MCP server for specific domain tasks
3. Build a research assistant combining Wikipedia and calculator

### Advanced

1. Implement dynamic MCP server discovery and connection
2. Create a multi-agent system sharing MCP tools
3. Build a production-ready agent with comprehensive error handling

## Next Steps

After mastering MCP + Agno integration:

1. **Explore advanced agent architectures**
2. **Build custom MCP servers for specific domains**
3. **Implement multi-agent systems**
4. **Deploy production Agno + MCP solutions**

## Further Reading

- [Agno Documentation](https://docs.agno.ai/)
- [MCP Specification](https://spec.modelcontextprotocol.io/)
- [Building Production AI Agents](https://docs.agno.ai/production)
- [MCP Server Development](https://modelcontextprotocol.io/docs/building-servers)

## Video Tutorials

**Hindi Tutorial**: https://youtu.be/iN1Jv_dFBco

## Troubleshooting

### Common Issues:

1. **MCP Connection Failures**:

   ```python
   # Test MCP server availability
   try:
       await multi_mcp_tools.connect()
       print("MCP connection successful")
   except Exception as e:
       print(f"MCP connection failed: {e}")
   ```

2. **Tool Discovery Issues**:

   ```python
   # Debug tool discovery
   tools = await multi_mcp_tools.get_tools()
   print(f"Available tools: {[tool.name for tool in tools]}")
   ```

3. **Performance Issues**:

   ```python
   # Add timeouts and error handling
   agent = Agent(
       model=OpenAIChat(id="gpt-4o-mini", timeout=30),
       tools=[multi_mcp_tools],
       description="Agent with timeout protection"
   )
   ```

4. **Memory Issues with Multiple Tools**:

   ```python
   # Monitor and limit resource usage
   import psutil

   if psutil.virtual_memory().percent > 80:
       print("High memory usage detected")
       # Implement cleanup or reduction strategies
   ```

---

**Note**: The integration of MCP with Agno represents the future of AI agent development - combining the simplicity and power of modern agent frameworks with the extensibility and standardization of protocol-based tool integration. This tutorial provides the foundation for building sophisticated, scalable, and maintainable AI systems.
