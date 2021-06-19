# Difference between the methods .pipe() and .subscribe() on a RXJS observable

[https://stackoverflow.com/questions/51269372/difference-between-the-methods-pipe-and-subscribe-on-a-rxjs-observable#:~:text=1%20Answer&text=The%20pipe%20method%20is%20for,easier%20to%20build%20smaller%20files](https://stackoverflow.com/questions/51269372/difference-between-the-methods-pipe-and-subscribe-on-a-rxjs-observable#:~:text=1%20Answer&text=The%20pipe%20method%20is%20for,easier%20to%20build%20smaller%20files).

Super easy concept but always forget ._.

> The `pipe` method is for chaining observable operators, and the `subscribe` is for activating the observable and listening for emitted values.  
> The `pipe` method was added to allow _webpack_ to drop unused operators from the final JavaScript bundle. It makes it easier to build smaller files.

# How to use `scheduled` API to create `Observable<T>`?

### Scheduler is language-independent common concept.  

Also implemented with RxJava, etc…  **to handle multiple Threads nicely.**
But Javascript has only one thread so.. implemented asynchronous tasks using `Event loop` on multiple queues.
It means javascript engine always infinitely loop these queues.

If there’s a task, just attach it into `Call stack`.
**So that schedulers intended to handle  **various task queues on Javascript.**

**0. Typedef**

`scheduled<T>(input: any, scheduler: SchedulerLike): Observable<T>`

1.  **Input**

why input is typed  `any`? => I don’t know….
In official document, it says input should be a kind of `arrayLike`  or  `iterable`s.
I think they judged it seems not required to specify type.
**allowed type for `input` is `ObservableInput`.**

`type ObservableInput<T> = SubscribableOrPromise<T> | ArrayLike<T> | Iterable<T>;`

**consumes:**  
- Observable,  
- Promise,  
- ArrayLike(having  `length`  prop and index starts from  `0`  and sequentially increases) 
ex)  `NodeList`  from querySelector,  
- Iterable (Object which implemented  `Iterable`  symbol === First call result of  `Generator`  function) 


2. **scheduler**

5 scheduler provided in default: you can use it with following syntax: 
`import { queueScheduler } from 'rxjs'`.  
but  `scheduled`  function requires to use these parameters except  `null`.

-   **null**

By not passing any scheduler, notifications are delivered synchronously and recursively. Use this for constant-time operations or tail recursive operations.

-   **queueScheduler**

Schedules on a queue in the  **current event frame**  (trampoline scheduler). Use this for iteration operations.It seems just using callback function synchronously but…I cannot assert.

-   **asapScheduler**

Schedules on the  **micro task queue**, which is the  **same queue used for promises**. Basically after the current job, but before the next job. Use this for asynchronous conversions.

-   **animationFrameScheduler**

Schedules task that will happen just before  **next browser content repaint**. Can be used to create smooth browser animations.**example:**  

```ts
    const scheduled: Observable<number> = scheduled([1, 2, 3], queueScheduler).pipe(  
      map(el => el * el),  
    );
```

**Related Docs (korean):**  
[https://12bme.tistory.com/572](https://12bme.tistory.com/572)  (RxJava)  

> So the key to asynchronous programming using the scheduler is:  
> It can separate threads that generate data flow from threads that communicate processed results to subscribers.

=> But we have only one thread bro……[http://sculove.github.io/blog/2018/01/18/javascriptflow/](http://sculove.github.io/blog/2018/01/18/javascriptflow/)**Official Docs (english):**  
[https://rxjs-dev.firebaseapp.com/guide/scheduler](https://rxjs-dev.firebaseapp.com/guide/scheduler)


# How to watch API result on single Action 

(with RxJS ~~future~~ current syntaxes)

**Use Scheduled API.**

ex)  Action PatchContentsTreeBySidebarTab =>  
MergeMap PatchScreen =>  
Observe FinishPatchScreen =>  
Action FinishPatchContentsTreeBySidebarTab.  
using with MergeMap (single Observable output)

## v1 - using `concat`

`concat` **has been deprecated.**

```ts
    /*  
     * @deprecated `concat` operator is deprecated. use concatAll() instead.  
     */   
    return concat<Action>(  
      patchAndFinishActions,  
      action$.pipe(  
        ofType(FinishPatchScreen),  
        filter(({ actionId }) => actionId === reqId),  
        take(1),  
        mergeMap(() => of(FinishPatchContentsTreeBy())),  
      ),  
    );
```

## v2 - using `of` with `concatAll`

**`of` has been deprecated.**

```ts
    return of(patchAndFinishActions, action$.pipe(  
      ofType(FinishPatchScreen),  
      filter(({ actionId }) => actionId === reqId),  
      take(1),  
      mergeMap(() => of(FinishPatchContentsTreeBy())),  
    )).pipe(  
      concatAll(),  
    );
```

## final - use scheduled API instead of them.

```ts
    return scheduled([  
      patchAndFinishActions, // or [patchAndFinishActions, withAnyActions]  
      action$.pipe(  
        ofType(FinishPatchScreen),  
        filter(({ actionId }) => actionId === reqId),  
        take(1),  
        mergeMap(() => of(FinishPatchContentsTreeBy())),  
      )],  
      queueScheduler,  
    ).pipe(  
      concatAll(),  
    );
```

Yes.

## Related Issues (on tslint, but RxJS source includes deprecation warning tho)

Rxjs “of” operator really deprecated? #4676  
[https://github.com/palantir/tslint/issues/4676](https://github.com/palantir/tslint/issues/4676)


# Comprehensive Guide to Higher-Order RxJs Mapping Operators: switchMap, mergeMap, concatMap (and exhaustMap)

https://blog.angular-university.io/rxjs-higher-order-mapping/

https://thebook.io/006934/part01/ch03/02/

굉장히 읽기 힘들지만 필요한 내용이 다 담긴 글..

> **연산자는 옵저버블 파이프라인에 로직을 삽입할 수 있는 선언적 함수 체인의 일부**입니다. 그리고 연산자는 **순수 함수**이자 **고차 함수**이기도 합니다. 즉, 연산자는 **대상 옵저버블 객체(소스라고 함)를 절대로 변경하지 않고 체인을
> 계속 유지하는 새로운 옵저버블 객체를 반환**합니다. 솔루션의 기본 요소인 비즈니스 로직을 구성하는 함수는 가능한 한 순수
> 함수를 사용해야 하기 때문에
> **이 시점부터 FP 모범 사례가 적용됩니다.**
