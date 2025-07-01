# adk-cloudrun

## Setup and Configuration

1. Prerequisites

```
Python 3.11+
Google Cloud CLI
A Google Cloud Account
```

1. Create Your Project Structure
Open your terminal. Create the following folder structure and virtual environment.

```bash
# Create the main project folder
mkdir trend-spotter && cd trend-spotter
# Create the Python package folder that will hold our code
mkdir trend_spotter
touch trend_spotter/__init__.py
touch trend_spotter/agent.py
touch trend_spotter/prompt.py
# Create the top-level configuration files
touch pyproject.toml requirements.txt
# Finally, create and activate a virtual environment
python3 -m venv venv && source venv/bin/activate
# (On Windows, use python -m venv venv && .\venv\Scripts\activate)
```

1. Open requirements.txt and add our single dependency:

```bash
google-adk
```

1. Install it from your terminal:

```bash
pip install -r requirements.txt
```

1. Configure Your Cloud Environment: These settings tell ADK how to securely connect to your Google Cloud account to use services like Vertex AI and Google Search.

```bash
export GOOGLE_GENAI_USE_VERTEXAI=true
export GOOGLE_CLOUD_PROJECT=addo-argolis-demo
export GOOGLE_CLOUD_LOCATION=us-central1
```

1. Log In to Your Account: Run this one-time command. It will open a browser for you to sign in, allowing ADK to make authorized requests on your behalf.

```bash
gcloud auth application-default login
```

## Building Your Agent

Now we’ll write the code and place it inside our trend_spotter package directory.

1. Define the Agent’s Brain (The Prompt)
The prompt contains all the instructions for our agent.

Note that we are guiding the LLM to specify the date range in the call to the GoogleSearch tool to make sure we are focusing on trends from the last week.

Open trend_spotter/prompt.py and add these instructions:

```python
# trend_spotter/prompt.py


TREND_SPOTTER_PROMPT = """
You are a helpful AI assistant and expert tech analyst for a new podcast called "The Agent Factory". Your goal is to generate a highly relevant and verifiable report about the latest developments in AI agents that specifically impact developers.

**Your multi-step plan is as follows:**

**Step 1: Discover the Current Date.**
Your very first action must be to find the current date.
- **Action**: Use the `Google Search` tool with a query like "what is today's date".
- From the search result, identify the current year, month, and day.

**Step 2: Formulate and Execute Search Queries with Date Operators.**
Now, you must formulate your search queries by embedding the date range directly into the query string using Google's `after:YYYY-MM-DD` and `before:YYYY-MM-DD` operators. Calculate these dates to cover the last 7 days.
- You must perform at least three initial searches to cover trends, releases, and questions.
- **Example Query Format**: `"AI agent trends after:2025-06-01 before:2025-06-08"`
- After the initial searches, you may perform 1-2 additional, more targeted searches if a category is missing information. **Do not perform more than 5 searches in total.**

**Step 3: Analyze the Results and Create the Report.**
Read through all the text and links from your searches. Your primary filter is to **only select topics, tools, and questions that have a direct and significant impact on developers building AI agents.**

**Critical Rule for Sourcing:** For every trend, release, or question you identify, you must first pinpoint the **single best search result** that provides the evidence. You will then use the URL from that **exact search result** as the source link for that item. **If you cannot find a specific source link for an item, do not include that item in the report.**

Based on these rules, create a report:
1.  The report **must begin with a header** specifying the date range used.
2.  The body of the report must have exactly three sections.
3.  For each item, you **must provide three pieces of information**: a 1-2 sentence explanation, the "Developer Impact" analysis, and the **verifiable source URL**.

The report format must be:

**🔥 Top 5 Trends for Agent Developers**
1.  **[Trend 1 Name]**: [A 1-2 sentence explanation of this trend.] (Source: [URL])
    * **Developer Impact**: [A 1-sentence explanation of why this matters to developers.]
2.  ... (up to 5 total)

**🚀 Top 5 Releases for Agent Developers**
1.  **[Release 1 Name]**: [A 1-2 sentence explanation of the tool, framework, or model.] (Source: [URL])
    * **Developer Impact**: [A 1-sentence explanation of why this matters to developers.]
2.  ... (up to 5 total)

**🤔 Top 5 Questions from Agent Developers**
1.  **[Question 1 Topic]**: [A 1-2 sentence explanation of what developers are asking.] (Source: [URL])
    * **Developer Impact**: [A 1-sentence explanation of why this matters to developers.]
2.  ... (up to 5 total)

Begin your work now by executing your plan.
"""
```

