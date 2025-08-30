# MCP Server Cloud Deployment Tutorial

## Overview

This tutorial demonstrates deploying **MCP servers to the cloud** for production use. You'll learn to set up remote MCP servers, configure cloud infrastructure, and build clients that can access MCP services from anywhere on the internet.

## Learning Objectives

By completing this tutorial, you will:

- Deploy MCP servers to cloud platforms (VPS, AWS, Azure, etc.)
- Configure HTTP transport for remote access
- Implement secure cloud-based MCP communications
- Build clients that connect to remote MCP servers
- Manage production MCP server infrastructure

## Why Deploy MCP Servers to the Cloud?

**Cloud deployment offers**:

- **Scalability**: Handle multiple concurrent clients
- **Availability**: 24/7 server uptime
- **Accessibility**: Access from anywhere
- **Resource Management**: Dedicated compute resources
- **Security**: Isolated environment with proper access controls

## Getting Started

### Prerequisites

```bash
pip install python-dotenv langchain-mcp-adapters langgraph "langchain[openai]" mcp uvicorn
```

### Environment Setup

1. **Local `.env` file**:

   ```env
   OPENAI_API_KEY=your_openai_api_key_here
   ```

2. **Cloud Server Environment**:
   ```bash
   # On your VPS/cloud server
   export HOST=0.0.0.0
   export PORT=8080
   ```

### Project Structure

```
6 MCP Server of Cloud/
├── custom_server.py            # Cloud-deployable MCP server
├── mcp_client_langgraph.py     # Client for remote MCP server
└── README.md                   # This tutorial
```

## Deployment Options

### Option 1: VPS Deployment (Recommended for beginners)

1. Get a VPS (DigitalOcean, Linode, Vultr)
2. Install Python and dependencies
3. Deploy and run the MCP server

### Option 2: Container Deployment (Docker)

1. Create Docker container with MCP server
2. Deploy to any container platform
3. Expose HTTP endpoint

### Option 3: Serverless Deployment

1. Package MCP server for serverless platforms
2. Deploy to AWS Lambda, Google Cloud Functions
3. Configure API Gateway

## Implementation Details

### 1. Cloud-Ready MCP Server

**Server Configuration**:

```python
import logging
import os
import uvicorn
from mcp.server.fastmcp import FastMCP

logging.basicConfig(level=logging.INFO, format='[%(levelname)s]: %(message)s')
log = logging.getLogger(__name__)

mcp = FastMCP("Math")

@mcp.tool()
def add(a: int, b: int) -> int:
    """Add two numbers"""
    return a + b

@mcp.tool()
def multiply(a: int, b: int) -> int:
    """Multiply two numbers"""
    return a * b

if __name__ == "__main__":
    host = os.getenv("HOST", "0.0.0.0")  # Listen on all interfaces
    port = int(os.getenv("PORT", "8080"))

    log.info(f"Starting MCP server on http://{host}:{port}/mcp")

    # Create ASGI app for HTTP transport
    app = mcp.streamable_http_app()
    uvicorn.run(app, host=host, port=port)
```

### 2. Remote Client Configuration

**Client Setup**:

```python
import asyncio
from langchain_mcp_adapters.client import MCPClient
from langchain_openai import ChatOpenAI
from langgraph.graph import StateGraph, MessagesState

async def main():
    # Connect to remote MCP server
    client = MCPClient(
        transport="http",
        url="http://your-server-ip:8080/mcp"  # Remote server URL
    )

    try:
        # Test connection and get tools
        tools = await client.get_tools()
        print(f"Connected! Found {len(tools)} tools")

        # Use tools with LangGraph
        model = ChatOpenAI(model="gpt-4o-mini")
        model_with_tools = model.bind_tools(tools)

        # Build and run workflow
        graph = build_workflow(model_with_tools, tools)

        # Interactive chat loop
        while True:
            user_input = input("User: ")
            if user_input.lower() in ["quit", "exit", "q"]:
                break

            # Process with remote MCP tools
            response = await process_query(graph, user_input)
            print(f"Assistant: {response}")

    finally:
        await client.close()
```

## Step-by-Step VPS Deployment

### 1. **Setup VPS**

**Choose a provider**:

- **DigitalOcean**: $5/month droplet
- **Linode**: $5/month nanode
- **Vultr**: $2.50/month instance
- **AWS EC2**: t2.micro (free tier)

