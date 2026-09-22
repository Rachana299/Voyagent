# Voyagent

Voyagent is an agentic AI travel planner that turns a natural-language travel request into a practical, data-informed trip plan. It combines a LangGraph workflow with external travel tools for weather, places, activities, transportation, currency conversion, and expense calculations.

The project provides:

- A Streamlit chat interface for travelers.
- A FastAPI backend with a JSON `/query` endpoint.
- A tool-enabled LangGraph agent that can reason over a request and call specialized tools.
- Configurable OpenAI and Groq model providers.
- Google Places to Tavily fallback behavior for destination research.

## Architecture

```text
Streamlit UI (8501)
				|
				| POST /query
				v
FastAPI service (8000)
				|
				v
LangGraph agent
	 |       |        |          |
Weather  Places  Expenses  Currency
```

The backend builds a graph with an agent node and a tool node. The model decides when to call a tool, receives the tool result, and continues until it can return a final answer.

## Capabilities

The agent can use the following tools when appropriate:

- Current weather and forecast lookup.
- Attraction, restaurant, activity, and transportation search.
- Hotel cost estimation.
- Total and daily trip expense calculations.
- Currency conversion between supported currencies.

By default, the workflow uses the Groq provider configured in `config/config.yaml`. OpenAI is also supported by the model loader.

## Requirements

- Python 3.10 or newer.
- API credentials for the model provider and any external services you plan to use.
- `pip` or `uv` for dependency installation.

## Setup

Create and activate a virtual environment from the project directory.

### Windows PowerShell

```powershell
py -3.10 -m venv .venv
\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
```

### Linux, macOS, or WSL

```bash
python3.10 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
pip install -r requirements.txt
```

## Environment configuration

Create a local `.env` file in the project root. Never commit this file or place real values in documentation.

```dotenv
GROQ_API_KEY=your_groq_key
OPENAI_API_KEY=your_openai_key
OPENWEATHERMAP_API_KEY=your_openweathermap_key
GPLACES_API_KEY=your_google_places_key
TAVILY_API_KEY=your_tavily_key
EXCHANGE_RATE_API_KEY=your_exchange_rate_key
```

Only the credentials required by the selected model and requested tools need to be configured. The repository ignores `.env` and `.env.*` files by default.

## Run locally

Start the backend and frontend in separate terminals from the project directory.

### Terminal 1: FastAPI backend

```bash
uvicorn main:app --reload --port 8000
```

Backend URLs:

- API: `http://localhost:8000`
- Interactive API documentation: `http://localhost:8000/docs`

### Terminal 2: Streamlit frontend

```bash
streamlit run streamlit_app.py
```

Open the frontend at `http://localhost:8501`. The Streamlit app sends user questions to the backend at `http://localhost:8000/query`.

## API usage

The backend accepts a travel question as JSON:

```bash
curl -X POST http://localhost:8000/query \
	-H "Content-Type: application/json" \
	-d "{\"question\":\"Plan a five-day trip to Goa with a daily budget and weather-aware activities.\"}"
```

Successful responses have this shape:

```json
{
	"answer": "..."
}
```

## Project structure

```text
agent/             LangGraph workflow and agent orchestration
config/             Model provider configuration
exception/          Exception handling helpers
logger/             Logging helpers
prompt_library/    System prompt definitions
tools/              LangChain tool adapters
utils/              External service and calculation utilities
main.py             FastAPI application and /query endpoint
streamlit_app.py   Streamlit user interface
requirements.txt   Python dependencies
```

