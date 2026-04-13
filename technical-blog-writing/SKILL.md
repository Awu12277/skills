---
name: technical-blog-writing
description: "Technical blog post writing with structure, code examples, and developer audience conventions. Covers post types, code formatting, explanation depth, and developer-specific engagement patterns. Use for: engineering blogs, dev tutorials, technical writing, developer content, documentation posts. Triggers: technical blog, dev blog, engineering blog, technical writing, developer tutorial, tech post, code tutorial, programming blog, developer content, technical article, engineering post, coding tutorial, technical content"
allowed-tools: Bash(infsh *)
---

# Technical Blog Writing

写给开发者看的技术博客，通过 [inference.sh](https://inference.sh) CLI 来完成。

> 说人话，别端着。这篇文章就是例子。

## Quick Start

> Requires inference.sh CLI (`infsh`). [Install instructions](https://raw.githubusercontent.com/inference-sh/skills/refs/heads/main/cli-install.md)

```bash
infsh login

# Research topic depth
infsh app run exa/search --input '{
  "query": "building REST API Node.js best practices 2024 tutorial"
}'

# Generate header image
infsh app run infsh/html-to-image --input '{
  "html": "<div style=\"width:1200px;height:630px;background:linear-gradient(135deg,#0f172a,#1e293b);display:flex;align-items:center;padding:60px;font-family:ui-monospace,monospace;color:white\"><div><p style=\"font-size:18px;color:#38bdf8;margin:0\">// engineering blog</p><h1 style=\"font-size:48px;margin:16px 0;font-weight:800;font-family:system-ui;line-height:1.2\">How We Reduced API Latency by 90% with Edge Caching</h1><p style=\"font-size:20px;opacity:0.6;font-family:system-ui\">A deep dive into our CDN architecture</p></div></div>"
}'
```


## Post Types

### 1. Tutorial / How-To

Step-by-step instruction. The reader should be able to follow along and build something.

```
Structure:
1. What we're building (with screenshot/demo)
2. Prerequisites
3. Step 1: Setup
4. Step 2: Core implementation
5. Step 3: ...
6. Complete code (GitHub link)
7. Next steps / extensions
```

| Rule | Why |
|------|-----|
| Show the end result first | Reader knows if it's worth continuing |
| List prerequisites explicitly | Don't waste time of wrong audience |
| Every code block should be runnable | Copy-paste-run is the test |
| Explain the "why" not just the "how" | Tutorials that explain reasoning get shared |
| Include error handling | Real code has errors |
| Link to complete code repo | Reference after tutorial |

### 2. Deep Dive / Explainer

Explains a concept, technology, or architecture decision in depth.

```
Structure:
1. What is [concept] and why should you care?
2. How it works (simplified mental model)
3. How it works (detailed mechanics)
4. Real-world example
5. Trade-offs and when NOT to use it
6. Further reading
```

### 3. Postmortem / Incident Report

Describes what went wrong, why, and what was fixed.

```
Structure:
1. Summary (what happened, impact, duration)
2. Timeline of events
3. Root cause analysis
4. Fix implemented
5. What we're doing to prevent recurrence
6. Lessons learned
```

### 4. Benchmark / Comparison

Data-driven comparison of tools, approaches, or architectures.

```
Structure:
1. What we compared and why
2. Methodology (so results are reproducible)
3. Results with charts/tables
4. Analysis (what the numbers mean)
5. Recommendation (with caveats)
6. Raw data / reproducibility instructions
```

### 5. Architecture / System Design

Explains how a system is built and why decisions were made.

```
Structure:
1. Problem we needed to solve
2. Constraints and requirements
3. Options considered
4. Architecture chosen (with diagram)
5. Trade-offs we accepted
6. Results and lessons
```

## Writing Rules for Developers

### Voice and Tone

| Do | Don't |
|----|-------|
| Be direct: "用连接池" | "你可能需要考虑使用..." |
| 说人话，像在跟同事聊天 | 写得像政府报告 |
| 用"我/我们"，带点个人色彩 | "本项目经过严谨论证..." |
| 具体数字: "p99 从 800ms 降到 90ms" | "性能显著提升" |
| 承认trade-offs: "这会让复杂度上升" | 假装方案完美无缺 |
| 可以吐槽，可以调侃，可以玩梗 | 正襟危坐，滴水不漏 |
| 结尾可以说"有问题随时留言" | "如需进一步信息，请联系..." |

### What Developers Hate

```
❌ "在当今飞速发展的技术世界..." (废话开场)
❌ "众所周知..." (既然众所周知，还写它干嘛？)
❌ "简单做 X" (对新手来说没什么是简单的)
❌ "很容易就能..." (读者骂你一句"你说的轻巧")
❌ "很明显..." (明显的话就不用写了)
❌ 营销话术："引领"、"赋能"、"闭环"、"打法"
❌ 开头堆砌3段背景才到正题
❌ "该方案经过精心设计..." (像在写PPT)
❌ 中英混杂："这个function很cool" (要嘛全中文，要嘛全英文)
❌ 标题党："你绝对不知道的X个秘诀！" (技术人不吃这套)
❌ 每句话都像在写年终总结
```

### Code Examples

| Rule | Why |
|------|-----|
| Every code block must be runnable | Broken examples destroy trust |
| Show complete, working examples | Snippets without context are useless |
| Include language identifier in fenced blocks | Syntax highlighting |
| Show output/result after code | Reader verifies understanding |
| Use realistic variable names | `calculateTotalRevenue` not `foo` |
| Include error handling in examples | Real code handles errors |
| Pin dependency versions | "Works with React 18.2" not "React" |

```
Good code block format:

```python
# What this code does (one line)
def calculate_retry_delay(attempt: int, base_delay: float = 1.0) -> float:
    """Exponential backoff with jitter."""
    delay = base_delay * (2 ** attempt)
    jitter = random.uniform(0, delay * 0.1)
    return delay + jitter

# Usage
delay = calculate_retry_delay(attempt=3)  # ~8.0-8.8 seconds
```
```

### Explanation Depth

| Audience Signal | Depth |
|----------------|-------|
| "Getting started with X" | Explain everything, assume no prior knowledge |
| "Advanced X patterns" | Skip basics, go deep on nuances |
| "X vs Y" | Assume familiarity with both, focus on differences |
| "How we built X" | Technical audience, can skip fundamentals |

**State your assumed audience level explicitly** at the start:

```
"This post assumes familiarity with Docker and basic Kubernetes concepts.
If you're new to containers, start with [our intro post]."
```

## Blog Post Structure

### The Ideal Structure

```markdown
# Title (contains primary keyword, states outcome)

[Hero image or diagram]

**TL;DR:** [2-3 sentence summary with key takeaway]

## The Problem / Why This Matters
[Set up why the reader should care — specific, not generic]

## The Solution / How We Did It
[Core content — code, architecture, explanation]

### Step 1: [First thing]
[Explanation + code + output]

### Step 2: [Second thing]
[Explanation + code + output]

## Results
[Numbers, benchmarks, outcomes — be specific]

## Trade-offs and Limitations
[Honest about downsides — builds trust]

## Conclusion
[Key takeaway + what to do next]

## Further Reading
[3-5 relevant links]
```

### Word Count by Type

| Type | Word Count | Why |
|------|-----------|-----|
| Quick tip | 500-800 | One concept, one example |
| Tutorial | 1,500-3,000 | Step-by-step needs detail |
| Deep dive | 2,000-4,000 | Thorough exploration |
| Architecture post | 2,000-3,500 | Diagrams carry some load |
| Benchmark | 1,500-2,500 | Data and charts do heavy lifting |

## Diagrams and Visuals

### When to Use Diagrams

| Scenario | Diagram Type |
|----------|-------------|
| Request flow | Sequence diagram |
| System architecture | Box-and-arrow diagram |
| Decision logic | Flowchart |
| Data model | ER diagram |
| Performance comparison | Bar/line chart |
| Before/after | Side-by-side |

```bash
# Generate architecture diagram
infsh app run infsh/html-to-image --input '{
  "html": "<div style=\"width:1200px;height:600px;background:#0f172a;display:flex;align-items:center;justify-content:center;padding:40px;font-family:system-ui;color:white\"><div style=\"display:flex;gap:40px;align-items:center\"><div style=\"background:#1e293b;border:2px solid #334155;border-radius:8px;padding:24px;text-align:center;width:160px\"><p style=\"font-size:14px;color:#94a3b8;margin:0\">Client</p><p style=\"font-size:18px;font-weight:bold;margin:8px 0 0\">React App</p></div><div style=\"color:#64748b;font-size:32px\">→</div><div style=\"background:#1e293b;border:2px solid #3b82f6;border-radius:8px;padding:24px;text-align:center;width:160px\"><p style=\"font-size:14px;color:#94a3b8;margin:0\">Edge</p><p style=\"font-size:18px;font-weight:bold;margin:8px 0 0\">CDN Cache</p></div><div style=\"color:#64748b;font-size:32px\">→</div><div style=\"background:#1e293b;border:2px solid #334155;border-radius:8px;padding:24px;text-align:center;width:160px\"><p style=\"font-size:14px;color:#94a3b8;margin:0\">API</p><p style=\"font-size:18px;font-weight:bold;margin:8px 0 0\">Node.js</p></div><div style=\"color:#64748b;font-size:32px\">→</div><div style=\"background:#1e293b;border:2px solid #334155;border-radius:8px;padding:24px;text-align:center;width:160px\"><p style=\"font-size:14px;color:#94a3b8;margin:0\">Database</p><p style=\"font-size:18px;font-weight:bold;margin:8px 0 0\">PostgreSQL</p></div></div></div>"
}'

# Generate benchmark chart
infsh app run infsh/python-executor --input '{
  "code": "import matplotlib.pyplot as plt\nimport matplotlib\nmatplotlib.use(\"Agg\")\n\nfig, ax = plt.subplots(figsize=(12, 6))\nfig.patch.set_facecolor(\"#0f172a\")\nax.set_facecolor(\"#0f172a\")\n\ntools = [\"Express\", \"Fastify\", \"Hono\", \"Elysia\"]\nrps = [15000, 45000, 62000, 78000]\ncolors = [\"#64748b\", \"#64748b\", \"#3b82f6\", \"#64748b\"]\n\nax.barh(tools, rps, color=colors, height=0.5)\nfor i, v in enumerate(rps):\n    ax.text(v + 1000, i, f\"{v:,} req/s\", va=\"center\", color=\"white\", fontsize=14)\n\nax.set_xlabel(\"Requests per second\", color=\"white\", fontsize=14)\nax.set_title(\"HTTP Framework Benchmark (Hello World)\", color=\"white\", fontsize=18, fontweight=\"bold\")\nax.tick_params(colors=\"white\", labelsize=12)\nax.spines[\"top\"].set_visible(False)\nax.spines[\"right\"].set_visible(False)\nax.spines[\"bottom\"].set_color(\"#334155\")\nax.spines[\"left\"].set_color(\"#334155\")\nplt.tight_layout()\nplt.savefig(\"benchmark.png\", dpi=150, facecolor=\"#0f172a\")\nprint(\"Saved\")"
}'
```

## Distribution

### Where Developers Read

| Platform | Format | How to Post |
|----------|--------|-------------|
| Your blog | Full article | Primary — own your content |
| Dev.to | Cross-post (canonical URL back to yours) | Markdown import |
| Hashnode | Cross-post (canonical URL) | Markdown import |
| Hacker News | Link submission | Show HN for projects, tell HN for stories |
| Reddit (r/programming, r/webdev, etc.) | Link or discussion | Follow subreddit rules |
| Twitter/X | Thread summary + link | See twitter-thread-creation skill |
| LinkedIn | Adapted version + link | See linkedin-content skill |

```bash
# Cross-post thread to X
infsh app run x/post-create --input '{
  "text": "New blog post: How We Reduced API Latency by 90%\n\nThe short version:\n→ Moved computation to edge\n→ Aggressive cache-control headers\n→ Eliminated N+1 queries\n\np99 went from 800ms to 90ms.\n\nFull deep dive with code: [link]"
}'
```

## Common Mistakes

| Mistake | Problem | Fix |
|---------|---------|-----|
| 没有 TL;DR | 读者划走看不到重点 | 开头2-3句话总结核心结论 |
| 代码跑不通 | 信任度直接归零 | 发之前一定跑一遍 |
| 不写版本号 | 三个月后代码废了 | "用的是 Node 20 + React 18.2" |
| "轻松实现 X" | 读者心里骂你 | 去掉"简单"、"容易"、"轻松" |
| 架构文章没有图 | 500字描述不如一张图 | 随手画个架构图 |
| 营销号语气 | 程序员直接关页面 | 有话直说，技术人喜欢实在的 |
| 不聊trade-offs | 看起来像软文 | 坦诚说哪里不好 |
| 开场堆一堆背景 | 读者早就跑了 | 两三段话直接进主题 |
| 依赖版本不锁定 | 教程变成废教程 | 写清楚版本，写清楚日期 |
| 结尾留个死链接 | 没有下文了 | 甩3-5个延伸阅读 |

## Related Skills

```bash
npx skills add inference-sh/skills@seo-content-brief
npx skills add inference-sh/skills@content-repurposing
npx skills add inference-sh/skills@og-image-design
```

Browse all apps: `infsh app list`

