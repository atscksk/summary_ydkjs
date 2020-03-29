# ch3 함수 vs 블록 스코프

## 3.1 함수 기반 스코프
함수 스코프 용례
```javascript
function foo(a) {
  var b = 2;
  function bar() {
    //...
  }
  var c = 3;
}
```
foo() 스코프 버블은 확인자 a, b, c와 bar를 포함  
bar()는 자체 스코프 버블이 있고, 글로벌 스코프 또한 스코프 버블을 지님  
글로벌 스코프는 foo 확인자 포함  
foo() 밖에서는 a, b, c, bar에 접근할 수 없음
따라서 다음의 코드는 ReferenceError 발생
```javascript
bar();
console.log(a, b, c);
```
foo() 안에서는 모두 접근 가능하며 bar() 내에서도 내부의 섀도 확인자가 선언되지 않았다는 가정 하에 접근 가능  
```javascript
function foo(a) {
  var b = 2;
  function bar() {
    console.log(c);
  }
  var c = 3;
  bar();
}
foo(1)
```
함수 스코프는 모든 변수가 함수에 속하고 함수 전체에 걸쳐 사용되며 재사용됨  
이러한 특성은 자바스크립트 변수의 동적 특성을 살릴 수는 있지만 스코프 전체에서 변수가 살아있다는 점이 예상치 못한 문제를 유발 가능  

## 3.2 일반 스코프에 숨기
함수에 대한 전통적인 개념
- 함수를 선언하고 그 안에 코드 삽입
- 작성한 코드에서 임의 부분을 함수 선언문에 포함 -> 해당 코드를 '숨기는' 효과  

해당 코드 주위에 새로운 스코프 버블이 생성됨. 즉, 감싸진 코드 안에 있는 모든 변수 또는 함수 선언문은 이전 코드에 포함됐던 스코프가 아니라 새로이 코드를 감싼 함수의 스코프에 묶임  

최소 권한의 원칙 - 스코프를 이용한 숨기기 방식을 사용하는 이유와 관련  
-> 모듈/객체의 API와 같은 소프트웨어 설계 시 필요한 것만 최소한으로 남기고 나머지는 숨김
어떤 스코프가 변수와 함수를 포함하는지에 관한 문제와 관련  
모든 변수와 함수가 글로벌 스코프에 존재한다면 중첩된 하위 스코프에서도 접근이 가능해짐 -> 최소 권한의 원칙 위배, 접근할 필요가 없어서 비공개로 남겨둬야할 많은 변수나 함수를 노출시키게 됨
```javascript
function doSomething(a) {
  b = a + doSomethingElse(a * 2);
  console.log(b * 3);
}

function doSomethingElse(a) {
  return a - 1;
}

var b;
doSomething(2);
```
변수 b와 함수 doSomethingElse - doSomething() 작업에서 '비공개'가 요구되는 부분  
변수 b와 doSomethingElse()에 접근할 수 있도록 하는 것은 불필요하며 위험  
접근 가능 확인자는 생각지 못한 방식으로 사용될 수 있으며 doSomething() 실행에 전제되는 가정들이 훼손될 수 있음  
적절한 설계 - 비공개 부분을 doSomething() 스코프 내부에 숨겨야 함
```javascript
function doSomething(1) {
  function doSomethingElse(a) {
    return a - 1;
  }
  var b;
  b = a + doSomethingElse(a * 2);
  console.log(b * 3);
}
doSomething(2);
```
b와 함수 doSomethingElse()는 외부에서 접근할 수 없음 -> 밖으로부터 영향을 밭지 않고, doSomething()만이 통제 가능  

