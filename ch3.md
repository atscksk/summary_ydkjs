# 3 네이티브

네이티브: 특정 환경에 종속되지 않은, ECMAScript 명세의 내장 객체  
가장 많이 쓰이는 네이티브  
내장 함수
- String()
- Number
- Boolean()
- Array()
- Object()
- Function()
- RegExp()
- Date()
- Error()
- Symbol() - ES6에서 추가됨

```javascript
var s = new String("Hello World");
console.log(s.toString());
```
생성자처럼 사용할 수 있으나 객체 래퍼가 리턴됨
```javascript
var a = new String("abc");
console.log(typeof a);
console.log(a instanceof String);
console.log(Object.prototype.toString.call(a));
```

객체 래퍼의 형태는 브라우저마다 다르게 생김
```javascript
var a = new String("abc");
console.log(a);
```
결론: new String("abc")은 "abc"를 감싸는 문자열 래퍼를 생성, 원시값 "abc"는 아님
```javascript
var a1 = new String("abc");
var a2 = new String("abc");
var b1 = "abc";
var b2 = "abc";
console.log(a1 === a2);
console.log(b1 === b2);
```

## 3.1 내부 [[Class]]
typeof가 'object'인 값에는 [[Class]]라는 내부 프로퍼티가 추가로 붙음  
직접 접근할 수 없고 Object.prototype.toString() 메서드에 값을 넣어 호출  
```javascript
var r1 = Object.prototype.toString.call([1, 2, 3]);
console.log(r1);
var r2 = Object.prototype.toString.call(/regex-literal/i);
console.log(r2);
```
대부분 내부 Class는 해당 값과 관련된 내장 네이티브 생성자를 가리키지만 그렇지 않을 때도 있음  

원시 값의 내부 Class  
null, undefined: Null(), Undefined()같은 네이티브 생성자는 없으나 내부 Class 값이 존재
```javascript
var n = Object.prototype.toString.call(null);
console.log(n);
var u = Object.prototype.toString.call(undefined);
console.log(u);
```

그 밖의 문자열, 숫자, 불리언 등의 단순 원시 값은 '박싱'과정을 거침
```javascript
var s = Object.prototype.toString.call("abc");
console.log(s);
var n = Object.prototype.toString.call(42);
console.log(n);
var b = Object.prototype.toString.call(true);
console.log(b);
```
내부 Class 값이 각각 String, Number, Boolean으로 표시  
=> 단순 원시 값은 해당 객체 래퍼로 자동 박싱됨

## 3.2 래퍼 박싱하기
원시 값에는 프로퍼티나 메서드가 없으므로 .length, .toString()으로 접근하기 위해 원시 값을 객체 래퍼로 감싸야 함  
자바스크립트는 원시 값을 자동 박싱(래핑)함
```javascript
var a = "abc";
a.length;
a.toUpperCase();
```
원시 값의 프로퍼티/메서드를 빈번하게 사용하면서 직접 객체 형태로 쓰지 않고 암시적 객체 생성이 작동되는 이유  
만약 개발자가 직접 객체 형태로 '선 최적화'한다면 프로그램 성능이 저하될 수 있음  

### 3.2.1 객체 래퍼의 함정
객체 래퍼 사용 시 유의점  
Boolean의 예
```javascript
var a = new Boolean(false);
if(!a) {
  console.log("Oops"); // 실행x
}
```
false를 객체 래퍼로 감싸면 'truthy'  
다음의 결과는 true가 출력됨
```javascript
var a = new Boolean(false);
console.log(Boolean(a));
```
수동으로 원시 값 박싱하기: Object() 함수 사용  
```javascript
var a = "abc";
var b = new String(a);
var c = Object(a);

console.log(a); // "string" 
console.log(b); // "object"
console.log(c); // "objcec"

console.log(b instanceof String);
console.log(c instanceof String);

var b1 = Object.prototype.toString.call(b);
var c1 = Object.prototype.toString.call(c);
console.log(b1);
console.log(c1);
```
객체 래퍼로 직접 박싱하는 것은 필요한 경우가 간혹 있으나 권장되지 않음

## 3.3 언박싱
valueOf()를 통해 객체 래퍼의 원시 값 추출
```javascript
var a = new String("abc");
var b = new Number(42);
var c = new Boolean(true);
console.log(a.valueOf(), b.valueOf(), c.valueOf());
```
암시적 언박싱  
b에는 언박싱된 원시 값 "abc"가 대입됨
```javascript
var a = new String("abc");
var b = a + "";
console.log(typeof a, typeof b);
```

