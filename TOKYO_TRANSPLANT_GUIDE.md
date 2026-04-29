# 도쿄 매거진 Transplant 가이드 (Volume II)

> **목적**: 오키나와 매거진의 자산 (코드 + 5 layer 문서화) 을 도쿄 개인용 매거진에 재활용하기 위한 명세서.
> **핵심 차이**: 오키나와는 가족 4명 공유 + 11살 조카 메인 (게임화 핵심) / 도쿄는 본인 단독 사용 (기능 효율 우선).
> **작성일**: 2026-04-29
> **대상 독자**: 도쿄 매거진 시작 시 첫 참조 — 사람 + Claude 둘 다.

---

## 1. 한 줄 요약

**도쿄 매거진** = 오키나와 구조 그대로 + 게임 요소 (미션·도장·Hidden·풀스크린 알림) **전부 제거** + 본인 단독 사용에 맞게 정보 밀도·메모·검색 우선.

---

## 2. 페르소나 차이 — 모든 결정의 출발점

| 항목 | 오키나와 | 도쿄 |
|---|---|---|
| 메인 사용자 | 11살 조카 (남) | 본인 (30대) |
| 보조 사용자 | 자형/누나/부모 | 없음 |
| 인원 | 4명 | 1명 |
| 사용 시나리오 핵심 | 게임화 (도파민) → 가족 함께 → 기록 | 출발 전 계획 → 여행 중 빠른 참조 + 메모 → 여행 후 회고 |
| 톤 | 친구한테 자랑할 만한, 카톡 알림 같은 | 어른스러운, 효율 중심, 차분 |
| 콘텐츠 무게 | 시각적, 카드형, 풀스크린 알림 | 정보 밀도, 검색 쉬움, 메모 공간 |
| "현지 수정 불가" 제약 | 강함 (가족 4명 영향) | 약함 (본인만, 폰에서 수정 가능) — 그래도 미리 정리하면 편함 |

→ 이 표가 **모든 transplant 결정의 기준**. 의심되면 항상 페르소나로 돌아가서 확인.

---

## 3. Tier 1 — 그대로 재사용 (게임 요소 무관)

### 3.1 5 Layer 위계 시스템 (`DOC-MAP.md` 기반)

```
L0 VISION         CONCEPT.md (도쿄 페르소나로 갱신)
                  PRD.md (도쿄 기능 명세 — 게임 요소 제외)
L1 PROJECT GUIDE  CLAUDE.md (오키나와 룰 그대로 + 도쿄 fixed facts 교체)
L2 IMPLEMENTATION DESIGN.md (state/디자인 토큰/Box Gap/보안 그대로)
                  ROUTE.md (3-call-sites consistency 그대로 — 도쿄 좌표만 교체)
                  ※ MISSIONS.md 는 도쿄에서 삭제 또는 비움 (게임 X)
L3 OPERATIONS     VERIFICATION.md ([Z][P][G] 그대로, [Q] 삭제 — Hidden 없음)
                  TODO.md (Level 별 sub-section 패턴 그대로)
L4 KNOWLEDGE      learning.md (오키나와 § 1~7·10·11 그대로, § 9·12 부분 X)
                  + 도쿄 § 새로 추가 (개인용 매거진 차이점)

INDEX:            DOC-MAP.md (5 Layer 다이어그램 + 영향 매트릭스 그대로)
```

**그대로 재사용 가능한 docs 섹션**:
- `CLAUDE.md` § 절대 호출 금지 룰 (init-path TDZ) — 모든 vanilla JS 프로젝트 공통
- `CLAUDE.md` § 장소 검증 프로세스 (출처 강도 🟢🟡🔴, 좌표 land-check) — 그대로
- `DESIGN.md` § State Model (3-namespace + sanitizer)
- `DESIGN.md` § Design Tokens (색상 + Box Gap)
- `DESIGN.md` § 보안 (Import 검증 4겹 방어)
- `DESIGN.md` § Auth Gate
- `DESIGN.md` § Dual-render Components (UI ↔ Canvas Parity Rule)
- `ROUTE.md` § 3-call-sites consistency rule (renderDayRoute / renderDayMap / assignPickNumbers)
- `VERIFICATION.md` § [Z] 자동 스모크, § [P] Canvas Parity, § [G] 동선
- `learning.md` § 1 모바일 CSS 함정, § 2 상태 관리, § 5 장소 검증, § 6 문서 위계, § 7 UI↔Canvas Parity, § 10 별/원 정렬 (별 → 원 only 로 단순화), § 11 자연스러운 텍스트, § 14 SNS 발췌
- `DOC-MAP.md` 전체 구조 — 영향 매트릭스에서 미션/Hidden 행만 삭제

