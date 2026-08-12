# Holographic Card Viewer

실제 홀로그램 카드를 손에 든 듯한 느낌을 구현한 모바일 웹 앱입니다.

## 핵심 기능

- **실시간 3D 카드 회전**: 포인터/기기 방향에 따라 카드가 실제로 기울어짐
- **홀로그램 반사 효과**: 회전과 독립적으로 움직이는 다층 반사 레이어
- **스마트폰 센서 지원**: iOS/Android DeviceOrientation
- **PC 마우스 폴백**: 데스크톱에서도 동일한 효과

## 파일 구조

```
index.html  - 단일 파일 프로토타입
README.md   - 문서
```

## 설정 (SETTINGS)

`index.html` 하단의 `SETTINGS` 객체에서 튜닝:

```javascript
const SETTINGS = {
  // 카드 3D 회전 (±14도)
  maxRotateX: 14,
  maxRotateY: 14,
  perspective: 850,
  
  // 홀로그램 이동 범위 (%)
  gradientTravel: 80,
  holoTravel: 55,
  sparkleTravel: 25,
  lightTravel: 100,
  
  // 홀로그램 이동 방향 (음수 = 카드와 반대)
  gradientDirX: -1,
  gradientDirY: -1,
  holoDirX: -0.6,
  holoDirY: -0.6,
  
  // 스무딩 속도
  cardSmoothing: 0.085,
  foilSmoothing: 0.10,
  
  // 데드존
  deadZone: 0.025,
};
```

## 좌표 시스템

모든 입력(포인터/자이로스코프)이 하나의 정규화된 좌표로 변환:

```
x = -1 ~ +1 (좌 ~ 우)
y = -1 ~ +1 (상 ~ 하)
```

### PC (마우스)
```javascript
const x = (clientX / innerWidth - 0.5) * 2;
const y = (clientY / innerHeight - 0.5) * 2;
```

### Mobile (DeviceOrientation)
```javascript
targetX = (gamma - centerGamma) / 30;  // ±30도 = ±1
targetY = (beta - centerBeta) / 30;
```

## 카드 회전 vs 홀로그램 이동

### 카드 회전
```
rotateY = currentX * 14  // 좌우 기울기
rotateX = currentY * -14  // 상하 기울기
```

### 홀로그램 이동 (서로 다른 방향)
```
gradient: 50 + foilX * -1 * 80   // 정반대
holo:     50 + foilX * -0.6 * 55 // 반대 (적음)
sparkle:  50 + sparkX * 0.3 * 25  // 같은 방향
light:    50 + foilX * -1 * 100   // 정반대
```

## 테스트

### iPhone
```bash
python -m http.server 8000
# Safari: http://localhost:8000
```

### Android
Chrome에서 열기

### PC
브라우저에서 열기 + 마우스 움직이기

## 디버그

카드 5번 빠르게 탭 → 좌표/회전角度 표시

## 프리셋

[data-preset] 속성으로 색상 변경:
- `rainbow`: 핑크→보라
- `gold`: 금색
- `silver`: 은색
- `galaxy`: 보라→진보라
- `aurora`: 녹색→청록