## 3.4 네이티브 - 생성자
배열, 객체, 함수, 정규식 값은 일반적으로 리터럴 형태로 생성  
리터럴은 생성자 형식으로 만든 것과 동일한 종류의 객체를 생성함 - 즉, 래핑되지 않은 값은 없음  
확실히 필요한 상황이 아니라면 생성자 사용은 가급적 사용을 자제

### 3.4.1 Array()
```javascript
var a = new Array(1, 2, 3);
var b = [1, 2, 3];
console.log(a);
console.log(b);
```
Array 생성자: 배열의 크기를 미리 정하는 기능  
인자로 하나의 숫자를 받으면 원소가 하나인 배열을 생성하지 않고, 숫자의 크기의 배열을 생성  
```javascript
var a = new Array(3);
console.log(a.length);
```

생성자로 생성한 배열, undefined로 구성된 배열, length를 통해 빈 슬롯으로 구성한 배열 비교
```javascript
var a = new Array(3);
var b = [ undefined, undefined, undefined ];
var c = [];
c.length = 3;
console.log(a); // [ <3 empty items> ] (크롬: [empty × 3])
console.log(b); // [ undefined, undefined, undefined ]
console.log(c); // [ <3 empty items> ] (크롬: [empty × 3])
```
파이어폭스에서 실행 시 a, c는 [ , , , ]  

a와 b는 실제로 다른 형태
```javascript
var a = new Array(3);
var b = [ undefined, undefined, undefined ];
a.join("-");
b.join("-");
console.log(a);
console.log(b);

var a1 = a.map(function(v, i){ return i; });
var b1 = b.map(function(v, i){ return i; });
console.log(a1);
console.log(b1);
```
a는 슬롯이 없기 때문에 map() 함수가 순회할 원소가 없음  
join()은 슬롯이 있다는 가정하에 length만큼 루프를 반복함  
-> map 함수는 이러한 가정을 하지 않기 때문에 빈 슬롯 배열이 입력되면 출력에 실패함

undefined 값이 원소로 채워진 배열 생성하기
```javascript
var a = Array.apply(null, { length: 3 });
console.log(a);
```
apply(): 모든 함수에서 사용 가능한 유틸리티  
첫 번째 인자 this: 객체 바인딩  
두 번째 인자: 인자의 배열 또는 유사배열. 원소들이 펼쳐져서 함수의 인자로 전달됨  
=> Array.apply()는 Array() 함수를 호출함과 동시에 { length: 3 } 객체 값을 펼쳐 인자로 투입  
apply() 내부에서는 0에서 length 직전까지 루프를 순회, 슬롯의 값이 존재하지 않기 때문에 모두 undefined를 반환

### 3.4.2 Object(), Function(), and RegExp()
Object(), Function(), RegExp() 생성자 사용도 선택사항(권장되지 않음)
```javascript
var c = new Object();
c.foo = "bar";
console.log(c);

var d = { foo: "bar" };
console.log(d);

var e = new Function("a", "return a * 2;");
var f = functiona(a) { return a * 2 };
function g(a) { return a * 2; };
console.log(e(2));
console.log(f(2));
console.log(g(2));

var h = new RegExp("^a*b+", "g");
var i = /^a*b+/g;
console.log(h);
console.log(i);
```
new Object()와 같은 생성자 폼은 사용할 일이 거의 없음  
Function 생성자는 함수의 인자나 내용을 동적으로 정의해야 하는 경우에 한해 유용 - 매우 제한적인 상황  
정규 표현식은 리터럴 형식으로 정의하는 것이 권장됨. 구문이 쉬우며, 성능상 이점  
=> 자바스크립트 엔진이 실행 전 정규 표현식을 미리 컴파일한 후 캐시  
정규표현식 생성자는 패턴을 동적으로 정의할 경우 유용
```javascript
var name = "Kyle";
var namePattern = new RegExp("\\b(?:" + name + ")+\\b", "ig");
var matches = "Kyle asdfg".match(namePattern);
console.log(matches);
```
new RegExp("패턴", "플래그") 형식으로 사용

### 3.4.3 Date(), Error()
리터럴 형식이 없으므로 다른 네이티브에 비해 유용  

date 객체  
new Date()로 생성  
날짜와 시각을 인자로 받음  
인자 생략 시 현재 날짜 / 시각  
유닉스 타임스탬프 값(1970년 1월 1일부터 현재까지 흐른 시간을 초 단위로 환산)을 얻는 용도 - getTime()  
Date.now()

