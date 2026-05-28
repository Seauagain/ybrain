---
name: feishu
description: Execute Lark/Feishu operations via lark-cli - search messages, fetch documents, create documents, and retrieve meeting notes. Use this skill whenever the user mentions reading Feishu/Lark messages, documents (docx/wiki), meeting notes (妙记), or wants to create/upload content to Feishu. Also trigger when user says "从飞书拉", "读取飞书", "发到飞书", "lark-cli", or references specific Feishu URLs.
---

# Lark Operations Skill

This skill helps you execute common Lark/Feishu operations using the `lark-cli` tool. It covers the four most frequent use cases from the Talos team workflow.

## Prerequisites

Before using any lark-cli commands, verify the tool is installed and authenticated:

```bash
# Check if lark-cli is installed
which lark-cli

# If not installed, guide the user to install:
# npm i -g @larksuite/cli

# Check authentication status
lark-cli auth status

# If not authenticated, guide the user:
# lark-cli auth login
```

If authentication fails, the user needs to run `lark-cli auth login` and scan the QR code with their Feishu app.

## Core Operations

### 1. Search Messages Globally

Use when the user wants to find messages across all chats by keyword, time range, or sender.

**Command pattern:**
```bash
lark-cli im +messages-search \
  --params '{"query":"<keyword>","start_time":"<unix_timestamp>","end_time":"<unix_timestamp>"}' \
  --format pretty
```

**When to use:**
- User says "搜索飞书消息", "find messages about X", "search for X in Feishu"
- User wants to find messages by keyword, date range, or sender
- User needs to locate specific conversations or information

**Key parameters:**
- `query`: Search keyword (required)
- `start_time`: Unix timestamp for start of time range (optional)
- `end_time`: Unix timestamp for end of time range (optional)
- `--format`: Output format - use `pretty` for human-readable, `json` for programmatic processing

**Output handling:**
- By default, display results to user
- If user asks to save, write to a file (suggest `.claude/state/digest-messages-<timestamp>.md`)
- If user mentions KB/knowledge base, remind them this should be processed and chunked (part of the 4-step workflow)

### 2. Fetch Document Content

Use when the user provides a Feishu document URL or wants to read a specific docx/wiki.

**Command pattern:**
```bash
lark-cli docs +fetch \
  --doc "<feishu_url_or_token>" \
  --api-version v2 \
  --format pretty
```

**When to use:**
- User provides a Feishu document URL (dptechnology.feishu.cn/docx/...)
- User says "读取这个文档", "fetch this doc", "read the Feishu document"
- User wants to extract content from a wiki page

**Key parameters:**
- `--doc`: Full Feishu URL or document token (required)
- `--api-version v2`: Always use v2 API (v1 is deprecated)
- `--format`: Use `pretty` for markdown output, `json` for structured data
- `--scope`: Optional - use `outline` to get just headings, `keyword` to search within doc, `section` to get specific section

**Advanced usage:**
```bash
# Get document outline only
lark-cli docs +fetch --doc "<url>" --api-version v2 --scope outline

# Search within document
lark-cli docs +fetch --doc "<url>" --api-version v2 --keyword "关键词" --context-before 2 --context-after 2

# Fetch specific section
lark-cli docs +fetch --doc "<url>" --api-version v2 --scope section --start-block-id "<block_id>"
```

**Output handling:**
- Display content to user by default
- If content is long (>500 lines), ask user if they want to save to file
- Suggest saving to `.claude/state/digest-doc-<doc_id>.md` for temporary storage
- If user mentions processing or KB, this is step 1 of the 4-step workflow

### 3. Create New Document

Use when the user wants to upload content to Feishu or create a new document.

**Command pattern:**
```bash
lark-cli docs +create \
  --api-version v2 \
  --title "<document_title>" \
  --content "<markdown_content_or_file_path>"
```

**When to use:**
- User says "发到飞书", "create a Feishu doc", "upload this to Lark"
- User wants to share content with team via Feishu
- User has markdown content that needs to be converted to Feishu format

**Key parameters:**
- `--title`: Document title (required)
- `--content`: Can be inline markdown string or path to markdown file (required)
- `--api-version v2`: Always use v2 API
- `--folder-token`: Optional - specify parent folder

**Content preparation:**
- If user provides inline text, you can pass it directly
- If content is long or complex, save to a temp file first, then reference the file path
- Feishu supports standard markdown: headings, lists, tables, code blocks, links

**Output handling:**
- The command returns a document URL
- Always show the URL to the user: "Document created: <url>"
- If user wants to share with specific people, guide them to use Feishu's sharing UI (lark-cli doesn't handle permissions directly)

**Example:**
```bash
# Create from inline content
lark-cli docs +create --api-version v2 --title "Meeting Summary" --content "# Summary\n\nKey points..."

# Create from file
echo "# Report\n\nContent here..." > /tmp/report.md
lark-cli docs +create --api-version v2 --title "Weekly Report" --content /tmp/report.md
```

