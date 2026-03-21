---
name: web-presentation
description: 순수 HTML, CSS, JavaScript로 웹 프레젠테이션(슬라이드)을 생성합니다. 외부 라이브러리 없이 단일 HTML 파일로 완성되며, 두 가지 모드를 지원합니다. (1) 슬라이드 모드 — GPU 가속 좌우 전환, 터치 스와이프, 슬라이드 카운터. (2) 스크롤 모드 — 수직 scroll-snap, IntersectionObserver 기반 등장 애니메이션, 15가지 템플릿. 사용자가 프레젠테이션, 슬라이드, 발표자료, PPT, 발표, 교육 자료, 튜토리얼, 레슨, 강의 슬라이드, 스크롤 프레젠테이션 등을 만들어달라고 요청할 때 이 스킬을 반드시 사용하세요. 'PPT 만들어줘', '슬라이드 생성', '발표자료 만들어줘', '교육 자료 만들어줘', '튜토리얼 슬라이드', '스크롤 프레젠테이션', 'create a presentation', 'make slides about', 'lesson slides', 'tutorial', 'プレゼン作って', 'スライド作って', 'レッスンスライド' 등 프레젠테이션 생성과 관련된 모든 요청에 반응합니다.
---

# 웹 프레젠테이션 생성기

순수 HTML + CSS + JavaScript만으로 단일 HTML 파일 프레젠테이션을 생성한다. 외부 라이브러리 의존성 없음 (Google Fonts CDN만 예외). 두 가지 모드를 지원한다:

- **슬라이드 모드**: GPU 가속 좌우 전환, 터치 스와이프, 슬라이드 카운터
- **스크롤 모드**: 수직 scroll-snap, IntersectionObserver `.reveal` 등장 애니메이션, 프로그레스 바

두 모드 모두 **15가지 공통 슬라이드 타입**, 다크/라이트 모드, 반응형 디자인, 접근성을 지원한다.

## 모드 선택 가이드

| 조건 | 모드 |
|------|------|
| 기본값 (명시 없음) | 슬라이드 모드 |
| "스크롤", "교육", "튜토리얼", "레슨", "강의", "콘텐츠" 키워드 | 스크롤 모드 |
| "발표", "PPT", "프레젠테이션" 키워드 | 슬라이드 모드 |
| 사용자가 직접 지정 | 해당 모드 |

## 워크플로우

1. **샘플 읽기** — 슬라이드 모드: `references/sample-presentation.html`, 스크롤 모드: `references/sample-scroll.html`
2. 사용자의 주제, 내용, 슬라이드 수 요구사항을 정리한다
3. 대화 컨텍스트에서 사용자 언어를 감지한다 (한국어, 일본어, 영어 등)
4. 슬라이드 구성을 설계한다 (타입별 배치)
5. 샘플의 CSS와 JS를 기반으로 **슬라이드 HTML 콘텐츠만 교체**하여 새 프레젠테이션을 작성한다
6. 작업 디렉토리에 `.html` 파일로 저장한다
7. `open` 명령으로 브라우저에서 확인한다

**핵심: 샘플을 먼저 읽는다.** CSS와 JS 코드를 직접 작성하지 말고, 샘플에서 가져와서 필요한 부분만 수정한다.

## 핵심 원칙

- **단일 파일**: 모든 CSS, JS가 HTML 안에 인라인. 별도 파일 없음
- **GPU 가속**: 전환 애니메이션은 `transform`과 `opacity`만 사용
- **뷰포트 피팅**: 모든 슬라이드가 스크롤 없이 화면에 맞아야 함
- **접근성**: `role`, `aria-roledescription`, `aria-label`, `aria-hidden`, `aria-live` 속성 포함
- **`prefers-reduced-motion`**: 모션 감소 설정 시 전환 시간 단축

## 언어 대응

