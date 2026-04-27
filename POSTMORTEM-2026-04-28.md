# Postmortem · 2026-04-28 — Mission Stamp Grid 마비 (Severity: Critical)

> **사용자 영향**: 도장 그리드 0/0 표시, 미션 시스템 전체 마비 (메인 9개 + 히든 6개 모두).
> **원인 한 줄**: `showPage()` 안에 TDZ 위험 함수 호출을 추가해 초기 IIFE 가 throw → 스크립트 전체 실행 중단.
> **재발 방지**: 자동 스모크 테스트 + init-path 함수 호출 룰 + defensive guard 패턴 명문화.

---

## 1. 무엇이 깨졌나 (Impact)

| 영역 | 정상 | 회귀 후 |
|---|---|---|
| 도장 그리드 (마무리 페이지) | 9개 카드 + 동적 카운트 | **빈 그리드, "0 / 0"** |
| 메인 미션 카운터 위젯 | "0/9 MISSION" | "0/5 MISSION" (HTML 초기값) |
| Hidden 탭 가시성 (state machine) | d1-hotel done 시 자동 등장 | 항상 default DOM 상태 |
| 미션 도장 상태 복원 (page reload) | state.missions → completed 클래스 자동 적용 | 적용 X (wireMissionCards 도 반파) |
| Day 동선 chip / Leaflet 지도 | 정상 렌더 | TDZ 이전엔 OK, 이후 함수 일부 미실행 |
| 일본 지도·인증 카드·다운로드 | 부분 작동 | 일부 기능 안 함 (DAY_HOTEL_COORDS TDZ) |

→ **사용자 한 줄: "조카가 아예 기능을 쓰지 못하게 됐다."**

## 2. 시간선 (Timeline)

| Commit | 변경 | 결과 |
|---|---|---|
| `8ec1422` | `showPage()` 끝에 `updateMissionCounter()` 호출 추가 (페이지 전환 시 카운터 context 갱신 의도) | 회귀 발생 — 사용자 기능 마비 |
| `f75f808` | 후속 기능 추가 (Legend 카드 + 별점 표시) | 회귀 미인지, 추가 기능만 빌드 |
| `38b55cb` | try/catch 감싸기 fix | **복구** |

회귀가 8ec1422 부터 38b55cb 까지 **3 커밋 동안 살아있었다**. 사용자 직접 보고 후에야 발견.

## 3. 근본 원인 (Root Cause)

### 3.1 TDZ (Temporal Dead Zone) 메커니즘

JavaScript 의 `const`/`let` 은 **선언된 라인이 실행되기 전엔 참조 불가** (TDZ).
함수 선언(`function`)은 hoisting 되지만, 함수 BODY 안에서 참조하는 `const` 는 호출 시점에 초기화돼 있어야 함.

```js
const MISSION_ID_RE = /…/;          // line 12071 — 여기서 초기화

function getAllMissions() {
  …
  if (!MISSION_ID_RE.test(id)) …   // 호출 시점에 line 12071 통과해야 함
  …
}
```

### 3.2 사고 경로

1. `showPage()` 정의 안에 `updateMissionCounter()` 호출 추가 (line ~9962)
2. `updateMissionCounter()` 는 `getMainMissions()` → `getAllMissions()` 호출
3. `getAllMissions()` 안에서 `MISSION_ID_RE` 참조
4. **초기 IIFE** (line 10304) 가 `showPage('cover', { skipScroll: true })` 호출
5. 이 시점에 `MISSION_ID_RE` (line 12071) 는 **아직 초기화되지 않음**
6. `ReferenceError: Cannot access 'MISSION_ID_RE' before initialization`
7. **uncaught** → 스크립트 실행 중단
8. `DAY_HOTEL` (10844), `MISSION_ID_RE` (12071), `buildStampGrid()` 호출 (12325) 등 **이후 모든 코드 미실행**
9. 결과: 미션 시스템 전체 마비

### 3.3 왜 함수가 hoisting 됐는데 const 는 안 됐나

- `function showPage(){}`, `function getAllMissions(){}`, `function updateMissionCounter(){}` 등은 모두 **함수 선언** → 스크립트 시작 시점에 hoisting (window 또는 script scope)
- 그래서 IIFE 에서 `showPage(initId)` 호출 가능
- 하지만 함수 BODY 가 실행될 때 안의 `MISSION_ID_RE` 참조 → 그 const 의 **초기화 시점**은 line 12071
- IIFE 는 line 10304 → **12071 이전** → TDZ

## 4. 탐지 실패 분석 (Why we missed it)

### 4.1 자동 감지 부재
- JS syntax check (`new Function(js)`) 통과 — TDZ 는 런타임 에러라 syntax 단계에서 못 잡음
- 자동 스모크 테스트 부재 — "DAY_HOTEL 이 정의됐나?" 같은 기본 점검이 없었음
- 사용자 피드백 받기 전엔 회귀 인지 X

