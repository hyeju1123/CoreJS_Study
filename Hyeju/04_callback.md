## 제어권 위임
1. 실행시점
2. 매개변수
3. this

### 실행시점
```
var cb = function () {
	console.log('1초마다 실행될 겁니다.');
} 
setInterval(cb, 1000); // setInterval이 1초마다 알아서 콜백함수를 실행해줌
```
=> setInterval에게 콜백함수 제어권을 넘겨줬으므로, 개발자가 1초마다 일일이 함수를 실행할 필요없음


### 매개변수
```
var arr = [1, 2, 3, 4, 5];
var entries = [];
arr.forEach(function(v, i) { // 첫번째 인자: 콜백함수
	entries.push([i, v, this[i]]);
}, [10, 20, 30, 40, 50]); // this로 인식할 대상(생략가능)
console.log(entries); // [[0, 1, 10], [1, 2, 20], [2, 3, 30], [3, 4, 40], [4, 5, 50]]
```
=> forEach에 콜백함수를 넘기면, 이 콜백함수에 들어올 매개변수의 내용, 순서는 전적으로 forEach에 정의된 방식에 따를 수 밖에 없음

### this
```
document.body.innerHTML = '<div id="a">abc</div>';
function cbFunc(x) {
	console.log(this, x); // <div id="a">abc</div>  PointEvent
}
document.getElementById('a')
	.addEventListener('click', cbFunc);
```
```
// 이벤트 핸들러 콜백함수의 this를 특정 객체(obj)로 직접 정해주고 싶다면?

document.body.innerHTML = '<div id="a">abc</div>';
function cbFunc(x) {
	console.log(this, x); // {a:1} PointEvent 
}
const obj = { a: 1 }
document.getElementById('a')
	.addEventListener('click', cbFunc.bind(obj));
```

## 콜백함수의 특징
- 다른 함수(A)의 인자로 콜백함수(B)를 전달하면, A가 B의 **제어권**을 갖게 됨
- 특별한 요청(ex. bind)이 없는 한 A에 **미리 정해놓은 방식**에 따라 B를 호출함
- 미리 정해놓은 방식이란,
	- 어떤 **시점**에 콜백을 호출할지
	- **인자**에는 어떤 값들을 지정할지
	- **this**에 무엇을 바인딩할지 등

## 주의: 콜백은 '함수'이다
```
var arr = [1, 2, 3, 4, 5];
var obj = {
	vals: [1, 2, 3],
	logValues: function(v, i) {
		if(this.vals) {
			console.log(this.vals, v, i);
		} else {
			console.log(this, v, i);
		}
	}
};
obj.logValues(1, 2); // 메소드로서 logValues를 `호출`. 따라서 logValues 안에서 this는 obj가 됨
```
```
var arr = [1, 2, 3, 4, 5];
var obj = {
	vals: [1, 2, 3],
	logValues: function(v, i) {
		if(this.vals) {
			console.log(this.vals, v, i);
		} else {
			console.log(this, v, i);
		}
	}
};
arr.forEach(obj.logValues); // 콜백함수로 `전달`
```
=> forEach에 obj.logValues를 넘겨주면 **콜백함수**로서 넘긴 것임. obj.logValues를 메소드로서 개발자가 호출한 것이 아님. arr.forEach(obj.logValues)에서는 호출을 하지 않음. **호출은 forEach가** 하는 것이고, forEach에 정해진 호출 방식에 따라 this가 달라질 수 있는 것임.
=> obj 객체에 logValues라는 함수만 떼어서 forEach에 전달한 것이고, forEach 역시 이것을 함수로서 호출을 함