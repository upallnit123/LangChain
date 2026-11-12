# LangChain 1.x overview

LangChain 1.x is organized around agents. The main package, `langchain`, exposes three entry points you use every day: `init_chat_model` for creating a chat model from a provider string, `create_agent` for building a tool-calling agent, and the `@tool` decorator for turning a Python function into a tool the model can call.

`langchain-core` holds the contracts: message types (SystemMessage, HumanMessage, AIMessage, ToolMessage), prompt templates, the Runnable interface, and output parsers. Every Runnable supports `invoke`, `stream`, and `batch`, plus async variants `ainvoke`, `astream`, and `abatch`.

`langgraph` is the runtime underneath `create_agent`. It models an agent as a graph of nodes and edges over a typed state. Graphs can be compiled with a checkpointer, which persists state between calls and makes human-in-the-loop interrupts possible.

Provider integrations live in separate packages: `langchain-openai`, `langchain-anthropic`, `langchain-aws`, `langchain-google-genai`, and `langchain-ollama`. The legacy chains from version 0.x moved to `langchain-classic`.

## Structured output

Call `model.with_structured_output(Schema)` with a Pydantic model to get a validated object instead of a string. Field descriptions are sent to the model as part of the schema, so they act as instructions. Use `Literal` types for enumerations and `Field(ge=, le=)` for numeric bounds.

## Middleware

`create_agent` accepts a `middleware` list. Built-in middleware includes `SummarizationMiddleware` (compresses old messages when the context grows), `HumanInTheLoopMiddleware` (pauses before selected tool calls), and `PIIMiddleware` (detects and redacts personal data). Custom middleware can run before or after the model call and can end the run early by returning `{"jump_to": "end"}`.