### 3.2.1 충돌 회피
변수와 함수를 스코프 안에 숨기는 것의 다른 장점 - 같은 이름을 가졌지만 다른 용도를 가진 두 확인자가 충돌하는 것을 피할 수 있음
```javascript
function foo(a) {
  function bar(a) {
    i = 3;
    console.log(a + i);
  }
  for (var i = 0; i < 10; i++) {
    bar(i * 2); // infinite loop
  }
}
foo();
```
i = 3은 foo()에서 for 반복문을 위해 선언된 변수 i의 값을 변경 -> 무한 반복 유발  
변수 i의 값이 3으로 고정되어 i<10 상태에 고정  
bar() 내부 대인문은 확인자의 이름이 무엇이든 지역 변수로 선언해서 사용해야 함 -> var i = 3으로 변경하여 문제 해결  

#### 글로벌 '네임스페이스'
글로벌 스코프에서 변수 충돌이 일어나기 쉬운 경우  
내부/비공개 함수와 변수가 적절하게 숨겨져 있지 않은 여러 라이브러리를 한 프로그램에서 불러올 때 라이브러리들이 서로 쉽게 충돌함  
일반적으로 글로벌 스코프에 하나의 고유 이름을 갖는 객체 선언문을 생성. 이후 해당 라이브러리의 네임스페이스로 이용  
네임스페이스를 통해 최상위 스코프의 확인자가 아니라 속성 형태로 라이브러리의 모든 기능이 노출됨
```javascript
var MyReallyCoolLibrary = {
  awesome: "stuff",
  doSomething: function() {
    //...
  },
  doAnotherThing: function() {
    //...
  }
};
```

#### 모듈 관리
다양한 의존성 관리자를 이용한 '모듈' 접근법  
어떤 라이브러리도 확인자를 글로벌 스코프에 추가할 필요 없이 특정 스코프로부터 의존성 관리자를 이용한 다양한 명시적인 방법으로 확인자를 가져와 사용 가능  
그러나 이런 도구를 사용한다고 해서 렉시컬 스코프 규칙에서 벗어날 수 있는 것은 아님  
의존성 관리자 - 스코프 규칙을 적용해 모든 확인자가 공유 스코프에 누출되는 것을 방지하고, 우발적인 스코프 충돌 예방을 위해 충돌 위험이 없는 비공개 스코프에 확인자를 보관  

## 3.3 스코프 역할을 하는 함수
```javascript
var a = 2;
function foo() {
  var a = 3;
  console.log(a);
}
foo();
console.log(a);
```
이상적인 방식은 아님  
foo()라는 이름의 함수를 선언하야 함 - foo 확인자의 스코프 오염. 함수를 직접 이름으로 호출해야만 감싼 코드 실행 가능  
해결 - 이름 없이 함수 선언 및 자동 실행
```javascript
var a = 2;

(function foo() {
  var a = 3;
  console.log(a);
})();

console.log(a);
```
위 코드에서 함수는 일반적인 선언문이 아닌 함수의 표현식으로 취급됨  
함수 이름 foo는 함수를 둘러싼 스코프에 묶이는 대신 함수 자신의 내부 스코프에 묶임  
함수 이름을 자기 내부에 숨기면 함수를 둘러싼 스코프를 불필요하게 오염시키지 않음  

## 3.3.1 익명 vs 기명
함수 표현식을 콜백 인자로 사용하는 사례
```javascript
setTimeout(function() {
  console.log("I waited 1 second!");
}, 1000);
```
익명 함수 표현식 - 확인자 이름이 없음  
함수 표현식은 이름 없이 사용 가능, 함수 선언문에는 이름이 빠져서는 안됨

익명 함수 표현식은 빠르고 쉽게 입력할 수 있어서 많은 라이브러리와 도구가 이 방식을 권장  
그러나 몇 가지 단점 존재
- 스택 추적 시 표시할 이름이 없어서 디버깅이 어려움
- 이름 없이 함수 스스로 재귀 호출을 하려면 arguments.collee (폐기 예정) 참조가 필요
- 이름은 그 자체가 기능을 설명하고, 쉬운 이해가 가능한 코드 작성에 도움  

