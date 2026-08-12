# Holographic Card Viewer

모바일 웹 기반의 홀로그램 카드 뷰어입니다. 실제 홀로그램 카드를 손에 든 듯한 느낌을 구현합니다.

## 파일 구조

```
index.html  - 단일 파일 프로토타입
README.md   - 문서
```

## 홀로그램 레이어 구조

```
.card
├── ::before (메인 홀로그램 밴드)
│   ├── 320% 크기의 rainbow gradient
│   ├── background-position으로 위치 제어
│   └── color-dodge + intensity 조절
│
├── ::after (멀티 텍스처 레이어)
│   ├── CSS Sparkle dots (6개)
│   ├── External holo texture (holo.png)
│   ├── Rainbow gradient overlay
│   └── background-blend-mode: overlay/color-dodge
│
├── .holo-specular (대각선 반사광)
├── .holo-glare (보조 반사광)
└── .holo-grain (미세 질감)
```

## 핵심 설정 (SETTINGS)

`index.html` 하단의 `SETTINGS` 객체에서 모든 값을 튜닝할 수 있습니다:

```javascript
const SETTINGS = {
  // 센서 범위 (±30도에서 최대 효과)
  sensorRange: 30,
  
  // 카드 자체 움직임 (최대 ±5도)
  cardRotateX: 5,
  cardRotateY: 5,
  cardTranslate: 3,
  
  // 레이어별 이동 범위 (%)
  gradientTravel: 75,
  holoTravel: 45,
  sparkleTravel: 20,
  lightTravel: 95,
  
  // 스무딩 (값 클수록 반응 빠름)
  cardSmoothing: 0.065,
  foilSmoothing: 0.10,
  sparkleSmoothing: 0.045,
  
  // 데드존 (이하 값은 무시)
  deadZone: 0.025,
  
  // 레이어별 이동 방향
  gradientDirectionX: -1,
  gradientDirectionY: -1,
  holoDirectionX: -0.6,
  holoDirectionY: -0.6,
  sparkleDirectionX: 0.25,
  sparkleDirectionY: 0.25,
  lightDirectionX: -1,
  lightDirectionY: -1,
};
```

## 센서 → CSS 좌표 변환

```
1. 장치 센서 (gamma/beta) - 기준점 대비 상대값 계산
2. dead zone 적용 (0.025 이하는 0으로)
3. ±30도로 정규화 → -1 ~ 1
4. 각 레이어별 다른 multiplier와 direction 적용
5. 50% 기준 + 이동량 = CSS position

예시:
  gradientX = 50 + foilX * (-1) * 75 = 5% ~ 95%
  lightX = 50 + foilX * (-1) * 95 = -45% ~ 145%
```

## Intensity 계산

```javascript
// 대각선 조합
const diagonal = x * 0.7 + y * 0.3;

// 특정 각도에서만 강해지는 효과
const highlight = smoothstep(-0.8, -0.2, diagonal) * (1 - smoothstep(0.2, 0.8, diagonal));

// 기본 강도 + 하이라이트boost
const intensity = 0.3 + highlight * 0.7;
```

## 테스트 방법

### iPhone
```bash
# 로컬 서버 필요
python -m http.server 8000
# Safari: http://localhost:8000
```

### Android
Chrome에서 열기 (로컬 서버 권장)

### PC
브라우저에서 파일 열기 + 마우스 움직이기

## 디버그 모드

카드 5번 빠르게 탭 → 좌표/강도 표시

## 프리셋

[data-preset] 속성으로 색상 변경:
- `rainbow`: 핑크→보라
- `gold`: 금색
- `silver`: 은색
- `galaxy`: 보라→진보라
- `aurora`: 녹색→청록
- default: 청록→마젠타
