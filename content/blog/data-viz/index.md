---
title: "Building DataViz Pro: My Journey into AI-Powered Data Visualization"
date: 2025-08-11
draft: false
description:
  "How I built a natural language data visualization tool using React, Python,
  and LLMs - a learning project that turned into something pretty cool"
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
  alt: "DataViz Pro demo - chat with your data using natural language"
---

## What I Built

> Ever wished you could just ask your data questions like "show me sales by
> region" and get instant charts? That's exactly what I built with DataViz Pro.
> {{< video src="demo.mp4" >}}

## Why I Built This

As someone diving deep into AI/ML, I kept seeing this recurring problem
everywhere: people drowning in data but struggling to get answers. Everyone has
spreadsheets and databases, but asking simple questions like "which products are
trending?" requires either SQL skills or bothering the tech team.

I thought - what if we could just talk to our data, instead of figuring out what
SQL entires to put in to get the desired output? So I decided to build upon
something that lets you upload a CSV, ask questions in plain English, and get
beautiful visualizations instantly. Plus, I wanted to learn and fimiliarise
myself with LangGraph so that's that.

## How It Works

The app has four main pieces working together:

**Frontend (Next.js/TypeScript)** - Clean interface where you can upload data
and chat with it in real-time

**AI Backend (Python + LangGraph)** - This is where the magic happens. It takes
your question, figures out what data you need, writes the SQL, and decides what
chart to show

**Database Server** - Handles all the SQL execution and file management

**Chat Manager** - Keeps track of your conversation so you can ask follow-up
questions

When you ask something like "show me the top selling products," it goes through
these steps:

1. Understands what you're asking
2. Looks at your data structure
3. Writes the SQL query
4. Checks if the query makes sense
5. Runs it and gets results
6. Picks the best chart type to display results
7. Outputs answer in interactive layout

It also remembers context. You can ask "show me sales by month" then follow up
with "now just Q4" and it knows what you mean.

## The Tricky Parts

Building this taught me that AI can be... temperamental. Getting the language
model to understand vague questions like "show me interesting trends" was harder
than I expected. Turns out prompt engineering is real?

The biggest headache was keeping conversations flowing. Imagine asking "show me
sales" then "break that down by region" - the AI needs to remember what "that"
refers to. I ended up building a conversation manager that tracks context
between requests.

Also, SQL errors are cryptic. LLMs can generates bad SQL, I had to build an
error checker so users get helpful messages instead of "syntax error at line 1."

## What I Used & What I Learned

**Tech Stack:**

- Next.js with TypeScript for the frontend - type safety is kinda nice
- Tailwind + Material-UI for styling (MUI's charts are pretty solid)
- Python + LangGraph for the AI flow (learning LangGraph and it's pretty neat)
- React Flow for those interactive node diagrams you see in the workflow
- GPT and Gemini as the language models
- SQLite for now (keeping it simple, but PostgreSQL is on the roadmap)

**Performance:** Most queries take 3-5 seconds, feels snappy enough. The app can
handle multi-table databases and remembers the last few messages for context.

**What I learned:**

- TypeScript is a nice - catching errors at compile time instead of runtime is
  chef's kiss
- Prompt engineering is important - tiny errors can flip AI result completely
- Real-time streaming makes everything feel more alive and interactive
- Material-UI's chart components are surprisingly powerful when you need quick,
  clean visualizations
- Building conversational AI is a token optimisation game

## Want to Try It?

You'll need Node.js 20+, Python 3.11+, and API keys for OpenAI or Google Gemini.
Oh, and LangGraph Studio makes life way easier.

**Quick Setup:**

```bash
git clone https://github.com/shubhpsd/datavisualization_langgraph.git
cd datavisualization_langgraph
./manage_services.sh setup    # Creates all the environment files
```

Then you need to add your API keys to the environment files it creates. The
setup script is pretty helpful - it'll create templates and tell you exactly
what to edit.

**Start Everything:**

```bash
./manage_services.sh start    # Starts all services automatically
```

This fires up:

- Frontend on localhost:3000
- SQLite server on localhost:3001
- Conversation API on localhost:5001
- Plus you'll need to start the LangGraph backend (I recommend LangGraph Studio)

**Pro tip:** The service management script is actually pretty useful - you can
use `./manage_services.sh status` to check what's running,
`./manage_services.sh logs` to debug issues, and `./manage_services.sh health`
to make sure everything's working.

If stuff breaks (it happens), it's usually port conflicts or missing API keys.
The script usually gives decent error messages to point you in the right
direction.

## What's Next

Adding support for PostgreSQL and MySQL would be better (SQLite is great for
demos but has limits). Also downloading/sharing features so teams can easily use
the dashboards.

## Wrapping Up

This project taught me that the future of data analysis isn't about replacing
analysts - it's about making data accessible to everyone. When you remove the
SQL barrier, people ask way more interesting questions.

Building DataViz Pro was my deep dive into conversational AI, and honestly, it
got me even more excited about the possibilities. We're just scratching the
surface of what's possible when you combine language models with structured
data.

If you're getting into AI/ML, I'd definitely recommend building something that
combines multiple technologies.

As always you will learn more by doing that by watching.

---

**🔗 Project Links:**

- [GitHub Repository](https://github.com/shubhpsd/data-viz)

**🛠️ Built With:** Next.js, TypeScript, Python, LangGraph, Material-UI,
GPT/Gemini

**📦 Quick Start:**

```bash
git clone https://github.com/shubhpsd/datavisualization_langgraph.git
cd datavisualization_langgraph
./manage_services.sh setup
./manage_services.sh start
```
