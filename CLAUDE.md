# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Ice Breaker is a Flask-based web application that generates personalized conversation starters by analyzing LinkedIn and Twitter profiles using LangChain and OpenAI.

## Development Commands

### Running the Application
```bash
# Install dependencies
pipenv install

# Run the Flask application
pipenv run python app.py

# Or directly with pipenv shell
pipenv shell
python app.py
```

### Code Quality Tools
```bash
# Format code with Black
pipenv run black .

# Sort imports with isort
pipenv run isort .

# Lint code with pylint
pipenv run pylint [module_name]
```

### Testing
```bash
# Note: No test files currently exist, but the README mentions pytest
pipenv run pytest .
```

## Architecture & Key Components

### Core Pipeline Flow
1. **Web Interface** (`app.py`): Flask server. Exposes `GET /` (renders `index.html`) and a single `POST /process` endpoint that takes a form field `name` and returns JSON with `summary_and_facts`, `interests`, `ice_breakers`, and `picture_url`. The entire pipeline is driven by a person's name.
2. **Main Logic** (`ice_breaker.py`): `ice_break_with(name)` is the single orchestration function that runs the whole pipeline. There is no CLI entry point — `if __name__ == "__main__"` is just `pass`, so the only real entry is through Flask.
3. **Agent System**: Profile lookup agents for LinkedIn and Twitter discovery
4. **Data Extraction**: Third-party integrations for scraping social media data
5. **AI Processing**: LangChain chains for generating summaries, interests, and ice breakers
6. **Output Parsing**: Structured response formatting using Pydantic models

### Key Modules

- **agents/**: LangChain ReAct agents for profile discovery
  - Uses Tavily for web search to find LinkedIn/Twitter profiles
  - Leverages LangChain hub prompts (hwchase17/react)

- **chains/**: Custom LangChain sequences for AI processing
  - Three distinct chains: summary, interests, ice breakers
  - Uses GPT-3.5-turbo with different temperature settings

- **third_parties/**: External API integrations
  - LinkedIn scraping via Scrapin.io API
  - Twitter data via Twitter API. NOTE: `ice_breaker.py` calls `scrape_user_tweets_mock` by default — the real `scrape_user_tweets` is imported but unused. To use live Twitter data, swap that call in `ice_break_with`.

- **tools/**: Utility functions for web search using Tavily

- **output_parsers.py**: Pydantic models for structured outputs (Summary, TopicOfInterest, IceBreaker)

## API Dependencies

Required environment variables in `.env`:
- `OPENAI_API_KEY` - OpenAI API for LLM
- `SCRAPIN_API_KEY` - Scrapin.io for LinkedIn data
- `TAVILY_API_KEY` - Tavily for web search
- `TWITTER_API_KEY`, `TWITTER_API_SECRET`, `TWITTER_ACCESS_TOKEN`, `TWITTER_ACCESS_SECRET` - Optional Twitter API

Optional LangSmith tracing:
- `LANGCHAIN_TRACING_V2=true`
- `LANGCHAIN_API_KEY`
- `LANGCHAIN_PROJECT=ice_breaker`

## Development Notes

- The application uses GPT-4o-mini for agents and GPT-3.5-turbo for chains
- Twitter integration uses the mock implementation (`scrape_user_tweets_mock`) as the active default path in `ice_breaker.py`, not just for testing — live Twitter scraping requires editing the call
- Flask runs in debug mode by default on host 0.0.0.0
- LangChain agents use verbose mode for debugging
- No unit tests are currently implemented despite pytest being mentioned