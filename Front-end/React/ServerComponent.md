### SSR

- 서버에서 렌더링을 마친 페이지를 클라이언트에게 먼저 보내주는 것
- JS가 연결되지 않아서, 사용자와 상호작용이 불가능
    
    → hydration : 상호작용을 가능하게 해줌
    
### 서버 컴포넌트

- 서버에서 렌더링이 된 이후에는 추가적인 작업이 필요 없다고 느껴지는 것
- 사용자와 상호작용이 없지만, 무거운 라이브러리를 사용하는 경우 용이함

---

- **서버 컴포넌트를 사용한다면,**
    - 오직 서버에서만 실행되는 컴포넌트를 생성할 수 있음
    - 리액트 컴포넌트 내에서 데이터베이스 쿼리를 작성하는 작업을 할 수 있음
- **서버 컴포넌트의 핵심은 다시 렌더링하지 않아도 된다는 것**
    - 서버에서 한 번 렌더링 된 값은 클라이언트로 전송되어서 제자리에 고정되기 때문에 라우터 레벨의 변경이 없다는 전제 하에 변경되지 않는다.
- **경계의 문제**

    ![스크린샷 2024-03-26 오후 11 57 14](https://github.com/intersoom/TSL/assets/78731710/7ed91d57-a701-4d3b-89ed-ba8b24580fce)

    - Article이 클라이언트 컴포넌트라서 리렌더링이 필요합니다. 그러면 하위 구성 요소도 다시 렌더링 되어야 합니다.
        
        하지만, 하위 컴포넌트들이 서버 컴포넌트라면 다시 렌더링할 방법이 없습니다. 
        
    
    이를 위해서 리액트 팀이 추가한 규칙:
    
    클라이언트 컴포넌트가 임포트하는 컴포넌트는 반드시 클라이언트 컴포넌트가 되어야 한다.
    
    그래서 모든 곳에 use client를 쓸 필요는 없음 ! → 경계가 시작되는 부분에만 작성하면 됨 
    
    - 그렇다면, 상위 컴포넌트에서 상태를 사용해야 하면 모든 것이 클라이언트 컴포넌트가 되어야 하나?
        
        **NO**
        
        어떻게?
        
        **“경계”**는 부모-자식의 문제가 ❌, 소유자의 문제 ⭕️
        
        ```jsx
        'use client'
        
        function Homepage() {
        	// 상태 관련 코드들..
        	return (
        		<body style={colorVariable}>
        			<Header />
        			<MainCount />
        		</body>
        	)
        }
        ```
        →  이 때, `Homepage`는 상태를 사용해야 하니까 클라이언트 컴포넌트가 되어야 함
        
        그렇게 되면, `<Header>`와 `<MainCount>`도 암시적으로 클라이언트 컴포넌트가 됨
        
        이를 해결하기 위해서
        
        상태 관련 코드들을 새로운 컴포넌트로 빼버리면 됨 !!
        
        ```jsx
        'use client'
        
        function ColorProvider() {
        	// 상태 관련 코드들..
        	
        	return (
        		<body style={colorVariable}>
        			{children}
        		</body>
        	)
        }
        ```
        
        ```jsx
        function Homepage() {
        	return (
        		<ColorProvider>
        			<Header />
        			<MainCount />
        		</ColorProvider>
        	)
        }
        ```
        
        → 더 이상 `Homepage`에는 `'use client'` 문이 필요 없다!
        
        → `Homepage`가 Header, `MainCount`를 소유하고 있는 것이기 때문에 `ColorProvider`가 부모든 말든 상관 없음!
        

### SSR vs 서버 컴포넌트

- **SSR**
    
    ```jsx
    import marked from "marked" // 35.9K
    import sanitizeHtml from "sanitize-html" // 206K
    
    function NoteWithMarkdown({text}){
    	cont html = sanitizeHtml(marked(text));
    	return (/* render */);
    }
    ```
    
    1. 컴포넌트 렌더링 화면을 클라이언트에 보내고
    2. hydration을 위해서 `marked`, `sanitizeHtml` 라이브러리를 또 보냄
- **Server Component**
    
    ```jsx
    import marked from "marked" // zero bundle
    import sanitizeHtml from "sanitize-html" // zero bundle
    
    function NoteWithMarkdown({text}){
    	cont html = sanitizeHtml(marked(text));
    	return (/* render */);
    }
    ```
    
    → 클라이언트에서 hydration 필요 없기 때문에, bundle 아예 보내지 않음
    
    **⚠️ 주의 ⚠️**
    
    - 일반적인 컴포넌트처럼 effect, state를 가질 수 없음
    - BUT 파일 분리 잘 하면 됨 (코드 스플리팅 활용 👍🏻)
    
    - ! DB에 직접 접근 가능함 !
