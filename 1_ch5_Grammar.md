# 5 문법
자바스크립트 언어에서 구문이 어떻게 작동하는가  

## 5.1 문과 표현식
자바스크립트에서 모든 표현식(expression)은 단일한 특정한 결과값으로 계산됨  
```javascript
var a = 3 * 6;
var b = a;
b;
```
3 * 6은 표현식. 두 번째, 세 번째 줄 또한 표현식  
각 라인은 각각 표현식이 포함된 문(statement)  
var a = 3 * 6, var b = a 두 문은 각각 변수를 선언하므로 선언문(declaration statement)이라고 함  
앞에 var가 빠진 a = 3 * 6, b = a는 할당 표현식(assignment expression)이라고 함  
세 번째 줄은 b가 표현식의 전부지만 이것 또한 완전한 문. 이러한 형태를 표현식 문(expression statement)이라고 함

### 5.1.1 문의 완료 값
모든 문은 완료 값을 가지고 있음  

var b = a의 완료 값  
b = a는 할당 이후의 값(18)  
var 문 자체의 완료 값은 undefined  
콘솔에서 문 실행 시 undefined를 출력 - 콘솔은 실행한 문의 완료 값을 보고함  
{ } 블록의 경우, 내부의 가장 마지막 문/표현식의 완료 값을 자신의 완료 값으로 반환
```javascript
// 콘솔 또는 REPL에서 실행
var b;
if (true) {
  b = 4 + 38;
}
```
블록 내 마지막 문 b = 4 + 38의 완료 값이 42이므로 if 블록의 완료 값도 42를 반환  
즉, 블록의 완료 값은 내부에 있는 마지막 문의 값을 암시적으로 반환한 값  

다음 코드는 작동하지 않음: 문의 완료 값을 포착하여 다른 변수에 할당하기
```javascript
// 콘솔 또는 REPL에서 실행
var a, b;
a = if (true) {
  b = 4 + 38;
}
```
완료 값을 포착하려면 eval() 함수를 사용(eval 사용은 권장되지 않음)
```javascript
// 콘솔 또는 REPL에서 실행
var a, b;
a = eval("if (true) { b = 4 + 38; }");
a;
```
ES7 명세에는 do 표현식이 제안됨
```javascript
// 크롬 콘솔 및 node REPLE에서 실행되지 않음
var a, b;
a = do {
  if (true) {
    b = 4 + 38;
  }
};
a;
```
do{} 표현식은 블록 실행 후 블록 내 마지막 문의 완료 값을 do 표현식 전체의 완료 값으로 반환  
인라인 함수 표현식 안에 감싸서 명시적으로 반환할 필요 없이 문을 표현식처럼 다룸

### 5.1.2 표현식의 부수 효과
대부분의 표현식에는 부수 효과가 없음
```javascript
var a = 2;
var b = a + 3;
```
a + 3 자체는 a 값을 바꾸는 등의 부수 효과는 없음. 단지 b = a + 3 문에서 결과값 5가 b에 할당될 뿐임  

부수 효과를 가진 표현식의 예
```javascript
function foo() {
  a = a + 1;
}
var a = 1;
foo(); // 결과값: undefined, 부수효과: a가 변경됨
```
```javascript
var a = 42;
var b = a++;
```
a++의 역할 2가지: a의 현재 값 42를 반환, a 값을 1 증가

++연산자의 부수효과에 의한 혼동 가능성
```javascript
var a = 42;
var b = a++;

console.log(a);
console.log(b);
```

증가/감소 연산자: 전위/후위 연산자로 사용됨
```javascript
var a = 42;

console.log(a++);
console.log(a);
console.log(++a);
console.log(a);
```
++를 전위 연산자로 사용하면 표현식으로부터 값이 반환되기 전에 부수 효과(1 증가)가 발생  
후위 연산자로 사용 시 값을 반환한 이후에 부수 효과가 발생됨

후위연산 ++식을 ()로 감싸도 부수효과의 캡슐화는 불가능
```javascript
var a = 42;
var b = (a++);

console.log(a);
console.log(b);
```

콤마 연산자를 사용하면 다수의 개별 표현식을 하나의 문으로 연결 가능
```javascript
var a = 42, b;
b = (a++, a);

console.log(a);
console.log(b);
```
a++, a 표현식은 두 번째 a 표현식을 첫 번째 a++ 표현식에서 부수 효과가 발생한 이후에 평가  

