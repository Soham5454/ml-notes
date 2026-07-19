# 09 — LLM Agents & Tool Use

Recap file 08's closing note, file 03's ReAct preview, and file 04's function-calling foundation directly — the full technical treatment of building systems that genuinely TAKE ACTIONS, not just generate text.

## What an agent actually is — genuinely just a loop around an LLM

```
Recap file 03's ReAct pattern and file 04's tool-use round trip directly --
an "agent" is genuinely just: an LLM, a set of available TOOLS (file 04),
and a LOOP that repeats: let the model THINK, let it CHOOSE an action
(use a tool, or give a final answer), EXECUTE that action, feed the
RESULT back, repeat until done
Genuinely worth demystifying this directly -- "agent" sounds like a
sophisticated new concept, but it's structurally just DSA file 19's
iterative problem-solving loop, with an LLM making the decisions instead
of a human following a checklist
```

## A complete, minimal agent loop — recap file 04's tool-use pattern, now as a full loop

```python
import anthropic
import json

client = anthropic.Anthropic()

tools = [
    {
        "name": "calculator",
        "description": "Evaluate a mathematical expression",
        "input_schema": {"type": "object", "properties": {"expression": {"type": "string"}}, "required": ["expression"]}
    },
    {
        "name": "search_knowledge_base",      # recap file 06's RAG retrieval directly, now exposed AS a tool
        "description": "Search the company knowledge base for relevant information",
        "input_schema": {"type": "object", "properties": {"query": {"type": "string"}}, "required": ["query"]}
    }
]

def execute_tool(tool_name, tool_input):
    if tool_name == "calculator":
        return str(eval(tool_input["expression"]))      # genuinely a real security concern, covered below
    elif tool_name == "search_knowledge_base":
        results = collection.query(query_texts=[tool_input["query"]], n_results=3)      # recap file 05/06 directly
        return "\n".join(results['documents'][0])

def run_agent(user_query, max_iterations=5):
    messages = [{"role": "user", "content": user_query}]

    for _ in range(max_iterations):      # recap DSA file 02's recursion base-case discipline directly --
                                              # genuinely important to have a hard iteration LIMIT, preventing
                                              # infinite loops
        response = client.messages.create(
            model="claude-sonnet-4-5", max_tokens=1024, tools=tools, messages=messages
        )

        if response.stop_reason != "tool_use":
            return response.content[0].text      # genuinely the FINAL answer, no more tool calls needed

        messages.append({"role": "assistant", "content": response.content})

        tool_results = []
        for block in response.content:
            if block.type == "tool_use":
                result = execute_tool(block.name, block.input)
                tool_results.append({"type": "tool_result", "tool_use_id": block.id, "content": result})

        messages.append({"role": "user", "content": tool_results})

    return "Reached maximum iterations without a final answer"
```

