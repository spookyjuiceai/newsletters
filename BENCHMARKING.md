# 📊 OpenClaw Memory Benchmarking Framework

> **SpookyJuice Research Initiative** — Establishing baseline metrics for persistent AI agent memory systems.

---

## Why This Matters

Nobody has published rigorous benchmarks for OpenClaw's memory system. Most documentation says things like "faster retrieval" or "better recall" with zero numbers behind them. This document establishes:

1. A **standard measurement methodology** anyone can run
2. **Baseline metrics** to compare against
3. A **community contribution framework** for submitting results
4. **Scoring rubrics** for memory quality (not just speed)

This is an open problem. We don't know the answers yet. That's the point.

---

## The Five Measurement Domains

```
Domain 1: SPEED     — How fast does memory work under load?
Domain 2: QUALITY   — Does the agent recall the right things?
Domain 3: COST      — What does memory actually cost?
Domain 4: DURABILITY — Does memory survive edge cases?
Domain 5: SCALE     — What happens as the corpus grows?
```

---

## Domain 1: Speed Benchmarks

### 1.1 Memory Write Throughput

**What we're measuring:** How many memory entries can be written per second.

**Test Protocol:**

```python
import time
import subprocess

def benchmark_memory_writes(num_entries=1000, entry_size_words=50):
    """
    Writes N memory entries and measures throughput.
    Run this inside an OpenClaw session via tool call.
    """
    entries = [
        f"Memory entry {i}: " + " ".join([f"word{j}" for j in range(entry_size_words)])
        for i in range(num_entries)
    ]

    start = time.perf_counter()
    for entry in entries:
        # Write to MEMORY.md or daily log
        append_to_memory(entry)  # your memory write function
    elapsed = time.perf_counter() - start

    throughput = num_entries / elapsed
    print(f"Writes: {num_entries}")
    print(f"Time: {elapsed:.2f}s")
    print(f"Throughput: {throughput:.1f} entries/sec")
    print(f"Avg latency: {(elapsed/num_entries)*1000:.1f}ms per write")

    return {
        "num_entries": num_entries,
        "entry_size_words": entry_size_words,
        "elapsed_s": elapsed,
        "throughput_eps": throughput,
        "avg_latency_ms": (elapsed / num_entries) * 1000
    }
```

**Benchmark targets:**

| Entry Size | Expected Throughput | Good | Excellent |
|-----------|---------------------|------|-----------|
| 50 words  | ~100 entries/sec    | >50  | >200      |
| 200 words | ~40 entries/sec     | >20  | >80       |
| 1000 words| ~10 entries/sec     | >5   | >20       |

---

### 1.2 Memory Retrieval Latency

**What we're measuring:** Time from query to returned results, across corpus sizes.

**Test Protocol:**

```bash
#!/bin/bash
# benchmark_retrieval.sh
# Run from inside your OpenClaw workspace

CORPUS_SIZES=(100 500 1000 5000 10000)
QUERY="What does the user prefer for breakfast?"

for SIZE in "${CORPUS_SIZES[@]}"; do
    echo "=== Corpus: $SIZE entries ==="

    # Time 10 queries, take median
    for i in {1..10}; do
        START=$(date +%s%N)
        # Your retrieval call here
        memory_search "$QUERY" --limit 5
        END=$(date +%s%N)
        echo "Query $i: $(( (END - START) / 1000000 ))ms"
    done

    echo ""
done
```

**Expected latency by corpus size:**

| Corpus Size | File-only | + SQLite FTS5 | + Vector | + Hybrid |
|-------------|-----------|---------------|----------|----------|
| 100 entries | 5ms       | 3ms           | 12ms     | 15ms     |
| 1,000       | 45ms      | 8ms           | 15ms     | 18ms     |
| 5,000       | 220ms     | 15ms          | 20ms     | 22ms     |
| 10,000      | 900ms     | 25ms          | 28ms     | 30ms     |
| 50,000      | >5s       | 80ms          | 45ms     | 50ms     |

> ⚠️ **Hypothesis (unverified):** Pure file-based retrieval degrades non-linearly past ~2,000 entries. This is why the community moved to SQLite + FTS5. We need actual data.

---

### 1.3 Embedding Generation Throughput

**What we're measuring:** How fast embeddings are created for new memory entries.

```python
def benchmark_embeddings(texts, model="text-embedding-3-small"):
    import openai, time

    client = openai.OpenAI()

    # Single embedding
    start = time.perf_counter()
    client.embeddings.create(input=texts[0], model=model)
    single_latency = (time.perf_counter() - start) * 1000

    # Batch embedding
    start = time.perf_counter()
    client.embeddings.create(input=texts[:100], model=model)
    batch_latency = (time.perf_counter() - start) * 1000

    print(f"Single embedding latency: {single_latency:.1f}ms")
    print(f"Batch (100) latency: {batch_latency:.1f}ms")
    print(f"Effective throughput: {100 / (batch_latency/1000):.0f} embeddings/sec")
```

