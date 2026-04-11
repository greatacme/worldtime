# World Time Visualization

세계 시간대를 시각적으로 표현하는 인터랙티브 웹 애플리케이션.

## 기술 스택

- **D3.js**: 지도 렌더링 및 지리 투영
- **TopoJSON**: 국가 경계 데이터
- **Vanilla JavaScript**: 시간대 계산, UI 인터랙션
- **CSS Grid/Flexbox**: 레이아웃
- **Intl API**: 시간대별 시간 계산

## 구조

**단일 파일 구성** (`index.html`)
- 모든 HTML, CSS, JavaScript가 포함됨
- CSS는 `<style>` 태그 내에 minified 형태
- JavaScript는 `<script>` 태그 내에 포함

## 주요 컴포넌트

### 1. 도시 목록 (좌측 패널)
- 선택된 도시 리스트 표시
- 각 도시별 현재 시간, 시간대, 국가 표시
- 도시 추가/제거 기능

### 2. 지도 (상단)
- D3.js로 렌더링된 세계 지도
- 선택된 도시 위치 마커 표시
- 시간대별로 착색 (낮/밤)

### 3. 타임스트립 (중앙)
- 시간대별 시간 흐름 표시
- 슬라이더로 ±24시간 이동 가능
- 현재 시각 기준으로 동적 계산

### 4. 베젤 (하단)
- 회전 가능한 24시간 시계
- 시간대 오프셋 표시 (-12h ~ +12h)
- 선택된 도시의 시간대 표시 (주황색)
- 마우스/터치로 회전 가능

## 핵심 함수

### 시간 계산
- `selTime()`: 현재 시간 + 슬라이더 오프셋
- `localHourAt(ts, tz)`: 특정 시간대의 시간 계산
- `bzCityOffset(tzId)`: 도시의 UTC 오프셋 계산 (±12시간 범위로 정규화)

### 베젤 관련
- `buildBezelSVG()`: 베젤 SVG 생성
- `bzApplyDeg(deg)`: 베젤 회전 각도 적용
- `bzPt(angleDeg, r)`: 극좌표를 SVG xy로 변환
- `bzSeoulFromDeg(deg)`: 베젤 회전각도에서 서울 시간 계산

### 지도 관련
- `initMapCanvas()`: 지도 캔버스 초기화
- `drawMapCanvas()`: 지도 렌더링
- `cityPos(lon, lat)`: 경위도 → 화면 좌표 변환

### UI 관리
- `renderList()`: 도시 목록 렌더링
- `renderMap()`: 지도 마커 렌더링
- `addCity(id)`: 도시 추가
- `removeCity(id)`: 도시 제거

## 데이터

### 도시 데이터 (DB)
```javascript
{id, name, en, country, flag, tz, lat, lon}
```

### 베젤 도시 (BZ_CITIES)
```javascript
{offset, name}
```
UTC 오프셋별 대표 도시 표시 (offset: 6~12)

## 주요 구현 특이사항

### 베젤 색상 변경 로직
선택된 도시의 시간대 오프셋을 계산하여 베젤 라벨 색상 변경
- 계산된 offset이 `BZ_CITIES`의 offset과 일치하면 주황색(#ffcc66)
- 미일치하면 파란색(#8aadd4)

### Offset 정규화 (중요)
`bzCityOffset` 함수에서 오프셋 정규화 시 경계값 처리:
```javascript
if(off>=12*60)off-=24*60;    // >= 사용
if(off<=-12*60)off+=24*60;   // <= 사용
```
`>` 또는 `<`만 사용하면 오클랜드(UTC+12) 같은 경계 도시에서 색상 변경 불가

### 지도 여백 제거
`.mapsec` 스타일에 음수 margin 적용하여 화면 가득 확장:
```css
margin:0 -22px;
width:calc(100% + 44px);
```

## 개발 시 주의사항

1. **시간대 계산은 Intl API 기반**
   - 브라우저 기본 제공이므로 외부 라이브러리 불필요
   - 서머타임 자동 반영

2. **베젤 오프셋은 정수 시간만 지원**
   - 인도(UTC+5:30) 같은 30분 단위 오프셋은 반올림됨

3. **단일 파일 구조 유지**
   - CSS 추가 시 `<style>` 내에 추가
   - JS 추가 시 마지막 `</script>` 직전에 추가

## 파일 크기 최적화

- CSS/JS minified 형태 (수동 minify)
- 불필요한 파일 제거 (worldtime.pptx, worldmap.png)
