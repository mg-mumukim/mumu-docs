---
name: jira
description: Query and manage Jira issues in the Hyperconnect Atlassian workspace (AURA project, etc.).
---

# Usage
/jira <action> [arguments]

Examples:
- `/jira LP-485` -- Get issue details
- `/jira my issues` -- List my in-progress issues
- `/jira search <JQL>` -- Run a JQL query

# Configuration

- **Cloud ID**: `hyperconnect.atlassian.net`
- **Primary MCP**: Use `mcp__atlassian__*` tools with `cloudId: "hyperconnect.atlassian.net"`.
- **Fallback MCP**: `mcp__tinder-atlassian__*` tools (no cloudId needed, but may have limited access).

Always pass `cloudId: "hyperconnect.atlassian.net"` and `responseContentFormat: "markdown"` to `mcp__atlassian__*` calls.

# Actions

## Get issue (default)
If the argument looks like an issue key (e.g. `LP-485`, `AURA-123`):
1. Call `mcp__atlassian__getJiraIssue` with the key.
2. Present a summary table: key, title, status, priority, assignee, dates.
3. Show description links (Notion, Slack, Google Docs, etc.) if present.

## My issues
If the argument is `my issues`, `my tickets`, or `mine`:
1. Call `mcp__atlassian__searchJiraIssuesUsingJql` with JQL: `assignee = currentUser() AND status = "In Progress" ORDER BY updated DESC`.
2. List issues in a table.

## Search
If the argument starts with `search`:
1. Take the rest of the argument as a JQL query.
2. Call `mcp__atlassian__searchJiraIssuesUsingJql` with that JQL.
3. Present results in a table.

## Comment
If the argument starts with `comment <ISSUE-KEY>`:
1. Call `mcp__atlassian__addCommentToJiraIssue` with the issue key and comment body.

## Transition
If the argument starts with `move <ISSUE-KEY> to <STATUS>`:
1. Call `mcp__atlassian__getTransitionsForJiraIssue` to find the matching transition.
2. Call `mcp__atlassian__transitionJiraIssue` with the transition ID.

# Output format
- Use a markdown table for issue lists.
- For single issues, use a key-value table followed by description content.
- Include clickable links: `https://hyperconnect.atlassian.net/browse/<KEY>`.
