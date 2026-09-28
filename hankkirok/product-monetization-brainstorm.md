# 한끼록 (HANKKIROK) — Product & Monetization Brainstorm

> 목적: 한끼록 개발 과정에서 나온 제품 방향, 수익화 아이디어, 음성 UX 및 비용 전략을 장기적으로 보존하기 위한 브레인스토밍 노트.
>
> 이 문서는 확정 요구사항이 아니라 제품 의사결정을 위한 living document다.

## 1. Product thesis

한끼록은 단순한 레시피 저장 앱이나 범용 AI 레시피 챗봇을 목표로 하지 않는다.

핵심 방향은 **"나에게 맞춰 계속 진화하는 개인 요리 시스템"**이다.

핵심 사용자 루프:

```text
레시피 발견/가져오기
  → 구조화
  → 내 기준으로 변환
  → 실제 요리
  → 요리 중 수정 기록
  → 내 Recipe/Fork로 축적
  → 다음 요리에 재사용
```

장기적으로 사용자가 축적할 핵심 자산:

- 개인 Recipe/Fork history
- 계량/단위 preference
- Taste Profile
- 조리 중 실제 수정 이력
- 영양/칼로리 목표
- 가족 단위 preference

## 2. Deterministic core

AI가 모든 레시피 JSON을 임의로 다시 작성하는 구조보다 다음 구조를 지향한다.

```text
User intent
  → AI interpretation (필요한 경우)
  → structured transformation
  → deterministic domain operation
  → RecipeVersion
  → RecipeGraph
  → persistence
```

RecipeGraph / Transformation / Fork provenance는 제품 기능의 기반이다.

예:

```text
원본 RecipeVersion
  → Fork
  → 2인분 변환
  → 개인 계량 기준 적용
  → 맛 조정
  → Child RecipeVersion
  → 최종 영양/칼로리 계산
```

원본과 변경 이력을 추적 가능하게 유지하는 것이 중요하다.

## 3. Free / Plus monetization hypothesis

초기에는 핵심 경험을 충분히 무료로 제공해 retention을 검증한다.

### Free 후보

- 레시피 저장/작성
- 기본 Fork
- 기본 인분 조절
- 기본 계량 변환
- Cooking Mode
- 개인 수정 기록
- 제한적인 음성 명령
- 제한적인 AI 사용

### HANKKIROK Plus 후보

초기 가격 가설: 월 3,900~5,900원 범위에서 검증.

- 고급 AI Recipe Fork
- 목적 기반 레시피 변환
- 개인 계량 Profile
- Taste Profile 기반 자동 조정
- 상세 영양/칼로리 분석
- 다이어트/고단백 등 목표 기반 transformation
- 높은 AI 사용량
- 고급 Voice Cooking Assistant
- 가족 Profile / 가족 공유

유료화의 핵심 메시지는 기능 개수가 아니라:

> **"한끼록이 나를 알고, 다른 레시피를 자동으로 내 방식으로 바꿔준다."**

## 4. Voice strategy

ChatGPT Voice와 같은 full realtime voice-to-voice를 기본 구조로 만들 필요는 없다.

한끼록에 적합한 기본 구조:

```text
사용자 음성
  ↓
STT
  ↓
Command Router
  ├─ 단순 명령 → local/deterministic handler
  ├─ recipe 변경 → Transformation engine
  └─ 복잡한 자연어 → LLM
  ↓
화면 텍스트 응답
```

사용자는 손이 자유롭지 않은 요리 상황에서 음성을 사용하고, AI는 화면에 간결한 텍스트를 표시하는 것을 기본 UX로 고려한다.

예:

- "다음 단계"
- "이전"
- "타이머 5분"
- "간장 반 스푼 더 넣었어"
- "이거 몇 칼로리야?"

### 비용 원칙

모든 음성 명령을 LLM으로 보내지 않는다.

단순 명령은 local parser / deterministic handler에서 처리하고, recipe 변경은 구조화된 Transformation으로 처리한다. 의미 해석이 필요한 경우에만 LLM을 호출한다.

이 구조는:

- API 비용 절감
- latency 감소
- 결과 결정성 향상
- RecipeGraph/Transformation과 자연스러운 연결

이라는 장점이 있다.

### Free vs Plus Voice 가설

**Free**
- push-to-talk
- 짧은 음성 명령
- 다음/이전/완료/타이머
- 제한적인 recipe 수정 기록