### 3.2 코드 — 그대로 재사용 (수십 컴포넌트)

| 영역 | 컴포넌트 | 비고 |
|---|---|---|
| 페이지 시스템 | `PAGE_IDS`, `showPage()`, 네비, 좌우 화살표, 하단 페이지네이션 | 페이지 ID 만 도쿄에 맞게 |
| 상태 / 영속성 | `loadState`, `saveState` (defensive), `sanitizeImportedState`, 3-namespace | sanitizer 화이트리스트 패턴 그대로 |
| 보안 | CSP meta, `DATA_IMG` 정규식, `SAFE_ID`, `MISSION_ID_RE` (이름만 `ITEM_ID_RE` 등으로) | |
| Auth | `AUTH_HASH_HEX` SHA-256 비교 | 비번 도쿄 별도 (또는 빼도 됨, 본인 단독) |
| Export / Import | 툴바 ↓↑ 버튼, JSON 다운로드 + sanitizer 거쳐 import | 그대로 |
| 디자인 토큰 | CSS `:root` (색상 + Box Gap `--box-gap-tight/md/loose`) | 색만 도쿄 톤으로 (예: 차분한 슬레이트, 골드 액센트) |
| 박스 컴포넌트 margin | `var(--box-gap-*)` 통일 | 그대로 |
| 페이지 안 박스 layout | `.page` flex `gap: 16px` + 박스 margin 추가 spacing | 그대로 |
| 옵션 카드 | `.opt`, `.opt-place`, `.opt-body`, `.opt-rating`, `.opt-write` (placeholder + extras) | 그대로 (옵션 = 장소 후보) |
| 동선 시스템 | `DAY_HOTEL`, `DAY_HOTEL_COORDS`, `computeMidInsertIdx`, `renderDayRoute`, `renderDayMap`, `assignPickNumbers` | 도쿄 호텔 좌표만 교체 |
| Leaflet 지도 | `L.map`, OSRM 폴리라인 | 그대로 |
| 별점 위젯 | `.opt-rating`, `state.ratings` | "How Excited?" → "어땠어?" 또는 "Rate" 으로 톤만 |
| 사진 업로드 | `addPhotoToTarget`, `resizeImage`, `makePhotoItem` (createElement 안전), 5MB 상한 | 그대로 |
| Place 검색 | `PLACE_QUERIES`, `resolveMapQuery`, `SKIP_MAP_PATTERNS` | 도쿄 장소 사전 새로 작성 |
| 일본 지도 SVG | `japan-map.svg` + Leaflet 또는 Canvas | 강조 좌표 도쿄로 (오키나와 X) |
| 모바일 반응형 | `clamp()`, `minmax(0, 1fr)`, `IntersectionObserver` (툴바) | 그대로 |
| Canvas 다운로드 | `downloadConquerImage` 기반 — 도쿄 Recap 카드로 변형 | Tier 2 참조 |
| visited-list | `getVisitedPlaces`, `renderVisitedList`, `downloadVisitedListImage` | ★/● 구분 빼고 ● 만 |
| 동적 옵션 | `.opt-write` placeholder + extras, sanitizer 처리 | 그대로 |

---

## 4. Tier 2 — 변형 필요 (게임 → 기능 톤)

### 4.1 마무리 페이지 — "Trip Conquered!" → "Tokyo Recap"

