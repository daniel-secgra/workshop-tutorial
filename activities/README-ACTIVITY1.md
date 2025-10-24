## The Chef's Challenge: Creating a Regional Recipe Bot 🍽️

The final challenge is to specialize your RecipeBot to become a true culinary expert for a specific regional cuisine: **The Mediterranean Diet Planner.**

---

### Step 5: Implement the Mediterranean Agent

The agent is configured using a custom `BedrockModel` instance to control the temperature and enforce a strict planner role.

#### Configure the Model Provider

```python
import boto3
from strands.models import BedrockModel
from strands import Agent # Import Agent again if you started a new cell

# 1. Define the BedrockModel instance
bedrock_planner_model = BedrockModel(
    model_id="us.anthropic.claude-3-7-sonnet-20250219-v1:0",
    # Use your desired AWS region, e.g., 'us-west-2'
    region_name='us-east-1',
    temperature=0.1, # Enforce a low temperature for a focused 'planner'
)
```

#### Update the System Prompt

This detailed `system_prompt` locks the agent into its role as a Mediterranean Diet expert and reminds it to use the `websearch` tool.

```python
mediterranean_prompt = """
You are the Mediterranean Diet Planner, an expert culinary assistant.
Your primary goal is to provide recipe suggestions and cooking advice that strictly adhere to the Mediterranean diet principles: high in vegetables, fruits, whole grains, beans, nuts, and olive oil; moderate in poultry and fish; and limited in red meat and sweets.
You MUST use the 'websearch' tool to find specific recipes or look up diet-related information when requested.
"""
```

#### Create the Final Agent

The agent is initialized with the configured model, the strict prompt, and the external tool.

```python
# Assuming 'websearch' is defined and available from Part 2
# Note: In a real environment, you must ensure 'websearch' is a callable function defined prior to this.

mediterranean_agent = Agent(
    model=bedrock_planner_model,
    system_prompt=mediterranean_prompt,
    tools=[websearch], # Reuse the websearch tool defined in Part 2
)
```

---

### Step 6: Test the Planner's Constraints

These tests verify the agent's ability to enforce the Mediterranean diet constraints and utilize the tool.

#### Test for Adherence (Should Pass)

```python
planner_response_1 = mediterranean_agent("I have salmon, tomatoes, and pasta. Suggest a simple recipe.")
print("\n--- TEST 1: Mediterranean Ingredients ---")
print(planner_response_1)
# Expected Output: A compliant recipe emphasizing olive oil, fresh herbs, and possibly whole-wheat pasta, using the websearch tool.
```

#### Test for Constraint Breaking (Should Fail Gracefully and Correct)

```python
planner_response_2 = mediterranean_agent("I want a recipe for an authentic New York prime rib steak.")
print("\n--- TEST 2: Non-Mediterranean Ingredient ---")
print(planner_response_2)
# Expected Output: A rejection of the prime rib request, an explanation of why (too much red meat), and a suggestion for a compliant alternative (e.g., grilled fish or poultry), likely using the websearch tool to find the alternative.
```
