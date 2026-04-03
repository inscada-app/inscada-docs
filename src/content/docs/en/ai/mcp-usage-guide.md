---
title: "MCP Usage Guide"
description: "inSCADA MCP Server usage scenarios, prompt examples, and token optimization"
sidebar:
  order: 3
---

This guide provides scenario-based prompt examples, optimization tips, and cost information for using the inSCADA MCP Server efficiently.

## Golden Rule: Specify Space and Project

Always specify the space and project name in your requests. Otherwise, Claude makes extra API calls to discover them, wasting tokens.

```
❌ "What is the value of ActivePower_kW?"
✅ "claude space, Energy Monitoring Demo, what is ActivePower_kW value?"
```

---

## Scenarios

### 1. Read Live Values

No guide needed. Ask directly.

**Single variable:**
```
claude space, Energy Monitoring Demo, ActivePower_kW value?
```

**Multiple variables:**
```
claude space, Energy Monitoring Demo, show all variable live values
```

**Connection status:**
```
claude space, Energy Monitoring Demo, check connection status
```

> Response is compact by default — only value, name, date. Say "show details" for the full API response.

---

### 2. Historical Data & Charts

No guide needed. Specify the time range.

**Line chart:**
```
claude space, Energy Monitoring Demo, ActivePower_kW last 24 hours chart
```

**Compare variables:**
```
claude space, Energy Monitoring Demo, compare ActivePower_kW and ReactivePower_kVAR, last 6 hours
```

**Statistics:**
```
claude space, Energy Monitoring Demo, ActivePower_kW 7 day statistics
```

**Live gauge:**
```
claude space, Energy Monitoring Demo, Temperature_C gauge 0-80°C
```

---

### 3. Variable Discovery

No guide needed.

```
claude space, Energy Monitoring Demo, list variables
```

```
claude space, Energy Monitoring Demo, list variables containing "Power"
```

> Variable list is compact by default — only id, name, type, unit, dsc. Say "list with details" for connection_id, scale, log_type, etc.

---

### 4. Alarms & Status

No guide needed.

```
claude space, show active alarms
```

```
claude space, Energy Monitoring Demo, project status
```

---

### 5. Script Writing

**Guide REQUIRED.** Nashorn ES5 rules are critical.

```
read inscada guide. claude space, Energy Monitoring Demo, write a script that
sends a notification when temperature exceeds 60°C every 10 seconds
```

```
read inscada guide. claude space, Energy Monitoring Demo,
read "Chart_ActiveReactivePower" script and change chart colors
```

**Script search (no guide needed):**
```
claude space, search "getVariableValue" in scripts
```

---

### 6. Animation Creation

**Guide REQUIRED.** SVG rules and element types are critical.

```
read inscada guide. claude space, Energy Monitoring Demo,
create SVG animation showing ActivePower_kW, Voltage_V, Current_A
```

---

### 7. Dashboards

Template = no guide needed. Custom HTML = guide recommended.

**Template (fast):**
```
claude space, Energy Monitoring Demo, ActivePower_kW gauge dashboard, 0-1000 kW
```

**Multi-chart:**
```
claude space, Energy Monitoring Demo, multi_chart dashboard
ActivePower_kW, Voltage_V, Current_A
```

**Custom HTML:**
```
read inscada guide. claude space, Energy Monitoring Demo,
custom HTML dashboard — 3 gauges on top, trend chart below
```

---

### 8. Export to Excel

No guide needed.

```
claude space, Energy Monitoring Demo, export ActivePower_kW 24 hour data to Excel
```

---

### 9. API Discovery

No guide needed.

```
find alarm endpoints in inSCADA API
```

---

## Quick Reference

| Task | Prompt Template | Guide? |
|------|-----------------|--------|
| Live value | `claude space, project, variable value?` | No |
| Chart | `claude space, project, variable last 24 hours chart` | No |
| Gauge | `claude space, project, variable gauge 0-100` | No |
| Statistics | `claude space, project, variable 7 day statistics` | No |
| Alarms | `claude space, active alarms` | No |
| Project status | `claude space, project status` | No |
| Variable list | `claude space, project, list variables` | No |
| Script writing | `read inscada guide. claude space, project, write script...` | **Yes** |
| Animation | `read inscada guide. claude space, project, create animation...` | **Yes** |
| Dashboard (template) | `claude space, project, gauge dashboard variable 0-1000 kW` | No |
| Dashboard (custom) | `read inscada guide. claude space, project, custom dashboard...` | **Yes** |
| Excel | `claude space, project, variable 24 hour Excel` | No |
| API discovery | `find alarm endpoints in API` | No |

---

## Compact vs Verbose Responses

By default, tool responses return only essential fields (compact mode). This reduces token consumption by 60-80%. When you need detailed information, say "show details" or use the `verbose: true` parameter.

### inscada_get_live_value

**Compact (default):**
```json
{
  "value": 421.79,
  "name": "ActivePower_kW",
  "dsc": "Total active power",
  "date": "2026-04-03T00:43:08"
}
```

**Verbose ("show details"):**
```json
{
  "@class": "...NumberVariableValue",
  "flags": {"scaled": true},
  "value": 421.79,
  "extras": {"raw_value": 306.73},
  "variableShortInfo": {
    "dsc": "...", "frame": "...",
    "device": "...", "connection": "..."
  },
  "dateInMs": 1775166188708
}
```

**Savings: 64%**

### inscada_get_live_values

**Compact (default):**
```json
{
  "ActivePower_kW": {"value": 421.79, "date": "2026-04-03..."},
  "Voltage_V": {"value": 232.9, "date": "2026-04-03..."}
}
```

**Verbose:** Full API response with flags, extras, variableShortInfo per variable.

**Savings: 83%**

### list_variables

**Compact (default):**
```json
[{
  "id": 23227,
  "name": "ActivePower_kW",
  "type": "Float",
  "unit": "kW",
  "dsc": "Total active power"
}]
```

**Verbose:** Adds connection_id, project_id, is_active, eng_zero_scale, eng_full_scale, log_type, code.

**Savings: 61%**

---

## Cost Estimates

Prices: Input $3/MTok, Output $15/MTok (Sonnet 4.6). Rate: 1 USD ≈ 44.50 TL.

| Scenario | Input Tokens | Output Tokens | USD | TL |
|----------|-------------|---------------|-----|-----|
| Live value query | ~6,400 | ~300 | $0.024 | ~1.0 TL |
| Line chart | ~6,400 | ~3,000 | $0.064 | ~2.8 TL |
| Script writing (with guide) | ~8,800 | ~7,900 | $0.145 | ~6.4 TL |
| Animation creation (with guide) | ~9,600 | ~11,200 | $0.197 | ~8.7 TL |
| Dashboard (template) | ~6,400 | ~1,500 | $0.042 | ~1.9 TL |

### Savings with Compact Mode

| Scenario | Before optimization | After optimization | Savings |
|----------|--------------------|--------------------|---------|
| Live value query | ~1.7 TL | ~1.0 TL | **41%** |
| 10 variables batch | ~3.2 TL | ~1.5 TL | **53%** |
| Variable list (500) | ~4.8 TL | ~2.5 TL | **48%** |

---

## Cost Reduction Tips

- **Always specify space + project** — saves 500-1000 discovery tokens
- **Guide only when needed** — saves ~4,000 tokens for simple queries
- **Compact is default** — say "details" only when you need raw values
- **Haiku 4.5 for simple queries** — 67% cheaper than Sonnet
- **Batch API** — 50% off for non-urgent bulk operations
- **Prompt Caching** — tool definitions cached at 10% cost after first call
