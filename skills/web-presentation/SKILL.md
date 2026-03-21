---
name: web-presentation
description: 순수 HTML, CSS, JavaScript로 웹 프레젠테이션(슬라이드)을 생성합니다. 외부 라이브러리 없이 단일 HTML 파일로 완성되며, 두 가지 모드를 지원합니다. (1) 슬라이드 모드 — GPU 가속 좌우 전환, 터치 스와이프, 슬라이드 카운터. (2) 스크롤 모드 — 수직 scroll-snap, IntersectionObserver 기반 등장 애니메이션, 14가지 템플릿. 사용자가 프레젠테이션, 슬라이드, 발표자료, PPT, 발표, 교육 자료, 튜토리얼, 레슨, 강의 슬라이드, 스크롤 프레젠테이션 등을 만들어달라고 요청할 때 이 스킬을 반드시 사용하세요. 'PPT 만들어줘', '슬라이드 생성', '발표자료 만들어줘', '교육 자료 만들어줘', '튜토리얼 슬라이드', '스크롤 프레젠테이션', 'create a presentation', 'make slides about', 'lesson slides', 'tutorial', 'プレゼン作って', 'スライド作って', 'レッスンスライド' 등 프레젠테이션 생성과 관련된 모든 요청에 반응합니다.
---

# 웹 프레젠테이션 생성기

순수 HTML + CSS + JavaScript만으로 단일 HTML 파일 프레젠테이션을 생성한다. 외부 라이브러리 의존성 없음 (Google Fonts CDN만 예외). 두 가지 모드를 지원한다:

- **슬라이드 모드**: GPU 가속 좌우 전환, 터치 스와이프, 슬라이드 카운터, 다크/라이트 모드
- **스크롤 모드**: 수직 scroll-snap, IntersectionObserver `.reveal` 등장 애니메이션, 14가지 템플릿, 다크/라이트 모드

## 모드 선택 가이드

| 조건 | 모드 |
|------|------|
| 기본값 (명시 없음) | 슬라이드 모드 |
| "스크롤", "교육", "튜토리얼", "레슨", "강의", "콘텐츠" 키워드 포함 | 스크롤 모드 |
| "발표", "PPT", "프레젠테이션" 키워드 포함 | 슬라이드 모드 |
| 사용자가 직접 지정 | 해당 모드 |

슬라이드 모드는 `references/sample-presentation.html`, 스크롤 모드는 `references/sample-scroll.html`을 읽어 시작한다.

## 슬라이드 모드 워크플로우

1. `references/sample-presentation.html`을 읽어 CSS/JS 전체 구조를 파악한다
2. 사용자의 주제, 내용, 슬라이드 수 요구사항을 정리한다
3. 대화 컨텍스트에서 사용자 언어를 감지한다 (한국어, 일본어, 영어 등)
4. 슬라이드 구성을 설계한다 (타입별 배치)
5. 샘플의 CSS와 JS를 기반으로 슬라이드 HTML 콘텐츠만 교체하여 새 프레젠테이션을 작성한다
6. 작업 디렉토리에 `.html` 파일로 저장한다
7. `open` 명령으로 브라우저에서 확인한다

샘플을 읽는 것이 핵심이다. CSS와 JS 코드를 직접 작성하지 말고, 샘플에서 가져와서 필요한 부분만 수정한다. 이렇게 하면 검증된 전환 애니메이션, 이벤트 처리, 레이아웃 시스템을 그대로 활용할 수 있다.

## 핵심 원칙

- **단일 파일**: 모든 CSS, JS가 HTML 안에 인라인. 별도 파일 없음
- **GPU 가속**: 전환 애니메이션은 `transform`과 `opacity`만 사용. Layout/Paint 유발 속성 금지
- **뷰포트 피팅**: 모든 슬라이드가 스크롤 없이 화면에 맞아야 함
- **접근성**: `role`, `aria-roledescription`, `aria-label`, `aria-hidden`, `aria-live` 속성 포함
- **`prefers-reduced-motion`**: 모션 감소 설정 시 전환 시간 단축

