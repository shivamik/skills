# AI Task Actions

AI tasks support three task types, each designed for different metadata management needs:

| Task Type | What It Does | When to Use |
|-----------|--------------|-------------|
| [select_tags](#select-tags) | Selects and applies tags from your vocabulary | Categorization, product attributes, building searchable taxonomies, or applying multiple labels |
| [select_metadata](#select-metadata) | Sets custom metadata field values from your vocabulary | Structured data like color, season, type, status, or single/multi-select dropdown fields |
| [yes_no](#yes/no) | Asks yes/no questions and executes conditional actions | Quality checks, compliance verification, binary classifications, or conditional workflows |

### Select tags

Analyzes the image and adds relevant tags from your controlled vocabulary. The AI compares what it sees in the image against your instruction, selects matching tags from your vocabulary while respecting min/max selection constraints, and adds the selected tags to the file. For example, an image of a living room might receive tags: `["sofa", "chair", "table", "lamp"]`.

**Configuration:**
```js
{
  "type": "select_tags",
  "instruction": "What types of furniture are visible in this image?",
  "vocabulary": ["sofa", "chair", "table", "desk", "bed", "shelving", "cabinet", "lamp"],
  "min_selections": 1,
  "max_selections": 4
}
```

**Parameters:**

| Parameter        | Type   | Required | Description                                                                    |
| ---------------- | ------ | -------- | ------------------------------------------------------------------------------ |
| `type`           | string | Yes      | Must be `"select_tags"`                                                        |
| `instruction`    | string | Yes      | Question or instruction (1-2000 characters)                                    |
| `vocabulary`     | array  | No      | Possible tag values (1-30 items, max 500 chars combined, no `%` character)   |
| `min_selections` | number | No       | Minimum tags to select (≥ 0). Default: no minimum                              |
| `max_selections` | number | No       | Maximum tags to select (≥ 1). Default: no maximum                              |


# Select metadata

Analyzes the image and sets a custom metadata field value from your vocabulary. The AI evaluates the image against your instruction, selects the best matching value(s) from vocabulary, validates against field type constraints, and sets the custom metadata field. For example, a metadata field `lighting` might be set to `"golden-hour"`.

**Configuration:**
```js
{
  "type": "select_metadata",
  "instruction": "What is the dominant lighting condition in this image?",
  "field": "lighting",
  "vocabulary": ["natural-daylight", "golden-hour", "overcast", "indoor-artificial", "low-light", "night"],
  "min_selections": 1,
  "max_selections": 1
}
```

**Parameters:**

| Parameter        | Type   | Required | Description                                                                    |
| ---------------- | ------ | -------- | ------------------------------------------------------------------------------ |
| `type`           | string | Yes      | Must be `"select_metadata"`                                                    |
| `instruction`    | string | Yes      | Question or instruction (1-2000 characters)                                    |
| `field`          | string | Yes      | Custom metadata field name (must already exist in your media library)          |
| `vocabulary`     | array  | No      | Possible values matching field type (1-30 items)                              |
| `min_selections` | number | No       | Minimum values to select (≥ 0). Default: no minimum                            |
| `max_selections` | number | No       | Maximum values to select (≥ 1). Default: no maximum                            |

{% callout style="warning" %}
**Important:**  
1. The custom metadata field must exist before using it in AI tasks. Create fields using the [Custom Metadata Fields API](/api-reference#create-custom-metadata-field) or in your dashboard under Settings → Media Library → Custom Metadata Fields.
2. Your vocabulary must match the field's schema definition. If you later change the field schema, AI tasks may fail to set values. Check the asset history to see why values were or weren't set.
{% /callout %}


### Yes/No

Asks a yes/no question about the image and executes different actions based on the answer. The AI evaluates the image and returns one of three responses: **Yes**, **No**, or **Unknown** (when the AI cannot confidently determine the answer). Each response can trigger different actions—tags added/removed and metadata set/unset.

For example, a high-quality image might receive tags `["print-ready", "high-quality", "approved"]` and metadata updates for quality status. If the AI cannot confidently assess quality, the `on_unknown` actions execute instead (e.g., tagging for manual review).

**Configuration:**
```js
{
  "type": "yes_no",
  "instruction": "Does this image meet quality standards for print publication (sharp focus, good lighting, high resolution)?",
  "on_yes": {
    "add_tags": ["print-ready", "high-quality", "approved"],
    "set_metadata": [
      { "field": "quality_status", "value": "approved" },
      { "field": "print_approved", "value": true }
    ]
  },
  "on_no": {
    "add_tags": ["web-only", "needs-improvement"],
    "remove_tags": ["print-ready", "approved"],
    "set_metadata": [
      { "field": "quality_status", "value": "rejected" },
      { "field": "print_approved", "value": false }
    ]
  },
  "on_unknown": {
    "add_tags": ["needs-review"],
    "set_metadata": [
      { "field": "quality_status", "value": "pending" }
    ]
  }
}
```

**Parameters:**

| Parameter     | Type   | Required | Description                                                              |
| ------------- | ------ | -------- | ------------------------------------------------------------------------ |
| `type`        | string | Yes      | Must be `"yes_no"`                                                       |
| `instruction` | string | Yes      | Yes/no question (1-2000 characters)                                      |
| `on_yes`      | object | No*      | Actions to execute if AI determines answer is "yes"                      |
| `on_no`       | object | No*      | Actions to execute if AI determines answer is "no"                       |
| `on_unknown`  | object | No       | Actions to execute if AI cannot confidently determine yes or no          |

{% callout style="info" %} 
At least one of `on_yes` or `on_no` is required.
{% /callout %}


# Example: Fashion e-commerce

This comprehensive example combines all three task types for a fashion retailer:

```js
{
  "name": "ai-tasks",
  "tasks": [
    {
      "type": "select_tags",
      "instruction": "What types of clothing or accessories are visible in this product image?",
      "vocabulary": [
        "dress", "shirt", "blouse", "t-shirt", "sweater", "jacket", 
        "coat", "pants", "jeans", "skirt", "shorts", "shoes", 
        "boots", "sneakers", "bag", "belt", "hat", "scarf", "jewelry"
      ],
      "min_selections": 1,
      "max_selections": 5
    },
    {
      "type": "select_metadata",
      "instruction": "What is the primary color of the main product?",
      "field": "primary_color",
      "vocabulary": [
        "black", "white", "gray", "beige", "brown", 
        "red", "pink", "orange", "yellow", "green", 
        "blue", "navy", "purple", "multi-color", "metallic"
      ],
      "min_selections": 1,
      "max_selections": 1
    },
    {
      "type": "select_metadata",
      "instruction": "What season or weather is this product suitable for?",
      "field": "season",
      "vocabulary": ["spring", "summer", "fall", "winter", "all-season"],
      "min_selections": 1,
      "max_selections": 2
    },
    {
      "type": "yes_no",
      "instruction": "Is this a formal or dressy item (suitable for office, weddings, formal events)?",
      "on_yes": {
        "add_tags": ["formal", "dressy", "occasion-wear"],
        "set_metadata": [
          { "field": "style_category", "value": "formal" },
          { "field": "dress_code", "value": "business-formal" }
        ]
      },
      "on_no": {
        "add_tags": ["casual", "everyday"],
        "set_metadata": [
          { "field": "style_category", "value": "casual" },
          { "field": "dress_code", "value": "casual" }
        ]
      }
    },
    {
      "type": "yes_no",
      "instruction": "Does this product appear to be luxury or high-end (designer labels, premium materials, high-end styling)?",
      "on_yes": {
        "add_tags": ["luxury", "premium", "designer"],
        "remove_tags": ["budget", "value"],
        "set_metadata": [
          { "field": "price_tier", "value": "premium" },
          { "field": "target_market", "value": "luxury" }
        ]
      },
      "on_no": {
        "add_tags": ["accessible", "value"],
        "remove_tags": ["luxury", "premium"],
        "set_metadata": [
          { "field": "price_tier", "value": "standard" },
          { "field": "target_market", "value": "mass-market" }
        ]
      }
    }
  ]
}
```

This configuration gives you complete product categorization in one upload. Each product image automatically gets tagged with product types, assigned a primary color, categorized by season, classified by style (formal vs. casual), and marked as luxury or standard—all without manual intervention. Upload thousands of products, and every single one gets consistent, searchable metadata that your team and customers can immediately filter and browse.

## Example: Travel Industry
This example adds only tags to the images for a travel company.

```js
{
  "name": "ai-tasks",
  "tasks": [
    {
      "type": "select_tags",
      "instruction": "In which city is this place located? If you are not able to identify the city, don't provide any tag.",
      "max_selections": 1
    },
    {
      "type": "select_tags",
      "instruction": "In which country is this place located? If you are not able to identify the city, don't provide any tag.",
      "max_selections": 1
    },
    {
      "type": "select_tags",
      "instruction": "What is the landscape in this image? Add any suitable tags to the image",
      "max_selections": 5
    },
    {
      "type": "select_tags",
      "instruction": "At what time of the day has this picture been taken possibly?",
      "vocabulary": [
        "morning", "afternoon", "evening", "night"
      ],
      "max_selections": 1
    },
    {
      "type": "select_tags",
      "instruction": "Does this image consist of no people, only one people, or two, or three or more?",
      "vocabulary": [
        "no people", "one people", "two people", "three people", "more people"
      ],
      "max_selections": 1
    },
    {
      "type": "select_tags",
      "instruction": "Does this image consist of a male or a female or both male and female or no people?",
      "vocabulary": [
        "male", "female", "both male and female", "no people"
      ],
      "max_selections": 1
    },
    {
      "type": "select_tags",
      "instruction": "Does this image consist of a group of solo travellers, a couple, or a family, or no people?",
      "vocabulary": [
        "solo travellers", "family", "couple", "no people"
      ],
      "max_selections": 1
    },
    {
      "type": "select_tags",
      "instruction": "If there is a famous monument in this image, then identify it. If you can't identify it, then don't add any tag.",
      "max_selections": 1
    }
  ]
}
```
## Example: Automotive (Cars)
This example adds only tags to the images for an automotive company.

```js
{
  "name": "ai-tasks",
  "tasks": [
    {
      "type": "select_tags",
      "instruction": "What is the body style of the vehicle shown in this image?",
      "vocabulary": [
        "sedan", "suv", "hatchback", "coupe","convertible","pickup truck", "minivan","luxury","sports car","wagon"
      ],
      "max_selections": 1
    },
    {
      "type": "select_tags",
      "instruction": "Which part of the car is primarily featured in this image?",
      "vocabulary": [
        "full exterior", "front view","rear view", "side profile", "dashboard", "steering wheel", "front seats", "rear seats", "trunk/boot", "engine bay", "wheel/rim", "headlight", "infotainment screen"
      ],
      "max_selections": 1
    },
    {
      "type": "select_tags",
      "instruction": "If the brand (make) of the car is clearly visible or recognizable, identify it. If not recognizable, do not add a tag.",
      "max_selections": 1
    },
    {
      "type": "select_tags",
      "instruction": "What is the primary color of the vehicle's exterior?",
      "vocabulary": [
        "white", "black", "silver", "grey", "blue", "red", "green", "yellow", "orange", "brown"
      ],
      "max_selections": 1
    },
    {
      "type": "select_tags",
      "instruction": "Is the image taken in an indoor setting (showroom/garage) or an outdoor setting?",
      "vocabulary": [
        "indoor - showroom", "indoor - garage", "outdoor - city/road", "outdoor - nature/offroad", "studio - plain background"
      ],
      "max_selections": 1
    },
    {
      "type": "select_tags",
      "instruction": "Are there any people present in or around the vehicle?",
      "vocabulary": [
        "no people", "driver only", "driver and passengers", "person standing outside", "model posing with car"
      ],
      "max_selections": 1
    },
    {
      "type": "select_tags",
      "instruction": "Does the vehicle appear to be a new/modern model or a vintage/classic car?",
      "vocabulary": [
        "modern",
        "vintage/classic"
      ],
      "max_selections": 1
    },
    {
      "type": "select_tags",
      "instruction": "Identify any specific features visible in the image.",
      "vocabulary": [
        "sunroof", "alloy wheels", "leather seats", "digital instrument cluster", "spoiler", "roof rails", "led headlights"
      ],
      "max_selections": 3
    }
  ]
}
```