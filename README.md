# AutoRex
## AI-Powered Multi-Agent Customer Support Automation System

**Project Type:** AI Automation / Multi-Agent Customer Support  
**Platform:** n8n  
**Primary Channel:** Telegram  
**Database & Vector Store:** Supabase / PostgreSQL  
**AI Integration:** OpenAI  
**Status:** Core workflow developed, tested, and audited

---

# 1. Project Overview

AutoRex is an AI-powered multi-agent customer support automation system designed to automate customer interactions, classify incoming requests, route them to specialized AI agents, retrieve relevant information from a knowledge base, and escalate cases requiring human intervention.

The system uses a centralized triage mechanism to understand incoming customer messages and determine the appropriate department, priority, sentiment, and language before routing requests to specialized agents.

Rather than relying on a single general-purpose AI agent, AutoRex uses a structured multi-agent architecture in which each specialized agent handles a defined category of customer requests.

The system also maintains conversation context, stores customer interactions, supports human escalation, and includes an evaluation pipeline for assessing AI-generated responses and routing decisions.

**The main objective is to build a structured, modular, and maintainable customer support automation system that combines AI reasoning with database operations and controlled workflow execution.**

# 2. The Problem

Traditional customer support systems often face several operational challenges:

- High volumes of repetitive customer inquiries.
- Manual classification and assignment of support requests.
- Delays in identifying the correct department.
- Inconsistent responses across different support categories.
- Difficulty maintaining conversation context.
- Limited visibility into AI response quality.
- The need to identify cases that require human intervention.

A single AI assistant may handle different topics, but without clear routing and specialized responsibilities, maintaining consistency and controlling the support process becomes more difficult.

AutoRex addresses these challenges through a multi-agent architecture with centralized classification, specialized processing, persistent conversation records, and human escalation.

# 3. Project Objectives

The system was designed around the following objectives:

1. Automate the initial handling of customer messages.
2. Classify customer requests before processing them.
3. Route requests to the appropriate specialized agent.
4. Maintain customer and conversation context.
5. Retrieve relevant information from a structured knowledge base.
6. Support human intervention when necessary.
7. Store messages and processing records.
8. Evaluate AI responses and routing decisions.
9. Improve workflow reliability through validation, retry handling, and duplicate-message checks.
10. Organize the architecture into understandable and maintainable workflow sections.

# 4. System Architecture

AutoRex follows a centralized intake and routing architecture.

The main processing flow is:

**Telegram Input → Message Validation → Customer Identification → Conversation Management → Message Storage → Context Construction → AI Triage → Department Routing → Specialized Agent → Response Preparation → Conversation Update → Telegram Response**

Additional paths handle human escalation and AI evaluation.

## 4.1 Main Architectural Components

### A. Telegram Intake

The Telegram Trigger receives incoming customer messages.

The intake process includes:

- Receiving Telegram updates.
- Normalizing incoming message data.
- Extracting message text and relevant metadata.
- Identifying the message language.
- Validating that the incoming message contains usable text.
- Stopping invalid or empty messages before further processing.

AutoRex currently supports text-based customer messages.

### B. Customer Identification

The system searches for the customer in the database.

This allows incoming messages to be associated with an existing customer record rather than creating unrelated customer records for every interaction.

### C. Conversation Management

AutoRex checks whether the customer already has an active conversation.

The workflow follows two possible paths:

- Existing conversation: retrieve and resolve the existing conversation.
- New conversation: create a conversation and resolve the new record.

Both paths are then unified into the main processing flow.

### D. Duplicate Message Protection

Before storing an incoming message, AutoRex checks whether the same Telegram message has already been recorded for the resolved conversation.

The duplicate-check process uses:

- `Check Duplicate Message`
- `Is Duplicate Message?`

Messages identified as duplicates are stopped from continuing through the normal processing path.

### E. Message Storage

Customer messages are stored in PostgreSQL.

The system also updates conversation timestamps and loads the relevant conversation context for subsequent processing.

### F. Conversation Context Construction

The workflow loads previous conversation information and builds a structured context for the AI agents.

This gives the triage and specialist agents access to relevant conversation history when interpreting follow-up messages.

# 5. AI Triage Engine

The Triage Agent is the central decision-making component of AutoRex.

Its responsibility is to analyze the customer's latest message, use relevant conversation history when needed, and generate a structured routing decision.

The triage output includes:

| Field | Purpose |
|---|---|
| `customer_message` | Original customer message |
| `intent` | Identified request type |
| `category` | Main request category |
| `department` | Destination department |
| `priority` | Request priority |
| `sentiment` | Customer sentiment |
| `language` | Detected language |
| `confidence` | Classification confidence |
| `needs_human` | Human escalation decision |
| `reason` | Explanation of the classification |

## 5.1 Supported Categories

AutoRex currently supports five routing categories:

- Billing
- Account
- Technical
- Sales
- General

Each category maps to a corresponding specialized agent.

## 5.2 Priority Classification

The triage process assigns one of four priority levels:

- Low
- Medium
- High
- Urgent

Priority is determined from the available evidence in the customer's request rather than simply from the presence of negative sentiment.

