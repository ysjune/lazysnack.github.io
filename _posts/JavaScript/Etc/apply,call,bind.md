## apply, call, bind

서버 사이드인 java 의 this 와 javascript 의 this 가 헷갈리는 부분이 많은데,   
this 와 관련된 내용인 call, apply, bind 에 대한 내용이 5장(참조 타입) 에 나와 이 부분은 따로 정리해보고자 한다.  

### this
- 우선 this.. java 에서는 자기 자신을 가리키지만, javascript 에서는 다르다.  
javascript 에서의 this 는 `함수가 어떻게 호출되었는지에 따라` 다르게 동적으로 매핑? 된다.

```javascript
var foo = function() {
	console.log(this);
}

foo(); // this == window

var obj = {foo : foo};
obj.foo(); // this == obj

var instance = new foo(); // this == instance
```
하지만 이렇게만 된다면 간단하겠지.. 다음을 보면

```javascript
1)
function foo() {
	console.log(this); // this == window
	function bar() {
		console.log(this); // this == window
	}
	bar();
}
foo();

2)
var obj = {
	foo : function() {
		console.log(this); // this == obj
		function bar() {
			console.log(this); // this == window
		}
		bar();
	}
};
obj.foo();
```

이외에도 더 있겠지만, 내부함수의 경우에는 this 는 전역을 가리키는 것을 알 수 있다.  
찾아보면 이 부분은 설계 결함으로  
> 메소드가 내부함수를 사용하여 자신의 작업을 돕게 할 수 없다는 것을 의미한다
라고 하는데, 들어도 무슨 말인지 잘...  
여하튼 이런 문제로 인해 this 가 전역을 회피하는 방법이 있는데, 그것이 바로 apply, call, bind 이다.  
~~(물론 이전 회사에서 사용했던 var me = this 이런 방식도 있긴 함..)~~

### apply
- this 를 특정 객체에 명시적으로 바인딩
- Function.prototype.apply
```javascript
obj.apply(thisArg, [argsArray])
//thisArg : 함수 내부의 this 에 바인딩할 객체
// argsArray : 함수에 전달할 argument 배열

--------------------------------------

var person = function(name) {
	this.name = name;
}

var foo = {};

person.apply(foo, ['snack']);

console.log(foo); // {name : snack}

```

### call
- apply()와 기능은 같지만, 2번째 파라미터가 배열이 아닌 하나의 인자
```javascript

person.apply(foo, [1,2,3]);

person.call(foo, 1,2,3);

```
콜백 함수 같이 내부 함수의 this 를 위해서 사용된다.

```javascript
function Person(name) {
	this.name = name;
}

Person.prototype.doSomething = function (callback) {
	if(typeof callback == 'function') {
		callback.call(this);
	}
};

function foo() {
	console.log(this.name);
}

var p = new Person("snack");
p.doSomething(foo); // 'snack'
```

### bind
- apply, call 과는 다르게 함수에 인자로 전달된 this 가 바인딩된 새로운 함수를 반환한다.
```javascript
window.color = "red";
var o = {color : "blue"};

function sayColor() {
	console.log(this.color);
}

var objectSayColor = sayColor.bind(o);
objectSayColor(); // blue
```
- apply, call 과 방법은 비슷하지만, 호출을 하지 않고 함수만 반환한다는 점이 다르다.
```javascript
call(this, 1,2,3) == bind(this)(1,2,3) // bind(this) 는 반환 (1,2,3)이 실행
```
