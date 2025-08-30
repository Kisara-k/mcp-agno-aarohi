# Agno Framework Tutorial

## Overview

This tutorial introduces the **Agno Framework**, a powerful Python library for building AI agents quickly and efficiently. Agno simplifies agent creation by providing high-level abstractions while maintaining flexibility for complex use cases.

## Learning Objectives

By completing this tutorial, you will:

- Understand the Agno framework architecture
- Learn how to create AI agents with minimal code
- Explore different language models integration
- Implement conversational AI agents
- Compare Agno with traditional agent frameworks

## What is Agno?

**Agno** is a modern Python framework for building AI agents that focuses on:

- **Simplicity**: Create agents with just a few lines of code
- **Flexibility**: Support for multiple language models and tools
- **Performance**: Optimized for production use
- **Extensibility**: Easy to add custom capabilities

## Getting Started

### Prerequisites

```bash
pip install -U agno
pip install python-dotenv
```

### Environment Setup

1. **Create a `.env` file** in the project root:

   ```env
   OPENAI_API_KEY=your_openai_api_key_here
   ```

2. **Get OpenAI API Key**:
   - Visit [OpenAI API](https://platform.openai.com/api-keys)
   - Create a new API key
   - Add it to your `.env` file

### Project Structure

```
1 Agno/
├── L1_agno.py                  # Basic Agno agent implementation
└── README.md                   # This tutorial
```

## How to Run

```bash
python L1_agno.py
```

## Implementation Details

### 1. Basic Agent Creation

The simplest Agno agent requires only a few lines:

```python
from agno.agent import Agent
from agno.models.openai import OpenAIChat

agno_agent = Agent(
    model=OpenAIChat(id="gpt-3.5-turbo"),
    description="Agno Q&A agent",
    markdown=False
)

agno_agent.print_response("What is the capital of India?", stream=True)
```

### 2. Component Breakdown

**Agent Class**: The main agent controller

- Handles conversation flow
- Manages model interactions
- Provides response formatting

**Model Integration**: Support for multiple LLM providers

- OpenAI (GPT-3.5, GPT-4)
- Anthropic (Claude)
- Google (Gemini)
- Local models via Ollama

**Response Handling**: Flexible output options

- Streaming responses
- Markdown formatting
- Custom response processing

## Key Features

### 1. **Multi-Model Support**

```python
# OpenAI Models
from agno.models.openai import OpenAIChat
model = OpenAIChat(id="gpt-4o-mini")

# Anthropic Models
from agno.models.anthropic import AnthropicChat
model = AnthropicChat(id="claude-3-sonnet")

# Google Models
from agno.models.google import GoogleGenerativeAI
model = GoogleGenerativeAI(id="gemini-pro")
```

### 2. **Tool Integration**

```python
from agno.tools.web import WebSearchTool
from agno.tools.file import FileManager

agent = Agent(
    model=OpenAIChat(id="gpt-3.5-turbo"),
    tools=[WebSearchTool(), FileManager()],
    description="Agent with web search and file management capabilities"
)
```

### 3. **Memory Systems**

```python
from agno.memory import ConversationMemory

agent = Agent(
    model=OpenAIChat(id="gpt-3.5-turbo"),
    memory=ConversationMemory(),
    description="Agent with conversation memory"
)
```

### 4. **Custom Instructions**

```python
agent = Agent(
    model=OpenAIChat(id="gpt-3.5-turbo"),
    description="You are a helpful Python programming assistant",
    instructions=[
        "Always provide code examples",
        "Explain concepts clearly",
        "Suggest best practices"
    ]
)
```

## Comparison: Agno vs. Traditional Frameworks

| Aspect               | LangChain | Traditional Agents | Agno      |
| -------------------- | --------- | ------------------ | --------- |
| **Setup Complexity** | High      | Very High          | Low       |
| **Code Lines**       | 50+       | 100+               | 5-10      |
| **Learning Curve**   | Steep     | Very Steep         | Gentle    |
| **Flexibility**      | High      | High               | High      |
| **Performance**      | Good      | Variable           | Optimized |

## Advanced Features

### 1. **Streaming Responses**

```python
# Enable streaming for real-time responses
agno_agent.print_response("Explain quantum computing", stream=True)
```

### 2. **Response Formatting**

```python
# Control output formatting
agent = Agent(
    model=OpenAIChat(id="gpt-3.5-turbo"),
    markdown=True,  # Enable markdown formatting
    show_tool_calls=True  # Show tool usage
)
```

### 3. **Error Handling**

```python
try:
    response = agent.run("Your question here")
except Exception as e:
    print(f"Agent error: {e}")
```

## Practical Examples

### 1. **Q&A Agent**

```python
qa_agent = Agent(
    model=OpenAIChat(id="gpt-3.5-turbo"),
    description="Knowledgeable Q&A assistant",
    markdown=False
)

qa_agent.print_response("What is the capital of India?", stream=True)
```

### 2. **Code Assistant**

```python
code_agent = Agent(
    model=OpenAIChat(id="gpt-4"),
    description="Python coding assistant",
    instructions=[
        "Provide working code examples",
        "Include error handling",
        "Explain the code logic"
    ]
)

code_agent.print_response("Write a function to sort a list of dictionaries by a key")
```

### 3. **Research Assistant**

```python
from agno.tools.web import WebSearchTool

research_agent = Agent(
    model=OpenAIChat(id="gpt-3.5-turbo"),
    tools=[WebSearchTool()],
    description="Research assistant with web search capabilities"
)

research_agent.print_response("Find recent developments in quantum computing")
```

## Best Practices

### 1. **Model Selection**

- **GPT-3.5-turbo**: Fast, cost-effective for simple tasks
- **GPT-4**: Higher quality responses for complex reasoning
- **GPT-4o-mini**: Balanced performance and cost

### 2. **Description Writing**

- Be specific about the agent's role
- Include expected behavior patterns
- Define output format preferences

### 3. **Tool Usage**

- Only include necessary tools
- Test tool combinations thoroughly
- Handle tool errors gracefully

### 4. **Performance Optimization**

- Use streaming for better user experience
- Cache frequently used responses
- Monitor token usage

## Performance Considerations

### Response Time Factors:

1. **Model Choice**: GPT-4 > GPT-3.5-turbo (speed)
2. **Tool Usage**: More tools = slower responses
3. **Context Length**: Longer conversations increase latency
4. **Streaming**: Improves perceived performance

### Cost Optimization:

1. **Choose appropriate models** for task complexity
2. **Limit context window** size when possible
3. **Use caching** for repeated queries
4. **Monitor token usage** closely

## Next Steps

After mastering basic Agno usage, explore:

1. **Agno RAG** (see `2 Agno RAG/`)
2. **Agno Memory** (see `3 Agno Memory/`)
3. **MCP with Agno** (see `7 MCP with Agno/`)
4. **Custom tool development**
5. **Multi-agent systems**

## Exercises

### Beginner

1. Create a simple calculator agent
2. Build a weather information agent
3. Implement a joke-telling agent

### Intermediate

1. Create an agent with custom tools
2. Build a multi-turn conversation agent
3. Implement error handling and recovery

### Advanced

1. Create a specialized domain expert agent
2. Build an agent with custom memory system
3. Implement agent-to-agent communication

## Troubleshooting

### Common Issues:

1. **API Key Errors**

   ```python
   # Verify API key is properly loaded
   import os
   print(os.getenv("OPENAI_API_KEY"))
   ```

2. **Model Not Found**

   ```python
   # Check available models
   from agno.models.openai import OpenAIChat
   print(OpenAIChat.available_models())
   ```

3. **Import Errors**
   ```bash
   # Ensure Agno is properly installed
   pip install -U agno
   ```

## Further Reading

- [Agno Documentation](https://docs.agno.ai/)
- [Agno GitHub Repository](https://github.com/agno-ai/agno)
- [Agno Examples](https://github.com/agno-ai/agno/tree/main/examples)
- [Building AI Agents Best Practices](https://docs.agno.ai/best-practices)

## Why Choose Agno?

### Advantages:

- **Rapid Prototyping**: Build agents in minutes
- **Production Ready**: Optimized for real-world use
- **Extensive Documentation**: Comprehensive guides and examples
- **Active Community**: Regular updates and support
- **Framework Agnostic**: Works with any Python project

### Use Cases:

- **Customer Support**: Automated help desk agents
- **Content Creation**: Writing and editing assistants
- **Data Analysis**: Research and insight generation
- **Education**: Tutoring and explanation systems
- **Automation**: Task automation and workflow management

---

**Note**: Agno represents the next generation of agent frameworks, prioritizing developer experience while maintaining the power and flexibility needed for sophisticated AI applications. This tutorial provides the foundation for building production-ready AI agents with minimal complexity.
