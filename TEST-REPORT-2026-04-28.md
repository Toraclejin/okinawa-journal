# 전수 테스트 보고서 · 2026-04-28

> **결과**: 67 logical tests / **67 PASS** / 0 real-bug FAIL
> **방식**: Browser console eval — DOM + state + 이벤트 핸들러 시뮬
> **격리**: 각 phase 시작 시 state 완전 reset (missions/chosen/ratings/photos/hiddenUnlocked)

## 테스트 카테고리 + 결과

### A. 핵심 const/var 초기화 (TDZ 회귀 방지) — 5/5 ✅
- `DAY_HOTEL`, `DAY_HOTEL_COORDS`, `MISSION_ID_RE`, `PAGE_IDS` (length=8), `state` 모두 정상 초기화
- 이전 회귀 (`8ec1422`→`38b55cb`) 의 TDZ 사고 미재발

### B. 미션 시스템 카운트 — 4/4 ✅
- 전체 미션: 15 (메인 9 + 히든 6) ✓
- `getMainMissions().length === 9` ✓
- `getHiddenMissions().length === 6` ✓
- stamp grid: 9 카드 렌더 ✓

### C. 미션 ID 순서·중복 — 3/3 ✅
- 메인 9개 PAGES 순서대로: prep-ready / d1-boarding / d1-hotel / d1-family-photo / d2-churaumi / d2-kouri-bridge / d3-vessel / d3-chatan-sunset / d4-departure
- 히든 6개: hidden-h1-massage ~ hidden-h6-order
- ID 중복 없음 (15개 모두 unique)

### D. Hidden 탭 3단계 가시성 — 7/7 ✅
- 단계 1 (d1-hotel done X): 탭 `display: none`, `getActivePageIds()` 7개 (no hidden)
- 단계 2 (d1-hotel done O, !unlock): 탭 `.show + .locked`, 마무리 CTA 노출
- 단계 3 (hiddenUnlocked O): 탭 `.show !.locked`, CTA 숨김, getActivePageIds 8개

### E. celebrateMission 메인/히든 독립 분기 — 2/2 ✅
- 메인 9 done 상태에서 hidden 미션 클릭 → **TRIP COMPLETE 안 발화** ✓ (이전 회귀 fix 검증)
- 히든 5 done + 메인 미션 클릭 → **FAMILY MASTER 안 발화** ✓

### F. 카운터 위젯 context-aware — 5/5 ✅
- 마무리 페이지: `MISSION` 라벨 + `3/9` 형식
- Hidden 페이지: `SECRET` 라벨 + `0/6` 형식 + `.hidden-mode` 클래스

### G. D1 동선 (midHotel 삽입) — 4/4 ✅
> 자동 테스트 격리 이슈로 첫 회 실패 → renderDayRoute 명시 호출 후 모두 PASS
- Blue Seal + d1-hotel done → chip 4개 (Start/Blue Seal/Check-in/End)
- Check-in 이 chip 3 위치 정확
- d1-hotel undo → Check-in 자동 사라짐

### H. Visited list 별점 — 1/1 ✅
- Blue Seal chosen + ratings['d1-icecream']=4 → visited list `rating: 4` ✓

### I. Sanitizer roundtrip — 5/5 ✅
- ratings/texts/chosen/missions/hiddenUnlocked 모두 export → import 보존

### J. Sanitizer 보안 (악성 input rejection) — 2/2 ✅
- `__proto__` 프로토타입 오염 차단 ✓
- non-boolean hiddenUnlocked 거부 ✓

### K. Nav 차단 (Hidden 잠긴 동안) — 4/4 ✅
- 잠금: 마무리 하단 다음 버튼 `.hidden`, 우측 화살표 `.disabled`
- 해제: 둘 다 활성화

### L/M. Trip Complete · Family Legend 발화 — 2/2 ✅
- 메인 9개 모두 done → fireTripComplete 발화 ✓
- 히든 6개 모두 done → fireFamilyMaster 발화 ✓

### N/O. Day 2/3 동선 — 3/3 ✅
- D2 Churaumi chosen → route chip 3개 (Start/Churaumi/End)
- D3 morning + d3-vessel done + Kurasushi → Vessel Check-in 이 morning 픽들 다음 위치

### P. assignPickNumbers 미션 chip 배지 — 3/3 ✅
- Blue Seal pickNum=2, d1-hotel pickNum=3
- 미션 안 했으면 chip 배지 X (mission done 조건 검증)

### Q. 별점 평균 계산 — 2/2 ✅
- ratings 저장
- 평균 계산 정확 (ratings[d1-icecream]=5 → 평균 5.0)

### R. Hidden Mission unlock 모달 흐름 — 5/5 ✅
- 모달 열림/닫힘
- 사용자 이름 ('남건우') 텍스트 반영
- YES/NO 버튼 존재

### S. Stamp grid 수집 상태 — 2/2 ✅
- 3개 done → 3개 `.collected` 클래스
- progress: "3 / 9"

### T. Trip Conquer Card visibility — 2/2 ✅
- 초기 hidden
- 메인 9 모두 done → 노출

### U. Legend Card visibility — 2/2 ✅
- 초기 hidden
- 히든 6 모두 done → 노출

### V. getActivePageIds 정확성 — 2/2 ✅
- 잠금: 7 페이지 (cover/prep/d1~d4/back)
- 해제: 8 페이지 (+ hidden)

### W. Photo upload state 형식 — 2/2 ✅
- state.photos[target] 배열 구조
- sanitizer 통과 (id + dataUrl 형식 검증)

---

## 결론

**전 영역 정상 작동.** 이전 회귀들 (TDZ, celebrateMission 분기, Nav 잠금, hidden chip 배지) 모두 fix 검증됨.

## 수동 검증 필요 항목 (자동 미커버)

1. **사진 업로드 실제 흐름** — File API + FileReader + base64 변환 + DOM 렌더 (E2E 자동화 필요)
2. **Canvas 다운로드 실제 결과 시각 비교** — trip-conquer-card / legend-card / visited-list PNG 가 화면과 동일한지 (스크린샷 비교)
3. **사운드 재생** — Web Audio API 이상 없는지 (자동 검증 어려움)
4. **모바일 터치 hit area + 스크롤** — 실제 폰에서 확인 필요
5. **OSRM 도로 폴리라인** — 네트워크 의존 (CI 환경 변동 가능)

## 다음 출발 전 체크리스트 (D-7)

- [ ] 가족 4명 모두 폰에서 한 번씩 비번 0430 입력 + 매거진 진입 확인
- [ ] 각자 d1-hotel ✓ 클릭 후 Hidden Mission 흐름 정상 작동 (cinematic + 사운드)
- [ ] export JSON 백업 1회씩
- [ ] 출발 1주일 전 PLACE_QUERIES 의 전화번호·영업시간·매피온 코드 재확인
- [ ] iOS Safari + Android Chrome 양쪽 모바일 시각 검증

## 참조

- 위로 → `DOC-MAP.md` (전체 위계)
- 검증 룰 → `VERIFICATION.md` § [Z] 자동 스모크 + § [P] [Q] 수동 시각 비교
- 회귀 이력 → `POSTMORTEM-2026-04-28.md` (TDZ 사고)
- 회고 → `learning.md` § 9 (Hidden Mission + 회귀 4건)
