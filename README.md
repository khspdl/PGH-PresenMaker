# PGH-PresenMaker

순수 HTML, CSS, JavaScript로 웹 프레젠테이션을 생성하는 Claude Code 플러그인입니다.

외부 라이브러리 없이 **단일 HTML 파일**로 완성되며, 브라우저에서 바로 실행할 수 있습니다.

## 두 가지 모드

### 슬라이드 모드

좌우 전환 기반의 전통적인 프레젠테이션 스타일입니다.

- **GPU 가속 슬라이드 전환** — `translate3d` + `opacity` 기반 부드러운 좌우 애니메이션
- **키보드 네비게이션** — 방향키, Space, PageUp/Down, Home/End
- **F키 풀스크린** — Fullscreen API로 전체화면 토글
- **터치/스와이프** — Pointer Events + Touch Events 폴백
- **슬라이드 카운터** — 우하단 `N / M` 형식
- **자동 순차 등장** — auto-fragment로 리스트 항목 순차 애니메이션
- **D키 다크/라이트 모드** — CSS 변수 기반 테마 전환

### 스크롤 모드

수직 스크롤 기반의 교육/튜토리얼 스타일입니다.

- **수직 scroll-snap** — 슬라이드 단위로 정확히 스냅되는 스크롤 네비게이션
- **IntersectionObserver 등장 애니메이션** — `.reveal` 클래스 기반 fade-in + slide-up
- **터치/스와이프** — 상하 스와이프로 슬라이드 이동
- **15가지 슬라이드 템플릿** — 타이틀, 전환, 목록, 다이어그램, 비교, 터미널, 코드 등
- **프로그레스 바** — 상단 스크롤 진행률 표시
- **Round 시스템** — 시리즈 콘텐츠를 여러 라운드로 분할
- **D키 다크/라이트 모드** — CSS 변수 기반 테마 전환

## 공통 기능

- **반응형 타이포그래피** — `clamp()` 기반 자동 크기 조절
- **다크 테마** — CSS 변수 기반, `--accent` 색상 커스터마이징 가능
- **다국어 지원** — 한국어, 영어, 일본어 등 자동 감지
- **접근성** — ARIA 속성, `prefers-reduced-motion` 대응
- **비렌더블로킹 폰트 로딩** — `media="print" onload` 패턴으로 빈 화면 방지

## 사용법

Claude Code에서 프레젠테이션 관련 요청을 하면 자동으로 스킬이 활성화됩니다.

**슬라이드 모드** (기본):
```
> Git 브랜치 전략에 대한 발표자료 만들어줘
> Create a presentation about REST API best practices
> Reactのhooksについてプレゼンを作ってください
```

**스크롤 모드**:
```
> Docker 시작하기 교육 자료 만들어줘
> Git 튜토리얼 슬라이드 만들어줘
> 스크롤 프레젠테이션으로 만들어줘
```

## 슬라이드 타입 (15가지 통합)

두 모드에서 공통으로 사용 가능한 통합 슬라이드 타입입니다.

| 타입 | 클래스 | 용도 |
|------|--------|------|
| 메인 타이틀 | `slide-title` | 큰 텍스트 오프닝 |
| 섹션 전환 | `slide-transition` | 섹션 구분자 |
| 기능 목록 | `slide-features` | 번호 매긴 항목 리스트 |
| 개념 다이어그램 | `slide-concept` | 화살표 연결 박스/카드 |
| 단계 플로우 | `slide-steps` | 번호 매긴 단계 흐름 |
| 2열 비교 | `slide-compare` | 좌우 비교 그리드 |
| 구조 다이어그램 | `slide-diagram` | 트리/계층 구조 |
| Before/After | `slide-before-after` | 문제-해결 대비 |
| 코드 | `slide-code` | 구문 강조 코드 블록 |
| 터미널 | `slide-terminal` | 명령어/프롬프트 표시 |
| 결과 카드 | `slide-result` | 체크 리스트 결과 |
| 프롬프트 | `slide-prompt` | 질문/인용 블록 |
| 요약 | `slide-summary` | 태그 기반 핵심 정리 |
| 장점 카드 | `slide-benefits` | 아이콘+제목+설명 카드 |
| CTA | `slide-cta` | 행동 유도 그리드 / 마무리 |

## 설치

```bash
claude plugin marketplace add khspdl/PGH-PresenMaker
claude plugin install PGH-PresenMaker@pgh-marketplace
```

## Author

**ParkGeonhee**