## 5.3 Sentiment Analysis

The system classifies customer sentiment as:

- Positive
- Neutral
- Frustrated
- Angry

## 5.4 Human Escalation Decision

The triage agent also determines whether a request should be escalated to a human representative.

This decision is represented through the `needs_human` field.

The result is normalized before reaching the escalation gate and department router.

# 6. Specialized AI Agents

AutoRex contains five specialized agents.

Each agent is responsible for a specific support domain.

## 6.1 Billing Agent

Handles billing-related customer inquiries.

Its scope includes:

- Billing cycles.
- Subscription information.
- Payment-related questions.
- Invoices and receipts.
- Refund-related inquiries.
- Billing discrepancies.

## 6.2 Account Agent

Handles account-related inquiries.

Its scope includes:

- Account access.
- Password reset guidance.
- Account recovery.
- Account information.
- Profile and account settings.
- Account-related questions.

## 6.3 Technical Support Agent

Handles technical support requests.

Its scope includes:

- Technical issues.
- Application errors.
- Unexpected product behavior.
- Troubleshooting.
- Technical functionality questions.
- Knowledge-base-supported technical guidance.

This agent is connected to the knowledge retrieval infrastructure.

## 6.4 Sales Agent

Handles product and purchasing-related inquiries.

Its scope includes:

- Product questions.
- Plan comparisons.
- Subscription options.
- Pricing inquiries.
- Upgrade questions.
- Purchase-related guidance.

## 6.5 General Support Agent

Handles inquiries that do not belong to the other specialized departments.

It provides general assistance within the defined scope of the support system.

# 7. Knowledge Base and RAG

AutoRex includes a knowledge retrieval component built around Supabase Vector Store and OpenAI embeddings.

The purpose of this component is to allow the connected agent to retrieve relevant information from stored knowledge rather than depending exclusively on the model's general knowledge.

## 7.1 Knowledge Base Components

The implemented infrastructure includes:

- Knowledge documents.
- Document chunking.
- Text embeddings.
- Supabase Vector Store.
- Knowledge retrieval.
- Integration with the technical support agent.

## 7.2 Knowledge Processing

The knowledge preparation process includes:

1. Loading knowledge documents.
2. Processing document content.
3. Dividing content into smaller chunks.
4. Generating embeddings.
5. Storing vector representations.
6. Making stored information available for retrieval.

The workflow also contains supporting knowledge-management operations for document and chunk creation and embedding updates.

## 7.3 Example Knowledge Content

The project was tested with subscription-plan information, including:

| Plan | Monthly Price | Main Features |
|---|---:|---|
| Basic | $9.99 | 5 projects, standard features, basic support |
| Pro | $19.99 | Unlimited projects, advanced features, priority support |
| Enterprise | $49.99 | Unlimited projects, advanced analytics, priority and dedicated support |

These entries demonstrate how structured product information can be stored and retrieved.

# 8. Human Escalation System

Human escalation is an important part of AutoRex.

The system is designed to avoid forcing every customer request through automated processing when human intervention is necessary.

## 8.1 Escalation Flow

The escalation process includes:

1. Receiving the escalation decision.
2. Checking whether human intervention is required.
3. Preparing escalation information.
4. Creating a human-escalation record in PostgreSQL.
5. Preparing the customer notification.
6. Sending a Telegram message to inform the customer.

## 8.2 Escalation Record

The human-escalation table contains fields including:

- `id`
- `conversation_id`
- `ticket_id`
- `reason`
- `description`
- `agent_confidence`
- `assigned_to`
- `status`
- `human_resolution`
- `created_at`
- `resolved_at`

This provides a structured record of cases requiring human intervention.

# 9. Response Management and Conversation Logging

After the specialized agent generates a response, AutoRex processes the result through a response-preparation stage.

The response path includes:

- Normalizing agent responses.
- Checking the escalation decision.
- Preparing the final customer response.
- Storing the assistant response.
- Updating the conversation.
- Sending the response through Telegram.

This ensures that the customer interaction is not limited to a model-generated answer but is also represented in the conversation records.

# 10. AI Evaluation Pipeline

AutoRex includes a separate evaluation pipeline for assessing AI processing results.

The evaluation workflow contains the following stages:

1. Prepare Evaluation Data.
2. Fetch Evaluation Messages.
3. Build Evaluation Record.
4. AI Evaluator.
5. Merge and normalize evaluation results.
6. Parse evaluation output.
7. Save Evaluation.

The evaluation pipeline is intended to provide a structured way to assess the quality of AI-generated responses and routing decisions.

It also separates evaluation from the normal customer-response path.

# 11. Database and Data Management

AutoRex uses Supabase/PostgreSQL as its persistent data layer.

The project database includes structures for customer records, conversations, messages, tickets, knowledge documents, knowledge chunks, triage results, evaluations, AI usage, and tool execution records.

The database supports several essential operations:

- Customer identification.
- Conversation persistence.
- Message storage.
- Knowledge storage.
- Escalation records.
- Triage-result storage.
- Evaluation records.

The database acts as the persistent foundation connecting the workflow's different processing stages.

# 12. Reliability and Error Handling

