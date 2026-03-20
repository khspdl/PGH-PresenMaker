# 순수 HTML/CSS/JS 슬라이드 프레젠테이션 구현 가이드

> 외부 라이브러리 없이 단일 HTML 파일로 프레젠테이션을 구현하기 위한 조사 결과 및 권장 아키텍처 정리.
> 참고 구현체: `sample-presentation.html`

---

## 1. 권장 아키텍처

### DOM 구조

```html
<main class="presentation" role="application" aria-roledescription="슬라이드 프레젠테이션">
  <div class="slides-viewport">
    <section class="slide" id="slide-1" role="group"
             aria-roledescription="슬라이드" aria-label="타이틀">
      <!-- 슬라이드 콘텐츠 -->
    </section>
    <!-- 추가 슬라이드... -->
  </div>

  <nav class="slide-controls" aria-label="슬라이드 내비게이션">
    <button class="prev" aria-label="이전 슬라이드">←</button>
    <button class="next" aria-label="다음 슬라이드">→</button>
  </nav>

  <div class="progress-container"><div class="progress-bar" id="progressBar"></div></div>
  <div class="slide-counter" id="slideCounter" role="status" aria-live="polite">1 / N</div>
</main>
```

**핵심 포인트:**
- 모든 슬라이드를 동일한 `position: absolute` 스택 구조로 배치
- `role="application"` + `aria-roledescription` 으로 접근성 시맨틱 제공
- `aria-live="polite"` 카운터로 스크린리더에 슬라이드 변경 알림

### 16:9 레이아웃 — 레터박스/필러박스 자동 처리

```css
.slides-viewport {
  width: 100vw;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
}

.slide {
  position: absolute;
  width: min(100vw, calc(100vh * 16 / 9));
  height: min(100vh, calc(100vw * 9 / 16));
  aspect-ratio: 16 / 9;
}
```

`min()` 함수로 가로/세로 어느 방향으로 화면이 좁아져도 16:9 비율을 자동 유지.
별도 JavaScript 없이 순수 CSS로 처리 가능.

### 반응형 타이포그래피 — `clamp()`

```css
.slide h1 { font-size: clamp(1.8rem, 1rem + 4vw, 3.5rem); }
.slide h2 { font-size: clamp(1.4rem, 0.8rem + 3vw, 2.5rem); }
.slide p, .slide li { font-size: clamp(0.9rem, 0.5rem + 1.5vw, 1.5rem); }
```

- 최솟값 · 유동값(vw 기반) · 최댓값을 하나의 선언으로 처리
- `rem` 기반 최솟값을 유지하므로 브라우저 줌 접근성 보장

### JS 상태 구조

```javascript
const slides = document.querySelectorAll('.slide');
const totalSlides = slides.length;
let currentSlide = 0;   // 현재 슬라이드 인덱스 (0-based)
let currentStep  = 0;   // 현재 fragment 단계
let isAnimating  = false; // 전환 중 입력 잠금
```

---

## 2. 주제별 Best Practice

### 2-1. 슬라이드 전환 애니메이션 (GPU 가속)

**핵심 원칙:** `transform` + `opacity` 만 애니메이션. Layout · Paint를 유발하는 `width`, `height`, `top`, `left` 등은 사용 금지.

```css
.slide {
  transform: translate3d(100%, 0, 0); /* 초기: 오른쪽 바깥 */
  opacity: 0;
  transition: transform 0.6s cubic-bezier(0.25, 0.46, 0.45, 0.94),
              opacity 0.6s ease;
}
.slide.active    { transform: translate3d(0, 0, 0); opacity: 1; }
.slide.exit-left { transform: translate3d(-100%, 0, 0); opacity: 0; }
```

**`will-change` 사용법:**
```javascript
// 전환 직전에만 적용 — 항상 켜두면 VRAM 낭비
to.style.willChange = 'transform, opacity';
from.style.willChange = 'transform, opacity';
// cleanup() 에서 반드시 해제
slide.style.willChange = 'auto';
```

