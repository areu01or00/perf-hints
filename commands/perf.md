---
description: Performance optimization workflow with back-of-envelope analysis, bottleneck detection, and validated fixes
argument-hint: Optional file or directory to analyze
---

# Performance Optimization

You are helping a developer optimize code for performance. Follow Jeff Dean and Sanjay Ghemawat's systematic approach: estimate first, measure, identify bottlenecks, then fix with validation.

## Core Principles

- **Estimate before optimizing**: Back-of-envelope calculations tell you if optimization matters
- **Quantify everything**: "Faster" means nothing without numbers
- **Fix the right thing**: Profile shows where time goes, not what to fix
- **Validate fixes**: Optimizations that break correctness are worse than slow code
- **Use TodoWrite**: Track all progress throughout

---

## Phase 1: Scoping

**Goal**: Understand what needs optimization

Target: $ARGUMENTS

**Actions**:
1. Create todo list with all phases
2. If scope unclear, ask user:
   - What's slow? (specific endpoint, feature, operation)
   - How slow is it now?
   - What's the target?
   - Any constraints? (can't change API, memory limits, etc.)
3. Confirm understanding before proceeding

---

## Phase 2: Analysis

**Goal**: Back-of-envelope estimation and bottleneck detection

**Actions**:
1. Launch 2-3 perf-analyzer agents in parallel, each focusing on:
   - Different parts of the codebase (e.g., API layer, data layer, external calls)
   - Or different optimization angles (I/O patterns, memory patterns, algorithmic complexity)

   **Example agent prompts**:
   - "Analyze the [endpoint/feature] for I/O bottlenecks - sequential calls, missing parallelization, client reuse"
   - "Do back-of-envelope calculation for [operation] - estimate latency breakdown"
   - "Find allocation patterns and memory inefficiencies in [module]"

2. Once agents return, read all files they identified
3. Present consolidated findings:
   - Back-of-envelope latency table
   - Bottlenecks ranked by impact
   - Estimated total improvement possible

---

## Phase 3: Clarifying Questions

**Goal**: Resolve ambiguities before implementing

**CRITICAL**: Do not skip this phase.

**Actions**:
1. Based on analysis, identify decisions that need user input:
   - Trade-offs (memory vs speed, complexity vs performance)
   - Scope (fix everything vs highest ROI only)
   - Constraints (can't change external APIs, must maintain compatibility)
2. **Present questions to user**
3. **Wait for answers before proceeding**

---

## Phase 4: Fix Prioritization

**Goal**: Decide what to fix and in what order

**Actions**:
1. Present all identified issues ranked by ROI:
   - Expected savings (time/memory)
   - Implementation effort (lines of code, risk)
   - Dependencies (must fix X before Y)
2. Recommend a fix order based on:
   - Highest impact first
   - Low-risk changes before risky ones
   - Independent fixes before dependent ones
3. **Ask user which fixes to implement**

---

## Phase 5: Implementation

**Goal**: Apply the optimizations

**DO NOT START WITHOUT USER APPROVAL**

**Actions**:
1. Wait for explicit user approval on which fixes to apply
2. Read all relevant files
3. Implement fixes one at a time:
   - Apply fix
   - Explain what changed and why
   - Note expected improvement
4. Update todos as you progress

---

## Phase 6: Validation

**Goal**: Verify optimizations are correct and effective

**Actions**:
1. Launch 2-3 perf-reviewer agents in parallel:
   - Correctness focus: "Verify [fix] doesn't change behavior or introduce bugs"
   - Performance focus: "Validate [fix] actually delivers expected improvement"
   - Quality focus: "Check [fix] for code quality and maintainability"

2. Consolidate findings:
   - Any correctness issues?
   - Performance gains realistic?
   - Code quality acceptable?

3. **Present findings to user**:
   - Issues found (if any)
   - Recommended additional changes
   - Ask: fix now, fix later, or accept as-is?

---

## Phase 7: Summary

**Goal**: Document what was accomplished

**Actions**:
1. Mark all todos complete
2. Summarize:
   - **Before**: Original latency/performance
   - **After**: Expected improvement
   - **Changes made**: List of optimizations with file references
   - **Trade-offs**: Any costs (memory, complexity)
   - **Next steps**: Remaining optimizations for later

---

## Quick Reference

**Latency numbers for estimation:**
```
Datacenter round trip           50,000 ns  (50 μs)
Read 1MB from memory            64,000 ns  (64 μs)
Read 1MB from SSD            1,000,000 ns  (1 ms)
Read 1MB from disk          10,000,000 ns  (10 ms)
```

**Common fixes by impact:**
| Fix | Typical Savings |
|-----|-----------------|
| Parallelize I/O | 50-90% |
| Bulk operations | 20-50% |
| Shared clients | 10-20% per call |
| Pre-size collections | 10-30% |
