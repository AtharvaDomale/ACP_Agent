# ACP_Agent

## Quickstart

> [!NOTE]
> This guide uses `uv`. See the [`uv` primer](https://agentcommunicationprotocol.dev/introduction/uv-primer) for more details.

**1. Initialize your project**

```sh
uv init --python '>=3.11' my_acp_project
cd my_acp_project
```

**2. Add the ACP SDK**

```sh
uv add acp-sdk
```

**3. Create an agent**

Let's create a simple "echo agent" that returns any message it receives.  
Create an `agent.py` file in your project directory with the following code:

```python
# agent.py
import asyncio
from collections.abc import AsyncGenerator

from acp_sdk.models import Message
from acp_sdk.server import Context, RunYield, RunYieldResume, Server

server = Server()


@server.agent()
async def echo(
    input: list[Message], context: Context
) -> AsyncGenerator[RunYield, RunYieldResume]:
    """Echoes everything"""
    for message in input:
        await asyncio.sleep(0.5)
        yield {"thought": "I should echo everything"}
        await asyncio.sleep(0.5)
        yield message


server.run()
```

**4. Start the ACP server**

```sh
uv run agent.py
```

Your server should now be running at http://localhost:8000.

**5. Verify your agent is available**

In another terminal, run the following `curl` command:

```sh
curl http://localhost:8000/agents
```

You should see a JSON response containing your `echo` agent, confirming it's available:

```json
{
  "agents": [
    { "name": "echo", "description": "Echoes everything", "metadata": {} }
  ]
}
```

**6. Run the agent via HTTP**

Run the following `curl` command:

```sh
curl -X POST http://localhost:8000/runs \
  -H "Content-Type: application/json" \
  -d '{
        "agent_name": "echo",
        "input": [
          {
            "role": "user",
            "parts": [
              {
                "content": "Howdy!",
                "content_type": "text/plain"
              }
            ]
          }
        ]
      }'
```

Your response should include the echoed message "Howdy!":

```json
{
  "run_id": "44e480d6-9a3e-4e35-8a03-faa759e19588",
  "agent_name": "echo",
  "session_id": "b30b1946-6010-4974-bd35-89a2bb0ce844",
  "status": "completed",
  "await_request": null,
  "output": [
    {
      "role": "agent/echo",
      "parts": [
        {
          "name": null,
          "content_type": "text/plain",
          "content": "Howdy!",
          "content_encoding": "plain",
          "content_url": null
        }
      ]
    }
  ],
  "error": null
}
```

**7. Build an ACP client**

Here's a simple ACP client to interact with your `echo` agent.  
Create a `client.py` file in your project directory with the following code:

```python
# client.py
import asyncio

from acp_sdk.client import Client
from acp_sdk.models import Message, MessagePart


async def example() -> None:
    async with Client(base_url="http://localhost:8000") as client:
        run = await client.run_sync(
            agent="echo",
            input=[
                Message(
                    parts=[MessagePart(content="Howdy to echo from client!!", content_type="text/plain")]
                )
            ],
        )
        print(run.output)


if __name__ == "__main__":
    asyncio.run(example())
```

**8. Run the ACP client**

```sh
uv run client.py
```

You should see the echoed response printed to your console. 🎉