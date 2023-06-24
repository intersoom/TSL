## 호이스팅

> **호이스팅이란?**
> 스코프 내부 어디서든 변수 선언은 최상위에 선언된 것처럼 행동

### var, let, const

- `var`: 선언하기 전에 사용할 수 있다
  - 🤔 **왜?** 호이스팅되기 때문에
  - 선언은 호이스팅 되지만, 할당은 호이스팅 되지 않음
    → 선언 전에 console에 찍어보면 `undefined` 나옴
- `let`, `const`: 선언하기 전에 사용할 수 없다
  - 🤔 **왜?**
    호이스팅은 되지만, TDZ(Temporal Dead Zone)의 영향을 받기 때문에
        → TDZ의 영역에 존재하는 코드들은 사용할 수 없음

        → 할당을 하기 전에 사용할 수 없음

        → 이는 코드를 예측 가능하게 하고, 잠재적인 버그를 줄임

### 호이스팅

- **정의:**
  스코프 내부 어디서든 변수 선언은 최상위에 선언된 것처럼 행동
- 호이스팅은 스코프 단위로 일어남
  **😇 문제가 없는 코드:**
  ```jsx
  let age = 30;

  function showAge() {
    console.log(age);
  }

  showAge();
  ```
  **👿 문제가 있는 코드:**
  ```jsx
  let age = 30;

  function showAge() {
    console.log(age); // error: TDD
    let age = 20;
  }

  showAge();
  ```
  → 호이스팅은 스코프 단위로 발생하기 때문에 age가 호이스팅 되면서 `console.log(age)` 부분은 TDZ 영역에 포함됨
  → 호이스팅 되지 않았다면 바깥에서 선언한 age = 30이 정상적으로 찍혀야함

### 변수

- **변수의 생성과정**
  1. 선언 단계
  2. 초기화 단계: undefined를 할당해주는 단계
  3. 할당 단계
- **종류에 따른 생성과정:**
  - **var**
    1. 선언 및 초기화 단계
    2. 할당 단계
  - **let**
    1. 선언 단계 (호이스팅)
    2. 초기화 단계 (실제 코드에 도달했을 때 → 그 전에 호출시, 레퍼런스 에러)
    3. 할당 단계
  - **const**
    1. 선언 + 초기화 + 할당
- **스코프**
  - **var:**
    `함수 스코프` 내에서 선언한 변수는 함수 스코프 내에서만 사용할 수 있음
    if문 같은 블록 스코프는 안에서 선언한 변수도 외부에서 접근 가능
  - **let, const:**
    `블록 스코프` (함수, if, for, while, try/catch) 내에서 선언한 변수는 블록 스코프 내에서만 사용할 수 있음