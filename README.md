# Prompt-Powered LinkedIn Engine (Make.com + n8n)

Turn AI-automation news into LinkedIn-ready posts with a human approval gate and scheduled publishing.

**Stack:** Make.com / n8n • OpenAI (GPT-4.1) • Perplexity • Google Sheets • Buffer/LinkedIn

**Loom walkthrough:** https://www.loom.com/share/075c0315114c4ea982d3fff5d69354d4?sid=261545b7-0248-4574-80d2-8e56ea8f40e9


## What this toolkit does

1. **Ingests** fresh content from RSS + Perplexity.
2. **Cleans & summarises** articles.
3. **Writes** LinkedIn-ready copy (hook → insights → takeaway → CTA + niche hashtags).
4. **Queues** drafts in Google Sheets for approval.
5. **Publishes** approved posts to LinkedIn via Buffer (or LinkedIn API where available).


## Quick start (step by step)

### 0) Create accounts
- Go to [https://www.make.com](https://www.make.com) → click **Sign up free**.  
- Go to [https://n8n.io](https://n8n.io) → click **Try it free** (Cloud) or self-host if advanced.  
- Prepare API keys:
  - [OpenAI API](https://platform.openai.com/)
  - [Perplexity API](https://docs.perplexity.ai/)
  - [Buffer](https://buffer.com/developers) or LinkedIn Developer (if approved)
  - Google Sheets (sign in with your Google account during workflow import)

### 1) Set up Google Sheet
Create a sheet named **Drafts** with columns:

Title | URL | Source | Summary | PostDraft | Hashtags | ImageIdea | Status | CreatedAt | LinkedInPostId | PostedAt


Keep `Status` = `Pending` for new rows. Set to `Approved` before publishing.

### 2) Import the blueprints

#### Make.com
1. Log in → click **Scenarios**.
2. Click **+ Create a new scenario**.
3. On the bottom bar, click the **⋮ (More)** menu → choose **Import blueprint**.
4. Upload:
   - `blueprints/make/make.com LinkedIn Posts Research.blueprint.json`
   - `blueprints/make/make.com Post to LinkedIn blueprint.json`
5. Connect services when prompted (Google Sheets, OpenAI, Buffer).
6. Save the scenario.

#### n8n
1. Log into n8n Cloud (or local install).
2. Go to **Workflows** → **+ New Workflow**.
3. In the top-right menu, click **Import from File**.
4. Upload:
   - `blueprints/n8n/LinkedIn Posts Research_n8n_blueprint.json`
   - `blueprints/n8n/Post on LinkedIn_n8n_blueprint.json`
5. Connect credentials (Google, OpenAI, Perplexity, Buffer).
6. Save & **Activate** the workflows.

### 3) Run the Research workflow
- In Make.com: click **Run once**.  
- In n8n: click **Execute workflow**.  
- Check Google Sheets: rows should appear with `Status = Pending`.

### 4) Approve & Publish
- In Google Sheets: set selected rows to `Approved`.
- Run the **Post to LinkedIn** workflow (or let it run on its schedule).
- Confirm LinkedIn posts are published and `LinkedInPostId` + `PostedAt` columns update.

## Prompts (summary)

**Perplexity (JSON only):**
- System: “Return strictly valid JSON array. No prose, no code fences.”  
- User: Find ~10 recent high-quality articles; return `{title,url,date,source}`.

**OpenAI – Brief Builder (GPT-4.1, temp 0.3):**
- Input: `article_title`, optional `article_summary`.
- Output JSON:  
  `{"topic":"…","views_on_topic":"…","target_audience":["…"],"hashtags_seed":["#…"]}`

**OpenAI – LinkedIn Post Writer (GPT-4.1, temp 0.5–0.7):**
- Spec: Hook ≤ 22 words; 2–3 insights; 1 takeaway; soft CTA; 3–5 niche hashtags (include **#KiDaFlow** if relevant); 130–170 words; strict JSON.
- Output JSON:  
  `{"post_text":"…","hashtags":["#…","#KiDaFlow"],"image_idea":"…"}`
  
> Full prompt texts (including troubleshooting & research prompts) are in `docs/prompts/`.

## Scheduling

- Example cron in n8n: **Mon & Wed 10:27** → `27 10 * * 1,3`  
- Example cron in Make.com: use the custom scheduler UI.


## Troubleshooting

- **Perplexity returns ```json fences** → strip code fences before JSON parse, or enforce “no code fences” in the System prompt.
- **“Buffer can’t be converted to text”** (Make.com) → ensure HTTP module outputs **String** before Text/Regex nodes.
- **Only one result from an array (n8n)** → use **Item Lists → Split Out Items** (or For Each) so each result becomes its own item.
- **LinkedIn API approval** → if not granted, use Buffer as the publishing path.

More examples in `docs/troubleshooting.md`.

## Repo layout

.
- ├─ README.md
- ├─ docs/
- │  ├─ Capstone\_Toolkit.md
- │  ├─ troubleshooting.md
- │  └─ prompts/
- │     ├─ perplexity\_prompt.md
- │     ├─ openai\_brief\_builder.md
- │     ├─ openai\_linkedin\_writer.md
- │     ├─ troubleshooting\_prompts.md
- │     └─ strategy\_prompts.md
- ├─ blueprints/
- │  ├─ make/
- │  │  ├─ make.com LinkedIn Posts Research.blueprint.json
- │  │  └─ make.com Post to LinkedIn blueprint.json
- │  └─ n8n/
- │     ├─ LinkedIn Posts Research\_n8n\_blueprint.json
- │     └─ Post on LinkedIn\_n8n\_blueprint.json
- └─ samples/
- ├─ google\_sheet\_template.csv
- └─ example\_output\_row\.json



## Contributing / Licence

- Issues and PRs welcome (typos, improvements, alt prompts).
- [MIT Licence](https://github.com/randy-kip/Automated-LinkedIn-Content-Engine-with-GenAI-Capstone-Project/blob/main/LICENSE).

## Acknowledgements

Thanks to Moringa School for the capstone brief and to Make.com, n8n, OpenAI, Perplexity, and Buffer for their APIs and docs.

