# z_tricky_react Archive

# Deep-dive into React Hooks

**English version is included in the thread of this conversation.**
React Hook 구현 관련해서 찾아봤던 내용들 공유하여 드립니다.

**훅의 2가지 규칙**  

1.  매 렌더링 시점에 실행되는 훅의 개수는 동일해야 함
2.  React 컴포넌트 내에서만 훅을 실행할 수 있음

위 규칙이 생기게 된(제약조건이 생긴) 이유를 유추할 수 있도록 돕는 글 몇 가지가 있습니다.

**TL;DR (약간의 논리 도약이 있습니다. React 내부 설계와 살짝 틀릴 수는 있으나 설명에는 문제가 없어보여서 적어봐요..)**

1. 훅은 React의 렌더 함수를 실행할 수 있는:Render Context를 훅 실행 스코프로 묶는:**클로저**입니다.
2. 훅의 실행 결과는 독립적인(?) 배열에 저장되는데,  
setState() 또는 useEffect, memo, callback 등의 deps 변경으로 인해 렌더 함수가 실행될 때, 다음 렌더링 시점에 훅 생성 순서가 달라지면  
이상한 state를 갱신해버릴 수 있습니다.
3. 마찬가지로 useCallback / useEffect 등의 훅도, deps의 갱신을 감지하고 새로운 context를 바인딩하는 함수를 생성해야 하는데  
이 때, 기억하고 있던 본인의 순서 (hook 실행 순서)가 달라지면 이상한 훅에서 값을 가져오거나, 아예 가져오지 못할 수 있습니다.
위와 같은 현상을 막기 위해 React에서 2개의 제약 조건을 어길 경우, 즉시 런타임 에러를 반환합니다.

**1번 글(how-are-function-components-different-from-classes)은**클래스 컴포넌트의 단점인, this.props의 mutation으로 인해 발생할 수 있는 이슈와  이를 해결하기 위해 사용했던 패턴으로,  
this.render 함수에서 prop을 캡처하고(계속 클로저를 만들어) 모든 사이드 이펙트를 render() 메서드 안에서 처리하는 방식을 소개하고
함수형 컴포넌트는 어떤 방식으로 문제를 해결하는 지 알려 줍니다.

**2번 글은 위 내용을 함수형 컴포넌트 사용자의 시점에서 간략하게 정리한 글입니다.**

**3/4번 글은 React Hook의 동작 방식을 따라해보는 시간입니다!**읽어보시고 궁금하거나 잘못된 사항을 찾으시면 말씀해주세요

