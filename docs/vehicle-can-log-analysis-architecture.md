# 차량 CAN 로그 지능형 분석 플랫폼 — 설계 방향 및 개발 핸드오프

> 작성일: 2026-11-23  
> 상태: 브레인스토밍 합의안 / 구현 전 설계 초안  
> 목적: 다음 대화나 Codex 개발 세션에서 이 문서만 읽고 논의를 이어갈 수 있도록 배경, 결정 사항, 구조, 검증 기준과 미결 사항을 기록한다.

## 0. 한눈에 보는 결론

**기존 LLM 로그 분석 챗봇을 유지**하면서, 별도 **자동 분석 워크벤치 탭**을 추가한다. 두 화면은 **동일한 로그 세션, 신호 조회 서비스, 분석 엔진, 이벤트 및 증거 저장소**를 공유한다. 자동 분석은 **차량 상태 + 이벤트 + 이상 탐지 통합 대시보드**를 첫 화면으로 하며, **적응형 하이브리드 분석**(전체 경량 스크리닝 → 상황/이벤트 기반 심층 분석 → 기능별 사양 검증)을 수행한다.

LLM은 질문 해석·분석 계획 제안·결과 설명·규칙 초안 생성을 담당한다. 시간 계산, 신호 처리, PASS/FAIL 판단은 **재현 가능한 결정론적 코드**가 담당한다. 원인 후보는 가설로 표시하고 실제 고장 원인 확정과 구분한다.

## 1. 프로젝트 배경 및 확인된 조건

- 사용자: 자동차 품질 엔지니어. 주행 평가, 정차 기능 평가, 고장 재현 등 다양한 차량 CAN 로그에 범용 적용하고 싶음.
- 기존 프로젝트: 사용자가 로그를 주고 **시그널, 시점, 상황, 레퍼런스**를 자연어로 제공하면 LLM 챗봇 형식으로 분석하는 기능이 존재함.
- 입력: **DBC가 결합된 MF4 + BLF/ASC 혼합**, DBC 확보 가능.
- 기존 챗봇 내부 구현은 **미확인**: LLM 생성 Python 실행인지, 기존 Python 도구 호출인지, 데이터 텍스트 전달인지 모름. 코드 확인 전 재사용 범위/수정 범위 확정 금지.
- 지향점: 기존 기능을 파괴하지 않고 자동 분석 기능을 확장. 자동 분석과 챗봇 사이 이벤트/신호/시간 범위 전달.
- 아직 확보하지 않은 것: 실제 로그 샘플, DBC 스키마/차종 정보, 기존 프로젝트 저장소·기술 스택·코드 구조, 시험 사양/허용치, PC 성능, 목표 처리 용량.
- **본 문서는 실제 코드나 로그를 검사한 결과가 아니라 설계 논의의 기록**이다.

## 2. 확정한 제품 방향

1. **통합형**: 범용 이상 탐지 + 기능별 품질 평가.
2. **UI**: 기존 AI 챗봇 유지, 자동 분석 워크벤치 별도 탭, 규칙 관리 및 분석 이력 추가. 자동 분석 화면에서 AI 보조 패널 호출 가능.
3. **자동 분석 첫 화면**: 차량 상태 + 주요 이벤트 + 이상 의심 구간을 하나의 타임라인/대시보드로 표시.
4. **적응형 하이브리드 엔진**: 모든 관측 가능 신호에 저비용 검사; 상황과 이벤트에 따라 신호 집합과 분석 깊이를 확장. 등록된 기능 평가 규칙은 1차 이상 후보가 없어도 실행.
5. **원본 보존과 추적성**: 원본 파일 변경 금지, 모든 결과에서 원본 파일·채널·메시지·신호·시점으로 추적.
6. **결정론적 판단**: LLM 추론은 승인 전 규칙/사양으로 자동 승격하지 않음.
7. **정상/이상 이분법 지양**: PASS, FAIL, SUSPECT, UNKNOWN, NOT_APPLICABLE, NOT_EXECUTED를 구분.
8. **차종별 차이 흡수**: 신호 이름이 아니라 의미 중심 공통 신호 ID를 사용하고, 차량별 프로파일/DBC 버전/기능별 규칙으로 차이를 처리.