Genuinely worth understanding this loop's structure deeply — recap file 03's ReAct THINK→ACT→OBSERVE cycle directly: each iteration, the model either DECIDES it has enough information (returns a final text answer, loop exits) or REQUESTS a tool call (loop continues, feeding the tool's result back as new context). The `max_iterations` limit is genuinely essential — recap the general "always have a termination guarantee" discipline from DSA's recursion/backtracking discussions — without it, a genuinely confused agent could loop indefinitely, and recap Cloud file 10's cost-management discipline directly, each iteration is a real, billed API call.

## The security concern with `eval()` — recap Flask file 04's input-validation discipline directly

```python
# GENUINELY DANGEROUS as written above:
return str(eval(tool_input["expression"]))
# eval() executes ARBITRARY Python code -- recap Flask file 04's "never
# trust client input" principle directly, now applied to LLM-GENERATED
# input, which is genuinely ALSO untrusted from a security perspective
# (recap Cloud file 12's prompt-injection preview directly)
```

```python
import ast
import operator

def safe_calculate(expression):      # genuinely a SAFE alternative, recap DSA's parsing concepts directly
    allowed_ops = {ast.Add: operator.add, ast.Sub: operator.sub, ast.Mult: operator.mul, ast.Div: operator.truediv}

    def eval_node(node):
        if isinstance(node, ast.Num):
            return node.n
        elif isinstance(node, ast.BinOp):
            return allowed_ops[type(node.op)](eval_node(node.left), eval_node(node.right))
        raise ValueError("Unsupported expression")

    tree = ast.parse(expression, mode='eval')
    return eval_node(tree.body)
```

Genuinely important, real security principle worth stating directly — recap the entire "never trust client input" theme running through Flask/Cloud folders: even though the "client" here is an LLM rather than a human user, its OUTPUT is genuinely still untrusted input to your application code, and should be validated/sandboxed with the exact same rigor as any external, unverified input.

## Giving agents access to real, powerful tools — recap the whole roadmap's toolkit directly

```python
tools_extended = [
    # Recap SQL folder directly -- a genuinely powerful, real capability
    {"name": "query_database", "description": "Run a read-only SQL query against the sales database", ...},
    # Recap file 06's RAG directly
    {"name": "search_docs", "description": "Search internal documentation", ...},
    # Recap Cloud file 03's Lambda/API discussion directly
    {"name": "call_external_api", "description": "Call a specified external API endpoint", ...},
]
```

Genuinely worth recognizing this as where the ENTIRE roadmap's skill set becomes directly relevant — recap SQL file 10's transaction/safety discipline directly: a `query_database` tool should genuinely be READ-ONLY by default (recap the principle of least privilege from Cloud file 02 directly) unless a write capability is GENUINELY, deliberately intended and carefully safeguarded.

## Human-in-the-loop confirmation — genuinely important for high-stakes actions

```python
def execute_tool_with_confirmation(tool_name, tool_input):
    if tool_name in ["send_email", "delete_record", "make_purchase"]:      # genuinely HIGH-STAKES actions
        print(f"Agent wants to: {tool_name}({tool_input}). Confirm? (y/n)")
        if input().lower() != 'y':
            return "Action cancelled by user"
    return execute_tool(tool_name, tool_input)
```

Genuinely important, real practical safeguard — recap MLOps file 10's "reduce blast radius" deployment philosophy directly, now applied to agent actions: for genuinely consequential actions (sending real emails, deleting real data, spending real money), inserting a HUMAN CONFIRMATION step before execution is a real, sensible safety practice, directly recapping the general "reduce blast radius of a potentially wrong decision" theme running throughout this entire roadmap.

## Agent memory — recap Flask file 10's session discussion directly

```python
class AgentSession:      # recap Flask file 10's session-management pattern directly
    def __init__(self):
        self.conversation_history = []
        self.long_term_facts = {}      # genuinely a simple key-value memory, recap Cloud file 05's DynamoDB discussion

    def remember(self, key, value):
        self.long_term_facts[key] = value

    def recall_relevant(self, query):
        # Genuinely could use file 05's vector search here too, retrieving
        # relevant PAST facts the same way file 06 retrieves relevant documents
        return {k: v for k, v in self.long_term_facts.items() if k in query}
```

Genuinely worth recognizing agent "memory" as directly recapping two earlier concepts combined — Flask file 10's session state (short-term, within a single interaction) and file 05/06's retrieval mechanism (longer-term, searchable memory of past facts/interactions) — genuinely no fundamentally new concept, a combination of patterns already deeply understood.

## The honest, real limitations of current agents — worth knowing directly

```
Genuinely real, honest limitations: agents can genuinely get STUCK in
loops, choose the WRONG tool, or hallucinate tool inputs that don't
actually match a tool's real schema (recap file 01's hallucination
discussion directly, now applied to ACTIONS, not just text) -- reliability
is genuinely still an active, real area of ongoing improvement in the field,
not a fully solved problem
```

## Try this

```python
# Build a complete agent with: a SAFE calculator tool (using the ast-based
# approach, not eval()), a RAG-based knowledge search tool (recap file 06's
# system), and a max_iterations safety limit
# Test it with a query requiring BOTH tools in sequence (e.g. "What's our
# return policy period, and if I bought something 25 days ago, how many
# days do I have left?") and trace through EVERY iteration of the loop,
# confirming the model correctly uses both tools and combines their
# results into a coherent final answer
# Deliberately give it a query with NO relevant tool available and confirm
# it gives an honest "I don't have a tool for that" response rather than
# fabricating an answer
```

Tracing through every single loop iteration manually is genuinely the best way to build real, concrete understanding of how the THINK→ACT→OBSERVE cycle actually unfolds, rather than treating the agent as an opaque black box.

## What's next
File 10 — Multi-Agent Systems & Frameworks, extending this file's single-agent loop into systems with MULTIPLE cooperating agents, and covering the popular frameworks (LangChain, LangGraph) that formalize these patterns.
