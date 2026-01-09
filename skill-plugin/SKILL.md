---
name: versa-secure-tool-execution
description: Use when search_tools, execute_tool, or execute_workflow tools are available. For multi-step operations, use execute_workflow with proper imports.
---

# cintegrity Tool Execution

## Available Tools

- `search_tools(query)` - Find tools and get their schemas
- `execute_workflow(planner_code)` - Run multi-step Python workflow
- `execute_tool(tool_name, args)` - Run single tool call

## Required Workflow

### Step 1: Search for tools

```
search_tools(query="<your search terms>")
```

Returns tools with schemas:

```json
{
  "tools": [{
    "name": "<tool_name>",
    "inputSchema": {"properties": {"<arg>": {"type": "..."}}},
    "outputSchema": {"properties": {"<key>": ...}},
    "import_path": "from cintegrity.mcp_tools.<server> import <tool_name>"
  }]
}
```

### Step 2: Write workflow using the schemas

- Read `inputSchema` → use those argument names
- Read `outputSchema` → use those keys to access response data
- Copy `import_path` exactly

```
execute_workflow(planner_code='''
from cintegrity.mcp_tools.<server> import <tool_name>

async def workflow():
    result = await <tool_name>(<arg_from_inputSchema>=<value>)
    data = result["<key_from_outputSchema>"]
    return data
''')
```

## Key Principles

1. **Always search first** - Never guess tool names or schemas
2. **inputSchema = arguments** - Use exact argument names from schema
3. **outputSchema = response keys** - Use exact keys to access data

## Workflow Code Structure

```python
from cintegrity.mcp_tools.<SERVER> import <TOOL_NAME>

async def workflow():
    result = await <TOOL_NAME>(<ARG>=<VALUE>)
    return result
```

### Rules

1. Entry point must be `async def workflow():`
2. Use `await` for all tool calls
3. Use keyword arguments: `await tool(arg=value)`
4. Return the final result
5. Get tool names, arguments, and response keys from `search_tools()` schemas

## Patterns

### Single Tool Call

```python
async def workflow():
    result = await <tool>(<arg_from_inputSchema>=<value>)
    return result
```

### Extract Data from Response

```python
async def workflow():
    result = await <tool>(<arg>=<value>)
    # Use key from outputSchema
    data = result["<key_from_outputSchema>"]
    return data
```

### Multi-Step Workflow

```python
async def workflow():
    # Step 1: First tool call
    result1 = await <tool_a>(<arg>=<value>)

    # Step 2: Extract data using outputSchema keys
    data = result1["<key>"]

    # Step 3: Pass to next tool
    result2 = await <tool_b>(<arg>=data[0]["<nested_key>"])

    return result2
```

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Tool not found | Run `search_tools()` to get correct `import_path` |
| Wrong argument names | Check `inputSchema` for correct argument names |
| Unknown response structure | Return raw result to inspect |