| Before (오키나와) | After (도쿄) |
|---|---|
| `trip-conquer-card` "오키나와 정복!" | `trip-recap-card` "도쿄 다녀왔다" 또는 단순 "Tokyo Recap" |
| `td-stats`: 9/9 MISSION / 평균★ / 사진 / 방문한 곳 | `tr-stats`: 방문한 곳 / 평균★ / 사진 / 메모 (또는 4 칸 구성 자유) |
| 일본 지도 + 오키나와 ★ pulse | 일본 지도 + **도쿄 ★** pulse (좌표 변경 필요) |
| 다운로드 PNG = 가족 단톡 자랑용 | 다운로드 PNG = 본인 SNS / 블로그 자료 |

**변경 핵심**:
- "정복!" 같은 게임 톤 → "다녀왔다" / "Recap" 같은 차분한 어른 톤
- "9/9 MISSION COMPLETE!" 같은 핵심 metric → 본인 메트릭 (방문 N 곳, 사진 N 장)
- 풀스크린 알림 (`fireTripComplete` 60 confetti + 도-미-솔-도) **삭제**

### 4.2 visited-list (Place & Action) — 미션 분리 제거

| Before | After |
|---|---|
| ★ 미션 / ● 옵션 시각 구분 (CSS flex centering) | ● 만 (모든 항목 동일 아이콘) |
| `isMission` flag | 제거 |
| `.vl-item.mission::before` ★ | 제거 (정규 ●) |
| Canvas downloadVisitedListImage 의 별 그리기 | 제거 (정규 arc) |

→ "방문한 곳" 단일 카테고리 (장소만). 행동 카운트 X.

라벨도 "Place & Action" → **"방문 기록"** 또는 **"Visited"** 로 변경 (Action 빠짐). 카운트 단위 "건" → **"곳"** 으로 환원.

### 4.3 미션 카운터 위젯 → 빼거나 단순 정보 위젯

| Before | After |
|---|---|
| `.mission-counter` "9/9 MISSION" 펄스 + amber 톤 | 삭제 또는 "방문 N 곳" / "오늘 N 일차" 같은 정보 위젯 |
| context-aware (메인/Hidden 토글) | 단일 표시 |

추천: **카운터 자체 제거** — 본인 사용이라 진척도 시각화 불필요. 페이지 안 정보가 자체로 충분.

### 4.4 사진 업로드 — 그대로지만 페이지 매핑 단순화

| Before | After |
|---|---|
| `data-photo-grid="day1"`, `mission-d1-family`, `hidden-shisa` 등 다양한 target | `day1` ~ `dayN` 단순 | (미션 사진 X) |
| `.mission-photo-grid`, `.photo-mission` 별도 클래스 | 삭제 (미션 사진 미션 없음) |

### 4.5 별점 위젯 — 라벨만 변경

| Before | After |
|---|---|
| `<span class="r-lbl">How excited?</span>` | `<span class="r-lbl">Rate</span>` 또는 한국어 `어땠어?` 같은 차분한 톤 |
| amber 별점 5 | 그대로 |

→ 11살 친화 표현 → 어른 톤. 기능 동일.

### 4.6 Canvas Parity 룰은 유지

- `trip-recap-card` ↔ `downloadConquerImage` 패턴 그대로
- visited-list ↔ `downloadVisitedListImage` 그대로
- **DESIGN.md § Dual-render Components** 표는 도쿄 컴포넌트로 행 갱신만

---

## 5. Tier 3 — 빼는 것 (게임 요소 전면 제거)

다음을 **HTML / CSS / JS 모두 삭제**:

### 5.1 미션 시스템 전체
- `.mission-card` 컴포넌트 전체 (15 개 카드)
- `.mission-stamp-btn`, `.mission-tag`, `.mission-title`, `.mission-desc`, `.mission-hint`, `.mission-num-pill`, `.mission-undo-btn`
- `.mission-action-row`, `.mission-photo-grid`, `.weather-backup`
- `data-mission`, `data-mission-type`, `data-mission-label`, `data-mission-num`, `data-photo-mission`, `data-weather-done`
- `state.missions` 필드 (state model 에서 제거)
- 함수: `getAllMissions`, `getMainMissions`, `getHiddenMissions`, `wireMissionCards`, `assignMissionNumbers`, `buildStampGrid`, `refreshStampGrid`, `updateMissionCounter`, `celebrateMission`, `fireMissionComplete`, `fireTripComplete`, `refreshDayRouteForCard`, `renderTimestamp`
- `MISSION_ID_RE` 정규식
- `.mission-counter` 위젯 + `.mc-card`, `.mc-icon`, `.mc-count`, `.mc-label`
- `.stamp-collection`, `.stamp-grid`, `.stamp-item`, `.stamp-num-badge`, `.stamp-icon-large`, `.stamp-day-tag`, `.stamp-name`, `#stamp-collection-progress`

