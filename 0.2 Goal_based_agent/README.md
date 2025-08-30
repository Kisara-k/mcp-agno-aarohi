# Goal-Based Agent Tutorial

## Overview

This tutorial demonstrates the implementation of **Goal-Based Agents** using LangChain and OpenAI. Unlike simple reflex agents, goal-based agents can plan sequences of actions to achieve specific objectives, making them more flexible and intelligent.

## Learning Objectives

By completing this tutorial, you will:

- Understand the concept of goal-based agents in AI
- Learn how to implement agents with specific objectives
- Explore conversation memory and state management
- Build interactive applications using Streamlit
- Compare goal-based vs. reflex agent architectures

## What is a Goal-Based Agent?

A Goal-Based Agent operates with a **specific goal or objective**:

- **Goal**: Collect user information (Name, Email, Skills)
- **Planning**: Determine what information is still needed
- **Action**: Ask appropriate questions to gather missing data
- **Completion**: Recognize when the goal is achieved

Unlike reflex agents, goal-based agents can:

- Plan sequences of actions
- Maintain state across interactions
- Adapt behavior based on progress toward goals

## Getting Started

### Prerequisites

```bash
pip install langchain langchain-openai openai python-dotenv streamlit PyMuPDF
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
0.2 Goal_based_agent/
├── goal_based_agent_v1.py      # CLI version
├── goal_based_agent_v3.py      # Streamlit web app
└── README.md                   # This tutorial
```

## How to Run

### Option 1: Command Line Interface (v1)

```bash
python goal_based_agent_v1.py
```

### Option 2: Web Interface (v3)

```bash
streamlit run goal_based_agent_v3.py
```

## Implementation Details

### 1. Goal Definition

**Primary Goal**: Collect the following 3 details from the user:

- **Full Name**
- **Email Address**
- **Skills**

**Success Criteria**: Once all information is collected, respond with:
_"Thanks! You're all set to apply. We've noted your name, email, and skills."_

### 2. Agent Architecture

```python
# Core components
llm = ChatOpenAI(model_name="gpt-3.5-turbo", temperature=0)
memory = ConversationBufferMemory(memory_key="chat_history", return_messages=True)

# Goal state tracking
application_info = {
    "name": None,
    "email": None,
    "skills": None
}
```

### 3. Information Extraction

The agent uses **Regular Expressions** to extract structured data:

```python
def extract_application_info(text: str) -> str:
    name_match = re.search(r"(?:my name is|i am)\s+([A-Z][a-z]+(?:\s+[A-Z][a-z]+)*)", text, re.IGNORECASE)
    email_match = re.search(r"\b[\w.-]+@[\w.-]+\.\w+\b", text)
    skills_match = re.search(r"(?:skills are|i know|i can use)\s+(.+)", text, re.IGNORECASE)
```

### 4. Goal-Oriented Behavior

The agent continuously checks progress toward the goal:

- **Identifies missing information**
- **Asks targeted questions**
- **Updates internal state**
- **Recognizes goal completion**

## Key Concepts Demonstrated

### 1. **Goal-Directed Behavior**

- Clear objective definition
- Progress tracking
- Completion recognition

### 2. **State Management**

- Persistent memory across conversations
- Information extraction and storage
- State-based decision making

### 3. **Natural Language Processing**

- Pattern recognition in user input
- Intent understanding
- Context-aware responses

### 4. **Interactive Applications**

- CLI and web interfaces
- Real-time conversation flow
- User experience optimization

## Comparison: Reflex vs. Goal-Based Agents

| Aspect              | Reflex Agent       | Goal-Based Agent    |
| ------------------- | ------------------ | ------------------- |
| **Decision Making** | Immediate reaction | Strategic planning  |
| **Memory**          | No memory          | Maintains state     |
| **Flexibility**     | Fixed responses    | Adaptive behavior   |
| **Complexity**      | Simple rules       | Complex reasoning   |
| **Goal Awareness**  | No goals           | Explicit objectives |

## Expected Interaction Flow

### Example Conversation:

```
Agent: Hello! I'm here to help you with your job application.
       Can you please tell me your name?

User: Hi, my name is John Smith

Agent: Nice to meet you, John! I've noted your name.
       Could you please provide your email address?

User: My email is john.smith@email.com

Agent: Great! I have your email. Now, what skills do you have?

User: I know Python, JavaScript, and machine learning

Agent: Thanks! You're all set to apply. We've noted your name,
       email, and skills.
```

## Advanced Features

### Alternative LLM Support

The code includes examples for multiple LLM providers:

```python
# OpenAI (default)
llm = ChatOpenAI(model_name="gpt-3.5-turbo", temperature=0)

# Ollama (local)
# llm = OllamaLLM(model="ollama/llama3.2", base_url="http://localhost:11434")

# Google Gemini
# llm = GoogleGenerativeAI(model="gemini-pro", google_api_key=os.environ["GOOGLE_API_KEY"])
```

### Memory Systems

- **ConversationBufferMemory**: Stores entire conversation history
- **Persistent state**: Maintains goal progress across sessions
- **Structured data extraction**: Converts natural language to structured information

## Exercises

### Beginner

1. Modify the goal to collect additional information (phone number, location)
2. Change the success message
3. Add validation for email format

### Intermediate

1. Implement multiple goals (e.g., different application types)
2. Add conversation history export
3. Create a progress indicator showing completion percentage

### Advanced

1. Implement goal hierarchies (sub-goals)
2. Add error handling for extraction failures
3. Create a multi-agent system with different specializations

## Next Steps

After mastering goal-based agents, explore:

1. **Learning Agents** (see `0.3 Learning Agent/`)
2. **Utility-based agents**
3. **Multi-agent systems**
4. **Advanced planning algorithms**

## Further Reading

- Russell, S. & Norvig, P. "Artificial Intelligence: A Modern Approach" - Chapter 2
- [LangChain Agent Documentation](https://python.langchain.com/docs/modules/agents/)
- [Goal-Oriented Action Planning](https://en.wikipedia.org/wiki/Goal-oriented_action_planning)

## Video Tutorial

**English Tutorial**: https://youtu.be/9WEqkhJMlVc

---

**Note**: Goal-based agents represent a significant step up in intelligence from reflex agents, introducing planning, memory, and objective-driven behavior that forms the foundation for more sophisticated AI systems.
