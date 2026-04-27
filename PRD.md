# PRD · Product Requirements — Okinawa Journal

> **Level 0 / `CONCEPT.md` 의 비전을 기능·플로우·데이터 모델로 구체화**
> 여기서 정한 것을 `CLAUDE.md` → `DESIGN/ROUTE/MISSIONS.md` 가 구현 디테일로 풀어냄.
> 출발 (2026-05-10) 후엔 PRD 동결.

## 1. Functional Requirements

### F-01. 페이지 구조 (7개)

| ID | 페이지 | 목적 | MUST/NICE |
|---|---|---|---|
| `cover` | 표지 + 목차 | 첫인상, 챕터 6개 (prep/d1~d4/back) 진입 | MUST |
| `prep` | 출발 전 | Packing list, Things I Want, prep-ready 미션 | MUST |
| `d1` | Arrival | 공항 도착 → 북부 이동 → Orion 체크인 → 일몰/디너 | MUST |
| `d2` | The Whale Shark | 추라우미 메인 + 코우리 대교 미션 + 옵션 | MUST |
| `d3` | Crossing South | 모토부 마지막 아침 → 차탄 이동 → Vessel 체크인 → 차탄 픽 | MUST |
| `d4` | Going Home | 공항 복귀 + 마지막 미션 | MUST |
| `back` | 마무리 | 도장 그리드 + 인증 카드 + 방문한 곳 + 메시지 | MUST |

### F-02. 미션 시스템 (9개)

총 9개 미션을 **선형 번호 1~9** 으로 식별 (`prep` → `d1` → `d2` → `d3` → `d4` 순서).

**MUST**:
- 각 미션 카드에 "Mission N · {라벨}" 표기 (자동 번호)
- 도장 클릭 → `state.missions[id] = { done: true, at }` 저장
- 카드 시각 변화 (앰버 배경 → 골드 도장)
- 마무리 페이지 stamp grid 에 좌상단 번호 badge (1~9)
- 9개 모두 done → "히든 엔딩" trip-conquer-card 등장

**NICE**:
- 부분 완료 시 카드별 풀스크린 mission-complete 1.8s 애니메이션
- 마지막 미션 시 fireTripComplete (3.2s, 60 confetti, sound chime)

상세 → `MISSIONS.md`

### F-03. 동선 시스템 (구글맵 연동)

**MUST**:
- Day 별 chip 가로 흐름 (`Start → 픽들 → End`)
- 이동일 (D1, D3) 호텔 체크인 chip 자동 삽입 — DOM 위치 기준 동적 계산 (`computeMidInsertIdx`)
- Leaflet 미니지도 + 직선 fallback + OSRM 도로 폴리라인
- chip 클릭 시 지도 flyTo + 부드러운 스크롤
- `.opt-place` / `.mood-card-caption` 클릭 → 구글맵 검색 모달 → 새 탭

**MUST NOT (의도적)**:
- 구글맵 API 키 사용 X (모든 호출은 검색 URL 딥링크)
- 가족 위치 공유 X

상세 → `ROUTE.md`

### F-04. 사용자 흔적 (State)

`localStorage[okinawa-journal-v1]` 단일 키.

| 영역 | 사용자 액션 |
|---|---|
| `ratings` | 옵션 카드 별점 (0~5) |
| `texts` | 메모·이름 입력 |
| `packing` | 짐 체크박스 |
| `photos` | 사진 업로드 (base64, 5MB/장 상한) |
| `chosen` | 사전 옵션 선택 (idx 배열) |
| `chosenWrites`, `chosenExtras`, `extras` | 사용자 추가 옵션 |
| `missions` | 미션 도장 |

**MUST**: 새로고침/재방문 후 모든 상태 복원, 누락 X.

상세 → `DESIGN.md` § State Model

### F-05. Export / Import

**MUST**:
- Toolbar 버튼 → JSON 파일 다운로드 (`okinawa-journal-{name}-{date}.json`)
- Import → 파일 선택 → sanitize → localStorage 덮어쓰기 → reload
- 이전 export 파일 import 시 모든 상태 복원 (검증된 roundtrip)

**보안 (절대 깨면 안 됨)**:
- 화이트리스트 sanitizer (`sanitizeImportedState`)
- `__proto__` / `constructor` / `prototype` 차단
- 사진 dataUrl 정규식 검증
- DOM 주입은 `createElement` + `.src=` (innerHTML 금지)

상세 → `DESIGN.md` § Export/Import + § 보안

### F-06. Auth Gate

**MUST**:
- 비번 `0430` (SHA-256 해시 hardcoded)
- 성공 시 1년 valid 토큰 (`localStorage.okinawa-auth-v1`)
- 토큰 있으면 즉시 매거진 진입

