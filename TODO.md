# TODO

## 1. Day별 시안성 강화
- [ ] PC에서 각 Day opener (Day One/Two/Three/Four) 가독성 최종 점검
- [ ] 모바일에서 같은 계열 색상 겹침 문제 확인
  - 파란 배경 + 파란 floaty 원형이 글자 가리는 곳
  - 노란 원(sun circle)이 타이틀 침범
  - mood card 패턴 배경 + 흰 글씨 대비 약한 곳
- [ ] 읽기 어려운 폰트 조합 점검 (Fraunces italic + 한글 겹침 등)

## 2. 장소 정보 검증
- [ ] 4일 × 옵션별 장소명/주소/영업시간/거리 실제 정보 크로스체크
  - Vessel Hotel Campana (차탄)
  - Sunset Beach · Blue Seal · Kurasushi · Jack's Steakhouse
  - Churaumi Aquarium · Kourikyo Bridge · American Village · Senagajima
  - Shurijo Castle 등
- [ ] 가격/요금 2026년 기준으로 갱신
- [ ] 공식 사이트/구글맵에서 현재 운영 중 여부 확인

## 3. 장소 탭 → 구글맵 연결
- [ ] 각 option card, mood card에 클릭 액션 추가
- [ ] 클릭 시 확인 모달: "구글맵에서 '{장소명}'을 여시겠습니까?" [Yes / No]
- [ ] Yes 시 `https://www.google.com/maps/search/?api=1&query={name}` 새 탭
- [ ] No 시 모달 닫기
- [ ] 실수 클릭 방지 (카드 전체보다 좁은 hit area, 또는 아이콘 버튼 별도 배치)

## 4. 추가 점검 항목 (제안)
- [ ] 표지 OKINAWA 뒤 floaty 원이 텍스트 가림 → 가족 공유 시 첫인상 중요
- [ ] 이미지 lazy loading (`loading="lazy"`) 모바일 데이터 절약
- [ ] 사진 추가 기능이 localStorage 용량 초과 시 에러 핸들링
- [ ] 인쇄/PDF용 `@media print` CSS 재검증 (페이지 넘김, 여백, 툴바 숨김)
- [ ] 별점 rating 키보드 조작 (접근성)
- [ ] 여행 전·중·후 상태 UI (갈 예정 → 현재 → 다녀온 후)
- [ ] 가족 공유용 QR 코드 또는 단축 링크
- [ ] Day opener 제목이 긴 경우 wrap 처리 검증
