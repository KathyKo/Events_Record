### **Building Deep Agents** by Sam Witteveen 
[Slides](Oct\ML SG - Oct\Building DeepAgents\DeepAgents MLSG 15-10-2025.pdf)
---

### **Overall Summary**

This document synthesizes an analysis of the evolution from simple AI agents ("Agents 1.0") to a sophisticated, robust new architecture known as "Deep Agents" or "Agents 2.0." This shift is driven by the failure of basic agents to handle complex, long-running tasks. The most critical takeaways are:

1.  **The Failure of Basic Agents:** "Agents 1.0" operate in a simple loop where their entire "brain" and memory exist within the LLM's context window. They fail at multi-step tasks (50+) due to **Context Overflow** (tool outputs pushing instructions out of memory) and **Loss of Goal**.  
2.  **The "Myth of the Better Model":** A prevalent myth is that a more powerful base model (e.g., GPT-5) will automatically solve these engineering challenges. The presenter argues this is false; we are "nowhere near" a model with full autonomy, and robust engineering is required.    
3.  **The Rise of "Flow Engineering":** As a solution, "Agents 1.5" introduced "Flow Engineering," which trades "full agency" for reliability. This is an "on-rails" approach that uses tightly controlled, graph-based loops, which is how most reliable production agents currently work.    
4.  **Deep Agents (Agents 2.0):** This new architecture handles complex, "long-horizon" tasks by making a fundamental shift. It **decouples planning from execution**  , delegates work to **specialized sub-agents**  , and manages a **persistent state external** to the context window.    
5.  **The Five Architectural Pillars:** Deep Agents are built on five key components:
    * **Explicit Planning:** Using a tool to create and track a "to-do list".  
    * **Sub-Agents:** An orchestrator delegates tasks to specialized agents, each with its own clean context (a concept called "Context Quarantine").  
    * **Persistent Memory (File System):** Offloading memory to a virtual file system (using `read_file`, `write_file`) to avoid "stuffing the context window".  
    * **Better Context Engineering:** Meticulously engineering behavior via highly detailed system prompts that can be thousands of tokens long.  
    * **Verification Systems:** A crucial fifth pillar that checks agent outputs to keep the task on the correct trajectory and "resets" the probability of success.

The architecture, implemented in libraries like **LangGraph**  , is designed to solve the core problem of **"Context Rot"**—the degradation of an agent's performance as its context window becomes polluted with stale or irrelevant information.

---

### **1. The Failure of Basic Agents (Agents 1.0)**

The initial "hype year" of agents was built on a simple concept defined as "Agents 1.0".

* **Definition:** An AI system operating in a simple, stateless loop. The process follows a basic pattern: User Prompt -> LLM Reasons (e.g., "I need a tool") -> Tool Call -> Observation (Tool Return) -> LLM Answer -> Repeat.  
* **The Core Limitation:** The agent's entire "brain" and memory exist *only* within the LLM's context window. This was a critical problem when context windows were small (e.g., 4k tokens), but the presenter argues that even with million-token windows, it remains an inefficient and problematic design.  
* **Why They Fail:** Basic agents are effective for simple, 5-15 step tasks but consistently fail on complex, 50-500 step projects. The failures are twofold:
    * **Context Overflow:** The conversation history fills up with raw tool outputs (like HTML or data), eventually pushing the original instructions and high-level goals out of the limited context window. 
    * **Loss of Goal:** Amidst the noise of many intermediate steps, the agent "forgets" the original objective.  
    * **No Recovery Mechanism:** If the agent goes down a wrong path, it has no ability to stop, backtrack, and try a new approach.  

### **2. The Evolution of Agentic Design (Agents 1.5)**

The failure of basic agents led to a debate between relying on better models versus better engineering.

* **The "Myth of the Better Model":** The presentation directly confronts the idea, associated with figures like Sam Altman, that a "better model" will eventually solve these agentic failures. The presenter argues this is a "myth" and that we are "nowhere near" a model capable of full autonomy without careful engineering.
* **Flows vs. Agency:** This realization led to "Agents 1.5," which centers on the concept of "Flows vs. Agency". Developers must choose between:
    * **High Agency:** Full autonomy, where the agent can choose any tool at any time. This offers flexibility but is highly unreliable, as "more can go wrong".  
    * **Low Agency (Flow Engineering):** A "tightly controlled" system where the agent's choices are limited. This is an **"on-rails"** approach, often visualized as a graph where an agent can only move to specific, predetermined nodes.  
* **Conclusion:** The "on-rails" Flow Engineering model is "much more reliable". The presenter notes that most agents "in production nowadays that are actually performing quite well" use this approach.  

### **3. Defining Deep Agents (Agents 2.0)**

"Deep Agents" (or "Agents 2.0") are the next architectural shift, designed to combine the power of autonomous agents with the reliability of flow engineering to tackle complex, "long-horizon" tasks.  

This architecture is defined by a set of key characteristics:

