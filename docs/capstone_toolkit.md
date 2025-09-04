# Beginner’s Toolkit: Automated LinkedIn Content Engine with GenAI

## 1. Objective
Use Generative AI to learn, build, and document a runnable system that converts AI-automation news into LinkedIn-ready copy with a human approval step and scheduled publishing.

## 2. Overview of the Technology
- **Orchestration:** Make.com and n8n (no/low-code automation)
- **Research:** RSS + Perplexity API
- **Authoring:** OpenAI (GPT-4.1) with strict JSON prompts
- **Approval:** Google Sheets
- **Publishing:** Buffer (or LinkedIn API where approved)

## 3. Minimal Example (Runbook)
1. Run the **Research** workflow once → verify new rows in Google Sheets with `Status = Pending`.
2. Edit copy if needed → set `Status = Approved`.
3. Run the **Posting** workflow (or let it schedule) → confirm LinkedIn post appears and the sheet updates `LinkedInPostId` and `PostedAt`.

## 4. Setup (Step by Step)

### 4.1 Accounts & Connections
- Make.com / n8n, Google Sheets, OpenAI, Perplexity, Buffer (LinkedIn Company connected).

### 4.2 Google Sheet
Create a sheet with columns:

Title | URL | Source | Summary | PostDraft | Hashtags | ImageIdea | Status | CreatedAt | LinkedInPostId | PostedAt


### 4.3 Import Blueprints
- Make.com:  
  - `blueprints/make/make.com LinkedIn Posts Research.blueprint.json`  
  - `blueprints/make/make.com Post to LinkedIn blueprint.json`
- n8n (optional):  
  - `blueprints/n8n/LinkedIn Posts Research_n8n_blueprint.json`  
  - `blueprints/n8n/Post on LinkedIn_n8n_blueprint.json`

### 4.4 Configure Nodes (high-level)
- **RSS** or **Perplexity** → JSON array.  
- **HTTP Request** → fetch article (response as **String**).  
- **Clean text** → remove tags, collapse whitespace.  
- **OpenAI – Brief Builder** (temp 0.3): returns `{topic, views_on_topic, target_audience[], hashtags_seed[]}`.  
- **OpenAI – LinkedIn Writer** (temp 0.5–0.7): returns `{post_text, hashtags, image_idea}`.  
- **Append to Google Sheets**: `Status = Pending`.  
- **Post workflow**: filter `Status = Approved` and `LinkedInPostId` empty → Buffer publish → update row.

## 5. Prompt Journal

### 5.1 Perplexity (JSON array only)
**System**

You are a research assistant returning strictly valid JSON array. No prose. No code fences.

**User**

Find the 10 most recent, high-quality articles or case studies in the last 30 days on AI automation, lead generation, voice agents, data scraping, and workflow automation.
Return strictly a JSON array of:
\[{ "title": "...", "url": "...", "date": "YYYY-MM-DD", "source": "domain.com" }]
Exclude spam/AI farms. Authoritative sources only.

### 5.2 OpenAI — Brief Builder (GPT-4.1, temp 0.3)
**System**
```

You are a concise marketing research assistant. Use UK English.
Return strict JSON only (no code fences, no commentary).

```
**User**
```

INPUTS
article\_title: "{{article\_title}}"
article\_summary\_optional: "{{article\_summary}}"

TASK

1. topic: short, compelling phrase (<= 12 words).
2. views\_on\_topic: 2–3 expert sentences (no fluff).
3. target\_audience: 3–5 concise audience segments.
4. hashtags\_seed: 4–6 niche hashtags (no #ai, #automation, #business).

FORMAT
{
"topic": "string",
"views\_on\_topic": "string",
"target\_audience": \["segment1","segment2","segment3"],
"hashtags\_seed": \["#tag1","#tag2","#tag3","#tag4"]
}

```

