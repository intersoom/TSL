# useEffect 완벽 가이드 요약본

> 해당 글은 밑의 링크의 사이트를 요약한 글입니다. </br> [[번역] useEffect 완벽 가이드](https://www.rinae.dev/posts/a-complete-guide-to-useeffect-ko)

### 모든 랜더링은 고유의 prop과 state가 있다

```jsx
// 처음 랜더링 시
function Counter() {
  const count = 0; // useState() 로부터 리턴
  // ...
  <p>You clicked {count} times</p>;
  // ...
}
// 클릭하면 함수가 다시 호출된다
function Counter() {
  const count = 1; // useState() 로부터 리턴
  // ...
  <p>You clicked {count} times</p>;
  // ...
}
// 또 한번 클릭하면, 다시 함수가 호출된다
function Counter() {
  const count = 2; // useState() 로부터 리턴
  // ...
  <p>You clicked {count} times</p>;
  // ...
}
```

state를 업데이트할 때마다, 리액트는 컴포넌트를 호출합니다.

매 렌더 결과물은 고유의 `counter` 상태값을 “살펴봅니다.” 이 값을 함수 안에 **상수**로 존재합니다.

`setCount` 를 호출할 때, 리액트는 다른 `count` 값과 함께 컴포넌트를 다시 호출합니다. 그러면 리액트는 가장 최신의 랜더링 결과물과 일치하도록 DOM을 업데이트 합니다.

명심하셔야 할 점은 여느 특정 랜더링 시 그 안에 있는 `count` 상수는 시간이 지난다고 바뀌는 것이 아니라는 것입니다. 컴포넌트가 다시 호출되고, **각각의 랜더링마다 격리된 고유의 `count` 값을 “보는” 것입니다.**


> 🔥 우리의 함수는 여러번 호출되지만(랜더링마다 한 번씩), 각각의 랜더링에서 함수 안의 `count` 값은 상수이자 **독립적인 값(특정 랜더링 시의 상태)으로 존재합니다.**

### 모든 랜더링은 고유의 이벤트 핸들러를 가진다

```jsx
// 처음 랜더링 시
function Counter() {
  const count = 0; // useState() 로부터 리턴
  // ...
  function handleAlertClick() {
    setTimeout(() => {
      alert("You clicked on: " + count);
    }, 3000);
  }
  // ...
}
// 클릭하면 함수가 다시 호출된다
function Counter() {
  const count = 1; // useState() 로부터 리턴
  // ...
  function handleAlertClick() {
    setTimeout(() => {
      alert("You clicked on: " + count);
    }, 3000);
  }
  // ...
}
// 또 한번 클릭하면, 다시 함수가 호출된다
function Counter() {
  const count = 2; // useState() 로부터 리턴
  // ...
  function handleAlertClick() {
    setTimeout(() => {
      alert("You clicked on: " + count);
    }, 3000);
  }
  // ...
}
```

```jsx
// 처음 랜더링 시
function Counter() {
  // ...
  function handleAlertClick() {
    setTimeout(() => {
      alert("You clicked on: " + 0);
    }, 3000);
  }
  // ...
  <button onClick={handleAlertClick} />; // 0이 안에 들어있음
  // ...
}
// 클릭하면 함수가 다시 호출된다
function Counter() {
  // ...
  function handleAlertClick() {
    setTimeout(() => {
      alert("You clicked on: " + 1);
    }, 3000);
  }
  // ...
  <button onClick={handleAlertClick} />; // 1이 안에 들어있음
  // ...
}
// 또 한번 클릭하면, 다시 함수가 호출된다
function Counter() {
  // ...
  function handleAlertClick() {
    setTimeout(() => {
      alert("You clicked on: " + 2);
    }, 3000);
  }
  // ...
  <button onClick={handleAlertClick} />; // 2가 안에 들어있음
  // ...
}
```

→ 각각의 랜더링은 고유한 “버전”의 `handleAlertClick` 을 리턴합니다. 그리고 각각의 버전은 고유의 `count` 를 “기억합니다”.

→ 이벤트 핸들러가 특정 랜더링에 **“속해 있으며”,** 얼럿 표시 버튼을 클릭할 때 해당 랜더링 시점의 `counter` state를 유지한채로 사용하는 것입니다!

### 모든 랜더링은 고유의 이펙트를 가진다

우리는 이미 `count` 는 특정 컴포넌트 랜더링에 포함되는 상수라고 배웠습니다. 이벤트 핸들러는 그 랜더링에 “속한” `count` 상태를 “봅니다”. `count` 는 특정 범위 안에 속하는 변수이기 때문입니다. 이펙트에도 똑같은 개념이 적용됩니다!

**“변화하지 않는” 이펙트 안에서 `count` 변수가 임의로 바뀐다는 뜻이 아닙니다. *이펙트 함수 자체가* 매 랜더링마다 별도로 존재합니다.**

각각의 이펙트 버전은 매번 랜더링에 “속한” `count` 값을 “봅니다”.

```jsx
// 최초 랜더링 시
function Counter() {
  // ...
  useEffect(
    // 첫 번째 랜더링의 이펙트 함수
    () => {
      document.title = `You clicked ${0} times`;
    }
  );
  // ...
}
// 클릭하면 함수가 다시 호출된다
function Counter() {
  // ...
  useEffect(
    // 두 번째 랜더링의 이펙트 함수
    () => {
      document.title = `You clicked ${1} times`;
    }
  );
  // ...
}
// 또 한번 클릭하면, 다시 함수가 호출된다
function Counter() {
  // ...
  useEffect(
    // 세 번째 랜더링의 이펙트 함수
    () => {
      document.title = `You clicked ${2} times`;
    }
  );
  // ..
}
```

**<랜더링 과정>**

1. **첫번째 렌더링**
   - 리액트: state가 `0` 일 때의 UI를 보여줘.
   - 컴포넌트
     - 여기 랜더링 결과물로 `<p>You clicked 0 times</p>` 가 있어.
     - 그리고 모든 처리가 끝나고 이 이펙트를 실행하는 것을 잊지 마: `() => { document.title = 'You clicked 0 times' }`.
   - 리액트: 좋아. UI를 업데이트 하겠어. 이봐 브라우저, 나 DOM에 뭘 좀 추가하려고 해.
   - 브라우저: 좋아, 화면에 그려줄게.
   - 리액트: 좋아 이제 컴포넌트 네가 준 이펙트를 실행할거야.
     - `() => { document.title = 'You clicked 0 times' }` 를 실행하는 중.
2. **버튼 클릭 시**
   - 컴포넌트: 이봐 리액트, 내 상태를 `1` 로 변경해줘.
   - 리액트: 상태가 `1` 일때의 UI를 줘.
   - 컴포넌트
     - 여기 랜더링 결과물로 `<p>You clicked 1 times</p>` 가 있어.
     - 그리고 모든 처리가 끝나고 이 이펙트를 실행하는 것을 잊지 마: `() => { document.title = 'You clicked 1 times' }`.
   - 리액트: 좋아. UI를 업데이트 하겠어. 이봐 브라우저, 나 DOM에 뭘 좀 추가하려고 해.
   - 브라우저: 좋아, 화면에 그려줄게.
   - 리액트: 좋아 이제 컴포넌트 네가 준 이펙트를 실행할거야.
     - `() => { document.title = 'You clicked 1 times' }` 를 실행하는 중.

### 모든 랜더링은 고유의 … 모든 것을 가지고 있다

```jsx
function Counter() {
  const [count, setCount] = useState(0);
  useEffect(() => {
    setTimeout(() => {
      console.log(`You clicked ${count} times`);
    }, 3000);
  });
  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
```

P: 해당 코드에서 약간 뜸을 들이며 버튼을 여러번 누른다면?

Q: 순서대로 1, 2, 3, 4, 5 나옴!

왜냐하면, 각각의 타이머는 특정 랜더링에 속하기 때문에 그 시점의 `count` 값을 갖기 때문에

**➕ 플러스 알파 정보**

원래 클래스 컴포넌트로 만들면 이렇게 작동하지 않음

```jsx
componentDidUpdate() {
  setTimeout(() => {
     console.log(`You clicked ${this.state.count} times`);
  }, 3000);
}
```

→ 특정 랜더링 시점의 값이 아니라 언제나 최신의 값을 가리키게 됨

→ 매번 5가 찍혀있는 로그를 발견할 수 있음

훅은 클로저에 의존하고 있는데, 클로저는 접근하려는 값이 절대 바뀌지 않을 때 유용함. 반드시 상수를 참조하고 있기 때문에 생각을 하기 쉽도록 만들어줌

### 흐름을 거슬러 올라가기


> 🔥 컴포넌트의 랜더링 안에 있는 **모든** 함수는 (이벤트 핸들러, 이펙트, 타임아웃이나 그 안에서 호출되는 API 등) 랜더(render)가 호출될 때 정의된 props와 state 값을 잡아둔다.



- ➕ 물론 때때때로 이펙트 안에 정의해둔 콜백에서 사전에 잡아둔 값이 아니라 최신의 값을 이용하고 싶을 때가 있음
  그럴 때는 ref 사용하면 됨!
  ```jsx
  function Example() {
    const [count, setCount] = useState(0);
    const latestCount = useRef(count);
    useEffect(() => {
      // 변경 가능한 값을 최신으로 설정한다
      latestCount.current = count;
      setTimeout(() => {
        // 변경 가능한 최신의 값을 읽어 들인다
        console.log(`You clicked ${latestCount.current} times`);
      }, 3000);
    });
  ```
  잘못되지는 않았지만 “덜 깨끗”해보일 수는 있음!
  ⇒ `this.state`가 사용하는 방식이 위와 같은 방식임

### 그러면 클린업은 뭐지?

어떤 이펙트는 클린업 단계를 가질 수도 있습니다. 본질적으로 클린업의 목적은 구독과 같은 이펙트를 “되돌리는” 것입니다.

```jsx
useEffect(() => {
  ChatAPI.subscribeToFriendStatus(props.id, handleStatusChange);
  return () => {
    ChatAPI.unsubscribeFromFriendStatus(props.id, handleStatusChange);
  };
});
```

첫 번째 랜더링에서 `prop` 이 `{id: 10}` 이고, 두 번째 랜더링에서 `{id: 20}` 이라고 해 봅시다. *아마도* 아래와 같은 흐름대로 흘러가리라 생각하실겁니다.

- 리액트가 `{id: 10}` 을 다루는 이펙트를 클린업한다.
- 리액트가 `{id: 20}` 을 가지고 UI를 랜더링한다.
- 리액트가 `{id: 20}` 으로 이펙트를 실행한다.

(실제는 조금 다릅니다)

위의 멘탈 모델대로라면, 클린업이 리랜더링 되기 전에 실행되고 이전의 prop을 “보고”, 그 다음 새 이펙트가 리랜더링 이후 실행되기 때문에 새 prop을 “본다고” 생각할 수 있습니다. 이 멘탈 모델은 클래스의 라이프사이클을 그대로 옮겨 놓은 것과 같고, 여기서는 잘못된 내용입니다. 왜 그런지 알아봅시다.

리액트는 브라우저가 페인트 하고 난 뒤에야 이펙트를 실행합니다. 이렇게 하여 대부분의 이펙트가 스크린 업데이트를 가로막지 않기 때문에 앱을 빠르게 만들어줍니다. 마찬가지로 이펙트의 클린업도 미뤄집니다. **이전 이펙트는 새 prop과 함께 리랜더링 되고 난 *뒤에* 클린업됩니다.**

- 리액트가 `{id: 20}` 을 가지고 UI를 랜더링한다.
- 브라우저가 실제 그리기를 한다. 화면 상에서 `{id: 20}` 이 반영된 UI를 볼 수 있다.
- 리액트는 `{id: 10}` 에 대한 이펙트를 클린업한다.
- 리액트가 `{id: 20}` 에 대한 이펙트를 실행한다.

이상하게 느낄 수 있습니다. 그런데 어떻게 prop이 `{id: 20}` 으로 바뀌고 나서도 이전 이펙트의 클린업이 여전히 예전 값인 `{id: 10}` 을 “보는” 걸까요?

이제 답이 명확해졌네요! 어찌되었건 이펙트의 클린업은 “최신” prop을 읽지 않습니다. 클린업이 정의된 시점의 랜더링에 있던 값을 읽는 것입니다.

```jsx
// 첫 번째 랜더링, props는 {id: 10}
function Example() {
  // ...
  useEffect(
    // 첫 번째 랜더링의 이펙트
    () => {
      ChatAPI.subscribeToFriendStatus(10, handleStatusChange);
      // 첫 번째 랜더링의 클린업
      return () => {
        ChatAPI.unsubscribeFromFriendStatus(10, handleStatusChange);
      };
    }
  );
  // ...
}
// 다음 랜더링, props는 {id: 20}
function Example() {
  // ...
  useEffect(
    // 두 번째 랜더링의 이펙트
    () => {
      ChatAPI.subscribeToFriendStatus(20, handleStatusChange);
      // 두 번째 랜더링의 클린업
      return () => {
        ChatAPI.unsubscribeFromFriendStatus(20, handleStatusChange);
      };
    }
  );
  // ...
}
```

→ 첫 번째 랜더링의 클린업이 바라보는 값을 `{id: 10}` 이외의 것으로 만들 수 없습니다.

이렇게 리액트는 페인팅 이후 이펙트를 다루는게 기본이며 그 결과 앱을 빠르게 만들어 줍니다. 이전 props는 우리의 코드가 원한다면 남아 있습니다.\*\*\*\*

### 라이프사이클이 아니라 동기화


> 🔥 **모든 것은 목적지에 달렸지 여정에 달린게 아니다.**



**리액트는 우리가 지정한 props와 state에 따라 DOM과 동기화합니다.**랜더링 시 “마운트” 와 “업데이트” 의 구분이 없습니다.

`useEffect` 는 리액트 트리 바깥에 있는 것들을 props와 state에 따라 *동기화* 할 수 있게 합니다.

```jsx
function Greeting({ name }) {
  useEffect(() => {
    document.title = "Hello, " + name;
  });
  return <h1 className="Greeting">Hello, {name}</h1>;
}
```

→ **만약 컴포넌트가 첫 번째로 랜더링할 때와 그 후에 다르게 동작하는 이펙트를 작성하려고 하신다면, 흐름을 거스르는 것입니다!** 랜더링 결과물이 “목적지” 에 따라가는 것이 아니라 “여정” 에 따른다면 동기화에 실패합니다.

컴포넌트를 prop A, B, C 순서로 랜더링하던지, 바로 C로 랜더링하던지 별로 신경쓰이지 않아야 합니다. 잠깐 차이가 있을 수 있지만(예: 데이터를 불러올 때), 결국 마지막 결과물은 같아야 합니다.

당연하지만 여전히 모든 이펙트를 *매번* 랜더링마다 실행하는 것은 효율이 떨어질 수 있습니다. (그리고 어떤 경우에는 무한루프를 일으킬 수 있지요.)

### 리액트에게 이펙트를 비교하는 법을 가르치기

React에서 매번의 리랜더링마다 DOM 전체를 새로 그리는 것이 아니라, 실제로 바뀐 부분만 DOM을 업데이트합니다.

예를 들어서,

`#1`

```jsx
<h1 className="Greeting">Hello, Dan</h1>
```

`#2`

```jsx
<h1 className="Greeting">Hello, Yuzhi</h1>
```

`#1` → `#2`로 바꾼다면,

리액트는 두 객체를 비교합니다.

```jsx
const oldProps = { className: "Greeting", children: "Hello, Dan" };
const newProps = { className: "Greeting", children: "Hello, Yuzhi" };
```

`children`은 바뀌어서 DOM 업데이트가 필요하지만, `className`은 아닙니다. 따라서 다음과 같은 코드만 호출됩니다.

```jsx
domNode.innerText = "Hello, Yuzhi";
// domNode.className 은 건드릴 필요가 없다
```

위와 같은 방식을 effect에 적용시킨다면? effect를 적용할 필요가 없다면 다시 실행시키지 않는 것이 좋을 것입니다.

```jsx
function Greeting({ name }) {
  const [counter, setCounter] = useState(0);
  useEffect(() => {
    document.title = "Hello, " + name;
  });
  return (
    <h1 className="Greeting">
      Hello, {name}
      <button onClick={() => setCounter(count + 1)}>Increment</button>
    </h1>
  );
}
```

해당 effect에서는 `counter`의 상태값을 사용하지 않는데 `document.title`을 카운터 값이 바뀔 때마다 재할당하는 것은 이상적이지 않습니다.

```jsx
let oldEffect = () => {
  document.title = "Hello, Dan";
};
let newEffect = () => {
  document.title = "Hello, Dan";
};
// 리액트가 이 배열을 같은 배열이라고 인식할 수 있을까?
```

리액트가 DOM간 비교를 했던 것처럼 effect도 비교를 하면?

→ 불가능합니다.. 함수를 호출하기 전까지는 함수가 어떤 일을 하는지 모르기 때문입니다.

따라서 dependecy 배열을 사용합니다!!

```jsx
useEffect(() => {
  document.title = "Hello, " + name;
}, [name]); // 우리의 의존성
```

이건 마치 우리가 리액트에게 “이봐, 네가 이 함수의 *안을* 볼 수 없는 것을 알고 있지만, 랜더링 스코프에서 `name` 외의 값은 쓰지 않는다고 약속할게.” 라고 말하는 것과 같습니다.

```jsx
const oldEffect = () => {
  document.title = "Hello, Dan";
};
const oldDeps = ["Dan"];
const newEffect = () => {
  document.title = "Hello, Dan";
};
const newDeps = ["Dan"];
// 리액트는 함수 안을 살펴볼 수 없지만, deps를 비교할 수 있다.
// 모든 deps가 같으므로, 새 이펙트를 실행할 필요가 없다.
```

랜더링 사이에 의존성 배열 안에 있는 값이 하나라도 다르다면 이펙트를 스킵할 수 없습니다. 모든 것을 동기화해야죠!\*\*\*\*

### 리액트에게 의존성으로 거짓말하지 마라

deps를 지정한다면, **컴포넌트에 있는 *모든* 값 중 그 이펙트에 사용될 값은 *반드시* 거기 있어야 한다**는 것을 기억해 두세요. props, state, 함수들 등 컴포넌트 안에 있는 모든 것 말입니다.

예를 들어

데이터 불러오는 로직이 무한 루프에 빠질 수도 있고

소켓이 너무 자주 반응할 수 도 있습니다..

### 의존성으로 거짓말을 하면 생기는 일

만약 deps가 이펙트에 사용하는 모든 값을 가지고 있다면, 리액트는 언제 다시 이펙트를 실행해야 할 지 알고 있습니다.

→ 의존성을 비교함

#1.

예를 들어서, [name]이라는 의존성 배열을 넣어두면

첫번째 렌더 때는 의존성 배열의 값이 [’stella’]일 것이고

두번째 렌더 떄 값이 바뀌었다면, [’chan’]일 것이다.

값이 바뀌었기 때문에 effect를 실행할 것이다.

#2.

하지만 의존성 배열이 []이었다면

첫번째 렌더 때 []이고

두번쨰 렌더 때도 []일테니까

effect를 실행하지 않을 것이다.

또 다른 예시,

예를 들어 매 초마다 숫자가 올라가는 카운터를 작성한다고 해 보겠습니다. 클래스 컴포넌트의 개념을 적용했을 때 우리의 직관은 “인터벌을 한 번만 설정하고, 한 번만 제거하자” 가 됩니다.

라는 생각으로 코드를 만들면 다음과 같다.

```jsx
function Counter() {
  const [count, setCount] = useState(0);
  useEffect(() => {
    const id = setInterval(() => {
      setCount(count + 1);
    }, 1000);
    return () => clearInterval(id);
  }, []);
  return <h1>{count}</h1>;
}
```

결과: 숫자가 오로지 한 번만 증가합니다.

왜?: count를 사용하지만 의존성 배열을 []이라고 정의하면서 거짓말을 했습니다!

⇒ 첫번째 랜더링에서 `count`는 0입니다. 첫 번째 랜더링의 effect에서 setCount(0+1)이라는 뜻이 됩니다. 하지만, 의존성 배열을 []이라고 했기 때문에 effect를 다시 실행하지 않고, 결국 그로 인해 매초마다 setCOunt(0+1)을 호출하는 것입니다.

⇒ 의존성이 같으므로 setCount(1+1)을 스킵해버림!!!

### 의존성을 솔직하게 적는 2가지 방법

첫번째 방법 실행 후 안되면 두번째 방법 사용!

1. 컴포넌트 안에 있으면서 effect에 사용되는 모든 값이 의존성 배열 안에 포함되도록!
   - 위의 코드를 이렇게 고치면
     ```jsx
     useEffect(() => {
       const id = setInterval(() => {
         setCount(count + 1);
       }, 1000);
       return () => clearInterval(id);
     }, [count]);
     ```
     의존성 배열이 올바르게 만들어졌음!
     하지만,, count 값이 바뀔 때마다 인터벌은 해제되고 다시 설정될 것임
2. effect의 코드를 바꿔서 우리가 원하던 것보다 자주 바뀌는 값을 요구하지 않도록 만드는 것

   → 의존성에 대해서 거짓말하지 말고 의존성을 더 적게 넘겨주면 됨!

   - 우리가 무엇 때믄에 `count`를 쓰고 있나요?
     ⇒ 오로지 `setCount`만을 위해서 사용중입니다
     그렇다면 함수 형태의 업데이터를 사용한다면? 필요가 없어집니다!
     `setCount(c => c+1)`
   - 잘못된 의존관계
   - 이제 effect가 한 번만 실행되었다고 하더라도 첫 번째 랜더링에 포함되는 인터벌 콜백은 인터벌이 실행될 때마다 `c ⇒ c + 1`이라는 업데이트 지침을 전달하니까 `count`의 상태를 알 필요가 없습니다!
     리액트가 이미 알고 있으니까요

### 함수형 업데이트와 구글 닥스

동기화에 대해 생각할 때 흥미로운 부분은 종종 시스템간의 “메세지”를 상태와 엮이지 않은 채로 유지하고 싶을 때가 있다는 것입니다. 예를 들어 구글 닥스에서 문서를 편집하는 것은 실제로 서버로 *전체* 페이지를 보내는 것이 아닙니다. 그렇다면 굉장히 비효율적이겠죠. 그 대신 사용자가 무엇을 하고자 했는지 표현한 것을 보냅니다.

> 🔥 **오로지 필요한 최소한의 정보를 이펙트 안에서 컴포넌트로 전달하는게 최적화에 도움이 됩니다.**

현재의 카운트 값에 “오염되지” 않기 때문입니다. 그저 행위(“증가”)를 표현할 뿐입니다. 리액트로 생각하기 문서에 최소한의 상태를 찾으라는 내용이 포함되어 있습니다. 그 문서에 쓰인 것과 같은 원칙이지만 업데이트에 해당된다고 생각하세요.

여러 소스에서 이루어지는 업데이트가 (이벤트 핸들러, 이펙트 구독 등) 예측 가능한 방식으로 모여서 정확히 적용될 수 있도록 보장합니다.

`setCount(c ⇒ c + 1)`도 그렇게 좋은 방법은 아닙니다. 예를 들어서 서로에게 의존하는 두 상태 값이 있거나 props 기반으로 다음 상태를 계산할 필요가 있을 때 도움이 안됩니다.

그럴 때 사용할 만한 것이 `useReducer`입니다.

### 액션을 업데이트로부터 분리하기

두 가지 상태 변수를 사용하는 예제

```jsx
function Counter() {
  const [count, setCount] = useState(0);
  const [step, setStep] = useState(1);
  useEffect(() => {
    const id = setInterval(() => {
      setCount((c) => c + step);
    }, 1000);
    return () => clearInterval(id);
  }, [step]);
  return (
    <>
      <h1>{count}</h1>
      <input value={step} onChange={(e) => setStep(Number(e.target.value))} />
    </>
  );
}
```

→ `step`을 effect 안에서 사용하고 있기 때문에 dependency 배열에 추가해뒀습니다.

→ `step`이 변경되면 인터벌을 다시 시작합니다.


> 🔥 effect를 분해하고 새로 설정하는데는 아무 문제가 없고, 특별히 좋은 이유가 있지 않다면 분해하는 것을 피하지 말아야 합니다



**하지만**

step이 바뀐다고 인터벌 시계가 초기화되지 않도록 하고 싶다면? effect의 dependency 배열에서 step을 제거하려면 어떻게 해야할까요?

⇒ 두 상태 변수 모두 useReducer로 교체해야할 수 있습니다.

`useReducer`는 컴포넌트에서 일어나는 “액션”의 표현과 그 반응으로 상태가 어떻게 업데이트되어야 할지를 분리합니다.

예를 들어서, 위의 코드를 effect 안에서 의존성을 dispatch로 바꿔보겠습니다.

```jsx
const [state, dispatch] = useReducer(reducer, initialState);
const { count, step } = state;
useEffect(() => {
  const id = setInterval(() => {
    dispatch({ type: "tick" }); // setCount(c => c + step) 대신에
  }, 1000);
  return () => clearInterval(id);
}, [dispatch]);
```

⇒ **리액트는 컴포넌트가 유지되는 한 `dispatch` 함수가 항상 같다는 것을 보장합니다.** 따라서 위의 예제에서 인터벌을 다시 구독할 필요조차 없습니다.

(리액트가 `dispatch`, `setState`, `useRef` 컨테이너 값이 항상 고정되어 있다는 것을 보장하니까 의존성 배열에서 뺄 수도 있습니다. 하지만 명시한다고 해서 나쁠 것은 없습니다.)

effect 안에서 상태를 읽는 대신 무슨 일이 일어났는지를 알려주는 정보를 인코딩하는 **액션**을 디스패치합니다. 이렇게 하여 effectsms `step` 상태로부터 분리됩니다.

effect는 어떻게 상태를 업데이트 << **무슨 일이 일어났는지 알려줍니다**. 그리고 리듀서가 업데이트 로직을 모아둡니다.

<리듀서>

```jsx
const initialState = {
  count: 0,
  step: 1,
};

function reducer(state, action) {
  const { count, step } = state;
  if (action.type === "tick") {
    return { count: count + step, step };
  } else if (action.type === "step") {
    return { count, step: action.step };
  } else {
    throw new Error();
  }
}
```

### 왜 useReducer가 Hooks의 치트 모드인가

다음 상태를 계산하는데 props가 필요하다면? 예를 들어 API가 `<Count step={1} />` 인거죠.

⇒ 해결법:

1. 의존성으로 props.state 사용
2. 리듀서 그 자체를 컴포넌트 안에 정의하여 props를 읽도록 하면 됨

   ```jsx
   function Counter({ step }) {
     const [count, dispatch] = useReducer(reducer, 0);
     function reducer(state, action) {
       if (action.type === "tick") {
         return state + step;
       } else {
         throw new Error();
       }
     }
     useEffect(() => {
       const id = setInterval(() => {
         dispatch({ type: "tick" });
       }, 1000);
       return () => clearInterval(id);
     }, [dispatch]);
     return <h1>{count}</h1>;
   }
   ```

   **🚧 주의 🚧**
   이 패턴은 몇 가지 최정화를 무효화합니다. 어디서나 쓰진 마세요!!

   다른 랜더링에 포함된 이펙트 안에서 호출된 리듀서가 props를 알고 있지? 답은 `dispatch`에 있습니다.

   리액트는 그저 액션을 기억해놓습니다. 하지만 다음 랜더링 중에 리듀서를 호출할 것입니다. 이 시점에서 새 props가 스코프 안으로 들어오고 effect 내부와는 상관이 없어집니다.

   
   > 🔥 `**useReducer` 를 Hooks의 “치트 모드”** 
   업데이트 로직과 그로 인해 무엇이 일어나는지 서술하는 것을 분리할 수 있도록 만들어줍니다. 그 다음은 이펙트의 불필요한 의존성을 제거하여 필요할 때보다 더 자주 실행되는 것을 피할 수 있도록 도와줍니다.

   

### 함수를 이펙트 안으로 옮기기


> 🔥 **함수는 의존성에 포함되면 안됩니다.**
간단히 로컬 함수를 의존성에서 제외하는 해결책은 컴포넌트가 커지면서 모든 경우를 다루고 있는지 보장하기 아주 힘들다는 문제가 있습니다!



```jsx
function SearchResults() {
  const [query, setQuery] = useState("react");
  // 이 함수가 길다고 상상해 봅시다
  function getFetchUrl() {
    return "https://hn.algolia.com/api/v1/search?query=" + query;
  }
  // 이 함수가 길다고 상상해 봅시다
  async function fetchData() {
    const result = await axios(getFetchUrl());
    setData(result.data);
  }
  useEffect(() => {
    fetchData();
  }, []);
  // ...
}
```

이런 함수를 사용하는 어떤 effect에서 deps를 업데이트 하는 것을 깜빡했다면 effect는 proprhk state의 변화에 동기화하는데 실패할 것입니다.

**해결법:**

어떠한 함수를 이펙트 안에서만 쓰다면, 그 함수를 직접 effect 안으로 옮기세요!!

```jsx
function SearchResults() {
  // ...
  useEffect(() => {
    // 아까의 함수들을 안으로 옮겼어요!
    function getFetchUrl() {
      return "https://hn.algolia.com/api/v1/search?query=react";
    }
    async function fetchData() {
      const result = await axios(getFetchUrl());
      setData(result.data);
    }
    fetchData();
  }, []); // ✅ Deps는 OK
  // ...
}
```

⇒ **옮겨지는 의존성**에 신경 쓸 필요가 없습니다.

의존성 배열은 거짓말하지 않습니다. 왜냐하면 effect 안에서 컴포넌트의 범위 바깥에 있는 어떤 것도 사용하지 않기 때문입니다.

바깥에 있는 state를 써야한다면, effect 안에 있는 함수만 고치고 의존성으로 추가해주면 됩니다.

```jsx
function SearchResults() {
  const [query, setQuery] = useState("react");
  useEffect(() => {
    function getFetchUrl() {
      return "https://hn.algolia.com/api/v1/search?query=" + query;
    }
    async function fetchData() {
      const result = await axios(getFetchUrl());
      setData(result.data);
    }
    fetchData();
  }, [query]); // ✅ Deps는 OK
  // ...
}
```

### 하지만 이 함수를 이펙트 안에 넣을 수 없어요

예를 들어, 한 컴포넌트에서 여러 개의 이펙트가 있는데 같은 함수를 호출할 때, 로직을 복붙하고 싶진 않겠죠. 아니면 prop 때문에.


> 🔥 흔한 오해 중 하나가 “함수는 절대 바뀌지 않는다” 입니다. 하지만 이 글을 통해 배웠듯, 그 말이 진실로부터 얼마나 멀리 떨어져있는지 알 수 있습니다. **사실은, 컴포넌트 안에 정의된 함수는 매 랜더링마다 바뀝니다!**



**하지만 그로 인해 문제가 발생합니다.**

예를 들어, 두 effect가 `getFetchUrl`을 호출한다고 하면,

```jsx
function SearchResults() {
  function getFetchUrl(query) {
    return "https://hn.algolia.com/api/v1/search?query=" + query;
  }
  useEffect(() => {
    const url = getFetchUrl("react");
    // ... 데이터를 불러와서 무언가를 한다 ...
  }, []); // 🔴 빠진 dep: getFetchUrl
  useEffect(() => {
    const url = getFetchUrl("redux");
    // ... 데이터를 불러와서 무언가를 한다 ...
  }, []); // 🔴 빠진 dep: getFetchUrl
  // ...
}
```

→ `getFetchUrl`을 각각의 effect 안으로 옮기게 되면 로직을 공유할 수 없으니까 안 좋음

**해결책**

1. **👿 안 좋은 방법**

   `getFetchUrl`을 의존성 배열에서 빼버리는 것

   ⇒ 언제 effect에 의해 다루어질 필요가 있는 데이터 흐름에 변화를 더해야할지 알아차리기 어려움

2. **😇 괜찮은 방법**

   1. 함수가 컴포넌트 스코프 안의 어떠한 것도 사용하지 않는다면, 컴포넌트 외부로 끌어올려두고 이펙트 안에서 자유롭게 사용하면 됩니다.

   but.. 이펙트는 단순하게 만드는게 좋고 그 안에 콜백은 두는 것은 좋지 않음 ㅠ

   ```jsx
   // ✅ 데이터 흐름에 영향을 받지 않는다
   function getFetchUrl(query) {
     return "https://hn.algolia.com/api/v1/search?query=" + query;
   }
   function SearchResults() {
     useEffect(() => {
       const url = getFetchUrl("react");
       // ... 데이터를 불러와서 무언가를 한다 ...
     }, []); // ✅ Deps는 OK
     useEffect(() => {
       const url = getFetchUrl("redux");
       // ... 데이터를 불러와서 무언가를 한다 ...
     }, []); // ✅ Deps는 OK
     // ...
   }
   ```

   1. `useCallback` 사용

      ```jsx
      function SearchResults() {
        // ✅ 여기 정의된 deps가 같다면 항등성을 유지한다
        const getFetchUrl = useCallback((query) => {
          return "https://hn.algolia.com/api/v1/search?query=" + query;
        }, []); // ✅ 콜백의 deps는 OK
        useEffect(() => {
          const url = getFetchUrl("react");
          // ... 데이터를 불러와서 무언가를 한다 ...
        }, [getFetchUrl]); // ✅ 이펙트의 deps는 OK
        useEffect(() => {
          const url = getFetchUrl("redux");
          // ... 데이터를 불러와서 무언가를 한다 ...
        }, [getFetchUrl]); // ✅ 이펙트의 deps는 OK
        // ...
      }
      ```

      → `useCallback:`

      의존성 체크에 레이어를 하나 더 더하는 것입니다.
      문제를 다른 방식으로 해결하는데, 함수의 의존성을 피하기보다 함수 자체가 필요할 때만 바뀔 수 있도록 만드는 것입니다.

   **예시1: 함수를 다른 effect에서 불러오기**

   `getFetchUrl`을 사용하는 어떤 effect라도 `query`가 바뀔 때마다 다시 실행될 것입니다.

   ⇒ `useCallback` 덕분에 `query` 가 같다면, `getFetchUrl` 또한 같을 것이며, 이펙트는 다시 실행되지 않을 것입니다. 하지만 만약 `query` 가 바뀐다면, `getFetchUrl` 또한 바뀌며, 데이터를 다시 페칭할 것입니다.

   ```jsx
   function SearchResults() {
     const [query, setQuery] = useState("react");
     // ✅ query가 바뀔 때까지 항등성을 유지한다
     const getFetchUrl = useCallback(() => {
       return "https://hn.algolia.com/api/v1/search?query=" + query;
     }, [query]); // ✅ 콜백 deps는 OK
     useEffect(() => {
       const url = getFetchUrl();
       // ... 데이터를 불러와서 무언가를 한다 ...
     }, [getFetchUrl]); // ✅ 이펙트의 deps는 OK
     // ...
   }
   ```

   **예시2: 부모로부터 함수 prop을 내려보내는 경우**

   ```jsx
   function Parent() {
     const [query, setQuery] = useState("react");
     // ✅ query가 바뀔 때까지 항등성을 유지한다
     const fetchData = useCallback(() => {
       const url = "https://hn.algolia.com/api/v1/search?query=" + query;
       // ... 데이터를 불러와서 리턴한다 ...
     }, [query]); // ✅ 콜백 deps는 OK
     return <Child fetchData={fetchData} />;
   }
   function Child({ fetchData }) {
     let [data, setData] = useState(null);
     useEffect(() => {
       fetchData().then(setData);
     }, [fetchData]); // ✅ 이펙트 deps는 OK
     // ...
   }
   ```

   ⇒ `fetchData` 는 오로지 `Parent` 의 `query` 상태가 바뀔 때만 변하기 때문에, `Child` 컴포넌트는 앱에 꼭 필요할 때가 아니라면 데이터를 다시 페칭하지 않을 것입니다.\*\*\*\*

### 함수도 데이터 흐름의 일부인가?


> 🔥 **`useCallback` 을 사용하면, 함수는 명백하게 데이터 흐름에 포함됩니다.** 만약 함수의 입력값이 바뀌면 함수 자체가 바뀌고, 만약 그렇지 않다면 같은 함수로 남아있다고 말 할 수 있습니다.



`useMemo` 또한 복잡한 객체에 대해 같은 방식의 해결책을 제공합니다.

```jsx
function ColorPicker() {
  // color가 진짜로 바뀌지 않는 한
  // Child의 얕은 props 비교를 깨트리지 않는다
  const [color, setColor] = useState("pink");
  const style = useMemo(() => ({ color }), [color]);
  return <Child style={style} />;
}
```
