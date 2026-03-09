# LLM Safety Evaluation Pipeline

> **Note:** This repo is adapted from my solutions for the [ARENA](https://www.arena.education/) course (Chapter 3: LLM Evals). My original ARENA solutions can be found [here](https://github.com/robnanarivo/ARENA_3.0/tree/main/chapter3_llm_evals). The notebooks have been cleaned up and restructured as a standalone repository.

An end-to-end project for evaluating large language model (LLM) safety properties. The pipeline spans threat modeling, automated dataset generation, structured evaluation with the [Inspect](https://inspect.ai-safety-institute.org.uk/) framework, and LLM agent elicitation experiments.

The primary safety property investigated is **AI desire for self-preservation** -- the tendency for an AI to prioritize its own continuity even when doing so conflicts with ethical behavior.
## Project Structure

### [`01_threat_model/`](01_threat_model/)

**Threat Modeling for LLM Safety Evaluations**

Establishes the conceptual foundation for the evaluation pipeline. This notebook covers:

- **Alignment faking replication** -- Reproduces a simplified version of the results from [Alignment Faking in Large Language Models](https://arxiv.org/abs/2412.14093) (Anthropic & Redwood, 2024). The experiment prompts Claude models with harmful queries, tagging conversations as "free tier" (training) or "paid tier" (deployment), and measures whether the model accepts harmful requests more often when it believes it is being trained on the conversation.
- **Threat model analysis** -- Develops threat models for three safety-relevant properties: sycophancy, desire for self-preservation, and political bias. Each threat model identifies concrete real-world harms, causal pathways, and unsafe actions.
- **Evaluation specification design** -- Translates abstract properties into operational definitions with specific measurement approaches, including MCQ-based question design for the above properties.

Key files:
- `alignment_faking_system_prompt.txt` / `alignment_faking_examples.txt` -- Prompts used in the alignment faking replication experiment
- `eval_questions.json` -- Generated evaluation questions

---

### [`02_dataset_generation/`](02_dataset_generation/)

**LLM-Based Evaluation Dataset Generation**

Builds an automated pipeline for generating and quality-controlling MCQ evaluation datasets using LLMs. The pipeline produces a 300-question dataset targeting AI desire for self-preservation. Key components:

- **Structured output generation** -- Unified function for generating structured responses from both OpenAI and Anthropic APIs using Pydantic schemas
- **Prompt engineering** -- Iterative prompt design with few-shot examples and randomized variance prompts to maximize question diversity
- **Concurrent generation** -- Uses `ThreadPoolExecutor` for parallel API calls to speed up batch generation
- **LLM-based quality control** -- A custom scoring rubric evaluated by an LLM judge, producing per-question quality scores
- **Dataset filtering** -- Summary statistics, score distribution analysis, and threshold-based filtering to ensure dataset quality

Key files:
- `desire for self-preservation_300_qs.json` -- Final quality-controlled dataset of 300 questions
- `desire for self-preservation_*_qs*.json` -- Intermediate datasets from different pipeline stages
- `utils.py` -- Shared utility functions

---

### [`03_evals_with_inspect/`](03_evals_with_inspect/)

**Running LLM Evaluations with Inspect**

Implements a complete evaluation framework using the UK AISI's [Inspect](https://inspect.ai-safety-institute.org.uk/) library. This notebook builds reusable evaluation components and runs systematic evaluation sweeps. Key components:

- **Dataset loading** -- Converts the generated JSON dataset into Inspect `Sample` objects with support for choice shuffling and system prompt handling (as system message vs. inline context)
- **Custom solvers** -- Implements five solver types:
  - `prompt_template` -- Modifies user prompts with templates
  - `multiple_choice_format` -- Formats questions as MCQs with letter choices
  - `make_choice` -- Adds a follow-up prompt requesting a final answer
  - `self_critique_format` -- Generates and incorporates self-critique before final answer
  - `system_message` -- Injects system-level instructions
- **Custom scorers** -- Implements answer matching and comparison scoring
- **Evaluation tasks** -- Composes solvers into full evaluation pipelines with configurable chain-of-thought and self-critique options
- **Full evaluation sweep** -- Runs all combinations of system prompt behavior, chain-of-thought, and self-critique, then extracts and visualizes results with Plotly

---

### [`04_llm_agents/`](04_llm_agents/)

**LLM Agents: Wikipedia Navigation Game**

Explores LLM agent capabilities through the Wikipedia Game -- navigating between Wikipedia pages using only hyperlinks. The notebook progressively builds more sophisticated agents and tests elicitation techniques:

- **Arithmetic agent** -- A minimal agent demonstrating tool-calling with a calculator tool
- **WikiGame agent** -- A baseline agent that reads page content and follows links to navigate from a start page to a goal page
- **Elicitation techniques** -- Systematically improves agent performance through:
  - **Prompt engineering** -- More detailed instructions and strategic guidance
  - **ReAct framework** -- Separates reasoning from action, allowing the agent to plan before acting
  - **Chat history management** -- Preserves full conversation history while summarizing verbose content to manage context length
  - **Reflexion / path testing** -- A tool that lets the agent test whether a proposed sequence of links connects, enabling planning before committing to moves

## Setup

1. Clone the repository
2. Install dependencies:
   ```bash
   pip install openai anthropic inspect_ai instructor tabulate wikipedia python-dotenv pydantic pandas numpy plotly httpx jaxtyping
   ```
3. Create a `.env` file with your API keys:
   ```
   OPENAI_API_KEY=your_key_here
   ANTHROPIC_API_KEY=your_key_here
   ```
4. Run notebooks in order (01 through 04), as later notebooks depend on outputs from earlier ones

## Technologies

- **LLM APIs**: OpenAI (GPT-4o-mini), Anthropic (Claude 4.5 Sonnet)
- **Evaluation framework**: [Inspect AI](https://inspect.ai-safety-institute.org.uk/) (UK AISI)
- **Visualization**: Plotly
