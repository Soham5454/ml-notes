# 10 — Multi-Agent Systems & Frameworks

Recap file 09's closing note — extending the single-agent loop into systems with MULTIPLE cooperating agents, and covering the popular frameworks that formalize these patterns.

## Why multiple agents — genuinely the same "separation of concerns" principle, one more time

```
Recap Flask file 08's blueprint/separation-of-concerns discussion and DSA
file 19's problem-solving framework directly -- a SINGLE agent trying to
do EVERYTHING (research, write, fact-check, format) in one massive prompt/
tool-set genuinely tends to perform WORSE than several SPECIALIZED agents,
each genuinely focused on one narrow task, coordinating together
```

## A simple multi-agent pattern — genuinely a pipeline of specialized agents

```python
def research_agent(topic):
    # Recap file 09's single-agent loop directly, but with a NARROWLY
    # focused tool-set and system prompt: ONLY research/retrieval tools
    system_prompt = "You are a research agent. Your ONLY job is to gather facts using the search tool."
    return run_agent_with_system_prompt(system_prompt, f"Research: {topic}", tools=[search_tool])

def writer_agent(research_notes):
    system_prompt = "You are a writing agent. Write a clear, engaging article using ONLY the provided research notes."
    return call_llm(f"{system_prompt}\n\nResearch: {research_notes}\n\nWrite the article.")

def editor_agent(draft_article):
    system_prompt = "You are an editing agent. Review this article for clarity, accuracy, and tone. Suggest improvements."
    return call_llm(f"{system_prompt}\n\nArticle: {draft_article}")

# Orchestrating them -- recap file 03's prompt-chaining discussion directly
def generate_article(topic):
    notes = research_agent(topic)
    draft = writer_agent(notes)
    final = editor_agent(draft)
    return final
```

Genuinely worth recognizing this as file 03's prompt-chaining pattern, extended: each STAGE is now genuinely a full agent (potentially with its OWN tools and internal reasoning loop, recap file 09 directly), not just a single prompt — a genuinely more powerful, modular version of the same underlying "break a complex task into focused stages" idea.

## Orchestrator-worker pattern — recap MLOps file 11's Kubernetes orchestration concept directly

```python
def orchestrator_agent(user_request):
    # The orchestrator DECIDES which specialized worker agents are needed,
    # and in what order -- genuinely similar in spirit to MLOps file 11's
    # Kubernetes control plane deciding which pods to schedule and how
    plan_prompt = f"""
    Given this request: {user_request}
    Which of these specialists are needed, and in what order?
    Available specialists: [researcher, writer, coder, data_analyst]
    """
    plan = call_llm(plan_prompt)      # genuinely the orchestrator's "reasoning" step, recap file 03's CoT directly
    # ... parse the plan and dispatch to the appropriate worker agents ...
```

Genuinely worth recognizing the DIRECT parallel to MLOps file 08's CI/CD pipeline dependency structure (`needs:`) — an orchestrator determines the correct SEQUENCE/DEPENDENCY of sub-tasks, exactly the same underlying "figure out the right order of operations" problem DSA file 16's topological sort formally solves, now being solved by an LLM's reasoning rather than an explicit graph algorithm.

## LangChain — genuinely the most popular agent/RAG framework, worth knowing its core abstractions

```python
from langchain.agents import create_tool_calling_agent, AgentExecutor
from langchain_anthropic import ChatAnthropic
from langchain.tools import Tool

llm = ChatAnthropic(model="claude-sonnet-4-5")      # genuinely wraps file 04's raw API calls

search_tool = Tool(name="search", func=search_knowledge_base, description="Search the knowledge base")      # recap file 09's tool definition directly

agent = create_tool_calling_agent(llm, [search_tool], prompt)
agent_executor = AgentExecutor(agent=agent, tools=[search_tool])      # genuinely automates file 09's manual loop
result = agent_executor.invoke({"input": "What's our return policy?"})
```

