# Reflection

## Design Decision

A key design decision in this project was using the AgentCore Code Interpreter for loyalty discount calculations instead of letting the LLM compute the math directly. LLMs are known to hallucinate arithmetic results, which is unacceptable when dealing with financial calculations like discounts and loyalty points. By offloading the business logic to a sandboxed Python runtime, the LLM's role is limited to deciding when to call the tool and what parameters to pass. It never performs the math itself. The actual calculation runs as deterministic Python code where business rules such as earn rates, tier percentages, and points-to-dollar conversions are explicitly encoded, making them auditable, testable, and reproducible.

## Challenge Encountered

A concrete challenge was parsing memory retrieval responses from AgentCore Memory. The content field returned in inconsistent formats, sometimes as a nested dictionary with a text key and other times as a plain string. The initial implementation assumed a single format, which caused silent failures when the other appeared. This meant customer context from previous sessions would not load, breaking the cross-session recall feature entirely. The fix was adding defensive type-checking to handle both structures before extracting the text, ensuring memories are always parsed correctly regardless of the response format.

## Production Consideration

Scaling this agent to production for a company with hundreds of millions of active customers raises significant cost concerns. The current model, Amazon Nova 2 Lite, is lightweight and cost-efficient per token, but every single invocation triggers multiple tool calls including knowledge base retrieval, Gateway tools, Code Interpreter, and memory operations. At millions of daily invocations, these costs compound quickly. A production strategy would involve benchmarking alternative Bedrock models such as Nova Micro, Nova Pro, or Claude Haiku across the same test scenarios to find the best tradeoff between response quality, tool-calling accuracy, and cost per request. Additionally, a model routing approach could optimize spend by directing simple queries like order status or FAQs to a cheaper model while reserving more capable models for complex multi-tool reasoning tasks like discount calculations. Not every query needs every tool, so selectively skipping unnecessary retrievals would further reduce per-request cost at scale.