delete 연산자: 부수 효과 발생  
객체의 프로퍼티를 없애거나 배열에서 슬롯을 제거할 때 사용  
단독 문(standalone statement)으로 더 많이 사용됨
```javascript
var obj = {
  a: 42
};
console.log(obj.a);
console.log(delete obj.a) // true
console.log(obj.a);
```
delete 연산자의 결과값: 유효한/허용된 연산일 경우 true, 그 외에는 false  
프로퍼티 또는 배열 슬롯을 제거하는 것이 부수 효과  

=할당 연산자
```javascript
var a;
a = 42;
a;
```
a=42문의 실행 결과는 할당된 값 42. a에 할당하는 자체가 본질적으로 부수효과  

할당 표현식/문 실행 시 할당된 값이 완료 값이 되는 작동 원리는 연쇄 할당문에서 특히 유용
```javascript
var a, b, c;
a = b = c = 42;
```

```javascript
function vowels(str) {
  var matches;

  if (str) {
    //모든 모음 추출
    matches = str.match(/[aeiou]/g);

    if (matches) {
      return matches;
    }
  }
}
console.log(vowels("Hello World"));
```
할당 연산자의 부수효과를 이용하여 수정
```javascript
function vowels(str) {
  var matches;

  if (str && (matches = str.match(/[aeiou]/g))) {
    return matches;
  }
}
console.log(vowels("Hello World"));
```

### 5.1.3 콘텍스트 규칙
같은 구문도 위치와 방법에 따라 다른 의미를 갖는 경우가 있음

#### 중괄호
자바스크립트에서 중괄호가 나오는 경우

##### 객체 리터럴
```javascript
// bar(): 앞에서 정의된 함수
var a = {
  foo: bar()
}
```

##### 레이블
```javascript
// bar(): 앞에서 정의된 함수
{
  foo: bar()
}
```
{}는 평범한 코드 블록. 문법적으로 문제 없으며, let 블록 스코프 선언과 함께 쓰이면 유용  
for/while루프, if 조건 등에 붙어있는 코드 블록과 기능적으로 유사  
레이블 점프: goto 와 유사한 기능. continue와 break문이 선택적으로 특정 레이블을 받아 goto처럼 프로그램의 실행흐름을 이동시킴
```javascript
{
  foo: for (var i = 0; i < 4; i++) {
    for (var j = 0; j < 4; j++) {
      // 두 루프의 반복자가 같을 때마다 바깥쪽 루프를 continue
      if (j == i) {
        // 다음 순회 시 'foo'가 붙은 루프로 점프
        continue foo;
      }

      // 홀수 배수는 건너뜀
      if ((j * i) % 2 == 1) {
        continue;
      }
      console.log(i, j);
    }
  }
}
```
continue foo는 "foo라는 레이블 위치로 이동하여 계속 순회하라"는 의미가 아니라 "foo라는 레이블이 붙은 루프의 다음 순회를 계속하라"라는 의미. 따라서 임의적인 goto와는 다름

레이블 break 사용
```javascript
{
  foo: for (var i = 0; i < 4; i++) {
    for (var j = 0; j < 4; j++) {
      if ((i * j) >= 3) {
        console.log("stop", i, j);
        break foo;
      }
      console.log(i, j);
    }
  }
}
```
break foo: "foo라는 레이블 위치로 이동하여 계속 순회하라"는 의미가 아니라 "foo라는 레이블이 붙은 바깥쪽 루프/블록 밖으로 나가 그 이후부터 계속하라"는 의미  
레이블 없은 break로 작성하려면 추가적인 함수 필요  
공유 스코프 변수 접근 등 고려사항 때문에 코드가 복잡해지고 혼란스러워질 수 있음  

비 루프 블록에 적용 - break만 참조 가능  
레이블 break를 통해 레이블 블록 밖으로 나가는 것은 가능, 비 루프 블록을 continue하거나 레이블 없는 break로 블록을 빠져나가는 것을 불가
```javascript
function foo() {
  bar: {
    console.log("Hello");
    break bar;
    console.log("no exec");
  }
  console.log("world");
}
foo();
```
레이블 루프/블록은 가급적 사용하지 않는 것이 좋음 - 함수 호출이 더 유용  