1. Assemble the Agent

The agent.py file connects our prompt and the search tool to a new ADK Agent.

Open trend_spotter/agent.py and add this code:

```python
# trend_spotter/agent.py
from google.adk.agents import Agent
from google.adk.tools import google_search
from . import prompt
# Use the "latest" tag to always get the most recent stable version of the model.
MODEL = "gemini-2.5-pro-preview-05–06"
# This single agent will perform all the work.
trend_spotter_agent = Agent(
model=MODEL,
name="trend_spotter_agent",
description="An agent that finds and reports on AI agent trends.",
# The agent's entire logic comes from our detailed prompt.
instruction=prompt.TREND_SPOTTER_PROMPT,
# We give the agent a single tool: the ability to search Google.
tools=[google_search],
)
# We assign it to `root_agent` by convention for ADK to discover.
root_agent = trend_spotter_agent
```

1. Making Your Agent Discoverable
To use the adk web command, we need to tell ADK where to find our agent. We do this in the pyproject.toml file.

Open pyproject.toml in your root directory and add the following configuration:

```toml
[project]
name = "trend_spotter"
version = "0.1.0"
# This section tells the ADK how to find our agent.
[tool.adk.agents]
trend_spotter = "trend_spotter.agent:root_agent"
```

## Running Your Agent
Now for the exciting part!

1. Install your agent:
Run this command from your project’s root directory. The -e . command installs your project in “editable” mode so the adk tool can find it.

```bash
pip install -e .
```

1. Launch the web interface:

```bash
adk web
```

Open the URL that appears in your terminal. In the web interface, select “trend_spotter” from the dropdown menu. You can now chat with your agent! Ask it: “Generate a report on the latest AI agent news.” This might take a few minutes, depending on the amount of searches you instruct the agent to perform in your prompt.

1. Debugging
The adk web interface is your best debugging tool. On the “Events” tab, you can see every step your agent takes, including which tools it calls and what the LLM is thinking. If the output isn’t right, your first step should always be to adjust the instructions in prompt.py.

## Deployment
The adk deploy cloud_run command deploys your agent code to Google Cloud Run.

Ensure you have authenticated with Google Cloud (`gcloud auth login and gcloud config set project <your-project-id>`) and setup your environment variables to deploy your agent to cloud run with one line command.

1. Setup environment variables
Optional but recommended: Setting environment variables can make the deployment commands cleaner.

```bash
# Set your Google Cloud Project ID
export GOOGLE_CLOUD_PROJECT="addo-argolis-demo"
# Set your desired Google Cloud Location
export GOOGLE_CLOUD_LOCATION="us-central1" 
# Example location
# Set the path to your agent code directory
export AGENT_PATH="./trend_spotter" 
# Assuming capital_agent is in the current directory
# Set a name for your Cloud Run service (optional)
export SERVICE_NAME="trend-spotter-service"
# Set an application name (optional)
export APP_NAME="trend-spotter-app"
```

1. Deployment to Cloud Run

```bash
adk deploy cloud_run \
 --project=$GOOGLE_CLOUD_PROJECT \
 --region=$GOOGLE_CLOUD_LOCATION \
 --service_name=$SERVICE_NAME \
 --app_name=$APP_NAME \
 --with_ui \
$AGENT_PATH
```

1. Testing your deployed agent
You can test your agent by simply navigating to the Cloud Run service URL provided after deployment in your web browser. (The URL should be similar to this: https://your-service-name-abc123xyz.a.run.app)