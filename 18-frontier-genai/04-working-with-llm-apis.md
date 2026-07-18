# 04 — Working with LLM APIs

Recap file 03's closing note — moving from prompt DESIGN into the actual technical implementation, genuinely the practical foundation for everything built in the rest of this folder.

## Basic API structure — recap Hugging Face file 19's chat-model discussion directly

```python
import anthropic      # genuinely similar structure across providers -- openai's client is structurally near-identical

client = anthropic.Anthropic(api_key=os.environ.get("ANTHROPIC_API_KEY"))      # recap Cloud file 02's
                                                                                     # credential-safety discipline directly

response = client.messages.create(
    model="claude-sonnet-4-5",
    max_tokens=1024,
    messages=[
        {"role": "user", "content": "Explain the attention mechanism briefly"}
    ]
)
print(response.content[0].text)
```

Genuinely worth recognizing the `messages` list structure as directly recapping Hugging Face file 19's chat-model conversation format, and file 02's system/user prompt distinction — this is genuinely the same conceptual structure, now the actual real API call.

## API keys and security — recap Cloud file 02's IAM/credential discipline directly

```python
# NEVER hardcode this -- recap the ENTIRE roadmap's credential-safety
# discipline (Flask files 08/10/11, MLOps file 02, Cloud file 02) directly
api_key = os.environ.get("ANTHROPIC_API_KEY")      # genuinely the SAME environment-variable pattern, one more time
```

Genuinely worth restating this ONE more time, given genuinely higher real stakes here — recap Cloud file 12's LLM-cost-tracking discussion directly: an exposed LLM API key can genuinely result in significant, real, unexpected charges if used maliciously, exactly the same risk category as the exposed AWS credentials warning from Cloud file 01, now for a genuinely different provider.

## Multi-turn conversations — genuinely maintaining context across turns

```python
conversation = [
    {"role": "user", "content": "What is a neural network?"},
]

response1 = client.messages.create(model="claude-sonnet-4-5", max_tokens=500, messages=conversation)
conversation.append({"role": "assistant", "content": response1.content[0].text})

conversation.append({"role": "user", "content": "How does that relate to the attention mechanism?"})
response2 = client.messages.create(model="claude-sonnet-4-5", max_tokens=500, messages=conversation)
```

Genuinely important, real practical detail worth flagging directly: the model itself has NO memory between API calls — recap the general "HTTP is stateless" discussion from Flask file 10's session explanation directly, genuinely the SAME underlying principle: the ENTIRE conversation history must be RE-SENT with every single API call for the model to have any context about prior turns. This has genuinely real, direct cost implications (recap file 01's token-pricing discussion) — longer conversations cost more per turn, since the growing history gets resent every time.

## Streaming responses — genuinely important for user experience

```python
with client.messages.stream(
    model="claude-sonnet-4-5", max_tokens=1024,
    messages=[{"role": "user", "content": "Write a short story about a robot"}]
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)      # genuinely prints incrementally, AS tokens arrive
```

Genuinely worth connecting this directly to Flask file 05's AJAX/loading-indicator discussion — recap that file's "responsive user experience" theme: streaming lets a user see the response being GENERATED progressively (word by word), rather than waiting for the ENTIRE response to complete before seeing anything — genuinely the same underlying "don't block the user unnecessarily" UX principle, now applied to LLM generation latency specifically.

## Function calling / Tool use — genuinely the foundational mechanism for file 09's agents

```python
tools = [
    {
        "name": "get_weather",
        "description": "Get the current weather for a location",
        "input_schema": {
            "type": "object",
            "properties": {"location": {"type": "string", "description": "City name"}},
            "required": ["location"]
        }
    }
]

response = client.messages.create(
    model="claude-sonnet-4-5", max_tokens=1024,
    tools=tools,
    messages=[{"role": "user", "content": "What's the weather in Mumbai?"}]
)

# The model responds with a TOOL USE request, NOT a direct text answer
if response.stop_reason == "tool_use":
    tool_call = response.content[-1]
    location = tool_call.input["location"]
    weather_result = actual_weather_api_call(location)      # genuinely YOUR code executes the real function

    # Send the result BACK to the model to get a final, natural-language answer
    conversation.append({"role": "assistant", "content": response.content})
    conversation.append({
        "role": "user",
        "content": [{"type": "tool_result", "tool_use_id": tool_call.id, "content": str(weather_result)}]
    })
    final_response = client.messages.create(model="claude-sonnet-4-5", max_tokens=1024, messages=conversation)
```

