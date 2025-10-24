# 🛠️ The AI Secretary Upgrade Challenge: In-Memory Edition 🧠

## Mission Objective

You are tasked with upgrading the **Strands Personal Assistant Agent** (nicknamed **"Chronos"** ⏳) to manage a schedule using simple **in-memory data structures**, while simultaneously enhancing its core reasoning abilities using Claude's **Extended Thinking** feature.

This exercise covers:

1.  Defining custom tools that interact with **in-memory state**.
2.  Using both the `@tool` decorator and the `TOOL_SPEC` dictionary.
3.  Configuring the `BedrockModel` to enable enhanced **Extended Thinking**.

## Phase 1: Custom Tool Blueprint (In-Memory)

We will use a simple global list, `APPOINTMENTS`, to simulate the database. Your first task is to define the custom tools that interact with this memory.

### Task 1.1: Environment Setup and Global Memory

Execute the necessary imports and define the global list that will hold all appointments.

```python
# 1.1 Setup and Global Memory
import json
import uuid
from datetime import datetime
from typing import Any, Dict, List, Optional
from strands import Agent, tool
from strands.models import BedrockModel
from strands_tools import calculator, current_time # Built-in tools
from strands.types.tools import ToolResult, ToolUse # Needed for TOOL_SPEC approach

# Global memory to simulate the database
APPOINTMENTS: List[Dict[str, str]] = []
```

### Task 1.2: Define Custom Tools

Define all three custom tools (`create`, `list`, `update`).

#### A. `create_appointment` (Decorator Approach)

This tool uses the simple **`@tool` decorator** and manipulates the global `APPOINTMENTS` list.

```python
# A. Custom Tool: create_appointment (@tool Decorator)
@tool
def create_appointment(date: str, location: str, title: str, description: str) -> str:
    """
    Create a new personal appointment in the in-memory schedule.
    Args:
        date (str): Date and time of the appointment (format: YYYY-MM-DD HH:MM).
        location (str): Location of the appointment.
        title (str): Title of the appointment.
        description (str): Description of the appointment.
    Returns:
        str: The ID of the newly created appointment.
    """
    # 1. Validate date format (simplified)
    try:
        datetime.strptime(date, "%Y-%m-%d %H:%M")
    except ValueError:
        return "ERROR: Date must be in format 'YYYY-MM-DD HH:MM'"

    # 2. Generate ID and create appointment
    appointment_id = str(uuid.uuid4())
    new_appointment = {
        'id': appointment_id,
        'date': date,
        'location': location,
        'title': title,
        'description': description
    }

    global APPOINTMENTS
    APPOINTMENTS.append(new_appointment)

    return f"Appointment with id {appointment_id} created"
```

#### B. `list_appointments` (Decorator Approach - Simpler)

```python
# B. Custom Tool: list_appointments (@tool Decorator)
@tool
def list_appointments() -> str:
    """
    List all available appointments from the in-memory schedule.
    Returns:
        str: A JSON string of the appointments available.
    """
    global APPOINTMENTS
    if not APPOINTMENTS:
        return "No appointments found."

    return json.dumps(APPOINTMENTS, indent=2)
```

#### C. `update_appointment` (TOOL_SPEC Approach)

This requires a slightly more complex function signature and return type (`ToolResult`), but is essential for showing the second tool definition method.

```python
# C. Custom Tool: update_appointment (TOOL_SPEC Approach)
UPDATE_APPOINTMENT_TOOL_SPEC = {
    "name": "update_appointment",
    "description": "Update an appointment based on the appointment ID. Any parameter can be updated.",
    "inputSchema": {
        "json": {
            "type": "object",
            "properties": {
                "appointment_id": {"type": "string", "description": "The appointment id."},
                "date": {"type": "string", "description": "New date (format: YYYY-MM-DD HH:MM).", "nullable": True},
                "location": {"type": "string", "description": "New location.", "nullable": True},
                "title": {"type": "string", "description": "New title.", "nullable": True},
                "description": {"type": "string", "description": "New description.", "nullable": True}
            },
            "required": ["appointment_id"]
        }
    }
}

# The function name must match the tool name from the TOOL_SPEC
def update_appointment(tool_use: ToolUse, **kwargs: Any) -> ToolResult:
    tool_use_id = tool_use["toolUseId"]
    args = tool_use["input"]
    appointment_id = args["appointment_id"]

    global APPOINTMENTS

    # Find the appointment by ID
    found_appt: Optional[Dict[str, str]] = next((appt for appt in APPOINTMENTS if appt['id'] == appointment_id), None)

    if not found_appt:
        return {
            "toolUseId": tool_use_id,
            "status": "error",
            "content": [{"text": f"Appointment {appointment_id} does not exist."}]
        }

    updates_made = False

    # Apply updates
    if 'date' in args and args['date']:
        try:
            datetime.strptime(args['date'], '%Y-%m-%d %H:%M')
            found_appt['date'] = args['date']
            updates_made = True
        except ValueError:
            return {
                "toolUseId": tool_use_id,
                "status": "error",
                "content": [{"text": "Date must be in format 'YYYY-MM-DD HH:MM'"}]
            }

    for field in ['location', 'title', 'description']:
        if field in args and args[field]:
            found_appt[field] = args[field]
            updates_made = True

    if not updates_made:
        return {
            "toolUseId": tool_use_id,
            "status": "success",
            "content": [{"text": "No updates provided. Appointment unchanged."}]
        }

    return {
        "toolUseId": tool_use_id,
        "status": "success",
        "content": [{"text": f"Appointment {appointment_id} updated with success."}]
    }
```

