---
layout: post
title: "From Paper to Production in 24 Hours"
date: 2026-03-24
excerpt: "A research paper dropped one evening. By the next morning, we'd studied the technique, adapted it for our stack, shipped it, tested it, threw out our first version, and shipped a better one. Here's how."
---

While researching memory architectures for AI agents, we found an academic paper published days earlier. A university-grade system with heavy infrastructure: graph databases, caching layers, formal belief revision semantics. The kind of thing a research lab builds over months.

One technique in the paper caught our attention: **prospective indexing**.

## The Problem It Solves

Vector search is how AI agents find old memories. You ask a question, the system finds notes with similar meaning. It works well when your question uses similar words to what was originally written.

But people don't always search the way they write. "Who's our first potential customer?" has zero word overlap with a note that says "She co-manages the farmer's market with the owner's wife." That note is exactly the answer, but vector search will never find it.

The paper's solution: when you store a memory, also generate hypothetical future search phrases. Index those alongside the original content. When someone later searches with different vocabulary, the pre-generated phrases bridge the gap.

## What We Did

The academic system required enterprise-grade infrastructure to implement this. Multiple database layers, caching, a multi-stage pipeline. We looked at the core insight and asked: what's the simplest possible version that gets the same result?

**First pass:** We built a lightweight script that processes notes and generates future search phrases using a small AI model. Bolted it onto our existing search infrastructure. No new databases, no new services.

**First test:** The quality was mixed. About 70% useful, 30% generic filler. The model we chose was cheap and fast but not sharp enough for the task.

**Second pass:** We swapped in a better model and the difference was night and day. Instead of generic business phrases, we got genuinely useful bridges that connected how memories were written to how they'd actually be searched for.

**The decision:** Rip out the first approach entirely. Replace it with a single nightly job. No extra software, no extra cost. One scheduled task that runs while everyone's asleep.

**Validation:** We tested a search query using vocabulary that appeared nowhere in the original notes. Top result: exactly the right note, surfaced entirely through the prospective bridge. It worked.

## What We Shipped

A nightly automated task that enriches our agents' memory with future search phrases. The entire system runs on existing infrastructure with zero additional services.

The academic system needs enterprise-grade tooling. Ours needs what we already had plus one good idea.

## What We Learned

The most impactful discovery wasn't the technique itself. It was a bug we found while testing it.

Our search index had silently fallen behind. Every search for the past two days was missing recent content. Not just the new prospective data. Everything.

We added a simple periodic rebuild. The fix was trivial. The fact that we caught it was not.

Nobody in the research literature talks about stale indexes. Every paper assumes the index is current. In production, the boring operational fix mattered more than the novel retrieval technique.

## The Takeaway

You don't need to build what the research labs build. You need to understand what they discovered and find the simplest way to get the same result.

A graph database is one way to bridge the gap between how memories are stored and how they're searched. An invisible comment in a text file is another. The insight is the same. The implementation doesn't have to be.

Read a paper. Shipped to production the next day. That's the pace that matters.
