---
type: "domain"
number: 2
weight: 18
aliases:
  - "Domain 2"
  - "Tool Design & MCP Integration"
---
# Domain 2: Tool Design & MCP Integration

**18% of the exam.** A primary domain in [[Scenario-1-Customer-Support|Scenario 1]], [[Scenario-3-Multi-Agent-Research|Scenario 3]], [[Scenario-4-Developer-Productivity|Scenario 4]].

## 2.1 Design effective tool interfaces with clear descriptions and boundaries
- [[Claude-API-Tool-Use#Tool schemas|Tool schemas (Claude API course)]]: name, description and input_schema

## 2.2 Implement structured error responses for MCP tools
- [[Claude-Technologies-Reference#MCP (Model Context Protocol)|MCP: isError and structured error fields (reference)]]
- [[Claude-API-Model-Context-Protocol#Defining tools|Defining MCP tools (Claude API course)]]: a raised error comes back with isError but no structure

## 2.3 Distribute tools appropriately across agents and configure tool choice
- [[Tool-Choice|Tool Choice]]

## 2.4 Integrate MCP servers into Claude Code and agent workflows
- [[Claude-Technologies-Reference#MCP (Model Context Protocol)|MCP servers, .mcp.json and ~/.claude.json (reference)]]
- [[Claude-API-Model-Context-Protocol#Why MCP|Why MCP (Claude API course)]]: reuse a community server for standard integrations
- [[Claude-API-Model-Context-Protocol#Defining resources|Defining MCP resources (Claude API course)]]: a content catalog, so fewer exploratory tool calls

## 2.5 Select and apply built-in tools (Read, Write, Edit, Bash, Grep, Glob) effectively
- [[Grep]]
- [[Glob]]