## 언어 대응

대화 컨텍스트에서 사용자 언어를 감지하여 적용한다.

| 항목 | 한국어 | 일본어 | 영어 |
|------|--------|--------|------|
| `<html lang>` | `ko` | `ja` | `en` |
| `--font-sans` 첫 번째 | `Noto Sans KR` | `Noto Sans JP` | `Inter` |
| ARIA 라벨 | 슬라이드, 이전/다음 | スライド, 前/次 | Slide, Previous/Next |
| 카운터 형식 | `1 / 10` | `1 / 10` | `1 / 10` |

Google Fonts `<link>` 태그에는 사용하는 폰트만 포함한다. 항상 `Inter`(라틴 텍스트용)와 `JetBrains Mono`(코드용)를 포함하고, 언어에 맞는 Noto Sans 변형을 추가한다.

**중요: Google Fonts를 비렌더블로킹으로 로드한다.** `rel="stylesheet"`만 쓰면 외부 CSS가 렌더를 블로킹하여 처음 열 때 빈 화면이 발생한다. 반드시 `media="print" onload="this.media='all'"` 패턴을 사용하고, `<noscript>` 폴백을 포함한다:

```html
<link href="https://fonts.googleapis.com/css2?..." rel="stylesheet" media="print" onload="this.media='all'">
<noscript><link href="https://fonts.googleapis.com/css2?..." rel="stylesheet"></noscript>
```

## CSS 디자인 시스템

기본 다크 테마의 CSS 변수 (사용자가 다른 색상을 요청하면 `--accent` 등을 변경):

```
--bg: #000000          --accent: #38bd84        --accent-dim: rgba(56,189,132,0.15)
--accent-glow: rgba(56,189,132,0.3)             --text-primary: #e8e8e8
--text-secondary: #888  --text-dim: #555         --border: #1a1a1a
```

타이포그래피는 `clamp()` 기반 반응형:
```
--title-size: clamp(2rem, 5vw, 4rem)     --h2-size: clamp(1.4rem, 3vw, 2.4rem)
--body-size: clamp(0.9rem, 1.4vw, 1.15rem)  --small-size: clamp(0.75rem, 1vw, 0.9rem)
```

배경 오버레이: scanline(`::before`) + grid(`::after`) 패턴을 슬라이드에 적용하여 질감을 준다.

## 슬라이드 타입 카탈로그

### 1. 타이틀 (`theme-title`)
```html
<section class="slide theme-title active" id="slide-1" role="group" aria-roledescription="슬라이드" aria-label="타이틀">
  <div class="terminal-badge">주제 키워드</div>
  <h1>제목 <span class="accent">강조어</span></h1>
  <div class="accent-line"></div>
  <p class="subtitle">부제목 텍스트</p>
</section>
```
**중요: 첫 슬라이드(slide-1)에는 반드시 `active` 클래스를 HTML에 직접 포함한다.** JS 실행 전에도 첫 슬라이드가 즉시 표시되어야 빈 화면 현상이 방지된다. `initFromHash()`가 동기적으로 재설정하므로 동작에 영향 없음.
h1에 gradient 텍스트 효과 적용 (`background-clip: text`).

### 2. 콘텐츠 (`theme-content`)
```html
<section class="slide theme-content" id="slide-N" ...>
  <h2>섹션 제목</h2>
  <ul>
    <li class="auto-fragment" style="--i:0">항목 1</li>
    <li class="auto-fragment" style="--i:1">항목 2</li>
    <!-- 최대 5개 -->
  </ul>
</section>
```

