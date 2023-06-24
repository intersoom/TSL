## 클로저

### 어휘적 환경

- 코드가 실행한 변수들이 `Lexical 환경`(전역)에 올라가게 됨
  - 즉, 호이스팅됨
  - 다만, let / const로 선언된 변수들은 초기화가 되지 않아서 바로 사용할 수 없음
    함수는 바로 초기화됨
- 함수가 실행될 때 새로운 `Lexical 환경`(내부)이 생기게 됨
  - 내부 `Lexical 환경`은 외부 `Lexical 환경`에 대한 참조를 갖는다
- **코드에서 변수를 찾는 과정:**
  1. 내부 `Lexical 환경`에서 변수를 찾고
  2. 없으면, 외부에서 찾고 (어떤 함수의 외부 `Lexical 환경`이 최상위라면 그건 곧 전역 `Lexical 환경`)
  3. 없으면, 전역에서 찾음
- 함수의 `Lexical 환경`에는 매개변수, 지역 변수들이 저장됨
- **예시:**

  ```jsx
  function makeAdder(x) {
    return function (y) {
      // -> y를 가지고 있고 상위함수인 makeAdder의 x에 접근 가능
      return x + y;
    };
  }

  const add3 = makeAdder(3);
  console.log(add3(2)); // -> add3 함수가 생성된 이후에도 상위함수인 makeAdder의 x에 접근 가능
  ```

  > **3** -참조→ **2** -참조→ **1**

  1. 전역 `Lexical 환경`
     - makeAdder: function
     - add3: function
  2. makeAdder `Lexical 환경`
     - x: 3
  3. 익명함수 `Lexical 환경`
     - y: 2

### 클로저

- **정의:** 함수와 `Lexical 환경`의 조합, 함수가 생성될 당시의 외부 변수를 기억하고 생성 이후에도 계속 접근이 가능한 것
    <aside>
    👉🏻 **위의 예시에서..**
    add3 함수가 생성된 이후에도 상위함수인 makeAdder의 x에 접근 가능한 것처럼
    
    </aside>

- **추가 예시(1):**

  ```jsx
  function makeAdder(x) {
    return function (y) {
      return x + y;
    };
  }

  const add3 = makeAdder(3);
  console.log(add3(2)); // 15
  const add10 = makeAdder(10);
  console.log(add10(5)); // 15
  console.log(add3(1)); // 4
  ```

  → `add10()`과 `add3()`은 서로 다른 환경을 보유하고 있음

- **추가 예시(2):**

  ```jsx
  function makeCounter() {
    let num = 0;

    return function () {
      return num++;
    };
  }

  let counter = makeCounter();

  console.log(counter()); // 0
  console.log(counter()); // 1
  console.log(counter()); // 2
  ```

  → **은닉화 성공 !**
  `num`에 접근이 불가함, 100씩 증가하는 것처럼 임의로 증가하는 수를 변경할 수도 없음
