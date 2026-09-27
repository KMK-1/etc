# AI Cooking Companion — Technical Specification

## 1. MVP Architecture

Recommended initial stack:

- Frontend: Next.js + TypeScript
- UI: Tailwind CSS
- Backend: Next.js Route Handlers / Server Actions for MVP
- Database/Auth: PostgreSQL + managed auth (Supabase is a practical MVP choice)
- Validation: Zod
- ORM: Prisma or Drizzle; choose one and use consistently
- Voice: browser/mobile audio capture → STT provider adapter
- AI: provider-agnostic LLM adapter returning structured JSON
- Timers/state: deterministic application logic
- Nutrition: deterministic nutrition service + external/public nutrition dataset adapter
- Deployment: Vercel-compatible web/PWA first

Do not couple domain logic to a specific AI vendor.

## 2. Architecture

```
UI / PWA
  ├─ Home
  ├─ Recipe Setup
  ├─ Cooking HUD
  ├─ Complete
  └─ My Recipes
        │
        ▼
Application Layer
  ├─ Cooking Session Service
  ├─ Recipe Adaptation Service
  ├─ Kitchen Translation Service
  ├─ Memory Service
  ├─ Nutrition Service
  └─ Voice Command Service
        │
        ▼
Domain Layer
  ├─ Recipe / RecipeVersion
  ├─ CookingSession
  ├─ CookingAction
  ├─ KitchenProfile
  └─ CookingMemory
        │
        ├───────────────┐
        ▼               ▼
   PostgreSQL       AI Gateway
                    ├─ STT adapter
                    └─ LLM adapter
```

## 3. Critical Rule: Canonical Recipe

Never store "1T" or "내 숟가락 1.7회" as the sole source of truth.

Canonical units:
- mass: g
- liquid volume: ml
- discrete food: count
- time: sec
- temperature: °C

Example:

```json
{
  "ingredientId": "soy-sauce",
  "amount": 15,
  "unit": "ml",
  "displayHint": "tbsp"
}
```

Kitchen Profile converts canonical values only for presentation/input interpretation.

## 4. Adaptation Pipeline

```
Source Recipe Version
  ↓
Serving Adapter
  ↓
Taste Adapter
  ↓
Nutrition Adapter
  ↓
Substitution Adapter
  ↓
Adapted Canonical Recipe
  ↓
Kitchen Translator
  ↓
Display Recipe
```

Each transformation creates an Adaptation record:
- type
- before
- after
- reason
- confidence
- temporary/permanent

This enables explainability, undo and Recipe Diff.

## 5. Cooking State Machine

```
DRAFT
  ↓
PREPARING
  ↓
COOKING ⇄ PAUSED
  ↓
COMPLETED
```

State mutations happen through structured CookingActions, never direct LLM writes.

Example:

```ts
type CookingAction =
  | { type: "NEXT_STEP" }
  | { type: "PREVIOUS_STEP" }
  | { type: "SET_INGREDIENT_AMOUNT"; ingredientId: string; amount: number; unit: CanonicalUnit; confidence: number }
  | { type: "START_TIMER"; durationSec: number }
  | { type: "SET_HEAT"; level: number | string }
  | { type: "UNDO"; targetActionId?: string }
  | { type: "CORRECT_ACTION"; targetActionId: string; replacement: CookingAction };
```

Persist an append-only action log. Current state is derived/applied from actions.

## 6. Voice Pipeline

```
Push to Talk
 → audio
 → STT
 → deterministic parser
 → confidence gate
 → [simple] structured action
 → [reasoning] LLM structured action
 → Zod validation
 → domain validation
 → state mutation
 → UI feedback
```

Fast-path intents:
- next / previous
- timer
- undo
- correction
- explicit ingredient + explicit amount
- explicit heat change

LLM-path:
- substitutions
- rescue
- ambiguous cooking questions
- recipe/taste reasoning

Never send the full account history to the LLM. Construct a minimal CookingContext.

## 7. Minimal CookingContext

```ts
interface CookingContext {
  dish: string;
  servings: number;
  currentStep: number;
  currentInstruction: string;
  relevantIngredients: Array<{
    id: string;
    planned: CanonicalAmount;
    actual?: CanonicalAmount;
  }>;
  activeTimers: TimerSummary[];
  currentHeat?: string;
  kitchenHints?: string[];
  recentActions: CookingActionSummary[];
}
```

