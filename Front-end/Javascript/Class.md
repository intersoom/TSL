## Class

- 클래스는 객체를 생성하기 위한 템플릿
- 클래스는 데이터와 메소드를 하나로 추상화함
- JavaScript의 클래스는 프로토타입을 이용해서 만들어졌지만, ES5의 클래스와는 다름
    - **관련된 추가적인 내용**
        
        *((추후에 공부해서 정리하기))*
        
        - 자바스크립트는 프로토타입 기반 객체지향 언어
        - 프로토타입 기반 객체지향 언어는 클래스가 필요 없는 객체지향 프로그래밍 언어
            
            → ES6의 클래스 없이도 프로토타입을 통해서 객체지향 언어의 상속 구현 가능
            
            ```jsx
            var Person = (function() {
            	// 생성자 함수
            	function Person(name) {
            		this.name = name;
            	}
            
            	// 프로토타입 메서드
            	Person.prototype.sayHi = function() {
            		console.log('my name is' + this.name);
            	};
            
            	// 생성자 함수 반환
            	return Person;
            }());
            
            // 인스턴스 생성
            var me = new Person('Lee');
            me.sayHi();
            ```
            
            → 하지만 클래스 기반 언어에 익숙한 프로그래머들을 위해서 클래스 기반 객체지향 프로그래밍 언어와 흡사한 새로운 객체 생성 메커니즘 제시 (일종의 [문법적 설탕](https://en.wikipedia.org/wiki/Syntactic_sugar))
            
        - **클래스와 생성자 함수의 차이점**
            
            
            | 클래스 | 생성자 함수 |
            | --- | --- |
            | new 연산자 없이 호출하면 에러남 | new 연산자 없이 호출하면 일반 함수로서 호출됨 |
            | 상속을 지원하는 extends와 super 키워드를 제공함 | extends와 super 키워드를 제공하지 않음 |
            | 호이스팅이 발생하지 않는 것처럼 동작함 (호이스팅 될 때, 초기화 안됨) | 변수 호이스팅이 발생함 |
            | 클래스 내의 모든 코드에는 암묵적으로 strict mode가 지정되어 실행되며 해제할 수 없다 | 암묵적으로 strict mode가 지정되지 않는다 |
            | 클래스의 constructor, 프로토타입 메서드, 정적 메서드는 모두 프로퍼티 어트리뷰트의 값이 fasle이다. 다시 말해, 열거되지 않음 | - |

### **정의**

Class는 **특별한 함수**

함수를 함수 표현식과 함수 선언으로 정의할 수 있듯이 class 문법도 class 표현식과 class 선언 두가지 방식을 제공함

- **Class 선언**
    - class 키워드를 사용
    - class의 이름을 가져야함
    - **예시**
        
        ```jsx
        class Rectangle {
          constructor(height, width) {
            this.height = height;
            this.width = width;
          }
        }
        ```
        
    - 호이스팅 될 때 초기화되지 않음
        
        → 정의하기 전에 사용할 수 없음
        
        ```jsx
        const p = new Rectangle(); // Reference Error
        
        class Rectangle {}
        ```
        
- **Class 표현식**
    - 이름을 가질 수도 있고, 갖지 않을 수도 있음
        
        ```jsx
        // 익명 클래스 표현식
        const Person = class {};
        
        // 기명 클래스 표현식
        const Person = class MyClass {};
        ```
        
    - **class는 일급 객체이다**
        - **🤔 why?**
            
            표현식으로 정의할 수 있다는 것은 값으로 사용할 수 있다는 것이기 때문에!
            
            > **일급 객체의 조건**
            > 
            > 1. 변수에 할당(assignment)할 수 있다.
            > 2. 다른 함수를 인자(argument)로 전달 받는다.
            > 3. 다른 함수의 결과로서 리턴될 수 있다.
            
            또한, 클래스는 함수임 → 따라서 클래스 = 일급 객체!
            
        - 클래스는 **일급 객체**로서 다음과 같은 특징을 갖는다
            - 무명의 리터럴로 생성할 수 있음 (런타임에 생성 가능)
            - 변수나 자료구조(객체, 배열 등)에 저장 가능
            - 함수의 매개변수에게 전달할 수 있음
            - 함수의 반환 값으로 사용할 수 있음
    - 이름을 가진 class 표현식의 이름은 클래스 body의 local scope에 한해 유효함
    - 클래스의 name 속성을 통해서 이름을 알 수 있음
- 클래스 몸체에는 0개 이상의 메서드만 정의할 수 있음
    - 정의할 수 있는 메서드 (3가지)
        - constructor (생성자)
        - 프로토타입 메서드
        - 정적 메서드
    
    ```jsx
    class Person {
    	// 생성자
    	constructor(name) {
    		// 인스턴스 생성 및 초기화
    		this.name = name; // name 프로퍼티는 public
    	}
    
    	// 프로토타입 메서드
    	sayHi() {
    		console.log(`my name is ${this.name}`);
    	}
    
    	// 정적 메서드
    	static sayHello() {
    		console.log('hello')
    	}
    }
    
    // 인스턴스 생성
    const me = new Person('Lee');
    
    // 인스턴스의 프로퍼티 참조
    console.log(me.name);
    // 프로토타입 메서드 호출
    me.sayHi();
    // 정적 메서드 호출
    Person.sayHello();
    ```
    

### **인스턴스 생성**

- 클래스는 생성자 함수이기에 `new` 연산자와 함께 호출되어 인스턴스를 생성함
    
    ```jsx
    class Person {}
    
    const me = new Person();
    console.log(me); // Person {}
    ```
    
- 클래스를 `new` 연산자 없이 호출하면 타입 에러가 발생함
    
    ```jsx
    class Person {}
    
    const me = Person();
    // TypeError
    ```
    
- 클래스 표현식으로 정의된 클래스의 경우 클래스를 가리키는 식별자를 사용해서 인스턴스를 생성하지 않고 기명 클래스 표현식의 클래스 이름을 사용해서 인스턴스를 생성하면 에러 발생함
    
    ```jsx
    const Person = class MyClass {};
    
    const me = new Person();
    
    // MyClass는 함수와 동일하게 클래스 몸체 내부에서만 유효한 식별자임
    console.log(MyClass); // ReferenceError: MyClass is not defined
    
    const you = new MyClass(); // ReferenceError: MyClass is not defined
    ```
