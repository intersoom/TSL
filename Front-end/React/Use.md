### 기본적인 비동기 함수 처리의 문제점

- 데이터 페칭을 위해서 반드시 hook을 사용해야함
- **Data fetching 라이브러리**
    - useQuery
- Suspense
    - Suspense + ErrorBoundary 활용
        
        → 데이터가 로딩 된 상태만 고려할 수 있게 됨
        
    - React-Query에서
        
        → Suspense 옵션 `true`로 바꿔주면 간단히 사용 가능
        
- 🤔 **문제점**
  
    
    바로 `fetch`하지 못하고 우리는 왜 **hook**을 사용해야할까?
    
    그냥 `fetch`를 바로 하면 안될까?ㅎ
    
    → 😂 이는 **불가능**함, `fetch` 함수는 `async` 함수일 수 없기 때문에
    
- **💡 해결책**
    
    Promise 결과를 동기적으로 꺼낼 수 있는 함수가 있다면?!
    
    → **use**
    
    ```jsx
    const promise = fetch('/api/data');
    const data = use(promise);
    ```
    

## use란?

- React에서 새로 제공하자 하는 hook
- **기능**
    - Promise의 일급 지원
    - async / await의 함수화
    - Suspense를 트리거 해주는 역할
        
        ```jsx
        <Suspense fallback={<div>Loading...</div>}>
        	<MyApp /> // use(promise)
        </Suspense>
        ```
        
        → `MyApp`이 렌더링 되는 과정에서 아직 `resolve`되지 않은 `promise`를 `use`를 하게 되면 `Suspense fallback`이 렌더링되는 형태
        
- **형태**
    
    ```jsx
    function use<T>(promise: Promise<T>): T
    ```
    
- **특징**
    - 조건부로 호출이 가능함!!
- **동작 원리**
    - 공식적으로 구현체는 정해지지 않음
    - **다른 라이브러리들은 어떻게 Suspense를 활용하고 있을까?**
        - unstable한 방식을 이용해서 임의의 함수가 promise를 throw하는 방식을 통해서 지원중
        - **Jotai에서 use 사용하는 방식**
            
            ```jsx
            import React from 'react'
            
            const use = React.use ||
            (promise) => {
            	if (promise.status === 'pending') {
            		return promise
            	} else if (promise.status === 'fulfilled') {
            		return promise.value as T
            	} else if (promise.status === 'rejected') {
            		return promise.reason
            	} else {
            		// 생략 (Promise의 result를 읽기 위한 코드)
            	}
            }
            ```
            

### 예제

- **기본적인 형태**
    - 유저의 인벤토리에서 아이템을 찾아주는 검색 기능
    - 하지만 inventory에는 아이템 = 코드로 작성되어있음
    - `useNormalItems`, `useEventItems`는 코드와 이름을 매칭할 수 있는 정보가 들은 object를 fetch해오는 Hook임
        
        ```jsx
        const useNormalItems = () => {
        	return useQuery(fetchNormalItems);
        }
        ```
        
    - **코드**
        
        ```jsx
        function useInventory({userId, search}) {
        	const { inventory } = useUserInfo(userId);
        	const normalItems = useNormalItems();
        	const eventItems = useEventItems();
        
        	return inventory
        	.filter((item) => {
        		if (!search) return true;
        		if (Normal Item이면) normalItems에서 이름 체크;
        		if (Event Item이면) eventItems에서 이름 체크;
        	return false
        	})
        }
        ```
        
    - **😡 Hook의 제약으로 인한 문제점**
        - 불필요한 Blocking → TTI(Time To Interactive: 유저가 인터랙션이 가능할 때까지의 시간) 증가 → UX 저하
        - 코드 응집도 저하 → DX 저하
- **문제점**
    1. **불필요한 Blocking**
        
        ```jsx
        function useInventory({userId, search}) {
        	const { inventory } = useUserInfo(userId);
        	const normalItems = useNormalItems();
        	const eventItems = useEventItems();
        
        	return inventory
        	.filter((item) => {
        		if (!search) return true;
        		if (Normal Item이면) normalItems에서 이름 체크;
        		if (Event Item이면) eventItems에서 이름 체크;
        	return false
        	})
        }
        ```
        
        → 해당 코드에서 만약 `search` 값이 없는 경우(초기 상태),
        
        ```jsx
        const normalItems = useNormalItems();
        const eventItems = useEventItems();
        ```
        
        해당 파트에서 블로킹이 발생하면서 TTI가 증가되었음에도 불구하고
        
        ```jsx
        if (!search) return true;
        ```
        
        이 때, 코드가 종료 되면서 로드 한 `normalItems`, `eventItems` 값이 전혀 쓰이지 않음
        
    2. **응집도 저하**
        
        ```jsx
        function useInventory({userId, search}) {
        	const { inventory } = useUserInfo(userId);
        	**const normalItems = useNormalItems(); // 로딩 
        	const eventItems = useEventItems(); // 로딩**
        
        	return inventory
        	.filter((item) => {
        		if (!search) return true;
        		**if (Normal Item이면) normalItems에서 이름 체크; // 사용
        		if (Event Item이면) eventItems에서 이름 체크; // 사용**
        	return false
        	})
        }
        ```
        
        `로딩하는 파트` ↔ `사용하는 파트`가 굉장히 떨어져있음 (응집도가 떨어짐)
        
        실제 개발에서 아이템 종류가 훨씬 늘어나게 된다면? 응집도가 **더** 떨어짐
        
        → **수백 종류 리소스 + 수십 종류 페이지 = 🤯**
        
        - **원인**
            
            **Hook은 최상단에서만 호출해야한다**라는 제약 조건 때문에 😡
            