## 3. 실제 품질 엔지니어 워크플로

1. **시험 세션 등록**: 원본 로그, DBC, 차량·ECU SW 버전, 시험 목적, 시험 조건/레퍼런스, 기록 시각 입력.
2. **데이터 건전성 검사**: 파일 읽기/DBC 디코딩 가능 여부, CAN 채널, 메시지 주기, 누락/오버런 가능성, 타임스탬프, invalid 값.
3. **차량 상태 재구성**: 전원, 주행/정차, 가속/감속, 제동, 선회, ADAS 상태 등 여러 독립 상태 축으로 표현.
4. **이벤트 추출**: 기능 요청, 상태 전이, 급변, DTC/통신 이벤트, 특정 조건 진입/이탈.
5. **자동 분석**: 범용 스크리닝 + 기능별 규칙 평가 + 이벤트 관계 분석.
6. **엔지니어 심층 검토**: 이벤트 클릭 → 전후 파형/관련 신호/판정 근거 → 챗봇 질의.
7. **판정 및 보관**: 엔지니어 승인/보류/수정, 근거 저장, 리포트 생성, 재사용 가능한 규칙 후보 등록.

## 4. 사용자 화면 / 상호작용

### 4.1 자동 분석 탭 (기본 화면)

- 상단: 파일명, 차량/시험 메타데이터, DBC 연결 상태, 분석 단계 및 데이터 건전성 상태.
- KPI: 로그 길이, 관측 신호 수, 분석 가능 기능 수, 이벤트 수, 이상 의심 수, 판정 불가 수.
- **다층 타임라인**: 전원/주행·정차/제동/ADAS 상태와 이벤트·이상 마커를 같은 시간축에 표시.
- 우선 검토 리스트: 시각, 현상, 관련 ECU/신호, 심각도, 증거 강도, 판정 상태, 데이터 커버리지.
- 이벤트 상세: 전후 파형, 트리거, 차량 상태, 측정 특징, 사용 규칙/사양, 판정 근거, 원본 링크.
- **이 이벤트를 AI와 분석**: 선택 이벤트의 session_id, event_id, 시간창, 신호 ID, finding ID를 챗봇에 전달. 원본 로그 전체를 프롬프트에 붙이지 않고 공통 조회 API를 사용.
- UI는 **부분 결과를 스트리밍**할 수 있어야 하며 분석 중/실패/취소/재실행 상태를 표시.

### 4.2 기존 챗봇 탭

- 기존 자연어 질문과 사용자 제공 레퍼런스 입력을 유지.
- 신호 탐색, 구간 조회, 통계/응답시간 측정, 규칙 실행 등 **공통 분석 도구**를 호출하도록 점진적 리팩터링.
- 챗봇에서 발견한 현상은 Finding 후보로 저장 가능.
- 사용자 레퍼런스를 규칙 초안으로 변환할 수 있지만 **검토·승인 전 자동 적용 금지**.

### 4.3 규칙/레퍼런스 관리

- 레퍼런스 3종: **사양 레퍼런스**(요구사항/허용치), **사례 레퍼런스**(정상/불량 로그), **상황 레퍼런스**(시험자 관찰·조작·노면 등).
- 입력된 주장과 실제 로그에서 확인된 사실을 별도 출처로 표시.
- 규칙 초안 → 신호 매핑 검증 → 테스트 → 엔지니어 승인 → 버전 발행 → 자동 실행.

## 5. 논리 아키텍처

