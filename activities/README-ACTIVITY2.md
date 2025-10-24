# Protocol Integration Challenge: Operation Strands-MCP

## Lore: The Oracle of AWS

You are an **Elite AI Architect** on the Strands team. Your mission is to upgrade the **"Oracle of AWS,"** a core intelligence responsible for answering mission-critical queries about the AWS cloud. The Oracle's main weakness is a lack of real-time, context-aware information retrieval.

Your solution is to integrate the Oracle with two external data feeds using the **Model Context Protocol (MCP)**:

1.  **The Scroll of Fundamentals (AWS Docs):** Provides foundational AWS knowledge. (Uses `stdio` transport)
2.  **The Whisper of Code (CDK Best Practices):** Provides advanced Infrastructure-as-Code guidance. (Uses `stdio` transport)
3.  **The Digital Calculator (Local Utility):** A simple, fast utility for computational tasks. (Uses `Streamable HTTP` transport)

Your ultimate goal is to create a single, unified Strands Agent that can instantly query all three services simultaneously.

---

## Phase 1: The Foundation - Setting up the Utilities

### **Task 1: The Digital Calculator (Streamable HTTP)**

You must launch the **Digital Calculator** on a local server using the **Streamable HTTP** transport.

1.  **Define and Start the Server:** Recreate the `mcp.FastMCP` server with the two tools: `calculator` and the slow-running `long_running_tool`. Start it in a background thread. _(Ensure you run the necessary imports from the tutorial: `import threading, time, datetime, FastMCP, mcp.client.streamable_http, Agent, MCPClient`)_

    ```python
    # 🚨 Architect's Code Block 1: Define and Start the Server
    import threading
    import time
    from mcp import FastMCP
    from strands.tools.mcp import MCPClient
    from mcp.client.streamable_http import streamablehttp_client
    from strands import Agent

    mcp = FastMCP("Digital Calculator Server")

    @mcp.tool(description="Calculator tool which performs simple integer addition")
    def calculator(x: int, y: int) -> int:
        return x + y

    @mcp.tool(description="This is a tool that takes a long time to run (25 seconds).")
    def long_running_tool(name: str) -> str:
        time.sleep(25)
        return f"Hello, the long-running process for {name} is complete."

    def main():
        # Start the server on http://localhost:8000/mcp
        mcp.run(transport="streamable-http", mount_path="mcp")

    # Start the server in a background thread
    server_thread = threading.Thread(target=main)
    server_thread.start()
    print("Digital Calculator Server online...")

    # Define the client transport function
    def create_streamable_http_transport():
        return streamablehttp_client("http://localhost:8000/mcp")

    # Create the MCPClient for the calculator
    calculator_mcp_client = MCPClient(create_streamable_http_transport)
    ```

### **Task 2: The Scroll & The Whisper (stdio)**

Establish connections to the two external knowledge sources using the **stdio** transport.

```python
# 🚨 Architect's Code Block 2: Connect to External Sources
from mcp import StdioServerParameters, stdio_client

# 1. Scroll of Fundamentals (AWS Docs)
aws_docs_mcp_client = MCPClient(
    lambda: stdio_client(
        StdioServerParameters(
            command="uvx", args=["awslabs.aws-documentation-mcp-server@latest"]
        )
    )
)

# 2. Whisper of Code (CDK Best Practices)
cdk_mcp_client = MCPClient(
    lambda: stdio_client(
        StdioServerParameters(command="uvx", args=["awslabs.cdk-mcp-server@latest"])
    )
)
print("External knowledge sources connected...")
```

---

## Phase 2: The Integration - Building the Oracle

### **Task 3: Assemble the Ultimate Agent**

Now, create the **"Oracle of AWS"** agent by gathering tools from _all three_ MCP clients. The agent must be capable of using multiple tools in parallel.

1.  Use the Python `with` statement to safely handle all three MCP clients.
2.  Combine the tool lists from all clients (`aws_docs`, `cdk`, and `calculator`).
3.  Set the **system prompt** to reflect the agent's new status as an all-knowing AWS expert.
4.  Crucially, set the **`max_parallel_tools`** parameter to **`3`** so it can use all three tool types concurrently (or a sensible high number like 5).

