# React Hook

## Hook이란?
> React 버전 16.8부터 React의 요소로 추가된 기능이다.
  기존의 클래스 바탕의 코드를 작성할 필요 없이 함수형 컴포넌트에서 상태값과 리액트의 여러 기능을 사용할 수 있다.

## Hook이 왜 생겨났는가?
기존의 React에서는 UI와 관련없는 로직들이 생겨날 경우, 이들을 재사용하는데 어려움을 겪었습니다. 이를 해결하기 위해서 `render-props`, `high order component`와 같은 패턴들을 이용했습니다.

하지만, 이러한 패턴들은 컴포넌트의 재구성을 강요합니다.
그래서 코드의 구성이 다음와 같은 그림(`wrapper hell`)과 같이 되버리죠. 
![image](https://github.com/intersoom/TSL/assets/78731710/648e84ab-2c03-478f-943c-e157172a6c63)

이러한 코드의 구성은 
- 코드의 추적을 어렵한다
- 성능에도 악영향을 미친다

이를 해결하기 위해서 만들어진게 Hook입니다.
이는 계층의 변화가 없이도 상태 관련 로직을 재사용할 수 있게 해줍니다.

또한, 클래스는 혼란스럽게 하는 지점들이 다수 존재합니다 (예를 들어, `this`).
그래서 함수형 컴포넌트를 지향하는데, hook이 이를 가능하게 해줍니다.

## Hook의 사용 규칙
> [리액트 공식 문서: Hook 규칙](https://ko.legacy.reactjs.org/docs/hooks-rules.html#only-call-hooks-at-the-top-level)

✅  **최상위(at the Top Level)에서만 Hook을 호출해야 합니다.** </br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;반복문, 조건문, 중첩된 함수 내에서 Hook을 실행하면 예상한 대로 동작하지 않을 우려가 있습니다.</br>
✅  **오직 리액트 함수 내에서만 사용되어야 합니다.** </br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;리액트 함수형 컴포넌트나 커스텀 Hook이 아닌 다른 일반 JavaScript 함수 안에서 호출해서는 안 됩니다.</br>

## Hook의 작동 방식
### useState의 작동 방식
Hook의 작동 방식은 Javascript의 [클로저](https://github.com/intersoom/TSL/blob/main/Front-end/Javascript/Closure.md)에 기반되어있습니다.

useState의 구성을 보면 총 네가지 변수로 이루어져있습니다.
```
let state = [];
let setters = [];
let firstRun = true;
let cursor = 0;
```

각각 무엇인지 설명해보겠습니다.
- `state`와 `setter`는 각각 우리가 흔히 state, setState로 알고 있는 값들을 담아주는 배열입니다.
- `firstRun`은 첫 렌더링 되었는지에 대한 상태를 나타내주는 변수입니다.
- `cursor`는 우리가 몇 번째 state에 접근하려고 하는지 알려주는 변수입니다. state를 변수에 담아뒀기 때문에 사용합니다.

이제부터 앞서 언급한 클로저가 등장합니다.
`setter`를 만들어주는 함수, `createSetter`라는 함수에 대해서 알아볼 것입니다.


해당 함수를 찬찬히 뜯어보면,
`cursor` 값을 인자로 받아서 `state[cursor]`에 접근해서 새로운 값(`newVal`)을 입력하는 함수를 반환해주고 있습니다.
> 이 부분이 클로저가 적용된 부분입니다.</br>
나중에 `setterWithCursor`가 `createSetter` 없이 함수가 담긴 변수를 통해서 혹은 배열의 인덱스를 통해서 접근했을 때도 해당 함수는 `cursor`의 값에 접근 가능할 것입니다.
```
function createSetter(cursor) {
  return function setterWithCursor(newVal) {
    state[cursor] = newVal;
  };
}
```

다음으로 useState의 pseudocode를 보겠습니다.

```
// This is the pseudocode for the useState helper
export function useState(initVal) {
  if (firstRun) {
    state.push(initVal);
    setters.push(createSetter(cursor));
    firstRun = false;
  }

  const setter = setters[cursor];
  const value = state[cursor];

  cursor++;
  return [value, setter];
}
```

`firstRun`의 초기값은 false이기 때문에 해당 조건문부터 실행이 될 것입니다.</br></br>
`firstRun`일 경우에 :
- useState(initValue)에서 넣어준 값(initValue)을 state의 초기값으로 `state` 배열에 넣어줍니다.
- 앞서 언급한 `createSetter` 함수를 활용해서 setterWithCursor 함수를 만들어서 `setters` 배열에 넣어줍니다.
- 마지막으로 이제 더 이상 초기실행이 아니기 때문에 `firstRun` 값을 false로 바꿔줍니다.
그리고 cursor 값을 ++ 해주고 우리가 아는 useState의 형식에 맞게 배열을 반환해줍니다.

마지막으로 hook을 사용하는 컴포넌트가 렌더되는 부분을 보겠습니다.
```
function RenderFunctionComponent() {
  const [firstName, setFirstName] = useState("Rudi"); // cursor: 0
  const [lastName, setLastName] = useState("Yardley"); // cursor: 1

  return (
    <div>
      <Button onClick={() => setFirstName("Richard")}>Richard</Button>
      <Button onClick={() => setFirstName("Fred")}>Fred</Button>
    </div>
  );
}
```
- useState("Rudi") 실행 시, cursor가 0이었으므로 이에 맞추어 state와 setter가 실행되고 배열에 들어갔을겁니다.
- useState("Yardley") 실행 시, cursor가 앞선 useState 실행으로 인해서 ++되었으므로 1입니다. 이에 맞추어 state와 setter가 실행되고 배열에 들어갔을겁니다.

```
// This is sort of simulating Reacts rendering cycle
function MyComponent() {
  cursor = 0; // resetting the cursor
  return <RenderFunctionComponent />; // render
}
```
React의 렌더링 사이클을 모방해서 이처럼 컴포넌트를 불러와보겠습니다.
```
console.log(state); // Pre-render: []
MyComponent();
console.log(state); // First-render: ['Rudi', 'Yardley']
MyComponent();
console.log(state); // Subsequent-render: ['Rudi', 'Yardley']

// click the 'Fred' button

console.log(state); // After-click: ['Fred', 'Yardley']
```
이렇게 정상적으로 작동할 것이라고 예상할 수 있습니다.
다시 말해, 컴포넌트가 렌더링될 때마다 `cursor`는 0부터 시작하고 1회 렌더링 시에서 useState를 1번 호출 할 때마다 `cursur`가 ++되면서 하나의 컴포넌트에 존재하는 state들에 접근하는 것입니다.

#### ➕ 반복문, 중첩문, 조건문 내에서 사용할 수 없는 이유
위와 같은 방식으로 useState가 만들어지고 사용되는데,</br>
중간에 조건문이 껴서 다음과 같이 useState가 첫렌더에만 선언이 되고 그 다음부터 호출이 되지 않는다면 어떨까요? 🤔
```
let firstRender = true;

function RenderFunctionComponent() {
  let initName;
  
  if(firstRender){
    [initName] = useState("Rudi");
    firstRender = false;
  }
  const [firstName, setFirstName] = useState(initName);
  const [lastName, setLastName] = useState("Yardley");

  return (
    <Button onClick={() => setFirstName("Fred")}>Fred</Button>
  );
}
```
- `firstRender`에서는 `cursor`가 0, 1, 2까지 존재했는데 다음번 렌더때는 0, 1만 존재할 것입니다.
- 또한, `firstRender`에서는 `cursor 0`이 Rudi를 담고 있었고 `cursor 1`이 Rudi를 담고 있었고 `cursor 2`가 Yardley를 담고 있었습니다.
- 따라서, 다음번 렌더때는 Rudi를 가르키고 있던 `cursor1`이 `cursor 0`이 되어서 또다른 Rudi를 가르킬 것이고 Yardley를 가르키고 있던 `cursor2`가 `cursor1`이 되어서 Rudi를 가르킬 것입니다.

다음과 같이 상태가 잘못 참조되는 일이 발생할 수 있기 때문에 **반복문, 중첩문, 조건문 내에서는 사용할 수 없습니다!**

### useEffect의 작동 방식
상태의 변화에 따른 사이드 이펙트를 위해서는 `useEffect`가 필요합니다.
기본적인 작동 방식은 의존성 배열 안에 있는 상태를 관찰하고 상태에 변화가 있다면, 콜백이 실행되는 방식입니다.
렌더링시, 최초에 한 번만 실행됩니다.
의존성 배열이 비어있다면, 최초 렌더링 한번시에만 콜백이 실행됩니다.

따라서 다시 말해, 의존성 배열을 관찰하면서 값이 변했다면 콜백 함수를 실행시키고 그렇지 않다면, 실행하지 않게 만들면 되는 것입니다.

```
const React = (function() {
  let hooks = []
  let idx = 0
  
  function useEffect(callback, depArray) {
    const oldDeps = hooks[idx] // 이미 저장되어있던 의존 값 배열이 있는지 본다.
    let hasChanged = true
  
    if (oldDeps) {
      // 의존 값 배열의 값 중에서 차이가 발생했는지 확인한다.
      hasChanged = depArray.some((dep, i) => !Object.is(dep, oldDeps[i]))
    }
    // 값이 바뀌었으니 콜백을 실행한다.
    if (hasChanged) {
      callback()
    }
    // useEffect도 훅의 일부분이다. hooks 배열에 넣어서 관리해준다.
    hooks[idx] = depArray
    idx++
  }

  function render(Component) {
    idx = 0 // 랜더링 시 훅의 인덱스를 초기화한다.
    const C = Component()
    C.render()
    return C
  }
  return { useEffect, render }
})()
```

따라서 위와 같이 구현하여서 `oldDeps`와의 차이를 확인하고 그렇다면, 콜백함수 실행 아니면 실행하지 않게 만들어습니다.

## Hook의 종류
#### 기본 Hook
- [useState](#usestate)
- [useEffect](#useeffect)
- useContext
#### 추가 Hooks
- useReducer
- useCallback
- useMemo
- useRef
- useImperativeHandle
- useLayoutEffect
- useDebugValue
- useTransition
- useId

공식 문서를 참고하면 이렇게 있습니다. 저는 일단, 자주 사용하는 useState ~ useRef까지만 정리해보겠습니다.
### useState
`state`는 컴포넌트가 유저 입력과 같은 정보를 기억하게 해주는 동적인 상태값입니다.
`useState`는 이러한 상태값과 그 값을 갱신하는 함수를 반환합니다.

사용법은 다음과 같습니다.
```
import { useState } from 'react';

const [state, setState] = useState(초기값);
 
/* 숫자형 */
const [state, setState] = useState(0);
 
/* 문자형 */
const [state, setState] = useState('');
 
/* Array */
const [state, setState] = useState([]);
 
/* object */
const [state, setState] = useState({});
 
/* boolean */
const [state, setState] = useState(true);
```

`setState`를 통해서 `state`를 갱신하면, 새 `state` 값을 받아서 컴포넌트 리렌더링을 큐에 등록합니다. 그러면 다음 리렌더링 시에 반환받은 `state` 값은 항상 갱신된 최신 `state`입니다.

함수적인 갱신도 가능합니다.
이전 `state`값을 이용해서 새로운 `state` 값을 계산하는 경우, 다음과 같이 사용 가능합니다.
```
function Counter({initialCount}) {
  const [count, setCount] = useState(initialCount);
  return (
    <>
      Count: {count}
      <button onClick={() => setCount(initialCount)}>Reset</button>
      <button onClick={() => setCount(prevCount => prevCount - 1)}>-</button>
      <button onClick={() => setCount(prevCount => prevCount + 1)}>+</button>
    </>
  );
}
```

이때, 주의할 점은 업데이트 함수가 현재 상태와 정확히 동일한 값을 반환한다면 바로 뒤에 일어날 리렌더링은 무시됩니다.
조금 더 자세하게 들어가보겠습니다.

다음과 같은 코드를 실행했다고 해봅시다.
```
const [movie, setMovie] = useState(['어바웃타임', '이터널 선샤인', '500일의 썸머']);
...생략...
onClick={()=> {
          let copy = movie;
          copy[0] = '꽃다발 같은 사랑을 했다';
          setMovie(copy)
        }}
```
이 때, 객체나 배열의 경우에는 원본 데이터를 직접 조작하는 것보다 기존 데이터는 보존되도록 작성하는 것이 중요하기 때문에 copy라는 변수를 만들어서 복사해보았습니다.
하지만, onClick을 실행해도 movie state의 [0] 값이 변하지 않습니다. 앞서 말씀드린대로 **동일한 값을 반환한다면 바로 뒤에 일어날 리렌더링이 무시된 것**일겁니다. (React에서는 [Object.is 비교 알고리즘](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object/is#Description) 사용)
값을 변경했는데, 왜 동일한 값으로 인식되고 리렌더링이 무시된걸까요?

이는 React보다는 Javascript에 대한 이해도가 필요합니다. 

Javascript 엔진에는 스택과 힙이라는 영역이 존재합니다.
스택은 원시값과 같은 정적 메모리를 할당하는 공간이고 힙은 참조값과 같은 동적 메모리를 할당하는 공간입니다.
이 때, Javascript의 모든 변수는 스택을 가리킵니다. 원시 값이 아닌 경우에는 스택에는 힙의 객체에 대한 참조가 포함됩니다.
왜냐하면, 앞서 언급한대로 Javascript에서 객체와 함수는 힙에 저장되기 때문입니다.

조금 더 명시적으로 설명해보자면,
```
const dog = {
  name: 'puppy',
  id: 1
}
```
에서 `dog`는 스택 영역에 저장되고 `{name, id}`의 내용은 힙 영역에 저장됩니다.
그래서 `dog`와 함께 스택 영역에는 힙 영역에 존재하는 `{name, id}`에 대한 참조가 함께 저장되는 것입니다.

다시 `useState` 내용으로 돌아가보겠습니다.

스택에 저장되어있는 movie의 값이 변했을까요? 아닙니다.
같은 곳을 참조하고 있고 참조하고 있는 힙의 영역이 변경되었을 뿐입니다.
그렇다면 어떻게 해야 `setMovie()`가 `state`가 변경되었다고 인식할까요? 새로운 배열에 대한 참조를 하면 되겠죠? 그렇다면 새로운 배열을 만들어줘서 스택에 있는 변수의 참조값을 바꿔줘야합니다.

간단한 방법은 Javascript의 `전개 구문 문법`을 사용하는 것입니다.
전개 구문 문법은 배열의 내용을 펼쳐서 **새로운 배열**을 만들어줍니다.
다시 말해, 힙의 영역에 새로운 공간이 만들어지고 스택에 있는 copy의 참조값은 새로운 공간에 대한 참조로 바뀌게 된다는 것입니다.
```
let copy = [...movie];
```
이렇게 바꿔주면,
```
onClick={()=> {
          let copy = [...movie]; //spread operator
          copy[0] = '겨울왕국';
          setMovie(copy)
        }}
```
setMovie가 값이 바뀐 것을 인지하고 리렌더링해주는 것을 확인할 수 있습니다.

### useEffect
`useEffect`는 리액트의 생명주기와 연관이 되었는 훅입니다. 
리액트의 함수 컴포넌트의 본문 안에서는 사이드 이펙트가 허용되지 않습니다. 이를 수행한다면 많은 버그들과 UI 불일치가 발생할 것이기 때문입니다. 따라서 리액트는 `useEffect`를 제공합니다.

`useEffect`에 전달된 함수의 실행 조건에 대해서 알아보겠습니다.

- **마운트:**</br>
&nbsp;&nbsp; - 컴포넌트가 페이지에 처음 **렌더링 된 후** **무조건** 실행</br>
- **업데이트:**</br>
&nbsp;&nbsp; - dependency 배열에 들어있는 state 값이 변경될 경우 실행</br>
&nbsp;&nbsp; - 부모가 리렌더링되는 경우 실행</br>
&nbsp;&nbsp; - context가 바뀔 때 실행</br>
- **언마운트:**</br>
&nbsp;&nbsp; - 컴포넌트가 페이지에서 사라질 때</br>

사용법은 다음과 같습니다.

```
// 1. 렌더링될 때마다 실행
useEffect(() => { 작업내용 });
 
// 2. 컴포넌트가 마운트될 때 한 번만 실행
useEffect(() => { 작업내용 },[]);
 
// 3. 컴포넌트가 마운트될 때 + dependency array의 값이 변경 될 때마다 실행
useEffect(() => { 작업내용 },[value]);
 
useEffect(() => { 
	
});
```
##### 1. 렌더링될 때마다 실행
기본적으로 useEffect의 동작은 첫번째 렌더링을 포함한 모든 렌더링이 완료된 후마다 실행되기 됩니다. 따라서 `1.`의 경우, dependency 배열이 존재하지 않기 때문에 state가 변경될 때마다 (즉, 화면이 리렌더링될 때마다) 실행이 됩니다.

##### 2. 컴포넌트가 마운트될 때 한 번만 실행, 3. 컴포넌트가 마운트될 때 + dependency array의 값이 변경 될 때마다 실행
dependency 배열을 빈 배열로 주면 처음 마운팅될 때 한 번 실행된 후, effect(useEffect 안에 작성된 함수)가 실행되지 않습니다. 
다시 말해, 기본적으로는 첫번째 렌더링 + 모든 렌더링 완료된 후마다 실행되지만, dependency 배열이 **존재**하는 경우에는 dependency 배열에 있는 props, state 변경에 따른 리렌더링 시에만 effect가 실행됩니다.