```text
UI: [자동 분석 워크벤치] [AI 챗봇] [규칙 관리] [분석 이력]
                         |
               API / Session / Job Manager
                         |
  +----------------------+---------------------+
  | Data Engine          | Context Engine      |
  | ingest, decode,      | vehicle states,     |
  | time align, cache    | event extraction    |
  +----------------------+---------------------+
  | Analysis Planner     | Analysis Engine     |
  | select depth/signals | universal + rules   |
  +----------------------+---------------------+
  | Evidence/Findings    | AI Orchestrator     |
  | trace & review       | tools + explanation |
  +----------------------+---------------------+
                         |
  Raw logs + DBC | Metadata DB | Signal Cache | Events/Findings
```

**초기 배포는 모듈형 단일 애플리케이션**을 우선한다. 불필요한 마이크로서비스 분리는 보류한다.

### 5.1 내부 데이터 객체 (초안)

- `LogSession`: session_id, 파일 목록/해시, 시험/차량 정보, DBC 버전, 타임베이스.
- `SignalDefinition`: canonical_signal_id, 원본 채널·CAN ID·메시지·신호, 단위, enum, 유효성/주기 정의, 매핑 상태.
- `SignalSeries`: 원본 시각·값·품질 플래그·원본 위치 참조.
- `VehicleContext`: 상태 축, 시간 구간, 확정/추론/불명, 사용 신호 및 조건.
- `Event`: 관측된 상태 전환/동작/현상, 시작·끝, 관련 신호, 추출 방법.
- `Feature`: 이벤트에서 계산된 응답 지연, 변화율, 편차 등 수치.
- `Finding`: 규칙/알고리즘, 판정, 심각도, 증거 강도, 커버리지, 설명.
- `Evidence`: 원본 참조, 신호/시간창, 파형 조회 정의, 계산식/파라미터, 규칙/엔진 버전.
- `AnalysisRun`: 실행 ID, 사용 설정·캐시 키·알고리즘 버전, 작업 상태/오류.

**Event와 Finding 분리**: AEB 작동은 이벤트이고, 작동 응답 사양 위반은 Finding이다. 하나의 이벤트에 여러 Finding이 연결될 수 있다.

## 6. 데이터 파이프라인

### Stage 0 — Ingestion / Index

- MF4/BLF/ASC 파일 식별, 크기/해시, 채널, 시간 범위, 메시지 목록, DBC 호환성 수집.
- MF4는 신호/그룹 접근을 활용; BLF는 압축 블록/순차 읽기 제약을 고려해 최초 스캔 및 캐시 인덱스 구축.
- 같은 시험의 복수 파일은 시간 기준이 다를 수 있음. 절대 시각·동기화 이벤트를 우선하고, 오프셋/드리프트 추정의 **근거 및 불확실성**을 보존.
- DBC 적용 키에 채널, 프레임 ID, standard/extended 구분, 송신 ECU, multiplex 및 DBC 버전 포함.
- 디코딩된 MF4 신호와 BLF에서 추출한 동일 원천 신호를 중복 증거로 계산하지 않음.

### Stage 1 — Data Quality / 전체 경량 스크리닝

- 디코딩 실패, 신호 유효성/invalid enum, 물리 범위, 결측/수집 공백, 주기 이상, 값 고착, 변화율, 상태 전환 빈도 등.
- **정차·sleep·정상적인 이벤트에 의한 고착/통신 중단은 고장으로 단정 금지**.
- 전체를 메모리에 적재하지 않고 청크/배치 처리. 경계 구간 상태(직전 샘플, 지속 시간, 상태 머신 상태)를 다음 청크로 전달.
- 첫 결과는 '이상 후보'이며 즉시 FAIL로 표시하지 않음.

### Stage 2 — Vehicle Context

- 서로 독립적인 상태 축:
  - Power: OFF / IGN ON / READY / UNKNOWN
  - Motion: STATIONARY / MOVING / UNKNOWN
  - Dynamics: ACCELERATING / DECELERATING / TURNING (동시 가능)
  - Function: AVAILABLE / ACTIVE / DISABLED / FAULT / UNKNOWN (기능별 확장)