### 5.2 Hidden Mission 시스템 전체
- 페이지 `data-page="hidden"` 자체 삭제 → `PAGE_IDS` 7 → 6 (또는 도쿄 일정에 맞게)
- `.hidden-mission`, `.hidden-mark`, `.hidden-page-head`, `.hidden-page-sub`, `.hidden-counter`, `.hc-num`, `.hc-divider`
- `.nav-hidden-tab`, `.hidden-tab-lock`, `.hidden-tab-label`, `.hidden-tab-new`
- `.hidden-unlock-cta`, `#hum-modal`, `#hidden-ticket-overlay`, `#family-master-overlay`
- `.legend-card`, `.lc-medal`, `.lc-tag`, `.lc-title`, `.lc-stats`, `.lc-stat`, `.lc-download-btn`, `.lc-okinawa-svg`
- `.family-master-msg`, `.fm-eyebrow`, `.fm-headline`, `.fm-body`
- `.huc-icon`, `.huc-tag`, `.huc-title`, `.huc-body`, `.huc-btn`
- `.ht-icon`, `.ht-tag`, `.ht-title`, `.ht-body`
- `.fmo-medal`, `.fmo-tag`, `.fmo-title`, `.fmo-body`
- `state.hiddenUnlocked` 필드
- 함수: `updateHiddenUnlockUI`, `showHiddenUnlockModal`, `hideHiddenUnlockModal`, `unlockHiddenMissions`, `updateHiddenCounter`, `updateLegendStats`, `fireFamilyMaster`, `fireTicketIssued`, `updateUnlockModalText`, `updateTicketOverlayText`, `getActivePageIds`, `downloadLegendImage`, `drawOkinawaSilhouetteToCanvas`, `getTravelerName`

### 5.3 풀스크린 cinematic + 사운드
- 모든 `.fullscreen-overlay` 류 (mission-complete, trip-complete, family-master, ticket-issued)
- Web Audio (`playGentleChime`, `playSineChime`) — 사운드 자체 X. 단순 toast "저장됨" 인디케이터만 유지

### 5.4 도장·티켓·트로피 emoji
- 🎫, 🏆, 🗝, 🎮 등 게임 emoji — 톤이 안 맞음
- 다만 단순 visual emoji (📷 사진, 🍱 음식, 🚆 기차) 는 OK

### 5.5 11살 친화 어휘
- "친구한테 자랑해도 돼"
- "비밀 보너스"
- "지금 풀기 / 나중에 풀게"
- "프로 트래블러" (도쿄에선 어울릴 수도 있지만 게임 톤이라 빼는 게 안전)

---

## 6. Tier 4 — 도쿄 특화 새로 추가 (선택)

### 6.1 일본어 phrase book 미니 (있으면 좋음)

페이지 1 개 또는 사이드 패널로:
- 인사: ありがとう (아리가또), すみません (스미마셍), はじめまして (하지메마시테)
- 식당: これください (코레 쿠다사이), お会計 (오카이케이), おすすめ (오스스메)
- 길: 駅はどこですか (에키와 도코데스카), まっすぐ (맛스구), 右/左 (미기/히다리)
- 결제: クレジットカード (쿠레짓또카-도), 現金 (겐킨)

오키나와 H5 미션 ("일본어 인사") 의 phrase 데이터 그대로 가져옴. 미션 wrapper 만 빼고 단순 reference card 로.

