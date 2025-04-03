# EmailAgent: AI agent Email Module for CTRM AI Suite

**EmailAgent is the dedicated intelligent email processing module for the CTRM AI Suite. It leverages advanced AI models and document processing capabilities to manage Gmail interactions, automate the extraction of critical trade data (like freight and business confirmations) from emails and attachments, and function as a standalone conversational email assistant.**

Designed for seamless integration, EmailAgent understands natural language commands, interacts directly with Gmail, and intelligently parses relevant information, reducing manual data entry and enhancing the core CTRM system's data richness and responsiveness.

## Core Capabilities

**As a Module of CTRM AI Suite:**

*   **Automated Data Extraction:** Intelligently parses emails and attachments (including PDFs using `unstructured` and OCR) to identify and extract key structured data relevant to CTRM workflows (e.g., Freight Confirmations, Trade Confirmations, Vessel details, Dates, Rates, Quantities).
*   **CTRM Integration Points:** Designed to push extracted structured data to the central CTRM AI Suite system (details in Integration section).
*   **Reduced Manual Entry:** Aims to significantly decrease the need for manual monitoring of mailboxes and data entry into the CTRM system.
*   **Audit Trail:** Provides conversational history and logging for traceability of information retrieval.

**As a Standalone Assistant:**

*   **Natural Language Interaction:** Converse with the agent in plain English via a Streamlit web UI.
*   **Gmail Management:**
    *   Search emails using various criteria (sender, subject, date, read/unread status, keywords).
    *   Retrieve specific email details (subject, sender, body, date, recipients).
    *   Send new emails with specified recipients, subject, and body.
*   **Attachment Processing:**
    *   Identifies emails with attachments.
    *   **PDF Text Extraction:** Extracts full text content from PDF attachments using direct extraction and OCR fallbacks.
*   **Contextual Awareness:** Remembers the context of the conversation (e.g., referring to "that last email").
*   **Robust Planning:** Uses the DeepSeek-V3 language model (via Together AI) for reliable task planning and tool selection.

## Demo Scenarios

*(Embed animated GIFs/screenshots showing relevant scenarios)*

**1. Automated Freight Confirmation Processing:**

`[Animated GIF showing an email with a PDF freight confirmation arriving, the user asking "Process the freight confirmation in the last email", and the agent extracting key details like Vessel, Commodity, Load Port, Rate, etc. Add a text overlay/note: "Extracted data ready for CTRM upload."]`

**2. Interactive Email Search & Query:**

`[Animated GIF showing a user asking "Find emails about Corn shipments from last month" and the agent returning relevant results.]`

**3. Standalone Email Assistant Task:**

`[Animated GIF showing the user asking "Send a follow-up email to trader@example.com asking for the ETA on vessel Luna Bianca" and the agent composing and sending the email.]`

*(Optional: Link to a longer video walkthrough)*

## Technology Stack

The EmailAgent module integrates several powerful technologies:

*   **LLM:** `DeepSeek-V3` (via `Together AI`) - For core reasoning, planning, and understanding.
*   **LLM Platform:** `Together AI` - Provides API access for LLM inference.
*   **Framework:** `LangChain` (Core concepts like Tools, Memory, Schemas), `Streamlit` (Standalone UI)
*   **Email Backend:** `Gmail API` via `google-api-python-client`
*   **Authentication:** `google-auth-oauthlib` / `google-auth-httplib2`
*   **Document Processing:**
    *   `unstructured` - Primary library for direct text/structure extraction from PDFs and other formats.
    *   `EasyOCR` - OCR library for image-based text extraction (PDF fallback).
    *   `pdf2image` - Converts PDF pages to images for EasyOCR.
    *   `Poppler` - **Required System Dependency** for PDF rendering used by `pdf2image` and `unstructured`.
*   **Core Language:** `Python 3.9+`
*   **Data Validation:** `Pydantic`

## Module Architecture Overview

The EmailAgent module is built using a modular, event-driven architecture internally. Key components include:

*   **Agent Core (`CustomZeroShotAgent`)**: Orchestrates the workflow, interacts with the LLM for planning, and executes tools. Uses few-shot prompting and robust parsing for reliability.
*   **LLM Service (`DeepSeekTogetherService`)**: Interfaces with the Together AI API.
*   **Services (`GmailService`, `EasyOCRService`)**: Implement specific functionalities like interacting with Gmail or performing OCR.
*   **Processors (`PDFProcessor`)**: Handle specific data types, like PDFs, using appropriate libraries.
*   **Tools (`BaseTool` implementations)**: Agent's callable capabilities.

