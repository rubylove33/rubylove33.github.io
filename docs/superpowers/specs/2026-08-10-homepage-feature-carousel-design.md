# 홈페이지 기능 섹션 — 슬라이드 레일 전환 및 스크린샷 갱신

- 작성일: 2026-08-10
- 대상 저장소: `rubylove33.github.io`
- 대상 파일: `index_kr.html`, `index.html`, `assets/`

## 배경

홈페이지의 앱 스크린샷 5장이 모두 2026-05-20 자 구버전이다. 그 사이 1.1.5·1.1.6 이
출시되어 화면이 실제로 바뀌었고(예: 홈 화면 세 번째 칩이 `🔥 1일` → `✓ 목표 성공!`),
쉬는 날 예외·타이머 리디자인 같은 신규 기능은 사이트에 전혀 드러나지 않는다.

사용자가 실기기(iPhone 16 Pro, 6.3인치)로 7개 화면을 새로 캡처하기로 했다. 기존
`duo-row`(2열 그리드) 구조는 타일 수가 짝수여야 해서 7개를 담지 못한다.

## 목표

1. 기능 섹션을 가로 슬라이드 레일로 바꿔 타일 개수 제약을 없앤다.
2. 스크린샷 8장(히어로 1 + 기능 7)을 현재 앱 화면으로 교체한다.
3. 1.1.5·1.1.6 신규 기능이 문구로 드러나게 한다.

## 범위 밖

- **Walk Clip(영상 공유) 카드** — 사용자 판단으로 이번 회차에서 제외. "운동 시작 전
  촬영 여부" 캡처(`KO/2_*`, `EN/2_*`)는 소셜미디어 홍보용으로만 쓰기로 했으므로
  사이트에는 넣지 않고 원본만 보관한다. 레일 구조상 나중에 `<article>` 하나만
  추가하면 되므로 후속 작업 부담이 없다.

문구 ③·⑥ 은 실제 캡처 화면과 어긋나 2026-08-11 에 수정했다. ③ 은 목표 화면이
"실제 걸은 시간"이 아니라 `총 목표 시간`·`운동 요일` 을 보여주기 때문이고, ⑥ 은
월간 화면의 쉬는 날이 `목표 달성` 과 구분된 별도 범례로 표시되기 때문이다.
- 철학·후원 섹션, 헤더/푸터, 새 소식·FAQ·약관 페이지는 건드리지 않는다.
- `docs/brand-guidelines.html` 은 대상이 아니다.

## 현재 구조

```
[히어로]   app_mockup-ko.png
[철학]     dino.png
[duo-row]  기록 중심의 트래킹 | 유연한 주간 목표
[duo-row]  의미 있는 통계     | 나만의 기록 아카이브
[후원]
```

CSS는 `style.css` 가 아니라 각 HTML 파일의 인라인 `<style>` 에 있고, `index_kr.html`
과 `index.html` 에 같은 내용이 중복된다. **모든 CSS 변경은 두 파일에 동일하게 적용해야
한다.** 사이트 전체에 `<script>` 는 0개다.

## 변경 후 구조

```
[히어로]      메인화면              ← 전체 폭 유지, 이미지만 교체
[철학]        dino                  ← 그대로
[주요 기능]   ‹ ①②③④⑤⑥⑦ ›       ← duo-row 2줄을 레일 1개로 대체
[후원]                              ← 그대로
```

`duo-row` 두 줄이 레일 하나로 축약되어 페이지 세로 길이가 현재보다 짧아진다.
nav 의 `<a href="#features">` 앵커는 레일 섹션이 이어받는다.

## 레일 구현

### 마크업

```html
<section class="tile bg-gray features" id="features">
    <h2>주요 기능.</h2>
    <div class="rail-wrap">
        <button class="rail-nav prev" type="button" aria-label="이전">‹</button>
        <div class="rail" id="featureRail">
            <article class="rail-card">
                <h3>남은 시간이<br>보이는 타이머.</h3>
                <p class="rail-lead">…</p>
                <div class="device"><img src="assets/feature-timer-ko.png" alt="…"></div>
            </article>
            <!-- ×7 -->
        </div>
        <button class="rail-nav next" type="button" aria-label="다음">›</button>
    </div>
</section>
```

