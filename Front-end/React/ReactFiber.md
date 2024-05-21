### Intro
최근에 동아리 친구들과 함께 <모던 리액트 딥다이브> 라는 책의 스터디를 진행하고 있다. 책을 읽으면서 블로그에도 틈틈히 내용들을 정리해서 올려보겠다!

### 히스토리
React Fiber 이전에는 리액트에서 코어 아키텍쳐 방식으로 <스택 재조정자>를 사용하였다.

그런데, 해당 방식에는 스택이라는 자료구조를 활용하는만큼, **😈 꺼내는 순서가 정해져 있다 😈** 는 문제점이 존재했다.

그래서 React 팀에서 v16부터 새롭게 제시한 방식이 Fiber 아키텍쳐이다.

### React Fiber의 목표
> 리액트 웹 어플리케이션에서 발생하는 애니메이션, 레이아웃, 사용자 인터랙션에 올바른 결과물을 만드는 반응성 문제를 해결하는 것

앞서 히스토리에서 언급한 대로, 스택 방식은 모든 과정이 **동기적**으로 이루어지므로, 리액트가 비효율적으로 작동되는 문제점을 가졌다.

하지만, Fiber는 이러한 문제점을 해결하기 위해서 비동기적으로 작동할 수 있게끔 만들어졌고, 다음과 같은 것들이 가능해졌다.

1. 작업을 작은 단위로 분할 + 쪼갠다 → 우선순위 매기기
2. 이러한 작업을 일시중지하고 다시 시작 가능
3. 이전에 했던 작업을 다시 재사용하거나 필요하지 않은 경우 폐기

이 모든 과정을 **비동기적**으로 처리할 수 있다.

### 용어 정리
용어가 조금 헷갈릴 수 있어서 먼저 짚고 넘어가겠다.
파이버에는 두 가지 의미가 존재한다.

- **파이버 (아키텍쳐)**

- **파이버 (노드)** : 리액트에서 관리하는 평범한 자바스크립트 객체이자, VDOM의 노드 객체이다. 

> **🤔 Fiber vs React Element**
Fiber는 React Element의 내용이 DOM에 반영되기 위해서 VDOM에 추가되어야하는데, 이를 위해서 React Element를 확장한 객체
>
> 즉, 정리하자면 **Fiber = React Element + (컴포넌트의 상태, life cycle, hook)**
> 
> 가장 큰 차이점은 Fiber는 mount될 때 만들어진 것을 재사용한다는 점이고 React Element는 계속해서 재생성 된다는 점이다.

