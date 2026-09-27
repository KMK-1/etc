# Quality Evidence Markup — Product & UX Concept

> 캡처 이미지에서 문제 부위를 가장 빠르고 명확하게 표시하기 위한 웹 기반 마크업 도구.
>
> **핵심 목표:** PowerPoint나 범용 이미지 편집기보다 적은 조작으로 품질 이슈 증거 이미지를 완성한다.

## 1. Product Principle

이 서비스는 MDF/MF4 자체를 분석하는 도구가 아니다. CANape, CANoe, INCA, Excel, 시험 화면 등에서 사용자가 이미 캡처한 이미지를 받아 **문제 위치 표시 → 설명 → 공유**를 극단적으로 빠르게 만드는 도구다.

대표 UX:

```text
Capture → Ctrl+V → Mark → Ctrl+C → Outlook / Teams / PPT
```

단순한 문제 표시라면 다음처럼 끝나는 경험을 목표로 한다.

```text
Ctrl+V → 문제 위치 더블클릭 → Ctrl+C
```

### 하지 않을 것

- Photoshop 수준의 이미지 편집
- PowerPoint 수준의 도형/서식 기능
- MDF/MF4 분석 엔진
- 복잡한 레이어 패널
- 기본 흐름에서 파일 저장을 강제하는 UX

---

## 2. Main Workspace

```text
┌──────────────────────────────────────────────┬──────────────┐
│                                              │ QUICK MARK   │
│                                              │              │
│           Captured Image                     │ ○ Circle     │
│                                              │ □ Box        │
│ Vehicle Speed ─────────╲____                 │ → Arrow      │
│                         ↑                    │ T Text       │
│                       ① Drop                 │ ① Number     │
│                                              │ ▌ Region     │
│ ACC State ────────────┐                      │ ✂ Crop       │
│                       └────                  │ Blur         │
│                                              │              │
│                                              │ Undo / Redo  │
│                                              │ Copy Result  │
└──────────────────────────────────────────────┴──────────────┘
```

중앙은 이미지/Canvas에 최대한 많은 공간을 사용한다. 우측 Sidebar는 자주 사용하는 도구만 노출하며, 세부 속성은 선택한 객체 주변의 작은 Floating Toolbar로 처리한다.

---

## 3. Image Input

### Priority 1 — Clipboard

`Ctrl+V`로 클립보드 이미지를 즉시 Canvas에 표시한다.

업무 흐름에서 **파일 저장 → 업로드 → 파일 선택** 단계를 제거하는 것이 가장 중요하다.

### 추가 입력

- Drag & Drop
- Image upload
- 여러 이미지 연속 Paste
- 기존 Canvas 교체 / 추가 선택

---

## 4. Core Markup Tools

### Circle

드래그만으로 원/타원을 생성한다.

Default:
- transparent fill
- 강조용 테두리
- 적당히 굵은 stroke

도구를 다시 선택하지 않아도 연속 생성할 수 있는 **Sticky Tool Mode**를 지원한다.

### Rectangle

Circle과 동일한 철학으로 동작한다.

- Drag → 생성
- resize handle
- move
- delete

### Arrow

시작점 → 끝점 Drag로 생성한다.

화살표 생성 직후 바로 키보드를 입력하면 Arrow Label을 생성할 수 있다.

```text
        ACC 해제
            ↓
────────────▼────────
```

별도의 Text 도구 선택을 줄이는 것이 목적이다.

### Text

Canvas를 클릭하면 즉시 입력한다.

- Enter: 완료
- Esc: 취소
- Drag: 이동
- 최소한의 font size 조절

### Number Marker

클릭할 때마다 번호가 자동 증가한다.

```text
① 속도 순간 Drop
② ACC State OFF
③ Brake Request 발생
```

번호 순서는 사용자가 reset/reorder 할 수 있다.

---

## 5. Quick Mark

서비스의 대표 기능 후보.

### Double Click Mark

Canvas의 문제 위치를 더블클릭하면 기본 문제 Marker를 즉시 생성한다.

예:

```text
      ○
      ①
```

사용자가 선호하는 기본 Marker 형태는 설정에서 변경할 수 있다.

### Keyboard Quick Tools

| Key | Action |
|---|---|
| 1 | Circle |
| 2 | Rectangle |
| 3 | Arrow |
| 4 | Text |
| 5 | Number Marker |
| R | Region Highlight |
| C | Crop |
| B | Blur |
| V / Esc | Select |
| Delete | Delete selected |
| Ctrl+Z | Undo |
| Ctrl+Shift+Z | Redo |
| Ctrl+C | Copy result |
| Ctrl+V | Paste image |
| Space + Drag | Pan |
| Mouse Wheel | Zoom |