### CSS

```css
.rail-wrap { position: relative; width: 100%; max-width: 1024px; margin: 0 auto; }
.rail {
    display: flex; gap: 20px;
    overflow-x: auto;
    scroll-snap-type: x mandatory;
    scroll-behavior: smooth;
    scroll-padding-inline: 24px;
    padding: 8px 24px 24px;
    scrollbar-width: none;
}
.rail::-webkit-scrollbar { display: none; }
.rail-card {
    flex: 0 0 clamp(260px, 72vw, 320px);
    scroll-snap-align: center;
    background: var(--tile-white);
    border-radius: 28px;
    padding: 32px 24px;
    display: flex; flex-direction: column; align-items: center;
}
.rail-nav {
    position: absolute; top: 50%; transform: translateY(-50%);
    width: 40px; height: 40px; border-radius: 50%;
    border: none; background: rgba(255,255,255,0.9);
    box-shadow: 0 2px 12px rgba(0,0,0,0.12);
    color: var(--ink); font-size: 20px; cursor: pointer;
    z-index: 2;
}
.rail-nav.prev { left: -4px; }
.rail-nav.next { right: -4px; }
@media (hover: none) { .rail-nav { display: none; } }
```

카드 폭 `clamp(260px, 72vw, 320px)` 의 의도:

- 모바일 — 1장 + 다음 장 일부가 보여 옆으로 더 있다는 것이 드러난다.
- 데스크톱(최대 1024px) — 3장이 보이고 오른쪽이 잘려 같은 신호를 준다.

터치 스와이프와 트랙패드 가로 스크롤은 브라우저 기본 동작으로 처리된다.

### JS (인라인, 약 10줄)

윈도우 마우스 사용자는 가로 스크롤 수단이 마땅치 않아 화살표 버튼이 필요하다.
외부 라이브러리 없이 파일 안에서 끝낸다.

```html
<script>
(function () {
    var rail = document.getElementById('featureRail');
    if (!rail) return;
    var step = function () { return rail.querySelector('.rail-card').offsetWidth + 20; };
    document.querySelector('.rail-nav.prev')
        .addEventListener('click', function () { rail.scrollBy({ left: -step(), behavior: 'smooth' }); });
    document.querySelector('.rail-nav.next')
        .addEventListener('click', function () { rail.scrollBy({ left: step(), behavior: 'smooth' }); });
})();
</script>
```

스크립트가 없거나 실패해도 스와이프·스크롤로 모든 카드에 접근할 수 있다(점진적 향상).

## 이미지 규격

전부 6.3인치(1206×2622)로 통일되므로 `.device img` 의 종횡비를 맞춰 크롭을 0으로 만든다.

```css
/* 변경 전 */ aspect-ratio: 207 / 448;   /* 1242×2688 */
/* 변경 후 */ aspect-ratio: 201 / 437;   /* 1206×2622 */
```

캡처 규칙:

- 기기 프레임·라운드 코너를 이미지에 넣지 않는다. `.device` 가 CSS로 그린다.
- 상태바를 포함한다(기존 이미지도 포함).
- 라이트 모드로 캡처한다.
- 한국어(`-ko`)·영어 두 벌을 각각 캡처한다.

## 카드 내용

| 순서 | 화면 | 파일 (KO / EN) | 상태 |
|---|---|---|---|
| 히어로 | 메인화면 | `app_mockup-ko.png` / `app_mockup.png` | 교체 |
| ① | 타이머 작동 | `feature-timer-ko.png` / `feature-timer.png` | 신규 |
| ② | 수동 기록 입력 | `feature-manual-ko.png` / `feature-manual.png` | 신규 |
| ③ | 주간 루틴 목표 설정 | `feature-goal-ko.png` / `feature-goal.png` | 교체 |
| ④ | 특정 쉬는 날 설정 | `feature-restday-ko.png` / `feature-restday.png` | 신규 |
| ⑤ | 주간 기록 | `feature-stats-ko.png` / `feature-stats.png` | 교체 |
| ⑥ | 월간 기록 | `feature-month-ko.png` / `feature-month.png` | 신규 |
| ⑦ | 템플릿 적용 사진 기록 | `feature-archive-ko.png` / `feature-archive.png` | 교체 |