### 4. Retrieve Meeting Notes (妙记)

Use when the user wants to fetch AI-generated meeting summaries, action items, or transcripts.

**Command pattern:**
```bash
lark-cli vc +notes \
  --minute-tokens "<meeting_token>" \
  --format pretty
```

**When to use:**
- User says "读取会议纪要", "fetch meeting notes", "get the 妙记"
- User references a specific meeting and wants the AI summary
- User needs action items or key points from a meeting

**Key parameters:**
- `--minute-tokens`: Meeting identifier (required) - can be extracted from meeting URL
- `--format`: Use `pretty` for readable output, `json` for structured data

**Finding the meeting token:**
- If user provides a meeting URL, extract the token from it
- If user describes a meeting ("yesterday's standup"), you may need to search first:
  ```bash
  lark-cli vc +list --params '{"start_time":"<timestamp>","end_time":"<timestamp>"}'
  ```

**Output handling:**
- Meeting notes typically include: summary, action items, key decisions, attendees
- Display the full content to user
- If user wants to process or save, suggest `.claude/state/digest-meeting-<date>.md`
- Meeting notes are excellent candidates for KB chunking (part of 4-step workflow)

## Output Format Flexibility

The skill adapts to user requests:

- **Display only**: Show results in conversation (default)
- **Save to file**: Write to specified path or suggest `.claude/state/` for temporary digests
- **Process further**: If user mentions "沉淀", "chunk", "KB", or "知识库", this is part of the 4-step workflow:
  1. lark-cli pulls data → `.claude/state/digest-*.md`
  2. Agent processes → `docs/chunks/<layer>/X.md`
  3. kb-auditor reviews → git commit
  4. Future sessions auto-load via CLAUDE.md routing

## Error Handling

Common issues and solutions:

**Authentication errors:**
```
Error: Not authenticated
→ Run: lark-cli auth login
```

**Document not found:**
```
Error: Document not accessible
→ Check URL is correct
→ Verify user has permission to access the document
→ Try opening the URL in browser first
```

**API version mismatch:**
```
Error: API v1 deprecated
→ Always use --api-version v2 for docs operations
```

**Rate limiting:**
```
Error: Too many requests
→ Wait a few seconds and retry
→ Consider batching operations if doing multiple fetches
```

## Integration with KB Workflow

When user mentions KB, knowledge base, or the 4-step workflow:

1. **Pull from Feishu** (this skill): Use appropriate lark-cli command
2. **Save to digest**: Write to `.claude/state/digest-<type>-<id>.md`
3. **Remind user**: "This is step 1 of the 4-step workflow. Next, I can help process this into KB chunks, or you can handle that separately."
4. **Don't auto-chunk**: Unless explicitly requested, stop after pulling data. The user may want to review first.

## Tips for Effective Use

- **Always use `--api-version v2`** for docs operations (v1 is deprecated)
- **Use `--format pretty`** for human review, `--format json` for programmatic processing
- **Check authentication first** if you encounter permission errors
- **Extract document tokens carefully** from URLs - they're usually the last part of the path
- **For long documents**, consider using `--scope outline` first to understand structure
- **Meeting tokens** are different from document tokens - don't confuse them

## Examples

**Example 1: Read a document and save**
```bash
# User: "读取 https://dptechnology.feishu.cn/docx/ABC123 并保存"
lark-cli docs +fetch --doc "https://dptechnology.feishu.cn/docx/ABC123" --api-version v2 --format pretty > /tmp/doc-ABC123.md
echo "Document saved to /tmp/doc-ABC123.md"
```

**Example 2: Search messages and create summary**
```bash
# User: "搜索最近关于 hackathon 的消息，整理成文档发到飞书"
# Step 1: Search
lark-cli im +messages-search --params '{"query":"hackathon"}' --format json > /tmp/messages.json

# Step 2: Process (you do this)
# ... extract key points, format as markdown ...

# Step 3: Create doc
lark-cli docs +create --api-version v2 --title "Hackathon Discussion Summary" --content /tmp/summary.md
```

**Example 3: Fetch meeting notes**
```bash
# User: "拉取昨天的会议纪要"
# First, find the meeting (if token not provided)
lark-cli vc +list --params '{"start_time":"1716825600","end_time":"1716912000"}' --format json

# Then fetch notes using the token
lark-cli vc +notes --minute-tokens "meeting_xyz" --format pretty
```

## When NOT to Use This Skill

- User wants to send instant messages (use `lark-cli im +send` directly, not covered by this skill)
- User wants to manage calendar events (use `lark-cli calendar` commands)
- User wants to manage contacts (use `lark-cli contact` commands)
- User needs advanced document editing (this skill focuses on read/create, not complex updates)

For operations beyond these four core use cases, refer to `lark-cli --help` or the specific subcommand help.
