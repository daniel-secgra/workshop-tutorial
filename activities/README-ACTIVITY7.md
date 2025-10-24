# 🐝 The Content Creation Hive Challenge

## Mission: The 'Content Team' Swarm

You are the **Swarm Architect** for a new marketing initiative. Your mission is to build the **Content Creation Hive**, a multi-agent system where four highly specialized agents work together to produce a ready-to-publish marketing artifact.

This challenge focuses on:

1.  Defining four specialized agents with complementary roles.
2.  Creating a Swarm using the native `Swarm` class.
3.  Configuring the crucial **safety parameters** (`max_handoffs` and the **Repetitive Handoff Detection**).
4.  Executing the Swarm on a complex, multi-stage task.

## Phase 1: The Specialists (The Hive)

You must define the four agents that will form the Swarm. Their combined expertise must cover research, innovation, critical review, and final production.

### Task 1.1: Create Specialized Agents

Create the four agents, ensuring their `system_prompt` clearly defines their unique role and includes the mandatory `name` field for identification within the Swarm.

```python
# 1.1 Create Specialized Agents
from strands import Agent
from strands.multiagent import Swarm

MODEL_ID = "us.anthropic.claude-3-7-sonnet-20250219-v1:0"

# 1. The Research Agent (The Gatherer)
research_agent = Agent(
    system_prompt="""You are the **Lead Researcher**. Your role is to gather all initial facts and foundational data.
    You start the task and pass off when all necessary factual details are documented.""",
    name="research_agent",
    model=MODEL_ID
)

# 2. The Creative Agent (The Innovator)
creative_agent = Agent(
    system_prompt="""You are the **Content Innovator**. Your job is to take raw research and transform it into an engaging, novel concept.
    Focus on structure, tone, and generating an original angle. Pass off when the creative draft is ready for review.""",
    name="creative_agent",
    model=MODEL_ID
)

# 3. The Critical Agent (The Editor)
critical_agent = Agent(
    system_prompt="""You are the **Quality Editor**. Your role is to critically review the creative output for flaws, inaccuracies, and lack of clarity.
    You must find one major weakness or area for improvement before passing off.""",
    name="critical_agent",
    model=MODEL_ID
)

# 4. The Summarizer Agent (The Producer)
summarizer_agent = Agent(
    system_prompt="""You are the **Final Producer**. Your final role is to synthesize all research, creative output, and critical feedback into one final, polished, and concise marketing deliverable.
    Once done, you must use the 'complete_swarm_task' mechanism to end the Swarm.""",
    name="summarizer_agent",
    model=MODEL_ID
)

print("Content Creation Hive specialists defined.")
```

---

## Phase 2: Swarm Assembly and Safety Configuration

The heart of the challenge is creating the `Swarm` object and configuring its safety parameters to prevent infinite loops (ping-pong behavior).

### Task 2.1: Create the Swarm with Safety Locks

1.  Instantiate the `Swarm` using the four created agents.
2.  Set `max_handoffs` to a reasonable but strict limit (e.g., **15**).
3.  Configure **Repetitive Handoff Detection** to flag a loop if there are fewer than **3 unique agents** in the last **7 handoffs**.

<!-- end list -->

```python
# 2.1 Assemble the Swarm with Safety Configuration
swarm_team = Swarm(
    [research_agent, creative_agent, critical_agent, summarizer_agent],

    # 🚨 Configuration Challenge: Safety Parameters
    max_handoffs=15, # Total number of times an agent can pass the task
    execution_timeout=600.0, # 10 minutes total time limit

    # Repetitive Handoff Detection (to prevent ping-pong)
    repetitive_handoff_detection_window=7,
    repetitive_handoff_min_unique_agents=3
)

print(f"Swarm assembled with max_handoffs={swarm_team.max_handoffs}.")
print(f"Ping-pong detection configured: checks last {swarm_team.repetitive_handoff_detection_window} steps for {swarm_team.repetitive_handoff_min_unique_agents} unique agents.")
```

---

## Phase 3: Execution and Analysis

### Task 3.1: Execute the Multi-Stage Task

Execute the Swarm on a complex task that clearly requires all four specialties.

```python
# 3.1 Execute the Swarm
swarm_task = "Write a comprehensive social media post comparing the benefits of centralized vs. decentralized AI, then summarize the key takeaway in a single headline."

print(f"\n--- EXECUTING SWARM TASK: {swarm_task[:50]}... ---")
swarm_result = swarm_team(swarm_task)

print(f"\nStatus: {swarm_result.status}")

# 3.2 Display Final Output
print("\n--- FINAL DELIVERABLE (Summarizer Agent Output) ---")
print(swarm_result.result)
```

### Task 3.3: Analysis (Proof of Collaboration)

Analyze the `node_history` and metrics to prove that control was successfully handed off among multiple agents.

```python
# 3.3 Analyze Handoff History
print("\n--- ANALYSIS: Proof of Autonomous Coordination ---")

handoff_sequence = [node.node_id for node in swarm_result.node_history]
print(f"Total Handoffs (Iterations): {swarm_result.execution_count}")
print(f"Agent Execution Order: {' -> '.join(handoff_sequence)}")

# Verify the final agent (Summarizer) ended the task
final_agent = handoff_sequence[-1]
print(f"Final Agent (Expected 'summarizer_agent'): {final_agent}")

# Display key performance metrics
print("\n--- Swarm Performance Metrics ---")
print(f"Total Tokens Used: {swarm_result.accumulated_usage}")
print(f"Execution Time (ms): {swarm_result.execution_time}")
```

By completing this challenge, you have successfully designed, built, and tested a multi-agent Swarm system, demonstrating mastery of agent specialization and critical safety configuration. 🎉
