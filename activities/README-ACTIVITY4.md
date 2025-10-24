# 👑 The Delegation Challenge: Building the Strands Command Center

## Mission: The 'Agents as Tools' Strategy

You are an **Orchestration Specialist** 👑. Your task is to build the **Strands Command Center**, a hierarchical AI system where a single **Orchestrator Agent (The Manager)** delegates user requests to three highly specialized **Tool Agents (The Specialists)**.

This challenge reinforces the idea that an agent can be a reusable tool, just like a function, leading to a modular and efficient architecture.

## Phase 1: The Specialists (Agents as Tools)

First, you'll define the three specialized agents and wrap them into callable Python functions using the `@tool` decorator.

### Task 1.1: The Specialists' Blueprints

Define the specialized agents, ensuring each has a **highly focused system prompt** and is wrapped in a Python function with clear documentation.

```python
# 1.1 Imports (Run this first)
import os
from strands import Agent, tool
from strands_tools import file_write

# Disable tool consent for smooth operation in this tutorial
os.environ["BYPASS_TOOL_CONSENT"] = "true"

# Define the base model ID for simplicity
MODEL_ID = "us.anthropic.claude-3-7-sonnet-20250219-v1:0"
```

#### A. The Research Analyst 🔎

This agent handles factual queries and is essential for Step 2.

```python
@tool
def research_analyst(query: str) -> str:
    """
    Handles factual inquiries, providing well-sourced information and analysis.
    Use this for questions about technology, history, or current events.
    """
    try:
        # Create a specialized agent inside the tool function
        research_agent = Agent(
            model=MODEL_ID,
            system_prompt="You are a specialized Research Analyst. Provide factual, well-sourced information ONLY.",
        )
        response = research_agent(query)
        return str(response)
    except Exception as e:
        return f"Error with Research Analyst: {str(e)}"
```

#### B. The Travel Planner ✈️

This agent handles all itinerary and travel advice.

```python
@tool
def travel_planner(query: str) -> str:
    """
    Creates detailed travel itineraries and provides travel-related advice.
    Use this for requests involving destinations, dates, or activity planning.
    """
    try:
        travel_agent = Agent(
            model=MODEL_ID,
            system_prompt="You are a specialized Travel Planning Assistant. Create detailed itineraries and travel suggestions.",
        )
        response = travel_agent(query)
        return str(response)
    except Exception as e:
        return f"Error with Travel Planner: {str(e)}"
```

#### C. The Recommendation Specialist 🛍️

This agent focuses on suggesting products and services.

```python
@tool
def recommendation_specialist(query: str) -> str:
    """
    Provides personalized product and service recommendations based on user preferences.
    Use this for shopping advice or product comparisons.
    """
    try:
        product_agent = Agent(
            model=MODEL_ID,
            system_prompt="You are a specialized Product Recommendation Specialist. Offer personalized product suggestions.",
        )
        response = product_agent(query)
        return str(response)
    except Exception as e:
        return f"Error with Recommendation Specialist: {str(e)}"
```

---

## Phase 2: The Orchestrator (The Manager)

Now, you'll create the primary agent whose only job is to read the user's request and delegate it to the correct specialist tool.

### Task 2.1: Define the Orchestrator

Define a strict `MAIN_SYSTEM_PROMPT` to guide the orchestrator's decision-making. Then, initialize the agent, providing the three specialized tools and the built-in `file_write` tool.

```python
# 2.1 Orchestrator Configuration
MAIN_SYSTEM_PROMPT = """
You are the **Strands Command Center Orchestrator**. Your job is to delegate tasks to the correct specialized agent tool:
1.  **research_analyst**: For factual questions (e.g., 'What is...?', 'Who invented...?').
2.  **travel_planner**: For requests involving trips, destinations, or itineraries.
3.  **recommendation_specialist**: For requests involving shopping, products, or services.
4.  **file_write**: To save text to a file.
5.  **Answer Directly**: For simple greetings or non-specialized queries.

Your ONLY priority is correct delegation. Do NOT try to answer the specialized query yourself.
"""

# Initialize the Orchestrator with ALL specialized agents as tools
orchestrator = Agent(
    model=MODEL_ID,
    system_prompt=MAIN_SYSTEM_PROMPT,
    tools=[
        research_analyst,
        travel_planner,
        recommendation_specialist,
        file_write, # Built-in utility tool
    ],
)

print("Strands Command Center Orchestrator is operational. Ready for delegation.")
```

---

## Phase 3: The Delegation Test (Single and Multi-Agent)

Test the Orchestrator with queries that require delegation, including a complex query that forces it to call multiple specialists.

### Task 3.1: Single Delegation Test

Ask three questions, each requiring a different specialist.

```python
print("\n--- TEST 1: Single Delegation ---")

# A. Test Research Analyst
query_A = "What are the core differences between a quantum computer and a classical computer?"
print(f"\nUser Query A (Research): {query_A}")
response_A = orchestrator(query_A)
print(f"Orchestrator's Final Response:\n{response_A}")

# B. Test Travel Planner
query_B = "I want a 5-day itinerary for a relaxing trip to the Swiss Alps."
print(f"\nUser Query B (Travel): {query_B}")
response_B = orchestrator(query_B)
print(f"Orchestrator's Final Response:\n{response_B}")

# C. Test Recommendation Specialist
query_C = "I need a recommendation for a durable, lightweight drone under $500."
print(f"\nUser Query C (Recommendation): {query_C}")
response_C = orchestrator(query_C)
print(f"Orchestrator's Final Response:\n{response_C}")
```

### Task 3.2: Multi-Agent Coordination Test 🤝

Pose a single query that requires the orchestrator to call two different specialist agents sequentially (or in parallel, depending on the model's capability) and then synthesize their responses.

```python
print("\n--- TEST 2: Multi-Agent Coordination ---")

complex_query = "What is the history of the Eiffel Tower, and based on that, can you suggest a souvenir I should buy when I visit?"

print(f"\nUser Query (Complex): {complex_query}")
response_complex = orchestrator(complex_query)

print("\nOrchestrator's Final Cohesive Response:")
print(response_complex)

# Optional: Inspecting the Orchestrator's internal messages confirms the tool calls.
print("\n--- Internal Messages (Proof of Delegation) ---")
# The messages will show: [User Query] -> [Tool Use 1] -> [Tool Result 1] -> [Tool Use 2] -> [Tool Result 2] -> [Final Answer]
for message in orchestrator.messages[-10:]:
    if message['role'] == 'tool':
        print(f"✅ DELEGATION SUCCESS: Orchestrator executed tool: {message['content'][0]['toolResult']['toolUseId']}")
```

By completing this challenge, you've mastered the **Agents as Tools** pattern, demonstrating how to build a powerful, modular, and hierarchical AI system by delegating complex problems to specialized sub-agents.
