자바스크립트 4장에서 객체 복사에 대한 내용을 다루다가   
__Object.assign()__ 을 통해 복사가 가능하다는 얘기를 잠깐 했었는데,  
자세히 모르니 이번 기회에 짚고 넘어가보자  
  
    
### Object.assign()
우선 [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign) 에서 하는 설명을 보면  
> The Object.assign() method is used to copy the values of all enumerable own properties from one or more source objects to a target object. It will return the target object.  

대상 객체로 하나 또는 그 이상의 열거 가능한 프로퍼티를 타겟오브젝트 복사할 때 사용.. 한다고 하는데, 글만 봐서는 잘 모르겠다.  

우선 문법을 살펴보면,

> Object.assign(target, ...sources)
>> parameter
>>> - target
>>> - sources

>> return
>>> - target Object

첫번째 파라미터인 target 은 설명의 target object 를 가리키고, sources 는 1개 혹은 그 이상의 열거 프로퍼티를 가리키는 것을 알 수 있다.  
그리고 반환값은 target object 이다.  

문법을 알아봤으니, 간단한 예제를 보면.
```javascript
const obj = { a: 1 };
const copy = Object.assign({}, obj);
console.log(copy); // { a: 1 }
obj.a = 2;
console.log(obj); // { a: 2 }
console.log(copy); // { a: 1 }
```
  
{} 에 obj의 프로퍼티를 복사(?) 했고, obj 와 copy 는 별개의 객체로 볼 수 있다.
이같이 obj 의 속성이 값일 경우엔 별 문제가 되지 않는다. 하지만 객체를 복사할 경우에는 얘기가 달라지는데
```javascript
let obj1 = { a: 0 , b: { c: 0}};
let obj2 = Object.assign({}, obj1);
console.log(obj2); // { "a": 0, "b": { "c": 0}}

1)  
obj1.a = 1;
console.log(obj1); // { "a": 1, "b": { "c": 0}}
console.log(obj2); // { "a": 0, "b": { "c": 0}}

2)  
obj2.a = 2;
console.log(obj1); // { "a": 1, "b": { "c": 0}}
console.log(obj2); // { "a": 2, "b": { "c": 0}}

3)  
obj2.b.c = 3;
console.log(obj1); // { "a": 1, "b": { "c": 3}}
console.log(obj2); // { "a": 2, "b": { "c": 3}}
```

1과 2의 경우에서는 값 자체이므로 서로 영향을 미치지 않는다.  
하지만, 3과 같이 객체의 경우에는 `__얕은 복사를__` 하기 때문에 참조값을 복사하게 된다.  
MDN 에서는 깊은 복사를 하기 위해서는 다음과 같은 방법을 하라고 얘기하는데

```javascript
obj1 = { a: 0 , b: { c: 0}};
let obj3 = JSON.parse(JSON.stringify(obj1));
obj1.a = 4;
obj1.b.c = 4;
console.log(obj3); // { a: 0, b: { c: 0}}
```
  
JSON 을 통하여 obj1 을 문자열로 변환하고, 다시 객체로 만드는 과정을 통하는 것이다. ~~(문자열로 만든 시점에서 참조 주소고 뭐고 없을 것 같긴 하다.)~~

알고자해서 작성한 내용은 여기서 끝이지만, assign 을 사용하는 방법은 여러가지 방법이 있으니, 좀 더 적어보겠다.

#### 1. 객체 병합
```javascript
const o1 = { a: 1, b: 1, c: 1 };
const o2 = { b: 2, c: 2 };
const o3 = { c: 3 };

const obj1 = Object.assign({}, o1, o2, o3);
console.log(obj1); // { a: 1, b: 2, c: 3 } // 뒤에 값에 의해 덮어쓰기

const obj2 = Object.assign(o1, o2, o3);
console.log(obj2) // { a: 1, b: 2, c: 3 }
console.log(o1) // { a: 1, b: 2, c: 3 } // 대상 자체가 변경
```

- 이후의 부분은 자바스크립트에 대해 공부를 더 한 후 개념을 알면 후술하는 것으로...

#### 2. 심볼 타입 프로퍼티 복사

#### 3. 프로토타입 체인의 속성, 열거불가 프로퍼티는 복사 불가

#### 4. 프리미티브 는 객체로 래핑
```javascript
var v1 = 'abc';
var v2 = true;
var v3 = 10;
var v4 = Symbol('foo');

var obj = Object.assign({}, v1, null, v2, undefined, v3, v4); 
// Primitives will be wrapped, null and undefined will be ignored.
// Note, only string wrappers can have own enumerable properties.
console.log(obj); // { "0": "a", "1": "b", "2": "c" }
```
문자열만 열거 가능한 프로퍼티를 가지고 있기 때문에 문자열만 복사가 됨을 알 수 있다.

#### 5. 복사중 예외 발생

#### 6. 접근자 복사

------------------------
~~적어보려고 했으나 모르는 게 너무 많다.~~

참고로, Object.assign() 의 경우 익스플로러에서는 동작하지 않는다고 한다. 
