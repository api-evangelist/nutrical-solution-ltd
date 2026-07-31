---
name: Analyze a recipe's nutrition with NutriCal
description: Find ingredients, create a recipe from them, and read back its nutrition analysis (calories, nutrients, allergens, diet types) using the NutriCal Food & Nutrition API.
api: openapi/nutrical-solution-ltd-openapi.yml
operations: [listIngredients, listRecipeCategories, createRecipe, getRecipe]
method: generated
source: openapi/nutrical-solution-ltd-openapi.yml + https://docs.nutrical.co
---

# Analyze a recipe's nutrition with NutriCal

Use this skill to turn a list of foods into a NutriCal recipe and read its
computed nutrition (calories, per-nutrient quantities, allergens, diet types).

## Auth
- Send the entity token in the `Access-Token` header on every request.
- Obtain the token from the Create Entity flow if you do not have one.
- Base URL: `https://api.nutrical.co` (use `https://preprodapi.nutrical.co` for testing).

## Steps
1. **Resolve ingredient ids** — `listIngredients`
   `GET /external/api/v2/ingredients/?search=<name>&paginate=1&limit=10`
   Read `data[].ingredient_id` for each food you want in the recipe.
2. **(Optional) pick a category** — `listRecipeCategories`
   `GET /external/api/v2/recipe-categories/` and take a `recipe_category_id`.
3. **Create the recipe** — `createRecipe`
   `POST /external/api/v2/recipes/` with body:
   ```json
   {
     "name": "My Recipe",
     "category_id": "<recipe_category_id>",
     "per_serving_weight_in_gm": 250,
     "ingredient_list": [
       { "ingredient_id": "<ingredient_id>", "quantity": 100, "unit": "G", "yield_percent": 100 }
     ]
   }
   ```
   The response `data` already includes `total_calorie`, `nutrients[]`, and `diet_type_list`.
4. **Read full detail later** — `getRecipe`
   `GET /external/api/v2/recipes/{recipeId}/` returns the full nutrient breakdown
   and ingredient list.

## Rules
- Pagination is page-number: `paginate=1`, `page`, `limit` (default 10). Read
  `pagination.max_page` / `next_page` to page through ingredient results.
- Success responses carry `status: SUCCESS`, `code: 900` (or a `message`).
- Handle `400` (bad input), `401` (bad/missing token), `500` (server error).
- NutriCal documents no idempotency key — do not blindly retry `createRecipe`
  on a timeout; list recipes first to check whether it was created.
