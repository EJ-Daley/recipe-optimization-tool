# Recipe Optimization Tool

## 1. Project Overview

Build a locally running Python application that stores personal recipes and ingredient information, then recommends combinations of meals that make efficient use of purchased ingredients.

The primary goal is to **reduce food waste, unnecessary grocery purchases, and the effort required to plan meals**. The application should go beyond simply identifying recipes with overlapping ingredients. It should consider ingredient quantities, existing pantry stock, typical purchase sizes, and the difficulty of using up perishable leftovers.

The application is intended for personal use on a single computer. It should not require a server, cloud deployment, or an internet connection during normal operation.

## 2. Core Objectives

1. Maintain a database of ingredients and their purchasing characteristics.
2. Maintain a database of recipes, including structured ingredient quantities and cooking instructions.
3. Recommend complementary meals based on selected recipes, available ingredients, and expected leftovers.
4. Provide a basic graphical interface for managing data and interacting with the optimizer.
5. Support future expansion into grocery budgeting, recipe-gap detection, and meal diversity.

## 3. Technology Stack and Architecture

Use Python for application logic and Tkinter for the graphical interface. Store persistent data in JSON files.

```text
recipe_optimizer/
├── ingredients.json
├── recipes.json
├── optimizer.py
└── app.py
```

- **ingredients.json:** Ingredient definitions, stocking status, perishability, and purchasing information.
- **recipes.json:** Recipe names, serving sizes, ingredient quantities, and cooking instructions.
- **optimizer.py:** Data loading, ingredient calculations, meal combination scoring, and recommendations.
- **app.py:** Tkinter interface for data entry, recipe selection, and displaying results.

Keep the user interface separate from the optimization engine. The optimizer should expose reusable functions that can be tested independently of Tkinter.

JSON files should be an implementation detail during normal use. The user should not need to manually edit recipe JSON to maintain the database.

## 4. Must-Have Requirements

### 4.1 Ingredient Database

Each ingredient should have a stable identifier and structured purchasing information.

Example:

```json
{
  "ingredient_id": "butter",
  "name": "Butter",
  "is_stocked": true,
  "perishability_score": 1,
  "replacement_cost": 6.00,
  "purchase_quantity_variable": false,
  "purchase_unit": "g",
  "purchase_quantity": 454
}
```

Fields:

| Field | Purpose |
|---|---|
| `ingredient_id` | Stable canonical identifier referenced by recipes |
| `name` | Human-readable ingredient name |
| `is_stocked` | Whether the ingredient is generally kept at home |
| `perishability_score` | Priority for using up leftover quantities, from 0 to 2 |
| `replacement_cost` | Estimated cost of replenishing a standard purchase quantity |
| `purchase_quantity_variable` | Whether the user can generally buy the amount they need |
| `purchase_unit` | Unit used for purchasing, such as `g`, `ml`, or `num` |
| `purchase_quantity` | Typical purchase quantity, or `null` for variable quantities |

The example values are illustrative rather than fixed defaults.

#### Perishability scoring

- **0:** Little or no urgency to use leftovers, such as salt, dried spices, flour, pasta, or canned goods.
- **1:** Prefer to use leftovers, but keeping some is generally acceptable, such as carrots, celery, butter, or shallots.
- **2:** Strongly prioritize using leftovers, such as spinach, buttermilk, or fresh herbs.

This score represents the optimization priority rather than literal shelf life. A separate shelf-life field is not required for the initial version.

Stocking status and perishability must remain independent. An ingredient can be a regular pantry item while still deserving a high leftover penalty.

#### Purchasing conventions

- Use `g` for grams, `ml` for millilitres, and `num` for countable items.
- Use `num` for items such as cans, carrots, or individual garlic bulbs where appropriate, rather than creating item-specific units.
- Record typical package sizes for ingredients usually purchased in fixed quantities.
- Use `purchase_quantity_variable: true` and `purchase_quantity: null` when quantities can generally be purchased as needed.
- Do not require package contents for every ingredient. For example, canned goods can initially be tracked by can count.

The application should eventually calculate unit costs from purchase cost and package quantity where applicable.

### 4.2 Recipe Database

Each recipe should contain its identity, serving size, structured ingredient list, and cooking instructions.

Example:

```json
{
  "recipe_id": "garlic_tomato_pasta",
  "name": "Garlic Tomato Pasta",
  "servings": 2,
  "ingredients": [
    {
      "ingredient_id": "pasta",
      "quantity": 200,
      "unit": "g"
    },
    {
      "ingredient_id": "canned_tomatoes",
      "quantity": 1,
      "unit": "num"
    },
    {
      "ingredient_id": "garlic",
      "quantity": 3,
      "unit": "num"
    },
    {
      "ingredient_id": "olive_oil",
      "quantity": 15,
      "unit": "ml"
    }
  ],
  "instructions": "1. Bring a pot of salted water to a boil.\n2. Slice the garlic and gently cook it in olive oil.\n3. Add the tomatoes and simmer while the pasta cooks.\n4. Combine and serve."
}
```

The example quantities are illustrative.

Requirements:

