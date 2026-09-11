# ResearcherAgent

> A Streamlit research workspace where specialized LangChain agents search, read, write, and critique a report on any topic.

<p align="center">
	<a href="https://python.org"><img src="https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white" alt="Python 3.10+"></a>
	<a href="https://streamlit.io"><img src="https://img.shields.io/badge/UI-Streamlit-FF4B4B?logo=streamlit&logoColor=white" alt="Streamlit"></a>
	<a href="https://www.langchain.com/"><img src="https://img.shields.io/badge/Orchestration-LangChain-1C3C3C" alt="LangChain"></a>
	<a href="https://console.groq.com/"><img src="https://img.shields.io/badge/LLM-Groq-F55036" alt="Groq"></a>
	<a href="https://tavily.com/"><img src="https://img.shields.io/badge/Search-Tavily-111827" alt="Tavily"></a>
</p>

<p align="center">
	<a href="#quickstart">Quickstart</a> |
	<a href="#how-it-works">How it works</a> |
	<a href="#configuration">Configuration</a> |
	<a href="#project-layout">Project layout</a> |
	<a href="#troubleshooting">Troubleshooting</a>
</p>

## Why this project

ResearcherAgent turns a broad question into a reviewable report through four focused stages:

- **Search Agent** finds recent web sources with Tavily.
- **Reader Agent** selects a useful result and extracts readable page content.
- **Writer Chain** turns the collected evidence into a structured report.
- **Critic Chain** scores the report and gives specific improvement suggestions.

The browser UI shows pipeline progress, keeps raw search and reader output available in expanders, and lets you download the final report as Markdown.

## Quickstart

### 1. Create an environment

```bash
git clone <your-repository-url>
cd Langchain-MultiAgent-Research-System

python -m venv .venv
```

Activate it:

```powershell
# Windows PowerShell
.\.venv\Scripts\Activate.ps1
```

```bash
# macOS/Linux
source .venv/bin/activate
```

### 2. Install dependencies

```bash
pip install -r requirements.txt
```

### 3. Add API keys

Create a `.env` file in the project root:

```env
GROQ_API_KEY=your_groq_api_key
TAVILY_API_KEY=your_tavily_api_key
```

Get credentials from [Groq Console](https://console.groq.com/keys) and [Tavily](https://app.tavily.com/home).

### 4. Launch the app

```bash
streamlit run app.py
```

Open the local URL printed by Streamlit, enter a research topic, and select **Run Research Pipeline**.

<details>
<summary><strong>Example prompts</strong></summary>

Try one of these topics:

```text
Future of LLMs in the technology industry
The latest AI agent architectures
Roadmap for AGI development over the next five years
```

</details>

## How it works

```mermaid
flowchart LR
		U[Research topic] --> S[Search Agent]
		S -->|Tavily results| R[Reader Agent]
		R -->|Extracted page text| W[Writer Chain]
		W -->|Draft report| C[Critic Chain]
		C --> O[Report + feedback]
		O --> D[Download Markdown]

		classDef input fill:#0f766e,color:#fff,stroke:#5eead4;
		classDef agent fill:#172554,color:#fff,stroke:#60a5fa;
		classDef output fill:#854d0e,color:#fff,stroke:#facc15;
		class U input;
		class S,R,W,C agent;
		class O,D output;
```

### The runtime sequence

| Stage | Component | Output |
| --- | --- | --- |
| 01 | `web_search` | Up to five recent search results with titles, URLs, and snippets |
| 02 | `scrape_url` | Clean article text using Trafilatura, Readability, and Beautiful Soup fallbacks |
| 03 | `writer_chain` | Introduction, key findings, conclusion, and source list |
| 04 | `critic_chain` | Score, strengths, improvement areas, and one-line verdict |

<details>
<summary><strong>What happens after you click Run?</strong></summary>

1. Streamlit resets the previous session results and marks the pipeline as running.
2. The search agent calls Tavily and returns source candidates.
3. The reader agent chooses a relevant URL and calls the scraping tool.
4. The writer chain receives both search snippets and extracted content.
5. The critic chain reviews the generated report.
6. Results are stored in Streamlit session state and rendered in the UI.

</details>

## Configuration

| Variable | Required | Used by |
| --- | --- | --- |
| `GROQ_API_KEY` | Yes | `ChatGroq` model calls |
| `TAVILY_API_KEY` | Yes | `web_search` |

The default model is `openai/gpt-oss-20b` with temperature `0`, configured in `src/agents/agents.py`. Change it there if your Groq account exposes a different model.

<details>
<summary><strong>Security checklist</strong></summary>

- Keep `.env` local and never commit API keys.
- Add `.env` to `.gitignore` before sharing the repository.
- Treat scraped web content as untrusted input.
- Review generated sources and claims before using a report as factual research.

</details>

## Project layout

```text
.
├── app.py                    # Streamlit interface and interactive run state
├── main.py                   # Small scraping smoke-test script
├── requirements.txt          # Python dependencies
└── src/
		├── agents/agents.py      # Search/reader agents and writer/critic chains
		├── pipelines/pipelines.py# Reusable terminal-style pipeline function
		└── tools/tools.py        # Tavily search and URL extraction tools
```

## Development commands

```bash
# Start the interactive UI
streamlit run app.py

# Check Python syntax without running API calls
python -m compileall app.py main.py src

# Exercise the standalone scraper smoke test
python main.py
```

## Troubleshooting

<details>
<summary><strong>Missing API key errors</strong></summary>

Confirm `.env` is in the same directory from which you launch Streamlit and that both variables are named exactly `GROQ_API_KEY` and `TAVILY_API_KEY`.

</details>

<details>
<summary><strong>Scraping returns little or no text</strong></summary>

Some sites block automated requests, require JavaScript, or expose very little server-rendered content. Try another search topic or source; the scraper already falls back from Trafilatura to Readability and then full-page text extraction.

</details>

<details>
<summary><strong>The app starts but a model call fails</strong></summary>

Check the selected Groq model in `src/agents/agents.py`, confirm your account has access to it, and verify that your installed packages match `requirements.txt`.

</details>

## Limitations

- Search and scraping depend on external services and website availability.
- The reader currently scrapes one URL selected by the reader agent.
- Generated reports need human verification, especially for fast-changing topics.
- There are currently no automated tests in the repository.

## License

See [LICENSE](LICENSE).

