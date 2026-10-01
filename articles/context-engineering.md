# What Is Context Engineering? What Comes After a Well-Written Prompt?

By **Cheng-Ho Wu** · October 1, 2026

[繁體中文原文 / Original article](https://chwu.site/blog/context-engineering/) · [Back to profile](../README.md)

As AI moves from answering a single question to completing multi-step tasks, we need to design more than the prompt: data retrieval, tool interfaces, memory, state, and the context supplied to the model at each step all matter.

![Context engineering selects the information a model needs from the data available to the system](https://chwu.site/assets/posts/context-engineering-hero.png)

When working with AI, we are already familiar with a few prompting principles: describe the task clearly, provide the necessary background, specify the output format, and add examples when needed.

But once AI starts handling real work, clear instructions alone leave some questions unanswered.

To answer a product question, it needs to find the applicable documentation. To modify code, it needs to read the relevant files and project rules. To resume a long-running task, it needs to know which steps are complete and which issues remain unresolved.

Where does this information come from? What should be provided now, and what can be looked up when needed? When documents change, tools return results, or work progresses, what should go into the next model call?

**Context engineering is the work of preparing, organizing, and continually updating the information a model needs to carry out a task.**

Prompt engineering remains an important part of this work. As a task grows from a single response into multiple steps, the design also needs to cover data retrieval, tool interfaces, memory, and task state.

## The Model Receives More Than Your Question

A single model call may include task instructions, conversation history, retrieved documents, tool descriptions, and the results of the previous step. Together, these form the context for that call.

For example, when a customer support AI receives the question, “Can I return this order?”, the instruction “Answer according to company policy” is not enough. It may also need:

- The order date
- The product category
- The applicable region
- The relevant version of the return policy

There is an important distinction here: **the data available to the system is often different from the data the model receives in the current call.**

An order stored in a database still needs to be queried. A policy in a knowledge base still needs to be retrieved. Even if previous conversations have been saved, they may not be included in full in the next call.

Context engineering therefore needs to manage two things:

1. How information is stored.
2. How the necessary information is selected at each step.

This applies to both single-turn questions and multi-step AI agents. The former may only need a well-chosen combination of instructions and data; the latter also needs context to be updated repeatedly as work progresses.

This article focuses on information design during task execution. Model training and fine-tuning take a different approach by changing the model's parameters.

## What Techniques Does Context Engineering Include?

Context engineering encompasses several techniques for retrieving, organizing, storing, and using information. Not every system needs all of them.

| Design task | Techniques and methods | Purpose |
| --- | --- | --- |
| Design instructions and examples | Prompt templates, dynamic instructions, few-shot prompting, example retrieval | Provide task-specific rules and select relevant examples |
| Retrieve external knowledge | RAG, hybrid retrieval, reranking, metadata filtering | Find relevant content and filter it by version, date, or scope of applicability |
| Manage conversation and task state | Conversation history selection, structured state, checkpoints | Preserve progress and confirmed facts, and decide which state to pass to the next model call |
| Build memory across tasks | Memory extraction, persistent storage, memory retrieval, updates, invalidation | Save information worth carrying forward, retrieve it when needed, and handle outdated or conflicting content |
| Design tool interfaces | Tool calling, tool schemas, dynamic tool selection, result filtering, pagination | Explain how to use tools and return enough information to support the next step |
| Manage context capacity | Token budgets, history trimming, context compaction, external storage, on-demand reading | Limit the input for each call while preserving necessary information and ways to retrieve the original material |
| Separate context for different tasks | Subagents, separate sessions, state partitioning, execution environment isolation | Let separate tasks handle their own details and exchange only the results they need |

Some of these are general engineering methods, and their names may vary across frameworks. This table maps the scope of the field; it is not a unified standard.

These methods are often combined. Retrieval may first identify candidate documents, reranking may select the most relevant passages, and a token budget may then limit how much content enters the current call. Tokens are the units a model uses to process content; they do not correspond directly to word or character counts.

How these techniques work together depends on the decision that needs to be made at that step.

## If the Data Exists, Why Is the Answer Still Wrong?

Return to the example of a return request. A customer support AI might answer incorrectly for several different reasons:

- The required policy document was never added to the system.
- The document exists, but the search did not find it.
- The search found a relevant document, but selected a version that does not apply.
- The retrieved content was correct, but important conditions were removed when it was condensed.
- All the conditions were provided, but the model still interpreted them incorrectly.

These failures require different fixes. Adding data cannot solve every search problem, and improving search does not guarantee correct interpretation.

For example, a document excerpt might say, “Returns are accepted within 30 days of purchase,” while the full policy also contains product-category and regional restrictions. Keeping only that sentence leaves the model without the information it needs to recognize exceptions.

Metadata can help preserve document dates, versions, and scope, and those fields can be used to filter retrieval results. If a question includes an exact product code, hybrid retrieval can combine keyword matching with semantic similarity. If too many candidates remain, reranking can narrow the selection.

When troubleshooting, inspect each stage of this path:

> **Data available to the system → Retrieved data → Content actually supplied to the model → Final answer**

![A troubleshooting path from system data through retrieved data and model context to the final answer](https://chwu.site/assets/posts/context-engineering-evidence-pipeline.png)

The first question is whether the evidence needed to support a correct answer actually made it into the current input.

## Tool Design Also Changes What the Model Sees

Tools do more than give AI additional capabilities. They also determine how it obtains new information.

A model typically uses a tool's name, description, and parameter definitions to decide when to call it. The surrounding system executes the tool and returns its results to the model, providing evidence for the next decision.

For example, if a tool returns an entire address book when looking for one contact, the model has to process a large amount of irrelevant information. A tool that searches by name and returns only the necessary fields provides information more closely matched to the task.

But brevity cannot be measured by word count alone. If a later query requires a contact ID, that ID should not be removed simply because it is hard to read. If a tool returns only part of a result set, the model should also know that more pages are available.

A tool interface needs to make three things clear:

### 1. When to Use It

Explain its purpose and the situations in which it applies.

### 2. How to Call It

Define the required parameters, formats, and constraints.

### 3. How to Interpret the Results

Explain what the content means, whether it is complete, and whether further queries are possible.

When many tools are available, the system can also choose which ones to expose at each stage. Checking progress and submitting changes are different tasks; the model does not necessarily need to see every available operation in every call.

## What Should Be Supplied Up Front, and What Should Be Read on Demand?

If there is little data and it will definitely be needed, providing it directly is usually simplest. When the data is extensive, or requirements change as the task progresses, on-demand reading becomes worth considering.

Coding agents illustrate this distinction well. Project rules can be supplied up front, while the agent searches for specific files and code sections based on the problem at hand.

Large search results can also be saved in external files, with relevant sections read later. The original data remains available without having to include all of it in every call.

| Approach | Best suited to | Main trade-off |
| --- | --- | --- |
| Supply up front | Small amounts of data that will definitely be used | Irrelevant content may continue to occupy context |
| Read on demand | Large amounts of data, with the next step's needs still uncertain | Adds lookup time and the risk of missing a necessary lookup |
| Combine both | Fixed rules alongside dynamic data | Requires deciding what to preload and what to retrieve |

The key to on-demand reading is preserving clues that make information discoverable, such as file paths, document names, or query criteria. Moving content out of context without leaving a way to retrieve it makes that content difficult to use later.

## Long Tasks Need Both Context Management and Handoffs

As conversations and tool calls accumulate, context grows. A larger context window can hold more content, but it does not automatically resolve conflicting versions, duplicate information, or outdated decisions.

Different problems call for different approaches.

### Compact History While Preserving the Reasoning Needed Later

Suppose a coding agent has tried several fixes. The next call may need the current approach, the reasons for choosing it, unresolved issues, and test results, rather than every repeated output.

If a summary retains only “Use approach B” but drops the reason approach A failed, later work may repeat the same mistake. The quality of compaction depends on whether necessary information survives, not just how much shorter the context becomes.

### Save Task State So Work Can Resume

Progress, completed steps, and remaining work can be stored as structured state and restored from checkpoints when work resumes. Preferences or knowledge worth preserving across tasks can be saved separately as memory and retrieved when needed.

This also illustrates the difference between state and memory. State often describes where a particular task stands; memory may remain useful in different tasks in the future.

Both need to be updated. If outdated goals or incorrect notes are repeatedly read, they can continue to distort later decisions.

### Separate Work That Can Be Completed Independently

In a research task, different subagents can investigate separate questions and return the necessary results. This reduces the detail any one agent has to handle, but also adds cost and handoff risks.

If subtasks are tightly coupled and need to exchange large amounts of information frequently, separating them may not be worthwhile. Isolating execution environments does not automatically isolate context, either; the system still needs to decide which results to return to the model.

## How Do You Know Whether Context Design Has Improved?

Start with a set of tasks that have clear evaluation criteria. Keep the questions, retrieved data, model inputs, tool results, and final answers, then compare them before and after a change.

For return-policy questions, you might check:

- Did the system use the applicable policy version?
- Did it preserve product and regional restrictions?
- Did it ask for clarification or explicitly acknowledge missing information?
- Could it identify the evidence supporting its answer?
- Did it add unnecessary queries, waiting time, or resource usage?

Keep the model and other settings as consistent as possible, and change one strategy at a time to make the effect easier to assess.

For example, shortening tool results might reduce usage but remove an identifier the model needs for verification. Lower cost alone would not make that change a success. Context design needs to be evaluated against both task outcomes and the resources required to achieve them.

## After Writing a Clear Prompt, What Should You Ask Next?

Once the prompt is clear, the next question worth asking is:

> **What does the model need to know at this step, and what has the system actually provided?**

Examining the gap between the two helps you decide whether to add data, improve retrieval, adjust tools, or reorganize task state.

## References

This article draws on public articles and official documentation. General scenarios have been adapted for explanation, and product examples are described as reported by their sources. Their effectiveness has not been independently tested for this article.

- [Anthropic: Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
- [LangChain: Context Engineering](https://www.langchain.com/blog/context-engineering-for-agents)
- [Anthropic: Writing effective tools for agents — with agents](https://www.anthropic.com/engineering/writing-tools-for-agents)
- [LangChain documentation](https://docs.langchain.com/)
- [LangGraph documentation](https://docs.langchain.com/oss/python/langgraph/overview)
- [Google Agent Development Kit](https://google.github.io/adk-docs/)
- [LlamaIndex documentation](https://docs.llamaindex.ai/)
- [Model Context Protocol documentation](https://modelcontextprotocol.io/)

---

English translation of [the original Traditional Chinese article](https://chwu.site/blog/context-engineering/) by Cheng-Ho Wu. Illustrations are retained from the original article.

This article was prepared in collaboration with AI. The original article was edited and reviewed by the author and is shared for learning and discussion. Please include a link to the original when citing it.