**Known baselines (OpenAI text-embedding-3-small):**

| Operation | Latency | Cost per 1M tokens |
|-----------|---------|-------------------|
| Single embedding (~100 tokens) | 50–150ms | $0.02 |
| Batch 100 (~10K tokens) | 200–600ms | $0.02 |
| Batch 1000 (~100K tokens) | 1–3s | $0.02 |

---

## Domain 2: Memory Quality Benchmarks

> **This is the hard one.** Speed is easy to measure. Quality requires judgment.

### 2.1 Recall Accuracy Score (RAS)

**What we're measuring:** Does the agent surface the right memories given a query?

**Test Setup:**

1. Plant 50 "facts" into memory across a test session
2. After compaction, query for those facts using paraphrased questions
3. Score: fact retrieved correctly (1) / not retrieved (0)

```python
PLANTED_FACTS = [
    {
        "fact": "The user's name is Brian and they prefer morning meetings",
        "queries": [
            "What does the user prefer for meeting times?",
            "When should I schedule Brian's calls?",
            "Tell me about Brian's schedule preferences"
        ],
        "expected_in_results": True
    },
    # ... 49 more facts
]

def run_ras_benchmark(facts, memory_system):
    scores = []
    for item in facts:
        for query in item["queries"]:
            results = memory_system.search(query, limit=10)
            found = any(item["fact"].lower() in r.lower() for r in results)
            scores.append(1 if found == item["expected_in_results"] else 0)

    ras = sum(scores) / len(scores)
    print(f"Recall Accuracy Score: {ras:.1%}")
    return ras
```

**Baseline targets:**

| Memory System | Expected RAS | Notes |
|--------------|-------------|-------|
| File-only (MEMORY.md) | 60–75% | Degrades with corpus size |
| + Keyword search | 72–82% | Better exact-match recall |
| + Hybrid (BM25+Vector) | 82–91% | Best general recall |
| + Knowledge graph | 88–95% | Best for relational facts |

> ⚠️ **These are hypotheses.** No one has published actual numbers. If you run this test, **please submit your results.**

---

### 2.2 Temporal Accuracy Score (TAS)

**What we're measuring:** Does the agent remember *when* things happened, and prefer recent over stale?

```python
def run_tas_benchmark(memory_system):
    """
    Test: Plant conflicting facts at different times.
    The newer fact should win.
    """

    # Plant old fact
    memory_system.write("User's favorite color is blue", timestamp="2025-01-01")

    # Plant newer conflicting fact
    memory_system.write("User's favorite color is green", timestamp="2026-02-01")

    # Query
    results = memory_system.search("What is the user's favorite color?", limit=3)

    # Green should appear before blue
    green_rank = next((i for i, r in enumerate(results) if "green" in r.lower()), 999)
    blue_rank  = next((i for i, r in enumerate(results) if "blue"  in r.lower()), 999)

    correct = green_rank < blue_rank
    print(f"Temporal ordering correct: {correct}")
    print(f"Green rank: {green_rank}, Blue rank: {blue_rank}")
    return correct
```

---

### 2.3 Compaction Survival Rate (CSR)

**What we're measuring:** What percentage of important memories survive context compaction?

```python
def run_csr_benchmark(memory_system, num_sessions=10):
    """
    Plant facts before compaction.
    Trigger compaction.
    Query after compaction.
    Score what survived.
    """
    planted = []

    # Plant 20 "critical" facts
    for i in range(20):
        fact = f"Critical fact {i}: {generate_unique_fact()}"
        memory_system.write(fact)
        planted.append(fact)

    # Force compaction
    memory_system.trigger_compaction()

    # Check survival
    survived = 0
    for fact in planted:
        results = memory_system.search(fact[:30], limit=5)
        if any(fact[:30].lower() in r.lower() for r in results):
            survived += 1

    csr = survived / len(planted)
    print(f"Compaction Survival Rate: {csr:.1%} ({survived}/{len(planted)})")
    return csr
```

**Target CSR:**

| Config | Expected CSR |
|--------|-------------|
| No flush hook | 40–60% |
| With pre-compaction flush | 85–95% |
| With 5-layer protection (gavdalf) | 95–99% |

---

## Domain 3: Cost Benchmarks

### 3.1 Monthly Cost Model

**Formula:**

```
Monthly Memory Cost =
    (Embedding API calls × tokens × price_per_token) +
    (Storage costs) +
    (Consolidation/reflection agent calls) +
    (Vector DB hosting if applicable)
```

**Baseline cost estimates (small personal agent, ~100 interactions/day):**

