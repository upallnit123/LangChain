# LangGraph basics

A LangGraph graph is built with `StateGraph(State)`, where `State` is a `TypedDict`. Each key may carry a reducer via `Annotated`; the `add_messages` reducer appends messages instead of overwriting them. Without a reducer, a node's returned value replaces the key.

Nodes are plain functions that take the state and return a partial update. Edges connect nodes; `START` and `END` are the special entry and exit points. `add_conditional_edges(source, router, mapping)` lets a router function choose the next node from the state.

A node can also return a `Command(update=..., goto=...)` to update state and choose the next node in one step.

## Persistence

Compiling with `checkpointer=InMemorySaver()` stores a checkpoint after every step. Calls that share a `thread_id` in `config["configurable"]` continue the same conversation. `get_state(config)` returns the current snapshot, `get_state_history(config)` lists every checkpoint, and `update_state` edits a checkpoint so you can replay from it. For production use `SqliteSaver` or `PostgresSaver`.

## Human in the loop

Inside a node, `interrupt(payload)` pauses the graph and surfaces the payload under `result["__interrupt__"]`. The caller resumes with `graph.invoke(Command(resume=value), config)`; the value becomes the return value of `interrupt`. Interrupts require a checkpointer and a thread_id.

## Streaming

`graph.stream(input, config, stream_mode=...)` yields events. Mode `values` yields the full state after each step, `updates` yields only each node's update, `messages` yields LLM tokens as they are produced, and `custom` yields whatever nodes write with `get_stream_writer()`.

## Fan-out

Return a list of `Send("worker", payload)` from a router to run a worker node once per payload in parallel. Collect results with a list reducer such as `operator.add`.
