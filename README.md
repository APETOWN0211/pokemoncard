# Holographic Card Viewer

성능 최적화된 홀로그램 카드 뷰어입니다.

## 핵심 변경사항

### 매 프레임 변경되는 CSS 속성

| 요소 | 속성 |
|------|------|
| `.card` | `transform` (rotateX, rotateY) |
| `.holo` | `transform` (translate3d), `opacity` |
| `.glare` | `transform` (translate3d), `opacity` |
| `.sparkle` | `transform` (translate3d), `opacity` |

### 변경되지 않는 속성

- ❌ `background-position` - 레이어를 translate로 이동
- ❌ `filter` (brightness, contrast) - 고정값 사용
- ❌ `box-shadow` - 고정값
- ❌ animated GIF - CSS radial-gradient로 대체
- ❌ `backdrop-filter` - 사용 안함

## 구조

```
.card-stage (perspective)
└── .card
    ├── .card__image
    ├── .holo (translate3d + opacity)
    ├── .glare (translate3d + opacity)
    └── .sparkle (translate3d + opacity)
```

## 설정

```javascript
const SETTINGS = {
  maxRotateX: 14,
  maxRotateY: 14,
  holoRange: 180,      // px
  glareRange: 220,     // px
  sparkleRange: 100,   // px
  smoothing: 0.1,
  deadZone: 0.02,
  sensorRange: 30,
};
```

## 테스트

```bash
# iPhone Safari
python -m http.server 8000
```

카드 5번 탭 → 디버그 표시
