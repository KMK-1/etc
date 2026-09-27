# AI Cooking Companion — Product & MVP Specification

> Status: Product direction locked for MVP
> Working concept: **Cook → Remember → Improve → Cook Again**
> Primary interaction: **Voice Input + Visual Output**

## 1. Product Vision

AI Cooking Companion is not another recipe generator.

It is a cooking companion that learns how a user actually cooks in their real kitchen. The user talks while cooking; the app maintains live cooking state, records actual ingredient amounts and changes, learns successful outcomes, and makes the next cooking session more personalized and reproducible.

### One-line value proposition

> 말하면서 요리하면 AI가 내 주방·입맛·실제 조리법을 기억하고, 다음번에는 더 나에게 맞는 레시피로 만들어준다.

### Product loop

```
Discover / Choose
      ↓
Adapt to Me
      ↓
Voice Cooking
      ↓
Actual Cooking Record
      ↓
Evaluate
      ↓
Cooking Memory
      ↓
Recipe Evolution
      ↓
Cook Again
```

The moat is not recipe generation itself. It is the accumulated **personal cooking memory** and the ability to reproduce and improve successful cooking.

---

## 2. Product Principles

1. **Cooking session first, recipe document second**
   - Users want to successfully cook today's meal, not maintain recipe documents.

2. **Hands cook, mouth inputs, eyes confirm**
   - Voice-first input.
   - Persistent visual output.
   - Avoid chat-style UI during active cooking.

3. **Record reality, not only the plan**
   - Planned amount and actual amount are separate fields.

4. **Personalization must be explainable**
   - Every automatic adaptation should expose why it changed.

5. **Today-only changes are not permanent preferences**
   - Temporary cooking decisions and durable preferences must be separate.

6. **LLM reasons; deterministic engines calculate**
   - Nutrition, unit conversion, scaling, timers, and state transitions should not depend on free-form LLM arithmetic when deterministic logic is possible.

7. **Never silently guess high-impact uncertain inputs**
   - Ingredient, amount, time, temperature, and safety-related uncertainty should be surfaced or confirmed.

8. **Reproducibility over novelty**
   - "Make the delicious one from last time again" is a first-class use case.

---

## 3. Target User

Primary:
- Home cooks who cook repeatedly but do not want to manually maintain recipes.
- Users who modify online recipes based on taste, family, cookware, or available ingredients.
- Users who want hands-free assistance while cooking.

Secondary:
- Beginner cooks who need step-by-step guidance and rescue.
- Diet/nutrition-conscious cooks who still want normal food.
- Couples/families with different taste preferences.

---

# 4. MVP — P0

The MVP must prove one loop:

> **Voice Cooking → Actual State → Cooking Memory → Cook Again**

Do not block MVP on community, refrigerator inventory, image recognition, or advanced nutrition.

## 4.1 Voice Cooking

MVP interaction should use push-to-talk.

Supported commands:

```
"다음"
"이전"
"간장 한 숟가락 넣었어"
"설탕은 반 숟갈만 넣을게"
"3분 타이머"
"방금 거 취소"
"간장 말고 참기름이야"
"두부 없어"
"너무 짠데?"
"이거 기억해"
```

Simple commands should bypass the large LLM whenever possible.

### Intent examples

```ts
type CookingIntent =
  | "NEXT_STEP"
  | "PREVIOUS_STEP"
  | "ADD_INGREDIENT"
  | "CHANGE_AMOUNT"
  | "REMOVE_INGREDIENT"
  | "START_TIMER"
  | "CHANGE_HEAT"
  | "CORRECT_LAST_ACTION"
  | "UNDO"
  | "SUBSTITUTION_REQUEST"
  | "COOKING_RESCUE"
  | "SAVE_PREFERENCE"
  | "QUESTION";
```

## 4.2 Cooking HUD

Do not present the active cooking experience as a chatbot.

Primary screen:

```
제육볶음                     STEP 4 / 7

양념을 넣고 볶아주세요

🔥 인덕션 5단
⏱ 02:34

실제 투입
✓ 돼지고기 500g
✓ 고추장 내 숟가락 1.5회
✓ 설탕 내 숟가락 0.5회
○ 대파

[ 🎙 말하기 ]
```

