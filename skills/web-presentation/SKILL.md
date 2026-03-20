---
name: web-presentation
description: 순수 HTML, CSS, JavaScript로 웹 프레젠테이션(슬라이드)을 생성합니다. 외부 라이브러리 없이 단일 HTML 파일로 완성되며, GPU 가속 슬라이드 전환, 키보드/터치 네비게이션, 프로그레스 바, 반응형 디자인, 풀스크린 모드를 포함합니다. 사용자가 프레젠테이션, 슬라이드, 발표자료, PPT, 발표 등을 만들어달라고 요청할 때 이 스킬을 반드시 사용하세요. 'PPT 만들어줘', '슬라이드 생성', '발표자료 만들어줘', 'create a presentation', 'make slides about', 'プレゼン作って', 'スライド作って' 등 프레젠테이션 생성과 관련된 모든 요청에 반응합니다.
---

# 웹 프레젠테이션 생성기

순수 HTML + CSS + JavaScript만으로 단일 HTML 파일 프레젠테이션을 생성한다. 외부 라이브러리 의존성 없음 (Google Fonts CDN만 예외). GPU 가속 슬라이드 전환, 반응형 디자인, 접근성을 기본 지원한다.

## 워크플로우

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
<section class="slide theme-title" id="slide-1" role="group" aria-roledescription="슬라이드" aria-label="타이틀">
  <div class="terminal-badge">주제 키워드</div>
  <h1>제목 <span class="accent">강조어</span></h1>
  <div class="accent-line"></div>
  <p class="subtitle">부제목 텍스트</p>
</section>
```
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
- **`references/sample-presentation.html`** — 10장짜리 완전한 동작 샘플. 이 파일의 CSS와 JS를 기반으로 슬라이드 HTML 콘텐츠만 교체하여 새 프레젠테이션을 만든다. 반드시 먼저 읽어야 한다.