단축키는 텍스트 입력 중에는 동작하지 않아야 한다.

---

## 6. Region Highlight

MDF/CANape 그래프 캡처에서 특히 중요한 기능.

시간축의 특정 구간을 Drag하면 여러 Signal Graph를 세로로 관통하는 반투명 영역을 만든다.

```text
                  Problem Region
                ┌──────────────┐
Speed ──────────┤              ├────────
                │              │
ACC ────────────┤              ├────────
                │              │
Brake ──────────┤              ├────────
                └──────────────┘
```

선택적으로 상단에 Label을 입력할 수 있다.

예:
- 문제 발생 구간
- 제동 발생
- CAN Timeout
- SW 변경점

---

## 7. Quick Labels / Quality Stamps

반복 입력을 줄이기 위한 업무용 Preset.

기본 후보:

- NG
- OK
- 발생
- 복귀
- 정상
- 비정상
- 변경점
- 의심 구간
- 개선 전
- 개선 후

사용자가 Custom Preset을 추가할 수 있다.

예:

```text
ACC 해제
DTC 발생
CAN Timeout
신호 미출력
제동 발생
```

Preset 클릭 → Cursor에 Label 부착 → Canvas 클릭으로 배치한다.

---

## 8. Smart Annotation Interaction

객체를 선택했을 때 전체 Sidebar를 바꾸기보다 객체 근처에 작은 Floating Toolbar를 표시한다.

```text
        ┌─────────────────────────┐
   ○ ←  │ Color │ Width │ Copy │ 🗑 │
        └─────────────────────────┘
```

자주 쓰지 않는 설정은 숨긴다.

### 공통 Object 동작

- Click: select
- Drag: move
- Handle drag: resize
- Delete: remove
- Ctrl+C / Ctrl+V: annotation duplicate
- Ctrl+D: duplicate
- Shift: 비율/축 constraint
- Esc: deselect

---

## 9. Zoom & Navigation

큰 로그 캡처에서도 정확하게 표시할 수 있어야 한다.

- Mouse Wheel: Zoom
- Space + Drag: Pan
- Fit to Screen
- 100%
- 200%

**중요:** Canvas zoom과 최종 이미지 해상도를 분리한다.

사용자가 200%로 확대하여 편집해도 결과물은 원본 이미지 좌표/해상도를 기준으로 렌더링한다.

---

## 10. Crop & Focus

### Standard Crop

영역 Drag → Enter → Crop.

### Crop to Selection

문제 영역을 선택하면 적절한 margin을 포함하여 자동 Crop한다.

### Keep Context / Focus Mode

전체 이미지는 유지하면서 선택 영역 외부를 약하게 Dim 처리한다.

```text
██████████████████████████
████ ┌─────────────┐ █████
████ │ Problem     │ █████
████ │   ○         │ █████
████ └─────────────┘ █████
██████████████████████████
```

보고자료에서 문제 위치를 즉시 인지시키는 용도다.

---

## 11. Before / After Compare

두 이미지를 넣으면 빠르게 비교 Layout을 생성한다.

```text
┌──── BEFORE ────┐   ┌──── AFTER ─────┐
│                │   │                 │
│       ○        │   │        ✓        │
│       NG       │   │                 │
│                │   │                 │
└────────────────┘   └─────────────────┘
```

지원 후보:

- Before / After
- NG / OK
- Vehicle A / Vehicle B
- SW Before / SW After
- Vertical / Horizontal layout

두 이미지의 Zoom/Pan 동기화는 후속 기능으로 고려한다.

---

## 12. Clipboard-first Output

가장 중요한 Output은 Download가 아니라 **Copy**다.

`Copy Result` 또는 `Ctrl+C`:

1. Base image
2. 모든 annotation
3. Crop/Focus 효과

를 합성한 최종 이미지를 Clipboard에 복사한다.

그 후 바로:

- Outlook
- Teams
- PowerPoint
- Excel
- 메신저

등에 Paste할 수 있어야 한다.

추가 Output:

- PNG
- JPG
- Original resolution export

---

## 13. Undo / Redo

모든 편집 작업은 Command History로 관리한다.

대상:

- create
- delete
- move
- resize
- style
- text
- crop
- region
- image placement

