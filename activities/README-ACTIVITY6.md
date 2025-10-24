# 🚀 Project Launch Decision System Challenge

## Scenario: The "Innovate or Wait" Decision

Your team is building a critical multi-agent system, the **Project Launch Decision System (PLDS)**. This system must evaluate a new product concept from **two independent perspectives** simultaneously (Parallel Processing) and then route the aggregated result to the correct leader for a final recommendation based on the combined analysis (Conditional Branching).

## Phase 1: The Components (Nodes)

Define the five specialized agents needed for the graph. Use clear, focused `system_prompt` instructions for each role.

### Task 1.1: Create Agents

Create the following five agents, using the specified names for the graph builder:

1.  **Market Analyst (Node ID: `market_expert`):** Focuses on user demand and competition.
2.  **DevOps Lead (Node ID: `devops_expert`):** Focuses on technical feasibility and infrastructure risks.
3.  **Synthesizer (Node ID: `synthesizer`):** Aggregates and combines the outputs from the two experts.
4.  **Final Classifier (Node ID: `classifier`):** Reads the Synthesizer's output and determines if the result is "HIGH RISK" or "LOW RISK." _It must only output one of these two phrases._
5.  **Executive Summary Agent (Node ID: `executive_summary`):** Provides a final recommendation if the result is "LOW RISK."
6.  **Mitigation Plan Agent (Node ID: `mitigation_plan`):** Provides a final action plan if the result is "HIGH RISK."

<!-- end list -->

```python
# 1.1 Create Specialized Agents
from strands import Agent
from strands.multiagent import GraphBuilder

MODEL_ID = "us.anthropic.claude-3-7-sonnet-20250219-v1:0"

# Parallel Experts
market_analyst = Agent(name="market_analyst", system_prompt="You are a market expert. Analyze user demand, competition, and potential for adoption. Output your findings concisely.")
devops_lead = Agent(name="devops_lead", system_prompt="You are a DevOps expert. Assess technical feasibility, infrastructure load, and deployment complexity. Output your findings concisely.")

# Core Integration
synthesizer = Agent(name="synthesizer", system_prompt="You are the Synthesis Engine. Combine the market analysis and devops assessment into a single, cohesive report on project viability.")

# Decision Branching
final_classifier = Agent(name="final_classifier", system_prompt="You are the Risk Classifier. Analyze the combined report and state ONLY one of these two phrases: 'HIGH RISK' or 'LOW RISK'. Do not add any other text.")
executive_summary = Agent(name="executive_summary", system_prompt="You are the Executive Summary Writer. If the project is LOW RISK, provide a brief, optimistic launch recommendation to the CEO.")
mitigation_plan = Agent(name="mitigation_plan", system_prompt="You are the Risk Mitigation Planner. If the project is HIGH RISK, outline the top 3 critical risks and a plan to address them.")

print("All 6 specialized agents (Nodes) defined.")
```

---

## Phase 2: The Structure (Edges & Conditions)

Now, you'll use the `GraphBuilder` to connect the agents, enforce parallel execution, and establish the conditional logic.

### Task 2.1: Build the Graph Structure

1.  Add all 6 agents as nodes.
2.  Set the entry points to be the two parallel experts.
3.  Define the edges to enforce the following flow:
    - **Parallel:** `market_expert` and `devops_expert` start immediately.
    - **Synchronization:** The `synthesizer` can only run after **both** `market_expert` and `devops_expert` have completed.
    - **Routing:** The `classifier` runs after the `synthesizer`.
    - **Conditional:** Use the `classifier`'s output to route the process to _either_ the `executive_summary` or the `mitigation_plan`.

<!-- end list -->

```python
# 2.1 Define Conditional Functions
def is_high_risk(state):
    """Condition: Check if the classifier output contains 'HIGH RISK'."""
    classifier_result = state.results.get("classifier")
    if not classifier_result:
        return False
    return "high risk" in str(classifier_result.result).lower()

def is_low_risk(state):
    """Condition: Check if the classifier output contains 'LOW RISK'."""
    classifier_result = state.results.get("classifier")
    if not classifier_result:
        return False
    return "low risk" in str(classifier_result.result).lower()

# 2.2 Assemble the Graph
builder = GraphBuilder()

# Add all 6 nodes
builder.add_node(market_analyst, "market_expert")
builder.add_node(devops_lead, "devops_expert")
builder.add_node(synthesizer, "synthesizer")
builder.add_node(final_classifier, "classifier")
builder.add_node(executive_summary, "executive_summary")
builder.add_node(mitigation_plan, "mitigation_plan")

# Set Parallel Entry Points
builder.set_entry_point("market_expert")
builder.set_entry_point("devops_expert")

# Edges for Synchronization (Synthesizer runs after BOTH experts)
builder.add_edge("market_expert", "synthesizer")
builder.add_edge("devops_expert", "synthesizer")

# Edge for Routing
builder.add_edge("synthesizer", "classifier")

# Conditional Edges (Branching)
builder.add_edge("classifier", "executive_summary", condition=is_low_risk)
builder.add_edge("classifier", "mitigation_plan", condition=is_high_risk)

# Finalize the graph
plds_graph = builder.build()
print("PLDS Graph built successfully with Parallel, Synchronization, and Conditional Edges.")
```

---

## Phase 3: The Trial Run

Test the PLDS graph with two different scenarios to ensure the conditional branching works correctly.

### Task 3.1: Scenario A - Technical Triumph (Route to LOW RISK)

Use a prompt that implies minimal technical risk and high demand. The graph should route to the **`executive_summary`** node.

```python
# 3.1 Run Scenario A (LOW RISK path)
low_risk_query = "New Project Concept: A simple, text-based daily AI joke service. Technology is simple API calls. Market demand is high, competition is fragmented. Budget is small."

print("\n\n--- SCENARIO A: LOW RISK (Expected Route: Executive Summary) ---")
result_low_risk = plds_graph(low_risk_query)

print("\nFINAL DECISION (Output Node):")
print(f"Response: {result_low_risk}")

print("\n--- EXECUTION ANALYSIS ---")
print("Nodes Executed:")
for node in result_low_risk.execution_order:
    print(f"- {node.node_id}")

print(f"\nClassifier Result: {result_low_risk.results['classifier'].result}")
```

### Task 3.2: Scenario B - Operational Hazard (Route to HIGH RISK)

Use a prompt that implies significant technical and operational risk. The graph should route to the **`mitigation_plan`** node.

```python
# 3.2 Run Scenario B (HIGH RISK path)
high_risk_query = "New Project Concept: A real-time, global video streaming service requiring proprietary hardware and massive, low-latency infrastructure. Market is saturated, but ROI is potentially huge."

print("\n\n--- SCENARIO B: HIGH RISK (Expected Route: Mitigation Plan) ---")
result_high_risk = plds_graph(high_risk_query)

print("\nFINAL DECISION (Output Node):")
print(f"Response: {result_high_risk}")

print("\n--- EXECUTION ANALYSIS ---")
print("Nodes Executed:")
for node in result_high_risk.execution_order:
    print(f"- {node.node_id}")

print(f"\nClassifier Result: {result_high_risk.results['classifier'].result}")
```

By completing this challenge, you have mastered the core principles of the Strands Agent Graph: **Nodes**, **Parallel Edges** (Synchronization), and **Conditional Edges** (Branching). You successfully built a decision system that dynamically routes the task based on the intermediate result of the classifier agent. 👏
