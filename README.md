
Here’s README content you can drop into a `README.md` file. (I’m in Ask mode, so I’m not creating or editing files. If you want the file created for you, switch to Agent mode.)

---

## Complete README content

```markdown
# OwnMCP — Weather MCP Server & Chat Client

A Python project that runs a **Model Context Protocol (MCP)** server for US weather alerts and an interactive chat client that uses that server via an LLM (Groq + LangChain) with conversation memory.

---

## Table of Contents

- [Overview](#overview)
- [Project Structure](#project-structure)
- [Prerequisites](#prerequisites)
- [Setup (Start to Finish)](#setup-start-to-finish)
- [Running the MCP Server](#running-the-mcp-server)
- [Running the Chat Client](#running-the-chat-client)
- [MCP Server Details](#mcp-server-details)
- [Weather Information](#weather-information)
- [Configuration](#configuration)
- [Troubleshooting](#troubleshooting)

---

## Overview

- **MCP server** (`server/weather.py`): Exposes tools and resources via the [Model Context Protocol](https://modelcontextprotocol.io/). It uses **FastMCP** and the **National Weather Service (NWS) API** to fetch active weather alerts for US states.
- **Chat client** (`server/client.py`): Connects to the weather MCP server using a config file, uses **LangChain + Groq** (e.g. `qwen/qwen3-32b`) as the LLM, and runs an interactive chat with **conversation memory**. The agent can call MCP tools (e.g. get weather alerts) when answering.

---

## Project Structure

```
ownmcp/
├── pyproject.toml          # Project deps: langchain-groq, mcp-use, mcp[cli], python-dotenv
├── uv.lock                 # Lock file (uv)
├── README.md               # This file
└── server/
    ├── weather.py          # MCP server (FastMCP): tools, resources, prompts
    ├── weather.json        # MCP client config (how to run the weather server)
    ├── client.py           # Interactive chat client (MCPAgent + Groq)
    └── .env                # GROQ_API_KEY (do not commit)
```

---

## Prerequisites

- **Python 3.11+**
- **uv** (recommended) or pip for installing dependencies
- **Groq API key** (for the chat client LLM)

---

## Setup (Start to Finish)

### 1. Clone or open the project

```bash
cd d:\chumma\demo\MCP\ownmcp
```

### 2. Create a virtual environment and install dependencies

With **uv** (from project root):

```bash
uv sync
```

Or with **pip**:

```bash
python -m venv .venv
.venv\Scripts\activate   # Windows
# source .venv/bin/activate  # Linux/macOS
pip install -e .
```

## Running the MCP Server

The client starts the server automatically via the config. To run the server **standalone** (e.g. to test or inspect tools):

```bash
# From project root, with uv
uv run --with mcp[cli] mcp run server/weather.py
```

Or from `server/`:

```bash
cd server
uv run --with mcp[cli] mcp run weather.py
```

The server exposes:

- **Tools:** e.g. `get_alerts(state)` (see [MCP Server Details](#mcp-server-details)).
- **Resources:** e.g. `echo://{message}`.
- **Prompts:** e.g. `greet_user(name, style)`.

---

## Running the Chat Client

From the **project root** (`ownmcp/`):

```bash
uv run python server/client.py
```

Or with the venv activated:

```bash
.venv\Scripts\activate
python server/client.py
```

The client:

1. Loads `server/weather.json` and starts the weather MCP server as a subprocess.
2. Connects to it with `MCPClient.from_config_file()`.
3. Runs an interactive loop with `MCPAgent` (Groq LLM, memory enabled).

**Commands in chat:**

- `exit` or `quit` — end the session.
- `clear` — clear conversation history.

You can ask things like: *“What are the weather alerts for California?”* and the agent will use the `get_alerts` tool when needed.

---

## MCP Server Details

- **Framework:** [FastMCP](https://github.com/jlowin/fastmcp) (`mcp.server.fastmcp`).
- **Name:** `"weather"`.

### Tools

| Tool            | Description                          | Parameters        |
|-----------------|--------------------------------------|--------------------|
| `get_alerts`    | Get active weather alerts for a US state | `state`: 2-letter code (e.g. `CA`, `NY`) |

- Implemented in `server/weather.py`.
- Calls the NWS API: `https://api.weather.gov/alerts/active/area/{state}`.
- Returns formatted alert text (event, area, severity, description, instructions) or a message if none found or on error.

### Resources

- **URI pattern:** `echo://{message}`
- **Handler:** `echo_resource(message)` — returns `Resource echo: {message}`.

### Prompts

- **Name:** `greet_user`
- **Parameters:** `name`, `style` (optional: `"friendly"` \| `"formal"` \| `"casual"`).
- **Purpose:** Returns a prompt string asking the LLM to generate a greeting for `name` in the given style.

---

## Weather Information

- **Source:** [National Weather Service (NWS) API](https://www.weather.gov/documentation/services-web-api).
- **Base URL:** `https://api.weather.gov`.
- **Used endpoint:** `GET /alerts/active/area/{state}` — active alerts for a state.
- **Data:** GeoJSON; the server formats each alert’s properties (event, areaDesc, severity, description, instruction) into readable text.
- **Scope:** US states only (two-letter state code).
- **No API key** required for NWS.

---

## Configuration

### `server/weather.json`

- **command:** `uv` (or `python` if you run the server with pip).
- **args:** Run the MCP CLI and pass `weather.py`. Update the path if your project lives elsewhere.

### `server/client.py`

- **Config path:** `server/weather.json` (relative to where you run the script; the code uses `"server/weather.json"` from project root).
- **LLM:** `ChatGroq(model="qwen/qwen3-32b")`.
- **Agent:** `MCPAgent` with `memory_enabled=True`, `max_steps=15`.

---

## Troubleshooting

1. **“Module not found” (e.g. `mcp_use`, `langchain_groq`)**  
   Ensure dependencies are installed: `uv sync` or `pip install -e .` from project root.

2. **Client can’t connect to server**  
   - Check that the path in `weather.json` points to your `weather.py`.  
   - On Windows, use double backslashes or forward slashes in the path.

3. **GROQ_API_KEY errors**  
   - Ensure `server/.env` exists and contains `GROQ_API_KEY=...`.  
   - The client loads `.env` with `load_dotenv()` (no path), so run `client.py` from a directory where `.env` is found, or set the path explicitly in code.

4. **No alerts or NWS errors**  
   - NWS can be rate-limited or temporarily down.  
   - Use a valid two-letter state code (e.g. `CA`, `TX`, `NY`).

5. **Running from a different directory**  
   - Run `python server/client.py` from `ownmcp/` so `server/weather.json` and `server/.env` resolve correctly, or adjust paths in code and config.

---



---

You can paste this into `ownmcp/README.md` (and optionally fix the path in `weather.json` for your machine). If you want the file created or paths adjusted automatically, switch to **Agent mode** and ask to create/update the README.