Design requirements:
- Large current action.
- Large heat/timer state.
- Actual ingredient state always visible.
- Latest recognized voice command briefly visible.
- Long AI explanations collapsed behind "자세히".
- One-handed recovery controls if voice fails.

## 4.3 Live Cooking State

```ts
interface CookingSession {
  id: string;
  recipeId: string;
  recipeVersionId: string;
  householdProfileId?: string;
  startedAt: string;
  completedAt?: string;
  currentStep: number;
  status: "PREPARING" | "COOKING" | "PAUSED" | "COMPLETED";
  plannedServings: number;
  actualServings?: number;
  ingredientStates: IngredientState[];
  heatEvents: HeatEvent[];
  timers: TimerState[];
  actions: CookingAction[];
  rating?: number;
  feedback?: string;
}

interface IngredientState {
  ingredientId: string;
  plannedCanonicalAmount: number;
  actualCanonicalAmount?: number;
  canonicalUnit: "g" | "ml" | "count";
  confidence: number;
  source: "recipe" | "voice" | "manual";
}
```

Never overwrite planned amounts with actual amounts.

## 4.4 My Kitchen

User-configurable kitchen profile:

```ts
interface KitchenProfile {
  measuringTools: MeasuringTool[];
  cookware: Cookware[];
  heatSources: HeatSource[];
}

interface MeasuringTool {
  id: string;
  name: string;             // 내 밥숟가락
  canonicalVolumeMl?: number;
  canonicalWeightG?: Record<string, number>;
}

interface Cookware {
  id: string;
  type: "PAN" | "POT" | "BOWL" | "OTHER";
  diameterCm?: number;
  capacityMl?: number;
  material?: string;
}

interface HeatSource {
  id: string;
  type: "INDUCTION" | "GAS" | "ELECTRIC";
  minLevel?: number;
  maxLevel?: number;
}
```

Display translation example:

```
Canonical: soy sauce 15 ml
Display: 내 밥숟가락 약 1.7회

Canonical: medium heat
Display: 내 인덕션 약 5단
```

Canonical values remain the source of truth.

## 4.5 Cooking Memory

After cooking, generate a structured memory from actual actions.

```
제육볶음 #4

계획
고추장 2T
설탕 1T

실제
고추장 1.5T
설탕 0.5T
인덕션 6단 / 6분

★★★★★
"단맛 이 정도가 딱 좋음"
```

Memory should contain:
- actual ingredient amounts;
- substitutions;
- heat/time changes;
- user corrections;
- final rating;
- free-text/voice feedback;
- temporary vs durable preferences;
- confidence.

## 4.6 Cook Again

Home must surface successful recent meals.

```
지난번 맛있었던 제육볶음
★★★★★

[ 그대로 다시 만들기 ]
```

"Cook Again" starts from the previous successful **actual cooking result**, not the original internet recipe.

---

# 5. P1 — Differentiating Features

## 5.1 Recipe Evolution

Cooking history evolves a personal recipe.

```
제육볶음 v1
  ↓ 설탕 감소
v2 ★★★★★
  ↓ 마늘 증가
v3 ★★★★★
  ↓
My Best
```

The system may suggest:

> 최근 3번 모두 설탕을 줄였고 만족도가 높았습니다. 기본 레시피에 반영할까요?

Never automatically turn a one-off change into a durable preference.

## 5.2 Cooking Rescue

Examples:
- 너무 짜
- 너무 달아
- 너무 매워
- 너무 묽어
- 너무 되직해
- 고기가 질겨
- 양념이 부족해

Rescue must use current Cooking State.

Example:

```
현재 상태 기준
1. 물 80–100ml 추가
2. 맛 확인
3. 필요하면 두부 100g 추가
```

If the user says "물 100 넣었어", the rescue action becomes part of the actual recipe record.

Food-safety decisions must use validated safety rules/data and conservative messaging rather than visual/LLM certainty alone.

