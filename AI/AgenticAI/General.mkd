# Agentic AI Notes - Non Platform Specific

## What are agents
1. Agentic programs are the gateway to the outside world for LLMs
2. AI Agents are programs where the LLLM outputs control the workflow
3. Influence of LLM's input on the code workflow is the level of agency of LLMs in the system 
    | Level | Description                                             | What it's Called | Example Pattern |
    |-------|---------------------------------------------------------|------------------|-----------------|
    | 0     | LLM output has no impact on program flow	              | Simple processor | process_llm_output(llm_response)                  |
    | 1     | LLM output determines basic control flow                | Router           | if (llm_decision()) path_a() else: path_b()        |
    | 2     | LLM output determines function execution                | Tool Call        | run_function(llm_chosen_tool, llm_chosen_args)    |
    | 3     | LLM output controls iteration and program continuation  | Multi-step Agent | while llm_should_continue(): execute_neect_step() |
    | 3     | One agentic workflow can start another agentic workflow | Multi-Agent	     | if llm_trigger(): execute_agent()                 |

4. Mulit Agent Code Structure
    System runs in a loop execexuting a new action at each step
            i. Action involved calling some prederetmined tools that are just functions
            Loops until its observations make it apparent that a satisfactory state has been reached

## When to Use Agents

1. Use when the worflow is not deterministic
2. If deterministic, do not use agents
    - Code like normal and use logic that can easily be 100% testable and predictable

## Code Agents
1. In a multi-step agent, at each step, the LLM can write and action in the form of calls to external tools
    - Generally written as a JSON structure that contains tool name and arguments to pass to the tool
    - However papers have shown that generating code is much better than generating JSON snippets
        - [Executable Code Actions Elicit Better LLM Agents](https://huggingface.co/papers/2402.01030_)
        - [DynaSaur: Large Language Agents Beyond Predefined Actions](https://huggingface.co/papers/2411.01747)
        - [If LLM Is the Wizard, Then Code Is the Wand: A Survey on How Code Empowers Large Language Models to Serve as Intelligent Agents](https://huggingface.co/papers/2401.00812)
2. Writting code instead of snippets is better for the following reasons:
    - Composability: could you nest JSON actions within each other, or define a set of JSON actions to re-use later, the same way you could just define a python function
    - Object management: how do you store the output of an action like generate_image in JSON?
    - Generality: code is built to express simply anything you can have a computer do.
    - Representation in LLM training data: plenty of quality code actions is already included in LLMs’ training data which means they’re already trained for this

## Libraries of Note
[Smol Agents](./SmolAgents.md)