인라인 함수 표현식 - 부작용 없이 익명 함수의 단점을 해결
```javascript
setTimeout(function timeoutHandler() {
  console.log("I waited 1 second!");
}, 1000);
```

### 3.3.2 함수 표현식 즉시 호출
```javascript
var a = 2;

(function foo() {
  var a = 3;
  console.log(a);
})();

console.log(a); 
```
함수를 둘러싼 ()는 함수를 표현식으로 변경함  
두 번째 ()는 함수를 실행  
즉시 호출 함수 표현식 - IIFE  
이름이 꼭 필요하지는 않으나 이름을 사용하는 것이 더 좋은 습관
```javascript
var a = 2;
(function IIFE() {
  var a = 3;
  console.log(a);
})();
console.log(a);
```
실행 ()가 함수 바깥 ()의 안에 포함되어도 동일하게 작동
```javascript
var a = 2;
(function IIFE() {
  var a = 3;
  console.log(a);
}());
console.log(a);
```
인자를 넘기는 방식
```javascript
// chrome에서 실행
var a = 2;
(function IIFE(global) {
  var a = 3;
  console.log(a);
  console.log(global.a);
})(window);
console.log(a);
```
기본 확인자 undefined 사용에 의한 문제 - 인자를 undefined라 짓고 인자로 아무 값도 넘기지 않으면 undefined 확인자의 값은 코드 블록 안에서 undefined 값을 지님
```javascript
undefined = true;
(function IIFE(undefined) {
  var a;
  if (a === undefined) {
    console.log("Undefined is safe here!");
  }
})();
```
실행할 함수가 호출문과 넘겨진 인자 뒤쪽에 오는 형태 - UMD(범용 모듈 정의) 프로젝트에서 사용됨
```javascript
var a = 2;
(function IIFE(def) {
  def(window);
})(function def(global) {
  var a = 3;
  console.log(a);
  console.log(global.a);
});
```
def는 후반부에 정의되어 전반부 함수 IIFE에 인자로 투입됨  
인자 함수 def가 호출되고 window가 global 인자로 넘겨짐

## 3.4 스코프 역할을 하는 블록
함수가 아닌 다른 스코프 단위 - 블록 스코프  
```javascript
for (var i = 0; i < 10; i++) {
  console.log(i);
}
```
i를 for 반복문과 관련해서만 사용 -> 변수 i를 for 반복문의 시작부에 선언하는 이유  
블록 스코프의 목적 - 변수를 최대한 사용처 가까이에서 최대한 작은 유효 범위를 갖도록 선언
```javascript
var foo = true;

if (foo) {
  var bar = foo * 2;
  bar = something(bar);
  console.log(bar);
}
```
변수 bar는 if문 안에서만 사용 - if 블록 안에 선언하는 것이 타당, 그러나 var 사용시 변수를 사용하는 위치는 중요하지 않음  
-> 선언된 변수는 항상 둘러싸인 스코프에 속하기 때문  
위 예는 가짜 블록 스코프 - bar를 의도치 않게 다른 곳에서 사용하지 않도록 상기시키는 역할을 할 뿐  
블록 스코프는 최소 권한 노출의 원칙을 확장하여 정보를 함수 안에 숨기고, 나아가 코드 블록 안에 숨기기 위한 도구  
자바스크립트는 블록 스코프를 지원하지 않음  

### 3.4.1 with
with 문 안에서 생성된 객체는 바깥 스코프에 영향을 주지 않고 with문이 끝날 때까지만 존재

### 3.4.2 try/catch
try/catch문 중 catch 부분에서 선언된 변수는 catch 블록 스코프에 속함
```javascript
try {
  undefined();
} catch (err) {
  console.log(err);
}
console.log(err);
```

