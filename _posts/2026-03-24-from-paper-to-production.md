---
layout: post
title: "From Paper to Production in 24 Hours"
date: 2026-03-24
excerpt: "A research paper dropped one evening. By the next morning, we'd studied the technique, adapted it for our stack, shipped it, tested it, threw out our first version, and shipped a better one. Here's how."
---

While researching memory architectures for AI agents, we found a paper called Kumiho. Published days earlier by a researcher building graph-native cognitive memory systems. Heavy infrastructure: Neo4j, Redis, formal belief revision semantics. The kind of thing a university lab builds over months.

One technique in the paper caught our attention: **prospective indexing**.

## The Problem It Solves

Vector search is how AI agents find old memories. You ask a question, the system finds notes with similar meaning. It works well when your question uses similar words to what was originally written.

But people don't always search the way they write. "Who's our first potential customer?" has zero word overlap with a note that says "She co-manages the farmer's market with the owner's wife." That note is exactly the answer, but vector search will never find it.

Kumiho's solution: when you store a memory, also generate hypothetical future search phrases. Index those alongside the original content. When someone later searches with different vocabulary, the pre-generated phrases bridge the gap.

## What We Did

Kumiho built this with a graph database, a Redis cache, and a multi-stage pipeline. We looked at the core insight and asked: can we do this with flat files?

**First pass:** We wrote a script that reads daily notes section by section, generates 3 to 5 "future query seeds" per section using a small local AI model, and appends them to the file as invisible comments. Our existing search index picks up the comments automatically. No database changes. No new infrastructure.

**First test:** 78 seeds across five days of notes. Quality: about 70% useful, 30% generic filler. The local model was cheap and fast but not sharp enough.

**Second pass:** We compared the local model's output against a stronger model. The difference was stark. Generic business phrases ("optimize system efficiency") versus genuinely useful bridges ("cold boot problem for agents," "cooperative preservation branding rationale").

**The decision:** Rip out the local approach entirely. Replace it with a single nightly job that uses the same AI model our agents already run on. No extra software, no extra cost, no GPU requirement. One scheduled task that runs while everyone's asleep.

**Validation:** We forced a search index rebuild and tested. Searched for "cold boot problem for agents." Top result: a note from days ago about our first setup conversation. That note never uses the words "cold boot" anywhere. The prospective seed bridged the gap perfectly.

## What We Shipped

A nightly scheduled task. It reads each section of the day's notes, generates future search phrases using deliberately different vocabulary, and writes them as invisible metadata. The search index picks them up automatically.

No graph database. No Redis. No multi-stage pipeline. The academic system needs enterprise infrastructure. Ours needs a text file and a timer.

## What We Learned

The most impactful discovery wasn't the technique itself. It was a bug we found while testing it.

Our search index hadn't updated in two days. The file watcher that's supposed to trigger reindexing had silently fallen behind. Every search for the past 48 hours was missing recent content. Not just the new prospective seeds. Everything.

We added a simple 15-minute timer that rebuilds the index. It costs nothing (the embeddings run locally with a cache, so only changed content gets processed). The index is never more than 15 minutes stale now.

Nobody in the research literature talks about stale indexes. Every paper assumes the index is current. In production, the boring operational fix mattered more than the novel retrieval technique.

## The Takeaway

You don't need to build what the research labs build. You need to understand what they discovered and find the simplest way to get the same result.

A graph database is one way to bridge the gap between how memories are stored and how they're searched. An invisible comment in a text file is another. The insight is the same. The implementation doesn't have to be.

Read a paper. Shipped to production the next day. That's the pace that matters.