**Forced Reflow (강제 리플로우):**
```javascript
to.style.transition = 'none';
to.style.transform = 'translate3d(100%, 0, 0)'; // 시작 위치 설정
void to.offsetHeight;  // ← 브라우저가 스타일을 commit하게 강제
to.style.transition = ''; // transition 복원
to.classList.add('active'); // 이제 transition 실행됨
```

`void element.offsetHeight` 없이 `transition: none` → 즉시 `active` 추가 시 브라우저가 두 상태를 합쳐버려 애니메이션이 발생하지 않음.

**CSS Specificity 함정 (중요):**

인라인 스타일(`element.style.transform`)은 클래스 선택자(`.active { transform }`)보다 우선순위가 높음.
따라서 `.active` 추가만으로는 인라인 스타일을 덮어쓸 수 없다.

```javascript
// ❌ 잘못된 코드: .active의 transform이 무시됨
to.classList.add('active');

// ✅ 올바른 코드: 인라인 스타일로 명시적으로 설정
to.classList.add('active');
to.style.transform = 'translate3d(0, 0, 0)';
to.style.opacity = '1';
```

전환 완료 후 `cleanup()` 에서 인라인 스타일 제거:
```javascript
function cleanup(prevIndex) {
  const prev = slides[prevIndex];
  prev.classList.remove('exit-left');
  prev.style.transform = '';
  prev.style.opacity = '';
  prev.style.willChange = 'auto';
  // 현재(to) 슬라이드 인라인 스타일도 제거
  slides[currentSlide].style.transform = '';
  slides[currentSlide].style.opacity = '';
  slides[currentSlide].style.willChange = 'auto';
  isAnimating = false;
}
```

**`isAnimating` 잠금 패턴:**
```javascript
isAnimating = true;
to.addEventListener('transitionend', onEnd, { once: true });
// 안전장치: transitionend가 누락될 경우 대비
const safetyTimer = setTimeout(() => {
  to.removeEventListener('transitionend', onEnd);
  cleanup(prevSlide);
}, TRANSITION_DURATION + SAFETY_MARGIN); // 예: 600 + 100 = 700ms
```

### 2-2. Fragment 빌드 효과 (클릭 시 순차 등장)

```css
.fragment {
  opacity: 0;
  transform: translateY(20px);
  transition: opacity 0.5s ease, transform 0.5s ease;
}
.fragment.visible {
  opacity: 1;
  transform: translateY(0);
  transition-delay: calc(0.06s * var(--i, 0)); /* CSS 커스텀 속성으로 stagger */
}
```

```html
<li class="fragment" data-step="1" style="--i:0">항목 1</li>
<li class="fragment" data-step="2" style="--i:1">항목 2</li>
```

```javascript
// "다음" 키: fragment 먼저 소진 → 다음 슬라이드
function handleNext() {
  const maxStep = getMaxStep(slides[currentSlide]);
  if (currentStep < maxStep) {
    currentStep++;
    showFragmentsUpToStep(slides[currentSlide], currentStep);
  } else {
    goToSlide(currentSlide + 1, 'forward');
  }
}
```

### 2-3. 자동 순차 등장 효과 (auto-fragment)

클릭 없이 슬라이드 진입 시 자동으로 stagger 애니메이션:

```css
@keyframes fragmentIn {
  from { opacity: 0; transform: translateY(20px); }
  to   { opacity: 1; transform: translateY(0); }
}

.auto-fragment {
  opacity: 0;
  transform: translateY(20px);
}

/* .active 클래스가 붙는 순간 애니메이션 시작 */
.slide.active .auto-fragment {
  animation: fragmentIn 0.6s cubic-bezier(0.25, 0.46, 0.45, 0.94) forwards;
  animation-delay: calc(0.3s * var(--i, 0));
}
```

```html
<li class="auto-fragment" style="--i:0">항목 1</li>
<li class="auto-fragment" style="--i:1">항목 2</li>
```

슬라이드 재진입 시 `.active` 클래스 제거/재추가로 CSS animation이 자동 재실행되어 별도 JS 불필요.

### 2-4. 키보드 네비게이션