### 3. 2열 레이아웃 (`theme-content` + `two-col`)
```html
<section class="slide theme-content" id="slide-N" ...>
  <h2>비교/분류 제목</h2>
  <div class="two-col">
    <div>
      <div class="col-label">좌측 레이블</div>
      <ul>
        <li class="auto-fragment" style="--i:0">...</li>
        <li class="auto-fragment" style="--i:1">...</li>
        <li class="auto-fragment" style="--i:2">...</li>
      </ul>
    </div>
    <div>
      <div class="col-label">우측 레이블</div>
      <ul>
        <li class="auto-fragment" style="--i:3">...</li>
        <li class="auto-fragment" style="--i:4">...</li>
        <li class="auto-fragment" style="--i:5">...</li>
      </ul>
    </div>
  </div>
</section>
```
`--i` 인덱스를 좌측→우측 순으로 연속 부여한다.

### 4. 코드 (`theme-code`)
```html
<section class="slide theme-code" id="slide-N" ...>
  <h2>코드 제목</h2>
  <div class="terminal-window">
    <div class="terminal-bar">
      <span class="terminal-dot"></span>
      <span class="terminal-dot"></span>
      <span class="terminal-dot"></span>
    </div>
    <pre class="code-block"><code>첫 번째 줄</code>
<code class="highlight">강조 줄</code>
<code>일반 줄</code></pre>
  </div>
</section>
```
구문 강조 클래스: `.kw`(키워드/파란색), `.str`(문자열/주황), `.cmt`(주석/초록), `.num`(숫자/연두).
코드는 최대 8~10줄. `<code>` 태그마다 한 줄. CSS counter로 줄번호 자동 생성.

### 5. 마무리 (`theme-end`)
```html
<section class="slide theme-end" id="slide-N" ...>
  <div class="terminal-badge">fin</div>
  <h1>감사합니다</h1>
  <div class="accent-line"></div>
  <p class="subtitle">마무리 메시지</p>
</section>
```
h1에 `pulseGlow` 애니메이션 적용.

## auto-fragment 규칙

- 리스트 `<li>` 요소에 `class="auto-fragment"` + `style="--i:N"` 추가
- N은 0부터 시작, 슬라이드 내에서 등장 순서대로 증가
- 2열 레이아웃: 좌측 열 0,1,2 → 우측 열 3,4,5
- CSS `.slide.active .auto-fragment`가 자동으로 stagger 애니메이션 트리거
- 슬라이드 재진입 시 `.active` 클래스 재추가로 애니메이션 자동 재실행

## 콘텐츠 밀도

뷰포트에 맞추는 것이 최우선. 스크롤이 발생하면 안 된다.

- 리스트: 슬라이드당 **최대 5개** 항목
- 텍스트 단락: 2~3줄 이내
- 코드: **최대 8~10줄**
- 내용이 많으면 여러 슬라이드로 분할

## 슬라이드 수 가이드

- 사용자가 지정하면 그대로 따른다
- 미지정 시 **7~12장**: 타이틀(1) + 본론(5~9) + 마무리(1)
- 주제의 깊이에 따라 자연스럽게 조절

## JS 기능: 키보드 네비게이션

샘플의 `setupKeyboard()` 함수에 F키 풀스크린과 D키 테마 전환이 포함되어 있다.
D/F 키는 `e.code` (물리적 키 위치) 기반으로 처리하여 한글(ㅇ/ㄹ), 일본어 IME 상태에서도 동작한다. switch문 밖에서 `e.code === 'KeyD'`, `e.code === 'KeyF'`로 분기한다.

전체 키 매핑:
| 키 | 동작 |
|----|------|
| →, ↓, Space, PageDown | 다음 슬라이드 |
| ←, ↑, PageUp | 이전 슬라이드 |
| Home | 첫 슬라이드 |
| End | 마지막 슬라이드 |
| F | 풀스크린 토글 |
| D | 다크/라이트 모드 전환 |

## 다크/라이트 모드 전환

샘플에 이미 구현되어 있다. 핵심 구조:

- **CSS**: `[data-theme="light"]` 선택자로 라이트 모드 변수 오버라이드. `--accent` 색상은 유지하고 `--bg`, `--text-*`, `--border`만 변경
- **토글 버튼**: 카운터 옆에 원형 버튼 (달/해 아이콘). `<button class="theme-toggle" id="themeToggle">`
- **JS**: `toggleTheme()` 함수가 `<html>`의 `data-theme` 속성을 토글하고 `localStorage`에 저장
- **D키**: `setupKeyboard()`에서 `case 'd': case 'D':` → `toggleTheme()` 호출

라이트 모드에서 추가로 오버라이드해야 하는 요소:
- `.slide::before` (scanline) — `rgba(0,0,0,0.01)` 계열로 변경
- `.slide::after` (grid) — accent 색상의 투명도를 0.04로 높임
- `.terminal-window` 배경 — `#fafafa`
- `.terminal-bar` 배경 — `#eee`
- `.slide-controls button` 배경 — `rgba(0,0,0,0.04)`
- `.theme-toggle` 배경 — `rgba(0,0,0,0.04)`

## 주의사항

샘플 코드에는 이미 올바른 패턴이 구현되어 있다. 샘플을 수정할 때 다음 사항을 깨뜨리지 않도록 주의한다:

- **Forced reflow**: `void to.offsetHeight` — transition 시작 위치 설정 후 반드시 호출. 없으면 애니메이션 미발생
- **인라인 스타일 specificity**: `.active` 추가 시 `style.transform`과 `style.opacity`도 명시 설정. 클래스만으로는 인라인 스타일을 덮어쓸 수 없음
- **cleanup**: 전환 완료 후 인라인 스타일과 `will-change` 반드시 해제
- **`isAnimating` 잠금**: `transitionend` + 700ms safety timer로 이중 보호
- **Space 키**: `e.preventDefault()` 없으면 페이지 스크롤 발생
- **`touchmove`**: `{ passive: false }` 필수

기술적 세부사항이 필요하면 `references/guide.md`를 참조한다.

## 참고 리소스

- **`references/guide.md`** — DOM 구조, GPU 가속, fragment, 네비게이션, URL hash, 브라우저 호환성 등 상세 기술 가이드. 구현 중 기술적 의문이 생기면 이 파일을 참조한다.
- **`references/sample-presentation.html`** — 슬라이드 모드 10장짜리 완전한 동작 샘플. 이 파일의 CSS와 JS를 기반으로 슬라이드 HTML 콘텐츠만 교체하여 새 프레젠테이션을 만든다. 반드시 먼저 읽어야 한다.
- **`references/sample-scroll.html`** — 스크롤 모드 8장짜리 완전한 동작 샘플. 14가지 템플릿 CSS와 SlidePresentation JS 클래스를 포함한다. 스크롤 모드 프레젠테이션을 만들 때 반드시 먼저 읽어야 한다.

---

# 스크롤 모드

수직 scroll-snap 기반 프레젠테이션. IntersectionObserver로 슬라이드 가시성을 감지하여 `.reveal` 등장 애니메이션을 트리거한다.

## 스크롤 모드 워크플로우

1. `references/sample-scroll.html`을 읽어 CSS/JS 전체 구조를 파악한다
2. 사용자의 주제, 내용, 슬라이드 수 요구사항을 정리한다
3. 대화 컨텍스트에서 사용자 언어를 감지한다
4. 슬라이드 구성을 설계한다 (템플릿 타입별 배치)
5. 샘플의 CSS와 JS를 기반으로 슬라이드 HTML 콘텐츠만 교체하여 새 프레젠테이션을 작성한다
6. 작업 디렉토리에 `.html` 파일로 저장한다
7. `open` 명령으로 브라우저에서 확인한다

샘플을 읽는 것이 핵심이다. CSS와 JS 코드를 직접 작성하지 말고, 샘플에서 가져와서 필요한 부분만 수정한다.

## 스크롤 모드 핵심 원칙

