---
title: "DataViz Pro: Building an AI-Powered Business Intelligence Platform"
date: 2025-08-11
draft: false
description:
  "Creating a full-stack business intelligence platform with natural language
  querying using React, Python, LangGraph, and GPT-4/Gemini LLM"
tags:
  [
    "ai",
    "llm",
    "business-intelligence",
    "react",
    "python",
    "langgraph",
    "data-visualization",
    "sql",
    "gemini",
    "gpt",
  ]
toc: true
cover:
  src: ./demo-cover.gif
  alt:
    "DataViz Pro demo showing AI-powered data visualization in action with
    natural language queries and real-time chart generation"
---

## The Vision: AI-Powered Data Analysis for Everyone

During my internship at GXCO, I encountered a common problem: non-technical
stakeholders struggling to extract insights from complex databases. They had the
questions, but not the SQL skills. This led me to build **DataViz Pro** - an
AI-powered business intelligence platform that transforms natural language
questions into SQL queries and dynamic visualizations.

## Architecture & Technical Implementation

DataViz Pro is built as a **microservices architecture** with four core
components:

**🎯 Core Services:**

- **React Frontend** (Port 3000): Modern UI with real-time streaming
- **Python Backend** (Port 8123): LangGraph-powered AI agent using GPT-4/Gemini
  LLM
- **SQLite Server** (Port 3001): Database query execution and file management
- **Conversation API** (Port 5000): Session management and chat history

**🧠 AI Agent Workflow:** The heart of the system is a **LangGraph workflow**
that processes queries through multiple stages:

1. **Question Parsing**: Determines relevance and identifies target database
   tables
2. **Noun Extraction**: Finds unique values for context-aware query generation
3. **SQL Generation**: Translates natural language to optimized SQL
4. **Validation**: Checks and fixes SQL syntax/logic errors
5. **Execution**: Runs queries against the database
6. **Visualization Selection**: Chooses appropriate chart types (bar, line, pie,
   scatter)
7. **Data Formatting**: Structures results for frontend consumption

**🔄 Context-Aware Conversations:** Unlike simple chatbots, DataViz Pro
maintains conversation context across multiple queries, enabling natural
follow-up questions like "Show me sales by category" → "Now filter for
electronics only" → "What about the top 5 products?"

**🎨 Dynamic Visualization Engine:** The platform automatically selects and
renders appropriate visualizations using AI-powered chart type selection based
on data characteristics.

**📊 Real-Time Streaming:** Built with Server-Sent Events (SSE) for live query
processing, allowing users to watch the AI agent work through problems
step-by-step.

**🏢 Complex Business Schema:**

Created a comprehensive e-commerce database with **14 interconnected tables**:

- **Customer Management**: customers, addresses, loyalty tiers
- **Product Catalog**: products, categories, suppliers, inventory
- **Order Processing**: orders, order_items, payment methods
- **Performance Analytics**: sales_performance, customer_reviews
- **Business Intelligence**: website_analytics, promotions

The platform handles complex business questions like "What are the top 5
products by sales revenue?", "Show customer loyalty distribution", "Which
employees exceeded their sales targets?", and "Compare monthly revenue trends".

## Challenges, Solutions & Lessons Learned

Key challenges included getting the AI to recognize exploratory questions as
relevant (solved through refined system prompts), maintaining conversation
context across microservices (solved with a dedicated conversation manager), and
making technical SQL errors user-friendly (solved through multi-layer error
handling with graceful degradation).

## Technology Stack & Performance

**Technology Stack:**

- **Frontend**: React/Next.js with real-time SSE updates, Tailwind CSS,
  Chart.js/Recharts
- **Backend**: Python with LangGraph, GPT-4/Gemini, raw SQL, Flask
- **Infrastructure**: Docker containers, SQLite database, UUID-based file
  management

**Performance Metrics:**

- Query Processing: Average 3-5 seconds from question to visualization
- Conversation Context: Maintains last 3 turns for follow-up accuracy
- Database Scale: Handles 14+ table schemas with complex relationships
- Concurrent Users: Designed for multi-user session isolation
- Visualization Types: 5+ chart types with intelligent auto-selection

**Key Lessons Learned:**

- Prompt engineering is critical for reliable LLM behavior
- Context management transforms chatbots into conversation partners
- Microservices enable scalability but require careful coordination
- Real-time features significantly improve user experience
- Natural language interfaces democratize data access

## Getting Started & Usage

**Prerequisites:** Node.js 20+, Python 3.11+, LangGraph Studio, API keys for
OpenAI or Google Gemini

**Quick Start:** The project includes a comprehensive service management script:

```bash
./manage_services.sh setup  # Creates environment files
./manage_services.sh start  # Starts all services
```

**Backend Options:** Choose between Python (recommended, uses Google
Gemini/OpenAI) or TypeScript (uses OpenAI)

**Common Issues:** Port conflicts (check with `./manage_services.sh status`),
environment setup (run `./manage_services.sh setup`), or file upload limits (1MB
max for SQLite/CSV files)

## Future Development

**Planned Features:** Multi-database support (PostgreSQL, MySQL), advanced
analytics with statistical modeling, collaborative dashboards, API integrations,
and React Native mobile app.

**Technical Roadmap:** Performance optimization through query caching, security
enhancements with role-based access control, Kubernetes scalability, and
fine-tuned domain-specific language models.

## Conclusion

DataViz Pro represents the future of business intelligence - where **domain
expertise matters more than SQL skills**. By combining modern AI capabilities
with thoughtful UX design, it transforms how organizations interact with their
data.

The platform demonstrates that with the right architecture, AI can bridge the
gap between complex data and actionable insights, making data-driven decision
making accessible to everyone.

## Demo

{{< video src="demo.mp4" >}}

> See how easy it is to upload data, ask natural language questions, and get
> beautiful visualizations powered by AI agents.

---

**🔗 Project Links:**

- [GitHub Repository](https://github.com/shubhpsd/data-viz)

**🛠️ Built With:** React, Python, LangGraph, GPT-4/Gemini LLM, SQLite, Docker

**📦 Quick Start:**

```bash
git clone https://github.com/shubhpsd/data-viz.git
cd data-viz
./manage_services.sh setup
./manage_services.sh start
```