Reliability was reviewed as a dedicated development stage.

The implemented measures include:

### Input Validation

Incoming messages are checked before entering the main workflow.

Empty or invalid text messages are stopped.

### Retry Handling

Retry On Fail was configured for the reviewed AI and database operations.

The configured retry settings include two attempts with a 2,000 ms wait between attempts.

### Duplicate Message Checks

The workflow checks for previously stored Telegram messages before continuing with normal processing.

### Execution Monitoring

The n8n execution history was reviewed during the reliability audit.

The audit was based on the workflow's tested behavior and the execution records reviewed during development.

# 13. Security Review

Security was reviewed across four areas:

1. API keys and database credentials are stored in n8n Credentials.
2. Credential exposure was reviewed.
3. Customer data handling was reviewed.
4. Access restrictions were reviewed across Supabase, PostgreSQL, n8n, and Telegram.

The project owner confirmed that the reviewed security points were correctly configured.

This represents a configuration review, not an independent penetration test or formal security certification.

# 14. Testing and Validation

The five specialized agents were tested individually.

## 14.1 Agent Testing Results

| Agent | Tests Passed | Total Tests | Result |
|---|---:|---:|---|
| Account | 4 | 4 | 100% |
| Billing | 4 | 4 | 100% |
| Technical Support | 5 | 5 | 100% |
| Sales | 8 | 8 | 100% |
| General Support | 6 | 6 | 100% |
| **Total** | **27** | **27** | **100%** |

These results represent the recorded test set used during development. They should not be interpreted as proof of perfect accuracy across all possible customer messages.

## 14.2 Additional Testing

The following components were also tested:

- Knowledge-base retrieval.
- Human escalation.
- Incoming-message validation.
- Duplicate-message checking.
- Main routing behavior.

The project also underwent a reliability and security review.

# 15. Technology Stack

| Technology | Role |
|---|---|
| n8n | Workflow orchestration and automation |
| OpenAI | AI models and embeddings |
| Supabase | Database and vector storage infrastructure |
| PostgreSQL | Persistent data storage |
| Telegram Bot API | Customer messaging interface |
| Vector Search | Knowledge retrieval |
| Structured Output | Consistent AI output format |

# 16. Workflow Organization

The main workflow is organized around a sequence of connected processing stages:

1. Message Intake.
2. Customer and Conversation Resolution.
3. Message Validation and Duplicate Protection.
4. Context Construction.
5. AI Triage.
6. Department Routing.
7. Specialized AI Agents.
8. Response and Human Escalation.
9. Evaluation Pipeline.

The workflow was visually reorganized to make its architecture and connections easier to understand while preserving the existing processing logic.

# 17. Project Scope

AutoRex focuses on two primary business functions:

- Customer Support.
- Sales Support.

The system does not include a dedicated marketing department or marketing agent.

The current intake channel is Telegram.

The current implementation focuses on text-based customer interactions.

# 18. Known Limitations and Future Improvements

The following items are possible future improvements, not claims about existing functionality.

### Reliability

- Strengthen duplicate protection using database-level uniqueness and conflict handling to address concurrent processing.
- Expand automated failure and recovery testing.
- Test behavior under higher message volumes.

### Knowledge Base

- Document the knowledge-update process.
- Expand the knowledge corpus.
- Test retrieval quality across a broader set of documents and questions.

### AI Evaluation

- Expand the evaluation dataset.
- Define consistent evaluation criteria.
- Track evaluation results over longer periods.
- Compare performance across agent versions.

### Production and Operations

- Conduct an independent security assessment.
- Test production behavior under realistic workloads.
- Document deployment, recovery, and maintenance procedures.

### Portfolio Presentation

- Publish a sanitized workflow export.
- Prepare a complete architecture diagram.
- Include workflow screenshots.
- Record an end-to-end demonstration.
- Publish the project documentation.

# 19. Project Outcomes

The development of AutoRex resulted in a multi-agent customer support workflow with:

- Centralized AI triage.
- Five specialized support agents.
- Structured customer and conversation management.
- Knowledge retrieval infrastructure.
- Human escalation.
- Persistent conversation logging.
- An AI evaluation pipeline.
- Input validation and duplicate-message checks.
- A reviewed security and reliability configuration.

The recorded specialized-agent test suite achieved 27 successful tests out of 27.

# 20. Conclusion

AutoRex demonstrates how workflow automation, large language models, database operations, vector search, and multi-agent architecture can be combined into a structured customer support system.

Its architecture separates request classification, specialized processing, escalation, and evaluation into distinct stages.

The result is a modular automation project that goes beyond a simple chatbot by incorporating persistent conversation management, specialized agent responsibilities, knowledge retrieval, and controlled routing.

**AutoRex represents an implementation of an AI-powered customer support automation architecture using n8n, OpenAI, Supabase, PostgreSQL, and Telegram.**

---

## Project Links and Media

**GitHub Repository:** [Add repository URL]

**Workflow Screenshots:** [Add screenshots]

**Architecture Diagram:** [Add diagram]

**Demo Video:** [Add demo URL]

**Workflow Export:** [Add sanitized export]

**Project Author:** [Add your preferred public name or username]
