# Documentation-on-different-langraph-use-case-

1. Use case finance - used async processing
2. Use case feedback system used asnyc while not was compulsory
3. Travel chatbot


<img width="694" height="453" alt="Screenshot 2025-12-23 at 7 48 59 PM" src="https://github.com/user-attachments/assets/41e4ea3f-03aa-4271-bc5b-e2fb7d797fc6" />


A. async
**
Why async is used in feedback pipeline only optionally**

- In feedback pipeline, all processing is row-by-row, synchronous in nature.

- Agents do not need to pause, stream partial results, or wait for user input.

- We could use async to parallelize rows or LLM calls, but the simple sequential for loop is enough for batch offline processing.

That’s why in my suggested runner.py:

async def run_pipeline(csv_path: str, source_type: str):
    graph = build_graph()
    records = csv_reader_agent(csv_path, source_type)

    results = []
    for state in records:
        final_state = await graph.ainvoke(state)
        results.append(final_state)

    return results


Here async is only needed because graph.ainvoke itself is an async call. If the LLM agent were synchronous, we wouldn’t need async at all.

**Why travel chatbot uses async heavily**

Streaming responses: partial text sent to user in real-time → must await LLM streams.

Interrupt/resume: user may ask to cancel or change topic → async allows “pause/resume” at any point.

Dynamic multi-agent handoff: decision to call a different agent depends on runtime user input → async ensures smooth switching without blocking.

Memory state: conversation history needs to be updated incrementally, often while waiting for LLM → async avoids blocking UI.

Example (travel bot):

async for event in travel_graph.astream(messages_state):
    st.write(event.partial_response)


astream allows streaming partial outputs.

async for is needed to incrementally process multi-turn messages.

4️⃣ Key takeaway

Feedback pipeline = offline batch, row-independent → async optional, sequential is fine.

Travel LangGraph = interactive multi-turn chat → async required for streaming, pause/resume, dynamic handoff.

Both designs are correct, just optimized for different problem classes.