Undo/Redo는 신뢰성이 매우 중요하다.

---

## 14. UX Rules

### Rule 1 — One action should usually equal one result

Circle을 만들기 위해 메뉴를 여러 단계 거치지 않는다.

```text
Bad
Insert → Shape → Circle → Style → Fill → Border

Good
Circle → Drag
```

### Rule 2 — Useful defaults over settings

사용자가 설정하지 않아도 처음부터 업무에 사용할 수 있는 Marker가 생성되어야 한다.

### Rule 3 — Repetition should become one click

반복되는 텍스트와 Marker는 Preset/Quick Label로 전환한다.

### Rule 4 — Clipboard is first-class

Upload/Download보다 Paste/Copy가 더 빠른 경로여야 한다.

### Rule 5 — Canvas space matters

도구 패널 때문에 원본 그래프가 지나치게 작아지면 안 된다.

### Rule 6 — Progressive complexity

초기 화면에는 필수 도구만 보이고, 고급 기능은 필요할 때만 노출한다.

---

## 15. MVP Scope

### P0 — 반드시 구현

- Clipboard image paste
- Drag & Drop / Upload
- Circle
- Rectangle
- Arrow
- Text
- Number Marker
- Region Highlight
- Select / Move / Resize
- Undo / Redo
- Zoom / Pan
- Crop
- Copy final image to Clipboard
- PNG export

### P1 — 사용성 차별화

- Double Click Quick Mark
- Keyboard shortcuts
- Sticky Tool Mode
- Quick Labels
- Custom Presets
- Floating Toolbar
- Focus / Keep Context
- Arrow + immediate text
- Original resolution rendering

### P2 — 확장

- Before / After
- Blur / Mosaic
- Multiple image layout
- annotation templates
- reusable style presets
- synchronized compare zoom
- annotation grouping
- image history / recent workspace

---

## 16. MVP Success Criteria

첫 버전의 성공 여부는 기능 개수가 아니라 **작업 시간 감소**로 판단한다.

대표 Scenario:

### Scenario A — 단일 문제 표시

```text
Ctrl+V
→ Double Click problem
→ Ctrl+C
```

목표: **5초 이내**

### Scenario B — 문제 위치 + 설명

```text
Ctrl+V
→ Arrow drag
→ "ACC 해제" 입력
→ Ctrl+C
```

목표: **10초 이내**

### Scenario C — 여러 문제 순서 표시

```text
Ctrl+V
→ Number Marker
→ ① click
→ ② click
→ ③ click
→ Ctrl+C
```

목표: **15초 이내**

### Scenario D — 로그 문제구간 강조

```text
Ctrl+V
→ Region
→ Drag problem interval
→ label
→ Ctrl+C
```

목표: **10초 이내**

---

## 17. Product Identity

이 제품은 범용 이미지 Editor가 아니다.

> **Quality Evidence Markup Tool**

특히 다음 업무를 빠르게 지원한다.

- CANape/CANoe/INCA 로그 캡처
- 차량 시험 결과
- DTC/Signal 이상 증거
- Excel/Table 캡처
- SW 변경 전후 비교
- 외관/부품 사진의 문제 위치 표시
- 협력사/R&D 전달용 Evidence
- 회의/PPT/메일용 Issue Screenshot

제품 경쟁력은 많은 기능이 아니라 **문제 위치를 표시하고 전달하기까지의 클릭 수를 얼마나 줄이느냐**에 있다.

---

## 18. Recommended First Prototype

첫 Prototype에서는 UI 디자인보다 Interaction 검증을 우선한다.

검증할 질문:

1. Paste 후 즉시 편집 가능한가?
2. Circle/Arrow를 PPT보다 빠르게 만들 수 있는가?
3. Annotation 선택/수정이 자연스러운가?
4. Zoom 상태에서도 정확하게 표시되는가?
5. Copy 후 Outlook/Teams/PPT에 바로 붙여넣을 수 있는가?
6. Quick Mark가 실제로 작업 시간을 줄이는가?
7. Region Highlight가 로그 분석 캡처에서 유용한가?
8. Number Marker가 설명 순서를 명확하게 만드는가?
9. 사용자가 설명 없이도 주요 기능을 발견할 수 있는가?
10. 10~20초 안에 대부분의 Evidence 이미지가 완성되는가?

이 질문들을 통과한 뒤 로그인, 사내망 배포, 저장소, 사용자 관리 등의 서비스 인프라를 설계한다.