| 항목 | 한국어 | 일본어 | 영어 |
|------|--------|--------|------|
| `<html lang>` | `ko` | `ja` | `en` |
| `--font-sans` 첫 번째 | `Noto Sans KR` | `Noto Sans JP` | `Inter` |
| ARIA 라벨 | 슬라이드, 이전/다음 | スライド, 前/次 | Slide, Previous/Next |

Google Fonts는 사용하는 폰트만 포함한다. 항상 `Inter`(라틴)와 `JetBrains Mono`(코드)를 포함하고, 언어에 맞는 Noto Sans 변형을 추가한다.

**비렌더블로킹 로드 필수:**
```html
<link href="https://fonts.googleapis.com/css2?..." rel="stylesheet" media="print" onload="this.media='all'">
<noscript><link href="https://fonts.googleapis.com/css2?..." rel="stylesheet"></noscript>
```

## CSS 디자인 시스템

```
--bg: #000000          --accent: #38bd84        --accent-dim: rgba(56,189,132,0.15)
--accent-glow: rgba(56,189,132,0.3)             --text-primary: #e8e8e8
--text-secondary: #888  --text-dim: #555         --border: #1a1a1a
```

타이포그래피 — `clamp()` 기반 반응형:
```
--title-size: clamp(2rem, 5vw, 4rem)     --h2-size: clamp(1.4rem, 3vw, 2.4rem)
--body-size: clamp(0.9rem, 1.4vw, 1.15rem)  --small-size: clamp(0.75rem, 1vw, 0.9rem)
```

배경 오버레이: scanline(`::before`) + grid(`::after`) 패턴을 슬라이드에 적용하여 질감을 준다.

## 슬라이드 타입 카탈로그 (15가지 통합)

모든 타입은 두 모드에서 공통으로 사용 가능하다. 슬라이드 모드에서는 `auto-fragment`, 스크롤 모드에서는 `.reveal`을 사용한다.

### 1. 메인 타이틀 (`slide-title`)
```html
<section class="slide slide-title">
  <div class="slide-content">
    <div class="terminal-badge">키워드</div>
    <div class="big-answer"><span class="accent">강조어</span><br>제목</div>
    <div class="sub">// 부제목</div>
  </div>
</section>
```
**중요: 슬라이드 모드에서 첫 슬라이드에는 반드시 `active` 클래스를 HTML에 직접 포함한다.** JS 실행 전에도 첫 슬라이드가 즉시 표시되어야 빈 화면 현상이 방지된다.

### 2. 섹션 전환 (`slide-transition`)
```html
<section class="slide slide-transition">
  <div class="slide-content">
    <div class="terminal-badge">배지</div>
    <div class="big-title"><span class="accent">강조</span> 제목</div>
    <div class="sub">// 부제목</div>
  </div>
</section>
```

### 3. 기능 목록 (`slide-features`)
```html
<section class="slide slide-features">
  <div class="slide-content">
    <div class="terminal-badge">배지</div>
    <h2 class="section-title" style="font-size: var(--h2-size);">제목</h2>
    <ul class="target-list">
      <li><span class="num">01</span> 항목 1</li>
      <li><span class="num">02</span> 항목 2</li>
    </ul>
  </div>
</section>
```

### 4. 개념 다이어그램 (`slide-concept`)
스타일 A — 화살표 연결 박스:
```html
<div class="diagram">
  <div class="diagram-box"><div class="box-label">레이블</div><div class="box-desc">설명</div></div>
  <div class="diagram-arrow">&rarr;</div>
  <div class="diagram-box output"><div class="box-label">결과</div><div class="box-desc">설명</div></div>
</div>
```
스타일 B — 카드 흐름:
```html
<div class="concept-flow">
  <div class="concept-card"><div class="card-label">레이블</div><div class="card-desc">설명</div></div>
  <div class="concept-arrow">&rarr;</div>
  <div class="concept-card">...</div>
</div>
```

### 5. 단계 플로우 (`slide-steps`)
```html
<div class="step-flow">
  <div class="step-item active"><span class="step-num">01</span> 단계 1</div>
  <span class="step-arrow">&rarr;</span>
  <div class="step-item"><span class="step-num">02</span> 단계 2</div>
</div>
```

