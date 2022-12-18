## this는 함수가 호출될 때 결정된다. 즉 함수를 호출하는 방식에 따라 this는 달라진다

## 1. 전역공간에서
- this: window / global 과 같은 전역 객체
- 전역 컨텍스트를 실행하는 주체가 전역 객체이기 때문

## 2. 함수 호출 시
- this: window / global 과 같은 전역 객체
```
function a() {
	console.log(this); // 전역 객체
}
a();  
```
=> 전역객체가 함수 a를 실행해주고 있으니 전역객체가 나오는 것은 이상할 게 없음

```
function b() {
	function c() {
		console.log(this); // 전역 객체
	}
	c();
}
b();
```
=> c함수를 호출하는 주체는 b함수인 것 같은데, 여기서도 this를 출력하면 전역객체가 나옴

=> 즉, 함수를 함수로서 호출하면 그 안에 있는 this는 무조건 전역객체에 바인딩되는 것
```
var d = {
	e: function() {
		function f() {
			console.log(this); // 전역 객체
		}
		f();  // 함수로서 호출함
	}
}
d.e();
```
=> 무조건 전역객체가 나온다는 점에서 혼란을 야기할 수 있기에, ES6부터는 this 바인딩을 하지 않는 **화살표 함수**가 등장함
```
var a = 10;
var obj = {
	a: 20,
	b: function() {
		var self = this; 
		console.log(this.a);  // 20
	
		const c = () => {
			console.log(this.a); 
			// this를 바인딩하지 않고 scope chain을 따라 올라가다 b 함수 안에서의 this를 찾게 됨. 따라서 c 함수 안에서의 this는 obj를 가리키게 됨
		}
		c();
	}
}
obj.b();
```
## 3. 메서드 호출 시
- this: 메서드를 호출한 객체(함수명 앞의 객체, 점 표기법의 경우 마지막 점 앞에 명시된 객체)
```
var a = {
	b: function() {
		console.log(this); // {b: f}
	}
}
a.b();
```
=> b함수를 a객체의 메서드로서 호출하고 있음
=> JS에서는 '클래스의 인스터스'냐와 상관없이 **어떤 객체와 관련된 동작**이기만 하면 메서드로 봄
```
var a = {
	b: {
		c: function() {
			console.log(this); // {c: f}
		}
	}
}
a.b.c();  
```

### 대괄호 표기법을 썼을 때는?
```
obj.func();  // this: obj
obj['func']();  // this: obj

person.info.getName();  // this: person.info
person.info['getName']();  // this: person.info
person['info'].getName();  // this: person['info']
person['info']['getName']();  // this: person['info']
```

### 메서드 내부함수에서의 우회법
```
var a = 10;
var obj = {
	a: 20,
	b: function() {
		console.log(this.a);  // 20
	
		function c() {
			console.log(this.a);  // 10 
			// 즉, window.a == 전역변수 a
			// 전역변수를 선언하면 자바스크립트 엔진은 이를 전역개체의 프로퍼티로 할당함
		}
		c();
	}
}
obj.b();
```
```
var a = 10;
var obj = {
	a: 20,
	b: function() {
		var self = this; // _this, that 같은 변수도 쓰임
		console.log(this.a);  // 20
	
		function c() {
			console.log(self.a);  // 20 
			// 함수 c안에서 self를 선언한 적이 없으니 scope chain 따라서 b 함수 안에서의 this, 즉 obj를 사용하게 됨
		}
		c();
	}
}
obj.b();
```
## 4. callback 호출 시
- this: 기본적으로는 함수 내부에서와 동일
### call, apply, bind 함수 배경지식(명시적으로 this를 바인딩하는 법)
```
function a(x, y, z) {
	console.log(this, x, y, z);
}
var b = {
	bb: 'bbb'
};

a.call(b, 1, 2, 3);  // {bb: "bbb"} 1 2 3 

a.apply(b, [1, 2, 3]);  // {bb: "bbb"} 1 2 3 

var c = a.bind(b);
c(1, 2, 3);  // {bb: "bbb"} 1 2 3 

var d = a.bind(b, 1, 20;
d(3);  // {bb: "bbb"} 1 2 3 
```

```
var callback = function() {
	console.dir(this);  // cb를 함수로서 호출했으니 전역객체가 나옴
}
var obj = {
	a: 1,
	b: function(cb) {
		cb();
	}
};
obj.b(callback);
```
```
var callback = function() {
	console.dir(this);  // obj의 메서드로 b를 호출했고 콜백함수는 b 함수의 this를 명시적으로 바인딩해서 출력하므로 obj가 나옴
}
var obj = {
	a: 1,
	b: function(cb) {
		cb.call(this);
	}
};
obj.b(callback);
```
=> 콜백함수 내부에서의 this는 콜백함수 자체가 제어할 수 있는 게 아님. 콜백함수를 넘겨받는 대상(위의 코드에서는 b함수)이 콜백함수를 어떻게 처리하느냐에 따라 this가 달라짐.

```
var callback = function() {
	console.dir(this);
};
var obj = {
	a: 1
};
setTimeout(callback, 100);
```
## 5. 생성자함수 호출 시