### 6.2 영수증·예산 트래커 (선택)
- 옵션 카드 안에 "예상 비용 ¥" 필드
- state.budget 신설: `{ day1: 12000, day2: 8500, ... }`
- 사용자가 직접 입력 가능
- 마무리 페이지 통계에 "총 지출 ¥X" 추가

→ 본인 사용이라 손쉬운 입력 (input field) 으로 충분.

### 6.3 환율 표시 (선택)
- ¥ → ₩ 변환 — 정적 (출발 전 환율 박아두고 1주일 전 갱신)
- 또는 동적 (외부 API — CSP 추가 필요 + 네트워크 의존)

→ 정적 추천 (안 깨짐, 단순).

### 6.4 빠른 메모 패드 (강추)

도쿄 매거진의 핵심 차별화. 본인 사용이라 여행 중 즉석 메모 강화:
- 각 페이지 상단 또는 우측에 "오늘의 메모" `<textarea>`
- state.notes 필드 추가, sanitizer 에 dispatcher 추가
- 키보드 자동 저장 (debounce)

오키나와의 `state.texts` 시스템 확장으로 구현 가능.

### 6.5 교통 정보 — JR 패스 / 지하철 모드

오키나와는 렌터카 필수, 도쿄는 지하철·JR 핵심:
- 옵션 카드 안에 "최근역" 필드 (예: 시부야역, 신주쿠역)
- 구글맵 검색어를 "역명 + 출구" 식으로 (예: "Shibuya Station Hachiko Exit")
- Suica/Pasmo 잔액 메모 필드 (선택)

`PLACE_QUERIES` 의 검색어 전략은 그대로 — 도쿄 장소 사전 새로 작성.

### 6.6 다국어 키 전략 (오키나와에서 검증된 패턴)

`CLAUDE.md § 다국어 키 전략` 그대로 가져옴:
- 일본어 한자: `'渋谷'`, `'新宿'`
- 한국어 음차: `'시부야'`, `'신주쿠'`
- 영어: `'shibuya'`, `'shinjuku'`

도쿄는 한자 + 가나 + 한국어 음차 + 영어 4 가지 표기 모두 매칭 필수.

---

## 7. 도쿄 매거진 단계별 작업 가이드

### 7.1 폴더 + 위계 셋업 (Day 1)

```bash
# 새 폴더
mkdir -p "/Users/jin/Desktop/Git/일본 여행 매거진/도쿄 여행 매거진"
cd "/Users/jin/Desktop/Git/일본 여행 매거진/도쿄 여행 매거진"

# 오키나와 docs 5 layer 복사 (CONCEPT/PRD/CLAUDE/DESIGN/ROUTE/VERIFICATION/TODO/learning/DOC-MAP)
cp ../오키나와\ 여행\ 매거진/{CONCEPT,PRD,CLAUDE,DESIGN,ROUTE,VERIFICATION,TODO,learning,DOC-MAP}.md .

# index.html 복사
cp ../오키나와\ 여행\ 매거진/index.html .
cp ../오키나와\ 여행\ 매거진/japan-map.svg .

# 게임 전용 docs 는 안 가져옴
# - MISSIONS.md (게임 시스템)
# - POSTMORTEM-2026-04-28.md (오키나와 사고 이력)
# - TEST-REPORT-* (오키나와 테스트 결과)
```

### 7.2 docs 도쿄 페르소나로 갱신 (Day 1)

순서:
1. **CONCEPT.md** — 페르소나 (조카 → 본인), Fixed Facts (도쿄 일정·호텔·항공·교통), Success Criteria 도쿄 기준
2. **PRD.md** — 기능 요구사항에서 게임 (F-02, F-XX) 항목 삭제, 메모/예산 등 새 기능 추가
3. **CLAUDE.md** — 여행 사실 + 절대 금지 룰 그대로 유지, "장소 검증 프로세스" 그대로
4. **DESIGN.md** — state model 에서 `missions`, `hiddenUnlocked` 제거, `notes`, `budget` 추가
5. **ROUTE.md** — 도쿄 호텔 좌표 (`DAY_HOTEL_COORDS`) 교체
6. **VERIFICATION.md** — [Q] Hidden 검증 섹션 삭제, 다른 항목 그대로
7. **TODO.md** — Level 별 sub-section 패턴 그대로, 도쿄 항목으로 채움
8. **learning.md** — § 1·2·5·6·7·10·11·14 그대로, § 3·4·9·12 부분만 (게임 빼고 동선/장소만)
9. **DOC-MAP.md** — 영향 매트릭스에서 미션 관련 행 (4.1) 삭제