### 6. 2열 비교 (`slide-compare`)
```html
<div class="compare-grid">
  <div class="compare-card">
    <div class="card-header">좌측 제목</div>
    <ul class="compare-list"><li><span class="bullet">&gt;</span> 항목</li></ul>
  </div>
  <div class="compare-card">
    <div class="card-header">우측 제목</div>
    <ul class="compare-list"><li><span class="bullet">&gt;</span> 항목</li></ul>
  </div>
</div>
```

### 7. 구조 다이어그램 (`slide-diagram`)
```html
<div class="agent-diagram">
  <div class="main-agent">메인 노드</div>
  <div class="arrows-row">↓ ↓ ↓</div>
  <div class="sub-agents">
    <div class="sub-agent">노드 A</div>
    <div class="sub-agent">노드 B</div>
  </div>
</div>
```

### 8. Before/After (`slide-before-after`)
```html
<div class="comparison-card">
  <div class="card-title">// 문제점</div>
  <ul class="pain-list"><li><span class="x">✕</span> 문제 1</li></ul>
</div>
```

### 9. 코드 (`slide-code`)
```html
<div class="terminal-window">
  <div class="terminal-bar">
    <span class="terminal-dot"></span><span class="terminal-dot"></span><span class="terminal-dot"></span>
  </div>
  <pre class="code-block"><code><span class="cmt">// 주석</span></code>
<code class="highlight"><span class="kw">const</span> x = <span class="num">1</span>;</code></pre>
</div>
```
구문 강조: `.kw`(키워드/파란색), `.str`(문자열/주황), `.cmt`(주석/초록), `.num`(숫자/연두). 코드 최대 8~10줄.

### 10. 터미널 (`slide-terminal`)
```html
<div class="terminal-window">
  <div class="terminal-bar">...</div>
  <div class="terminal-body">
    <div class="prompt-line"><span class="prompt">&gt; </span><span class="cmd">명령어</span></div>
    <div class="prompt-line success">&#10003; 완료</div>
    <span class="cursor-blink"></span>
  </div>
</div>
```

### 11. 결과 카드 (`slide-result`)
```html
<div class="result-card">
  <ul class="result-list"><li><span class="check">&#10003;</span> 결과 항목</li></ul>
</div>
```

### 12. 프롬프트 (`slide-prompt`)
```html
<div class="prompt-card"><p class="card-text">질문이나 프롬프트 내용</p></div>
```

### 13. 요약 (`slide-summary`)
```html
<div class="tag-row"><span class="tag-pill">키워드</span></div>
<p class="summary-text">요약 메시지</p>
```

### 14. 장점 카드 (`slide-benefits`)
```html
<div class="benefit-list">
  <div class="benefit-card">
    <div class="benefit-icon">아이콘</div>
    <div class="benefit-text"><div class="benefit-title">제목</div><div class="benefit-desc">설명</div></div>
  </div>
</div>
```

### 15. CTA (`slide-cta`)
```html
<div class="cta-grid">
  <div class="cta-card"><div class="cta-icon">아이콘</div><div class="cta-label">레이블</div></div>
</div>
<p class="cta-msg">마무리 메시지</p>
```

## 애니메이션 규칙

### auto-fragment (슬라이드 모드)
- 요소에 `class="auto-fragment"` + `style="--i:N"` 추가 (N은 0부터)
- `.slide.active .auto-fragment`가 stagger 애니메이션 트리거
- 2열 레이아웃: 좌측 0,1,2 → 우측 3,4,5
- 슬라이드 재진입 시 `.active` 재추가로 자동 재실행

### .reveal (스크롤 모드)
- `.slide-content` 직접 자식에 `class="reveal"` 추가
- IntersectionObserver가 슬라이드 50% 진입 시 `.visible` → fade-in + slide-up
- nth-child 기반 stagger delay: 1번째 0.1s ~ 6번째 0.6s
- 내부 리스트 항목(li)에도 `.reveal` 개별 부여하여 stagger 효과

