# AI Cooking Companion — MVP UX Specification

## UX thesis

The app is not a chat screen.

During cooking the user should understand the next action in roughly one glance.

> 손은 요리하고, 입으로 입력하고, 눈으로 확인한다.

# Screen 1 — Home

Primary hierarchy:

```
오늘 뭐 해먹을까요?

[ 🎙 요리 시작 / 메뉴 말하기 ]

지난번 맛있었던 요리
┌────────────────────┐
│ 제육볶음       ★★★★★ │
│ 3일 전 · 23분        │
│ [ 그대로 다시 만들기 ] │
└────────────────────┘

최근 요리
김치찌개
계란말이
된장찌개
```

MVP priorities:
1. Start cooking
2. Cook Again
3. Recent memories

Do not lead with social feed or refrigerator dashboard.

# Screen 2 — Recipe Setup

```
제육볶음

2인분   [-] 2 [+]

내 주방 기준
🥄 간장 15ml → 내 숟가락 약 1.7회
🔥 중불 → 인덕션 약 5단
🍳 28cm 프라이팬

이번 레시피 변경
설탕 10g → 5g
└ 지난번 만족했던 양

[ 요리 시작 ]
```

Rules:
- Separate recipe changes from Kitchen Translation.
- Show "why" for personalized changes.
- Allow "원본으로" reset.
- Advanced settings remain collapsed.

# Screen 3 — Cooking HUD

Most important screen.

```
제육볶음                         4 / 7

양념을 넣고
2분간 볶아주세요

🔥 5단                   ⏱ 01:42

실제 투입
✓ 돼지고기 500g
✓ 고추장 내 숟가락 1.5
✓ 설탕 내 숟가락 0.5
○ 대파

방금 들은 말
"설탕 반 숟갈 넣었어"       [수정]

              [ 🎙 ]
```

Interaction:
- Push-to-talk button is the dominant control.
- Swipe/tap fallback for next/previous.
- Active timer remains visible across steps.
- Ingredient state is scannable, not paragraph text.
- Current instruction uses large typography.
- Screen should remain awake during active cooking where platform permits.

## Voice feedback

On successful parse:

```
✓ 설탕 0.5스푼 기록
```

On uncertain parse:

```
간장 양을 정확히 못 들었어요.
[1스푼] [2스푼] [모르겠음]
```

On correction:

```
↩ 간장 기록 취소
✓ 참기름 1스푼으로 수정
```

# Screen 4 — Rescue Overlay (P1)

User:
> "너무 짠데?"

Overlay:

```
요리 복구

현재 상태 기준
① 물 80–100ml 먼저 추가
② 맛을 다시 확인
③ 필요하면 두부 100g 추가

[ 물 100ml 넣었어 ]
[ 다른 방법 ]
```

Rescue should not replace the Cooking HUD. It is a temporary layer.

# Screen 5 — Complete

```
오늘의 제육볶음

23분 · 2인분

실제로 달라진 점
고추장 2T → 1.5T
설탕 1T → 0.5T
마늘 +0.5T

맛은 어땠나요?
☆ ☆ ☆ ☆ ☆

[ 딱 좋아요 ]
[ 조금 달아요 ]
[ 조금 짜요 ]
[ 직접 말하기 ]

[ 완료 ]
```

P1 nutrition block:

```
최종 영양 · 1인분
🔥 580 kcal
💪 단백질 42g
탄수화물 26g · 지방 32g

계산 커버리지 96%
```

# Screen 6 — Cooking Memory

```
우리집 제육볶음

⭐ Best Version

고추장 내 숟가락 1.5
설탕 내 숟가락 0.5
마늘 내 숟가락 1.5
인덕션 6단 · 약 6분

"단맛 이 정도가 딱 좋음"

[ 그대로 다시 만들기 ]

History
v3  ★★★★★
v2  ★★★★☆
v1  ★★★☆☆
```

# Screen 7 — My Kitchen

Onboarding should ask only for high-value calibration.

```
내 계량도구

밥숟가락
[ 9 ] ml

계량 큰술
[ 15 ] ml

내 화구
인덕션
1 ───────── 9

자주 쓰는 팬
28cm 프라이팬
```

Do not force full kitchen setup before first cook.

Allow:
- Skip
- Add later
- Calibrate after cooking

# P1 — Fork Preview

```
민수님의 제육볶음
        ↓
내 주방으로 가져오기

Recipe 변화
설탕       10g → 5g
마늘       10g → 15g

내 주방 표시
간장 15ml → 내 숟가락 1.7회
중불      → 인덕션 약 5단

[ 이 버전으로 요리하기 ]
```

Never present Kitchen Translation as a Recipe Diff.

# P1 — Nutrition Adaptation

```
현재             587 kcal
목표             500 kcal 이하

추천 조정
밥 180 → 130g       -65 kcal
기름 5 → 3ml        -18 kcal
설탕 5 → 3g          -8 kcal

예상             496 kcal

[ 적용 ]
[ 밥은 그대로 ]
```

# Navigation

MVP bottom navigation:

```
홈     내 레시피     내 주방
```

Cooking Mode hides normal navigation to reduce accidental exits.

# Empty states

No memories:
> 첫 요리를 완료하면 맛있었던 조리법을 자동으로 기억해드려요.

No Kitchen Profile:
> 평소 쓰는 숟가락과 화구를 등록하면 레시피를 우리 집 기준으로 바꿔드려요.

# UX guardrails

- No long chat transcript in Cooking Mode.
- No mandatory manual cooking diary.
- No mandatory perfect refrigerator inventory.
- No hidden personalization.
- No auto-saving temporary preference as permanent.
- No nutrition number presented as exact when mapping is incomplete.
- No food-safety certainty based only on an image.
- Voice failure must always have touch fallback.

# MVP usability test

Give a user a seeded 제육볶음 recipe and ask them to cook while deliberately changing:
- one ingredient amount;
- one timer;
- one mistaken voice command.

Success means the user can finish without opening a keyboard, the final recap correctly reflects the changes, and Cook Again reproduces the final successful version.
