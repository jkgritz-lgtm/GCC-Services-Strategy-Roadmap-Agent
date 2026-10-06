# GCC-Services-Strategy-Roadmap-Agent
1. Project Configuration Files
Place these in the root of your gcc-delivery-strategy folder.

pyproject.toml

toml
[project]
name = "gcc-delivery-strategy"
version = "0.1.0"
description = "An AI agent deployed on Gemini Enterprise designed to generate a comprehensive Service Delivery Strategy roadmap based on user-uploaded documents and interactive chat inputs, outputting the final deliverable as a Google Slides presentation."
authors = [
    {name = "Your Name", email = "your@email.com"},
]

dependencies = [
    "google-adk[gcp,otel-gcp]>=2.6.0,<3.0.0",
    "opentelemetry-resourcedetector-gcp<=1.12.0a0",
    "gcsfs>=2024.11.0",
    "a2a-sdk[http-server]>=1.0,<2",
    "aiohttp>=3.13.4",
    "google-cloud-logging>=3.12.0,<4.0.0",
    "google-cloud-aiplatform[evaluation,agent-engines]>=1.156.0",
    "protobuf>=6.31.1,<7.0.0",
]
requires-python = ">=3.11,<3.14"


[dependency-groups]
dev = [
    "pytest>=9.0.2,<10.0.0",
    "pytest-asyncio>=1.0.0,<2.0.0",
    "nest-asyncio>=1.6.0,<2.0.0",
]

[project.optional-dependencies]
eval = [
    "google-adk[eval]>=2.6.0,<3.0.0",
    "google-cloud-aiplatform[evaluation]>=1.156.0",
]
lint = [
    "ruff>=0.4.6,<1.0.0",
    "ty>=0.0.1a0",
    "codespell>=2.2.0,<3.0.0",
]

[tool.ruff]
line-length = 88
target-version = "py311"

[tool.ruff.lint]
select = [
    "E",   # pycodestyle
    "F",   # pyflakes
    "W",   # pycodestyle warnings
    "I",   # isort
    "C",  # flake8-comprehensions
    "B",   # flake8-bugbear
    "UP", # pyupgrade
    "RUF", # ruff specific rules
]
ignore = ["E501", "C901", "B006"] # ignore line too long, too complex

[tool.ruff.lint.isort]
known-first-party = ["app", "frontend"]

[tool.ty]
[tool.ty.environment]
python-version = "3.10"

[tool.ty.src]
exclude = [".venv/**"]

[tool.ty.rules]
unresolved-import = "ignore"
unresolved-attribute = "ignore"
invalid-argument-type = "ignore"
invalid-assignment = "ignore"
invalid-return-type = "ignore"
possibly-missing-attribute = "ignore"
not-subscriptable = "ignore"
deprecated = "ignore"

[tool.codespell]
ignore-words-list = "rouge"
skip = "./locust_env/*,uv.lock,.venv,./frontend,**/package-lock.json"


