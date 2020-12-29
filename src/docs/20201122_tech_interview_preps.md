# Vuex / Redux 차이, Redux가 복잡해보이는 이유

## 결국 복잡한 비즈니스 로직을 작성하게 되면 Redux의 미들웨어와 같은 영역이 필요해진다.

Middleware 레이어가 없어 여러 비동기 Action을 캡슐화하기 힘들다.
모두 async/await로 처리하거나 해야 하는데
결국 복잡한 데이터 처리를 간결하게 작성할 수 있도록 돕는 패턴을 미들웨어 레이어에서 제공함
Redux-saga는 제네레이터 (이터레이터 생성)으로 해결했고,
Redux-observable 에선 RxJS(Observer[Observable] + Iterator[.next] + 함수형 프로그래밍 연산자)로 해결함.

비슷한 문제를 최근 Redux 대체하는 상태 관리 라이브러리에서도 공유한다. `recoil` , `jotai` 등.
`Atom` 이라는 단위로 작업하는데 결국 `Atom` 여러개를 묶어서 작업하면 `molecules` 가 되는 거고 `molecules` 여러 개 묶어서 처리하려면 고분자합성비동기처리패턴..비스무레한 게 나오는 것이고... 이것이 미들웨어 레이어와 비슷한 구현이 될 것이다.

https://jbee.io/react/thinking-about-global-state/
최근 jbee 선생님(?)의 전역 상태 저장소에 대한 단상을 봤는데,
이미 Angular에서 Singleton Service로 도메인 별 상태를 분리해 관리하는 패턴을 써왔기 떄문에.. (서버사이드와 비슷한 방식으로 도메인을 분리함을 알 수 있음)
전역 상태로 모든 API Result를 관리하는 것도 언젠가는 오버헤드가 큰 패턴이라며 Deprecated 될 수도 있지 않나 생각해봅니다.


# 자바스크립트 프로토타입을 다른 언어로 개발하던 엔지니어에게 설명해보세요!

## 첫면접 질문

자바스크립트의 모든 원시 타입은 `Object` 입니다.
그리고 모든 `Object`는 `prototype`을 가지고 있습니다.

이 `프로토타입`안에는 `constructor` 속성이 존재하는데, 자기 자신을 참조합니다.

`new` 키워드로 새로운 `object`를 생성하면, 생성자 함수의 `prototype` 속성을 `[[prototype]]` 속성으로 참조합니다.

이 `부모 프로토타입`에 직접 접근하려면 `[[prototype]]` 속성을 사용하거나,
브라우저 환경일 경우 `__proto__` 로 접근할 수 있고,
`Object.getPrototypeOf` 으로도 접근할 수 있습니다.
**편의상 `__proto__`로 표기하겠습니다.**

어떤 `인스턴스`에서 특정 `식별자`를 호출하면,
먼저 `식별자`를 `인스턴스` 내에서 찾고,
`인스턴스`에 없다면, 해당 `인스턴스`의 `prototype` 에서 찾습니다.
`인스턴스`의 `prototype`에도 없다면, `__proto__` 를 통해 부모 `프로토타입`에서 찾습니다.
이 과정을 반복하는 것이 `프로토타입 체이닝`입니다.

다시 위의 호출 우선순위를 짚어 보면,
부모 프로토타입에 같은 식별자를 가진 속성이 있더라도
생성한 인스턴스의 `prototype` 에서 같은 식별자를 가진 속성이 먼저 실행됩니다.

여기에서 부모 프로토타입은 `상속`된 클래스,
현재 인스턴스에서 선언된 속성은 `오버라이딩`에 빗대어 이해할 수 있겠습니다. :)

추가로, 프로토타입은 `__proto__ ` 형태로 직접 호출하지 않아도 됩니다. 
ex) `array.__proto__.forEach` === `array.forEach`.
언어 설계자가 의도한 내용이라, 그대로 이해하면 되겠습니다.

# 자바스크립트의 실행 컨텍스트를 설명해보세요!

## 함수 호출 당시의 주변 환경을 수집하는 것

### 함수의 호출 = 실행 컨텍스트 생성

### 함수의 호출 완료 시점 = 호출 스택에서 pop 될 때

`LexicalEnvironment`, `VariableEnvironment`, `thisBinding` 을 가집니다.

