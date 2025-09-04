# Troubleshooting Prompts

These prompts were used during debugging of the workflows in Make.com and n8n.

---

## 1. JSON Fences (Perplexity)

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

---

## 2. Looping / Splitting Arrays (n8n)

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