`feature-track-ko.png` / `feature-track.png` 는 ①②가 흡수하므로 삭제한다. 단
아래 "이미지 대기 중 처리"에서 임시 참조로 쓰이므로, **삭제는 신규 캡처가 모두
도착해 `src` 를 교체한 뒤에 수행한다.**

섹션 제목은 한국어 `주요 기능.`, 영어 `Features.` 로 하고 `.tile h2` 스타일을 따른다.

### 문구

굵은 글씨가 `<h3>`, 아랫줄이 `.rail-lead` 다. `<br>` 위치까지 그대로 반영한다.

| | 한국어 | English |
|---|---|---|
| ① | **남은 시간이<br>보이는 타이머.**<br>목표까지 얼마 남았는지, 달성한 순간, 목표를 넘겨 걸은 시간까지 한눈에. | **A timer that shows<br>the finish line.**<br>Time left, the moment you hit your goal, and how far past it you walked. |
| ② | **깜빡했어도<br>괜찮아요.**<br>타이머를 못 켰다면 시작 시간까지 직접 입력해 남기면 됩니다. | **Forgot to<br>start it?**<br>Add the walk by hand — start time included. |
| ③ | **유연한<br>주간 목표.**<br>요일별로 다르게. 이번 주 총 목표 시간과 운동 요일을 한눈에 확인합니다. | **Flexible<br>weekly goals.**<br>A different target each day — with the week's total goal time and active days at a glance. |
| ④ | **오늘만<br>쉬어도 돼요.**<br>반복 목표는 그대로 두고 특정 날짜만 쉬는 날로. 통계와 달성 계산에도 반영됩니다. | **Rest just<br>for today.**<br>Mark one date as a rest day without touching your repeating schedule — stats follow along. |
| ⑤ | **이번 주,<br>지난 주.**<br>화살표로 거슬러 올라가도 그때 기록한 막대가 그대로 남아 있습니다. | **This week,<br>last week.**<br>Step back through the arrows and the bars you actually recorded are still there. |
| ⑥ | **한 달을<br>한 장으로.**<br>목표 달성·미달성·쉬는 날이 한 화면에. 쉬는 날도 계획의 일부로 표시됩니다. | **A month<br>at a glance.**<br>Achieved, missed, and rest days in one view — rest days shown as part of the plan. |
| ⑦ | **나만의<br>기록 아카이브.**<br>템플릿과 GIF 전부 무료. 그날의 기록을 사진으로 남겨보세요. | **Your own<br>archive.**<br>Every template and GIF is free. Keep the day as a picture. |

신규 기능이 드러나는 지점: ③ 의 "이번 주 총 목표와 실제 걸은 시간"(1.1.6 목표 화면
주간 합계), ④ 쉬는 날 예외 전체(1.1.6), ⑤ 의 "거슬러 올라가도 막대가 남아 있다"
(1.1.5 이전 주/달 조회 수정), ⑥ 의 "쉬는 날도 달성으로 센다"(1.1.5).

## 검증

- 두 HTML 파일을 `html.parser` 로 파싱해 미닫힘 태그가 없는지 확인한다.
- `assets/` 참조 경로가 모두 실재하는 파일인지 확인한다(`grep -o 'src="assets/[^"]*"'`).
- 브라우저에서 데스크톱 폭과 720px 이하 모바일 폭 양쪽을 확인한다.
- 화살표 버튼 클릭, 터치 스와이프, 트랙패드 가로 스크롤이 모두 동작하는지 확인한다.
- JS를 끈 상태에서도 스크롤로 7장 모두 접근 가능한지 확인한다.

## 이미지 대기 중 처리

캡처가 도착하기 전에는 레일 구조·CSS·문구까지 구현하고, 아직 없는 이미지는 기존
파일을 임시로 가리킨다. 캡처가 오면 `assets/` 에 넣고 `src` 만 교체한다.
