# 1 타입
## 1.2 내장 타입
- null
- undefined
- boolean
- number
- string
- object
- symbol

```javascript
console.log(typeof undefined === "undefined");
console.log(typeof true === "boolean");
console.log(typeof 42 === "number");
console.log(typeof "42" === "string");
console.log(typeof { life: 42} === "object");
console.log(typeof Symbol() === "symbol"); // es6
```

typeof의 반환 값은 7가지 내장 타입과 1:1로 매칭되지 않음  

null: null을 반환하지 않음
```javascript
console.log(typeof null === "object");
```
null 값의 정확한 검사 - null은 falsy한 유일한 원시 값이지만 타입은 object
```javascript
var a = null;
console.log(!a && typeof a === "object");
```
typeof function
```javascript
console.log(typeof function a(){/**/} === "function");
```
실제로는 function은 object의 하위 타입: 호출 가능한 객체

함수의 프로퍼티
```javascript
function a(b, c) {
  /* */
}
console.log(a.length);
```
함수의 length 프로퍼티: 함수에 선언된 인자 개수

배열: 객체  
숫자 인덱스를 가지며, length 프로퍼티가 자동으로 관리됨
```javascript
console.log(typeof [1, 2, 3] === "object");
```
## 1.3 값, 타입
타입 강제가 존재하지 않음  
변수에 처음 할당된 값과 이후에 할당할 값이 동일한 타입일 필요는 없음  
typeof: 변수에 할당된 값의 타입을 알아봄  
typeof의 반환값 타입은 문자열
```javascript
console.log(typeof typeof 42);
```

### 1.3.1 값이 없는 vs 선언되지 않은
값이 없는 변수의 값은 undefined
```javascript
var a;
var resA = typeof a;
console.log('resA', resA);
var b = 42;
var c;
b = c;

var resB = typeof b;
var resC = typeof c;
console.log('resB', resB);
console.log('resC', resC);
```
  
값이 없는 undefined: 접근 가능한 스코프에 변수가 선언되었으나 현재 아무런 값도 할당되지 않은 상태  
선언되지 않은 undefined: 접근 가능한 스코프에 변수 자체가 선언조차 되지 않은 상태
```javascript
var a1;
console.log('a1', a1, typeof a1);
// console.log('b1', b1, typeof b1);
console.log('typeof a1, b1', typeof a1, typeof b1);
// 선언되지 않은 변수도 typeof로 검사하면 undefined를 출력
```

## 1.3.2 선언되지 않은 변수
typeof를 활용한 전역변수 체크 방법
```javascript
// if (DEBUG) {
//   console.log("디버깅을 시작합니다.");
// }
if (typeof DEBUG !== "undefined") {
  console.log("디버깅을 시작합니다.");
}
// 내장 API 기능 체크 시
if (typeof atob === "undefined") {
  atob = function(){/* */}
}
```

var atob로 선언하지 않음
var atob로 선언할 경우 코드 실행을 건너뛰더라도 선언 자체가 최상위 스코프로 호이스팅됨
var atob로 선언하면 다음과 같은 코드로 해석됨
```javascript
/* 
var atob;
if (typeof atob === "undefined") {
  atob = function() {}
}
*/

if (typeof atob1 === "undefined") {
  var atob1 = function(a){
    console.log(a + 1)
  }
}
atob1(1);
```

typeof 없이 전역 변수를 체크하는 방법
```javascript
/*
if (window.DEBUG) {

}
if (!window.DEBUG) {

}
*/
```
선언되지 않은 변수 때와는 달리 어떤 객체의 프로퍼티로 접근할 때에는 
그 프로퍼티가 존재하지 않아도 ReferenceError가 나지 않음
그러나 window 객체를 통한 전역변수 참조는 삼가야 함
전역변수가 꼭 window 객체로 호출되지는 않기 때문



typeof의 사용  
다른 코드를 복사하여 사용할 때, 특정 변수값이 정의되어 있는지 체크해야 하는 상황
```javascript
function doSomethingCool() {
  var helper = (typeof FeatureXYZ !== "undefined") ? FeatureXYZ : function(){/*기본 XYZ 기능*/}
  var val = helper();
}

(function() {
  function FeatureXYZ() {}

  function doSomethingCool() {
    var helper = (typeof FeatureXYZ !== "undefined") ? FeatureXYZ : function(){/*기본 XYZ 기능*/}
    var val = helper();
  }
  doSomethingCool();
})();
```

의존성 주입 설계 패턴 시
명시적으로 의존관계를 전달
```javascript
function doSomethingCool1(FeatureXYZ) {
  var helper = FeatureXYZ || function() {};
  var val = helper();
}
```