## 5.3 Recipe Fork

Fork means:

> **Copy another cooking experience → translate it into my kitchen → optionally adapt it to my taste/goals → cook it.**

Pipeline:

```
Source Recipe
    ↓
Canonical Recipe
(g / ml / count / °C / sec)
    ↓
Serving Scaling
    ↓
Optional Taste Adaptation
    ↓
Optional Nutrition Adaptation
    ↓
My Kitchen Translation
    ↓
My Fork
    ↓
Cooking Session
```

### Important

The original recipe is immutable.

Kitchen translation is not a recipe modification.

Example:

```
15 ml → 내 숟가락 1.7회
중불 → 내 인덕션 약 5단
```

## 5.4 Recipe Diff

Separate:

### Recipe Diff
Actual recipe changes.

```
설탕   10g → 5g    -50%
마늘   10g → 15g   +50%
```

### Kitchen Translation
Same canonical recipe, different user-facing representation.

```
간장 15ml → 내 숟가락 1.7회
중불 → 인덕션 5단
```

Never mix the two.

---

# 6. Nutrition Engine

Nutrition is calculated from the final recipe, not merely the source recipe.

## 6.1 Calculation pipeline

```
Actual Ingredient Input
        ↓
Kitchen Measurement Conversion
        ↓
Canonical g / ml / count
        ↓
Food/Nutrition Database Mapping
        ↓
Deterministic Nutrition Calculation
        ↓
Whole Dish Nutrition
        ↓
Per Serving Nutrition
```

The LLM must not invent kcal/macros.

Final cooking result should support:

```
총 1,740 kcal
580 kcal / serving

Protein      42g
Carbohydrate 26g
Fat          32g
Sodium       1,150mg
```

Show:
- source recipe estimate;
- adapted recipe estimate;
- actual final result when enough actual data exists;
- confidence/coverage when some ingredients are estimated.

## 6.2 Nutrition Adaptation

Do not create a separate "diet food universe".

Transform food the user already wants to eat.

Modes:
- Calories ↓
- Protein ↑
- Sodium ↓
- Carbohydrates ↓
- Balanced

Example:

```
현재 587 kcal / serving
목표 ≤ 500 kcal

밥 180g → 130g    -65 kcal
기름 5ml → 3ml    -18 kcal
설탕 5g → 3g       -8 kcal

예상 496 kcal
```

User constraints override optimization:

> "밥은 줄이기 싫어."

The engine should search for other adjustments.

---

# 7. P2 — Community Intelligence

Community is not a generic recipe SNS.

Its purpose is to transfer **tested cooking experience**.

## 7.1 Structured Cooking Tips

```
"양파를 먼저 볶으면 설탕을 줄여도 단맛이 납니다."

실제 적용 N회
더 좋음 / 비슷함 / 이전이 좋음

[ 다음번에 시험하기 ]
```

Only show statistical percentages when sufficient real data exists.

## 7.2 Cooking Experiment

```
Community Tip
     ↓
Save as Experiment
     ↓
Next Cooking Session
     ↓
Try It
     ↓
Compare Outcome
     ↓
Personal Cooking Memory
     +
Community Evidence
```

Outcome:
- Better
- Similar
- Previous was better

## 7.3 Recipe Lineage

```
Source User Recipe v8
        ↓ Fork
My Recipe v1
        ↓
My changes
        ↓
My Recipe v2
```

Preserve lineage and recipe diffs.

Future trust signals:
- actual completed cooks;
- cook-again rate;
- fork count;
- tried-tip outcomes;
- repeat satisfaction.

Avoid optimizing primarily for likes/followers.

---

# 8. Household Profile

Future extension:

```ts
interface HouseholdProfile {
  id: string;
  name: string; // 우리 부부
  members: HouseholdMember[];
  servingDefault: number;
  combinedTasteProfile?: TasteProfile;
}
```

The question becomes "Who is eating?" rather than only "How many servings?"

Examples:
- 나
- 배우자
- 아이
- 손님
- 우리 부부

This can influence serving size and taste adaptation.

---

