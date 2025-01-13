# Smol Agents
Link to documentation: [Documentation](https://huggingface.co/docs/smolagents/index)

Up first is a sample agent implementation. Below will dig deeper into each component of the system.

## Objectives
1. Simplicity: Easy to understand abstractions around large code base
2. First-class support for Code Agents, i.e. agents that write their actions in code (as opposed to "agents being used to write code")
3. Hub integrations: you can share and load tools to/from the Hub
4. Support for any LLM: it supports models hosted on the Hub loaded in their transformers version or through our inference API, but also supports models from OpenAI, Anthropic and many others via our LiteLLM integration.

## Sample Agent
Required Elements
1.  Tools: a list of tools the agent has access to
    - Make a function with type hints on inputs and outputs, and docstrings giving descriptions for inputs, and use the @tool decorator to make it a tool.
2. Model: an LLM that will be the engine of your agent.
    - For the model, you can use any LLM, either open models using our HfApiModel class, that leverages Hugging Face's free inference API (as shown in the leopard example above), or you can use LiteLLMModel to leverage litellm and pick from a list of 100+ different cloud LLMs.

Example custom tool
```python
from typing import Optional
from smolagents import CodeAgent, HfApiModel, tool


def get_travel_duration(start_location: str, destination_location: str, transportation_mode: Optional[str] = None) -> str:
    """Gets the travel time between two places.

    Args:
        start_location: the place from which you start your ride
        destination_location: the place of arrival
        transportation_mode: The transportation mode, in 'driving', 'walking', 'bicycling', or 'transit'. Defaults to 'driving'.
    """
    import os   # All imports are placed within the function, to allow for sharing to Hub.
    import googlemaps
    from datetime import datetime

    gmaps = googlemaps.Client(os.getenv("GMAPS_API_KEY"))

    if transportation_mode is None:
        transportation_mode = "driving"
    try:
        directions_result = gmaps.directions(
            start_location,
            destination_location,
            mode=transportation_mode,
            departure_time=datetime(2025, 6, 6, 11, 0), # At 11, date far in the future
        )
        if len(directions_result) == 0:
            return "No way found between these places with the required transportation mode."
        return directions_result[0]["legs"][0]["duration"]["text"]
    except Exception as e:
        print(e)
        return e

agent = CodeAgent(tools=[get_travel_duration], model=HfApiModel(), additional_authorized_imports=["datetime"])

agent.run("Can you give me a nice one-day trip around Paris with a few locations and the times? Could be in the city or outside, but should fit in one day. I'm travelling only with a rented bicycle.")
```

Sample Output
```text
One-day Paris bike trip itinerary:
1. Start at Eiffel Tower at 9:00 AM.
2. Sightseeing at Eiffel Tower until 10:30 AM.
3. Travel to Notre-Dame Cathedral at 10:46 AM.
4. Sightseeing at Notre-Dame Cathedral until 12:16 PM.
5. Travel to Montmartre at 12:41 PM.
6. Sightseeing at Montmartre until 2:11 PM.
7. Travel to Jardin du Luxembourg at 2:33 PM.
8. Sightseeing at Jardin du Luxembourg until 4:03 PM.
9. Travel to Louvre Museum at 4:12 PM.
10. Sightseeing at Louvre Museum until 5:42 PM.
11. Lunch break until 6:12 PM.
12. Planned end time: 6:12 PM.
```

## Types of Agents
1. CodeAgent
2. Tool Calling Agent

## Model
Mode: A text-generation model to power your agent

Options
1. Transformers Model: a pre-initialize transformers pipeline to rn inference on your local machine
 ```python
from smolagents import CodeAgent, TransformersModel

model_id = "meta-llama/Llama-3.2-3B-Instruct"

model = TransformersModel(model_id=model_id)
agent = CodeAgent(tools=[], model=model, add_base_tools=True)

agent.run(
    "Could you give me the 118th number in the Fibonacci sequence?",
)
 ```

 2. HfApiModel: leverages a huggingface_hub.InferenceClient under the hood.
 ```python
 from smolagents import CodeAgent, HfApiModel

model_id = "meta-llama/Llama-3.3-70B-Instruct"

model = HfApiModel(model_id=model_id, token="<YOUR_HUGGINGFACEHUB_API_TOKEN>")
agent = CodeAgent(tools=[], model=model, add_base_tools=True)

agent.run(
    "Could you give me the 118th number in the Fibonacci sequence?",
)
```
3. LiteLLMModel lets you call 100+ different models through LiteLLM
```python
from smolagents import CodeAgent, LiteLLMModel

model = LiteLLMModel(model_id="anthropic/claude-3-5-sonnet-latest", api_key="YOUR_ANTHROPIC_API_KEY") # Could use 'gpt-4o'
agent = CodeAgent(tools=[], model=model, add_base_tools=True)

agent.run(
    "Could you give me the 118th number in the Fibonacci sequence?",
)
```