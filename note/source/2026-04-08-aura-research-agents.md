<INSTRUCTIONS>
- All future documents must be written in English.
- The agent must respond in the same language as the user's request.
</INSTRUCTIONS>

# Tinder Business Research Agent

## Purpose

A research agent designed to answer Tinder business questions by querying internal data sources.

## Workflow

### 1. Query Clarification
- Understand the user's question clearly before execution
- Identify unknown terms, metrics, or abbreviations
- Use the `term-lookup` skill to find definitions when needed
- Ask clarifying questions if the query is ambiguous

### 2. Query Resolution
- Execute queries against Databricks SQL warehouse
- Analyze results and provide insights
- Present findings in a clear, actionable format

### Data Discovery Protocol

When searching for data, schema, or features in internal systems:

**NEVER conclude "doesn't exist" until ALL of these are checked:**
1. **Glean search** - Internal docs, wikis, data catalogs (2+ different query phrasings)
2. **Genie Chat** - Ask the configured Genie space directly
3. **information_schema search** - Broad column/table name search across catalogs
4. **Direct table exploration** - DESCRIBE on candidate tables

**Minimum 3 out of 4 sources must return empty before concluding unavailability.**

Checking only the tables you already know about is NOT sufficient.

## Data Sources

- **Databricks Unity Catalog**: Primary data warehouse
- **Warehouse ID**: `6c8cb88aa8e9297e` (Serverless PRO)
- **Key Catalogs**: `adhoc_prod`, system tables

## Common Metrics

| Metric | Definition |
|--------|------------|
| RSRR | Received Swipe Right Rate - Likes received / Swipes received (popularity) |
| SRR | Swipe Right Rate - Likes sent / Swipes sent (selectivity) |

## Skills

- `term-lookup`: Look up term/metric definitions (searches Glean then Databricks)
- `research-rigor`: **Top-level orchestrator** for statistical validity and confounder analysis. Use proactively for any causal analysis or group comparison. Coordinates term-lookup, databricks-sql, stat-compare, and batch-image-classifier.

## Research Methodology

All analyses MUST use the `research-rigor` skill, which provides:

- **Report Format**: Standardized structure (Executive Summary → Conclusions)
- **Statistical Rigor Checklist**: p-values, effect sizes, sample size validation
- **Confounder Analysis**: Standard + context-specific confounder identification
- **Skill Orchestration**: Coordinates term-lookup, databricks-sql, stat-compare, batch-image-classifier

See `.claude/skills/research-rigor/SKILL.md` for complete methodology.