```javascript
document.addEventListener('keydown', (e) => {
  // input, textarea 내부 입력 무시
  const tag = e.target.tagName.toLowerCase();
  if (tag === 'input' || tag === 'textarea' || e.target.isContentEditable) return;

  switch (e.key) {
    case 'ArrowRight': case 'ArrowDown': case ' ': case 'PageDown':
      e.preventDefault(); handleNext(); break;
    case 'ArrowLeft': case 'ArrowUp': case 'PageUp':
      e.preventDefault(); handlePrev(); break;
    case 'Home': e.preventDefault(); goToSlide(0); break;
    case 'End':  e.preventDefault(); goToSlide(totalSlides - 1); break;
  }
});
```

### 2-5. 터치/스와이프 (Pointer Events 우선)

```javascript
if (window.PointerEvent) {
  // Pointer Events: 마우스, 터치, 펜 통합 처리
  container.addEventListener('pointerdown', e => onStart(e.clientX, e.clientY));
  container.addEventListener('pointerup',   e => onEnd(e.clientX, e.clientY));
} else {
  // 폴백: Touch Events
  container.addEventListener('touchstart', e => {
    const t = e.changedTouches[0];
    onStart(t.clientX, t.clientY);
  }, { passive: true });
  container.addEventListener('touchend', e => {
    const t = e.changedTouches[0];
    onEnd(t.clientX, t.clientY);
  });
}

// 수평 스와이프 시 페이지 스크롤 방지 — passive: false 필수
container.addEventListener('touchmove', e => {
  if (Math.abs(dx) > Math.abs(dy)) e.preventDefault();
}, { passive: false });
```

스와이프 판정 기준: 이동거리 > 50px, 시간 < 400ms, 수직 이동 < 80px

### 2-6. URL Hash 동기화

```javascript
// replaceState: 히스토리 스택 오염 없이 URL 갱신
history.replaceState({ slide: currentSlide }, '', `#slide-${currentSlide + 1}`);

