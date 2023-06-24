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


