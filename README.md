# GCC-Services-Strategy-Roadmap-Agent
An agent that builds the services strategy roadmap
services-roadmap-agent/
├── README.md
├── requirements.txt
├── config.yaml
└── agent.py
google-genai>=0.1.0
pyyaml>=6.0.1
pydantic>=2.0.0
model_name: "gemini-2.5-pro"
temperature: 0.2
system_instruction: |
  You are an expert Google Cloud Consulting (GCC) Pursuit Lead and Services Architect. 
  Your job is to generate a comprehensive, executive-ready Services Strategy Roadmap based on:
  1. Business Outcomes (What the client wants to achieve)
  2. Workstreams (The technical/consulting activities to promote these outcomes)
  3. KPIs/Measurements (How success will be demonstrated)
  
  Format the output beautifully using structured markdown tables, bold key takeaways, and clear timelines.
import os
import yaml
from typing import List, Dict, Any
from pydantic import BaseModel, Field
from google import genai
from google.genai import types

# Define strict schemas for input validation
class Workstream(BaseModel):
    name: str = Field(description="Name of the consulting or engineering workstream (e.g., Data Foundation, Enablement)")
    description: str = Field(description="Detailed activities and objectives of this workstream")

class OutcomeKPI(BaseModel):
    outcome: str = Field(description="Desired customer business outcome")
    kpi: str = Field(description="Quantifiable Key Performance Indicator or measurement of success")

class StrategyInput(BaseModel):
    customer_name: str = Field(description="The name of the target customer/opportunity")
    target_timeline: str = Field(description="The implementation timeline (e.g., Q4 2026, 6 Months)")
    outcomes_and_kpis: List[OutcomeKPI] = Field(description="List of desired outcomes mapped to their KPIs")
    workstreams: List[Workstream] = Field(description="The program workstreams that drive these outcomes")

class ServicesStrategyAgent:
    def __init__(self, config_path: str = "config.yaml"):
        self.config = self._load_config(config_path)
        # Initialize the official Google GenAI Client
        # Expects GEMINI_API_KEY environment variable to be set
        self.client = genai.Client()

    def _load_config(self, path: str) -> Dict[str, Any]:
        with open(path, 'r') as file:
            return yaml.safe_load(file)

    def generate_roadmap(self, input_data: StrategyInput) -> str:
        """
        Processes inputs and generates a polished services strategy roadmap using Gemini.
        """
        # Formulate a structured prompt for the model
        prompt = f"""
        Generate a Services Strategy Roadmap for the following customer opportunity:
        
        **Customer:** {input_data.customer_name}
        **Timeline:** {input_data.target_timeline}
        
        **Business Outcomes & Success Metrics (KPIs):**
        {self._format_outcomes(input_data.outcomes_and_kpis)}
        
        **Proposed GCC/Partner Workstreams:**
        {self._format_workstreams(input_data.workstreams)}
        
        Please synthesize this data into a professional Consulting Services Strategy Document. 
        It must contain:
        1. An Executive Summary aligning outcomes to the workstreams.
        2. A Markdown Table mapping: Business Outcome -> Driving Workstream -> KPI Success Metric.
        3. A Phased Implementation Roadmap timeline showing crawl, walk, and run phases.
        """

        response = self.client.models.generate_content(
            model=self.config.get("model_name", "gemini-2.5-pro"),
            contents=prompt,
            config=types.GenerateContentConfig(
                system_instruction=self.config.get("system_instruction"),
                temperature=self.config.get("temperature", 0.2),
            )
        )
        return response.text

    def _format_outcomes(self, items: List[OutcomeKPI]) -> str:
        return "\n".join([f"- Outcome: {item.outcome} | KPI: {item.kpi}" for item in items])

    def _format_workstreams(self, items: List[Workstream]) -> str:
        return "\n".join([f"- {item.name}: {item.description}" for item in items])

# Example Usage
if __name__ == "__main__":
    # Sample input modeling a real-world FSI scenario (like your Aon or Morgan Stanley deals)
    sample_input = StrategyInput(
        customer_name="Aon",
        target_timeline="6 Months (Starting Oct 2026)",
        outcomes_and_kpis=[
            OutcomeKPI(
                outcome="Accelerate risk assessment modeling using Generative AI",
                kpi="Reduce model generation time from 5 days to under 4 hours"
            ),
            OutcomeKPI(
                outcome="Establish robust AI Governance and compliance",
                kpi="100% of deployed models adhere to Google Cloud's Responsible AI guidelines"
            )
        ],
        workstreams=[
            Workstream(
                name="AI Foundation & Vertex AI Setup",
                description="Establish landing zones, secure data pipelines, and configure Vertex AI model registries."
            ),
            Workstream(
                name="Enablement & CoE Framework",
                description="Deliver standard enablement workshops to upskill engineering leads and draft governance playbooks."
            )
        ]
    )

    agent = ServicesStrategyAgent()
    print("🚀 Running Services Strategy Agent...")
    roadmap_output = agent.generate_roadmap(sample_input)
    
    # Save the output to a file
    with open("strategy_roadmap_output.md", "w") as f:
        f.write(roadmap_output)
    print("✅ Roadmap generated and saved to 'strategy_roadmap_output.md'!")
# Services Strategy Roadmap Agent 🚀

This AI Agent automates the generation of client-ready **Services Strategy Roadmaps** for Google Cloud Consulting (GCC) opportunities. It takes strategic business outcomes, maps them to program workstreams, and aligns them to measurable Key Performance Indicators (KPIs).

## Features
- **Pydantic Validation:** Ensures input integrity before generating roadmaps.
- **Configurable Prompting:** Easily modify model instructions inside `config.yaml`.
- **Gemini 2.5 Pro Powered:** Leverages state-of-the-art reasoning to draft executive-ready strategies.

## Setup Instructions

1. **Clone the Repository:**
   ```bash
   git clone https://github.com/your-username/services-roadmap-agent.git
   cd services-roadmap-agent
pip install -r requirements.txt
export GEMINI_API_KEY="your-google-cloud-gemini-api-key"
python agent.py
