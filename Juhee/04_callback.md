# 01 콜백 함수란?

콜백 함수 : 다른 코드의 인자로 넘겨주는 함수  
(콜백 함수를 넘겨받은 코드는 필요에 따라 적절한 시점에 콜백 함수를 실행)

# 02 제어권

### 02 - 1 호출시점

예제 1

```
var count = 0;
var timer = setInterval(function(){
    console.log(count);
    if(++count > 4) clearInterval(timer);
}, 300);
```

여기서 잠깐, setInterval의 구조 알아보기

```
var intervalID = scope.setInterval(func, delay[, param1, param2, ...]);
```

scope 에는 Window 객체 또는 Worker의 인스턴스가 들어올 수 있음  
=> 두 객체 모두 setInterval 메서드를 제공하기 때문

- 일반적인 브라우저 환경에서는 window를 생략해서 함수처럼 사용 가능
- 매개변수로는 func, delay 값을 반드시 전달해야하고, 세 번째 매개변수부터는 선택적!  
  => func는 함수, delay는 밀리초 단위 숫자, 그 외의 것들은 func 함수 실행 시 매개변수로 전달할 인자들
- func에 넘겨준 함수는 매 delay 마다 실행되며 어떤 값도 리턴하지 않음
- setInterval을 실행하면 반복적으로 실행되는 내용 자체를 특정할 수 있는 고유한 ID값이 반환됨, 이를 변수에 담는 이유는 반복 실행되는 중간에 종료(clearInterval) 할 수 있게 하기 위함

예제 1 조금 쉽게 변형하고 다시 보기

```
var count = 0;
var cbFunc = function(){
    console.log(count);
    if(++count > 4) clearInterval(timer);
};

var timer = setInterval(cbFunc, 300);

// 실행 결과
// 0 (0.3초)
// 1 (0.6초)
// 2 (0.9초)
// 3 (1.2초)
// 4 (1.5초)
```

timer 변수에 setInterval의 ID 값이 담김!

### 02 - 2 인자

```
var newArr = [10, 20, 30].map(function(currentValue, index){
    console.log(currentValue, index);
    return currentValue + 5;
});
console.log(newArr);

// 실행 결과
// 10 0
// 20 1
// 30 2
// [15, 25, 35]
```

Array의 prototype에 담긴 map 메서드의 구조

```
Array.prototype.map(callback[, thisArg])
callback: function(currentValue, index, array)
```

첫번째 인자로 callback 함수를 받고, 생략 가능한 두 번째 인자로 콜백함수 내부에서 this로 인식할 대상을 특정할 수 있음 (thisArg를 생략할 경우에는 일반적인 함수처럼 전역객체가 바인딩 됨)

" 메서드의 대상이 되는 배열의 모든 요소들을 처음부터 끝까지 하나씩 꺼내어 콜백 함수를 반복 호출하고, 콜백 함수의 실행 결과들을 모아 새로운 배열을 만듦 "

구성 : 콜백 함수 (현재값, 현재값의 인덱스, map 메서드의 대상이 되는 배열 자체)

> map 메서드와 JQuery의 차이  
> JQuery의 메서드들은 기본적으로 첫 번째 인자에 index가, 두 번째 인자에 currentValue가 옴

만약 제이쿼리 방식처럼 순서를 바꾸어 map 메서드를 사용해본다면?

```
var nerArr2 = [10, 20, 30].map(function(index, currentValue){
    console.log(index, currentValue);
    return currentValue + 5;
});
console.log(newArr2);

// 실행 결과
// 10 0
// 20 1
// 30 2
// [5, 6, 7]  -> 기존 map 순서를 지켰을 때와는 다른 결과 출력 !!!
```

5, 6, 7이 나오는 이유는 0+5, 1+5, 2+5 됐기 때문

### 02 - 3 this

콜백 함수도 함수이기 때문에 기본적으로는 this가 전역객체를 참조하지만, 제어권을 넘겨받을 코드에서 콜백 함수에 별도로 this가 될 대상을 지정한 경우에는 그 대상을 참조하게 됨