![https://overreacted.io/ko/how-are-function-components-different-from-classes/](https://overreacted.io/ko/how-are-function-components-different-from-classes/)  
[https://boxfoxs.tistory.com/395](https://boxfoxs.tistory.com/395)[https://www.awesomezero.com/development/reacthook/](https://www.awesomezero.com/development/reacthook/)  
[https://www.netlify.com/blog/2019/03/11/deep-dive-how-do-react-hooks-really-work/](https://www.netlify.com/blog/2019/03/11/deep-dive-how-do-react-hooks-really-work/)


# useMemo / useCallback

[https://rinae.dev/posts/review-when-to-usememo-and-usecallback](https://rinae.dev/posts/review-when-to-usememo-and-usecallback)  
[https://ideveloper2.dev/blog/2019-06-14--when-to-use-memo-and-use-callback/](https://ideveloper2.dev/blog/2019-06-14--when-to-use-memo-and-use-callback/)

두분 다 같은 원문을 읽고 적은 내용같습니다. 저도 아래와 같이 정리해봤는데 어떻게 생각하시나요?

1.  useMemo: 결과값을 저장할 때.
(실행결과가 무거울때,  **컴포넌트거나 컴포넌트거나 컴포넌트거나**)
3.  useCallback:  **함수 레퍼런스 자체를 저장(KingGodCalculationFnWithDispatchAndSetState)** 해야 할 때 **ex) 이벤트 핸들러 함수 메모이제이션!**
4.  짧고 빠르게 실행되는 변수/함수는 **꼭 컴포넌트 안에서 메모이제이션하지 않아도 됩니다**.
컴포넌트 밖으로 빼도 되고 유틸로 빼도 됩니다.

함수 자체를 어떤 값의 변경 시점마다 다시 선언해야 하는 경우를 생각해봤는데,  
**함수 내에서 Dispatch / SetState 로 직접 컴포넌트에 영향을 줘야 할 경우를 제외하고는**  
**모두 유틸리티 함수로 간주하고 스코프 밖으로 빼도 문제 없을 것 같다**는 생각이 들었습니다.
다소 파격적인 정의라 맞는 지는 모르겠지만 언뜻 보기에 되게 클리어해서 공유해봅니다.  
같이 생각해보면 좋을 주제 같아요!


# useEffect 이쁘게 쓰기

(대충 클린 코드 하면 된다는 글)  
커스텀 훅으로 목적에 따라 코드를 묶는 패턴을 선호하는 듯 합니다.
medium 구독할까 말까 맨날 고민하네요.
가끔 Limit 생겨도 볼 수 있는 글도 있고 아닌 것도 있던데, 기준이 뭔지 모르겠습니다.  

위는 원문, 아래는 번역문입니다.  
[https://medium.com/swlh/useeffect-4-tips-every-developer-should-know-54b188b14d9c](https://medium.com/swlh/useeffect-4-tips-every-developer-should-know-54b188b14d9c)  
[https://ui.toast.com/weekly-pick/ko_20200916/?fbclid=IwAR0n3Bta8PBD353lomxnnP89hYbk7RGrMSNynrsy89LYZ_VawNag2tsHz-M](https://ui.toast.com/weekly-pick/ko_20200916/?fbclid=IwAR0n3Bta8PBD353lomxnnP89hYbk7RGrMSNynrsy89LYZ_VawNag2tsHz-M)

# Why using the `**children**` prop makes `React.memo()` not work

첫 번째 글 제목입니다.  
`children` prop을 받는 컴포넌트를 메모이제이션하려다 데이고 쓴 글같습니다. (방금 저도 데임ㅇ.ㅇ)
두 번째 글(**Advanced memoization and effects in React**) 역시,
1번째 글에서 영감을 받아 쓴 것으로 보입니다.
글에서 Deep equality check를 지원하는 `react-fast-compare` 라이브러리 사용하는 방법을 제시하는데,  
`ContentsListItem` 처럼 많은 갯수가 렌더링되고, 각각 무거운 `ReactNode` 를 `children` 으로 받는 컴포넌트의 경우  
`shallowEqual` 을 사용하는 게 더 좋은 방법으로 보이고..따라서,  
**모든 자식 컴포넌트(ContentsListMarkerItem, etc…)에** `**memo**` **를 붙여주는 쪽으로 처리하려 합니다.**

[https://gist.github.com/slikts/e224b924612d53c1b61f359cfb962c06](https://gist.github.com/slikts/e224b924612d53c1b61f359cfb962c06)  
[https://gist.github.com/slikts/fd3768de1493419ed9506002b452fcdc](https://gist.github.com/slikts/fd3768de1493419ed9506002b452fcdc)Stackoverflow - useMemo on non-primitive value  
[https://stackoverflow.com/questions/53074551/when-should-you-not-use-react-memo](https://stackoverflow.com/questions/53074551/when-should-you-not-use-react-memo)ㄱㄱ

# Difference between UseSelector and MapStateToProps

I think a little different thing that is using  `useSelector`  for a "stable" instance of  `Content`  will not invoke re-render.

In the case of invoking re-render on every-tick is simply describe when we make(return) new object in every call of useSelector like below:
```ts
const badPractice: object = useSelector((s: T.State) => {
  printingContentId: s.Pages.Content.printingContentId,
  selectedGroupId: s.Pages.Content.selectedGroupId,
});
```
Also it can be resolved when you pass `Shallow Comparison` functions into second parameter:

```ts
import { shallowEqual } from 'react-redux';

const itWorksNice: object = useSelector((s: T.State) => {
  printingContentId: s.Pages.Content.printingContentId,
  selectedGroupId: s.Pages.Content.selectedGroupId,
}, shallowEqual);
```

Because  [@hyeokjoo](https://github.com/hyeokjoo)  already discuss about that  `UseSelector`  hook intended to get Single data from State.

Previously when we using Class Component, also we used  `connect`  function in React-redux,  
and it gets  `mapStateToProps`  parameters with passing  `object`.

So the default strategy of  `mapStateToProp`  is  `shallowEqual`  and  `useSelector`  is  `===`.

Also, we got POC what changes other index data in the same Collection could not invoke re-render(another collection instances could not be regenerated).  
=> When we call action and reducer seems makes new array and object, but it could not really generate new object on every calls on Reducer.

**So the conclusion is "Using object-type value in state is safe, but we should avoid to get fresh object without shallowEqual".**