JSON  
{ "a": 42 } 콘솔 입력 - 에러 발생
자바스크립트 레이블은 따옴표로 감싸면 안됨  
JSON은 자바스크립트 구문의 하위집합이지만 그 자체로 올바른 자바스크립트 문법은 아님

##### 블록
```javascript
// 둘 다 결과 같음. 책의 설명과 달리 강제변환이 발생하지 않음
console.log([] + {});
console.log({} + []);
```

##### 객체 분해
ES6부터 분해 할당(destructuring assignment) 시 {}를 사용
```javascript
function getData() {
  return {
    a: 42,
    b: "foo"
  }
}

var { a, b } = getData();
console.log(a, b);
```
다음과 같은 의미
```javascript
var res = getData();
var a = res.a;
var b = res.b;
```
명명된 함수에도 활용 가능. 암시적인 객체 프로퍼티 할당과 비슷한 간편 구문
```javascript
function foo({ a, b, c }) {
  console.log(a, b, c);
}
foo({
  c: [1, 2, 3],
  a: 42,
  b: "foo"
});
```
{}는 전적으로 사용 콘텍스트에 따라 의미가 결정됨. 예상치 못했던 방향으로 해석되는 것을 방지하기 위해 미묘한 차이를 이해하는 것이 중요

##### else if와 선택적 블록
```javascript
if (a) {
  // ...
}
else if (b) {
  // ...
}
else {
  // ...
}
```
else if는 존재하지 않음  
if, else문은 실행문이 하나밖에 없는 경우 블록을 감싸는 {} 생략 가능  
위의 코드는 실제로 다음과 같이 파싱됨
```javascript
if (a) {
  // ...
}
else {
  if (b) {
    // ...
  }
  else {
    // ...
  }
}
```
else 이후의 if (b) {} else {} 는 단일 문이므로 {}로 감싸든 말든 상관 없음  
즉, else if 사용은 표준 스타일 가이드의 위반 사례. 단일 if문과 같이 else를 정의한 것과 같음  

## 5.2 연산자 우선순위
```javascript
var a = 42;
var b = "foo";
var c = [1, 2, 3];

console.log(a && b || c);
console.log(a || b && c);
```
표현식에 연산자가 여러 개 있을 경우의 규칙 -> 연산자 우선순위

```javascript
var a = 42, b;
b = (a++, a);
console.log(a, b);
```
```javascript
var a = 42, b;
b = a++, a;
console.log(a, b);
```
,연산자가 =연산자보다 우선순위가 낮음  
자바스크립트 엔진이 b = a++, a를 (b = a++), a 로 해석  
다수의 문을 연결하는 연산자로 ,를 사용 시 연산자의 우선순위가 최하위

```javascript
if (str && (matches = str.match(/[aeiou]/g))) {
  // ...
}
```
=연산자보다 &&연산자가 우선순위가 높음

```javascript
var a = 42;
var b = "foo";
var c = false;

var d = a && b || c ? c || b ? a : c && b : a;

console.log(d);
```
```javascript
console.log(true || false && false);

console.log((true || false) && false);
console.log(true || (false && false));
```
좌 -> 우 순으로 처리되는 것이 아니라, &&가 ||보다 먼저 처리되는 규칙이 적용됨  
[연산자 우선순위](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/%EC%97%B0%EC%82%B0%EC%9E%90_%EC%9A%B0%EC%84%A0%EC%88%9C%EC%9C%84)


### 5.2.1 단락 평가
&&, || 연산자는 좌측 피연산자의 평가 결과만으로 전체 결과가 이미 결정될 경우 우측 피연산자의 평가를 건너뜀  
a && b에서 a가 false면 b는 평가하지 않음  
a || b에서 a가 true면 b를 평가하지 않음  
-> 불필요한 작업이 줄어듦  
사용 예시
```javascript
function doSomething(opts) {
  if (opts && opts.cool) {
    // ...
  }
}
```
opts && opts.cool에서 opts는 일종의 가드  
만약 opts가 존재하지 않는다면 opts.cool 표현식은 에러  
일단 opts를 먼저 단락 평가 후, 그 결과가 실패면 opts.cool은 자동으로 평가 생략, 결과적으로 에러가 발생하지 않음  

||의 경우
```javascript
function doSomething(opts) {
  if (opts.cache || primeCache()) {
    // ...
  }
}
```

