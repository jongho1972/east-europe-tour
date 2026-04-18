# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 프로젝트 개요

이 디렉토리는 두 개의 독립적인 웹 프로젝트로 구성되어 있습니다.

| 프로젝트 | 경로 | 배포 URL |
|----------|------|----------|
| 여행 코스 지도 (제목: 동유럽 여행 코스 2026) | `index.html` | https://jhawk-east-europe-tour.netlify.app |

## 배포 방식

- **저장소**: https://github.com/jongho1972/east-europe-tour
- GitHub `main` 브랜치 푸시 시 Netlify 자동 배포
- `.gitignore`로 PDF, xlsx, gmap, gsheet, txt 파일은 제외 (`index.html`, `CLAUDE.md`만 관리)

## 비밀번호 게이트

- 페이지 진입 시 오버레이형 `#auth-gate`가 `body.auth-locked`와 함께 노출
- 비밀번호: `0000` (스크립트 내 `PW` 상수)
- sessionStorage 저장 금지 — 매 로드 재인증 (Chrome 세션 복원 이슈 방지)
- 테마: 동유럽 프로젝트 블루 그라디언트(`#2c3e50 → #3498db`)

## 동유럽여행일정지도.html 구조

단일 HTML 파일에 CSS, JavaScript가 모두 포함된 self-contained 구조입니다.

**지도 라이브러리**: Google Maps JavaScript API
- API 키: Google Cloud Console "My Maps Project" 프로젝트
- 도메인 제한: `https://jhawk-east-europe-tour.netlify.app/*`, `http://localhost/*`
- Maps JavaScript API 활성화 필요 (Google Cloud Console → 라이브러리)
- 로컬 테스트: `python3 -m http.server 8080` 실행 후 `http://localhost:8080/동유럽여행일정지도.html`

**모바일 레이아웃** (`max-width: 680px`):
- `#mobile-tabs`: `position:fixed; top:0; height:44px; z-index:9999` — 항상 최상단 고정
- `#sidebar`: `position:fixed; top:44px; left:0; right:0; bottom:0; overflow-y:scroll` — 탭 아래 전체 스크롤
- `#map`: 기본 `display:none` → `body.show-map` 시 `position:fixed; top:44px; left:0; right:0; bottom:0`
- 지도 초기화: `switchMobileTab('map')` 클릭 시 `setTimeout(createMap, 200)` — CSS 적용 후 Google Maps 초기화
- Google Maps는 `display:none` 컨테이너에서 초기화 불가 → lazy init 필수
- `body.show-map` 클래스 추가/제거로 일정↔지도 전환

**핵심 데이터**: `DAYS` 배열 — 11일차 일정, 각 day마다 `places[]` 배열로 장소 정의
```js
{ day: N, date: "N일차", city: "도시명", cityKey: "budapest|vienna|prague|all", color: "#hex",
  places: [{ time: "시간대", label: "장소명", lat: ..., lng: ... }] }
```
- `date` 필드는 날짜 확정 전까지 `"N일차"` 형식 사용 (확정 후 `"7/3(목)"` 형식으로 복원)

**경로선**: `routeLines` 배열 — 각 선에 `days: [N]` 속성으로 하이라이트 연동 일차 지정
```js
{ pts: [[lat,lng],...], color, dash: bool, tip, days: [N], line: <google.maps.Polyline> }
```
- `dash: true`인 경우 `strokeOpacity:0` + `icons` repeat 패턴으로 점선 구현 (D6 할슈타트, D7 경로)
- 경로선 스타일 변경은 `setLineStyle(r, 'normal'|'highlighted'|'dimmed')` 함수 사용

**마커**: `google.maps.Marker` + SVG 아이콘 (`makeIcon(color, dayNum, scale)`)
- 하이라이트 시 `scale(1.25)` 적용: `makeIcon(color, dayNum, 1.25)`로 아이콘 교체

