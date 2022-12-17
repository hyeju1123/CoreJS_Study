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
첫 번째 인자에는 메서드의 this가 배열을 가리킬 것이므로 배열의 i번째 요소 값을,  
두 번째 인자에는 i 값을,  
세 번째 인자에는 배열 자체를 지정해 호출 함  
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

# 04 콜백 함수 내부의 this에 다른 값 바인딩하기

# 05 콜백 지옥과 비동기 제어

# 06 정리