- 상태 판정: 유효성 확인 → 히스테리시스 → 최소 유지 시간 → 전이 규칙 → 근거 저장.
- 정차 예시(**사양 아님**): 0.5 km/h 이하 500ms 유지 → 정차, 1.0 km/h 이상 200ms 유지 → 주행.
- 누락 신호가 있으면 가능한 상태만 추론하고 UNKNOWN을 유지. 사용자 설명으로 확정 상태를 덮어쓰지 않음.

### Stage 3 — Event / Feature Extraction

- Event: 제동 요청, ADAS 활성, ECU 리셋 의심, DTC 상태 변화, 상태 전환, 급감속 등.
- Feature: 응답 지연, 압력 상승률, 차속·휠속도 편차, CAN 메시지 주기 분포 등.
- Event → Feature → Finding 관계로 기록하고 계산의 원본 시간창과 파라미터를 저장.

### Stage 4 — Adaptive Planner

- 우선순위 요소: 이벤트 유형/중요도, 1차 이상 후보, 차량 상태, 기능 관련성, 필수 신호 확보율, 처리 비용.
- **1차 확장** 직접 관련 요청/상태/피드백 → **2차** 차량 거동/교차 검증 → **3차** ECU 진단/전원/통신.
- 신호 관계는 초기에 승인된 **Signal Relationship Graph**로 정의. LLM의 관계 추론은 승인 전 후보로만 사용.
- **중요**: 1차 스크리닝에서 이상이 없어도 사전에 지정된 기능 평가 규칙은 조건이 충족되면 반드시 실행.

### Stage 5 — Parallel Analysis

**Universal Analyzer**:
1. 데이터 유효성·범위
2. 값 고착/급변/변화율
3. 메시지 주기/누락
4. 시간 관계/응답 지연
5. 신호 간 일관성
6. 상태 전이
7. 정상 패턴 대비 통계적 편차(후속)
8. ML 다변량 탐지(충분한 검증 데이터 확보 후)

**Functional Validator**:
- ADAS(AEB, ACC 등), EBS, EPB, 조향, 통신 등 기능별 규칙.
- 전제 조건, 트리거, 기대 응답, 허용 시간, 예외 조건, 필수 신호, 사양 출처를 명시.
- CAN 데이터만으로 실제 물리 성능을 확정할 수 없는 경우 별도 측정 필요 표시.

### Stage 6 — Evidence / Review

- 중복 이벤트 통합, 상호 관련 Finding 연결, 파형·시각·원본·사양·계산 근거 구성.
- 엔지니어의 승인/반려/추가 분석 요청 기록.
- 보고서는 재생성 가능하고, 원본 및 버전 참조를 유지.

## 7. 신호 자동 분류 / 의미 모델

**A. 정적 DBC 추출**: 메시지, 송신 ECU, 채널, CAN ID, 신호명, 단위, scaling, enum, 설명, 주기(있는 경우).

**B. 의미 후보 추론**: 이름·단위·설명·ECU 기반 기능 그룹(제동/ADAS/조향/동역학/진단)과 역할(command/state/feedback/measurement) 분류. LLM은 제안자.

**C. 승인된 차량 프로파일**: 공통 의미 ID → 실제 신호를 명시적 매핑. Verified / Review / Unmapped 상태. 매핑 버전, 근거, 승인자 보관.

예시(실제 차량 사양이 아님):

```yaml
profile_id: truck_platform_a
version: 1
signals:
  brake.request:
    source:
      bus: CAN1
      message: EBS_Control
      signal: EBS_BrkReq
    role: command
    unit: percent
    mapping_status: verified
  brake.pressure.actual:
    source:
      bus: CAN1
      message: EBS_Status
      signal: BrkPressAct
    role: feedback
    unit: bar
    mapping_status: verified
```

매핑되지 않은 신호에도 주기·값 변화 등 의미 비의존 검사는 가능하지만 기능적 PASS/FAIL은 금지한다.

## 8. 규칙 엔진 설계

