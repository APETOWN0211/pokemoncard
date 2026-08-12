# Holographic Card Viewer

모바일 웹 기반의 홀로그램 카드 뷰어입니다. 포켓몬 카드, 스포츠 카드, 포토카드 등에 실제 홀로그램 포일 효과를 시뮬레이션합니다.

원본 Reference: [simeydotme/PrQKgo](https://codepen.io/simeydotme/pen/PrQKgo)

## 주요 파일

```
index.html  - 단일 파일 프로토타입 (HTML + CSS + JS 통합)
README.md    - 본 문서
```

## 홀로그램 레이어 구조

원본 CodePen의 `::before`와 `::after` 방식을 채택하여 다중 배경 레이어를 겹칩니다.

```
.card
├── ::before (메인 홀로그램 밴드)
│   ├── color-dodge 혼합
│   ├── --gradient-x/y로 위치 제어
│   └── intensity에 따른 opacity/contrast 변화
│
├── ::after (멀티 텍스처 레이어)
│   ├── Sparkle dots (CSS 생성)
│   ├── Diagonal diffraction patterns
│   ├── Rainbow gradient overlay
│   ├── background-blend-mode: overlay
│   └── mix-blend-mode: color-dodge
│
├── .holo-specular (반사광 스팟)
│   └── radial-gradient + screen 혼합
│
└── .holo-glare (대각선 반사 띠)
    └── linear-gradient + color-dodge
```

## 센서 → CSS 좌표 변환

```
장치 센서 → 상대값 계산 → [-1, 1] 정규화 → CSS % 위치

1. 센서 원시값
   - gamma: 좌우 기울기 (-90° ~ 90°)
   - beta: 앞뒤 기울기 (-180° ~ 180°)

2. 상대값 계산
   - relativeGamma = gamma - centerGamma
   - relativeBeta = beta - centerBeta

3. [-1, 1] 범위로 정규화
   - targetX = clamp(relativeGamma / 25, -1, 1)
   - targetY = clamp(relativeBeta / 25, -1, 1)

4. 레이어별 parallax 계산
   - --gradient-x = 50 + currentX * 70  (±70%)
   - --holo-x = 50 + currentX * 45      (±45%)
   - --sparkle-x = 50 + currentX * 20   (±20%)
   - --light-x = 50 + currentX * 90     (±90%)
```

## Intensity 계산

```javascript
// 기울기 거리에 따른 강도 계산
const distance = Math.sqrt(x * x + y * y);
const intensity = clamp(distance * 1.2, 0, 1);

// intensity로 opacity, brightness, contrast 조절
--holo-opacity: 0.3 + intensity * 0.5;
--holo-brightness: 0.7 + intensity * 0.5;
--holo-contrast: 0.8 + intensity * 0.5;
```

## 테스트 방법

### iPhone에서 테스트

1. 로컬 서버 실행 (필수 - CORS 때문)
```bash
# Python 3
python -m http.server 8000

# 또는 npx
npx serve .
```

2. Safari에서 `http://localhost:8000` 열기
3. 카드 이미지 업로드
4. **"모션 효과 시작"** 버튼 탭
5. 권한 허용 요청 시 **"허용"** 선택
6. 폰을 기울이면 홀로그램 효과 확인

### Android에서 테스트

1. Chrome에서 `index.html` 열기 (또는 로컬 서버 사용)
2. 카드 이미지 업로드
3. **"모션 효과 시작"** 버튼 탭

### PC에서 테스트

1. 브라우저로 `index.html` 열기
2. 카드 이미지 업로드
3. **"모션 효과 시작"** 버튼 클릭
4. 마우스를 화면 중앙에서 움직이기

## 디버그 모드

카드 영역을 5번 빠르게 탭하면 좌표 정보 표시:
- `gamma`, `beta`: 센서 원시값
- `targetX/Y`: [-1, 1] 목표값
- `currentX/Y`: 보간된 현재값
- `intensity`: 0~1 강도값

## 컬러 프리셋

`data-preset` 속성으로 홀로그램 색상 변경 가능:

```html
<div class="card" data-preset="rainbow">
<div class="card" data-preset="gold">
<div class="card" data-preset="silver">
<div class="card" data-preset="galaxy">
<div class="card" data-preset="aurora">
```

커스텀 색상 설정:
```css
.card {
  --color1: rgb(0, 231, 255);  /* 첫 번째 포일 색 */
  --color2: rgb(255, 0, 231);  /* 두 번째 포일 색 */
}
```

## 핵심 구현 포인트

### 카드 자체는 절대 움직이지 않음

```css
/* ❌ 이렇게 하지 마세요 */
.card {
  transform: rotateX(...) rotateY(...) rotateZ(...);
  perspective: 1000px;
}

/* ✅ 이렇게 해야 합니다 */
.card {
  transform: none !important;
  perspective: none !important;
}
```

### 원본 CodePen 방식의 레이어 구조

```css
/* 메인 홀로그램 밴드 */
.card::before {
  background-image: linear-gradient(115deg, transparent..., var(--color1)...);
  background-size: 300% 300%;
  background-position: var(--gradient-x) var(--gradient-y);
  mix-blend-mode: color-dodge;
  opacity: var(--holo-opacity);
  filter: brightness(var(--holo-brightness)) contrast(var(--holo-contrast));
}

/* 멀티 텍스처 */
.card::after {
  background-image: 
    radial-gradient(...),  /* sparkles */
    repeating-linear-gradient(...),  /* diffraction */
    linear-gradient(...);  /* rainbow */
  background-blend-mode: overlay;
  mix-blend-mode: color-dodge;
}
```

### 센서 스무딩

```javascript
// 값 업데이트와 DOM 변경 분리
targetX = sensorValue;  // sensor callback에서
targetY = sensorValue;

// 별도 루프에서 보간
function animate() {
  currentX += (targetX - currentX) * 0.08;
  updateHologram();  // DOM 업데이트
  requestAnimationFrame(animate);
}
```

## 브라우저 호환성

| 브라우저 | 모션 센서 | 참고 |
|---------|----------|------|
| iOS Safari 13+ | ✅ | `requestPermission()` 필요 |
| Chrome Android | ✅ | 별도 권한 불필요 |
| Chrome Desktop | ⚠️ | 마우스 폴백 사용 |
| Firefox | ⚠️ | 마우스 폴백 권장 |
| Safari Desktop | ⚠️ | 마우스 폴백 사용 |

## 라이선스

MIT License