### 3.4.3 let
ES6에서 추가된 키워드  
선언한 변수에 대해 해당 블록 스코프를 활용
```javascript
var foo = true;
if (foo) {
  let bar = foo * 2;
  bar = something(bar);
  console.log(bar);
}
console.log(bar);
```
let을 이용해 변수를 현재 블록에 붙이는 것은 비명시적 - 변수가 어느 블록 스코프에 속한 것인지 착각하기 쉬움  
블록을 명시적으로 생성하여 문제를 해결  
```javascript
var foo = true;
if (foo) {
  { // explicit block
    let bar = foo * 2;
    bar = something(bar);
    console.log(bar);
  }
}
console.log(bar);
```
호이스팅 - 선언문이 어디에서 선언됐든 속하는 스코프 전체에서 존재하는 것처럼 취급되는 것  
let을 사용한 선언문은 속하는 스코프에서 호이스팅 효과를 받지 않음  
-> let으로 선언된 변수는 실제 선언문 전에는 명백하게 존재하지 않음
```javascript
console.log(bar);
let bar = 2;

// var와의 차이 비교
console.log(foo);
var foo = 2;
```

#### 가비지 콜렉션
블록 스코프 - 메모리 회수를 위한 클러저 및 가비지 콜렉션과도 연관
```javascript
function process(data) {
  // do somthing
}
var someReallyBigData = { /*...*/ };
process(someReallyBigData);
var btn = document.getElementById("my_button");
btn.addEventListener("click", function click(evt) {
  console.log("button clicked");
}, false);
```
someReallyBigData 변수는 click 함수와 관련이 없음 - 이론적으로는 process() 실행 후 많은 메모리를 차지하는 someReallyBigData를 수거  
그러나 click 함수가 해당 스코프 전체의 클로저를 갖고 있기 때문에 엔진은 데이터를 여전히 남겨둠  

블록 스코프는 someReallyBigData가 더는 필요 없다는 사실을 엔진에게 명료하게 알릴 수 있음 - 변수의 영역을 한정
```javascript
function process(data) {
  // do somthing
}

{
  let someReallyBigData = { /*...*/ };
  process(someReallyBigData);
}
var btn = document.getElementById("my_button");
btn.addEventListener("click", function click(evt) {
  console.log("button clicked");
}, false);
```

#### let 반복문
```javascript
for (let i = 0; i < 10; i++) {
  console.log(i);
}
console.log(i);
```
let은 단지 i를 for 반복문에 묶었을 뿐만 아니라 반복문이 돌 때마다 변수를 다시 묶어서 이전 반복의 결과값이 제대로 들어가도록 함  
```javascript
{
  let j;
  for (j = 0; j < 10; j++) {
    let i = j;
    console.log(i);
  }
}
```
let 선언문 - 둘러싼 함수 스코프가 아니라 가장 가까운 임의의 블록에 변수를 붙임  
따라서 이전에 var 선언문을 사용해서 작성된 코드는 함수 스코프와 연계가 있을 수 있으므로 리팩토링 시 단순히 var를 let으로 바꾸는 것 이상의 노력이 필요
```javascript
var foo = true, baz = 10;

if (foo) {
  var bar = 3;
  if (baz > bar) {
    console.log(baz);
  }
  //...
}
```
다음과 같이 리팩토링됨
```javascript
var foo = true, baz = 10;

if (foo) {
  var bar = 3;
  //...
}

if (baz > bar) {
  console.log(baz);
}
```
블록 스코프 변수 사용 시 주의 필요
```javascript
var foo = true, baz = 10;

if (foo) {
  let bar = 3;
  
  if (baz > bar) {
    console.log(baz);
  }
  //...
}
```

### 3.4.4 const
const 역시 블록 스코프를 생성하지만, 선언된 값은 고정됨. 선언 후 값 변경 시도 시 오류 발생
```javascript
var foo = true;

if (foo) {
  var a = 2;
  const b = 3;
  a = 3;
  b = 4;
}

console.log(a);
console.log(b);
```
