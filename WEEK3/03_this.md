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

### 함수를 실행하는 방법 - 함수로서 호출하기 vs 메서드로서 호출하기

프로그래밍에서 함수와 메서드 : 미리 정의한 동작을 수행하는 코드 뭉치  
함수와 메서드를 구분하는 기준 - 독립성
함수 : 그 자체로 독립적인 기능을 수행함 / 메서드 : 자신을 호출한 대상 객체에 관한 동작을 수행

자바스크립트는 상황별로 this 키워드에 다른 값을 부여하게 함으로써 이를 구현함

> 메서드는 객체의 프로퍼티에 할당된 함수이다 ?  
> 반은 맞고 반은 틀린 설명!  
> 어떤 함수를 객체의 프로퍼티에 할당한다고 해서 그 자체로서 무조건 메서드가 되는 것이 아님  
> 객체의 메서드로서 호출할 경우에만 메서드로 동작하고 그렇지 않으면 함수로 동작함

```
var func = function (x){
    console.log(this, x);
}
func(1);    // Window { ... } 1  => 함수 앞에 점이 없으니 함수로 호출했구나!

var obj = {
    method: func
}
obj.method(2); // { method: f } 2  => method 앞에 점이 있으니 메소드로 호출했구나!
```

어떤 함수를 호출할 때 그 함수 이름 앞에 객체가 명시돼 있는 경우 : 메소드로 호출한 것  
그 외의 경우 : 함수로 호출한 것

```
var obj = {
    method: function(x){ console.log(this, x); }
};
obj.method(1);   // { method: f } 1
obj["method"](2);   // { method: f } 2
```

### 메서드로서 호출할 때 그 메서드 내부에서의 this

메서드 내부에서의 this 예

```
var obj = {
    methodA: function(){ console.log(this); }
    inner: {
        methodB: function(){ console.log(this); }
    }
}
obj.methodA();   // { methodA: f, inner: { ... } }
obj["methodA"]();   // { methodA: f, inner: { ... } }

obj.inner.methodB();  // { methodB: f }
obj.inner["methodB"]();  // { methodB: f }
obj["inner"].methodB();  // { methodB: f }
obj["inner"]["methodB"]();  // { methodB: f }
```

### 함수로서 호출할 때 그 함수 내부에서의 this

- 함수 내부에서의 this
  어떤 함수를 함수로서 호출할 경우에는 this가 지정되지 않음!  
  이유 : 원래 this에는 호출한 주체에 대한 정보가 담기는데 함수는 개발자가 코드에 직접 관여해서 실행한 것이기 때문에 호출 주체의 정보를 알 수 없음

> 2장 실행 컨텍스트 내용, "실행 컨텍스트를 활성화 할 당시 this가 지정되지 않은 경우 this는 전역 객체를 바라본다"

따라서 함수에서의 this는 전역 객체를 가리키며, 더글라스 크락포드는 이를 설계상 오류라고 지적함!

- 메서드 내부함수에서의 this

```
var obj1 = {
    outer: function (){
        console.log(this);  // obj1
        var innerfunc = function (){
            console.log(this);  // 전역객체 (Window)
        }
        innerfunc();   >> 함수로 호출한 것

        var obj2 = {
            innerMethod: innerfunc
        };
        obj2.innerMethod();  >> 메소드로서 호출한 것 // obj2
    }
}
obj1.outer();  >> 메소드로서 호출한 것
```

> 희망사항
> 호출 주체가 없을 때에는 자동으로 전역객체를 바인딩하지 않고 호출 당시 주변 환경의 this를 그대로 상속받아 사용할 수 있다면 좋겠다 -> 스코프 체인과의 일관성 지킬 수 있도록  
> (변수를 검색하면 가장 가까운 스코프의 L.E를 찾고 없으면 상위 스코프를 탐색하듯이)

내부함수에서의 this를 우회하는 방법

1. 변수 이용 -> 그저 상위 스코프의 this를 저장해서 내부함수에서 활용하려는 수단

```
var obj = {
    outer: function(){
        console.log(this);
        var innerFunc1 = function(){
            console.log(this);
        }
        innerFunc1();

        var self = this;
        var innerFunc2 = function (){
            console.log(self);
        }
        innerFunc2();
    }
}
obj.outer();
```

2. this를 바인딩(메모리에 값을 할당)하지 않는 함수 (화살표함수) 사용  
   (주의, ES5 환경에서는 화살표 함수 사용 불가함)

```
var obj = {
    outer: function(){
        console.log(this);   // { outer: f }
        var innerFunc = () => {
            console.log(this);   // { outer: f }
        }
        innerFunc();
    }
};
obj.outer();
```

화살표 함수는 실행 컨텍스트를 생성할 때 this 바인딩 과정 자체가 빠지게 되어 상위 스코프의 this를 그대로 활용할 수 있게 됨

3. 그 외 call, apply 등의 메서드를 활용해 함수를 호출할 때 명시적으로 this를 지정하는 방법