- **단일 파일**: 모든 CSS, JS가 HTML 안에 인라인
- **수직 scroll-snap**: `html { scroll-snap-type: y mandatory; }`, 각 슬라이드 `scroll-snap-align: start`
- **IntersectionObserver**: 슬라이드가 50% 이상 보이면 `.visible` 클래스 추가 → `.reveal` 애니메이션 트리거
- **뷰포트 피팅**: 각 슬라이드 `100vh` (`100dvh`), 스크롤은 슬라이드 간 이동에만 사용
- **`prefers-reduced-motion`**: 모션 감소 시 `.reveal`은 opacity만 전환, `scroll-behavior: auto`

## 스크롤 모드 슬라이드 타입 카탈로그

### 1. 메인 타이틀 (`slide-title`)
```html
<section class="slide slide-title">
  <div class="slide-content">
    <div class="terminal-badge reveal">키워드</div>
    <div class="big-answer reveal">
      <span class="accent">강조어</span><br>제목
    </div>
    <div class="sub reveal">// 부제목</div>
  </div>
</section>
```

### 2. 섹션 전환 (`slide-transition`)
```html
<section class="slide slide-transition">
  <div class="slide-content">
    <div class="terminal-badge reveal">배지</div>
    <div class="big-title reveal"><span class="accent">강조</span> 제목</div>
    <div class="sub reveal">// 부제목</div>
  </div>
</section>
```

### 3. 기능/타겟 목록 (`slide-features`)
```html
<section class="slide slide-features">
  <div class="slide-content">
    <div class="terminal-badge reveal">배지</div>
    <h2 class="section-title reveal" style="font-size: var(--h2-size);">제목</h2>
    <ul class="target-list">
      <li class="reveal"><span class="num">01</span> 항목 1</li>
      <li class="reveal"><span class="num">02</span> 항목 2</li>
      <li class="reveal"><span class="num">03</span> 항목 3</li>
    </ul>
  </div>
</section>
```

### 4. 개념 다이어그램 (`slide-concept`)
스타일 A — 화살표 연결 박스:
```html
<section class="slide slide-concept">
  <div class="slide-content">
    <div class="terminal-badge reveal">배지</div>
    <h2 class="section-title reveal" style="font-size: var(--h2-size);">제목</h2>
    <div class="diagram reveal">
      <div class="diagram-box">
        <div class="box-label">레이블</div>
        <div class="box-desc">설명</div>
      </div>
      <div class="diagram-arrow">&rarr;</div>
      <div class="diagram-box output">
        <div class="box-label">결과</div>
        <div class="box-desc">설명</div>
      </div>
    </div>
  </div>
</section>
```
`.output` 클래스 추가 시 accent 테두리 + `pulseGlow` 애니메이션.

스타일 B — 카드 흐름:
```html
<div class="concept-flow reveal">
  <div class="concept-card">
    <div class="card-label">레이블</div>
    <div class="card-desc">설명</div>
  </div>
  <div class="concept-arrow">&rarr;</div>
  <div class="concept-card">...</div>
</div>
```

### 5. 단계 플로우 (`slide-steps`)
```html
<section class="slide slide-steps">
  <div class="slide-content">
    <div class="terminal-badge reveal">배지</div>
    <h2 class="section-title reveal" style="font-size: var(--h2-size);">제목</h2>
    <div class="step-flow reveal">
      <div class="step-item active">
        <span class="step-num">01</span>
        단계 1
      </div>
      <span class="step-arrow">&rarr;</span>
      <div class="step-item">
        <span class="step-num">02</span>
        단계 2
      </div>
      <span class="step-arrow">&rarr;</span>
      <div class="step-item">
        <span class="step-num">03</span>
        단계 3
      </div>
    </div>
  </div>
</section>
```
`.active` 클래스로 현재 단계를 강조한다.

