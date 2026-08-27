# Key Takeaways: Customer Support Chatbot Optimization & Evaluation Journey

This optimization journey serves as a textbook example of **Prompt Engineering** and **Agent Evaluation** in practice. By systematically iterating from an initial score of 0.81 to 0.88, and ultimately achieving a perfect **1.00 score**, the key was transitioning from simple prompt tweaking to diagnosing the model's underlying cognitive biases across **semantic understanding**, **output formatting**, and **eval dataset coverage**.

AgentCore has no condition nodes — say so explicitly.
*Note on Architecture Diagram:* 
Per instructor guidance, the architecture flow was visualized using AWS Bedrock Flow Builder (containing a condition node to illustrate logical routing branching), while the underlying execution logic follows the AgentCore implementation using single-prompt classification without standalone condition nodes in code.

---

## 1. Problem Diagnosis & Key Resolutions

### A. Intent Classification Ambiguity (Score: 0.81 → 0.88)
* **Root Cause**: The model took an overly literal or conservative approach when encountering edge-case inputs (e.g., student discounts or promotional code requests), failing to route them accurately between FAQ and general queries, default-categorizing them as `OTHER`.
* **Resolution**: Implemented explicit **Forced Routing rules** with clear boundary definitions for each classification category, removing model hesitation and eliminating routing errors.

### B. Rigid Feature Extraction (Score: 0.81 → 0.88)
* **Root Cause**: When checking required fields for bug reports (such as `stepsToReproduce`), the model relied on strict literal pattern matching. It expected structured bullet points and overlooked natural conversational descriptions (e.g., *"I open the app, tap profile, and it crashes"*) that actually contained the necessary reproduction steps.
* **Resolution**: Relaxed the semantic tolerance for information extraction, guiding the model toward **semantic comprehension** rather than rigid pattern matching.

### C. Leaking Reasoning Steps & Internal Formatting Noise (Score: 0.88 → 1.00)
* **Root Cause**: The model generated internal thinking tags (`<thinking>...</thinking>`) or category prefixes (e.g., `OTHER REQUESTS.`), which degraded evaluator metrics.
* **Resolution**: Placed top-level **negative constraints (`CRITICAL INSTRUCTIONS`)** at the very beginning of the system prompt to explicitly restrict thinking outputs and internal labels. Re-ran the automated build script (`create_harness.py`) and regenerated datasets (`run-9`) to ensure the updated prompts were properly deployed.

### D. Insufficient Evaluation Dataset Coverage (Eval Dataset Design)
* **Root Cause**: A small or uniform set of test prompts risks **overfitting**, creating a false sense of accuracy while masking vulnerabilities in production.
* **Resolution**: Expanded the evaluation benchmark with diverse phrasing, edge-case scenarios, incomplete user inputs, and adversarial prompt injections to ensure high statistical confidence.

---

## 2. Core Insights & Lessons Learned

1. **Vague Instructions Do Not Work**: Telling a model to "think carefully" when it acts overly conservative or literal yields no real improvement. Effective prompting requires **explicit boundaries and in-context criteria**.
2. **Engineering Pipeline Matters As Much As Prompts**: Even a perfect prompt will not produce results if the build and deployment pipeline (e.g., regenerating dataset versions and running AWS Bedrock Evaluation Jobs) is not properly executed.
3. **Dataset Diversity Dictates Production Stability**: Increasing prompt quantity and variety is critical for identifying blind spots. A robust evaluation dataset bridges the gap between a 100% score in testing and reliable performance in production.
4. **Shift to Systematic Analysis**: Shifting from trial-and-error prompt tweaks to systematic diagnostics (evaluating classification, extraction, formatting, and test coverage) is key to building enterprise-grade LLM agents.
