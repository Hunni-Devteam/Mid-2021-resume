# 20210106 - UseEffect Deps-related discussions on 

## Shabriwa Today at 4:59 PM

Hello @frontend, I came across this code when working on Cesium, and I have a question :sweat_smile:

```ts
useEffect(() => {
    if (threeDOrthoMapId === undefined || viewer === undefined || viewer.isDestroyed()) return;
    const imagery: ImageryLayer | undefined = viewer?.imageryLayers
      .addImageryProvider(makeS3TileImageryProvider(threeDOrthoMapId)) as ImageryLayer | undefined;
    return () => {
      if (imagery && !viewer.isDestroyed()) viewer?.imageryLayers.remove(imagery);
    };
  }, [viewer, threeDOrthoMapId]);
```

Does anyone know why Cesium viewer instance is a dependency of useEffect, when it's not even "reactive", it's just an instance?

What is the effect on the useEffect ? :thinking_face: I tried googling it but it came up nothing, if anyone knows let me know. Thank you~

 안녕하세요 @frontend, 세슘 작업을 하다가 this code를 발견했는데 질문이 있습니다 :sweat_smile:
 (code)
  Cesium viewer 인스턴스가 useEffect의 종속성인 이유를 알고 계십니까? "반응적"도 아니고 단지 인스턴스일 뿐입니다. useEffect에 어떤 영향을 미칠까요? :thinking_face: 검색해 봤는데 아무 것도 안 나왔어요. 누가 저한테 알려주신다면요. 감사합니다.
(edited)

#### 7 replies

## Yejun Lee  17 minutes ago

@Hun Kang Could you explain this for briwa?
@Hun Kang 브리와를 위해 이것을 설명해 주시겠습니까?
(edited)
:blob_excited:
1
:pepe:
1

## Hun Kang  14 minutes ago
Explicit dependency on useEffect callback. It means that callback depends state of viewer as well
useEffect 콜백에 대한 명시적 종속성입니다. 콜백도 viewer의 상태에 따라 달라진다는 의미입니다.
(edited)

## Shabriwa  13 minutes ago
@Hun Kang Ah so you're saying if viewer changes, the useEffect gets retriggered? My question was viewer won't change since it's not a reactive instance technically :thinking_face: But you're saying it's needed that way regardless. Is that correct?
@Hun Kang 아, 그러면 viewer가 바뀌면 useEffect이 다시 트리거된다는 말씀이신가요? 제 질문은 viewer - 기술적으로는 사후 대응적인 사례가 아니기 때문에 변화하지 않을 것이라는 것이었지만, 여러분은 어쨌든 그런 방식이 필요하다고 말하고 있습니다. 그것이 맞습니까?
(edited)

## Hun Kang  13 minutes ago
And I think maybe that can be executed before initializing viewer , it means useEffect should detect viewer mutates undefined to Viewer -like instance
또한 viewer을 초기화하기 전에 실행할 수 있다고 생각합니다. useEffect가 뷰어 돌연변이를 undefined에서 Viewer와 같은 인스턴스를 감지해야 합니다.
(edited)

## Hun Kang  11 minutes ago
Yep!
Even if  viewer is immutable , enumerating outer-dependencies on deps (second parameter on useEffect/useCallback/useMemo) is best practice as I know.
 네!
만약  viewer 가 immutable 하더라도, deps에 대한 외부 의존성(useEffect/useCallback/useMemo에 대한 두 번째 매개 변수)만 열거하는 것이 제가 아는 바입니다.
(edited)
:+1:
1

## Shabriwa  11 minutes ago
Got it, much thanks @Hun Kang!
알겠어요, 정말 고마워요@Hun Kang!
(edited)
:joy:
1

## Hun Kang  7 minutes ago
Additionaly I think it’s helpful to track outer-dependencies on hooks callback.
Also that feature is supported on eslint plugin , but that plugin force to add dispatch on deps and it seems unnecessary :sweat_smile:
So yes, we can keep it bruh
또한 후크 콜백에 대한 외부 종속성을 추적하는 것이 도움이 된다고 생각합니다.
또한 이 기능은 eslint 플러그인에서 지원되지만 플러그인은 dep에 디스패치를 추가하도록 강제하며 불필요해 보입니다.sweat_smile:
그래, 우린 비밀로 할 수 있어
(edited)
스크린샷 2021-01-06 오후 5.09.08.png 
스크린샷 2021-01-06 오후 5.09.08.png


:+1:
1