→ 위계 + 영향 매트릭스 + 작성 룰은 그대로. 콘텐츠만 도쿄.

### 7.3 코드 transplant (Day 2~3)

순서 (회귀 위험 낮은 것부터):

```
1. CSP meta + 디자인 토큰 + Box Gap (그대로)
2. Auth Gate (비번 도쿄 별도, 또는 제거)
3. PAGE_IDS + 네비 (도쿄 일정에 맞춰 페이지 수 결정)
4. 옵션 (.opt) 카드 시스템 (그대로 — 미션 wrapping 만 제거)
5. PLACE_QUERIES 도쿄 장소 사전 새로 작성 (가장 시간 많이 듦)
6. DAY_HOTEL + 동선 시스템 (좌표만 교체)
7. 사진 업로드 (그대로 — target 만 단순화)
8. visited-list (★ 분리 제거, ● 만)
9. trip-recap-card (다운로드 패턴 + 일본 지도 + 도쿄 ★)
10. Export/Import (sanitizer)
11. 모바일 반응형 점검
12. (선택) 메모 패드 / 일본어 phrase / 예산 트래커
```

각 단계 후 `VERIFICATION.md [Z]` 스모크 테스트 실행.

### 7.4 검증 (Day 4)

오키나와 `TEST-REPORT-2026-04-28-v2.md` 의 자동 검증 패턴 그대로:
- Static (JS syntax + HTML 구조)
- Unit (sanitizer + getter 함수들 + PAGE_IDS)
- State Persistence (3-namespace + roundtrip)
- Integration (페이지 전환 + 옵션 선택)
- E2E (모든 페이지 진입 + 동선 + 다운로드)
- Mobile / Tablet / Desktop 3 viewport
- Visual screenshot 비교

미션 / Hidden 관련 테스트 항목은 자연스럽게 안 만들어도 됨 (시스템 자체 X).

---

## 8. 재활용 함정 + 회귀 방지

### 8.1 절대 빠뜨리면 안 되는 것 (오키나와에서 학습한 회귀)

| 함정 | 룰 |
|---|---|
| **TDZ** (init-path 함수 호출) | `CLAUDE.md § 절대 호출 금지 룰`, init-path 에서 후방 const 참조 함수 호출 금지. try/catch 또는 setTimeout 으로 격리 |
| **3-namespace 동시 점검** | `state.chosen` / `chosenWrites` / `chosenExtras` — 한 곳 건드리면 6 곳 코드 위치 다 봐야 |
| **UI ↔ Canvas Parity** | `DESIGN.md § Dual-render Components` 표 — UI 변경 시 canvas 도 같은 commit |
| **sanitizer 화이트리스트** | 새 state 필드 추가 시 sanitizer 대응 블록 필수 추가. 안 그러면 import 시 데이터 손실 |
| **장소 검증** | 공식 → Tabelog/Tripadvisor → 블로그 강도 순. 좌표는 Google Maps land-check |
| **3-call-sites consistency** (동선) | `renderDayRoute` / `renderDayMap` / `assignPickNumbers` 동시 갱신 |
| **모바일 CSS** | `clamp()` 사용 (고정 px X), `minmax(0, 1fr)` 사용 (`1fr` 단독 X), `grid-area` 깊이 정확히 |
| **--coral / --lime 같은 색 alias** | 디자인 토큰 변경 시 alias 까지 추적 |
| **다국어 키** | `PLACE_QUERIES` 한 장소당 한자 + 가나 + 한국어 + 영어 4 표기 모두 등록 |

### 8.2 새로 도입하지 말 것 (실험적 영역)