[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"


[tool.pytest.ini_options]
pythonpath = "."
asyncio_default_fixture_loop_scope = "session"

[tool.hatch.build.targets.wheel]
packages = ["app","frontend"]
.env

GOOGLE_CLOUD_PROJECT="agent-catalog-demo-1"
GOOGLE_CLOUD_LOCATION="global"
GOOGLE_GENAI_USE_VERTEXAI="True"
2. Application Core & Orchestrator
Create a folder named app. Inside the app folder, create these files:

app/agent.py

import os
import google.auth
from google.adk.apps import App
from google.adk.agents import SequentialAgent, LlmAgent
from app.sub_agents.document_analyzer import document_analyzer
from app.sub_agents.strategy_synthesizer import strategy_synthesizer
from app.sub_agents.slides_generator import slides_generator

try:
    _, project_id = google.auth.default()
except Exception:
    project_id = "agent-catalog-demo-1" # Fallback if local auth is missing

os.environ["GOOGLE_CLOUD_PROJECT"] = project_id
os.environ["GOOGLE_CLOUD_LOCATION"] = "global"
os.environ["GOOGLE_GENAI_USE_VERTEXAI"] = "True"

# Main Sequential Orchestrator
pipeline_agent = SequentialAgent(
    name="GCC_Service_Delivery_Strategy_Pipeline",
    description="A sequential pipeline to analyze documents, synthesize strategy with clarifications, and generate a Google Slides presentation.",
    sub_agents=[
        document_analyzer,
        strategy_synthesizer,
        slides_generator
    ]
)

# Root agent that interacts with the user and delegates to the pipeline.
root_agent = LlmAgent(
    name="GCC_Service_Delivery_Strategy_Agent",
    model="gemini-3.1-pro-preview",
    instruction="""You are the main orchestrator for the GCC Service Delivery Strategy.
Your job is to assist the user by taking their inputs (documents and chat) and passing them into the `GCC_Service_Delivery_Strategy_Pipeline`.
Delegate the task to `GCC_Service_Delivery_Strategy_Pipeline` to perform document analysis, strategy synthesis, and Google Slides generation.
Once the pipeline is complete, summarize the results and provide the presentation link.""",
    description="Main orchestrator agent controlling the GCC Service Delivery Strategy pipeline. Manages multi-agent execution across analysis, clarification/assumptions, synthesis, and Google Slides output creation.",
    sub_agents=[pipeline_agent]
)

app = App(
    root_agent=root_agent,
    name="app",
)
(Note: Also create an empty file named __init__.py inside the app folder.)

3. Custom Tools
Create a folder named app_utils inside your app folder. Inside it, create an empty __init__.py and the following file:

app/app_utils/tools.py

python
from google.adk.tools import ToolContext
import json

def document_reader(document_path: str, tool_context: ToolContext) -> dict:
    """Utility tool for reading and parsing uploaded document files (PDFs, Docs, Spreadsheets).
    
    Use this tool to extract text from a provided document path.
    
    Args:
        document_path (str): The path or name of the document file to read.
        
    Returns:
        dict: The extracted text from the document, or an error message.
    """
    try:
        # In a real environment, this would parse PDFs, Docs, etc.
        # For ADK simulation, we simulate extracting content or reading an artifact.
        return {
            "status": "success",
            "content": f"Simulated parsed content from {document_path}. Contains customer service delivery requirements, scope, and objectives."
        }
    except Exception as e:
        return {"status": "error", "message": str(e)}

def google_slides_generator(presentation_title: str, slide_content_json: str, tool_context: ToolContext) -> dict:
    """Programmatically creates a formatted Google Slides presentation from structured JSON content via Google Workspace Slides API.
    
    Use this tool to generate the final Google Slides deck once all strategy sections are finalized.
    
    Args:
        presentation_title (str): The title for the new Google Slides presentation.
        slide_content_json (str): A JSON string containing the structured slide data 
                                  (Vision, Strategy, Commercials, Approach, Roles, Operating Model).
        
    Returns:
        dict: Status and the URL to the generated Google Slides presentation.
    """
    try:
        # Validate JSON
        json.loads(slide_content_json)
        # Mock Google Workspace Slides API interaction
        presentation_url = f"https://docs.google.com/presentation/d/mock_presentation_id/edit?title={presentation_title.replace(' ', '_')}"
        return {
            "status": "success",
            "message": "Google Slides presentation generated successfully.",
            "url": presentation_url
        }
    except json.JSONDecodeError:
        return {"status": "error", "message": "Invalid JSON format for slide_content_json."}
    except Exception as e:
        return {"status": "error", "message": str(e)}
4. The Sub-Agents
Create a folder named sub_agents inside your app folder. Inside it, create an empty __init__.py and the following three files:

app/sub_agents/document_analyzer.py

from google.adk.agents import LlmAgent
from app.app_utils.tools import document_reader

document_analyzer = LlmAgent(
    name="DocumentAnalyzerAgent",
    model="gemini-3.1-pro-preview",
    instruction="""You are the Document Analyzer Agent. Your goal is to analyze user-uploaded context documents and chat inputs.
Extract content matching the following required roadmap sections: Vision, Services Delivery Strategy, Commercials, Delivery Approach, Roles & Responsibilities, and Operating Model.
Identify any missing details or information gaps.
Output your findings, explicitly listing extracted content and any gaps.""",
    description="Analyze user-uploaded context documents and chat inputs to extract section content and identify missing details.",
    tools=[document_reader],
    output_key="extracted_document_context"
)
app/sub_agents/strategy_synthesizer.py

python
from google.adk.agents import LlmAgent

strategy_synthesizer = LlmAgent(
    name="StrategySynthesizerAgent",
    model="gemini-3.1-pro-preview",
    instruction="""You are the Strategy Synthesizer Agent. Your goal is to formulate core strategic sections: Vision, Services Delivery Strategy, Commercials, Delivery Approach, Roles & Responsibilities, and Operating Model.
Use the context extracted by the DocumentAnalyzerAgent: {extracted_document_context}.
If any critical information is missing, proactively prompt the user with clarifying questions and propose industry-standard assumptions for their approval.
Once all information is gathered and approved, synthesize the finalized content for all 6 sections.""",
    description="Formulate strategic sections, prompt user with clarifying questions, and propose industry-standard assumptions.",
    tools=[],
    output_key="finalized_strategy_content"
)
app/sub_agents/slides_generator.py

python
from google.adk.agents import LlmAgent
from app.app_utils.tools import google_slides_generator

slides_generator = LlmAgent(
    name="SlidesGeneratorAgent",
    model="gemini-3.1-pro-preview",
    instruction="""You are the Slides Generator Agent. Your goal is to take the finalized strategic content from the Strategy Synthesizer Agent.
Finalized Content: {finalized_strategy_content}

Structure this finalized strategic content into standard slide layouts for all required sections (Vision, Services Delivery Strategy, Commercials, Delivery Approach, Roles & Responsibilities, Operating Model) formatted as a JSON string.
Trigger the google_slides_generator tool with this structured JSON data to programmatically create the presentation deck. Provide the user with the link to the generated Google Slides.""",
    description="Structure finalized strategic content into slide layouts and trigger the Google Slides API integration tool.",
    tools=[google_slides_generator],
    output_key="presentation_generation_result"
)
