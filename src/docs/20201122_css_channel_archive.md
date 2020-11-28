# z_tricky_css Archive

# content-visibility: the new CSS property that boosts your rendering performance

**Improve initial load time by skipping the rendering of offscreen content.**

초기 렌더링 속도를 획기적으로 향상시킬 수 있는 CSS content-visibility: auto 속성이 Chrome 버전 85부터 지원됩니다.

예제 이미지로 미루어 보았을 때, “viewport를 벗어난 요소의 렌더링을 건너뛰는 기능” 으로 보여, 솔루션 내에 적용할 수 있는 부분은 많지 않아보입니다.
다만, 모바일 환경에서 많은 Measurement List를 그려야 할 때 유용하게 쓸 수 있을 것 같습니다 :)!

이 속성이 기존 Vitrual Scrolling 기법을 대체할 수 있을까요?
=> 2번째 문서 하단에 해당 내용이 명시되어 있습니다!

> Some great use cases for content-visibility: hidden are when
> implementing advanced virtual scrollers, and measuring layout.

Viewport 크기와 요소 사이즈로 렌더링 영역을 예측하고; 영역 밖의 요소를 아예 렌더하지 않는 전략이라 굉장히 흡사해 보입니다.

`contain-intrinsic-size` 라는 속성도 추가되었는데, virtual list에서 각 리스트 컴포넌트의 예측 크기를(3번째 글, API의 `approxItemHeight` prop과 비슷한) 제공해

렌더링 결과물의 크기를 미리 제공할 수 있습니다.

아래는 두 번째 글의 기존 레이아웃 숨김 기능과의 비교 내용입니다.

 `display: none` 은 엘리먼트의 렌더링 상태를 삭제하고 요소를 숨깁니다. 다시 보여줄 때 같은 내용을 담은 새로운 요소를 렌더링하는 것과 다름없는 비용이 발생합니다.

`visibility: hidden` 은 렌더링 상태를 유지하고 요소를 숨깁니다. 그러나, 요소를 document 및 서브 트리에서 삭제하지 않고 여전히 공간을 차지합니다. 화면 상에선 숨겨져 있지만, 필요할 때 렌더링 상태를 갱신할 수 있습니다.

`content-visibility: hidden`은 렌더링 상태를 유지하고 요소를 숨깁니다. 발생해야 할 변경사항이 있는 경우, 요소가 다시 표시될 때에만 발생합니다.

원문:

> Compare it to other common ways of hiding element’s contents:
> 
> display: none: hides the element and destroys its rendering state.
> This means unhiding the element is as expensive as rendering a new
> element with the same contents.
> 
> visibility: hidden: hides the element and keeps its rendering state.
> This doesn’t truly remove the element from the document, as it (and
> it’s subtree) still takes up geometric space on the page and can still
> be clicked on. It also updates the rendering state any time it is
> needed even when hidden.
> 
> content-visibility: hidden, on the other hand, hides the element while
> preserving its rendering state, so, if there are any changes that need
> to happen, they only happen when the element is shown again (i.e. the
> content-visibility: hidden property is removed).

모든 고객사가 Chrome 버전 85 이상을 사용한다고 확신할 수 없기 때문에, 지금은 Electron 등의 임베디드 환경에서만 제한적으로 활용될 것 같습니다. :sadpepe:

재밌네요!

https://developers.google.com/web/updates/2020/08/nic85#content-visibility

https://web.dev/content-visibility/

https://ionicframework.com/docs/api/virtual-scroll

# 관성 스크롤 (Inertia scrolling)

대부분의 모바일 기기에서 빼놓을 수 없는 스크롤 UX인, 관성 스크롤을 소개하는 글을 찾아봤습니다.