### 콜백 함수 호출 시 그 함수 내부에서의 this

함수 A의 제어권을 다른 함수 B(또는 메서드 B)에게 넘겨준다면, 함수 A는 콜백함수  
-> 함수 A는 함수 B의 내부 로직에 따라 실행되고 함수 B 내부 로직에서 정한 규칙에 따라 this 값이 결정됨

콜백함수도 함수이기 때문에 기본적으로는 this가 전역객체를 참조하지만,  
콜백 함수의 제어권을 가지는 함수가 콜백 함수에서의 this를 무엇으로 할 지 결정하기 때문에  
this는 무조건 이것이다 라고 정의할 수 없음!

```
setTimeout(functioin(){ console.log(this); }, 300);

[1, 2, 3, 4, 5].forEach(function(x){
    console.log(this, x);
});

document.body.innerHTML += "<button id="a">클릭</button>;
document.body.querySelector("#a")
.addEventListener("click", function(e){
    console.log(this, e);
});


```

### 생성자 함수 내부에서의 this

생성자 함수 : 어떤 공통된 성질을 지니는 객체들을 생성하는 데 사용하는 함수

객체지향 언어에서의 '생성자 = 클래스', '클래스를 통해 만든 객체 = 인스턴스'

프로그래밍 속 생성자(클래스)란 ? 구체적인 인스턴스를 만들기 위한 일종의 틀  
인간의 공통 속성(직립 보행, 언어 구사, 도구 사용 등)을 모아 인간 집합을 정의한 것 : 클래스  
각 인스턴스들은 위의 인간의 공통 속성을 지니지만 저마다의 개성도 존재함

자바스크립트는 함수에 생성자로서의 역할을 함께 부여했음  
new 명령어와 함께 함수를 호출하면 해당 함수가 생성자로서 동작하게 됨!!  
그리고 어떤 함수가 생성자 함수로서 호출된 경우 내부에서의 this는 새로 만들어지는 인스턴스가 됨

생성자 함수를 호출(new 명령어와 함께 함수를 호출)하면 우선 생성자의 prototype 프로퍼티를 참조하는 \_\_proto\_\_ 라는 프로퍼티가 있는 객체(인스턴스)를 만들고, 미리 준비된 공통 속성 및 개성을 해당 객체 this에 부여함 => 구체적인 인스턴스 만들어짐

```
var Cat = function (name, age){
    this.bark = "야옹";
    this.name = name;
    this.age = age;
}

var choco = new Cat("초코", 7);
var nabi = new Cat("나비", 5);
console.log(choco, nabi);

/* 결과
Cat { bark : "야옹", name : "초코", age : 7 }
Cat { bark : "야옹", name : "나비", age : 5 }
*/
```

# 02 명시적으로 this를 바인딩하는 방법

this에 별도의 대상을 바인딩하는 방법

### call 메서드

-> call 메서드 : 메서드의 호출 주체인 함수를 즉시 실행하도록 하는 명령  
-> 임의의 객체를 this로 지정할 수 있도록 함

```
Function.prototype.call(thisArg[, arg1[, arg2[, ...]]])
```

call 메서드의 첫번째 인자를 this로 바인딩하고, 이후의 인자들을 호출할 함수의 매개변수로 함

```
var func = function (a, b, c){
    console.log(this, a, b, c);
};

func(1, 2, 3);   // Window{ ... } 1 2 3
func.call({ x : 1 }, 4, 5, 6 );   // { x : 1 } 4 5 6

```

```
var obj = {
    a: 1,
    method: function (x, y) {
        console.log(this.a, x, y);
    }
};

obj.method(2, 3);   // 1 2 3
obj.method.call({ a : 4 }, 5, 6 );   // 4 5 6
```

### apply 메서드

```
Function.prototype.apply(thisArg[, arg1[, arg2[, ...]]])
```

call 메서드와 기능적으로 완전히 동일!

차이점 ?  
call 메서드 : 첫 번째 인자를 제외한 나머지 모든 인자들을 호출할 함수의 매개변수로 지정  
apply 메서드 : 두 번째 인자를 배열로 받아 그 배열의 요소들을 호출할 함수의 매개변수로 지정

```
var func = function (a, b, c){
    console.log(this, a, b, c);
};

func.apply({ x : 1 }, [4, 5, 6]);   // { x : 1 } 4 5 6


var obj = {
    a: 1,
    method: function (x, y){
        console.log(this.a, x, y);
    }
};
obj.method.apply({ a: 4 }, [5, 6]);   // 4 5 6
```

### call / apply 메서드

call 이나 apply 메서드를 이용하여 자바스크립트 더욱 다채롭게 사용하기 예시

- 유사배열객체에 배열 메서드 적용