규칙 3종: **Universal / Functional / Reference-Based**.

- 선언형 YAML/JSON: 적용 조건, 필수/선택 신호, 트리거, 측정 방법, 판정 기준, 누락 데이터 정책, 사양 출처, 버전.
- 복잡한 알고리즘은 Python 플러그인으로 구현; YAML은 실행 조건과 파라미터 지정.
- 임의 Python/eval 문자열을 규칙에서 실행하지 않음. 타입 검증된 제한적 조건 DSL 사용.
- 신호·단위 호환성, 시간 단위, 비교 연산, enum을 스키마에서 검증.
- 규칙 버전과 사양 출처 없이는 확정 FAIL 기준으로 사용하지 않음.

```yaml
rule_id: BRAKE_RESPONSE_DELAY
version: 1.0
required_signals:
  - brake.request
  - brake.pressure.actual
trigger:
  type: rising_edge
  signal: brake.request
conditions:
  - signal: vehicle.motion
    op: eq
    value: MOVING
measurement:
  algorithm: response_delay
  response_signal: brake.pressure.actual
  threshold:
    op: gte
    value: 5.0
    unit: bar
  window_ms: 2000
validation:
  max_delay_ms: 500
on_missing_data: UNKNOWN
```

위의 **5 bar/500ms는 설명용 임의 수치**이며 실제 사양 확인 전 사용 금지.

### 결과 상태

- PASS: 해당 조건에서 사양 만족
- FAIL: 사양 위반이 유효 데이터로 확인
- SUSPECT: 통계적/관계적 이상 후보, 사양 위반 확정 아님
- UNKNOWN: 데이터·동기화·매핑·근거 부족
- NOT_APPLICABLE: 조건 미충족
- NOT_EXECUTED: 실행되지 않음(시간/설정/오류 등 이유 포함)

Severity(잠재 영향), Evidence Strength(근거 강도), Reproducibility(반복성), Coverage(신호/조건 충족률)를 **별도 축**으로 저장. 심각하지만 근거 부족한 사건은 심각도 높음 + UNKNOWN/SUSPECT 가능.

## 9. 오탐 방지 원칙

1. **Data Validity Gate**: 신호 invalid, 기록 공백, 디코딩 문제를 먼저 제외/보류.
2. **Operating Context Gate**: 정차, 선회, ABS 작동, ECU Sleep, 기능 억제 조건 고려.
3. **Temporal Gate**: 최소 지속 시간, 히스테리시스, 디바운스, 경계 조건 처리.
4. **Cross-Signal Gate**: 독립적 관련 신호로 교차 확인. 단 모든 규칙에 필수는 아님.
5. **Specification Gate**: 승인된 기능 사양에 근거한 PASS/FAIL.

정상적인 선회 중 좌우 휠 속도 차이, ABS 중 압력 변동, 정차 중 차속 고정, Sleep 중 통신 중단, 기능 억제 조건에서 ADAS 비활성 등을 대표 오탐 방지 테스트로 포함한다.

## 10. 대용량 데이터 처리 및 성능 전략

- 원본: 읽기 전용 MF4/BLF/ASC + DBC 보존.
- 메타데이터/세션/규칙/결과: SQLite로 MVP 시작.
- 디코딩 신호 캐시: 컬럼 기반 Parquet; 분석 쿼리는 Polars 또는 DuckDB 검토.
- MF4: asammdf; BLF/ASC: python-can; DBC: cantools/canmatrix 후보. 실제 지원 범위와 버전은 프로토타입으로 검증.
- **청크/배치 처리 + 상태 유지**; 전체 데이터를 메모리에 로딩하지 않음.
- **원본 비동기 타임스탬프 유지**; 임의 공통 주기로 리샘플링한 값은 분석 원본으로 사용하지 않음. 보간·as-of join은 허용 오차와 정책 명시.
- 파형 UI용 min/max envelope 또는 이벤트 보존 다운샘플링. 실제 판정은 원본 해상도에서 수행.
- 캐시 키: 원본 파일 해시 + DBC 해시 + 채널/매핑 프로파일 버전 + 알고리즘/규칙 버전 + 실행 설정.
- 캐시를 레이어별로 무효화: 규칙 변경 시 원본 디코딩을 재실행하지 않도록 설계.
- 긴 작업은 백그라운드 Job Manager, 진행률·취소·재시도·부분 결과 제공.
- 성능 목표 수치는 PC 사양과 1GB/10GB 등 대표 로그를 확보한 후 실측 확정.