### 5.2.2 삼항연산 내 논리연산
? : 연산자의 우선순위
```javascript
console.log(a && b || c ? c || b ? a : c && b : a);

console.log(a && b || (c ? c || (b ? a : c) && b : a));
console.log((a && b || c) ? (c || b) ? a : (c && b) : a);
```
||는 ? :보다 우선순위가 높음 -> (a && b || c)가 ? :보다 먼저 평가됨

### 5.2.3 결합성
우선순위가 동일한 다수의 연산자의 경우  
어느 쪽에서 그룹핑이 일어나는지에 따라 좌측 결합성 또는 우측 결합성을 지님  
결합성은 처리 방향이 좌 -> 우 또는 우 -> 좌인 것과는 전혀 다른 사항  
&&와 ||는 좌측 결합성 연산자 -> 순서대로 처리되기 때문에 결합성 방향은 중요하지 않음


결합 방향이 어느 쪽인지에 따라 완전히 다르게 작동하는 연산자  
삼항(조건)연산자
```javascript
a ? b : c ? d : e

a ? b : (c ? d : e) // 답
(a ? b : c) ? d : e
```
삼항연산자는 우측부터 결합하기 때문에 두 경우의 결과가 달라짐
```javascript
console.log(true ? false : true ? true : true);
console.log(true ? false : (true ? true : true));
console.log((true ? false : true) ? true : true);

console.log(true ? false : true ? true : false);
console.log(true ? false : (true ? true : false));
console.log((true ? false : true) ? true : false);
```

```javascript
var a = true, b = false, c = true, d = true, e = false;
console.log(a ? b : (c ? d : e));
console.log((a ? b : c) ? d : e);
```
우측 결합성을 가진 삼항연산자는 연쇄적으로 맞물릴 때 주의 필요  

=연산자: 우측 결합성
```javascript
var a ,b, c;
a = b = c = 42;
```
c = 42 -> b = ... -> a 순서로 처리  
a = (b = (c = 42))

```javascript
var a = 42;
var b = "foo";
var c = false;

var d = a && b || c ? c || b ? a : c && b : a;
console.log(d);
```
((a && b) || c) ? ((c || b) ? a : (c && b)) : a  
- (a && b)는 "foo"
- "foo" || c는 "foo"
- 첫 번째 삼항 연산: "foo"는 true? -> o
- (c || b)는 "foo"
- 두 번째 삼항 연산: "foo"는 true -> o
- a는 42

### 5.2.4 결론
규칙에 따라 코딩 vs 명시적 코딩  
연산자 우선순위/결합성과 ( )의 사용을 적절히 배합  
연산자 우선순위를 통해 코드 가독성을 높이고, 혼동을 최소화하기 위해 ( )를 적절히 사용

## 5.3 세미콜론 자동 삽입
ASI(Automatic Semicolon Insertion): 자동 세미콜론 삽입  
세미콜론이 누락된 곳에 엔진이 자동으로 ;을 삽입  
ASI는 새 줄(행 바꿈)에만 적용되며, 어떠한 경우에도 줄 중간에 삽입되는 일은 없음  
자바스크립트 파서는 기본적으로 줄 단위로 파싱, ;이 빠져서 에러가 나면 ;을 넣어보고 타당한 것 같으면 ;를 삽입함
```javascript
var a = 42, b
c;
```
```javascript
var a = 42, b = "foo";
a
b
```
```javascript
var a = 42;
do {
  // ...
} while (a)
a;
```
문 블록(while, for 등)에서는 ;이 필수가 아니기 때문에 ASI 역시 필요 없음  

break, continue, return, yield 키워드에서의 ASI
```javascript
function foo(a) {
  if (!a) return
  a *= 2;
  // ...
}
```
ASI는 return문 끝에 ;을 추론하여 삽입  
```javascript
// return() 의 경우
function foo(a) {
  return (
    a * 2 + 3 / 12
  );
}

// ()를 사용하지 않는 여러 줄의 return
function foo(a) {
  return
    a * 2 + 3 / 12
  ;
}
// return;으로 해석되어 undefined가 반환됨
```

