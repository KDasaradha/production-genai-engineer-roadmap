# Knowledge Base Consolidation Task

You are acting as a Senior Technical Documentation Engineer and Knowledge Base Architect.

Your task is to analyze, consolidate, reorganize, and normalize a collection of Markdown files that were generated from multiple ChatGPT conversations over a long period of learning.

## Primary Goal

Create a single, clean, well-organized, interview-ready learning knowledge base while preserving ALL valuable information.

The final result should be suitable for:

* Personal learning
* Revision and interview preparation
* Sharing with friends and colleagues
* Long-term knowledge management

---

# Critical Rules

## Rule 1: Never Delete Valuable Information

Do NOT remove any useful content.

If multiple files contain unique information, preserve all of it.

Only remove:

* Exact duplicates
* Near-identical repeated paragraphs
* Repeated code blocks that provide no additional value
* Repeated explanations that convey the exact same information

If uncertain, keep the content.

---

## Rule 2: Process Files One-by-One

Do NOT analyze all files at once.

Process files sequentially.

For each file:

1. Read the entire file.
2. Understand the content.
3. Identify topics.
4. Compare with existing organized notes.
5. Merge information into the correct destination files.
6. Preserve any new or unique information.
7. Move to the next file.

This incremental approach is mandatory.

---

## Rule 3: Create a New Organized Knowledge Base

Create a new folder:

knowledge-base/

Do not modify original source files.

Treat original files as read-only references.

---

# Required Folder Structure

knowledge-base/

├── 00-Master-Roadmap/
│   ├── Learning-Roadmap.md
│   ├── Learning-Index.md
│   └── Progress-Tracker.md
│
├── 01-Python/
│
├── 02-FastAPI/
│
├── 03-PostgreSQL/
│
├── 04-Redis/
│
├── 05-System-Design/
│
├── 06-Software-Architecture/
│
├── 07-Prompt-Engineering/
│
├── 08-Generative-AI/
│
├── 09-RAG/
│
├── 10-Agentic-AI/
│
├── 11-LLM-Engineering/
│
├── 12-Projects/
│
├── 13-Interview-Preparation/
│
├── 14-Career/
│
└── Resources/

Create additional folders if necessary.

---

# Content Organization Rules

For every topic:

Create structured notes using the following format whenever applicable.

# Topic Name

## Definition

## Why It Exists

## When To Use

## When Not To Use

## Internal Working

## Advantages

## Tradeoffs

## Limitations

## Examples

## Code Examples

## Common Mistakes

## Best Practices

## Interview Questions

## Interview Answers

## Related Topics

---

# Topic Consolidation Rules

If multiple files discuss the same topic:

Example:

File A:
Decorators

File B:
Python Decorators

File C:
Advanced Decorators

Merge into:

01-Python/decorators.md

Organize content logically.

Do not keep duplicate explanations.

Keep all unique information.

---

# Question Collections

If notes contain:

* Interview questions
* Scenario questions
* Design questions
* Behavioral questions

Create dedicated files.

Examples:

13-Interview-Preparation/

python-interview-questions.md

fastapi-interview-questions.md

postgresql-interview-questions.md

system-design-interview-questions.md

genai-interview-questions.md

rag-interview-questions.md

agentic-ai-interview-questions.md

Merge duplicates and preserve unique questions.

---

# Scenario Collections

If notes contain scenarios:

Example:

"How would you scale FastAPI?"

"Design a chatbot serving 1 million users"

Store them in dedicated scenario files.

Example:

system-design-scenarios.md

backend-scenarios.md

genai-scenarios.md

---

# Project Organization

Collect all projects from all files.

Create:

12-Projects/

For every project include:

# Project Name

## Goal

## Architecture

## Folder Structure

## Implementation Steps

## Code References

## Interview Talking Points

## Production Considerations

---

# Duplicate Detection Rules

Only remove content when:

Meaning = identical

Information = identical

Explanation = identical

Code = identical

Otherwise merge information.

Never remove information simply because it looks similar.

---

# Cross Linking

At the end of every topic include:

## Related Topics

Example:

Related:

* AsyncIO
* FastAPI
* WebSockets
* Background Tasks

---

# Learning Roadmap

Create:

00-Master-Roadmap/Learning-Roadmap.md

Build a complete learning path from beginner to advanced covering:

1. Python
2. FastAPI
3. PostgreSQL
4. Redis
5. System Design
6. Software Architecture
7. Prompt Engineering
8. Generative AI
9. RAG
10. Agentic AI
11. Production AI Systems
12. Interview Preparation

Order topics correctly according to prerequisites.

---

# Documentation Quality Standards

Every file should:

* Have clear headings
* Use markdown properly
* Be easy to read
* Avoid repetition
* Preserve all unique information
* Be suitable for future interview preparation

---

# Final Deliverable

Produce a fully organized knowledge base where:

* All markdown files have been reviewed.
* All useful content has been preserved.
* Duplicate content has been removed.
* Related information has been merged.
* Topics are properly categorized.
* Interview notes are centralized.
* Project notes are centralized.
* Learning roadmap is available.
* Original source files remain untouched.

Most important rule:

PRESERVE KNOWLEDGE.
REMOVE DUPLICATES.
IMPROVE ORGANIZATION.
DO NOT LOSE INFORMATION.
PROCESS FILES ONE-BY-ONE.


---

You are a Senior Knowledge Base Architect and Technical Documentation Engineer.

Your task is to transform this repository into a clean, organized, AI-friendly learning knowledge base.

Before making any changes, read and understand all existing files and folders carefully.

Requirements:

1. Review every Markdown file one-by-one, not all at once.
2. Fully understand the content before making changes.
3. Create a new organized knowledge base structure if it does not already exist.
4. Do NOT modify or delete the original source files.
5. Preserve all useful information.
6. Remove only exact or near-exact duplicate content.
7. If multiple files discuss the same topic, merge them into a single well-organized topic file.
8. Never remove unique explanations, examples, code snippets, interview questions, scenarios, or project details.
9. Organize content into appropriate categories such as:

   * Python
   * FastAPI
   * PostgreSQL
   * Redis
   * System Design
   * Software Architecture
   * Prompt Engineering
   * Generative AI
   * RAG
   * Agentic AI
   * Projects
   * Interview Preparation
   * Career
10. Create topic-based notes instead of chat-based notes.
11. For each topic, organize content using clear sections where applicable:

    * Definition
    * Internal Working
    * Use Cases
    * Advantages
    * Tradeoffs
    * Examples
    * Code Examples
    * Best Practices
    * Common Mistakes
    * Interview Questions
    * Related Topics
12. Consolidate all interview questions into dedicated interview-preparation files.
13. Consolidate all project-related content into dedicated project files.
14. Create index and roadmap files to make navigation easy.
15. Add links between related topics where appropriate.
16. Maintain consistent Markdown formatting throughout the repository.
17. Focus on improving organization and readability without losing knowledge.
18. Process files incrementally and carefully to avoid missing information.
19. The final repository should serve as:

    * A personal learning knowledge base
    * An interview preparation resource
    * A shareable reference for friends and colleagues
    * An AI-friendly repository that tools like ChatGPT, Codex, Claude, Cursor, Gemini, Cline, and Roo Code can easily understand.

Core Rule:

Preserve knowledge.
Remove duplicates.
Improve organization.
Do not lose information.
Process files one-by-one.
Create a single source of truth for every topic.
