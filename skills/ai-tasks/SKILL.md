---
name: ai-tasks
description: Apply AI-powered analysis to images for business-specific tagging, metadata extraction, and quality checks using controlled vocabularies. Use when user wants to analyze images and apply structured metadata in ImageKit.

---

# AI Tasks Skill

## When to use
- User wants to tag images with business-specific categories
- User needs controlled vocabularies (predefined value lists)
- User wants yes/no quality checks on images
- User needs to extract metadata using natural language analysis

## AI Task Structure

Each AI Task contains **1-10 sub-tasks** with:

1. **Instruction** (required): Natural language question about the image
2. **Action Type** (required): `select_tags`, `select_metadata`, or `yes_no`
3. **Vocabulary** (optional): 1-30 predefined approved values

## Action Types

### select_tags
Adds tags from vocabulary. Supports multiple selections.
```json
{
  "type": "select_tags",
  "instruction": "What body style is this vehicle?",
  "vocabulary": ["sedan", "suv", "hatchback"],
  "max_selections": 1
}
```

### select_metadata
Sets custom metadata field (field must exist in DAM).
```json
{
  "type": "select_metadata",
  "instruction": "What is the primary color?",
  "field": "primary_color",
  "vocabulary": ["red", "blue", "white", "black"],
  "max_selections": 1
}
```

### yes_no
Binary quality check with conditional actions.
```json
{
  "type": "yes_no",
  "instruction": "Is the product completely visible?",
  "on_yes": { "add_tags": ["framing_ok"] },
  "on_no": { "add_tags": ["needs_reshot"] }
}
```

## Gotchas

- **Instruction clarity**: Be specific ("What is the collar type?" not "Describe the image")
- **Vocabulary design**: Use business terminology, non-overlapping, 1-30 items max
- **Field requirements**: For `select_metadata`, field must exist in DAM first
- **Tag values**: Cannot contain `%` character
- **yes_no tasks**: Must have at least one of `on_yes` or `on_no`
- **Vocabulary length**: Max 500 characters combined (select_tags only)
- **Scope**: Works on visual content only (images/videos)

## Applying AI Tasks

- **Via Saved Extensions**: Create and apply via dashboard/API
- **Via API at Upload**: Include in `extensions` array
- **Via Path Policies**: Auto-apply to files in specific folders