CtxSort **(드래그&드랍 라이브러리)** 구현할 때, 
브라우저의 스크롤 액션을 동적으로 제어하기 힘들어서(가끔 엘리먼트가 집혀있는데도 브라우저 스크롤이 동작해서 이상하게 움직임)
스크롤러를 직접 구현했었는데, 완전한 UX 제공을 위해서 관성 스크롤 구현을 필요로 합니다.

이미 여러 Smooth scroll 관련 라이브러리가 존재하나, CtxSort 통합시킬 수 있는 지는 미지수입니다. 
 
UX 개선 고려할 시간이 있다면 꼭 넣어보고 싶은 feature 이기도 해서 공유해봐요.

**우리가 알고 있는 대부분의 iOS / MacOS UX 제안/구현한 사람이, 방금 소개한 관성 스크롤을 만든 ’바스 오딩’이라고 합니다.**  

> **스티브 잡스 전기에 “한 디자이너(a designer)“라고만 등장하는 바람에 그 이름을 알 수가 없었다.**

라는 내용이 있는데, 흥미롭네요.

[https://news.appstory.co.kr/plan10657](https://news.appstory.co.kr/plan10657)  
[https://slownews.kr/65798](https://slownews.kr/65798)

# CSS 방법론

https://gomdoreepooh.github.io/notes/smacss-bem-oocss

저번 주에 요 주제로 얘기하다가, 문득 원우님이 ‘방법론’이라는 단어에 거부감을 표하신 게 기억이 났습니다.

이미 자연스럽게 행하고 있는 내용을 거추장스러운 미사여구 붙여가면서 띄워준 것 같아서…그렇지 않을까 싶습니다.

잘 쓰려고 하다 보면 아래 방법과 비슷하게 쓰게 된다~ 정도로 보고 넘어가도 될 것 같습니다.

## BEM

주요 컨텍스트 (Block) - wrapper-like component. 화면에 보여질 블록.
`header`

내부 요소 (Element) - icon, title, description, etc. . .
`header__logo`
`logo__image`

수식어 (Modifier) - 모양, 상태. active, disabled, color/size scheme
`header__logo-active`
`header__logo-lg`

## OOCSS

구조 (컴포넌트 모양: Box Model)과 스타일?(Color / etc..)을 분리
범용 컴포넌트 작성 시 유용하게 쓰입니다.

아래는 Bootstrap의 작명법입니다.

`btn (inline-block), btn-block (block)` [구조]
`btn-primary, btn-outline` [스타일]
`navbar (block relative) navbar-fixed (fixed)` [구조]
`navbar-inverted` [스타일]

## 다른 프레임워크는….어떻게…

**Bootstrap**

```html
<button class="btn btn-primary btn-active"></button>
```
Bootstrap의 작명법은 각 컴포넌트 클래스 간 충돌 방지를 위해 컴포넌트 접두사를 붙입니다. (btn- input- table-)
단일 클래스명으로 탐색이 가능하기 때문에 빠르게 동작합니다.
BEM의 장점과 같습니다. (Nested Selector (>, space) 사용하지 않음)

**Semantic UI**

“읽을 수 있는” CSS 클래스명을 사용하는 Semantic UI 같은 프레임워크도 있습니다.
```html
<button class="ui inverted primary button">Primary</button>
```
.ui.button.primary 형식으로 여러 클래스를 읽을 수 있는, 의미있는 단어로 구성하고 확장합니다.

**TailwindCSS**

모든 CSS 속성을 유틸리티 클래스처럼 사용하는 프레임워크도 있습니다.

https://tailwindcss.com/

```html
<input class="bg-white focus:outline-none focus:shadow-outline border border-gray-300 rounded-lg py-2 px-4 block w-full appearance-none leading-normal" type="email" placeholder="jane@example.com">
```

이게 좋은 건진 모르겠네요………….:0

**Element UI**

BEM과 유사한 작명법을 사용합니다.

# Icon Fonts!

[https://fontawesome.com/6](https://fontawesome.com/6)

최애 라이브러리 Font Awesome의 v6 사전 예약이 진행중입니다!
  
아이콘 라이브러리를 사용하면 좋은 점은 익히 아시겠지만..
일일이 .svg 파일을 추가하지 않아도 되고, 
폰트로 바로 불러쓸 수 있어 색상/크기 변경도 자유롭고 아무튼..
**생산성을 정말 정말 많이 많이 올려 줍니다.**

아이콘 폰트 라이브러리를 도입해 쓸 날이 올 진 모르겠지만 언젠가 쓴다면 … ,. .  

프론트엔드 팀원 암 발병률이 낮아지는 데에 결정적인 역할을 하리라 믿어 의심치 않습니다.

--아래 이미지는 암암리에 구한(?) 모 회사의 레이아웃인데,  
3번 이미지의 상단 메뉴를 4번처럼 리팩토링하는 데에 약 4시간 정도가 소요되었습니다.

내부에 야후 플래시게임 느낌 나는 아이콘 수백개를 모두 갈아치우는 작업이었는데,
거의 모든 상황에 알맞는 아이콘이 있어서 감동했던 기억이 나네요(2000개가 넘어요).

~~svg 크기 조정하다 지쳐서 공유합니다~~

# 하드웨어 가속, 웹앱 최적화

하드웨어 가속(translate3D) 속성을 이미지에 사용하면 메모리를 상당히 많이 차지하므로 조심히 써야 합니다 :0!

####  [하이브리드  모바일  웹앱을  위한  유용한  웹뷰  설정](https://blog.cornerstone.sktelecom.com/post/52040853408/%ED%95%98%EC%9D%B4%EB%B8%8C%EB%A6%AC%EB%93%9C-%EB%AA%A8%EB%B0%94%EC%9D%BC-%EC%9B%B9%EC%95%B1%EC%9D%84-%EC%9C%84%ED%95%9C-%EC%9C%A0%EC%9A%A9%ED%95%9C-%EC%9B%B9%EB%B7%B0-%EC%84%A4%EC%A0%95)

# Material Design 공식 문서 - 디자인 시스템 논의 중 인용

**[https://material.io/design/layout/spacing-methods.html#baseline-grid](https://material.io/design/layout/spacing-methods.html#baseline-grid)**  
Material Design layouts are visually balanced. Most measurements align to an 8dp grid applied, which aligns both spacing and the overall layout.  
Smaller components, such as icons and type, can align to a 4dp grid.n줄요약  

1.  Material design 은  `8dp`  단위로 설계됨 (dp === ios `pt` === 72ppi 의 8px). icon이나 쪼매난 컴포넌트는 4dp 단위로 쪼개도 됨
2.  n배(글에선 1.5배) 해상도 가진 기기 (아이폰, dpi 높은 기기들)에서 홀수 단위 렌더링할때 어렵게 느낌
3.  UI 일관성 유지 가능, 개발자가 매번 측정할 필요 없어서 좋음, 변수로 관리하기 좋음, 아무튼 좋음….

# CSS DPI 관련 문서

[https://spoqa.github.io/2012/07/06/pixel-and-point.html](https://spoqa.github.io/2012/07/06/pixel-and-point.html)

`CSS 12pt === 16px`  

하지만 웹 브라우져 상에서의 해상도는 약간 예외적입니다.

(아직까지는) 가장 보편적으로 사용되고 있는 웹 브라우저인 Microsoft의 Internet Explorer는 역시 자사제품답게 96 dpi의 해상도를 가지고 있습니다.

하지만 Google Chrome이나 Apple의 Safari가 쓰는 Webkit 엔진의 경우도 72 dpi가 아닌 96 dpi의 해상도를 기준으로 삼고 있습니다.

이는 CSS가 계획될 당시 96 dpi를 기준으로 만들어졌기 때문입니다.

# Shadow DOM

<Input /> 엘리먼트는 왜 텍스트가 width 보다 커져도 계산된 크기가 늘어나지 않을까?

[https://wit.nts-corp.com/2019/03/27/5552](https://wit.nts-corp.com/2019/03/27/5552)

Shadow DOM을 “DOM 내의 DOM”으로 생각할 수도 있지만, 원래의 DOM 트리에서 완전히 분리된 고유의 요소와 스타일을 가진 DOM 트리입니다. 

Shadow DOM은 웹 작성자가 사용하도록 최근에 지정되었지만, 사용자 에이전트에서 폼 요소와 같이 복잡한 구성요소를 만들고 스타일을 입히기 위해 수년 동안 사용되어 왔습니다.

더 깊게 파고들면, `<input>` 요소가 실제로 여러 작은 `<div>` 요소로 구성되어 트랙과 슬라이더를 자체적으로 제어하는 것을 볼 수 있습니다.

호스트 HTML 문서에는 단순한 `<input>` 요소가 노출되지만, 그 내부에는 DOM의 글로벌 범위에 포함되지 않는 HTML 요소와 스타일 구성 요소들이 있습니다.

# img 태그의 비밀

[https://stackoverflow.com/questions/2402761/is-img-element-block-level-or-inline-level](https://stackoverflow.com/questions/2402761/is-img-element-block-level-or-inline-level)

> * Note that browsers technically use `display: inline` (as seen in the developer tools) but they are giving special treatment to images. They still follow all traits of `inline-block`.

inline인 듯 inline 아닌 inline-block 같은 inline인 너 :thinking_face::thinking_face::thinking_face::thinking_face:

# Interlaced PNG

[https://blog.codinghorror.com/progressive-image-rendering/](https://blog.codinghorror.com/progressive-image-rendering/)  
[https://en.wikipedia.org/wiki/Interlacing_(bitmaps)](https://en.wikipedia.org/wiki/Interlacing_(bitmaps))

인터레이스된 이미지는 linear 하게 렌더링되지 않고, 저화질 -> 고화질로 픽셀을 채워 넣으며 렌더링됩니다.  

related to @Yejun Lee @frontend and also @_design

# CSS 레이아웃을 배웁시다

이 사이트에서는 모든 웹 사이트의 레이아웃에 사용되는 CSS의 기초를 다룹니다.  

이곳에서는 여러분이 선택자와 프로퍼티, 값이 무엇인지 알고 있다고 가정합니다.

그리고 레이아웃 작업이 여러분을 짜증 나게 만드는 작업일 수도 있겠지만 이미 한두 가지 레이아웃을 알고 있다고 가정합니다.

HTML과 CSS를 처음부터 배우고 싶다면 [이 튜토리얼](http://learn.shayhowe.com/html-css/)을 참고하세요.

그렇지 않다면 이 튜토리얼이 여러분의 다음 프로젝트에 도움될 수 있을지 알아봅시다.

[http://ko.learnlayout.com/](http://ko.learnlayout.com/) 

position attrs (static, absolute, relative, fixed, sticky[? 거의쓸일x])  
display attrs (block, inline, inline-block, flex, grid, table, etc…..)float clear (ie9 clearfix)  
box-sizing

# CSS Margin collapsing

[https://velog.io/@raram2/CSS-%EB%A7%88%EC%A7%84-%EC%83%81%EC%87%84Margin-collapsing-%EC%9B%90%EB%A6%AC-%EC%99%84%EB%B2%BD-%EC%9D%B4%ED%95%B4](https://velog.io/@raram2/CSS-%EB%A7%88%EC%A7%84-%EC%83%81%EC%87%84Margin-collapsing-%EC%9B%90%EB%A6%AC-%EC%99%84%EB%B2%BD-%EC%9D%B4%ED%95%B4)

[https://developer.mozilla.org/ko/docs/Web/CSS/CSS_Box_Model/Mastering_margin_collapsing](https://developer.mozilla.org/ko/docs/Web/CSS/CSS_Box_Model/Mastering_margin_collapsing)