LexicalEnvironment

변수 선언 수집 `Environment Record`.
상위 컨텍스트 참조: `OuterEnvironmentReference`

현재 스코프에 없는 함수를 호출하면 `OuterEnvironmentReference`를 통해 상위 컨텍스트에서 함수를 찾습니다.

프로토타입 체인과 동작 방식이 유사합니다 => 스코프 체인.
상위 컨텍스트를 참조하게 되면 메모리에서 **실행 컨텍스트 정보를 유지**합니다! => 클로저.

V8 엔진에서는 이를 최적화해 참조하고 있는 변수만 메모리에 유지하나,
ECMAScript 명세에서는 컨텍스트 전체를 유지해야 한다고 정의되어 있습니다. ***출처필요**

# 자바스크립트의 비동기 처리 방식을 언어 동작방식에 빗대어 설명해보세요.

자바스크립트는 단일 쓰레드에서 동작하도록 설계된 언어이기 때문에,
비동기 처리를 위해 이벤트 루프라는 개념을 사용합니다.

대표적으로 브라우저에서의 비동기 호출은 Web API에서 처리하고,
처리가 완료된 작업의 콜백은 각각 호출된 API에 따라
`Task queue`, `Microtask queue`, `Animation frame queue`에 추가됩니다.
브라우저에 따라 우선순위가 다를 수 있으나, 
Chrome의 경우 `AnimationFrame`, `Microtask(Promise)`, `Task(setTimeout)` 순서대로 호출 스택에 추가됩니다.

**RxJS Scheduler와 자바스크립트 비동기 처리 관련 문서**
http://sculove.github.io/blog/2018/01/18/javascriptflow/

# async/await 를 직접 구현한다면? 

컴파일할 때 await 아래 구문을 모두 `promise.then` 으로 래핑하면 될 것 같습니다.
try-catch 구문이 존재할 경우 catch 영역을 수집해 `Promise.catch`콜백으로 추가.

# 짧은 질문

## var, let, const 변수 선언방식 차이

`var`로 선언된 변수는 실행 컨텍스트 수집 시점에  `undefined`로 값이 먼저 할당됩니다.
`let`, `const`로 선언된 변수는 값이 할당되지 않고, 할당되기 전에 호출할 경우 오류를 반환합니다 => `temporal dead zone`.

`var` 사용해 같은 식별자를 가지는 변수를, 같은 스코프에서 선언하면, 순서에 의존적이므로 디버깅이 힘들어질 수 있습니다. => 호이스팅

## `this`란 무엇인가...

자바스크립트에서 `this`는 인스턴스 메서드에서 사용될 때를 제외하면 모두 최상위 객체를 참조합니다.

인스턴스의 메서드로 호출될 때는 `호출 주체`를 `this`로 할당합니다.

```js
const lorem = {
  test: 1,
  getTest: function() {
    return this.test;
  },
};
// undefined
lorem.getTest();
// 1
const test = lorem.getTest;
// undefined
test()
// return undefined because window.test is undefined
```

## this.apply, this.call, this.bind 차이

모두 `function`에 `this`를 명시적으로 바인딩할 때 사용합니다.

#### function.call

첫 파라미터로 주어지는 `this`를 함수에 바인딩하고, 이어지는 파라미터를 `args`로 취하고, 함수 실행 결과를 반환합니다.

#### function.apply

첫 파라미터로 주어지는 `this`와, 추가 파라미터를 `array`로 취하고, 함수 실행 결과를 반환합니다.

#### function.bind

첫 파라미터로 주어지는 `this`를 바인드한 함수를 반환합니다.

## 콜백을 간단하게 설명해볼까요?

### 호출 주체(this)를 위임하는 함수

`this`를 호출하는 쪽에서 정하거나, `thisArg`로 전달받을 수 있습니다.

`addEventListener` 에 넘기는 콜백 함수의 경우 첫 번째 파라미터인 `event`를 브라우저로부터 주입받고, `this`로 이벤트 리스너 선언 주체인 `element`를 바인딩합니다.

`jQuery.each` 함수의 경우 콜백 내부에서의 `this`가 `jQuery` 인스턴스를 가리킵니다.

또는 `thisArg`를 직접 제공받는 경우도 있습니다.
대표적인 예로, `Array.filter` 등이 있습니다.