```
var obj = {
    0: "a",
    1: "b",
    2: "c",
    length: 3
};

Array.prototype.push.call(obj, "d");
console.log(obj);   // { 0: "a", 1: "b", 2: "c", 3: "d", length: 4 }

var arr = Array.prototype.slice.call(obj);  -> slice 메서드를 이용해 객체를 배열로 전환함
console.log(arr);   // [ "a", "b", "c", "d" ]
```

> 객체에는 배열 메서드를 직접 사용할 수 없으나 키가 0 또는 양수인 프로퍼티가 존재하고 length 프로퍼티의 값이 0 또는 양수인 객체(배열의 구조와 유사할 경우)이면 call 또는 apply 메서드를 사용하여 배열 메서드를 차용할 수 있음!

> slice 메서드 : 시작 인덱스 값과 마지막 인덱스 값을 받아 시작 값부터 마지막 값의 앞부분까지의 배열 요소를 추출하는 메서드이며, 매개변수를 아무것도 넘기지 않을 경우 원본 배열의 얕은 복사본을 반환함  
> 즉, call 메서드를 이용해 원본인 유사배열 객체의 얇은 복사를 수행한 것인데 slice 메서드가 배열 메서드이기 때문에 복사본은 배열로 반환하게 된 것

함수 내부에서 접근할 수 있는 arguments 객체도 유사배열객체이므로 위의 방법을 이용해 배열로 전환 가능  
querySelectorAll, getElementsByClassName 등의 Node 선택자로 선택한 결과인 NodeList도 마찬가지

```
function a (){
    var argv = Array.prototype.slice.call(arguments);
    argv.forEach(function (argv) {
        console.log(arg);
    });
}
a(1, 2, 3);

document.body.innerHTML = "<div>a</div><div>b</div><div>c</div>";
var nodeList = document.querySelectorAll("div");
var nodeArr = Array.prototype.slice.call(nodeList);
nodeArr.forEach(function (node){
    console.log(node);
});
```

### bind 메서드

ES5에서 추가된 기능으로 call과 비슷하지만 즉시 호출되지 않고 넘겨받은 this 및 인수들을 바탕으로 새로운 함수를 반환하기만 하는 메서드  
새로운 함수를 호출할 때 전달했던 인수들의 뒤에 이어서 등록

```
Function.prototype.bind(thisArg[, arg1[, arg2[, ...]]])
```

bind 메서드는 '함수에 this를 미리 적용하기' / '부분 적용 함수 구현하기' 두 가지 목적

```
var func = function (a, b, c, d) {
    console.log(this, a, b, c, d);
};
func(1, 2, 3, 4);   // Window {...} 1 2 3 4

var bindFunc1 = func.bind({ x : 1 });
bindFunc1(5, 6, 7, 8);   // { x : 1 } 5 6 7 8

var bindFunc2 = func.bind({ x : 1 }, 4, 5);
bindFunc2(6, 7);   // { x : 1 } 4 5 6 7
bindFunc2(8, 9);   // { x : 1 } 4 5 8 9
```

### 화살표 함수의 예외사항

### 별도의 인자로 this를 받는 경우(콜백함수 내에서의 this)

# 03 정리

...
...

# 추가 공부 내용 (제로초 블로그 참고)

유사배열객체란 무엇인가?

```
var array = [1,2,3];
array;  // [1, 2, 3]
var nodes = document.querySelectorAll("div");   // NodeList [div, div, div, div, div, ...]
var els = document.body.children;   // HTMLCollection [noscript, link, div, script, ...]
```

위의 예제에서 array는 배열이고, nodes와 els는 유사배열임  
둘 다 []로 감싸져있기 때문에 배열과 유사배열의 차이를 알기 어려움

Array.isArray 메서드(배열인지를 판단해주는 메서드)를 사용하여 판단하기  
('판단할대상 instanceof Array' 으로도 판단 가능)

```
Array.isArray(array);   // true
Array.isArray(nodes);   // false
Array.isArray(els);   // false
```

직접 배열 리터럴로 선언한 array만 배열이고 그 외에는 []로 감싸져있지만 배열이 아닌 유사배열임

유사배열이 만들어지는 과정 예시

```
var yoosa = {
    0: "a",
    1: "b",
    2: "c",
    length: 3
}
```

위의 yoosa 객체가 바로 유사배열임 (키가 숫자이고, length 속성을 가짐 -> 배열도 객체라는 성질을 이용한 트릭)  
-> 배열처럼 yoosa[0], yoosa[1], yoosa.length 등이 가능하다는 점

그렇다면 왜 배열과 유사배열을 구분해야하는가 ?  
-> 유사배열의 경우 배열의 메서드를 사용할 수 없기 때문!!!  
-> 그래서 call, apply를 사용하여 배열 메서드를 사용하도록 하는 것

ES6에서는 더 이상 안 보이지만, 존재하는 또 다른 유사배열의 예시

```
function arrayLike(){
    console.log(arguments);
    [].forEach.call(arguments, function(el){ console.log(el) });
}
arrayLike(4,5,6);
```