## 8. Voice Confidence

Recommended behavior:
- high confidence + reversible: apply immediately
- medium confidence: apply but show visible correction chip where safe
- low confidence or high-impact: confirm

High-impact examples:
- very large ingredient amount
- ambiguous ingredient identity
- temperature/safety instruction
- destructive correction

Store raw transcript + parsed action + confidence.

## 9. My Kitchen Translation

```ts
translateAmount(canonicalAmount, ingredient, kitchenProfile)
translateHeat(genericHeat, heatSourceProfile)
```

Examples:
- 15 ml soy sauce → user spoon 1.67
- medium heat → induction level 5 (profile estimate)

Do not imply exact physical equivalence for heat. Store calibration source and confidence.

## 10. Planned vs Actual

Never mutate planned recipe amounts during cooking.

```
planned_recipe_version
        +
actual_action_log
        ↓
final_cooking_result
```

This separation powers:
- recap
- Recipe Evolution
- Cook Again
- nutrition recalculation
- recipe diff

## 11. Cook Again

Cook Again should clone the successful FinalCookingResult into a new personal recipe version or session plan.

It must not silently fall back to the original source recipe.

## 12. Nutrition Service

```
final ingredient canonical amount
 → nutrition food mapping
 → amount-normalized nutrient calculation
 → dish totals
 → serving totals
```

Return:
- kcal
- protein
- carbohydrate
- fat
- sodium
- optional fiber/sugar later
- mapping coverage
- confidence

If only 85% of ingredient mass is mapped, UI must not present the result as exact.

LLM may explain or propose adaptations but cannot be the arithmetic source of truth.

## 13. Recipe Fork

Fork preserves:
- sourceRecipeId
- sourceVersionId
- parentForkId if applicable
- lineage

Fork creates a canonical copy/version first.

Kitchen translation is computed per viewer and is NOT persisted as a recipe mutation.

## 14. Rescue

Rescue request payload:

```ts
{
  issue: "TOO_SALTY",
  cookingContext,
  availableIngredients?: [...]
}
```

LLM response must conform to a RescuePlan schema and must be validated before display.

Safety-sensitive requests should route through a separate rules layer.

## 15. API Surface — MVP

Suggested endpoints:

```
POST /api/voice/transcribe
POST /api/cooking/:id/actions
GET  /api/cooking/:id
POST /api/cooking/:id/complete

GET  /api/recipes
GET  /api/recipes/:id
POST /api/recipes/:id/start

GET  /api/memories
POST /api/memories/:id/cook-again

GET  /api/kitchen-profile
PUT  /api/kitchen-profile
```

P1:
```
POST /api/recipes/:id/fork
GET  /api/recipes/:id/diff
POST /api/cooking/:id/rescue
GET  /api/cooking/:id/nutrition
```

## 16. Suggested Project Structure

```
src/
  app/
    (app)/
      page.tsx
      cook/[sessionId]/page.tsx
      recipes/[recipeId]/page.tsx
      memories/page.tsx
      kitchen/page.tsx
    api/
  components/
    cooking/
    recipe/
    kitchen/
    voice/
  domain/
    cooking/
    recipe/
    kitchen/
    nutrition/
  services/
    ai/
    stt/
    nutrition/
  lib/
    db/
    validation/
    units/
```

## 17. Offline/Failure Behavior

Cooking must degrade gracefully.

If AI fails:
- next/previous still works
- timers still work
- manual ingredient edits still work
- session state remains locally recoverable

Persist active session state locally and sync to server.

Never make an active recipe unusable because an LLM endpoint is down.

## 18. Security & Privacy

Voice audio:
- default to transient processing
- retain transcript/action only unless user explicitly opts into audio history
- document provider retention policies

User cooking history may reveal lifestyle information; scope access per user.

Community sharing must copy only explicitly shared recipe/session fields.

## 19. Observability

Track:
- STT latency
- intent parse latency
- LLM latency
- action correction rate
- undo rate
- failed state mutations
- cooking completion
- Cook Again conversion

Do not log raw voice/audio into general application logs.

## 20. MVP Definition of Done

A deployed user can:
1. create a Kitchen Profile;
2. open a seeded recipe;
3. start a Cooking Session;
4. issue voice commands;
5. see deterministic live state changes;
6. correct/undo a command;
7. finish cooking;
8. rate the result;
9. see the actual final recipe saved;
10. use Cook Again to reproduce it.