### 6. 2열 비교 (`slide-compare`)
```html
<section class="slide slide-compare">
  <div class="slide-content">
    <div class="terminal-badge reveal">배지</div>
    <h2 class="section-title reveal" style="font-size: var(--h2-size);">제목</h2>
    <div class="compare-grid">
      <div class="compare-card reveal">
        <div class="card-header">좌측 제목</div>
        <ul class="compare-list">
          <li><span class="bullet">&gt;</span> 항목</li>
        </ul>
      </div>
      <div class="compare-card reveal">
        <div class="card-header">우측 제목</div>
        <ul class="compare-list">
          <li><span class="bullet">&gt;</span> 항목</li>
        </ul>
      </div>
    </div>
  </div>
</section>
```

### 7. 구조 다이어그램 (`slide-diagram`)
```html
<section class="slide slide-diagram">
  <div class="slide-content">
    <div class="terminal-badge reveal">배지</div>
    <h2 class="section-title reveal" style="font-size: var(--h2-size);">제목</h2>
    <div class="agent-diagram reveal">
      <div class="main-agent">메인 노드</div>
      <div class="arrows-row">↓ ↓ ↓</div>
      <div class="sub-agents">
        <div class="sub-agent">노드 A</div>
        <div class="sub-agent">노드 B</div>
        <div class="sub-agent">노드 C</div>
      </div>
    </div>
  </div>
</section>
```

### 8. Before/After (`slide-before-after`)
```html
<section class="slide slide-before-after">
  <div class="slide-content">
    <div class="terminal-badge reveal">before</div>
    <h2 class="section-title reveal" style="font-size: var(--h2-size);">제목</h2>
    <div class="comparison-card reveal">
      <div class="card-title">// 문제점</div>
      <ul class="pain-list">
        <li><span class="x">✕</span> 문제 1</li>
        <li><span class="x">✕</span> 문제 2</li>
      </ul>
    </div>
  </div>
</section>
```

### 9. 터미널 (`slide-terminal`)
```html
<section class="slide slide-terminal">
  <div class="slide-content">
    <div class="terminal-badge reveal">배지</div>
    <h2 class="section-title reveal" style="font-size: var(--h2-size);">제목</h2>
    <div class="terminal-window reveal">
      <div class="terminal-bar">
        <div class="terminal-dot" style="background: #ff5f57;"></div>
        <div class="terminal-dot" style="background: #febc2e;"></div>
        <div class="terminal-dot" style="background: #28c840;"></div>
      </div>
      <div class="terminal-body">
        <div class="prompt-line">
          <span class="prompt">&gt; </span>
          <span class="cmd">명령어</span>
        </div>
        <div class="prompt-line success">&#10003; 완료</div>
        <span class="cursor-blink"></span>
      </div>
    </div>
  </div>
</section>
```

### 10. 결과 카드 (`slide-result`)
```html
<section class="slide slide-result">
  <div class="slide-content">
    <div class="terminal-badge reveal">배지</div>
    <h2 class="section-title reveal" style="font-size: var(--h2-size);">제목</h2>
    <div class="result-card reveal">
      <ul class="result-list">
        <li><span class="check">&#10003;</span> 결과 항목 1</li>
        <li><span class="check">&#10003;</span> 결과 항목 2</li>
      </ul>
    </div>
  </div>
</section>
```

### 11. 프롬프트/질문 (`slide-prompt`)
```html
<section class="slide slide-prompt">
  <div class="slide-content">
    <div class="terminal-badge reveal">배지</div>
    <h2 class="section-title reveal" style="font-size: var(--h2-size);">제목</h2>
    <div class="prompt-card reveal">
      <p class="card-text">질문이나 프롬프트 내용</p>
    </div>
  </div>
</section>
```