## 11. 챗봇과 자동 분석 연결 계약

자동 분석 이벤트 선택 시 전달하는 것은 로그 전체가 아니라 **구조화된 참조**다.

```json
{
  "session_id": "test-001",
  "event_id": "BRK-001",
  "time_window": {"start_s": 860.0, "end_s": 865.0},
  "related_signal_ids": [
    "brake.request",
    "brake.pressure.actual",
    "vehicle.speed"
  ],
  "finding_ids": ["F-001"],
  "user_question": "이 이벤트의 원인 후보와 추가 확인 신호를 분석해줘"
}
```

LLM은 허가된 도구(`resolve_signals`, `get_signal_window`, `calculate_response_delay`, `get_vehicle_context`, `get_findings`, `run_rule` 등)를 호출한다. 분석 도구는 원본 데이터·버전·파라미터를 포함한 증거를 반환한다. LLM의 설명은 관측 사실, 사양 위반, 가설, 추가 확인 요청을 구분한다.

## 12. 테스트 및 품질 게이트

- **Golden Logs**: 정상 주행, 정차 기능 평가, 고장 재현 로그를 최소 1개씩 확보.
- **Synthetic Fault Injection**: 응답 지연, 누락, 타임스탬프 역전/드리프트, 값 고착, invalid, DBC 불일치, 청크 경계 이벤트 주입.
- **Expected Events/Findings**: 엔지니어가 시간창·기대 판정·근거를 라벨링.
- **측정 지표**: 이벤트 탐지 Precision/Recall, 오탐 건수/시간, 시간 측정 오차, UNKNOWN 적절성, 처리 시간, 최대 메모리, 재현성.
- **비교 기준**: CANoe/CANalyzer/수작업 기준 분석 결과와 대조.
- **회귀 테스트**: DBC/매핑/규칙/엔진 변경 시 Golden Logs 전체 재실행.
- **안전한 판정**: 데이터 불충분 시 PASS 또는 FAIL을 억지로 출력하지 않음.

## 13. 구현 로드맵

### Phase 0 — 기존 프로젝트 실사 (최우선)
- Git 저장소/브랜치, UI, 로그 파서, DBC 처리, LLM 호출 경로, Python 실행 여부, 캐시/데이터 모델, 테스트 확인.
- 기존 챗봇 동작을 Golden Test로 고정.
- 변경하지 않고 재사용할 부분과 분리할 공통 서비스 식별.
- **산출물**: 현행 아키텍처 맵, 리팩터링 계획, 위험 목록.

### Phase 1 — Data Foundation + Context
- LogSession/SignalDefinition/SignalSeries 데이터 계약.
- MF4/BLF/ASC + DBC 로딩, 인덱싱, 품질 리포트, 캐시.
- 기본 차량 상태(전원/주행·정차/가감속/제동) 추출.
- **산출물**: 차량 상태 타임라인 + 데이터 건전성 카드.

### Phase 2 — Event + Universal Screening
- 범용 주기/결측/고착/급변/상태 전환 이벤트.
- Event/Feature/Finding/Evidence 계약 및 결과 저장.
- 적응형 Planner v1(승인된 신호 관계 그래프 기반).
- **산출물**: 자동 분석 대시보드, 이상 의심 목록, 파형 상세.

### Phase 3 — 챗봇 연동
- 이벤트 → 챗봇 컨텍스트 패키지.
- 기존 챗봇이 공통 신호/분석 API를 사용하도록 점진적 전환.
- **산출물**: 양방향 탐색 및 근거 연결.

