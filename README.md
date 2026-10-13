---
title: deep_research
app_file: app.py
sdk: gradio
sdk_version: 6.14.0
---

# Deep Research Agent

A cross-business research assistant that turns a question into a detailed web research report. Built with the OpenAI Agents SDK and Gradio, it plans searches, runs them concurrently, synthesizes the findings, and delivers the report through email or Pushover.

## How it works

1. **Plan:** The planner produces structured search queries and a reason for each query. It requests five searches by default.
2. **Search:** A search agent uses the web search tool for each query. Searches run concurrently with `asyncio.gather`, and each agent is instructed to return a short summary of fewer than 300 words.
3. **Write:** The writer combines the original question and search summaries into a Markdown report. Its prompt targets 5–10 pages and at least 1,000 words; actual output may vary.
4. **Deliver:** The email agent prepares a subject and formatted HTML email, then calls the delivery tool. With email disabled, the tool sends the subject and plain-text body through Pushover instead.
5. **Display:** The Gradio interface shows progress updates and, after delivery completes, the final Markdown report. Each run also includes a link to its OpenAI trace.

The writer returns a structured `ReportData` object containing `short_summary`, `markdown_report`, and `follow_up_questions`. The current interface displays only `markdown_report`.

## Repository contents

| File | Purpose |
| --- | --- |
| `app.py` | Main Gradio interface with an Investigate button, example questions, and streamed progress updates. |
| `research_manager.py` | Coordinates planning, concurrent searches, report generation, delivery, and tracing. |
| `planner_agent.py` | Defines the planner and the `WebSearchItem` / `WebSearchPlan` output schemas. |
| `search_agent.py` | Defines the web search agent with required tool use. |
| `writer_agent.py` | Defines the report-writing agent and the `ReportData` schema. |
| `email_agent.py` | Formats the report for delivery and exposes the email/Pushover function tool. |
| `messenger.py` | Sends SMTP email or posts messages to the Pushover API. |
| `styles.py` | Custom CSS, JavaScript, header markup, and example queries for the main interface. |
| `simple.py` | Alternative minimal Gradio interface using the same research pipeline. |
| `requirements.txt` | Python dependencies: Pydantic, python-dotenv, OpenAI, OpenAI Agents SDK, Gradio, and requests. |
| `.gitignore` | Excludes local environments, `.env` files, caches, and other generated files. |
| `LICENSE` | Apache License 2.0. |

## Local setup

Clone the repository and create a Python virtual environment:

```bash
git clone https://github.com/rudrakshsyal/deep_research_agent.git
cd deep_research_agent
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
```

Create a `.env` file in the repository root. For email delivery:

```dotenv
OPENAI_API_KEY=your_openai_api_key
DEFAULT_MODEL_NAME=gpt-5.4-mini
HOW_MANY_SEARCHES=5
USE_EMAIL=true
EMAIL_ADDRESS=you@example.com
EMAIL_SMTP_SERVER=smtp.example.com
EMAIL_APP_PASSWORD=your_email_app_password
```

Use an OpenAI account with access to the configured model and web search tool. Model and web search API usage may incur charges. Keep real credentials in the ignored `.env` file.

### Configuration

| Variable | Default | Purpose |
| --- | --- | --- |
| `OPENAI_API_KEY` | None | Authentication for OpenAI API calls. |
| `DEFAULT_MODEL_NAME` | `gpt-5.4-mini` | Model used by all four agents. |
| `HOW_MANY_SEARCHES` | `5` | Integer number of searches requested in the planner prompt. |
| `USE_EMAIL` | `true` | Selects email when its value is `true` (case-insensitive); other values select Pushover. |
| `EMAIL_ADDRESS` | None | Both sender and recipient for email delivery. |
| `EMAIL_SMTP_SERVER` | None | SMTP server; the implementation uses port 587 with STARTTLS. |
| `EMAIL_APP_PASSWORD` | None | Password used to authenticate to the SMTP server. |
| `PUSHOVER_USER` | None | Pushover user key, needed when `USE_EMAIL=false`. |
| `PUSHOVER_TOKEN` | None | Pushover application token, needed when `USE_EMAIL=false`. |

For Pushover delivery, replace the email settings with:

```dotenv
USE_EMAIL=false
PUSHOVER_USER=your_pushover_user_key
PUSHOVER_TOKEN=your_pushover_application_token
```

Delivery is part of every research run: setting `USE_EMAIL=false` switches to Pushover rather than disabling notifications. The current status messages still refer to email in either mode. Modules load `.env` with `override=True`, so values in that file override existing environment variables.

## Run

```bash
python app.py
```

Open the local URL printed by Gradio, enter a research question, and click **Investigate** or press **Enter**. Progress appears in the report area while the agents work, followed by the final report after the delivery step.

For the minimal interface, the repository also includes:

```bash
python simple.py
```

`requirements.txt` does not pin package versions. The README metadata specifies Gradio `6.14.0` for Spaces, while `simple.py` uses a different theme argument placement from the main app; compatibility of that alternative entry point depends on the installed Gradio version.

## Current behavior and limitations

- Reports are displayed in the browser and passed to the delivery agent; the application does not save report files or maintain a research history.
- The pipeline has no application-level retry or recovery logic around failed searches or delivery. Delivery is awaited before the final report is yielded to the interface.
- Search summaries feed the writer, but the code does not enforce a citation schema or validate source references in the finished report.
- The Pushover helper does not check the HTTP response for delivery errors, so a completion message does not verify successful receipt.

## License

Licensed under the [Apache License 2.0](LICENSE).