**Helper Function for TOOL_SPEC:** Because the `update_appointment` function uses the `TOOL_SPEC` format, it must be combined with the spec before being passed to the agent.

```python
# Helper to combine function and TOOL_SPEC
def get_update_tool():
    return (update_appointment, UPDATE_APPOINTMENT_TOOL_SPEC)
```

---

## Phase 2: Chronos Pro Agent Assembly

### Task 2.1: Configure the Thinking Model

Configure the **Chronos Pro** agent by adding the necessary fields to the `BedrockModel` to enable **Extended Thinking**.

```python
# 2.1 Configure the Thinking Model (The Upgrade)
thinking_model = BedrockModel(
    model_id="us.anthropic.claude-3-7-sonnet-20250219-v1:0",
    additional_request_fields={
        "thinking": {
            "type": "enabled",
            "budget_tokens": 2048, # Allocate reasoning budget
        }
    },
)

print("Thinking Model configured for Chronos Pro.")
```

### Task 2.2: Create Chronos Pro

Initialize the agent with the new `thinking_model`, all built-in and custom tools, and a prompt that encourages thinking.

```python
# 2.2 Create Chronos Pro Agent
thinking_system_prompt = """You are Chronos Pro, an advanced personal assistant. You manage appointments in a schedule, can perform math, and know the current time.
You are trained to think through your steps before answering.
Use the 'create_appointment' tool to book new items and provide the ID. Use the 'update_appointment' tool to change existing items.
"""

chronos_pro = Agent(
    model=thinking_model,
    system_prompt=thinking_system_prompt,
    tools=[
        current_time,
        calculator,
        create_appointment, # Custom tool (Decorator)
        list_appointments,  # Custom tool (Decorator)
        get_update_tool(),  # Custom tool (TOOL_SPEC via helper)
    ],
)

print("Chronos Pro Agent initialized with In-Memory Tools and Thinking enabled.")
```

---

## Phase 3: The Triple Test Drive

### Task 3.1: Sequential Actions Test

Ask Chronos Pro to perform two sequential actions: **book an appointment** and then **update it** using the information it received from the first step.

```python
# 3.1 Test 1: Create Appointment
print("\n--- TEST 1: Creating an Appointment ---")
create_query = "Book 'Project Alpha Kickoff' for 2025-11-01 10:00 in London. Topic: Agent Architecture."
result_1 = chronos_pro(create_query)
print(f"Chronos Pro Response (Create): \n{result_1}")

# Get the ID from the last message (a manual step to simulate user knowledge)
# Note: In a real flow, the agent provides the ID in its response, so the user knows it.
first_appointment_id = result_1.split("id ")[-1].split(" created")[0].strip()

# 3.2 Test 2: Update Appointment using the returned ID
print("\n--- TEST 2: Updating the Appointment ---")
update_query = f"I need to change the location of that 'Project Alpha Kickoff' to Paris instead of London. The ID is {first_appointment_id}."
result_2 = chronos_pro(update_query)
print(f"Chronos Pro Response (Update): \n{result_2}")

# 3.3 Test 3: Verification (Direct Tool Call)
print("\n--- TEST 3: Direct Verification of Update ---")
verification_result = chronos_pro.tool.list_appointments()
print("Current Appointments (Verified):")
print(verification_result)
```

### Task 3.4: Observe Thinking

Finally, examine the agent's messages to confirm the **`reasoningContent`** blocks were generated during the complex update task, proving the **Extended Thinking** upgrade is active.

```python
# 3.4 Observe Reasoning Output
print("\n--- ANALYSIS: Agent Thinking Process ---")

# The messages from the complex update (result_2)
update_messages = chronos_pro.messages[-4:]

for message in update_messages:
    if message['role'] == 'assistant':
        print(f"\nAssistant Message (Time: {message['timestamp']}):")
        for content_block in message['content']:
            if 'reasoningContent' in content_block:
                print("🧠 EXTENDED THINKING BLOCK FOUND:")
                print(content_block['reasoningContent']['text'])
            elif 'toolUse' in content_block:
                print("🛠️ TOOL USE INTENDED:")
                print(json.dumps(content_block['toolUse'], indent=2))
            elif 'text' in content_block:
                print("➡️ FINAL RESPONSE:")
                print(content_block['text'])

print("\n\nCustom tools are working, and the Extended Thinking feature is active! Mission complete. 🎉")
```
