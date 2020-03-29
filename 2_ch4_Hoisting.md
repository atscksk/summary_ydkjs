# 4 호이스팅

## 4.1 닭이 먼저냐 달걀이 먼저냐
```javascript
// code 1
a = 2;
var a;
console.log(a);
```
결과는 undefined가 아닌 2

```javascript
// code 2
console.log(a);
var a = 2;
```
2 출력 또는 ReferenceError가 아니라 undefined 출력  
선언문이 먼저인가 대입문이 먼저인가  

## 4.2 컴파일러는 두 번 공격한다
자바스크립트 엔진은 코드를 인터프리팅하기 전에 컴파일 실행  
컴파일레이션 단계 중 모든 선언문을 찾아 적절한 스코프에 연결해주는 과정 존재 - 렉시컬 스코프의 핵심  
변수와 함수 선언문 모두 실제 실해오디지 전에 먼저 처리됨  

자바스크립트가 var a = 2;를 해석하는 과정
- var a;
- a = 2;  
첫째 구문은 선언문 - 컴파일레이션 단계에서 처리  
둘때 구문은 대입문 - 실행 단계까지 내버려둠  
따라서 code1은 다음과 같이 처리
```javascript
var a;
a = 2;
console.log(a);
```
code2 처리 방식
```javascript
var a;
console.log(a);
a = 2;
```

변수와 함수 선언문은 선언된 위치에서 코드의 꼭대기로 '끌어올려'짐 -> 호이스팅(hoisting)  
즉, 선언문이 대입문보다 먼저임  

선언문만 끌어올려지고 다른 대입문이나 실행 로직 부분은 그대로 있음. 호이스팅으로 코드 실행 로직 부분이 재배치된다면 혼란 발생 위험
```javascript
foo();
function foo() {
  console.log(a);
  var a = 2;
}
```
함수 foo()의 선언문이 끌어올려졌기 때문에 foo()를 첫 줄에서 호출 가능  

호이스팅은 스코프별로 작동함  
위 예제를 해석하여 다시 작성한 코드
```javascript
function foo() {
  var a;
  console.log(a);
  a = 2;
}
foo();
```

함수 표현식의 경우
```javascript
foo();
var foo = function bar() {
  // ...
}
```
변수 확인자 foo는 끌어올려져 둘러싼(글로벌) 스코프에 붙으므로 foo() 호출은 실패하지 않지만(ReferenceError가 발생하지 않음), foo가 아직 값을 갖고 있지 않는데 foo()가 undefined 값을 호출하려고 하기 때문에 TypeError가 발생  

함수 표현식이 이름을 가져도 그 이름 확인자는 해당 스코프에서 찾을 수 없음
```javascript
foo();
bar();
var foo = function bar() {
  //...
}
```
호이스팅을 적용하여 해석한 코드
```javascript
var foo;

foo(); // TypeError
bar(); // ReferenceError

foo = function() {
  var bar = ...self...
  // ...
}
```

## 4.3 함수가 먼저다
함수가 먼저 끌어올려지고 다음으로 변수가 먼저 올라감
```javascript
foo();
var foo;

function foo() {
  console.log(1);
}

foo = function() {
  console.log(2);
};
```
다음과 같이 해석됨
```javascript
function foo() {
  console.log(a);
}
foo();

foo = function() {
  console.log(2);
};
```
var foo는 중복 선언문  
var foo는 function foo() 선언문보다 앞서 선언됐지만, 함수 선언문이 일반 변수 위로 끌어올려짐  
중복 함수 선언문은 앞선 것들을 겹쳐 씀
```javascript
foo();

function foo() {
  console.log(1);
}

var foo = function() {
  console.log(2);
};

function foo() {
  console.log(3);
}
```
일반 블록 안에서 보이는 함수 선언문은 보통 둘러싼 스코프로 끌어올려지나, 그렇지 않은 경우도 있음  
-> 다음과 같은 동작이 버전 변경에 따라 바뀔 수 있기 때문에 블록 내 함수 선언은 지양하는 것이 좋음
```javascript
// TypeError 발생
foo();

var a = true;
if (a) {
  function foo() { console.log("a"); }
} else {
  function foo() { console.log("b"); }
}
```