# 9. Supporting Features — Deliberately Secondary

These features are useful but must not define the product.

## Ingredient / Refrigerator State

Prefer fuzzy state:

```
계란   충분
대파   얼마 안 남음
두부   없음
```

Update opportunistically from Cooking Sessions.

> "계란 두 개 넣었어" → inventory decreases automatically.

Do not require users to maintain a perfect inventory ledger.

## Contextual Menu Recommendation

Recommendation dimensions:

```
Available ingredients
× time
× cleanup effort
× mood
× taste
× household
× nutrition goal
```

Example input:

> 오늘 피곤하고 설거지 하기 싫어.

## Image Input

Useful for:
- dish identification;
- cooking-state questions;
- rescue context;
- ingredient recognition.

Do not position "photo → recipe" as the primary differentiator.

## Shopping List

Generate missing ingredients from planned meals. Later integrate purchase confirmation with fuzzy inventory.

---

# 10. Core Data Model

Minimum conceptual entities:

```
User
 ├─ KitchenProfile
 ├─ TasteProfile
 ├─ NutritionGoal
 ├─ HouseholdProfile[]
 └─ CookingMemory[]

Recipe
 ├─ RecipeVersion[]
 ├─ Ingredient[]
 ├─ Step[]
 └─ RecipeLineage

CookingSession
 ├─ PlannedRecipeVersion
 ├─ ActualIngredientState[]
 ├─ CookingAction[]
 ├─ Timer[]
 ├─ HeatEvent[]
 ├─ RescueEvent[]
 ├─ Rating
 └─ Feedback

CookingExperiment
 ├─ SourceTip
 ├─ RecipeVersionBefore
 ├─ AppliedChange
 └─ Outcome
```

## Canonical units

Internally prefer:
- solid ingredients: g;
- liquids: ml;
- discrete ingredients: count;
- temperature: °C;
- time: seconds.

Display units are user-facing translations.

---

# 11. Adaptation Layers

Do not destructively edit recipes.

Use layers:

```
SourceRecipe
  + ServingAdjustment
  + HouseholdAdjustment
  + TasteAdjustment
  + NutritionAdjustment
  + AvailabilitySubstitution
  = AdaptedCanonicalRecipe

AdaptedCanonicalRecipe
  + KitchenProfile
  = DisplayRecipe

DisplayRecipe
  + ActualCookingActions
  = FinalCookingResult
```

This architecture is important for explainability, undo, recipe diff, fork lineage, and nutrition recalculation.

---

# 12. Voice Processing Architecture

```
Microphone
   ↓
STT
   ↓
Intent Parser
   ↓
Confidence Gate
   ├─ high confidence + deterministic command
   │      ↓
   │   State Engine
   │
   └─ reasoning required
          ↓
         LLM
          ↓
   Structured Action
          ↓
Validation
          ↓
Cooking State
          ↓
UI Render
```

Examples that should normally avoid a large LLM:
- next/previous;
- timer;
- undo;
- explicit ingredient amount;
- explicit heat change.

Examples likely requiring reasoning:
- substitutions;
- rescue;
- ambiguous questions;
- taste adaptation;
- recipe explanation.

All LLM outputs that mutate Cooking State should be converted into validated structured actions first.

---

# 13. Uncertainty & Correction

Voice recognition will fail. Correction UX is therefore core, not an edge case.

Support:

```
"방금 거 취소"
"간장 말고 참기름"
"한 숟갈 아니고 반 숟갈"
```

State-changing actions should retain:
- raw transcript;
- parsed action;
- confidence;
- correction chain.

Low-confidence critical values should trigger a compact confirmation.

Never silently convert:

> "한... 두 숟갈 정도"

into an exact 2 spoon measurement.

---

# 14. End-of-Cook Recap

After completion:

```
오늘의 제육볶음

23분
3인분

변경
고추장 2T → 1.5T
설탕 1T → 0.5T
양파 +0.5개

최종 영양
580 kcal / 인분
단백질 42g

내 평가
★★★★★
"단맛 줄인 게 좋았음"

[ 내 레시피에 반영 ]
[ 다음에도 그대로 ]
[ 공유 ]
```