### F-07. 가족 흔적 다운로드 (마무리 페이지)

**MUST**:
- 인증 카드 → 1080×1240 Canvas PNG 다운로드 (일본 지도 + 통계 + 날짜)
- 방문한 곳 리스트 → 동적 높이 PNG (Day 별 그룹 + 시간 순)

### F-08. 모바일 우선 디자인

**MUST**:
- iPhone Safari + Android Chrome 정상 작동
- 가로 스크롤 절대 X
- 폰트 `clamp()` 기반 자동 조정
- 터치 hit area 충분 (44pt+)

## 2. Non-Functional Requirements

| 항목 | 기준 |
|---|---|
| 첫 페이지 로드 | 3G 환경에서 5초 이내 (단일 HTML + WebFont) |
| 호스팅 | GitHub Pages 무료 — `main` push → 1~2분 자동 배포 |
| 외부 의존 | Google Fonts, Pretendard CDN (jsDelivr), Unsplash 이미지, OSRM demo, OpenStreetMap tiles |
| 외부 API 키 | **0개** — 모두 무료/공개 |
| 백업 | git + 사용자 export JSON |
| 접근성 | 스크린 리더 aria-label, 키보드 탐색 지원 (NICE) |

## 3. User Flows

### Flow A — 출발 전 (조카)
```
URL 받음 → 비번 0430 입력 → 표지
→ Mission 1 (Pre-Trip) 도장 클릭 → 골드 도장 ✓
→ 다음 페이지 → 호기심 ↑
```

### Flow B — 여행 중 (운전자)
```
다음 일정 모름 → D2 페이지 → Today's Route chip 흐름 보기
→ 다음 chip 클릭 → 미니지도 flyTo → "구글맵에서 열기" → 외부 앱
→ 호텔 도착 → hotel-card 의 매피온 코드 → 네비 입력
```

### Flow C — 여행 중 (조카)
```
오늘 미션 뭐 있지? → D3 페이지 → Mission 7 (Vessel 도착) 카드 보임
→ 호텔 도착 시 도장 클릭 → 풀스크린 1.8s 애니메이션
→ 다음 미션 (Mission 8 차탄 선셋) 자연스럽게 안내
```

### Flow D — 9번째 미션 클릭 (조카)
```
모든 8개 + 마지막 1개 done
→ fireTripComplete (3.2s + confetti + 사운드)
→ 마무리 페이지 자동 이동
→ trip-conquer-card 화려한 등장 (japan-map.svg + 오키나와 골드 ★)
→ 통계 4칸 (도장 9/9, 평균 만족도, 사진, 방문 곳)
→ "히든 엔딩" 메시지 펼쳐짐
→ 인증 카드 PNG 다운로드 → 가족 단톡 공유
```

### Flow E — 백업 (메인테이너)
```
월 1회 → btn-export → JSON 파일 받음 → 안전한 곳 보관
새 기기 / 데이터 분실 → btn-import → 파일 선택 → 덮어쓰기 확인 → reload
→ 모든 상태 복구
```

## 4. Out of Scope (다시 확인)

- 실시간 가족 간 동기화 / 멀티플레이어
- 결제·호텔 예약·항공권 연동
- 다국어 i18n (한국어 + 일본어 표기 혼용)
- 계정 시스템 (비번 1개)
- 푸시 알림 / 푸시 서버
- 백엔드 — 모든 처리 클라이언트 사이드

## 5. Risks & Mitigation

| 리스크 | 영향 | 대응 |
|---|---|---|
| 출발 후 매거진 버그 발견 | 가족이 잘못된 정보로 이동 | 출발 전 `VERIFICATION.md` 100% 통과 + 두 명 모바일 테스트 |
| GitHub Pages 다운타임 | 매거진 안 열림 | 가족별로 최소 1회 사전 접속 → 브라우저 캐시 + export JSON 백업 |
| localStorage 가득참 | 사진 업로드 실패 | 사진 5MB/장 상한 + 안내 멘트 + export 후 정리 권장 |
| 외부 의존 (Unsplash, jsDelivr) 다운 | 이미지·폰트 깨짐 | 핵심 정보(텍스트·동선·미션)는 이들 없이도 작동 |
| 공식 사이트 영업시간 변경 | 휴무일 방문 헛걸음 | 출발 직전 (D-7) 재검증 — `CLAUDE.md` § 장소 검증 |

## 6. 참조

- 위로 → `CONCEPT.md` (왜 이걸 만드나)
- 아래로 → `CLAUDE.md` (Claude 가 작업할 때 따르는 운영 가이드)
- 도메인별 → `DESIGN.md`, `ROUTE.md`, `MISSIONS.md`
- 검증 → `VERIFICATION.md`
- 백로그 → `TODO.md`
- 회고 → `learning.md`
