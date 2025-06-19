# Multi-server MCP architecture

A demonstration project showing how to create and use multiple Model Context Protocol (MCP) servers with LangChain and Google's Gemini AI model. This project implements two MCP servers - a math server for arithmetic operations and a weather server for location-based weather queries.

## Overview

This project demonstrates:
- Creating MCP servers using FastMCP
- Connecting multiple MCP servers to a single client
- Using different transport protocols (stdio and HTTP)
- Integrating MCP tools with LangChain agents
- Leveraging Google's Gemini 2.0 Flash model for AI responses

## Architecture

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   client.py     │    │   mathserver.py  │    │ weatherserver.py│
│                 │    │                  │    │                 │
│ ┌─────────────┐ │    │ ┌──────────────┐ │    │ ┌─────────────┐ │
│ │ LangChain   │ │    │ │ FastMCP      │ │    │ │ FastMCP     │ │
│ │ Agent       │ │    │ │ Math Tools   │ │    │ │ Weather API │ │
│ │             │◄┼────┼►│ - add()      │ │    │ │ - get_weather│ │
│ │ Gemini 2.0  │ │    │ │ - multiply() │ │    │ │             │ │
│ │ Flash       │ │    │ └──────────────┘ │    │ └─────────────┘ │
│ └─────────────┘ │    │                  │    │                 │
└─────────────────┘    │ Transport: stdio │    │ Transport: HTTP │
                       └──────────────────┘    └─────────────────┘
```

## Prerequisites

- Python 3.8+
- Google API Key for Gemini
- Required Python packages (see Installation)

## Installation

1. **Clone the repository**:
```bash
git clone https://github.com/naakaarafr/Multi-Server-MCP-Architecture.git
cd Multi-Server-MCP-Architecture
```

2. **Install required packages**:
```bash
pip install langchain-mcp-adapters langgraph langchain-google-genai python-dotenv mcp
```

3. **Set up environment variables**:
Create a `.env` file in the project root:
```env
GOOGLE_API_KEY=your_google_api_key_here
```

To get a Google API key:
- Visit [Google AI Studio](https://makersuite.google.com/app/apikey)
- Create a new API key
- Copy the key to your `.env` file

## Project Structure

```
Multi-Server-MCP-Architecture/
│
├── client.py           # Main client that connects to MCP servers
├── mathserver.py       # MCP server for math operations
├── weatherserver.py    # MCP server for weather queries
├── .env               # Environment variables (Google API key)
├── LICENSE            # MIT License
└── README.md          # This file
```

## File Descriptions

### client.py
The main client application that:
- Configures and connects to multiple MCP servers
- Sets up a LangChain agent with Gemini 2.0 Flash
- Demonstrates tool usage with example queries
- Handles different transport protocols for each server

### mathserver.py
A simple MCP server providing mathematical operations:
- `add(a, b)`: Adds two integers
- `multiply(a, b)`: Multiplies two integers  
- Uses stdio transport for communication

### weatherserver.py
A mock weather MCP server:
- `get_weather(location)`: Returns weather information for a location
- Uses HTTP transport for communication
- Returns a mock response for demonstration

## Usage

### Starting the Servers

The math server (stdio transport) is started automatically by the client. For the weather server, you have two options:

**Option 1: Let the client handle it** (if using compatible MCP client):
The client will attempt to start the weather server automatically.

**Option 2: Start manually** (recommended):
```bash
# In a separate terminal
python weatherserver.py
```

### Running the Client

```bash
python client.py
```

The client will:
1. Connect to both MCP servers
2. Execute a math query: "what's (3 + 5) x 12?"
3. Execute a weather query: "what is the weather in California?"
4. Display the results

### Expected Output

```
Math response: To calculate (3 + 5) x 12, I need to first add 3 and 5, then multiply by 12.

First: 3 + 5 = 8
Then: 8 × 12 = 96

Therefore, (3 + 5) x 12 = 96

Weather response: According to the weather service, it's always raining in California.
```

## Transport Protocols

This project demonstrates two MCP transport protocols:

### STDIO Transport (mathserver.py)
- Communication via standard input/output
- Suitable for local, process-based servers
- Automatically managed by the client

### HTTP Transport (weatherserver.py)
- Communication via HTTP requests
- Suitable for network-based servers
- Requires manual server startup or service management

## Customization

### Adding New Math Operations

To add new math operations to the math server:

```python
@mcp.tool()
def subtract(a: int, b: int) -> int:
    """Subtract b from a"""
    return a - b

@mcp.tool()
def divide(a: float, b: float) -> float:
    """Divide a by b"""
    if b == 0:
        raise ValueError("Cannot divide by zero")
    return a / b
```

### Creating Real Weather Integration

Replace the mock weather function with a real API:

```python
import requests

@mcp.tool()
async def get_weather(location: str) -> str:
    """Get real weather data for a location"""
    api_key = "your_weather_api_key"
    url = f"http://api.openweathermap.org/data/2.5/weather?q={location}&appid={api_key}"
    
    response = requests.get(url)
    data = response.json()
    
    if response.status_code == 200:
        temp = data['main']['temp'] - 273.15  # Convert from Kelvin to Celsius
        description = data['weather'][0]['description']
        return f"Weather in {location}: {temp:.1f}°C, {description}"
    else:
        return f"Could not get weather for {location}"
```

### Adding More Servers

To add additional MCP servers, update the client configuration:

```python
client = MultiServerMCPClient({
    "math": {
        "command": "python",
        "args": ["mathserver.py"],
        "transport": "stdio",
    },
    "weather": {
        "url": "http://localhost:8000/mcp",
        "transport": "streamable_http",
    },
    "newserver": {
        "command": "python",
        "args": ["newserver.py"],
        "transport": "stdio",
    }
})
```

## Troubleshooting

### Common Issues

1. **Google API Key Error**:
   - Ensure your `.env` file contains a valid `GOOGLE_API_KEY`
   - Check that the API key has the necessary permissions

2. **Weather Server Connection Failed**:
   - Make sure the weather server is running on `http://localhost:8000/mcp`
   - Check for port conflicts

3. **Math Server Not Found**:
   - Ensure `mathserver.py` is in the same directory as `client.py`
   - Check file permissions

4. **Module Import Errors**:
   - Verify all required packages are installed
   - Check Python version compatibility

### Debug Mode

To enable more detailed logging, add this to your client:

```python
import logging
logging.basicConfig(level=logging.DEBUG)
```

## Contributing

To contribute to this project:

1. Fork the repository
2. Create a feature branch
3. Add your improvements
4. Test thoroughly
5. Submit a pull request

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Repository

GitHub: [https://github.com/naakaarafr/Multi-Server-MCP-Architecture](https://github.com/naakaarafr/Multi-Server-MCP-Architecture)

## Resources

- [MCP Documentation](https://modelcontextprotocol.io/)
- [LangChain Documentation](https://python.langchain.com/)
- [Google AI Studio](https://makersuite.google.com/)
- [FastMCP Documentation](https://github.com/jlowin/fastmcp)

## Next Steps

Consider extending this project by:
- Adding authentication to servers
- Implementing error handling and retry logic
- Creating a web interface for the client
- Adding more sophisticated tools and servers
- Implementing server health checks and monitoring