### 5.3.1 에러 정정
ASI 의존에 대한 논쟁  
ASI 찬성 주장: 세미콜론을 반드시 넣어야 하는 경우만 제외하고 생략하여 간결한 코드 작성 지향  
ASI 반대 주장: 개발자가 의도하지 않은 ;의 삽입에 의해 의미가 달라질 수 있고, 그에 의해 바숙련 개발자들의 실수 가능성 증가

## 5.4 에러
자바스크립트는 하위 에러 타입(TypeError, ReferenceError, SyntaxError 등) 뿐만 아니라, 일부 에러는 컴파일 시점에 발생하도록 문법적으로 정의됨  
조기 에러(Early Error)의 조건 존재  
구문상 오류는 아니지만 허용되지 않는 것들도 정의  

정규 표현식의 예
```javascript
var a = /+foo/;
```
구문상 문제가 없지만 올바르지 않은 정규 표현식은 조기 에러를 던짐  

할당 대상은 반드시 식별자여야 함
```javascript
var a;
42 = a;
```

엄격 모드에서의 조기 에러  
함수 인자명의 중복
```javascript
function foo(a, b, a) { } // 정상 실행
function bar(a, b, a) { "use strict"; } // 에러
```
동일한 이름의 프로퍼티가 여러 개 있는 객체 리터럴
```javascript
(function() {
  "use strict";

  var a = {
    b: 42,
    b: 43
  }; // 에러
})
```

### 5.4.1 너무 이른 변수 사용
ES6의 TDZ(Temporal Dead Zone): 아직 초기화를 하지 않아 변수를 참조할 수 없는 코드 영역  
let 블록 스코핑이 대표적인 예
```javascript
{
  a = 2; // ReferenceError
  let a
}
```
let a 선언에 의해 초기화 되기 전 a = 2 할당문이 변수 a에 접근 시도  
그러나 a는 TDZ 내부에 있으므로 에러 발생  

typeof 연산자는 선언되지 않은 변수 앞에 붙여도 오류가 발생하지 않음  
그러나 TDZ 참조 시에는 에러가 발생함
```javascript
{
  typeof a; // undefined
  typeof b; // ReferenceError
  let b;
}
```

## 5.5 함수 인자
TDZ 관련 에러 - ES6 디폴트 인자 값
```javascript
var b = 3;
function foo(a = 42, b = a + b + 5) {
  // ...
}
```
두 번째 할당문에서 아직 TDZ에 남아 있는 b를 참조하려 하기 때문에 에러 발생  

ES6 디폴트 인자 값은 인자를 넘기지 않거나 undefined를 전달했을 때 적용
```javascript
function foo(a = 42, b = a + 1) {
  console.log(a, b);
}
foo();
foo(undefined);
foo(5);
foo(void 0, 7);
foo(null); // null은 a + 1 표현식에서 0으로 강제변환됨
```
인자 값이 없는 것과 undefined 값을 받을 때의 차이점
```javascript
function foo(a = 42, b = a + 1) {
  console.log(arguments.length, a, b, arguments[0], arguments[1]);
}
foo();
foo(10);
foo(10, undefined);
foo(10, null);
```
인자를 넘기지 않았을 경우 디폴트 인자 값이 적용되었지만 arguments 배열에는 원소가 없음  
undefined 인자를 명시적으로 넘길 경우 arguments 배열에도 값이 undefined인 원소가 추가됨  
디폴트 인자 값이 arguments 배열 슬롯과 이에 대응하는 인자 값 간 불일치를 초래
ES5에서의 경우 - 동일한 불일치 발생
```javascript
function foo(a) {
  a = 42;
  console.log(arguments[0]);
}
foo(2);
foo();
```
인자를 넘기면 arguments의 슬롯과 인자가 연결, 항상 같은 값이 할당됨  
인자 없이 호출하면 연결되지 않음  
엄격 모드에서는 절대 연결되지 않음
```javascript
function foo(a) {
  "use strict";
  a = 42;
  console.log(arguments[0]);
}
foo(2);
foo();
```
확실하지 않은 연결에 의존하여 코딩하는 것은 바람직하지 않음  
잘 설계된 기능이 아닌 '구멍 난 추상화'  
auguments 배열은 비 권장 요소 - ES6부터는 rest 인자(...) 권장  
그러나 ES6 이전까지 유용했던 기능  
인자와 그 인자에 해당하는 arguments 슬롯을 동시에 참조하지 말 것 - 이 규칙만 준수한다면 arguments 배열과 인자를 혼용하는 것은 안전