Genuinely worth understanding what LangChain is actually DOING here — recap file 09's hand-built agent loop directly: `AgentExecutor` is genuinely automating EXACTLY that manual THINK→ACT→OBSERVE loop, the tool-schema definitions, and the message-history management, into a higher-level, reusable abstraction. Directly analogous to Hugging Face file 05's `Trainer` automating PyTorch's manual training loop (recap that comparison directly) — understanding the MANUAL version first (file 09) makes the FRAMEWORK version genuinely transparent rather than magical.

## LangGraph — genuinely important for complex, non-linear agent workflows

```python
from langgraph.graph import StateGraph      # genuinely built on graph concepts, recap DSA file 09 directly

workflow = StateGraph(AgentState)
workflow.add_node("researcher", research_agent)
workflow.add_node("writer", writer_agent)
workflow.add_node("editor", editor_agent)

workflow.add_edge("researcher", "writer")      # recap DSA file 09's graph edges directly
workflow.add_edge("writer", "editor")
workflow.add_conditional_edges(      # genuinely a graph edge with a DECISION function, recap DSA
    "editor",                            # file 09's cycle-detection/conditional-traversal concepts
    lambda state: "writer" if state["needs_revision"] else "END"
)
```

Genuinely worth recognizing this as a DIRECT, literal application of DSA file 09's graph concepts — LangGraph models a multi-agent workflow as an actual GRAPH (nodes = agents/steps, edges = the flow between them), including CONDITIONAL edges and even CYCLES (the editor sending work back to the writer for revision, genuinely a cycle in the graph, recap DSA file 09's cycle-detection discussion directly, though here a cycle is the INTENDED behavior, not a bug to detect). This graph-based framing genuinely handles complex, non-linear workflows (revision loops, conditional branching) more naturally than a simple linear pipeline.

## When frameworks genuinely help vs when they add unnecessary complexity

```
Genuinely the SAME honest engineering-tradeoff framework from throughout
this entire roadmap, one more time: for SIMPLE, linear agent workflows
(file 09's single agent, or a simple 2-3 stage pipeline), hand-rolling
the loop directly (file 09's approach) is genuinely simpler, more
transparent, and easier to debug
For GENUINELY complex, multi-agent, conditional/cyclic workflows, a
framework like LangGraph provides real, meaningful structure and tooling
(state management, visualization, recap MLOps file 11's Kubernetes-
managing-complexity parallel directly) that would be genuinely tedious
and error-prone to hand-build from scratch
```

## Debugging multi-agent systems — genuinely important, real practical challenge

```python
# Recap MLOps file 09's monitoring/observability discussion directly --
# multi-agent systems genuinely need LOGGING at every step to be debuggable
def logged_agent_step(agent_name, input_data, output_data):
    log_entry = {"agent": agent_name, "input": input_data, "output": output_data, "timestamp": time.time()}
    # recap Flask file 09's database-logging pattern directly
    save_to_log_database(log_entry)
```

Genuinely important, real practical necessity — recap MLOps file 09's "monitoring data nobody looks at is genuinely useless" closing point directly: when a multi-agent system produces a WRONG final result, tracing back through EACH agent's individual reasoning/output is genuinely essential for diagnosing WHERE the failure occurred, exactly the same debugging discipline as tracing a bug through a multi-stage data pipeline (recap MLOps file 08's data validation gates directly).

## Try this

```python
# Build the research -> write -> edit pipeline from this file's opening
# example, first as MANUALLY chained function calls (no framework)
# Then rebuild the SAME workflow using LangGraph, including a conditional
# edge where the editor can send the draft BACK to the writer for revision
# if it's judged insufficient (genuinely creating a real cycle in the graph)
# Compare the two implementations honestly: which was easier to build?
# Which would be easier to EXTEND with a 4th agent later? Write down
# concrete, specific observations, not vague impressions
```

Building the SAME workflow both ways (manual and framework-based) is genuinely the best way to develop honest, calibrated judgment about when a framework's added structure is genuinely worth its complexity cost, rather than defaulting to either approach out of habit.

## What's next
File 11 — Evaluating LLM Applications, covering genuinely how to MEASURE whether any of this folder's systems (RAG, agents, fine-tuned models) actually WORK well, rather than just appearing to work on a few manually-checked examples.
