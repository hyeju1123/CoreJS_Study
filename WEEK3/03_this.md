# 01 상황에 따라 달라지는 this

다른 대부분의 객체지향 언어에서의 this : 클래스로 생성한 인스턴스 객체  
자바스크립트에서의 this : 어디서든 사용 가능 -> 상황에 따라 this가 바라보는 대상이 달라짐

실행 컨텍스트가 생성될 때 자바스크립트의 this도 결정됨  
실행 컨텍스트는 함수를 호출할 때 생성되므로 this는 함수를 호출할 때 결정되는 것!  
함수를 어떤 방식으로 호출하느냐에 따라 값이 달라지게 됨

### 전역공간에서의 this

전역 공간에서 this는 전역 객체(window, global)
브라우저 환경에서 전역객체 : window

```
console.log(this);
console.log(window);
console.log(this === window);  // true
```

Node.js 환경에서 전역객체 : global

```
console.log(this);
console.log(global);
console.log(this === global);  // true
```

전역변수와 전역객체 1

```
var a = 1;
console.log(a);  // 1 (window.a에서 window.이 생략된 것으로 봐도 무방함)
console.log(window.a);   // 1
console.log(this.a);  // 1
```

전역 공간에서의 this는 전역객체를 의미하므로 두 값이 같은 값을 출력하는 것은 당연함
자바스크립트의 모든 변수는 실은 특정 객체(실행컨텍스트의 L.E)의 프로퍼티로서 동작하기 때문!!

'전역변수를 선언하면 자바스크립트 엔진은 이를 전역객체의 프로퍼티로 할당한다'

전역변수와 전역객체 2

```
var a = 1;
window.b = 2;
console.log(a, window.a, this.a);  // 1 1 1
console.log(b, window.b, this.b);  // 2 2 2

window.a = 3;
b = 4;
console.log(a, window.a, this.a);  // 3 3 3
console.log(b, window.b, this.b);  // 4 4 4

```

전역변수와 전역객체 3 : 예외
전역변수 선언과 전역객체의 프로퍼티 할당 사이에 전혀 다른 경우가 있음 (삭제 명령에 대하여)

```
var a = 1;
delete window.a;   // false
console.log(a, window.a, this.a);    // 1 1 1
```

```
var b = 2;
delete b;   // false
console.log(b, window.b, this.b);   // 2 2 2
```

```
window.c = 3;
delete window.c;   // true
console.log(c, window.c, this.c);   // Uncaught ReferenceError : c is not defined
```

```
window.d = 4;
delete d;   // true
console.log(d, window.d, this.d);   // Uncaught ReferenceError : c is not
```

처음부터 전역객체의 프로퍼티로 할당한 경우에는 삭제가 되는 반면, 전역변수로 선언한 경우에는 삭제가 되지 않음 (=> 사용자가 의도치 않게 삭제하는 것을 방지하기 위한 전략)  
전역변수를 선언하면 자바스크립트 엔진이 이를 자동으로 전역객체의 프로퍼티로 할당하면서 추가적으로 해당 프로퍼티의 configurable 속성(변경 및 삭제 가능성)을 false로 정의함

> 결론,  
> var로 선언한 전역변수와 전역객체의 프로퍼티는 호이스팅 여부 및 configurable 속성 여부에서 차이가 있음!

### 메서드로서 호출할 때 그 메서드 내부에서의 this

함수를 실행하는 방법 - 함수로서 호출하기 vs 메서드로서 호출하기

프로그래밍에서 함수와 메서드 : 미리 정의한 동작을 수행하는 코드 뭉치

### 함수로서 호출할 때 그 함수 내부에서의 this

### 콜백 함수 호출 시 그 함수 내부에서의 this

### 생성자 함수 내부에서의 this

# 02 명시적으로 this를 바인딩하는 방법

### call 메서드

### apply 메서드

### call / apply 메서드

### bind 메서드

### 화살표 함수의 예외사항

### 별도의 인자로 this를 받는 경우(콜백함수 내에서의 this)

# 03 정리