**하이라이트 동작**:
- 사이드바 날짜 클릭 → `toggleDay(N)` → `highlightDay(N)` 호출
- 선택 날: 마커 scale(1.25) + zIndex:1000, 연관 경로선 굵게(weight:5)
- 비선택: 마커 opacity 0.15, 경로선 opacity 0.08
- 재클릭 또는 도시 탭 전환 → `resetHighlight()`

**팝업**: `google.maps.InfoWindow` (단일 인스턴스 `infoWindow` 재사용)
- 경로선 hover 툴팁: 별도 `tooltipWindow` 인스턴스 사용

**시간대 badge 색상 규칙**:
- 오전: 노란색 / 오후: 청록색 / 저녁·야간·출발·도착: 빨간색 / 종일: 초록색

## MyHomepage 구조

- `index.html`: 프로필 섹션 + 여행 일정 iframe 섹션
- `index.css`: 모든 스타일 (Nanum Gothic 폰트)
- `jh.jpg`: 프로필 사진
- 여행 일정은 `jhawk-east-europe-tour.netlify.app`을 **iframe으로 임베드** — 직접 코드 없음

**링크 카드 패턴** (`.travel-btn`):
```html
<a class="travel-btn" href="URL" target="_blank" rel="noopener">
  <span class="travel-btn-icon">이모지</span>
  <span class="travel-btn-text">
    <span class="travel-btn-label">제목</span>
    <span class="travel-btn-sub">부제</span>
  </span>
  <span class="travel-btn-arrow">↗</span>
</a>
```

## 여행 일정 정보

- **일정**: 2026년 7월 3일(목) ~ 7월 13일(일), 9박 11일
- **항공**: OZ547 (출발 12:35, 18:05 도착) / OZ546 (18:50 출발, 익일 13:10 도착)
- **나라별 색상**: 헝가리 `#e74c3c` (D1~D3) / 오스트리아 `#3498db` (D4~D7) / 체코 `#9b59b6` (D8~D10)
- **숙박**: 부다페스트 2박(D1-2) / 비엔나 3박(D3-5) / 잘츠부르크 1박(D6) / 체스키크룸로프 1박(D7) / 프라하 2박(D8-9)
- **숙소 위치 좌표**: D1 숙소(D8 호텔) `47.4971,19.0509` / D3 빈(호텔 자이트가이스트) `48.1824,16.3781` / D6 잘츠부르크(아들러호프) `47.8063,13.0394` / D7 체스키(라르고) `48.81132,14.31369` / D8 프라하(Hotel Leon D'Oro) `50.0837,14.4211`
- **경유**: D6 할슈타트 현지 투어(오전) → 잘츠부르크 현지 투어(오후) → 잘츠부르크 숙박 / D7 잘츠부르크 관광 → CK셔틀(16:00) → 체스키크룸로프 / D8 체스키크룸로프 오전 관광 → 프라하 이동
- **인원**: 20대 여성 2명, 50대 여성 1명, 50대 남성 1명

## 일정 수정 시 주의사항

- D5(월요일): 빈 미술사 박물관 **월요일 휴관** → D4(일요일) 또는 D3(토요일)에 배치 필요
- D4(일요일): 벨베데레 → 호프부르크 → 빈 국립 오페라 순서 (일요일 — 주요 관광지 모두 오픈)
- D5(월요일): 쇤브룬 → 노이바우 → 카를 성당 → Naschmarkt 순서 (미술사 박물관 휴관이므로 제외)
- D6: 할슈타트 현지 투어 (오전) → 잘츠부르크 현지 투어 (오후) → 잘츠부르크 숙박 (아들러호프)
- D7: 잘츠부르크 시내 관광 (성 안드레 성당·미라벨 정원·모차르트 생가·호엔 잘츠부르크 성) → CK셔틀 16:00 → 체스키크룸로프 숙박 (라르고)
- D8: 체스키크룸로프 오전 관광 후 프라하로 이동 (샌딩 투어 or 버스)
- 빈→잘츠부르크 실선 D6 연동 / 잘츠부르크↔할슈타트 점선(파랑) D6 연동 / 잘츠부르크→체스키 점선(회색) D7 연동 / 체스키→프라하 실선 D8 연동