### 4.2 위계 점검 룰 부재
- DESIGN.md 에 "init-path 에서 호출되는 함수 추가 시 TDZ 점검" 룰 없음
- DOC-MAP.md 영향 매트릭스에 "showPage 같은 핵심 함수 변경 시 검토 항목" 부재
- VERIFICATION.md 자동 검증에 "기본 const 초기화 확인" 항목 부재

### 4.3 변경 점검 부재
- commit 8ec1422 "미션 카드 ↔ 동선·chip 즉시 동기" — 의도는 좋았으나 showPage 변경 영향 미점검
- preview 에서 빠르게 시각 검증만 하고 console 에러 미확인
- 자동 회귀 테스트로 "stampItemsCount === 9" 검증 안 함

## 5. 즉시 수정 (Already done, commit 38b55cb)

```js
// showPage 안
try { if (typeof updateMissionCounter === 'function') updateMissionCounter(); } catch(_) {}
```

→ 초기 IIFE 시점엔 silently skip, 이후 정상 페이지 전환 시엔 작동.

## 6. 재발 방지 룰 (Prevention)

### 6.1 [코드 룰] init-path 함수 호출 금지 패턴

`showPage()`, `loadState()`, 또는 IIFE 에서 직접/간접 호출되는 함수에는 **다음 함수 호출을 추가하면 안 됨**:
- `getAllMissions()` / `getMainMissions()` / `getHiddenMissions()` (MISSION_ID_RE 참조)
- `renderDayRoute()` / `renderDayMap()` (DAY_HOTEL, DAY_HOTEL_COORDS 참조)
- 기타 후방 정의된 const/let 을 참조하는 함수

**예외 패턴**: try/catch 로 감싸거나 setTimeout 으로 비동기화

```js
// ❌ 위험
function showPage(id) {
  ...
  updateMissionCounter();  // 초기 IIFE 시 TDZ 가능
}

// ✅ 안전 1 — try/catch defensive guard
function showPage(id) {
  ...
  try { updateMissionCounter(); } catch(_) {}
}

// ✅ 안전 2 — 비동기화
function showPage(id) {
  ...
  setTimeout(() => updateMissionCounter(), 0);
}
```

### 6.2 [자동 검증] 스모크 테스트 (VERIFICATION.md [Z] 신설)

매 commit 후 preview 에서 자동 실행:

```js
(() => {
  const checks = {
    DAY_HOTEL: typeof DAY_HOTEL,
    MISSION_ID_RE: (() => { try { return typeof MISSION_ID_RE; } catch(e) { return 'TDZ'; } })(),
    stampItems: document.querySelectorAll('.stamp-item').length,
    mainMissions: getMainMissions?.().length,
    hiddenMissions: getHiddenMissions?.().length,
    progress: document.getElementById('stamp-collection-progress')?.textContent,
  };
  const expected = {
    DAY_HOTEL: 'object',
    MISSION_ID_RE: 'object',
    stampItems: 9,
    mainMissions: 9,
    hiddenMissions: 6,
  };
  // 모두 expected 와 일치해야 PASS
  return checks;
})();
```

이 체크 통과 못 하면 → **commit 자체 보류 + fix 우선**.

### 6.3 [문서 룰] DOC-MAP 영향 매트릭스에 추가

| 만지는 것 | 같이 봐야 할 것 |
|---|---|
| `showPage()` / 초기 IIFE / `loadState()` 변경 | TDZ 위험 함수 호출 점검 → DESIGN.md § Defensive Init-Path Pattern → VERIFICATION.md § [Z] 스모크 |

### 6.4 [프로세스 룰] commit 전 필수 점검

핵심 기능에 영향 줄 수 있는 변경 (특히 init 경로 / 페이지 전환 / 미션 시스템) 시:

1. JS syntax check (`new Function(js)`) — 통과
2. **Preview 에서 스모크 테스트 실행** — DAY_HOTEL 정의 여부 + stampItems === 9 확인
3. console error 없는지 확인 (`preview_console_logs level: error`)
4. 변경 영향 받는 *.md 동기

이 4단계 통과 못 하면 push 금지.

## 7. 사용자 사과

이번 변경은 **의도와 무관하게 핵심 기능을 마비시킴**. 사용자(특히 11살 조카) 가 매거진 못 쓰게 만들었던 시간이 있었음.
명백한 책임:
- 변경 영향 사전 검증 부재
- 회귀 자동 감지 시스템 부재
- 문서화 부족 (TDZ + init-path 룰 부재)

위 룰들을 즉시 docs 에 반영 + commit 전 자동 스모크 테스트 절차 추가로 재발 방지.

## 참조

- 위로 → `DOC-MAP.md` (영향 매트릭스 갱신됨)
- 룰 → `CLAUDE.md` § 절대 호출 금지 룰 (init-path)
- 패턴 → `DESIGN.md` § Defensive Init-Path Pattern
- 검증 → `VERIFICATION.md` [Z] 스모크 테스트
- 회고 → `learning.md` § 9 — TDZ 회귀 사례 (이번 사건)