Genuinely worth understanding this deeply, as the direct technical foundation for file 09's full agent treatment — recap file 03's ReAct pattern directly: `tools` defines a SCHEMA (genuinely similar to Flask file 07's request-validation schema, or Hugging Face file 02's structured tokenizer input) describing what functions are AVAILABLE, and the model decides WHEN and HOW to call them, but the ACTUAL EXECUTION happens in YOUR code, not inside the model itself — the model genuinely only REQUESTS a tool call; your application is responsible for running it and reporting the result back.

## Structured output — recap file 02's JSON-format-specification discussion, now enforced

```python
response = client.messages.create(
    model="claude-sonnet-4-5", max_tokens=1024,
    messages=[{"role": "user", "content": "Extract name and age from: John is 34 years old"}],
    tools=[{
        "name": "extract_info",
        "description": "Extract structured information",
        "input_schema": {
            "type": "object",
            "properties": {"name": {"type": "string"}, "age": {"type": "integer"}},
            "required": ["name", "age"]
        }
    }],
    tool_choice={"type": "tool", "name": "extract_info"}      # genuinely FORCES the model to use this schema
)
```

Genuinely worth recognizing this as a real, practical technique BEYOND agentic tool use — using the tool/function-calling mechanism to FORCE genuinely structured, reliably-parseable output, directly recapping file 02's "specify output format" discussion but with an actual SCHEMA-ENFORCED guarantee, rather than just hoping the model follows a text-based format instruction.

## Error handling and retries — recap Flask file 06's error-handling discipline directly

```python
import time

def call_llm_with_retry(prompt, max_retries=3):
    for attempt in range(max_retries):
        try:
            return client.messages.create(model="claude-sonnet-4-5", max_tokens=1024,
                                             messages=[{"role": "user", "content": prompt}])
        except anthropic.RateLimitError:      # genuinely a real, common error type
            wait_time = 2 ** attempt      # recap the general exponential-backoff pattern
            time.sleep(wait_time)
        except anthropic.APIError as e:
            app.logger.error(f"API call failed: {e}")      # recap Flask file 06's logging discipline directly
            raise
```

Genuinely worth recognizing this as directly recapping Flask file 06's "never let an unhandled exception crash the app" discipline — LLM APIs genuinely have real failure modes (rate limits, transient network errors, occasional service outages) that a genuinely robust application needs to handle gracefully, exactly the same reliability discipline from that whole folder.

## Rate limits and cost tracking — recap file 01/Cloud file 10's cost discussion directly

```python
def call_llm_with_logging(prompt):
    response = client.messages.create(model="claude-sonnet-4-5", max_tokens=1024,
                                         messages=[{"role": "user", "content": prompt}])
    # recap MLOps file 09's prediction-logging discussion directly
    log_usage(input_tokens=response.usage.input_tokens, output_tokens=response.usage.output_tokens)
    return response
```

## Try this

```python
# Build a small Python script that: (1) makes a basic single-turn API call,
# (2) maintains a multi-turn conversation across 3 exchanges, correctly
# re-sending history each time, (3) uses streaming to print a response
# incrementally, and (4) defines ONE simple tool (e.g. a basic calculator
# function) and correctly handles the full tool-use round trip: model
# requests the tool, your code executes it, result goes back to the model,
# model gives a final answer
# Wrap all API calls in the retry/error-handling pattern from this file
```

Building all four patterns (basic call, multi-turn, streaming, tool use) in one script is genuinely the complete technical foundation the rest of this folder builds directly on top of.

## What's next
File 05 — Vector Databases & Embeddings Deep Dive, covering the retrieval infrastructure that directly addresses file 01's hallucination/knowledge-cutoff limitations — the foundation for file 06's RAG systems.
