---
name: ogarniai-api
description: >
  Guide for working with Ogarni.AI personal finance data. Activate when users ask about
  receipts, expenses, spending summaries, subscriptions, financial categories, or budgeting.
  Also activate when users want to build integrations with the Ogarni.AI REST API, need
  authentication help, or ask about the MCP server setup.
---

# Ogarni.AI Data Access Guide

## Choosing Your Approach

| Context | Method | Details |
|---------|--------|---------|
| **Claude Code / MCP client** | MCP tools (preferred) | Auth is automatic — just call `mcp__ogarniai-mcp__*` tools directly |
| **Building an integration** | REST API | See `references/endpoints.md` for full endpoint reference |
| **Code examples needed** | Reference files | See `references/examples.md` for curl, Python, JavaScript |

## MCP Tools (Claude Code)

The ogarniai-mcp server handles authentication automatically. Call tools directly — no setup needed beyond having the MCP server connected.

All available tools are prefixed `mcp__ogarniai-mcp__ogarniai_*` and cover: documents, summaries, categories, tags, notifications, groups, mailboxes, deduplication, loyalty accounts, bank statements, and recurring expenses. Use the tool descriptions for parameter details.

## Common User Intents

**"Analyze my spending" / "How much did I spend?"**
1. Use `ogarniai_get_current_period` with preset `current-month` or `current-week`
2. For custom ranges, use `ogarniai_get_summary_by_period` with start/end dates
3. Present `categoryTotals` as a breakdown and `totalAmount` as the headline number

**"Find receipts from [store]" / "What did I buy at [store]?"**
1. Use `ogarniai_list_documents` to get recent documents
2. Filter results by `storeName` in your response (API doesn't support store-name filtering)
3. For item details, call `ogarniai_get_document` on matching documents

**"Show my subscriptions" / "What recurring expenses do I have?"**
1. Use `ogarniai_get_recurring_expenses` — returns all active subscriptions
2. Add `includeInactive: true` to show cancelled ones too

**"Compare this week to last week"**
1. Call `ogarniai_get_current_period` twice: once with `current-week`, once with `last-week`
2. Compare `totalAmount` and `categoryTotals` between the two

**"What categories do I spend most on?"**
1. Use `ogarniai_get_current_period` or `ogarniai_get_summary_by_period`
2. Sort `categoryTotals` by `totalAmount` descending
3. Use `ogarniai_list_categories` if you need category metadata (emoji, subcategories)

**"Show my weekly summary"**
1. Use `ogarniai_get_weekly_summary` for the latest AI-generated summary
2. The `summary` field contains a natural-language overview

## REST API (For Integrations)

Base URL: `https://api.ogarni.ai`

Authentication: All requests require `X-API-Key: oai_YOUR_TOKEN` header. Tokens are created at **app.ogarni.ai > Settings > API Tokens**.

For full endpoint reference with parameters and response schemas, load `references/endpoints.md`.
For code examples in curl, Python, and JavaScript, load `references/examples.md`.

### Security Best Practices
- Store tokens in `OGARNIAI_API_TOKEN` env var — never hardcode
- Use `read` scope unless write access is needed
- Rotate tokens every 90 days
- All endpoints require HTTPS

## MCP Server Installation

For users who need to set up the ogarniai-mcp server:

**Package:** `github:IsidoreSoftware/ogarniai-mcp`
**Required env var:** `OGARNIAI_API_TOKEN`
**Docs:** https://www.ogarni.ai/docs/mcp/instalacja