별도의 this를 지정하는 방식 및 제어권에 대한 이해를 높이기 위한 map 구현 예시

```
Array.prototype.map = function(callback, thisArg){
    var mappedArr = [];
    for(var i = 0; i < this.length; i++){
        var mappedValue = callback.call(thisArg || window, this[i], i, this);
        mappedArr[i] = mappedValue;
    }
    return mappedArr;
};
```

메서드 구현의 핵심 : call/apply 메서드  
this에는 thisArg 값이 있을 경우 그 값을, 없을 경우 전역 객체를 지정하고  
첫 번째 인자에는 메서드의 this가 배열을 가리킬 것이므로 배열의 i번째 요소 값을, 두 번째 인자에는 i 값을, 세 번째 인자에는 배열 자체를 지정해 호출 함  
그 결과가 변수 mappedValue에 담겨 mappedArr의 i번째 인자에 할당됨

this에 다른 값이 담기는 이유  
=> 제어권을 넘겨받을 코드에서 call/apply 메서드의 첫 번째 인자에 콜백 함수 내부에서의 this가 될 대상을 명시적으로 바인딩하기 때문

```
setTimeout(function(){ console.log(this); }, 300);

[1, 2, 3, 4, 5].forEach(function(x){
    console.log(this);
});

document.body.innerHTML += `<button id="a>클릭</button>`;
document.body.querySelector("#a").addEventListener("click", function(e){
    console.log(this, e);
});
```

# 03 콜백 함수는 함수다

메서드를 콜백 함수로 호출하는 경우 -> 메서드가 아닌 함수로서 호출됨

```
var obj = {
    vals: [1, 2, 3],
    logValues: function(v, i){
        console.log(this, v, i);
    }
};
obj.logValues(1,2);  // { vals: [1, 2, 3], logValues: f } 1 2
[4, 5, 6].forEach(obj.logValues);
// Window { ... } 4 0
// Window { ... } 5 1
// Window { ... } 6 2
```

위 예시코드에서 알아본 차이점  
메서드로 정의됐을 때 : obj.logValues 처럼 메서드 이름 앞에 점이 있으니 this는 obj를 가리키고, 인자로 넘어온 1, 2가 출력됨  
콜백 함수로 전달됐을 때 : 메서드를 전달한 것이 아닌 obj.logValues가 가리키는 함수만 전달한 것(직접적인 연관이 없어진 상황) -> forEach에 의해 콜백이 함수로서 호출되고 별도 this를 지정하는 인자를 지정하지 않았으므로 함수 내부 this는 전역객체를 바라봄

어떤 함수의 인자에 객체의 메서드를 전달하더라도 이는 결국 메서드가 아닌 함수라는 것

# 04 콜백 함수 내부의 this에 다른 값 바인딩하기

객체의 메서드를 콜백함수로 전달하면 해당 객체를 this로 바라볼 수 없게됨  
그런데 만약 콜백 함수 내부에서 this가 해당 객체를 바라보게 하고싶다면 어떻게 해야할까?

콜백 함수 내부의 this에 다른 값을 바인딩하는 방법 1 - 전통적인 방식

```
var obj1 = {
    name: "obj1",
    func: function(){
        var self = this;
        return function(){
            console.log(self, name);
        };
    }
};
var callback = obj1.func();
setTimeout(callback, 1000);
```

obj1.func 메서드 내부에서 self 변수에 this를 담고 익명 함수를 선언함과 동시에 반환했음  
이후 obj1.func를 호출하면 앞서 선언한 내부함수가 반환되어 callback 변수에 담김  
이후 callback을 setTimeout 함수에 인자로 전달하면 1초 뒤 callback이 실행되면서 obj1을 출력하는 것

위의 코드에서 func 함수 재활용하는 예

```
var obj2 = {
    name: "obj1",
    func: obj1.func
};
var callback2 = obj2.func();
setTimeout(callback2, 1500);

