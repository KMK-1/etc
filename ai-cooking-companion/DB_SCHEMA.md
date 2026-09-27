# AI Cooking Companion — Data Model

This is a conceptual relational schema for the MVP. PostgreSQL is assumed.

## Core relationships

```
users
 ├─ kitchen_profiles
 │    ├─ measuring_tools
 │    ├─ cookware
 │    └─ heat_sources
 ├─ taste_profiles
 ├─ recipes (owned/personal)
 ├─ cooking_sessions
 └─ cooking_memories

recipes
 └─ recipe_versions
      ├─ recipe_ingredients
      └─ recipe_steps

cooking_sessions
 ├─ cooking_actions
 ├─ session_ingredient_states
 ├─ timers
 └─ session_feedback
```

## users

- id uuid PK
- email
- display_name
- created_at
- updated_at

## kitchen_profiles

- id uuid PK
- user_id uuid FK UNIQUE
- name
- created_at
- updated_at

## measuring_tools

- id uuid PK
- kitchen_profile_id FK
- name
- kind
- volume_ml nullable
- calibration_confidence decimal
- created_at

Examples:
- 내 밥숟가락 / 9 ml
- 계량 큰술 / 15 ml
- 국자 / 65 ml

Ingredient-specific mass conversion should live in a separate conversion table if needed; do not assume 1 ml = 1 g.

## cookware

- id uuid PK
- kitchen_profile_id FK
- name
- type
- diameter_cm nullable
- capacity_ml nullable
- material nullable

## heat_sources

- id uuid PK
- kitchen_profile_id FK
- name
- type
- min_level nullable
- max_level nullable
- calibration_json jsonb
- confidence decimal

## taste_profiles

- id uuid PK
- user_id FK UNIQUE
- sweetness decimal
- saltiness decimal
- spiciness decimal
- garlic decimal
- metadata jsonb

Use normalized preference scales rather than baking recipe amounts into this table.

## recipes

- id uuid PK
- owner_user_id nullable
- title
- visibility enum(PRIVATE, UNLISTED, PUBLIC)
- source_type enum(SEED, USER, FORK, GENERATED)
- source_recipe_id nullable
- fork_parent_recipe_id nullable
- created_at
- updated_at

## recipe_versions

- id uuid PK
- recipe_id FK
- version integer
- parent_version_id nullable
- servings decimal
- status enum(DRAFT, ACTIVE, ARCHIVED)
- change_summary nullable
- created_at

Unique(recipe_id, version).

## ingredients

Canonical ingredient catalog.

- id uuid PK
- canonical_name
- category
- default_canonical_unit enum(G, ML, COUNT)
- aliases jsonb

## recipe_ingredients

- id uuid PK
- recipe_version_id FK
- ingredient_id FK
- amount decimal
- canonical_unit enum(G, ML, COUNT)
- optional boolean
- preparation_note nullable
- sort_order integer

## recipe_steps

- id uuid PK
- recipe_version_id FK
- step_number integer
- instruction
- duration_sec nullable
- generic_heat nullable
- metadata jsonb

## recipe_adaptations

Records meaningful canonical recipe changes.

- id uuid PK
- source_version_id FK
- result_version_id nullable
- type enum(SERVING, TASTE, NUTRITION, SUBSTITUTION, HOUSEHOLD)
- before_json jsonb
- after_json jsonb
- reason
- confidence decimal
- is_temporary boolean
- created_at

Kitchen display translation does NOT belong here.

## cooking_sessions

- id uuid PK
- user_id FK
- recipe_id FK
- planned_recipe_version_id FK
- status enum(PREPARING, COOKING, PAUSED, COMPLETED, ABANDONED)
- current_step integer
- planned_servings decimal
- actual_servings decimal nullable
- started_at
- completed_at nullable
- created_at
- updated_at

## session_ingredient_states

- id uuid PK
- cooking_session_id FK
- ingredient_id FK
- planned_amount decimal
- actual_amount decimal nullable
- canonical_unit enum(G, ML, COUNT)
- confidence decimal
- last_action_id nullable

Unique(cooking_session_id, ingredient_id).

## cooking_actions

Append-only event log.

- id uuid PK
- cooking_session_id FK
- sequence integer
- action_type
- payload_json jsonb
- raw_transcript nullable
- parse_confidence decimal nullable
- source enum(VOICE, TOUCH, SYSTEM, AI)
- supersedes_action_id nullable
- reverted_at nullable
- created_at

Never delete corrected actions. Mark them superseded/reverted.

## cooking_timers

- id uuid PK
- cooking_session_id FK
- label nullable
- duration_sec
- started_at
- paused_at nullable
- completed_at nullable
- cancelled_at nullable

## heat_events

- id uuid PK
- cooking_session_id FK
- heat_source_id nullable
- canonical_label nullable
- user_level nullable
- confidence decimal
- created_at

## session_feedback

- id uuid PK
- cooking_session_id FK UNIQUE
- rating integer
- feedback_text nullable
- sweetness_feedback nullable
- saltiness_feedback nullable
- spiciness_feedback nullable
- cook_again boolean nullable
- created_at

## cooking_memories

- id uuid PK
- user_id FK
- cooking_session_id FK UNIQUE
- recipe_id FK
- final_recipe_version_id nullable
- memory_summary
- is_successful boolean
- created_at

A successful memory is a candidate for Cook Again.

## final_cooking_results

Optional but recommended materialized result.

- id uuid PK
- cooking_session_id FK UNIQUE
- final_ingredients_json jsonb
- final_steps_json jsonb
- final_servings decimal
- total_duration_sec nullable
- created_at

This snapshot makes historical reproduction robust even if source recipes later evolve.

# Nutrition — P1

## nutrition_foods

- id uuid PK
- external_source
- external_id
- ingredient_id nullable
- name
- basis_amount decimal
- basis_unit
- kcal
- protein_g
- carbohydrate_g
- fat_g
- sodium_mg
- fiber_g nullable
- metadata jsonb

## session_nutrition_results

- id uuid PK
- cooking_session_id FK UNIQUE
- total_kcal
- protein_g
- carbohydrate_g
- fat_g
- sodium_mg
- mapped_ingredient_ratio decimal
- confidence decimal
- per_serving_json jsonb
- created_at

# Community — P2

## community_tips
- id
- author_user_id
- recipe_id nullable
- title
- body
- structured_change_json
- visibility
- created_at

## cooking_experiments
- id
- user_id
- tip_id
- cooking_session_id nullable
- status enum(SAVED, PLANNED, TRIED)
- outcome enum(BETTER, SAME, WORSE) nullable
- created_at

## recipe_lineage

Usually derivable from recipes.fork_parent_recipe_id, but a lineage table can be added if multi-parent provenance becomes necessary.

# Important invariants

1. Canonical units are the source of truth.
2. Planned amounts and actual amounts are never conflated.
3. Cooking actions are append-only.
4. Corrections preserve history.
5. Kitchen translation does not mutate RecipeVersion.
6. Fork preserves source lineage.
7. Nutrition derives from canonical final amounts.
8. Cook Again uses a final successful result, not merely the original source.