**Plus**
- 더 자연스러운 연속 음성 Cooking Assistant
- 복합 자연어 recipe 수정
- Taste Profile을 고려한 제안
- 조리 상황 기반 질문/답변
- 더 높은 음성/AI 사용량

Realtime audio-to-audio는 비용과 UX 가치가 실제로 입증된 이후 선택적으로 검토한다.

## 5. Taste Profile

장기 차별화 후보 중 하나.

요리 후 또는 조리 중 사용자가 남기는 피드백:

- 조금 짰음
- 조금 달았음
- 매웠음
- 고기 양 적당
- 다음에는 마늘 더
- 소스가 부족했음

이 기록을 누적해 개인 Taste Profile을 구축한다.

향후 다른 Recipe를 Fork할 때:

```text
원본
간장 3T
설탕 2T
고춧가루 2T

↓ 내 입맛 적용

간장 2.5T
설탕 1.3T
고춧가루 1.5T
```

처럼 **제안**할 수 있다.

AI가 제안하더라도 실제 저장되는 변경은 가능하면 명시적인 structured Transformation으로 표현한다.

## 6. Recipe import

성장에 중요한 후보 기능.

사용자가 외부에서 발견한 레시피를 한끼록으로 가져온 뒤 즉시 자신의 기준으로 변환할 수 있어야 한다.

목표 UX:

```text
공유 URL / 텍스트 / 사진
  → 한끼록 Import
  → Recipe 구조화
  → 내 인분/계량/Taste 설정 적용
  → Cooking Mode
  → 내 Fork 저장
```

외부 서비스의 이용약관, 저작권 및 API 정책은 실제 구현 시 별도로 검토한다.

## 7. Fork as growth loop

Fork는 단순 편집 기능이 아니라 공유/성장 기능으로 발전할 수 있다.

예:

```text
원본 제육볶음
  ↓
A의 덜 단 2인분 Fork
  ↓ 공유
B가 자신의 기준으로 Fork
  ↓
C가 다시 Fork
```

공유 화면에서는 원본 대비 변경점을 설명할 수 있다.

예:

- 설탕 -35%
- 4인분 → 2인분
- 고기 600g → 400g
- 개인 계량 기준 적용
- 최종 1인분 kcal

CTA 후보:

**내 한끼록으로 Fork**

Recipe provenance가 이 growth loop의 기술적 기반이 된다.

## 8. Nutrition / diet

칼로리와 영양 계산은 원본 Recipe가 아니라 **최종 effective RecipeVersion**을 기준으로 계산해야 한다.

개념:

```text
normalized ingredient identity
+ effective quantity
+ nutrition reference data
→ deterministic nutrition calculation
```

Diet Mode 역시 별도의 임의 AI recipe generator보다는 Transformation engine 위에 구축하는 것을 우선 검토한다.

예:

- 저칼로리
- 고단백
- 저염
- 당류 감소

AI는 변경안을 제안할 수 있지만 deterministic calculation과 provenance는 유지한다.

## 9. Commerce opportunity

장기적으로 RecipeGraph가 필요한 식재료를 알고 있다면 장보기와 연결할 수 있다.

```text
오늘의 Recipe
  ↓
필요 재료
  ↓
집에 있는 재료 제외
  ↓
Shopping List
  ↓
식재료 구매 연결
```

배너 광고보다 cooking workflow 안에 자연스럽게 들어가는 affiliate/commerce가 제품 경험과 더 잘 맞을 가능성이 있다.

구독 이후의 추가 수익원 후보:

- 식재료/장보기 affiliate
- 주방용품 제휴
- creator recipe pack
- family plan
- 장기적으로 B2B food/nutrition integrations

## 10. Revenue milestones (hypothesis)

예시 가격을 월 4,900원으로 가정할 경우:

- 유료 사용자 약 205명 → 월 구독 매출 약 100만원
- 유료 사용자 약 2,041명 → 월 구독 매출 약 1,000만원

이는 gross subscription revenue 단순 계산이며 실제 순수익은 앱스토어 수수료, 결제 수수료, AI/STT 비용, 서버 비용, 세금 등에 따라 달라진다.

초기에는 매출보다 retention과 반복 사용을 먼저 검증한다.

## 11. Metrics to watch before aggressive monetization

다운로드 수보다 다음 행동 지표를 우선 본다.

1. Recipe import/save → 실제 Cooking Mode 시작률
2. 첫 요리 → 두 번째 요리 재방문율
3. Fork / personalization 반복 사용률
4. 음성 기능 사용 후 조리 완료율
5. 사용자가 실제 변경사항을 자신의 Recipe에 저장하는 비율
6. 공유 Recipe → Fork conversion