The recap is generated automatically from session state. Do not make the user manually write a cooking log.

---

# 15. Primary Screens

MVP:

1. **Home**
   - Start cooking
   - Cook Again
   - Recent successful dishes

2. **Recipe Setup**
   - servings / eaters
   - source recipe
   - adaptation summary
   - My Kitchen translation

3. **Cooking HUD**
   - current step
   - heat
   - timer
   - actual ingredients
   - microphone

4. **Cooking Complete**
   - actual changes
   - rating
   - short feedback
   - final recipe

5. **Cooking Memory / My Recipe**
   - history
   - successful version
   - Cook Again

P1/P2:
6. Fork preview + Diff
7. Nutrition adaptation
8. Community / Experiments
9. My Kitchen settings

---

# 16. MVP Acceptance Scenario

The MVP is successful technically if this scenario works end-to-end:

1. User selects 제육볶음.
2. App converts recipe measurements into My Kitchen units.
3. User starts Cooking Mode.
4. User says "간장 한 숟가락 넣었어."
5. Actual ingredient state updates.
6. User says "설탕은 반 숟가락만 넣을게."
7. Planned and actual values remain separately recorded.
8. User says "3분 타이머."
9. Timer starts without LLM reasoning.
10. User says "간장 말고 참기름이야."
11. Previous action is corrected.
12. User completes cooking.
13. App creates final actual recipe.
14. User rates it highly.
15. Session becomes Cooking Memory.
16. Home shows "Cook Again".
17. Next session reproduces the successful actual recipe.

If this loop is not excellent, do not prioritize community or refrigerator features.

---

# 17. Product Roadmap

## P0 — Prove the core loop
- Voice Cooking
- Intent routing
- Cooking State
- My Kitchen measurement translation
- Actual vs planned ingredient tracking
- correction / undo
- Cooking Memory
- Cook Again

## P1 — Make it meaningfully different
- Recipe Evolution
- Cooking Rescue
- Recipe Fork
- Recipe Diff
- deterministic final nutrition calculation
- Nutrition Adaptation

## P2 — Build network effects
- Community recipes
- structured tips
- Cooking Experiments
- recipe lineage
- taste-similar experience signals
- Household Profile

## P3 — Convenience ecosystem
- fuzzy refrigerator state
- contextual menu recommendation
- shopping list
- image recognition
- camera-assisted cooking questions
- optional TTS / hands-free mode

---

# 18. What We Explicitly Do NOT Lead With

Do not market the product primarily as:
- AI recipe generator;
- photo-to-recipe app;
- refrigerator inventory app;
- ingredient-based recipe finder;
- calorie counter;
- generic cooking social network.

Those can be features.

The product identity is:

> **AI that remembers how you actually cook.**

---

# 19. Key Product Metrics

Early MVP:
- Cooking Session completion rate
- voice command success/correction rate
- percentage of sessions with actual recipe deviations captured
- Cook Again usage
- repeat dish cooking rate
- post-cook rating completion

Later:
- successful recipe reproduction rate
- Recipe Evolution adoption
- Fork → completed cook conversion
- saved tip → actual experiment conversion
- experiment positive outcome rate
- 7/30-day cooking retention

Avoid vanity metrics such as generated recipe count.

---

# 20. Product North Star

A user should eventually be able to say:

> **"제육볶음 하자."**

And the app should already understand:
- who usually eats it;
- how many servings;
- the user's preferred sweetness/spiciness;
- the user's real measuring spoon;
- the user's cookware and heat scale;
- the most successful previous version;
- any saved experiment worth trying;
- optional nutrition goal.

Then it should prepare the recipe and guide the cooking session with minimal interaction.

That is the long-term experience.

---

## Final Product Definition

**AI Cooking Companion = Personal Cooking Memory + Real-time Cooking State + Kitchen Translation + Recipe Evolution**

The core promise:

> **오늘의 요리를 기록하는 것이 아니라, 오늘의 요리가 자동으로 내일의 더 좋은 레시피가 된다.**