### 5.3 OpenAI — LinkedIn Post Writer (GPT-4.1, temp 0.5–0.7)
**System**
```

You are a copywriter with strong LinkedIn skills. Use UK English.
Be extremely confident and creative. Prefer short sentences and sentence fragments.
Return strict JSON only (no code blocks, no commentary).

```
**User**
```

CONTEXT: You are a copywriter specialised in captivating attention on LinkedIn.

ACTION: Create a LinkedIn post on the topic that resonates with the target audience. Provide one image idea.
Hook must push emotional buttons and retain readers.

Topic: \[{{topic}}]
My Views on topic: \[{{views\_on\_topic}}]
Target audience: \[{{target\_audience}}]

SPECIFICATIONS:
— 130–170 words (cap 175).
— Hook ≤ 22 words; 2–3 insights; 1 actionable takeaway; a soft CTA.
— 3–5 niche hashtags (avoid generic #ai, #automation). Include #KiDaFlow if relevant.
— Structure: Hook → Insights → Takeaway → CTA → Hashtags.

Return strict JSON:
{
"post\_text": "string",
"hashtags": \["#tag1","#tag2","#tag3","#KiDaFlow"],
"image\_idea": "one sentence (no text on image)"
}

```

### 5.4 Prompt — Troubleshooting JSON Fences
**System**
```

You are a strict JSON formatter. Input may include code fences like `json ... `.
Remove all code fences and return the JSON only. No commentary.

```
**User**
```

The model output contains leading `json and trailing `.
Strip them and return clean JSON only.
If invalid, reformat into valid JSON.

```

### 5.5 Prompt — Debugging Looping / Splitting Arrays
**System**
```

You are a workflow debugger for n8n.
I will paste JSON output from a node.
Your task: explain if it's an array or object, and show me how to split it into multiple items.

```
**User**
```

Given this JSON:
{"search\_results": \[{"title":"…","url":"…"},{"title":"…","url":"…"}]}
Tell me how to use "Item Lists → Split Out Items" so that each result becomes one item downstream.

```

### 5.6 Prompt — Content Strategy Research
**System**
```

You are a content strategist.
Advise on how to source high-quality AI automation content for LinkedIn repurposing.
Give clear sources (RSS feeds, APIs, aggregators).

```
**User**
```

I need to set up an automation that creates LinkedIn posts about AI automation, lead generation, voice agents, data scraping, and workflow automation.
What are the best RSS feeds, APIs, or sources to use, and how can I combine them with a model like Perplexity for breadth + depth?

```

### 5.7 Prompt — Repurposing for LinkedIn
**System**
```

You are a LinkedIn growth copywriter.
Your task is to take curated content from research and show me multiple ways to reframe it for LinkedIn.

```
**User**
```

Given an article about workflow automation saving time and money:

1. Write a narrative post.
2. Write a punchy listicle.
3. Write a contrarian take.
   Return 3 short drafts.

````

---

## 6. Testing

- **Unit:** Run research once; verify clean text > 600 chars; check JSON outputs parse.
- **Integration:** Drafts appear in Google Sheets; Approval flips to Approved; Posting workflow writes back `LinkedInPostId`.
- **Schedules:** Example cron (n8n) Mon & Wed 10:27 → `27 10 * * 1,3`.

## 7. Troubleshooting & Fixes

- **RSS digest with many links:** split embedded items with regex or code; iterate each.
- **Perplexity fences:** strip ```json/``` before parsing, or enforce “no fences”.
- **Text parser errors (“Buffer can’t be converted to text”):** ensure HTTP response is **String**.
- **Only one item from array (n8n):** use **Item Lists → Split Out Items** before downstream nodes.
- **LinkedIn API approvals:** if not granted, use **Buffer** as the publishing path.

## 8. Reflections & Learning

- Prompt design and strict JSON outputs dramatically improved reliability.
- No-code tools reduce boilerplate, but **data hygiene** (HTML→text, arrays→items) is crucial.
- A simple **human approval** step prevents brand mishaps and supports iteration.
- This pipeline maps directly to KiDaFlow’s goals: repeatable research + content generation that feeds lead generation.

## 9. References

- Make.com, n8n, OpenAI, Perplexity, Buffer product documentation.
- Course capstone guidance (deliverables, format, and evaluation criteria).