This internal design promotes separation of concerns. Externally, the module is designed to expose necessary hooks or interfaces for integration with the broader CTRM AI Suite.

For a more detailed view of the *internal* architecture, see the `ARCHITECTURE_OVERVIEW.md` document. *(Link if including that file)*. A full, detailed architecture document (`architecture.md`) is maintained within the private source code repository.

## Advanced Prompt Engineering and Agent Orchestration

Achieving reliable and effective agent behavior requires more than simply connecting a Large Language Model (LLM) to a set of tools. EmailAgent incorporates specific engineering techniques to guide the LLM and orchestrate the system's components:

1.  **Structured Zero-Shot Planning with Few-Shot Examples:**
    *   Instead of relying on complex agent executors, EmailAgent employs a custom `CustomZeroShotAgent`. This agent uses a meticulously crafted **Prompt Template (`PLANNING_PROMPT_TEMPLATE`)**.
    *   This template provides the LLM (DeepSeek-V3) with clear **role definition**, dynamically updated **tool descriptions**, **conversation history**, and the **user's request**.
    *   Critically, the template includes **Few-Shot Examples**. These demonstrate the *exact* desired JSON output format for various scenarios (tool actions, questions to user, final answers), significantly improving the LLM's ability to adhere to the strict output structure required for programmatic parsing and reducing planning errors.

2.  **Implicit Chain-of-Thought (CoT) via History and Reasoning:**
    *   While not using a separate explicit CoT chain, the agent fosters step-by-step reasoning. The LLM is instructed to generate a `reasoning` field within its JSON plan, explaining its choice of action.
    *   The full conversation history, including previous thoughts, actions, and crucially, the **observations** (tool results), is fed back into the prompt for subsequent planning steps. This allows the LLM to implicitly follow a chain of thought, correcting its plan based on previous outcomes or using information retrieved in earlier steps.

3.  **Context Injection and Tool Definitions:**
    *   The agent dynamically injects context into the prompt, such as which services (like Gmail) are currently authenticated (`StatusTracker`), ensuring the LLM only plans actions that are feasible.
    *   Tools are defined using LangChain's `BaseTool` interface, each with a Pydantic `args_schema`. This provides a structured way for the agent (and the LLM via the prompt) to understand what parameters each tool requires.

4.  **Robust Output Parsing and Parameter Handling:**
    *   Anticipating variations in LLM output, the agent includes logic to specifically **extract JSON from markdown code fences** (```json ... ```).
    *   It performs **whitespace and newline normalization** before attempting to parse the JSON, preventing common errors.
    *   Before executing a tool, the agent **validates the parameters** provided by the LLM against the tool's `args_schema`, ensuring all required arguments are present.
    *   Tool execution consistently uses **keyword argument unpacking (`**kwargs`)**, allowing tools to define clear, multi-argument signatures rather than relying on parsing a single input string.

5.  **Orchestration Logic:**
    *   The `CustomZeroShotAgent` acts as the central orchestrator, managing the loop of: LLM planning -> Plan parsing & validation -> Tool execution -> Observation gathering -> History update -> Repeat.
    *   This custom loop provides fine-grained control over the agent's behavior and error handling compared to using high-level LangChain agent executors directly.

These techniques collectively represent significant engineering effort to enhance the base capabilities of the LLM, resulting in a more reliable, predictable, and useful AI agent capable of performing complex email and document processing tasks within the CTRM domain.

## Integration with CTRM AI Suite

EmailAgent is designed as a component of the larger CTRM AI Suite. Its primary integration function is to monitor specified Gmail accounts, process incoming emails and attachments, extract relevant structured trading data, and push this data into the central CTRM system.

The envisioned integration mechanism involves:

1.  **Data Extraction:** EmailAgent processes emails/PDFs (triggered by events or schedules) and identifies key data points (e.g., vessel name, cargo type, quantity, price, dates, counterparties) based on predefined schemas or LLM-driven analysis.
2.  **Data Transmission:** The extracted, structured data is then transmitted to the CTRM AI Suite's main application/database via appropriate mechanisms (e.g., message queues, dedicated APIs).

*(Adapt License/Contributing sections as appropriate for the CTRM AI Suite project context)*
