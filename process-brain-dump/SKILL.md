---
name: process-brain-dump
description: Process a brain dump text file and extract structured tasks as JSON. Use when user wants to process a voice transcript or unstructured notes into actionable tasks.
---

# Process Brain Dump

## Purpose

Transform unstructured brain dump text (typically voice transcriptions) into a structured JSON array of actionable tasks.

## Task Types (Enum)

- `research` - Topics requiring investigation, web searches, or information gathering
- `todo_item` - Personal action items, errands, cancellations, administrative tasks
- `order_item` - Physical goods to purchase/order online
- `generate_memo_draft` - Ideas requiring a written memo or document draft
- `generate_spec_draft` - Technical or project specifications needing formal documentation

## Output Schema

```json
{
  "tasks": [
    {
      "type": "research | todo_item | order_item | generate_memo_draft | generate_spec_draft",
      "title": "Brief descriptive title (5-10 words)",
      "prompt": "Full context and details for the task, including any specific requirements mentioned",
      "tools": ["tool1", "tool2"]
    }
  ]
}
```

## Tool Suggestions by Task Type

- **research**: `WebSearch`, `WebFetch`, `parallel-search:web_search_preview`, `parallel-task:createDeepResearch`
- **todo_item**: `Bash`, `Read`, `Edit` (depends on the specific task)
- **order_item**: `WebSearch`, `WebFetch` (for finding where to order)
- **generate_memo_draft**: `Write`, `Read`
- **generate_spec_draft**: `Write`, `Read`, `WebSearch` (for research during drafting)

## Instructions

1. **Read the brain dump file** using the Read tool
2. **Filter signal from noise**: Brain dumps often contain:
   - Casual conversation (ignore)
   - Background chatter (ignore)
   - Actual actionable items (extract)
3. **Identify actionable items** by looking for phrases like:
   - "I need to..." / "I should..." / "We need to..."
   - "Research topic..." / "Look into..."
   - "Order..." / "Buy..."
   - Ideas that need documentation or specs
4. **Classify each task** into the appropriate type
5. **Generate the JSON output** with all extracted tasks
6. **Write the JSON** to a file named `tasks.json` in the same directory as the input file

## Example Extraction

**Input text:**
> "Oh yeah, I need to cancel my Netflix subscription. Also research topic - what are the best standing desks under $500? And I should order more coffee beans from that place I like."

**Output:**
```json
{
  "tasks": [
    {
      "type": "todo_item",
      "title": "Cancel Netflix subscription",
      "prompt": "Cancel the Netflix subscription",
      "tools": ["WebFetch"]
    },
    {
      "type": "research",
      "title": "Best standing desks under $500",
      "prompt": "Research the best standing desks available for under $500, comparing features, reviews, and value",
      "tools": ["WebSearch", "WebFetch"]
    },
    {
      "type": "order_item",
      "title": "Order coffee beans",
      "prompt": "Order more coffee beans from the preferred supplier",
      "tools": ["WebSearch"]
    }
  ]
}
```

## Processing Guidelines

- **Be comprehensive**: Extract ALL actionable items, even if they seem minor
- **Preserve context**: Include relevant surrounding context in the prompt field
- **Deduplicate**: If the same task is mentioned multiple times, combine into one entry
- **Infer intent**: Voice transcripts may have errors - interpret the likely intended meaning
- **Group related items**: If multiple related sub-tasks exist, you may group them or keep separate based on logical action boundaries