**Operating System**: Ubuntu 22.04 LTS (recommended)

### 2. **Initial Server Setup**

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Python and pip
sudo apt install python3 python3-pip -y

# Install git (if needed)
sudo apt install git -y

# Create project directory
mkdir /opt/mcp-server
cd /opt/mcp-server
```

### 3. **Install Dependencies**

```bash
# Install Python packages
pip3 install fastapi uvicorn mcp

# For FastMCP specifically
pip3 install fastmcp
```

### 4. **Deploy Server Code**

**Upload your server script**:

```bash
# Upload custom_server.py to /opt/mcp-server/
# You can use scp, git, or direct file transfer

# Make it executable
chmod +x custom_server.py
```

### 5. **Configure Firewall**

```bash
# Open port 8080 for MCP server
sudo ufw allow 8080

# Enable firewall
sudo ufw enable

# Check status
sudo ufw status
```

### 6. **Run the Server**

**Manual start**:

```bash
cd /opt/mcp-server
python3 custom_server.py
```

**Production setup with systemd**:

```bash
# Create systemd service file
sudo nano /etc/systemd/system/mcp-server.service
```

**Service configuration**:

```ini
[Unit]
Description=MCP Math Server
After=network.target

[Service]
Type=simple
User=ubuntu
WorkingDirectory=/opt/mcp-server
ExecStart=/usr/bin/python3 /opt/mcp-server/custom_server.py
Restart=always
RestartSec=10
Environment=HOST=0.0.0.0
Environment=PORT=8080

[Install]
WantedBy=multi-user.target
```

**Enable and start service**:

```bash
sudo systemctl daemon-reload
sudo systemctl enable mcp-server
sudo systemctl start mcp-server

# Check status
sudo systemctl status mcp-server
```

## Docker Deployment

### 1. **Create Dockerfile**

```dockerfile
FROM python:3.11-slim

# Set working directory
WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy server code
COPY custom_server.py .

# Expose port
EXPOSE 8080

# Set environment variables
ENV HOST=0.0.0.0
ENV PORT=8080

# Run server
CMD ["python", "custom_server.py"]
```

### 2. **Create requirements.txt**

```txt
fastmcp
uvicorn[standard]
```

### 3. **Build and Run**

```bash
# Build Docker image
docker build -t mcp-math-server .

# Run container
docker run -d -p 8080:8080 --name mcp-server mcp-math-server

# Check logs
docker logs mcp-server
```

### 4. **Deploy to Container Platform**

**Docker Hub deployment**:

```bash
# Tag and push to Docker Hub
docker tag mcp-math-server username/mcp-math-server
docker push username/mcp-math-server

# Deploy on any Docker-compatible platform
docker run -d -p 8080:8080 username/mcp-math-server
```

## Security Considerations

### 1. **Network Security**

**Configure HTTPS**:

```python
# Use reverse proxy (nginx) for HTTPS
# Or configure uvicorn with SSL certificates
uvicorn.run(
    app,
    host="0.0.0.0",
    port=8080,
    ssl_keyfile="path/to/key.pem",
    ssl_certfile="path/to/cert.pem"
)
```

**Firewall rules**:

```bash
# Only allow necessary ports
sudo ufw allow ssh
sudo ufw allow 8080
sudo ufw deny incoming
sudo ufw allow outgoing
```

### 2. **Authentication & Authorization**

**Add API key authentication**:

```python
from fastapi import HTTPException, Header

@mcp.middleware()
async def verify_api_key(request, call_next):
    api_key = request.headers.get("X-API-Key")
    if api_key != os.getenv("MCP_API_KEY"):
        raise HTTPException(status_code=401, detail="Invalid API key")
    return await call_next(request)
```

**Client with authentication**:

```python
client = MCPClient(
    transport="http",
    url="http://your-server:8080/mcp",
    headers={"X-API-Key": "your-api-key"}
)
```

### 3. **Rate Limiting**

```python
from slowapi import Limiter, _rate_limit_exceeded_handler
from slowapi.util import get_remote_address

limiter = Limiter(key_func=get_remote_address)

@mcp.tool()
@limiter.limit("10/minute")
def add(a: int, b: int) -> int:
    """Add two numbers with rate limiting"""
    return a + b
