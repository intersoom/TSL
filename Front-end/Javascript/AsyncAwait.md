### Promise의 단점

- 콜백 함수와 유사한 단점을 보유하고 있음
- `.then()` 체인을 길게 이어 나가면 콜백 함수와 마찬가지로 코드의 가독성이 떨어지고 에러 파악이 어려워짐

### Async / Await란?

- 함수를 정의할 때 `async`를 함수 앞에 붙이면, **“이 함수는 비동기적인 함수이고 Promise를 반환한다”라고 선언하는 것**
- Promise 생성 함수가 아니어도 반환 값이 Promise를 반환함

```jsx
async function handleSubmit() {
	...
	return paymentData
	// return Promise.resolve(paymentData) // 위와 같은 결과
}
```

- `await`는 `async` 함수 안에만 사용할 수 있는 특별한 문법
    - Promise를 반환하는 함수 앞에 `await`를 붙이면, 해당 Promise의 상태가 바뀔 때까지 코드가 기다림
    - **Promise가 성공 상태 또는 실패 상태로 바뀌기 전까지는 다음 연산을 시작하지 않음**
    - 비동기적인 작업 → 동기적으로 바꿈
    
    ```jsx
    async function handleSubmit() {
      const paymentData = await paymentWidget.requestPayment({
        orderId: "KOISABLdLiIzeM-VGU_8Z", // 주문 ID(직접 만들어주세요)
        orderName: "토스 티셔츠 외 2건" // 주문명
      });
      console.log(paymentData);
    	return paymentData
    }
    ```
    
    - `await`는 `then()`과 같은 역할
    - 다만, 콜백 함수를 등록할 필요가 없기 때문에 더 편리함
    - 체이닝으로 인해 코드가 복잡해질 일도 없음

### 에러 핸들링

```jsx
async function handleSubmit() {
      try {
        const paymentData = await paymentWidget.requestPayment({
          orderId: "KOISABLdLiIzeM-VGU_8Z", // 주문 ID(직접 만들어주세요)
          orderName: "토스 티셔츠 외 2건" // 주문명
        });
		console.log(paymentData);
        // 성공 처리
        return paymentData;
      } catch (error) {
        // 에러 처리
        console.log(error.message);
      }
    }
```

- 간단히 `try/catch`를 사용하면 됨