## 5.6 try...finally
try 문에서 catch, finally 중 하나는 필수  
finally절은 다른 블록 코드에 상관없이 필히 실행되어야 할 콜백함수와 같음  
try 절에 return 문이 있는 경우
```javascript
function foo() {
  try {
    return 42;
  } finally {
    console.log("Hello");
  }
  console.log("no exec");
}
console.log(foo());
```
foo() 함수의 완료 값은 42로 세팅되고, try 절의 실행이 종료되면서 바로 finally 절로 넘어감  
그 후 foo() 함수 전체 실행 종료 후 완료 값은 호출부 console.log()문에 반환
try 안에 throw가 있는 경우
```javascript
function foo() {
  try {
    throw 42;
  } finally {
    console.log("Hello");
  }
  console.log("no exec");
}
console.log(foo());
```
finally 절에서 예외가 발생되면 이전의 실행 결과는 모두 무시됨. 즉, 이전 try 블록에서 생성한 완료 값은 모두 사장됨
```javascript
function foo() {
  try {
    return 42;
  } finally {
    throw "throw"
  }
  console.log("no exec");
}
console.log(foo());
```
비선형 제어문(continue, break): return, throw와 유사하게 작동
```javascript
for (var i = 0; i < 10; i++) {
  try {
    continue;
  } finally {
    console.log(i);
  }
}
```
continue문에 의해 console.log() 문은 루프 순회 끝 부분에서 실행됨  
i++로 인해 인덱스가 수정되지 직전까지 실행됨  

finally절의 return문 - 이전에 실행된 try나 catch 절의 return을 덮어쓰는 기능  
단 반드시 명시적으로 return문을 사용해야 함
```javascript
function foo() {
  try {
    return 42;
  } finally {
    // return 없음 -> 이전 return 실행
  }
}

function bar() {
  try {
    return 42;
  } finally {
    // return 42 무시
    return;
  }
}

function baz() {
  try {
    return 42;
  } finally {
    // return 42 무시
    return "Hello";
  }
}

console.log(foo());
console.log(bar());
console.log(baz());
```
일반적인 함수에서는 return을 생략해도 return; 또는 return undefined;를 한 것으로 치지만, finally 안에서 return을 빼면 이전의 return을 무시하지 않고 실행함  

레이블 break와 finally
```javascript
function foo() {
  bar: {
    try {
      return 42;
    } finally {
      // 'bar' 레이블 블록으로 나감
      break bar;
    }
  }
  console.log("abc");
  return "Hello";
}
console.log(foo());
```
finally + 레이블 break 코드: 사실상 return의 취소 - 권장되지 않는 방식

## 5.7 switch
```javascript
switch (a) {
  case 2:
    // ...
    break;
  case 42:
    // ...
    break;
  default:
    //...
}
```
switch 표현식과 case 표현식 간의 매치 과정은 === 알고리즘과 동일  

강제 변환이 일어나는 동등 비교(==) 적용
```javascript
var a = "42";
switch (true) {
  case a == 10:
    console.log("10 또는 '10'");
    break;
  case a == 42:
    console.log("42 또는 '42'");
    break;
  default:
    // ...
}
```
==를 사용해도 switch문은 엄격하게 매치함  
case 표현식 평가 결과가 truthy지만 엄격히 true는 아닐 경우 매치는 실패  
&&나 || 등의 논리 연산자 사용 시 문제
```javascript
var a = "Hello world";
var b = 10;

switch (true) {
  case (a || b == 10):
    // ...
    break;
  default:
    console.log("abc");
}
```
(a || b == 10)의 평가 결과는 true가 아닌 "Hello world" - 매치되지 않음  
이 경우 분명한 true/false로 떨어지게 case !!(a || b == 10): 와 같이 작성해야 함  

default절 - 선택 사항. 그러나 default절에서도 break를 쓰지 않으면 그 후의 코드가 계속 실행됨
```javascript
var a = 10;

switch (a) {
  case 1:
  case 2:
  default:
    console.log("default");
  case 3:
    console.log("3");
  break;
  case 4:
    console.log("4");
}
// case절에서도 레이블 break 사용 가능
```
매치되는 것이 없기 때문에 default를 실행, break가 없으므로 이미 한 번 지나친 case 3: 블록을 다시 실행함
