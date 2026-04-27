# TODO — Okinawa Journal

> **Level 3 / 백로그.** 위계별 분류 — 작업 시 어떤 도메인 문서를 봐야 하는지 한눈에.
> 우선순위: P0 (출발 전 필수, 5/10 까지) > P1 (있으면 좋음) > P2 (다음 시즌 검토)

## L0 · CONCEPT / PRD 단계 (비전·요구사항)

### P0 — 출발 전 확정
- [x] 미션 9개 정의 + 페이지별 배치 → `MISSIONS.md`
- [x] 동선 시스템 미들 호텔 메커니즘 → `ROUTE.md`
- [x] 위계 doc 정리 → `DOC-MAP.md`
- [ ] PRD F-01 ~ F-08 모두 구현 + 검증 통과

### P2 — 다음 시즌
- [ ] Volume II (다른 여행지) 템플릿화 — 색상 토큰·장소 데이터만 교체로 가능한지

## L1 · CLAUDE.md (운영 룰)

### P1 — 룰 강화
- [ ] CLAUDE.md 의 "장소 검증 프로세스" 가 `VERIFICATION.md` 의 자동 검증과 동기 되도록 케이스 추가
- [ ] 디자인 토큰 변경 시 alias 추적 자동화 (CSS 변수 의존 그래프) — 검토만

## L2 · 도메인 명세 (DESIGN / ROUTE / MISSIONS)

### P0 — 출발 전

#### Day별 시안성
- [ ] PC에서 각 Day opener (Day One/Two/Three/Four) 가독성 최종 점검 → `DESIGN.md` § 디자인 토큰
- [ ] 모바일에서 같은 계열 색상 겹침 문제 확인 → `DESIGN.md`
  - 파란 배경 + 파란 floaty 원형이 글자 가리는 곳
  - 노란 원(sun circle)이 타이틀 침범
  - mood card 패턴 배경 + 흰 글씨 대비 약한 곳
- [ ] 읽기 어려운 폰트 조합 점검 (Fraunces italic + 한글 겹침)

#### 장소 정보 검증 (CLAUDE.md § 장소 팩트 검증 프로세스)
- [ ] 4일 × 옵션별 장소명/주소/영업시간/거리 실제 정보 크로스체크
  - Vessel Hotel Campana (차탄)
  - Sunset Beach · Blue Seal · Kurasushi · Jack's Steakhouse
  - Churaumi Aquarium · Kourikyo Bridge · American Village · Senagajima
  - Shurijo Castle 등
- [ ] 가격/요금 2026년 기준으로 갱신
- [ ] 공식 사이트/구글맵에서 현재 운영 중 여부 확인 (D-7, D-1)

### P1
- [ ] 표지 OKINAWA 뒤 floaty 원이 텍스트 가림 → 가족 공유 시 첫인상 중요
- [ ] Day opener 제목이 긴 경우 wrap 처리 검증

## L3 · 운영·검증 (VERIFICATION / 자동화)

### P0
- [ ] `VERIFICATION.md` 자동 검증 [G] 항목 — 9개 미션 수 일치 확인
- [ ] D1 Blue Seal 픽 chosen 시 midInsertIdx 동작 ROUTE.md 와 일치하는지 자동 검증 케이스 추가
- [ ] D3 morning + afternoon 픽 모두 chosen 시 midInsertIdx = 모닝 픽 개수 검증
- [ ] Export → Import roundtrip 검증 케이스 추가

### P1
- [ ] 가족 단톡 공유 가능한 QR 코드 또는 단축 링크
- [ ] 인쇄/PDF용 `@media print` CSS 재검증 (페이지 넘김, 여백, 툴바 숨김)
- [ ] 이미지 lazy loading (`loading="lazy"`) — 모바일 데이터 절약
- [ ] 별점 rating 키보드 조작 (접근성)
- [ ] 사진 추가 기능이 localStorage 용량 초과 시 에러 핸들링 강화

### P2
- [ ] PWA / offline 캐시 (검토만)
- [ ] 여행 전·중·후 상태 UI (갈 예정 → 현재 → 다녀온 후)

## L4 · 학습 (learning.md)

### P0 — 출발 전 정리
- [ ] 미션 시스템 구현 회고 → `learning.md`
- [ ] 동선 mid-hotel 메커니즘 회고 → `learning.md`
- [ ] 문서 위계 정리 회고 → `learning.md` (이번 작업 정리)

### P1 — 출발 후
- [ ] 가족 실제 사용 후기 종합 → `learning.md`
- [ ] 다음 시즌 적용 가능한 패턴 추출

---

## 작업 룰

1. **추가** — 어느 Level 의 문제인지 분류 후 해당 섹션에 추가. P0/P1/P2 우선순위 표기.
2. **완료** — `[ ]` → `[x]` 으로 체크. 큰 항목이면 commit hash 도 표기.
3. **갱신** — 영향 받는 *.md 같이 갱신했는지 확인 (DOC-MAP.md 영향 매트릭스).
4. **출발 후** — P0 가 안 끝났으면 매거진은 미완료 상태로 가족 공유. 우선순위 재조정.

## 참조

- 위로 → `CONCEPT.md` (왜 만드나) · `PRD.md` (뭘 만드나)
- 룰 → `CLAUDE.md` (Claude 운영) · `DOC-MAP.md` (문서 인덱스)
- 도메인 → `DESIGN.md` · `ROUTE.md` · `MISSIONS.md`
- 검증 → `VERIFICATION.md`
- 회고 → `learning.md`
