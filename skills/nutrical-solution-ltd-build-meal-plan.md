---
name: Build a customer meal plan with NutriCal
description: Create a meal-plan customer, then list and read meal plans and their per-meal recipes and nutrient totals using the NutriCal Food & Nutrition API.
api: openapi/nutrical-solution-ltd-openapi.yml
operations: [createMealPlanCustomer, listMealPlanCustomers, listMealPlans, getMealPlan]
method: generated
source: openapi/nutrical-solution-ltd-openapi.yml + https://docs.nutrical.co
---

# Build a customer meal plan with NutriCal

Use this skill to onboard a meal-plan customer and retrieve their meal plans
with per-meal recipes and nutrient totals.

## Auth
- Send the entity token in the `Access-Token` header on every request.
- Base URL: `https://api.nutrical.co` (test on `https://preprodapi.nutrical.co`).

## Steps
1. **Create the customer** — `createMealPlanCustomer`
   `POST /external/api/v2/meal-plan-customers/` with profile fields (name,
   email, age, height, weight, gender, `allergen_id_list`, `diet_type_id_list`).
   Read `data.mealplan_customer_id` from the response.
   - Resolve `allergen_id_list` via `listAllergens` (`GET /public/api/v1/allergens/`)
     and `diet_type_id_list` from recipe/diet metadata.
2. **Confirm the customer** — `listMealPlanCustomers`
   `GET /external/api/v2/meal-plan-customers/?search=<name>` to find the id again.
3. **List their meal plans** — `listMealPlans`
   `GET /external/api/v2/meal-plans/?meal_plan_customer_id__list=<id>&paginate=1&limit=10`
   Optionally filter by `date` (YYYY-MM-DD).
4. **Read a plan's detail** — `getMealPlan`
   `GET /external/api/v2/meal-plans/{mealPlanId}/` returns `total_nutrients`,
   and `categories[]` (BREAKFAST/LUNCH/...) each with a `recipe_list[]` and
   per-recipe nutrient totals.

## Rules
- Pagination is page-number: `paginate=1`, `page`, `limit`.
- Recipe assets (cover images) are served under `https://d1rgzvt1rjtuai.cloudfront.net/`.
- Success responses carry `status: SUCCESS`, `code: 900`.
- Handle `401` (bad/missing token) by re-issuing the entity Access-Token.