<!-- end list -->

```python
# 🚨 Architect's Code Block 3: Assemble the Oracle
# Define the model ID
MODEL_ID = "us.anthropic.claude-3-7-sonnet-20250219-v1:0"

with aws_docs_mcp_client, cdk_mcp_client, calculator_mcp_client:
    # 1. Gather all tools
    all_tools = (
        aws_docs_mcp_client.list_tools_sync()
        + cdk_mcp_client.list_tools_sync()
        + calculator_mcp_client.list_tools_sync()
    )

    # 2. Create the Oracle Agent
    oracle_agent = Agent(
        model=MODEL_ID,
        tools=all_tools,
        system_prompt="You are the Oracle of AWS, an elite AI capable of answering complex queries by integrating knowledge from AWS Documentation, CDK Best Practices, and a local calculator utility. Be concise and accurate.",
        max_parallel_tools=5 # Enable high concurrency for complex queries
    )

    print("Oracle of AWS Agent initialized with 3 external MCP systems.")
```

---

## Phase 3: The Trial - Mission-Critical Query

### **Task 4: The Triple-Threat Query**

Issue a single, complex query that requires the agent to utilize **at least two, and ideally all three,** of its new MCP tools simultaneously to provide the most complete answer.

- **Query Focus:**
  - **AWS Docs Tool:** Needed for Bedrock pricing or a similar factual detail.
  - **CDK Tool:** Needed for a CDK-related best practice.
  - **Calculator Tool:** Needed for a simple math calculation within the response.

<!-- end list -->

```python
# 🚨 Architect's Code Block 4: The Triple-Threat Query
mission_query = "What is the key difference between AWS Fargate and EC2 spot instances? Also, what are the best practices for handling secrets in an AWS CDK deployment, and finally, what is 42 minus 8?"

print("\n--- INITIATING MISSION-CRITICAL QUERY ---")
response = oracle_agent(mission_query)

print("\n✨ ORACLE RESPONSE (Final Integrated Answer): ✨")
print(response)
```

### **Task 5: Direct Tool Override (The Emergency Stop)**

Before shutting down the systems, demonstrate control by directly invoking the **`long_running_tool`** and immediately enforcing a **timeout** to confirm the agent's ability to handle unresponsive external services.

_(Remember to adjust the `timedelta` for `read_timeout_seconds` to be short, e.g., 5 seconds, since the tool is designed to take 25 seconds.)_

```python
# 🚨 Architect's Code Block 5: The Emergency Stop
from datetime import timedelta

try:
    with calculator_mcp_client:
        print("\n--- INITIATING EMERGENCY LONG-RUNNING TOOL TEST ---")
        result = calculator_mcp_client.call_tool_sync(
            tool_use_id="emergency-test-1",
            name="long_running_tool",
            arguments={"name": "Protocol Override Test"},
            # Set timeout to 5 seconds, much shorter than the 25-second tool run time
            read_timeout_seconds=timedelta(seconds=5),
        )
        print(f"Tool execution status: {result.get('status', 'N/A')}")
        print(f"Tool result content: {result.get('content', 'N/A')}")
except Exception as e:
    print(f"\n✅ EMERGENCY STOP SUCCESSFUL! Tool call failed due to timeout as expected.")
    print(f"Error captured: {str(e)}")
finally:
    # Safely stop the background thread
    # In a real environment, you'd need a robust way to shut down the server process.
    # For this exercise, acknowledge the server thread is running in the background.
    print("\n--- PROTOCOL INTEGRATION CHALLENGE COMPLETE. SYSTEMS OFFLINE. ---")
```

---

By completing this challenge, the Architect has successfully demonstrated mastery over:

1.  **Multiple MCP Transports:** Setting up both `stdio` and `Streamable HTTP`.
2.  **Tool Aggregation:** Combining tools from three independent MCP servers into a single agent.
3.  **Agent Configuration:** Setting the `max_parallel_tools` parameter for enhanced performance.
4.  **Direct Control:** Invoking tools directly and managing timeouts for long-running processes.