```

## Monitoring and Logging

### 1. **Application Logging**

```python
import logging
from datetime import datetime

# Configure detailed logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('/var/log/mcp-server.log'),
        logging.StreamHandler()
    ]
)

@mcp.tool()
def add(a: int, b: int) -> int:
    """Add two numbers"""
    logging.info(f"Addition requested: {a} + {b}")
    result = a + b
    logging.info(f"Addition result: {result}")
    return result
```

### 2. **Health Checks**

```python
@mcp.get("/health")
async def health_check():
    return {
        "status": "healthy",
        "timestamp": datetime.utcnow().isoformat(),
        "tools_count": len(mcp.tools)
    }
```

### 3. **Metrics Collection**

```python
import time
from collections import defaultdict

metrics = {
    "requests_total": defaultdict(int),
    "request_duration": defaultdict(list),
    "errors_total": defaultdict(int)
}

@mcp.middleware()
async def collect_metrics(request, call_next):
    start_time = time.time()
    endpoint = request.url.path

    try:
        response = await call_next(request)
        metrics["requests_total"][endpoint] += 1
        duration = time.time() - start_time
        metrics["request_duration"][endpoint].append(duration)
        return response
    except Exception as e:
        metrics["errors_total"][endpoint] += 1
        raise

@mcp.get("/metrics")
async def get_metrics():
    return metrics
```

## Best Practices

### 1. **Deployment**

- **Use process managers** (systemd, PM2, supervisor)
- **Configure proper logging** for production debugging
- **Set up automated backups** for stateful services
- **Use reverse proxies** (nginx, Caddy) for HTTPS

### 2. **Scalability**

- **Implement horizontal scaling** with load balancers
- **Use connection pooling** for database connections
- **Cache frequently accessed data**
- **Monitor resource usage** and auto-scale

### 3. **Reliability**

- **Implement health checks** for monitoring
- **Set up automated restarts** on failures
- **Use multiple availability zones**
- **Monitor and alert** on service issues

### 4. **Security**

- **Use HTTPS** for all communications
- **Implement authentication** and authorization
- **Regularly update** dependencies
- **Monitor for security** vulnerabilities

## Troubleshooting

### Common Issues:

1. **Connection Refused**:

   ```bash
   # Check if server is running
   sudo systemctl status mcp-server

   # Check port availability
   sudo netstat -tlnp | grep 8080

   # Check firewall rules
   sudo ufw status
   ```

2. **HTTP vs HTTPS Issues**:

   ```python
   # Ensure client uses correct protocol
   client = MCPClient(
       transport="http",  # or "https"
       url="http://your-server:8080/mcp"
   )
   ```

3. **Performance Issues**:

   ```bash
   # Monitor server resources
   htop

   # Check server logs
   sudo journalctl -f -u mcp-server

   # Monitor network traffic
   sudo iftop
   ```

## Exercises

### Beginner

1. Deploy a simple MCP server to a VPS
2. Configure firewall and basic security
3. Test remote connection from local client

### Intermediate

1. Set up HTTPS with SSL certificates
2. Implement API key authentication
3. Add monitoring and health checks

### Advanced

1. Deploy with Docker and container orchestration
2. Implement load balancing for multiple servers
3. Set up automated CI/CD deployment pipeline

## Next Steps

After mastering cloud deployment, explore:

1. **MCP with Agno Integration** (see `7 MCP with Agno/`)
2. **Multi-region MCP deployments**
3. **Microservices architecture with MCP**
4. **Enterprise MCP server management**

## Further Reading

- [DigitalOcean VPS Tutorial](https://www.digitalocean.com/docs/droplets/)
- [Docker Deployment Guide](https://docs.docker.com/get-started/)
- [FastAPI Production Deployment](https://fastapi.tiangolo.com/deployment/)
- [Nginx Reverse Proxy Setup](https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/)

## Monitoring Tools

### Recommended Tools:

- **Uptime Monitoring**: UptimeRobot, Pingdom
- **Log Management**: ELK Stack, Splunk
- **Performance Monitoring**: New Relic, Datadog
- **Infrastructure Monitoring**: Prometheus + Grafana

---

**Note**: Cloud deployment of MCP servers enables building truly scalable, production-ready AI agent ecosystems. This tutorial provides the foundation for deploying robust, secure, and scalable MCP infrastructure that can support enterprise-level AI applications.