이 지표가 약하면 paywall 최적화보다 core product loop를 먼저 개선한다.

## 12. Tentative product roadmap

현재 deterministic foundation을 먼저 완성한다.

```text
PHASE 1  RecipeGraph
PHASE 2  Persistence + Transformation/Fork
PHASE 3  Quantity & Unit Engine
PHASE 4  Nutrition
PHASE 5  Taste Profile
PHASE 6  Recipe Import
PHASE 7  Cooking Mode + Voice
PHASE 8  Sharing / Fork discovery
PHASE 9  Plus / Billing
PHASE 10 Commerce experiments
```

실제 repository 상태와 사용자 검증 결과에 따라 순서는 변경할 수 있다.

## 13. Product principle

한끼록의 장기 방향을 한 문장으로 표현하면:

> **레시피를 저장하는 앱이 아니라, 다른 사람의 레시피를 나의 계량·입맛·목표·조리 습관에 맞춰 계속 진화시키는 개인 Cooking System.**

AI 자체를 상품으로 파는 것이 아니라, 사용자의 누적된 요리 맥락과 deterministic transformation을 결합한 개인화 경험을 상품으로 만든다.


## 14. On-device AI strategy

장기적으로 한끼록의 기본 음성 기능은 **On-device First + Cloud Fallback** 구조를 우선 검토한다.

On-device란 한끼록 서버나 외부 AI API가 아니라 사용자의 스마트폰 내부 CPU/GPU/NPU에서 작은 AI 모델과 parser를 직접 실행하는 방식이다.

목표 구조:

```text
사용자 음성
  ↓
[On-device]
음성 감지 / STT / command parsing
  ↓
명확한 명령
  ├─ 다음 / 이전
  ├─ 타이머
  ├─ 단위/수량 인식
  └─ 간단한 ingredient modification
  ↓
deterministic Recipe/Transformation engine

애매하거나 복잡한 자연어
  ↓
Cloud AI fallback
  ↓
structured transformation proposal
  ↓
deterministic validation/application
```

### 기대 효과

- 사용자 증가에 따른 중앙 GPU/STT 서버 부하 증가폭 감소
- API 및 inference 운영비 절감
- 짧은 명령의 latency 감소
- 기본 기능의 offline 동작 가능성
- 음성 원본을 서버로 보내지 않는 privacy-friendly UX 가능
- 사용자 수가 증가하면 각 사용자 기기의 연산 자원을 활용하므로 중앙 inference bottleneck을 줄일 수 있음

### Free / Plus와의 연결

**Free 후보**
- push-to-talk
- on-device STT/command
- 다음/이전/완료
- 타이머
- 간단한 ingredient quantity 수정
- offline-capable basic cooking commands

**Plus 후보**
- hands-free / continuous voice
- 복합 자연어 이해
- Taste Profile 기반 조정
- 상황형 Cooking Assistant
- cloud LLM fallback의 높은 사용량

비용이 큰 기능을 구독 가치와 연결하되, 기본 Cooking Mode가 네트워크/API 장애 때문에 사용 불가능해지지 않는 것을 목표로 한다.

### Engineering caution

On-device는 무료 인프라가 아니다. 다음 trade-off를 실제 기기에서 측정해야 한다.

- 앱/모델 다운로드 크기
- RAM 사용량
- 배터리
- 발열
- inference latency
- iOS/Android 구현 차이
- 구형 기기 성능
- 한국어 요리 표현 인식 정확도

따라서 초기 MVP부터 모든 AI를 on-device화하지 않는다.

개발 단계 가설:

```text
PoC
개발 PC에서 Korean STT + parser 검증
  ↓
MVP/Beta
Cloud STT + deterministic parser 중심으로 빠르게 검증
  ↓
실사용 데이터/명령 패턴 확보
  ↓
자주 쓰는 command부터 on-device 이전
  ↓
Cloud fallback 최적화
```

핵심 원칙:

> **가능한 요청은 가장 저렴하고 빠른 계층에서 해결하고, 복잡성이 높아질 때만 상위 AI 계층으로 escalation한다.**

```text
On-device command
      ↓ 필요 시
Deterministic engine
      ↓ 필요 시
Small/local model
      ↓ 필요 시
Cloud high-capability LLM
```

이 구조를 한끼록의 장기 AI inference architecture 후보로 유지한다.