- React, Vue, Svelte 같은 프레임워크 — vanilla 단일 HTML 유지 (5 년 후에도 호환)
- 빌드 도구 (Vite, webpack) — 단일 파일 직접 push → GitHub Pages 자동 배포
- TypeScript — 타입 검증보다 런타임 sanitizer 가 충분
- 외부 DB / 백엔드 — localStorage 단일 기기로 충분 (본인 사용)
- 푸시 알림 / Service Worker — 정적 HTML 한정

### 8.3 두 세션 충돌 방지 (오키나와 § 두 세션이 같은 벽을 칠하면)

- 도쿄 폴더 작업 시 **한 폴더 = 한 Claude 세션** 룰 엄수
- 오키나와 세션이 도쿄에 기여하고 싶으면 → 부모 폴더에 `OKINAWA_TO_TOKYO_PORT.md` 명세서로 넘기기 (직접 도쿄 파일 수정 X)

---

## 9. 도쿄 매거진 신규 의사결정 체크리스트

도쿄 시작 전 본인이 정해야 할 것:

### 9.1 Fixed Facts
- [ ] 일정 — 며칠? D1 ~ D?
- [ ] 항공 — 도착 / 귀국 시간
- [ ] 숙소 — 호텔 N 박, 주소, 좌표
- [ ] 교통 — JR Pass 사용? Suica? 지하철 위주?
- [ ] 비번 — Auth Gate 쓸지? 본인만 사용해서 빼도 OK

### 9.2 페이지 구조
- [ ] 페이지 수 — 표지 + 여행 전 + Dx + 마무리. Hidden 페이지 X
- [ ] 페이지 이름 — `cover`, `prep`, `d1`, `d2`, ..., `back`. 또는 도쿄 컨셉 (`shibuya`, `asakusa` 같은 지역별?)

### 9.3 콘텐츠
- [ ] 카테고리 — 음식, 쇼핑, 전시, 야경, 카페, 공원 등 어떤 톤?
- [ ] 옵션 (.opt) 카드 — 각 시간대당 몇 개 후보?
- [ ] 사진 업로드 target — day1~dayN 단순? 또는 카테고리별?

### 9.4 톤
- [ ] 어휘 — "Tokyo Diary" / "방문 기록" / "Travel Log" / "도쿄 노트" 어떤 분위기?
- [ ] 색상 — 오키나와는 블루+민트+앰버. 도쿄는 차분한 슬레이트? 골드? 모노톤?
- [ ] 폰트 — 그대로 유지 (Pretendard + Archivo Black + Fraunces + Space Grotesk)?

### 9.5 추가 기능 (선택)
- [ ] 메모 패드 (`state.notes` 신설) — 강추
- [ ] 예산 트래커 — 본인 단독이라 부담 없음
- [ ] 일본어 phrase 카드 — 오키나와 H5 데이터 재활용
- [ ] 환율 표시 — 정적 박아두기

---

## 10. 한 줄 요약

> **도쿄 매거진 = 오키나와의 5 layer 위계 + 코드 골격 + 회귀 방지 룰 그대로** + **게임 요소 (미션 / Hidden / 도장 / 풀스크린 / 사운드 / 11살 어휘) 전면 제거** + **본인 페르소나 (메모 / 예산 / 일본어 phrase / 어른 톤) 강화**.

**작업량 추정**: docs 갱신 0.5일 + 코드 transplant 1.5일 + 콘텐츠 (PLACE_QUERIES + 옵션 카드) 2~3일 + 검증 0.5일 = **약 5일**.

**이 가이드 자체는 도쿄 시작 시 첫 참조 자료** — 도쿄 폴더의 `learning.md` § 0 또는 `CLAUDE.md` 끝에 link 추가 권장.

---

## 11. 참조

- 출처 → `오키나와 여행 매거진/` 의 모든 docs (특히 `DOC-MAP.md`, `learning.md` § 1~12)
- 회귀 사례 → `POSTMORTEM-2026-04-28.md` (TDZ Critical 사고)
- 검증 패턴 → `TEST-REPORT-2026-04-28-v2.md` (~150 케이스 자동 검증)
- 도쿄 시작 시 → 이 파일 + `learning.md § 6 문서 위계 정리` 정독