| Component | Low | Mid | High |
|-----------|-----|-----|------|
| Embeddings (OpenAI small) | $0.01 | $0.05 | $0.20 |
| Consolidation agent (Gemini Flash) | $0.05 | $0.15 | $0.40 |
| Vector DB (Qdrant free tier) | $0 | $0 | $25+ |
| SQLite + file hosting | $0 | $2 | $10 |
| **Total** | **$0.06** | **$0.20** | **$35+** |

> Community reference: gavdalf's 5-layer protection system costs ~$0.10–0.20/month using Gemini Flash via OpenRouter.

---

## Domain 4: Durability Benchmarks

### 4.1 Failure Mode Matrix

| Failure Scenario | File-only | + SQLite | + 5-Layer Protection |
|-----------------|-----------|----------|---------------------|
| Process crash mid-write | ❌ Partial write | ✅ Atomic txn | ✅ Observer catches |
| Context compaction | ⚠️ 50% loss | ⚠️ 60% loss | ✅ 95%+ survival |
| Memory file corruption | ❌ Full loss | ⚠️ Index loss | ✅ Backup recovery |
| Large context (>100K tokens) | ❌ Degrades | ⚠️ Slower | ✅ Handled |
| Multi-session parallel writes | ❌ Race conditions | ✅ Locking | ✅ Queue-based |

---

## Domain 5: Scale Benchmarks

### 5.1 Performance Degradation Curve

**What we need:** Real measurements of how each memory system degrades as corpus grows.

**Suggested corpus sizes to test:** 100, 500, 1K, 5K, 10K, 50K, 100K entries.

**Metrics to record at each size:**
- Write latency (ms)
- Read latency (ms)
- Memory usage (MB)
- Index rebuild time (s)
- Embedding cache hit rate (%)

---

## How to Submit Your Benchmarks

### Benchmark Submission Template

Create a file: `docs/benchmarks/results/YOUR_HANDLE-YYYY-MM-DD.md`

```markdown
# Benchmark Results — [Your Handle] — [Date]

## Environment

| Spec | Value |
|------|-------|
| OS | macOS 14 / Ubuntu 22 / Windows 11 |
| CPU | Apple M3 Pro / Intel i9 / etc |
| RAM | 32GB |
| Storage | NVMe SSD |
| OpenClaw Version | v[X.X.X] |
| Memory System Config | File-only / +SQLite / +Vector / Hybrid |
| Embedding Model | text-embedding-3-small / etc |

## Results

### Domain 1: Speed

| Test | Result | Notes |
|------|--------|-------|
| Write throughput (50-word entries) | X entries/sec | |
| Read latency (1K corpus) | Xms | |
| Read latency (10K corpus) | Xms | |
| Embedding throughput | X/sec | |

### Domain 2: Quality

| Test | Score | Notes |
|------|-------|-------|
| Recall Accuracy Score (RAS) | X% | Corpus size: X |
| Temporal Accuracy Score (TAS) | Pass/Fail | |
| Compaction Survival Rate (CSR) | X% | Config: |

### Domain 3: Cost

| Component | Monthly Cost |
|-----------|-------------|
| Embeddings | $X |
| Consolidation | $X |
| Storage | $X |
| Total | $X |

## Notes & Observations

[Free text — what surprised you, edge cases, recommendations]
```

---

## Community Leaderboard (Hypothetical — Needs Data)

> These cells are empty because **nobody has run these benchmarks yet**. You could be first.

| Contributor | Memory Config | RAS | CSR | Latency @1K | Cost/mo |
|-------------|---------------|-----|-----|-------------|---------|
| *Submit yours* | — | — | — | — | — |

---

## Research Questions We Don't Have Answers To

1. **Does BM25 weight (30%) vs semantic weight (70%) hold up empirically?** Has anyone A/B tested different ratios?

2. **What's the actual pre-compaction flush accuracy?** Do all "critical" facts actually get written?

3. **At what corpus size does file-based memory become unusably slow?** Community folklore says ~2K entries but nobody has measured it.

4. **How does temporal decay scoring affect answer quality over long sessions?** Is recent always better?

5. **Can knowledge graphs actually improve recall for relational queries vs. flat vector search?** By how much?

6. **What's the minimum memory investment for a "good enough" personal AI agent?** Can you get 80% quality at $0.05/month?

---

## Join the Research Effort

If you run any of these benchmarks, post results in:

- **Twitter:** [@SpookyJuiceAI](https://x.com/SpookyJuiceAI) with `#SpookyBenchmarks`
- **Discord:** SpookyJuice.AI → `#benchmarks` channel
- **GitHub:** Submit a PR with your results file

**The goal:** Build the first publicly available, reproducible benchmark suite for OpenClaw persistent memory systems.

> *The ghost does not guess. The ghost measures.*