### Phase 4 — 기능 평가 / 규칙 자산화
- YAML/JSON Rule Schema + 실행기 + 검증/승인 워크플로.
- ADAS/EBS/EPB 중 하나를 시범 기능으로 선택.
- **산출물**: 사양 기반 PASS/FAIL/UNKNOWN + 리포트.

### Phase 5 — 고도화
- 정상/불량 조건 정합 비교, 유사 사례 검색, 통계/ML 탐지, 반복 고장 패턴 분석.

## 14. 기술 후보 및 디렉터리 구조 (미확정)

```text
vehicle-log-analysis/
  adapters/           # mdf, blf, asc, dbc
  domain/             # session, signal, context, event, finding, evidence
  engine/
    normalization/
    data_quality/
    context/
    event_detection/
    analysis_planner/
    rule_engine/
  plugins/
    universal/
    adas/
    braking/
  profiles/
    vehicles/
    rules/
    relationships/
  storage/            # raw references, metadata, parquet cache
  ai/                 # LLM orchestration, tool calls
  api/
  ui/
  tests/
    golden/
    synthetic/
    integration/
  docs/
```

기존 프로젝트가 이미 있으므로 **이 디렉터리를 무조건 새로 만들지 말고** 현행 구조에 맞게 조정한다.

## 15. 미결정 사항 / 다음 대화에서 확인할 질문

1. **기존 챗봇 프로젝트 저장소 경로와 브랜치**는 무엇인가? (가장 우선)
2. 기존 챗봇의 로그 분석 방식은 LLM 생성 코드 실행 / 고정 함수 호출 / 텍스트 전달 / 혼합 중 무엇인가? **코드 검사로 확인**.
3. 대표 로그 3종(주행, 정차 평가, 고장 재현)과 DBC 샘플 제공 가능 여부.
4. 실제 로그 크기(GB), 메시지/신호 수, CPU/RAM, 로컬/사내망 배포 제약.
5. 차량별 DBC와 기능 사양이 어느 정도 표준화되어 있는가?
6. 초기 기능별 정밀 평가 대상: AEB, EBS, EPB 등 우선순위.
7. 평가 결과를 사내 공유/외부 LLM 전송할 수 있는지 보안 정책 확인.
8. 상태 모델·Signal Relationship Graph의 최소 승인 범위 및 책임자.
9. 대시보드 UI 기술 스택을 기존 앱과 통일할지 여부.

## 16. 다음 개발 세션에 전달할 지시문

> 이 문서는 현재까지 합의한 차량 CAN 로그 통합 분석 플랫폼의 설계 핸드오프다. **먼저 기존 챗봇 저장소를 읽기 전용으로 실사**하고, 현재 아키텍처·로그 파서·DBC 처리·LLM 분석 방식·테스트를 파악하라. 기존 동작을 보존하면서 자동 분석 탭과 공통 분석 코어를 추가하는 최소 변경안을 제안하라. 설계상 결정된 항목과 미확정 항목을 구분하라. Phase 0 결과로 (1) 현행 구조, (2) 재사용 모듈, (3) 제안 데이터 계약, (4) API 경계, (5) 첫 번째 MVP의 수용 기준, (6) 구현 순서 및 리스크를 보고하라. 사용자의 승인 전 대규모 리팩터링이나 사양 임계값 임의 확정은 하지 말라.

---

### 문서 유지 규칙

- 설계가 바뀌면 날짜, 변경 이유, 확정/미확정 상태를 함께 기록.
- 실제 코드에서 확인한 사실과 아이디어/가설을 구분.
- 임의 예시 신호명, 임계값, 시간 수치를 실제 사양으로 오해하지 않도록 명시.
- 다음 논의는 **Phase 0 기존 챗봇 실사 → 공통 데이터 계약 → MVP 수용 기준** 순서로 진행.