Error() 생성자  
앞에 new 유무 상관없이 결과는 동일  
실행 스택 콘텍스트를 포착하여 객체(자바스크립트 엔진 대부분이 읽기 전용 프로퍼티인 .stack으로 접근 가능)에 담는 용도  
이 실행 콘텍스트는 함수 호출 스택, error 객체가 만들어진 줄 번호 등 디버깅에 도움이 될 만한 정보를 포함  
일반적으로 throw 연산자와 함께 사용
```javascript
function foo(x) {
  if (!x) {
    throw new Error("no x!");
  }
}
var x = false;
foo(x);
```
Error 객체 인스턴스에는 message 프로퍼티와 type 등 읽기 전용 프로퍼티가 포함  
그러나 읽기 편한 포멧으로 에러 메시지를 확인하려면 stack 프로퍼티 대신 error 객체의 toString() 사용

### 3.4.4 Symbol()
Symbol: ES6에서 추가된 새로운 원시 값 타입  
충돌 염려 없이 객체 프로퍼티로 사용 가능한 특별한 '유일 값'  
주로 ES6의 특수한 내장 로직에 쓰기 위해 고안됨  
Symbol.create, Symbol.iterator 와 같이 Symbol 함수 객체의 정적 프로퍼티로 접근  
심벌은 객체가 아니고 단순한 스칼라 원시 값임  
사용법
```javascript
obj[Symbol.iterator] = function(){/**/}
```
직접 정의하기 위해 Symbol() 네이티브를 사용  
new를 붙이면 에러가 나는 유일한 네이티브 생성자
```javascript
var mysym = Symbol("my own symbol");
console.log(mysym);
console.log(mysym.toString());
console.log(typeof mysym);

var a = {};
a[mysym] = "foobar";
var o = Object.getOwnPropertySymbols(a);
console.log(o);
```
private(전용) 프로퍼티는 아니지만 사용 목적에 맞게 대부분 전용 또는 특별한 프로퍼티로 사용  
추후 언더스코어(_, 전용/특수/내부 프로퍼티임을 표기하기 위해 사용됨)를 대체할 가능성

### 3.4.5 네이티브 프로토타입
내장 네이티브 생성자는 각자의 .prototype 객체를 보유  
prototype 객체에는 해당 객체의 하위 타입별로 고유한 로직이 포함  

String 객체의 예
- indexOf(): 문자열에서 특정 문자의 위치를 검색
- charAt(): 문자열에서 특정 위치의 문자를 반환
- substr(), subString(), slice(): 문자열의 일부를 새로운 문자열로 추출
- toUpperCase() / toLowerCase(): 대문자 / 소문자로 변환된 새로운 문자열 생성
- trim(): 앞/뒤 공란이 제거된 새로운 문자열 생성
prototype delegation에 의해 모든 문자열이 위 메서드를 같이 쓸 수 있음
```javascript
var a = " abc ";
console.log(a.indexOf("c"));
console.log(a.toUpperCase());
console.log(a.trim());
```

```javascript
var a = typeof Function.prototype;
console.log(a);
console.log(Function.prototype());

var b = RegExp.prototype.toString(); // 빈 regex
console.log(b);
var b1 = "abc".match(RegExp.prototype);
console.log(b1);
```

네이티브 프로토타입 변경 - 권장되지 않음
```javascript
Array.isArray(Array.prototype); // true
Array.prototype.push(1, 2, 3);
console.log(Array.prototype);

Array.prototype.length = 0;
```

Function.prototype - 함수
RegExp.prototype - 정규 표현식
Array.prototype - 배열

#### 프로토타입은 디폴트
변수에 적절한 타입의 값이 할당되지 않은 상태에서는
- Function.prototype - 빈 함수
- RegExp.prototype - 빈 정규 표현식(아무것도 매칭하지 않음)
- Array.prototype - 빈 배열
```javascript
function isThisCool(vals, fn, rx) {
  vals = vals || Array.prototype;
  fn = fn || Function.prototype;
  rx = rx || RegExp.prototype;

  return rx.test(vals.map(fn).join(""));
}
// var a = isThisCool(); // 실행 안됨
// console.log(a);

// var b = isThisCool(["a", "b", "c"], function(v){ return v.toUpperCase(); }, /C/);
var b = isThisCool(["a", "b", "c"], function(v){ return v.toUpperCase(); }, /D/);
console.log(b);
```
