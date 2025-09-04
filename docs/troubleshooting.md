## Common issues

### Perplexity returns code fences
Strip ```json/``` before parsing, or enforce “no code fences” in System prompt.

### “Buffer can’t be converted to text” (Make.com)
Ensure HTTP → Response is **String** before Text/Regex nodes.

### Only one array item (n8n)
Use **Item Lists → Split Out Items** or **For Each** before mapping fields like `url`.

### Posting to LinkedIn (n8n)
If your LinkedIn developer app isn’t approved, publish via **Buffer API** and write back `LinkedInPostId`.