* It decouples planning from execution.  
* It manages a persistent state or memory *external* to the LLM's context window.  
* It delegates work to specialized sub-agents.  
* It is designed to solve problems over longer time runs (minutes or hours, not seconds).  

### **4. The Five Pillars of Deep Agent Architecture**

The presenter identifies five key architectural pillars that enable Deep Agents to function.  

**1. Explicit Planning**
This pillar moves planning from an implicit "chain of thought" step to an explicit, managed process.  
* **Mechanism:** The agent uses a planning tool (e.g., `write_to_dos`) to create and maintain an actionable to-do list within its state.  
* **Impact:** This list is reviewed and updated between steps, tracking tasks as "pending," "in_progress," or "completed". This provides a robust **recovery mechanism**: if a step fails (e.g., a weather API returns an error), the agent can update its plan to try a new approach (e.g., use a search engine) instead of just retrying the failed step.  

**2. Sub-Agents and Context Quarantine**
Instead of one "jack-of-all-trades" agent, Deep Agents use an "Orchestrator" to delegate work to a team of specialized sub-agents (e.g., "Researcher," "Coder," "File Agent").  
* **Mechanism:** This is also known as the "agent as a tool" pattern. When a task is delegated, the sub-agent is given a fresh, clean context with *only* the specific instructions it needs. This is referred to as **"Context Quarantine"**.  
* **Impact:** This prevents the main orchestrator's context from being "polluted" with low-level details. The sub-agent can run multiple tool calls and retries on its own, returning *only the final, synthesized answer* to the orchestrator. This also makes agents highly reusable (e.g., a good RAG sub-agent can be used in many projects).

**3. Persistent Memory (The File System)**
This pillar addresses the "biggest mistake" in agent design: "stuffing the context window". Deep Agents shift from "remembering everything" to "knowing where to find it".  
* **Mechanism:** Agents are given tools to interact with a virtual file system (e.g., `read_file`, `write_file`, `ls`). Intermediate results, raw data, and research notes are "offloaded" to this external memory.  
* **Impact:** The context window is kept clean and focused on the immediate task. If an orchestrator needs a detail from a 50-page report, it doesn't read the whole report into context; it reads the summary from its file system, and can load the full file only if necessary.  

**4. Advanced Context Engineering**
A Deep Agent's behavior is not emergent; it is "meticulously engineered" through its system prompt.  
* **Mechanism:** These agents use highly detailed system prompts that act as an "operating manual". These prompts are often very long, ranging from 2,000 to 20,000 tokens.  
* **Impact:** The prompt defines protocols for planning, rules for spawning sub-agents, standards for file naming, and strict output formats. This detailed instruction set is the foundation that guides the agent.  

**5. Verification Systems**
This is the presenter's fifth proposed pillar, which he notes is critical but less discussed publicly.  
* **Mechanism:** Verification involves checking the agent's outputs at different steps to ensure they are high-quality and aligned with the plan. This can be done with code compilers, human-in-the-loop checks, or other checking systems.  
* **Impact:** This system is essential for long-running tasks. The presenter explains the "Agent Trajectory" problem: over many autonomous steps, the probability of success decays (e.g., 0.95 -> 0.90 -> 0.81 -> ... -> 0.49) . A verifier acts as a "reset," catching errors and fixing the trajectory, bringing the success chance back up.  

### **5. The Goal: Solving "Context Rot" in Long-Horizon Tasks**

Together, these five pillars are designed to solve the two fundamental problems of long-running agents: goal loss and context pollution.

* **Context Rot:** This is the primary enemy of long-horizon agents. It is the "gradual drift" where the agent's context window "gets polluted" with stale, contradicted, or irrelevant facts, making it "no longer able to attend" to the correct information. This tends to happen as context grows (e.g., 50k-100k tokens).
* **The Deep Agent Solution:**
    * **Persistent Memory** prevents context rot by offloading information.  
    * **Sub-Agents** prevent context pollution by isolating complex work.  
    * **Explicit Planning** prevents goal loss by keeping a constant, high-level to-do list.  
    * **The Plan** enables recovery from failure, allowing the agent to try a new approach.  

### **6. Real-World Examples and Implementation**

This architecture is not just theoretical. The presentation cites several key examples:

* **Magentic-One (Microsoft):** A "fundamental paper" from 2024 that introduced the "Task Ledger" (an explicit planning tool) and used sub-agents like a Coder and WebSurfer .
* **AI Co-Scientist (Google):** A "real wake-up call" for Google. This agent acts like a PhD student, performing literature reviews and generating novel hypotheses. The resulting theses were judged by Stanford professors as being PhD-worthy.  
* **ClaudeCode (Anthropic):** A prime example of a deep agent, recently renamed the "Claude Agent SDK".  
* **Implementation (LangGraph):** This Deep Agent architecture is made accessible via the `deepagents` open-source library, which is built on **LangGraph**. LangGraph serves as the runtime, representing agents as graphs, which is ideal for the "on-rails" control flow required by Flow Engineering.