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