### **해결**

- **Hook을 쓰지 못하는 곳**
    - 조건부, 반복문
    - return문 다음
    - 이벤트 핸들러
    - 클래스 컴포넌트
    - useMemo, useReducer, useEffect에 전달한 클로저
- **use를 쓸 수 있는 곳**
    - 조건부, 반복문
    - return문 다음

실제로 리소스 데이터가 쓰이는 곳에서 use를 활용해서 데이터를 불러옴

```jsx
function useInventory({userId, search}) {
	const { inventory } = useUserInfo(userId);
	
	return inventory
	.filter((item) => {
		if (!search) return true;
		**if (Normal Item이면) use(fetchNormalItems)에서 이름 체크; 
		if (Event Item이면) use(fethcEventItems)에서 이름 체크;** 
	return false
	})
}
```

- **결과**
    - Top Level Hook 제거 → DX 향상
    - 필요한 순간 리소스 로딩 → UX 향상

### use의 문제점

1. **중복 fetch**
    
    ```jsx
    function useInventory(...){
    	...
    	use(fetchNormalItems());
    	...
    }
    ```
    
    - **해당 코드의 버그 🐛**
        
        resolve → 리렌더 → fetch → 🔄
        
    - **해결법**
        - `cache` 추가 (아직 추가되지 않은 기능)
            
            ```jsx
            const fetchNormalItems = cache(()=>{
            	return fetch('res/normal-items');
            })
            ```
            
            - 동일한 promise 인스턴스 관리
            - + promise 수명 관리 (refetch)
        - `lodash.moize` 사용
2. **Request Waterfall**
    - 불필요하게 순차적으로 로딩 되는 문제 (엄밀히 따지면 request waterfall 현상에서 발생하는 문제)
    - **예제**
        - ✅ normal items과 event items는 동시에 로딩되는게 이상적
        - ❌ 순차적으로 로딩되는 것은 안 좋음
        
    - **해결책**
        - 보통 Data Fetching Library에서는 `Prefetch`, `Parellel Query` 사용
        - `**Prefetch` 구현**
            - fetch를 사전에 한 번 더 해주면 됨!
                
                ```jsx
                function useInventory({userId, search}) {
                	const { inventory } = useUserInfo(userId);
                	**fetchNormalItems(); fetchEventItems();** 
                
                	return inventory
                	.filter((item) => {
                		if (!search) return true;
                		if (Normal Item이면) use(fetchNormalItems)에서 이름 체크; 
                		if (Event Item이면) use(fetchEventItems)에서 이름 체크; 
                	return false
                	})
                }
                ```
                
                해당 부분에서 부분에서 `Blocking`은 발생하지 않음 ❌
                
                BUT
                
                응집성 문제가 재 발생함
                
            - **Dynamic Prefetch**
                
                → `prefetch` 대상을 런타임에 결정함
                
                ```jsx
                const fetchNormalItems = () => {
                	현재 페이지에서 normalItem 사용했다고 localStorage에 기록
                	return fetch('/res/normal-items');
                }
                ```
                
                ```jsx
                document.onready = () => {
                	const names = localStorage에서 사용 기록 읽기
                	names.forEach((XXX) => fetchXXX())
                }
                ```
                
                → 페이지 접속하면 발생하는 이벤트 핸들러에 prefetch 적용
                
                **➕ 헷갈린 부분 정리**
                
                - 🤔 어느 시점에 localStorage에 기록되는 것인가?
                    
                    → 최초로 `fetchNormalItems`을 호출한 시점
                    
                - 🤔 그렇다면, 최초의 호출 시점에서는 Prefetch가 안되는가?
                    
                    → 맞다! 이 문제를 해결하기 위해서는 initial 값을 초기에 세팅해주면 된다
