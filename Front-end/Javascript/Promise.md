### **비동기 작업**

특정 코드의 로직이 끝날 때까지 기다리지 않고, 나머지 코드를 먼저 실행하는 것

- **비동기 작업은 웹 개발에 많이 사용됨**
    
    서버에서 데이터를 불러올 때 오래 걸릴 수도 있는데, 그 동안 다른 코드를 실행하지 않고 가만히 기다리면 로딩이 오래 걸리기 때문에
    
- **비동기 작업**을 **순차적**으로 실행하는 경우 존재
    - **예시**
        
        연산이 끝나기 전에 연산 결과를 파라미터로 사용하는 함수를 실행하면 에러남
        
    - **해결법**
        - **콜백 함수**
            
            특정 로직이 끝났을 때 원하는 코드를 실행할 수 있음
            
            하지만, 콜백에 또 콜백을 부르면 코드가 복잡해지고 에러 처리도 힘들어짐
            
        - **Promise**
            
            콜백의 단점을 Promise를 통해서 해결할 수 있음
            

### Promise

- **Promise란?**
    
    비동기 함수가 반환하는 객체의 종류
    
    - 함수의 성공 또는 실패 상태를 알려줌
    - 콜백을 직접 호출하는 방식 대신 Promise로 콜백을 부를 수 있음
    - **장점:**
        - 비동기 처리 시점 확인 가능
        - 비동기 함수의 결과를 쉽게 확인 가능
        - 에러 어디서 발생했는지 쉽게 확인 가능
- **Promise를 생성하는 방법**
    
    ```jsx
    function requestPayment(paymentData) {
    	// ...
    	return new Promise((resolve, reject) {    // Pending 상태
    		if(isSuccess) {
    			resolve(data)                         // 성공 상태
    		}
    		else {
    			reject(error)                         // 실패 상태
    		}
    	})
    }
    ```
    
    ---
    
    **✅ 이해한 내용 추가**
    
    - requestPayment 함수 자체는 동기적으로 실행되니까 return까지는 빠르게 도달해서 비동기적으로 응답을 기다림 → 이 상태가 pending
    - 응답이 왔을 때, 성공이면 resolve 실행 / 실패이면 reject 실행
    
    ---
    
    - **결제 요청 전**
        - Promise가 **대기(pending)** 상태
    - **결제 요청 성공**
        - Promise가 **fulfilled**된 상태
        - Promise 생성 함수에 있는 `resolve()` 메서드가 호출됨
        - Promise가 성공 상태로 바뀌고 `data`값을 가지게 됨
    - **결제 요청 실패**
        - Promise가 **rejected**된 상태
        - Promise 생성 함수에 있는 `reject()` 메서드가 호출됨
        - Promise가 실패 상태로 바뀌고 `error` 데이터를 가지게 됨
- **Promise의 상태들**
    - **대기(Pending):** 비동기 함수가 아직 시작하지 않은 상태
    - **성공(Fulfilled):** 비동기 함수가 성공적으로 완료된 상태
    - **실패(Rejected):** 비동기 함수가 실패한 상태
    </br>
    <img width="484" alt="스크린샷 2023-10-24 오전 3 37 16" src="https://github.com/intersoom/TSL/assets/78731710/eb5034aa-9a40-40f3-b168-d9fb874b3b63">

    
- **Promise를 처리하는 방법**
    - Promise를 처리할 때는 `then()` 또는 `catch()` 메서드를 사용
    - 각 메서드의 파라미터에는 콜백함수를 넣음
        - 요청 성공 상태의 Promise가 반환되면 `then()` 메서드가 호출됨
        - 요청 실패 상태의 Promise가 반환되면 `catch()` 메서드가 호출됨


    ```jsx
    paymentWidget.requestPayment({
      orderId: "t9JI0Bs1SVdJxRs8yjiQJ",            
      orderName: "토스 티셔츠 외 2건",                    
    })
    .then(function (data) {
      console.log(data);
    })
    .catch(function (error) {
      if (error.code == "NEED_CARD_PAYMENT_DETAIL") {
        console.log(error.message);
      }
    ```
