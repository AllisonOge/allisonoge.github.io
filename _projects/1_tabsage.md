---
layout: page
title: TabSage
description: retrieval-augmented QA assistant for your browser tabs
img:
importance: 2
category: fun
disqus_comments: true
---

Modern browsing is increasingly noisy. Pages are longer, content is denser, and users often need to cross-reference multiple tabs before they can answer a single question. This creates a productivity problem: people spend too much time searching, scrolling, and switching context.

TabSage was built to make web browsing more efficient by adding an AI assistant at the browser level. Instead of being tied to a single website, it works alongside your browsing session, helps you find relevant information faster, and keeps answers grounded in the pages you are viewing.

## Project Goal

The goal of TabSage is to help users reach their objective faster while browsing by:

* Reducing time spent scanning long pages.
* Connecting relevant information across tabs.
* Providing grounded answers with clear source references.
* Helping users retain useful information through saved notes and self-testing.

## Solution

TabSage is a browser-level assistant with access to the active tab (and supporting browsing context) that uses a user-selected reasoning model (LLM) to assist while browsing.

Core capabilities include:

* Ask questions about the current webpage while browsing.
* Highlight supporting sources on the page so answers are verifiable.
* Save notes from previous chats and reuse them as context later.
* Generate MCQs from saved notes for quick self-testing and retention.

This design is intended for broad use: students, researchers, developers, and general web users who regularly work across many pages and need faster information retrieval.

## System Design

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/tabsage_design.png" title="System Architecture" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    System Architecture
</div>

TabSage follows a retrieval-augmented generation (RAG) pipeline. When a user asks a question, the system retrieves the most relevant context and passes it to the reasoning model so the answer is grounded in what the user is actually viewing.

### How It Works

1. **Page snapshotting**: capture active-page content (DOM text and links).
2. **Chunking + embeddings**: split content into chunks and convert them to embeddings.
3. **Vector storage**: store embeddings in a per-page vector store.
4. **Query retrieval**: embed the user query and run top-k similarity search.
5. **Grounded response generation**: send retrieved chunks to the LLM to produce an answer with references.

### Conversational RAG

A basic RAG setup can ignore conversation state, which often causes repetitive responses or loss of context. To address this, TabSage uses conversational RAG:

* Chat history is retained.
* New user messages are rewritten into standalone queries before retrieval.
* Retrieval therefore reflects the full conversational intent, not just the latest message.

### Multi-Source Grounding

Retrieval context comes from two sources:

1. The current webpage content.
2. User-saved notes from previous chats.

Saved notes act as reusable memory and can also be converted into MCQs to support studying and retention.

### Per-Page Persistence (Token Optimization)

A key implementation detail is persistence per active page. TabSage builds and indexes a vector store for each page, then reuses it across multiple user interactions until the page changes.

This avoids re-embedding the same content repeatedly, which reduces latency and token usage.

## Main Results

We evaluated the impact of persisting the page vector store across repeated interactions on the same page. For the same number of chats with the LLM, we measured up to **33% token reduction** (**80.3k -> 53.8k**).

This is expected: without caching, each interaction repeats embedding work on the same page content. With caching, the vector store is created once and reused for subsequent queries.

In simple terms, if each page embedding pass costs `T` tokens and there are `N` interactions on the same page:

* Without caching: token cost grows roughly with repeated embedding work across interactions.
* With caching: the page embedding cost is paid once, then reused.

The observed reduction confirms that persistent page-level indexing is a practical optimization for browser-based RAG assistants.

## Challenges and Future Scope

### 1. Navigation / Roadmap-Style Queries

A major challenge was answering navigation-style questions such as:

* "Where can I find the roadmap?"
* "Where is the changelog?"
* "Where are the docs?"

Page-only retrieval often misses these because the answer may live in site navigation, linked pages, or related pages that are not in the current page chunk set.

To improve this, we added a **host-level retrieval layer** on top of the normal page vector store.

As the user browses, TabSage extracts host signals from HTML, including:

* Internal links
* Anchor text
* Nearby context
* Page titles
* Headings

These signals are cached in Chrome storage and used to build a separate host-corpus vector index.

At query time, TabSage runs **dual retrieval**:

1. Search against the active-page chunks.
2. Search against the host corpus.

The results are then merged and deduplicated so the assistant can return better page suggestions and more accurate links.

### 2. Tradeoff: Better Navigation vs Higher Cost

This host-level retrieval is a pragmatic workaround, not a full site-understanding system. It does **not** store the full DOM tree. Instead, it stores compact summaries of visited pages and internal link candidates, plus seeded "hub" paths such as `/docs`, `/roadmap`, and `/changelog`.

The tradeoff is cost:

* More chunking and embeddings
* Larger retrieval corpus
* More prompt context
* Higher token usage

Future iterations should reduce this overhead with:

* Smarter summarization of host pages
* Tighter host-corpus limits
* Better link ranking
* Non-embedding strategies for navigation-style lookup

## Current Status / What Is Missing

The prototype demonstrates the core browsing assistant workflow, source-grounded answers, reusable notes, and token savings from page-level vector-store persistence.

Planned improvements include:

* Better ranking for site-navigation and docs discovery
* More aggressive token/cost optimization for host-level retrieval
* Improved UX around notes, citations, and MCQ generation
* Stronger evaluation across more websites and query types

---
**Demo Video**: [3-minute walkthrough](https://www.youtube.com/watch?v=hktVt-WFbmo){:target="_blank"}

**Project link**: [link to the deployed product](){:target="_blank"}

**Github/source code**: [TabSage source code](https://github.com/swactech/tabsage.git){:target="_blank"}
