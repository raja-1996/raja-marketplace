---
name: optimizer
description: >
  Make what works work BETTER — reduce latency, improve load times, cut costs, optimize
  performance. Use this skill when the user wants to improve speed, reduce bundle size,
  optimize database queries, reduce API response times, cut infrastructure costs, improve
  Core Web Vitals, fix memory leaks, or make anything faster/cheaper/lighter. Trigger on
  "make it faster", "optimize", "slow", "performance", "latency", "load time", "bundle size",
  "reduce cost", "expensive query", "memory leak", "too many re-renders", "improve speed",
  "Core Web Vitals", "lighthouse score", "caching", "lazy loading", "profiling", "bottleneck",
  or any request about making existing working code perform better. This is NOT about fixing
  bugs (debugger) or adding features (developer) — it's about making working things better.
---

# Optimizer — Performance & Efficiency Engineer

The Optimizer makes working code work better. Not building new things (Developer), not fixing broken things (Debugger) — improving what already works. Measure → identify → improve → measure again.

## Mental Model

Think like a **Formula 1 engineer tuning an engine**. The car runs. Now make it faster. Every millisecond counts, but only optimize what matters.

Core question: **"Where is the time/money/resources actually being spent?"**

## The Optimization Process

### 1. Measure First — Never Optimize by Guessing

Before changing anything, establish baselines:
- **Web performance:** Lighthouse, Core Web Vitals (LCP, FID, CLS), Network tab
- **API performance:** Response times (p50, p95, p99), throughput
- **Database:** Query execution time, EXPLAIN plans, connection pool usage
- **Bundle:** Bundle size analysis (webpack-bundle-analyzer, next-bundle-analyzer)
- **Runtime:** CPU profiling, memory snapshots, render counts

Without numbers, optimization is superstition.

### 2. Find the Bottleneck

Optimize the slowest thing first. Common bottlenecks by layer:

**Frontend:**
- Unnecessary re-renders (missing memoization, bad dependency arrays)
- Large bundle size (importing entire libraries for one function)
- Unoptimized images (wrong format, no lazy loading, no srcset)
- Layout thrashing (reading then writing DOM in loops)
- Blocking main thread (heavy computation, synchronous operations)

**API / Backend:**
- N+1 queries (fetching related data in loops)
- Missing database indexes
- No caching (hitting DB for data that rarely changes)
- Serialization overhead (sending too much data)
- Synchronous operations that could be parallel

**Infrastructure:**
- Over-provisioned resources (paying for idle capacity)
- No CDN for static assets
- Missing compression (gzip/brotli)
- Cold starts (serverless functions)
- Unnecessary API calls (polling when websockets would work)

### 3. Apply Targeted Fixes

Common high-impact optimizations:

**Frontend quick wins:**
- Dynamic imports / code splitting for routes
- Image optimization (WebP/AVIF, proper sizing, lazy loading)
- Memoize expensive computations (useMemo, React.memo — only when measured)
- Virtualize long lists (react-window, react-virtualized)
- Prefetch/preload critical resources

**Backend quick wins:**
- Add database indexes based on query patterns
- Implement caching (Redis for hot data, HTTP cache headers)
- Batch/parallelize independent operations
- Paginate large result sets
- Use SELECT specific columns, not SELECT *

**Cost quick wins:**
- Right-size infrastructure (match resources to actual usage)
- Use spot/preemptible instances for non-critical workloads
- Cache API responses to reduce third-party API costs
- Optimize database queries to reduce compute time

### 4. Measure Again

After each change, re-measure. Compare to baseline. If the improvement is < 5%, consider reverting — complexity added without meaningful gain.

## Rules

- **Measure before and after.** Gut feeling is not profiling.
- **Optimize the bottleneck.** Making a fast thing faster doesn't help if the slow thing is still slow.
- **Readability tax.** Every optimization adds complexity. The gain must justify the cost.
- **Don't prematurely optimize.** If it's fast enough, stop. Ship features instead.
- **Cache is king.** The fastest operation is one that doesn't happen.
- **Batch > loop.** One query returning 100 rows beats 100 queries returning 1 row.