### 12. 요약 (`slide-summary`)
```html
<section class="slide slide-summary">
  <div class="slide-content">
    <div class="terminal-badge reveal">정리</div>
    <h2 class="section-title reveal" style="font-size: var(--h2-size);">제목</h2>
    <div class="tag-row reveal">
      <span class="tag-pill">키워드 1</span>
      <span class="tag-pill">키워드 2</span>
    </div>
    <p class="summary-text reveal">요약 메시지</p>
  </div>
</section>
```

### 13. 장점/특징 카드 (`slide-benefits`)
```html
<section class="slide slide-benefits">
  <div class="slide-content">
    <div class="terminal-badge reveal">배지</div>
    <h2 class="section-title reveal" style="font-size: var(--h2-size);">제목</h2>
    <div class="benefit-list">
      <div class="benefit-card reveal">
        <div class="benefit-icon">아이콘</div>
        <div class="benefit-text">
          <div class="benefit-title">제목</div>
          <div class="benefit-desc">설명</div>
        </div>
      </div>
    </div>
  </div>
</section>
```

### 14. CTA 그리드 (`slide-cta`)
```html
<section class="slide slide-cta">
  <div class="slide-content">
    <div class="terminal-badge reveal">배지</div>
    <h2 class="section-title reveal" style="font-size: var(--h2-size);">제목</h2>
    <div class="cta-grid">
      <div class="cta-card reveal">
        <div class="cta-icon">아이콘</div>
        <div class="cta-label">레이블</div>
      </div>
      <!-- 최대 3개 -->
    </div>
    <p class="cta-msg reveal">마무리 메시지</p>
  </div>
</section>
```

## .reveal 규칙

- `.slide-content` 직접 자식에 `class="reveal"` 추가
- IntersectionObserver가 슬라이드 50% 진입 시 `.visible` 클래스 추가 → `.reveal` 요소에 fade-in + slide-up 애니메이션
- nth-child 기반 stagger delay: 1번째 0.1s, 2번째 0.2s, ..., 6번째 0.6s
- `.slide-content` 안의 직접 자식만 nth-child로 카운트됨

## 스크롤 모드 슬라이드 수 가이드

- 사용자가 지정하면 그대로 따른다
- 단독 프레젠테이션: **7~15장**
- **Round 시스템**: 시리즈 콘텐츠는 8~12장/라운드로 분할하여 별도 HTML 파일로 생성. 파일명: `slides-round1.html`, `slides-round2.html` 등

## 스크롤 모드 JS: SlidePresentation 클래스

샘플의 `SlidePresentation` 클래스에 다음 기능이 포함되어 있다:

| 메서드 | 기능 |
|--------|------|
| `createNavDots()` | 네비게이션 닷 동적 생성 |
| `setupIntersectionObserver()` | 슬라이드 가시성 감지 → `.visible` 클래스 |
| `setupKeyboardNav()` | ↑↓←→ Space 네비게이션 + D/F 키 |
| `setupScrollTracking()` | 프로그레스 바 업데이트 |
| `setupThemeToggle()` | 테마 전환 버튼 이벤트 |
| `initTheme()` | localStorage에서 테마 복원 |
| `toggleTheme()` | 다크/라이트 모드 전환 |
| `goToSlide(index)` | `scrollIntoView({ behavior: 'smooth' })` |
| `toggleFullscreen()` | Fullscreen API |

D/F 키는 `e.code` 기반으로 처리하여 한글/일본어 IME에서도 동작한다.

## 스크롤 모드 주의사항

- CSS와 JS를 직접 작성하지 말고 `sample-scroll.html`에서 가져온다
- `.reveal` 클래스는 `.slide-content`의 직접 자식에만 부여한다
- 내부 리스트 항목(li)에는 `.reveal`을 개별 부여하여 stagger 효과를 준다
- 각 슬라이드는 `100vh`(`100dvh`)에 맞아야 한다. 콘텐츠가 넘치면 분할한다
- `scroll-snap-type: y mandatory`는 `<html>` 요소에 설정한다
- Space 키 `e.preventDefault()` 필수 (페이지 스크롤 방지)