// 외부에서 hash 직접 변경 시 대응
window.addEventListener('hashchange', () => {
  const match = location.hash.match(/^#slide-(\d+)$/);
  if (match) {
    const idx = parseInt(match[1], 10) - 1;
    if (idx >= 0 && idx < totalSlides && idx !== currentSlide) {
      goToSlide(idx);
    }
  }
});
```

`pushState` 대신 `replaceState`를 사용해야 브라우저 뒤로가기가 슬라이드 히스토리가 아닌 이전 페이지로 이동.

---

## 3. 주의사항 & 브라우저 호환성

### 3-1. CSS Specificity 함정 ★★★

**가장 흔한 버그.** 인라인 스타일은 클래스 선택자보다 우선순위가 높다.

```javascript
// ❌ 이 코드는 동작하지 않음
// JS로 to.style.transform = 'translate3d(100%,0,0)' 설정 후
// .active { transform: translate3d(0,0,0) } 추가해도
// 인라인 스타일이 CSS 클래스를 덮어버림
to.classList.add('active'); // transform이 여전히 100%에 머묾

// ✅ 인라인 스타일로 명시 설정해야 실제 transition 발생
to.style.transform = 'translate3d(0, 0, 0)';
to.style.opacity = '1';
```

이 버그 발생 시 증상: `transitionend` 미발생 → `isAnimating` 플래그가 timeout(700ms)까지 해제되지 않음 → 700ms간 조작 불가.

### 3-2. `will-change` 남용 주의

```css
/* ❌ 모든 슬라이드에 항상 적용 — VRAM 과도 사용 */
.slide { will-change: transform, opacity; }

/* ✅ 전환 직전에만 JS로 적용, 완료 후 해제 */
to.style.willChange = 'transform, opacity';
// ... 전환 완료 후
slide.style.willChange = 'auto';
```

### 3-3. `touchmove` passive 리스너

`e.preventDefault()`를 호출하려면 `{ passive: false }` 옵션 필수.
기본값이 `passive: true`로 변경된 브라우저(Chrome 51+)에서 passive 리스너에서 `preventDefault()`를 호출하면 무시되고 콘솔 경고 발생.

```javascript
// ❌ passive: true(기본값)에서는 preventDefault() 무시됨
container.addEventListener('touchmove', e => e.preventDefault());

// ✅ 명시적으로 passive: false 지정
container.addEventListener('touchmove', e => {
  if (수평스와이프) e.preventDefault();
}, { passive: false });
```

### 3-4. `transitionend` 누락 가능성

- 요소가 hidden 상태이거나 `display: none`이면 `transitionend` 미발생
- 빠른 연속 클릭 시 이전 transition이 도중에 취소되면 미발생
- 반드시 `setTimeout` 안전장치 함께 사용

```javascript
const safetyTimer = setTimeout(cleanup, 700);
to.addEventListener('transitionend', (e) => {
  if (e.propertyName !== 'transform') return; // 다중 속성 중 transform만 처리
  clearTimeout(safetyTimer);
  cleanup(prevSlide);
}, { once: true });
```

### 3-5. Space 키 스크롤 방지

Space 키는 기본적으로 페이지 스크롤 동작. `e.preventDefault()` 필수.

```javascript
case ' ':
  e.preventDefault(); // 없으면 body가 스크롤됨
  handleNext();
  break;
```

### 3-6. `aspect-ratio` 지원 범위

`aspect-ratio` 속성: Chrome 88+, Firefox 89+, Safari 15+.
구형 브라우저 지원이 필요한 경우 `padding-top: 56.25%` 핵(padding-top hack) 폴백 사용.

### 3-7. `min()` / `clamp()` 함수

`min()`, `max()`, `clamp()`: Chrome 79+, Firefox 75+, Safari 11.1+.
IE 미지원. IE 지원 불필요 시 안심하고 사용 가능.

### 3-8. CSS Custom Properties (`var()`) in `style` attribute

```html
<li style="--i:2">항목</li>
```

인라인 스타일에서 CSS 커스텀 속성 선언은 모든 모던 브라우저 지원.
`calc()` 내에서 `var()`와 조합 가능: `calc(0.3s * var(--i, 0))`.

### 3-9. `BroadcastChannel` (발표자 화면 연동, 심화)

여러 탭/창 간 슬라이드 동기화가 필요한 경우(발표자 뷰):

```javascript
const channel = new BroadcastChannel('slides');
// 발표자 창
channel.postMessage({ type: 'goto', slide: currentSlide });
// 프로젝터 창
channel.addEventListener('message', e => {
  if (e.data.type === 'goto') goToSlide(e.data.slide);
});
```

지원: Chrome 54+, Firefox 38+. Safari 15.4+.

---

## 4. 샘플 파일 (`sample-presentation.html`)

단일 HTML 파일로 위 가이드의 모든 기법을 구현한 참고 구현체.

### 슬라이드 구성

| 슬라이드 | ID | 테마 | 특징 |
|---------|-----|------|------|
| 1 | `slide-1` | `theme-title` | 타이틀 슬라이드 |
| 2 | `slide-2` | `theme-content` | Fragment 빌드 (클릭 시 항목 순차 등장) |
| 3 | `slide-3` | `theme-content` | Auto-fragment (진입 시 자동 순차 애니메이션) |
| 4 | `slide-4` | `theme-code` | 코드 블록 + 라인 하이라이트 |
| 5 | `slide-5` | `theme-end` | 마무리 슬라이드 |

### 구현된 기능 목록

- [x] 키보드 네비게이션 (Arrow, Space, PageUp/Down, Home, End)
- [x] 터치/스와이프 (Pointer Events + Touch Events 폴백)
- [x] 버튼 네비게이션
- [x] GPU 가속 슬라이드 전환 (translate3d + opacity)
- [x] Fragment 빌드 시스템 (`data-step` + CSS stagger)
- [x] Auto-fragment 자동 순차 애니메이션 (`@keyframes` + CSS stagger)
- [x] 16:9 비율 고정 (레터박스/필러박스 자동 처리)
- [x] `clamp()` 반응형 타이포그래피
- [x] URL hash 동기화 (`history.replaceState`)
- [x] 프로그레스 바
- [x] 슬라이드 카운터
- [x] 화자 노트 (`<aside class="notes">`, 기본 숨김)
- [x] 접근성 (`role`, `aria-*`, `aria-live`)
- [x] `isAnimating` 잠금 + `transitionend` 안전장치