### Fiber 실제 코드
[리액트 공식 레포 실제 코드](https://github.com/facebook/react/blob/main/packages/react-reconciler/src/ReactFiber.js#L134)
```jsx
function FiberNode(tag, pendingProps, key){
  // Instance
  this.tag = tag; // fiber의 종류를 나타냄
  this.key = key;
  this.type = null; // 추후에 React element의 type을 저장
  this.stateNode = null; // 호스트 컴포넌트에 대응되는 HTML element를 저장

  // Fiber
  this.return = null; // 부모 fiber
  this.child = null; // 자식 fiber
  this.sibling = null; // 형제 fiber
  this.index = 0; // 형제들 사이에서의 자신의 위치

  this.pendingProps = pendingProps; // workInProgress는 아직 작업이 끝난 상태가 아니므로 props를 pending으로 관리
  this.memoizedProps = null; // Render phase가 끝나면 pendingProps는 memoizedProps로 관리
  this.updateQueue = null; // 컴포넌트 종류에 따라 element의 변경점 또는 라이프사이클을 저장
  this.memoizedState = null; // 함수형 컴포넌트는 훅을 통해 상태를 관리하므로 hook 리스트가 저장된다.

  // Effects
  this.effectTag = NoEffect; // fiber가 가지고 있는 side effect를 기록
  this.nextEffect = null; // side effect list 
  this.firstEffect = null; // side effect list
  this.lastEffect = null; // side effect list 

  this.expirationTime = NoWork; // 컴포넌트 업데이트 발생 시간을 기록
  this.childExpirationTime = NoWork; // 서브 트리에서 업데이트가 발생할 경우 기록

  this.alternate = null; // 반대편 fiber를 참조
}
```

- `tag` : Fiber는 항상 **무언가**와 1 : 1의 대응 관계를 갖는다.
그래서 그 대응 관계에 있는 것을 다음 표를 참고하여서 `tag`라는 속성에 기입해둔다. 

```js
var FunctionComponent = 0
var ClassComponent = 1
var IndeterminateComponent = 2
var HostRoot = 3
var HostPortal = 4
var HostComponent = 5
var HostText = 6
var Fragment = 7
var Mode = 8
var ContextConsumer = 9
var ContextProvider = 10
var ForwardRef = 11
var Profiler = 12
var SuspenseComponent = 13
var MemoComponent = 14
var SimpleMemoComponent = 15
var LazyComponent = 16
var IncompleteClassComponent = 17
var DehydratedFragment = 18
var SuspenseListComponent = 19
var ScopeComponent = 21
var OffscreenComponent = 22
var LegacyHiddenComponent = 23
var CacheComponent = 24
var TracingMarkerComponent = 25
```
>  **💡 참고 💡**
`HostComponet` = `div`, paragraph, etc

- `stateNode` : Fiber 자체에 대한 참조 정보를 가지고 있음, 이를 바탕으로 리액트는 파이버와 관련된 상태에 접근

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;앞서 언급한 *무언가* 와의 참조 관계를 유지할 수 있도록 도와준다.

- `return`, `child`, `sibling`, `index` : Fiber 간의 관계를 나타내는 속성들

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Fiber Tree에는 `children` 관계는 없고, `child`와 `sibling`의 관계만 존재한다.


```jsx
this.return = null; // 부모 fiber
this.child = null; // 자식 fiber
this.sibling = null; // 형제 fiber
this.index = 0; // 형제들 사이에서의 자신의 위치
```

다음과 같은 예시를 보면 이해가 빠를 것이다.

```html
<div>
	<h1>1</h1>
	<h2>2</h2>
	<h3>3</h3>
</div>
```

![](https://velog.velcdn.com/images/intersoom/post/b92b9351-0b38-4930-bf41-5551d8c5940c/image.png)


`첫번째 자식`만 자식 관계로 취급되고 나머지의 자식들은 형제 관계로 취급된다.

- `pendingProps` : 아직 작업을 미처 처리하지 못한 props
- `memoizedProps` : pendingProps를 기준으로 렌더링이 완료된 이후에 pendingProps를 memoizedProps에 저장해 관리함
- `updateQueue` : 상태 업데이트, 콜백 함수, DOM 업데이트 등 필요한 작업을 담아두는 큐
    
    → 구조
    
    ```tsx
    type UpdateQueue = {
    	first : Update | null
    	last : Update | null
    	hasForceUpdate : boolean
    	callbackList : null | Array<Callback> // setState로 넘긴 콜백 목록
    }
    ```
    
- `memoizedState` : 함수 컴포넌트의 훅 목록이 저장됨
- `alternate` : 리액트 파이버 트리와 이어질 개념. 리액트의 트리는 2개인데, 이 alternate는 반대편 트리 파이버를 가리킴

### Fiber Tree
이제 `alternate` 속성에 대해서 설명하면서 살짝 언급한 **Fiber Tree**에 대해서 알아보자.

#### 재조정

Fiber는 재조정자가 관리하는데, 

이는 가상 DOM(메모리 상에 UI 관련된 정보들을 띄우는 것)과 실제 DOM을 비교해 변경 사항을 수집하며, 만약 이 둘 사이에 차이가 있다면 변경에 관련된 정보를 가지고 있는 Fiber를 기준으로 화면에 렌더링 요청을 하는 것이다.

> **재조정 = 가상 DOM ↔ 실제 DOM을 비교하는 알고리즘**

> **💭 참고**
> 
> Note : 최근에 React 팀은 "가상 DOM"이라는 용어가 그리 대단한게 아니라고 밝혔습니다. Dan Abramov는 최근에 다음과 같이 얘기했습니다.
저는 "가상 DOM"이라는 용어를 폐기했으면 합니다. 이 용어는 2013년에는 말이 됐습니다. 왜냐하면 그때는 사람들이 React가 매번 렌더 할 때 마다 DOM 노드를 생성한다고 가정했기 때문입니다. 하지만 최근에는 이렇게 가정하는 사람이 거의 없습니다. "가상 DOM"은 마치 무슨 DOM 관련 이슈에 대한 임시방편 (Workaround)인 것 처럼 들립니다. 하지만 React는 그런게 아니에요.
React는 "value UI"입니다. React의 핵심 원칙은 UI는 문자열이나 배열처럼 그저 값 (value)이라는 겁니다. 여러분은 이 값을 변수에 저장하고, 어디든지 전달할 수 있으며, JavaScript의 제어 흐름 (Control Flow) 등등을 사용할 수 있습니다. 이러한 표현력 (expressiveness)이 핵심입니다. 변경 사항을 DOM에 적용하는 걸 막기 위한 비교 행위같은게 아닙니다.
심지어 React는 항상 DOM을 대표하지도 않습니다. 예를 들어 <Message recipientId={10} /> 같은 것은 DOM이 아닙니다. 개념적으로 React는 Message.bind(null, { recipientId: 10 })와 같은 게으른 함수 (Lazy Function) 호출을 대표합니다.

#### 실제 Fiber Tree

Fiber Tree는 앞서 언급한 대로 다음과 같이 2개의 트리로 구성 되어있다.

이를 **더블 버퍼링 구조**라고 한다. (그래픽 용어로 널리 활용됨) 

![](https://velog.velcdn.com/images/intersoom/post/5a4b63d3-a45c-4400-9828-ae35c69a9920/image.png)


> **🤔 더블 버퍼링이란?**
> 
> 예를 들어서, 트리에 내용을 모두 지워야 하는 경우가 있다고 해보자.
> 
> 이 때, 하나의 트리만을 이용한다고 하면 트리의 내용을 모두 지워지는 중간 중간 과정을 사용자가 볼 수 있다. 그렇게 되면, 사용자 경험의 큰 저하를 줄 것이다. _화면이 막 반짝 반짝할 테니까.._
> 
> 이러한 사용자 경험 저하를 막기 위해서 A라는 트리를 보여줌과 동시에, B라는 트리에서 내용을 지우는 작업을 진행한다. 그리고 해당 작업이 완료 되면, 화면에 보이는 트리를 A에서 B로 대체함으로서, 처리 과정은 사용자가 볼 수 없게 하되 처리 결과만을 바로 바꿔서 보여주는 것이다!

그렇다면, 이런 더블 버퍼랑 구조를 활용한 React의 VDOM (편의상, 가상 DOM을 VDOM이라고 하겠다)은 어떻게 구성되어있을까?

`Current Tree`와 `WorkInProgress Tree`(WIP Tree라고 하겠다)로 구성되어 있다.

앞서 더블 버퍼링을 설명하면서 언급한 A 트리가 Current Tree이고, B 트리가 WIP Tree이다.

이 둘은 Fiber의 속성에서 언급한, alternate 속성으로 서로에 대한 참조를 유지하고 있고, 뒤에서 설명할 render phase에서 commit phase로 넘어가면서 둘의 역할이 스위치된다. 

즉, current Tree를 가리키는 pointer가 A 트리를 가리키고 있다가, commit phase로 넘어가면서, B 트리를 가리키게 된다는 것이다.

#### 파이버의 작업 순서

1. `beginWork()` 함수 실행을 통해 파이버 작업을 수행 → 자식 없는 파이버를 만날 때까지 트리 형식으로 시작된다.

2. 1번에서 작업이 끝난다면 그다음 `completeWork()` 함수를 실행해 파이버 작업을 완료한다.

3. 형제가 있다면 형제로 넘어간다.

4. 2, 3번이 모두 끝나면 `return`으로 돌아가 자신의 작업이 완료되었음을 알린다.

처음에 트리가 생성됨 → 이후에 setState 등의 업데이트 발생한다면, 기존의 생성된 트리를 활용해서 작업을 함 workInProgress 트리에서 **“가급적 새로운 파이버를 생성하지 않는다”** _(Fiber와 React Element의 차이점)_

#### Life Cycle

![](https://velog.velcdn.com/images/intersoom/post/78abd958-b075-4197-98f4-57fa480e2137/image.png)

- **render phase**

해당 단계는 **비동기적**으로 이루어진다.

1. 재조정 단계
2. 재조정자(reconciler)가 element 추가, 수정, 삭제와 같은 reconciler가 컴포넌트의 변경을 DOM에 적용하기 위해 수행하는 일(work)을 scheduler에 등록한다.
3. scheduler가 work를 실행한다.

이 때, reconciler가 scheduler에 일을 등록하면서, `abort`, `top`, `restart`와 같은 함수를 이용하여 일의 우선순위를 매길 수 있다. 

- **commit phase**

해당 단계는 **동기적**으로 이루어진다.

1. 재조정한 VDOM을 즉, WIP 트리의 내용을 실제 DOM에 반영한다. 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;반영 직후에는, `실제 DOM 내용 === WIP 트리 내용` 이기 때문에 WIP 트리가 Current 트리가 되는 것이라고도 할 수 있다.

2. DOM 조작 일괄 처리 후, 리액트가 콜스택을 비워준 다음에 브라우저가 paint 시작