var obj3 = { name: "obj3" };
var callback3 = obj1.func.call(obj3);
setTimeout(callback3, 2000);
```

callback2에는 obj2의 func를 실행한 결과를 담아 이를 콜백으로 사용했음

=> 그러나 위의 전통적인 방식은 실제로 this를 사용하지 않으며 번거로움 (그럴거면 this를 안쓰겠다)

그래서 보여주는 콜백 함수 내부에서 this를 사용하지 않는 경우

```
var obj1 = {
    name: "obj1",
    func: function(){
        console.log(obj1.name);
    }
};
setTimeout(obj1.func, 1000);
```

훨씬 간결하고 직관적이지만, 작성한 함수를 this를 이용해 다양한 상황에서 재활용할 수 없음!!

# 05 콜백 지옥과 비동기 제어

콜백 지옥 : 콜백 함수를 익명 함수로 전달하는 과정이 반복되며 코드의 들여쓰기가 감당하기 힘들 정도로 깊어져 가독성이 떨어지고 코드 수정이 어려워 지는 현상
-> 주로 비동기적 작업(이벤트 처리, 서버 통신)을 수행할 때 이런 형태가 자주 등장함

동기 : 현재 실행중인 코드가 완료된 후에야 다음 코드를 실행하는 방식
비동기 : 현재 실행 중인 코드의 완료 여부와 무관하게 다음 코드를 실행하는 방식

동기적인 코드와 비동기적인 코드의 차이
-> CPU의 계산에 의해 즉시 처리가 가능한 대부분의 코드는 동기적인 코드  
 (비록 계산 식이 복잡해서 CPU가 계산하는 데 시간이 많이 필요하더라도 이는 동기적인 코드)  
-> 별도의 요청, 실행 대기, 보류 등과 관련된 코드는 비동기적 코드

#콜백지옥 해결 방법 1 - 기명함수로 변환하기  
-> 코드의 가독성을 높이고 함수 선언과 함수 호출을 구분할 수 있기 때문에 위에서 아래로 순서대로 읽어내려가는 데에 어려움이 없음

#콜백지옥 해결 방법 2 - 비동기 작업의 동기적 표현 (ES6의 Promise 이용하기)  
-> 비동기 작업이 완료될 때 resolve 또는 reject 를 호출하는 방법으로 비동기 작업의 동기적 표현을 가능하게 함
(구체적인 설명 : new 연산자와 함께 호출한 Promise의 인자로 넘겨주는 콜백 함수는 호출할 때 바로 실행되지만, 그 내부 resolve 또는 reject 함수를 호출하는 구문이 있을 경우 둘 중 하나가 실행되기 전까지는 then이나 catch 구문으로 넘어가지 않음)

#콜백지옥 해결 방법 3 - 비동기 작업의 동기적 표현 (Generator)

```
var addCoffee = function(prevName, name) {
    setTimeout(function () {
        coffeeMaker.next(prevName ? prevName + ", " + name : name);
    }, 500);
};

var coffeeGenerator = function*(){
    var espresso = yield addCoffee("","에스프레소");
    console.log(espresso);
    var americano = yield addCoffee(espresso, "아메리카노");
    console.log(americano);
    var mocha = yield addCoffee(americano, "카페모카");
    console.log(mocha);
    var latte = yield addCoffee(mocha, "카페라떼");
    console.log(latte);
};

var coffeeMaker = coffeeGenerator();
coffeeMaker.next();
```

#콜백지옥 해결 방법 4 - 비동기 작업의 동기적 표현 (Promise + async/await)  
비동기 작업을 수행하고자 하는 함수 앞에 async를 표기하고 함수 내부에서 실질적인 비동기 작업이 필요한 위치마다 await를 표기하는 것만으로 뒤의 내용으로 Promise로 자동 전환함  
-> 해당 내용이 resolve 된 이후에야 다음으로 진행함 (Promise의 then과 흡사한 효과)