- Use stable recipe IDs to link recipes to canonical ingredient IDs.
- Store each recipe's baseline serving count.
- Store ingredient quantities in units that make sense for cooking.
- Store instructions as a multiline string, not as a collection of parsed steps.
- Keep ingredient properties in the ingredient database rather than duplicating them in recipes.
- Support scaling recipes according to the desired number of servings.

The initial implementation should support straightforward units and quantities. More complex conversions, such as cloves to grams or volume to weight, can be added later rather than requiring an exhaustive conversion system upfront.

The structured ingredient list is the source of truth for optimization. The application should not attempt to extract ingredient quantities from the instructions.

### 4.3 Optimization Engine

The optimizer is the central feature. It should evaluate combinations of recipes and recommend meal plans that use ingredients efficiently.

#### Input modes

**Mode A: Start with selected recipes**

The user selects one or more recipes they definitely want to cook. The optimizer recommends one or two additional recipes that complement those choices and reduce problematic leftovers.

**Mode B: Start with ingredients**

The user identifies ingredients they want to use up. The optimizer recommends recipes that use those ingredients, then allows the user to build a meal plan interactively.

#### Core optimization considerations

1. **Ingredient overlap:** Prefer combinations that use shared ingredients efficiently.
2. **Quantity utilization:** Estimate how much of each ingredient will be used across the selected meals.
3. **Perishability:** Penalize leftover quantities more heavily for ingredients with higher perishability scores.
4. **Purchase requirements:** Distinguish ingredients already stocked from ingredients that must be purchased.
5. **Purchase quantities:** Account for typical package sizes when estimating what will remain after cooking.
6. **Recipe scaling:** Allow serving counts to affect ingredient quantities and leftover estimates.
7. **Fixed selections:** Preserve user-selected recipes by default. Offer replacements as optional alternatives rather than silently replacing the user's choices.

The optimizer must not simply maximize the number of shared ingredients. Two recipes that both use spinach may still leave most of a purchased quantity unused. Recommendations should consider the actual amounts used and remaining.

Likewise, leftover quantity should be evaluated in context. A small leftover amount of flour is usually less concerning than the same quantity of a highly perishable ingredient.

#### Recommendation output

For each proposed meal combination, display:

- Selected recipes and serving counts.
- Ingredients that need to be purchased.
- Estimated quantities used and remaining.
- Ingredients that are already stocked.
- Estimated grocery purchase cost.
- An explanation of why the additional recipes were recommended.

The scoring logic should be modular and transparent enough to adjust as the system is tested with real recipes.

### 4.4 Tkinter User Interface

A basic graphical interface is a must-have because manually entering and maintaining recipes in JSON would make the application unnecessarily cumbersome.

The initial interface should support:

- Adding, viewing, editing, and deleting ingredients.
- Adding, viewing, editing, and deleting recipes.
- Entering recipe ingredients through structured form fields.
- Entering and editing instructions in a multiline text box.
- Selecting recipes the user wants to cook.
- Entering ingredients the user wants to use up.
- Running the optimizer and reviewing its recommendations.

The interface does not need elaborate styling. Prioritize usability, reliable data entry, and avoiding repetitive manual work.

Validate user input and references between recipes and ingredients. For example, prevent a recipe from silently referencing an ingredient that does not exist in the ingredient database.

## 5. Should-Have Requirements

### 5.1 Grocery Budgeting

Allow the user to specify a weekly grocery budget.

The system should estimate the cost of newly purchased ingredients and help identify meal combinations that stay within the budget.

A useful future distinction is between:

- **Cash spending:** The estimated cost of items that must be purchased this week.
- **Inventory consumption:** The estimated replacement value of ingredients already stocked and used.

These values should be reported separately. Using existing inventory should not be counted as new cash spending, although its replacement value can help assess the overall economic cost of the meal plan.

The optimizer should also consider package sizes and account for the fact that purchasing an ingredient can leave a quantity available for future meals.

### 5.2 Recipe-Gap Detection

Identify recurring ingredient-utilization problems across the user's recipe database.

For example, if recipes frequently leave unused buttermilk, blueberries, or spinach, the system should recognize that the database lacks enough complementary recipes for those ingredients.

Potential functionality:

- Track ingredients that repeatedly generate undesirable leftovers.
- Identify ingredients with few useful recipe combinations.
- Suggest adding recipes that use those ingredients alongside ingredients already present in the user's recipe collection.
- Reduce the recommendation priority of recipes that repeatedly cause known leftover problems until better complementary recipes are available.

This feature should distinguish a genuine database gap from an occasional leftover that is reasonable or unavoidable.

## 6. Might-Have Requirements

These features should not delay the initial implementation.

### 6.1 Flavor Diversity

Avoid recommending combinations that are overly repetitive in flavor, cuisine, or preparation style.

For example, a plan should not necessarily consist entirely of tomato-based meals simply because they share several ingredients.

### 6.2 Nutritional Diversity

Consider nutritional balance across a planned sequence of meals, potentially including:

- Protein, fat, and carbohydrates.
- Fibre.
- Vegetables and greens.
- Selected vitamins and micronutrients.

This would require additional recipe metadata or a nutritional data source. It should be introduced only after the basic optimization engine works reliably.

### 6.3 Additional Recipe Metadata

Potential additions include cuisine, flavour profile, preparation time, dietary tags, and nutritional information. The data model
