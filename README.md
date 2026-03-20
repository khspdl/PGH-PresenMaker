# PGH-PresenMaker

순수 HTML, CSS, JavaScript로 웹 프레젠테이션을 생성하는 Claude Code 플러그인입니다.

외부 라이브러리 없이 **단일 HTML 파일**로 완성되며, 브라우저에서 바로 실행할 수 있습니다.

## 주요 기능

- **GPU 가속 슬라이드 전환** — `translate3d` + `opacity` 기반 부드러운 애니메이션
- **키보드 네비게이션** — 방향키, Space, PageUp/Down, Home/End
- **F키 풀스크린** — Fullscreen API로 전체화면 토글
- **터치/스와이프** — Pointer Events + Touch Events 폴백
- **프로그레스 바** — 상단 진행률 표시
- **슬라이드 카운터** — 우하단 `N / M` 형식
- **자동 순차 등장** — auto-fragment로 리스트 항목 순차 애니메이션
- **반응형 타이포그래피** — `clamp()` 기반 자동 크기 조절
- **다크 테마** — CSS 변수 기반 테마 시스템
- **다국어 지원** — 한국어, 영어, 일본어 등 자동 감지
- **접근성** — ARIA 속성, `prefers-reduced-motion` 대응

## 사용법

Claude Code에서 프레젠테이션 관련 요청을 하면 자동으로 스킬이 활성화됩니다.

```
> Git 브랜치 전략에 대한 발표자료 만들어줘
> Create a presentation about REST API best practices
> Reactのhooksについてプレゼンを作ってください
```

## 슬라이드 타입

| 타입 | 클래스 | 용도 |
|------|--------|------|
| 타이틀 | `theme-title` | 오프닝 슬라이드, gradient 텍스트 |
| 콘텐츠 | `theme-content` | 리스트, 설명 |
| 2열 | `theme-content` + `two-col` | 비교, 분류 |
| 코드 | `theme-code` | 터미널 스타일 코드 블록 |
| 마무리 | `theme-end` | 엔딩, glow 애니메이션 |

## 설치

```bash
claude plugin marketplace add khspdl/PGH-PresenMaker
claude plugin install PGH-PresenMaker@pgh-marketplace
```

## Author

**ParkGeonhee**