## 콘텐츠 밀도 / 슬라이드 수

뷰포트에 맞추는 것이 최우선. 스크롤이 발생하면 안 된다.

- 리스트: 슬라이드당 **최대 5개** 항목
- 텍스트 단락: 2~3줄 이내
- 코드: **최대 8~10줄**
- 내용이 많으면 여러 슬라이드로 분할

슬라이드 수:
- 사용자가 지정하면 그대로 따른다
- 미지정 시 **7~12장**: 타이틀(1) + 본론(5~9) + 마무리(1)
- 스크롤 모드 Round 시스템: 시리즈 콘텐츠는 8~12장/라운드로 분할하여 별도 HTML 파일로 생성

## JS 기능

두 모드 모두 `SlidePresentation` 클래스 패턴을 사용한다.

| 기능 | 슬라이드 모드 | 스크롤 모드 |
|------|--------------|------------|
| 이동 방식 | `transform` 전환 (goToSlide) | `scrollIntoView` (goToSlide) |
| 키보드 | ←→↑↓ Space PgUp/Dn Home/End | ←→↑↓ Space |
| 터치/스와이프 | 좌우 스와이프 (Pointer Events + Touch 폴백) | 상하 스와이프 (Pointer Events + Touch 폴백) |
| 풀스크린 | F키 | F키 |
| 테마 전환 | D키 + 토글 버튼 | D키 + 토글 버튼 |
| 잠금 보호 | `isAnimating` + 700ms 안전 타이머 | — (네이티브 scroll-snap) |
| 프로그레스 | `scaleX` 기반 진행 바 | `width` 기반 진행 바 |
| URL hash | `#slide-N` 동기화 | — |

D/F 키는 `e.code` (물리적 키 위치) 기반으로 처리하여 한글(ㅇ/ㄹ), 일본어 IME 상태에서도 동작한다.

## 다크/라이트 모드 전환

- `[data-theme="light"]` 선택자로 CSS 변수 오버라이드. `--accent`는 유지, `--bg`, `--text-*`, `--border`만 변경
- 토글 버튼: 좌하단 원형 (달/해 아이콘)
- `localStorage`에 테마 저장, 페이지 새로고침 시 복원
- 라이트 모드에서 카드/리스트 아이템 배경을 `#fafafa`로 오버라이드해야 함

## 주의사항

샘플 코드에 이미 올바른 패턴이 구현되어 있다. 수정 시 다음을 깨뜨리지 않도록 주의:

- **Forced reflow**: `void to.offsetHeight` — transition 시작 위치 설정 후 반드시 호출 (슬라이드 모드)
- **인라인 스타일 specificity**: `.active` 추가 시 `style.transform`과 `style.opacity`도 명시 설정
- **cleanup**: 전환 완료 후 인라인 스타일과 `will-change` 반드시 해제
- **`isAnimating` 잠금**: `transitionend` + 700ms safety timer로 이중 보호 (슬라이드 모드)
- **Space 키**: `e.preventDefault()` 없으면 페이지 스크롤 발생
- **`touchmove`**: `{ passive: false }` 필수 (슬라이드 모드)
- **스크롤 모드 `.reveal`**: `.slide-content` 직접 자식에만 부여
- **`scroll-snap-type: y mandatory`**: `<html>` 요소에 설정 (스크롤 모드)

기술적 세부사항은 `references/guide.md`를 참조한다.

## 참고 리소스

- **`references/guide.md`** — DOM 구조, GPU 가속, fragment, 네비게이션, URL hash, 브라우저 호환성 등 상세 기술 가이드
- **`references/sample-presentation.html`** — 슬라이드 모드 10장짜리 완전한 동작 샘플. 이 파일의 CSS와 JS를 기반으로 새 프레젠테이션을 만든다
- **`references/sample-scroll.html`** — 스크롤 모드 8장짜리 완전한 동작 샘플. 이 파일의 CSS와 JS를 기반으로 스크롤 모드 프레젠테이션을 만든다
