# 4 강제변환
## 4.1 값 변환
어떤 값을 다른 타입의 값으로 바꾸는 과정이 명시적이면 'type casting', 암시적이면 'coercion(강제변환)'  
박싱은 강제 변환이 아님  
강제변환을 하면 문자열, 숫자, 불리언 등의 원시값 중 하나가 됨  
명시적 강제변환 - 의도적으로 타입변환을 발생시킴  
암시적 강제변환 - 불분명한 부수 효과로부터 발생하는 타입변환
```javascript
var a = 42;
var b = a + ""; // 암시적 강제변환
var c = String(a); // 명시적 강제변환
console.log(a, b, c);
```
공백 문자열과의 +연산은 문자열 접합 처리를 의미, 부수효과로서 숫자를 문자열로 강제변환함  
String() 함수는 값을 인자로 받아 명백하게 문자열 타입으로 강제변환함  

## 4.2 추상 연산
### 4.2.1 ToString
문자열이 아닌 값을 문자열로 변환하는 작업  
내장 원시 값은 본연의 문자열화 방법이 정해져 있음(null -> 'null', undefined -> 'undefined')  
숫자는 너무 작거나 큰 값은 지수 형태로 변경됨
```javascript
var a = 1.07 * 1000 * 1000 * 1000 * 1000 * 1000 * 1000 * 1000;
console.log(a.toString());
```
일반 객체는 특별히 지정하지 않으면 기본적으로 (Object.prototype.toString()) toString() 메서드가 내부 Class를 반환  
배열의 toString(): 모든 원소 값이 분리된 형태로 반환됨
```javascript
var a = [1, 2, 3];
console.log(a.toString());
```

#### JSON 문자열화
ToString은 JSON.stringify()를 통한 JSON 문자열화와 관련  
JSON 문자열화와 toString() 변환은 기본적으로 같은 로직
```javascript
var a = JSON.stringify(42);
var b = JSON.stringify("42");
var c = JSON.stringify(null);
var d = JSON.stringify(true);
console.log(a, b, c, d);
```

JSON 표현형으로 확실히 나타낼 수 있는 값(JSON-safe value)은 모두 JSON.stringify()로 문자열화 가능  
표준 JSON 규격을 벗어난(다른 언어로 이식하여 JSON 값으로 쓸 수 없는) 값은 불가  
인자가 undefined, 함수, 심벌 값이면 자동으로 누락시키며, 배열에 포함되어 있으면 null로 변환시킴
```javascript
var a = JSON.stringify(undefined);
var b = JSON.stringify(function(){});
var c = JSON.stringify([1, undefined, function(){}, 4]);
var d = JSON.stringify({ a: 2, b: function(){} });
console.log(a, b, c, d);
```

JSON.stringify()에 circular reference 객체를 넘기면 에러 발생  
객체 자체에 toJSON() 메서드가 정의되어 있다면 먼저 이 메서드를 호출하여 직렬화한 값을 반환  

부적절한 JSON값이나 직렬화할 수 없는 값을 문자열하려면 toJSON() 메서드를 따로 정의해야 함
```javascript
var o = {};
var a = {
  b: 42,
  c: o,
  d: function(){}
}

// 'a' circular reference
o.e = a;
// JSON.stringify(a); // error

// JSON값으로 직렬화하는 함수를 따로 정의
a.toJSON = function() {
  // b만 포함
  return { b: this.b }
}
console.log(JSON.stringify(a));
```

toJSON()은 문자열을 문자열화할 의도가 아니라면 정확하지 않을 가능성이 높음  
문자열화하기 적당한 JSON 안전 값으로 바꾸는 역할. JSON 문자열로 바꾸는 것이 아님
```javascript
var a = {
  val: [1, 2, 3],
  toJSON: function() {
    return this.val.slice(1);
  }
};

var b = {
  val: [1, 2, 3],
  toJSON: function() {
    return "[" + this.val.slice(1).join() + "]";
  }
};

var a1 = JSON.stringify(a);
var b1 = JSON.stringify(b);
console.log(a1, typeof a1);
console.log(b1, typeof a1);
```

배열 또는 함수 형태의 대체자를 JSON.stringify()의 두 번째 선택 인자로 지정하여 객체를 재귀적으로 직렬화하면서 필터링  
대체자가 배열이면 전체 원소는 문자열이여야 하고 각 원소는 객체 직렬화의 대상 프로퍼티명  
즉 포함되지 않은 프로퍼티는 직렬화 과정에서 빠김  
대체자가 함수면 처음 한 번은 객체 자신에 대해, 그 다음은 각 객체 프로퍼티별로 한 번씩 실행하면서 매번 키와 값 인자를 전달  
직렬화 과정에서 해당 키를 건너뛰려면 undefined를, 그 외엔 해당 값을 반환
```javascript
var a = {
  b: 42,
  c: "42",
  d: [1, 2, 3]
};
var j1 = JSON.stringify(a, ["b", "c"]);
var j2 = JSON.stringify(a, function(k, v) {
  if (k !== "c") return v;
});
console.log(j1);
console.log(j2);
```

JSON.stringify()의 세 번째 선택인자: 스페이스. 읽기 쉽도록 들여쓰기  
들여쓰기를 할 빈 공간의 개수를 숫자로 지정하거나 문자열을 지정하여 각 들여쓰기 수준에 사용  
- 문자열: 10자 이상이면 앞에서 10자까지만 잘라 사용
```javascript
var a = {
  b: 42,
  c: "42",
  d: [1, 2, 3]
}
var a1 = JSON.stringify(a, null, 3);
console.log(a1);
var a2 = JSON.stringify(a, null, "-----");
console.log(a2);
```

JSON.stringify()는 직접적인 강제변환 형식은 아니나 다음의 이유로 ToString 강제 변환과 연관
- 문자열, 숫자, 불리언, null 값이 JSON으로 문자열화하는 방식은 ToString 추상 연산 규칙에 따라 문자열 값으로 강제변환되는 방식과 동일
- JSON.stringify()에 전달한 객체가 자체 toJSON() 메서드를 갖고 있다면, 문자열화 전 toJSON()이 자동으로 호출되어 JSON 안전 값으로 '강제변환'됨

### 4.2.2 ToNumber
true -> 1, false -> 0  
undefined -> NaN  
null -> 0  
문자열 -> 숫자 리터럴. 변환 실패 시 NaN  
0이 앞에 붙은 8진수는 8진수로 처리하지 않고 일반 10진수로 처리함  
객체: 동등한 원시 값으로 변환 후 그 결과값을 변환함
```javascript
var a = {
  valueOf: function() {
    return "42";
  }
};
var b = {
  toString: function() {
    return "42";
  }
}
var c = [4, 2];
c.toString = function() {
  return this.join("");
}
var a1 = Number(a);
var b1 = Number(b);
var c1 = Number(c);
var d = Number("");
var e = Number([]);
var f = Number(["abc"]);
console.log(a1, b1, c1, d, e, f);
```
valueOf()를 쓸 수 있고 반환 값이 원시 값이면 그대로 강제변한하되, 그렇지 않을 경우 toString()을 이용하여 강제변환  
원시값으로 바꿀 수 없을 때는 TypeError 오류를 던짐

### 4.2.3 ToBoolean
1과 0이 각각 true, false에 해당하는 다른 언어와 달리 자바스크립트에서는 숫자와 불리언은 서로 별개  

#### falsy
자바스크립트의 모든 값들은 다음 중 하나임
- 불리언으로 강제변환 시 false가 되는 값
- 위 값을 제외한 나머지. true가 되는 값

ES5 명세에 정의된 falsy 값
- undefined
- null
- false
- +0, -0, NaN
- ""

ToBoolean 연산에 대한 인자의 변환 내용
- undefined: false
- Null: false
- Boolean: 인자 값과 동일
- Number: +0. -0, NaN이면 false, 그 외에는 true
- String: 공백 문자열이면 false, 그 외에는 true
- Object: true
```javascript
console.log(Boolean({}));
```

#### falsy 객체
falsy값을 둘러싼 객체 래퍼는 true
```javascript
var a = new Boolean(false);
var b = new Boolean(0);
var c = new Boolean("");
console.log(Boolean(a&&b&&c));
```
falsy 객체는 순수 자바스크립트의 일부는 아니고 브라우저 환경에서 생성되는 객체의 일부  
document.all  
-> 비표준. 비 권장/폐기됨  
-> document.all에 의존하는 레거시 코드가 존재, 현대 표준 코드 베이스에서 혼선 유발
-> 타입 체계를 변경하여 document.all이 falsy로 동작하도록 수정

#### truthy 값
falsy 값 목록에 없는 모든 값들
```javascript
var a = "false";
var b = "0";
var c = "''";
console.log(Boolean(a&&b&&c));
```
```javascript
var a = [];
var b = {};
var c = function(){};
console.log(Boolean(a&&b&&c));
```

## 4.3 명시적 강제변환
분명하고 확실한 타입변환  

### 4.3.1 문자열 <-> 숫자
String()과 Number() 함수 사용  
new 키워드가 붙지 않기 때문에 객체 래퍼를 생성하는 것이 아님
```javascript
var a = 42;
var b = String(a);

var c = "3.14";
var d = Number(c);

console.log(b);
console.log(d);
```

원시값.toString(): 자바스크립트 엔진이 자동으로 값을 객체 래퍼로 박싱함  
+연산: 피연산자를 숫자로 강제변환
```javascript
var a = 42;
var b = a.toString();

var c = "3.14";
var d = +c;

console.log(b);
console.log(d);
```
헷갈리기 쉬우므로 단항 연산자를 다른 연산자와 인접하여 사용하는 것은 권장되지 않음
```javascript
var a = "3.14";
var b = 5
console.log(d + +c);
```

#### 날짜 -> 숫자
단항연산자 +는 Date 객체를 숫자로 강제변환하는 용도로도 쓰임  
```javascript
var d = new Date("Mon, 18 Aug 2013 08:53:06 CDT");
console.log(d);
console.log(+d); // unix 타임스탬프 표현형
```
현재 시각을 타임스탬프로 바꿈
```javascript
var timestamp = +new Date();
```
강제변환 없이 Date 객체로부터 타임스탬프를 얻어내는 방법(더 명시적이므로 권장됨)
```javascript
var timestamp = new Date().getTime();
console.log(timestamp);

// es5에 추가된 정적 함수 Date.now()
var timestamp2 = Date.now();
console.log(timestamp2);
```
구 버전 브라우저에서 Date.now() 사용하기
```javascript
if (!Date.now) {
  Date.now = function() {
    return +new Date();
  };
}
```
날짜 타입은 강제변환보다는 Date.now() (현재의 타임스탬프), new Date().getTime() (특정 날짜의 타임스탬프) 를 쓰는 게 더 나음

#### 틸드(~)
32비트 숫자로 강제변환 후 NOT 연산(각 비트를 거꾸로 뒤집음)

indexOf(): 단순히 위치를 확인하는 기능보단 어떤 하위 문자열이 다른 문자열에 포함되어 있는지 조사하는 용도로 더 많이 쓰임
```javascript
var a = "Hello World";

// 전부 true
a.indexOf("lo") >= 0;   // found
a.indexOf("lo") != -1;  // found
a.indexOf("ol") < 0;    // not found
a.indexOf("ol") == -1;  // not found
```
indexOf()에 ~를 붙이면 어떤 값을 강제변환하여 불리언 값으로 적절하게 만들 수 있음
```javascript
var a = "Hello World";

~a.indexOf("lo");    // -4 -> true
~a.indexOf("ol");    // 0 -> false
!~a.indexOf("ol");  //  true
```
~은 indexOf()로 검색 결과 실패 시 -1을 0(false값)으로, 그 외에는 true 값으로 변경함

### 4.3.2 숫자 형태의 문자열 파싱
```javascript
var a = "42";
var b = "42px";

var a1 = Number(a);
var a2 = parseInt(a);
console.log(a1, a2);

var b1 = Number(b);
var b2 = parseInt(b);
console.log(b1, b2);
```
문자열로부터의 숫자 값 파싱: 비숫자형 문자를 허용  
왼쪽에서 오른쪽 방향으로 파싱하다가 숫자가 아닌 문자를 만나면 파싱이 중단됨
강제변환: 비숫자형 문자를 허용하지 않고 NaN을 반환  
```javascript
var hour = parseInt(08);
var minute = parseInt(09);
console.log(hour, minute);
```

#### 비문자열 파싱
```javascript
console.log(parseInt(1/0, 19));
```
비문자열이 첫 번째 인자로 투입될 경우, 비문자열을 최대한 문자열로 강제변환하려고 시도함 - 예외를 던지지 않음

```javascript
console.log(parseInt(new String("42")));
```

```javascript
var a = {
  num: 21,
  toString: function() { return String(this.num * 2) }
};
console.log(parseInt(a));
```
parseInt()는 인자 값을 강제로 문자열로 바꾼 다음 파싱을 시작하는 로직

parseInt(1/0, 19) => parseInt("Infinity", 19)  
첫 번째 문자 "I"는 19진수 18에 해당, 두 번째 "n"은 0-9 a-i (19진수의) 범위 밖의 문자이므로 파싱은 멈춤

```javascript
var a = parseInt(0.000008);
console.log(a);
// 0.000008 -> 0

var b = parseInt(0.0000008);
console.log(b);
// 8e-7 -> 8

var c = parseInt(false, 16);
console.log(c);
// false -> fa

var d = parseInt(parseInt, 16);
console.log(d);
// function... -> f

var e = parseInt("0x10");
console.log(e);
// 16

var f = parseInt("103", 2);
console.log(f);
// 2
```

### 4.3.3 비 불리언 -> 불리언
Boolean(): 명시적이지만 자주 쓰이지는 않음
```javascript
var a = Boolean("0");
var b = Boolean([]);
var c = Boolean({});
console.log(a, b, c);

var d = Boolean("");
var e = Boolean(0);
var f = Boolean(null);
var _g
var g; = Boolean(_g);
console.log(d, e, f, g);
```

! 부정 단항 연산자는 값을 불리언으로 명시적 강제변환  
그러나 그 과정에서 truthy, falsy까지 바뀌기 때문에 !! 형태로 이중부정 연산자를 사용
```javascript
var a = "0";
var b = [];
var c = {};
console.log(!!a, !!b, !!c);

var d = "";
var e = 0;
var f = null;
var g
console.log(!!d, !!e, !!f, !!g);
```

## 4.4 암시적 변환
부수 효과(변환)가 명확하지 않게 숨겨진 형태로 일어나는 타입 변환  
분명하지 않은 타입변환  
목적: 코드의 장황함, 보일러플레이트, 불필요한 상세 구현을 줄이는 것

### 4.1.1 '암시적'의 의미
엄격한 타입 언어상 이론적 의사코드의 예: 임의의 타입 y값을 SomeType으로 바꾸기  
SomeType x = SomeType(AnotherType(y))  
y의 값과 무관하게 곧바로 SomeType으로 변환될 수 없음 - 중간단계 필요  
먼저 AnotherType으로 변환된 뒤 SomeType으로 최종 변환  
=> 상세한 변화과정이 표면적으로 드러날 필요 없음  
=> 중간 단계가 숨겨짐으로써 코드 가독성 향상 및 세부적인 구현부의 추상화에 도움

### 4.4.2 문자열 <-> 숫자
+연산자: 숫자의 덧셈, 문자열 접합
```javascript
var a = "42";
var b = "0";

var c = 42;
var d = 0;

console.log(a + b);
console.log(c + d);
```
+연산자에 의한 문자열 접합과 숫자 간 덧셈의 차이  
피연산자가 한쪽 또는 양쪽 모두 문자열인지 아닌지에 따라 문자열 붙이기 여부가 결정되는 것이 아님
```javascript
var a = [1, 2];
var b = [3, 4];
console.log(a + b);
```
피연산자 모두 문자열이 아님에도 문자열로 강제변환된 후에 접합  
-> ToNumber 추상 연산이 객체를 다루는 방법과 일치  
valueOf()에 배열을 넘기면 단순 원시 값을 반환할 수 없으므로 다음으로 toString()이 처리  
두 배열은 각각 "1, 2", "3, 4"가 되고 +에 의해 두 문자열이 합쳐져 최종적으로 "1, 23, 4"가 됨  
```javascript
var a = 42;
var b = a + "";
console.log(b);
```
숫자는 공백 문자열 ""와 더하면 문자열로 강제변환됨  
a + ""는 valueOf()에 값을 전달하여 호출하고 그 결과값은 ToString 추상연산을 통해 문자열로 변환됨  
그러나 String(a)는 toString()을 직접 호출함  
원시 숫자 값이 아닌 객체라면 결과값이 달라질 수 있음
```javascript
var a = {
  valueOf: function() { return 42; },
  toString: function() { return 4; }
};
console.log(a + "");
console.log(String(a));
```
valueOf()와 toString()을 직접 구현한 객체가 있다면 강제변환 과정에서 결과값이 달라질 수 있음을 유의  

문자열 -> 숫자
```javascript
var a = "3.14";
var b = a - 0;
console.log(b);
```
-, *, / 연산자는 숫자 뺄셈, 곱셈, 나눗셈 기능밖에 없음 -> a를 숫자로 강제변환  

객체 값에 -연산
```javascript
var a = [3];
var b = [1];
console.log(a - b);
```
+연산과 유사. 각 배열은 문자열로 강제변환된 뒤 숫자로 강제변환되고 마지막으로 -연산이 수행됨

### 4.4.3 불리언 -> 숫자
복잡한 형태의 불리언 로직을 숫자 덧셈 형태로 단순화할 때 유용

```javascript
function onlyOne(a, b, c) {
  return !!((a && !b && !c) || (!a && b && !c) || (!a && !b && c)) ;
}
var a = true;
var b = false;

var r1 = onlyOne(a, b, b);
var r2 = onlyOne(b, a, b);
var r3 = onlyOne(a, b, a);
console.log(r1, r2, r3);
```
세 인자 중 정확히 하나만 true인지 아닌지를 확인하는 함수  
true 체크 시 암시적 강제변환을 하고 최종 한환 값을 포함한 다른 부분은 명시적 강제변환  
인자 수가 많아지면 코드가 복잡해짐 -> 불리언 값을 숫자로 변환하면 쉽게 해결
```javascript
function onlyOne() {
  var sum = 0;
  for (var = 0; i < arguments.length; i++) {
    // false 값은 건너뜀
    // 0으로 취급. 그러나 NaN은 피해야 함
    if (arguments[i]) {
      sum += arguments[i];
    }
  }
  return sum == 1;
}
var a = true;
var b = false;

var r1 = onlyOne(b, a);
var r2 = onlyOne(b, a, b, b, b);
var r3 = onlyOne(b, b);
var r4 = onlyOne(b, a, b, b, b, a);
console.log(r1, r2, r3, r4);
```
true를 숫자로 강제변환하면 1  
sum += arguments[i]에서 암시적 강제변환이 발생  

명시적 강제변환 버전
```javascript
function onlyOne() {
  var sum = 0;
  for (var i = 0; i < arguments.length; i++) {
    sum += Number(!!arguments[i]);
  }
  return sum === 1;
}
```
!!arguments[i]를 통해 인자 값을 true/false로 강제변환하기 때문에 비불리언 값을 넘겨도 동작됨

### 4.4.4 불리언 값으로의 암시적 강제변환
불리언으로의 암시적 강제변환이 일어나는 표현식: 불리언이 아닌 값이 사용되면 ToBoolean 연산 규칙에 따라 불리언 값으로 암시적 강제변환됨
- if()문
- for(;;)에서 두 번째 조건
- while() 및 do~while()
- ? : 삼항 연산 시 첫 번째 조건
- || 및 &&의 좌측 피연산자
```javascript
var a = 42;
var b = "abc";
var c;
var d = null;

if(a) {
  console.log('Yes');
}
while(c) {
  console.log('Never');
}

c = d ? a : b;
console.log(c);

if((a && d) || c) {
  console.log('Yes');
}
```

### 4.4.5 &&와 || 연산자
타 언어의 '논리 연산자'로서의 기능 이상의 기능을 보유 -> '선택 연산자' 또는 '피연산자 선택 연산자'  
=> 다른 언어와 달리 실제 결과값이 논리(불리언) 값이 아니기 때문 - 두 피연산자의 값들 중 하나가 선택됨
```javascript
var a = 42;
var b = "abc";
var c = null;

console.log(a || b);
console.log(a && b);
console.log(c || b);
console.log(c && b);
```
우선 피연산자의 불리언 값을 평가, 피연산자가 비 불리언 타입이면 먼저 강제변환 후 평가를 계속  
|| 연산자는 그 결과가 true면 첫 번째 피연산자 값을, false면 두 번째 피연산자 값을 반환  
&& 연산자는 true면 두 번째 피연산자 값을, false면 첫 번째 피연산자 값을 반환  
결과값은 항상 피연산자의 값 중 하나임. 강제변환된 평가의 결과가 아님  
c && b에서 c는 null이므로 false. 그러나 && 표현식은 평과 결과인 false가 아니라 c의 값인 null에 의해 판별  
다음의 과정으로 이해
```javascript
var a = 42;
var b = "abc";

var r1 = a || b;
var r2 = a ? a : b;

var r3 = a && b;
var r4 = a ? b : a;

console.log(r1, r2);
console.log(r3, r4);
```
활용 예시
```javascript
function foo(a, b) {
  a = a || "hello";
  b = b || "world";

  console.log(a + " " + b);
}
foo();
foo("Oh my", "God!");

// 주의
foo("That's it!", "");
```
주의  
두 번째 인자가 false값이므로 b = b || "world"에서 디폴트 값 "world"가 할당  
falsy값은 무조건 건너뛸 경우에만 사용. 그렇지 않을 시 조건 평가식을 삼항 연산자로 더욱 명시적으로 지정해야 함  

가드연산자: &&연산자의 특성  
첫 번째 피연산자의 평가 결과가 true일 때에만 두 번째 피연산자를 선택  
첫 번째 표현식이 두 번째 표현식의 '가드' 역할
```javascript
function foo() {
  console.log(a);
}
var a = 42;
a && foo();
```
a가 true일 때에민 foo()가 호출됨  

if문 또는 for문 상에서의 복합 논리 표현식 작동  
먼저 복합 표현식이 평가된 후 불리언으로 암시적 강제변환이 일어남
```javascript
var a = 42;
var b = null;
var c = "foo";

if (a && (b || c)) {
  console.log("Yes");
}
```
a && (b || c) 표현식의 실제 결과는 true가 아닌 "foo"  

### 4.4.6 심벌 강제변환
심벌 -> 문자열의 명시적 강제변환은 허용되나 암시적 강제변환은 금지, 시도만 해도 에러 발생
```javascript
var s1 = Symbol("Good");
var sb1 = String(s1);
console.log(sb1);

var s2 = Symbol("bad");
var sb2 = s2 + "";
console.log(sb2);
```
심벌 값은 숫자로 변환되지 않음(양방향 모두 에러)  
불리언 값으로는 명시적/암시적 강제변환(항상 true) 가능  

## 4.5 느슨한/엄격한 동등 비교
느슨한 동등 비교는 == 연산자, 엄격한 동등 비교는 === 연산자를 사용  
동등함 비교 시 ==는 강제변환을 허용, ===는 강제변환을 허용하지 않음

### 4.5.1 비교 성능
타입이 같은 두 값의 동등 비교 시, ==와 ===는 동일한 알고리즘으로 동작  
강제변환이 필요하다면 느슨한 동등 연산자를, 필요하지 않다면 엄격한 동등 연산자를 사용

### 4.5.2 추상 동등 비교
비교할 두 값이 같은 타입이라면 값에 대한 동등 비교  
값의 동등함에 대한 예외
- NaN은 그 자신과도 동등하지 않음
- +0과 -0은 동등하지 않음  
객체의 느슨한 동등비교 시 두 객체가 정확히 똑같은 값에 대한 참조일 경우에만 동등

#### 문자열 -> 숫자
```javascript
var a = 42;
var b = "42";

console.log(a === b);
console.log(a == b);
```
느슨한 동등 비교 시 피연산자의 타입이 다르면, 비교 알고리즘에 의해 한쪽 또는 양쪽 피연산자 값이 알아서 암시적 강제변환됨  
- Type(x)가 Number고 Type(y)가 String이면, x == ToNumber(y) 비교 결과를 반환
- Type(x)가 String이고 Type(y)가 Number면, ToNumber(x) == y 비교 결과를 반환  
명세 상, 문자열이 숫자로 강제변환: 강제변환은 ToNumber 추상 연산이 담당  

#### 불리언 변환
```javascript
var a = "42";
var b = true;
console.log(a == b);

var c = true;
var d = "42";
console.log(c == d);
```
- Type(x)가 불리언이면 ToNumber(x) == y의 비교 결과를 반환
- Type(y)가 불리언이면 x == ToNumber(y)의 비교 결과를 반환  
ToBoolean은 전혀 개입하지 않으며 숫자 값의 true/false 여부는 == 연산과는 전혀 무관  
==의 피연산자 한쪽이 불리안 값이면 해당 값이 먼저 숫자로 강제변환됨

===는 true/false의 강제변환을 허용하지 않음 => 불리언 비교 시 느슨한 동등비교는 사용하지 않아야 함
```javascript
var a = "42";

if (a == true) {
  console.log("1");
}

if (a === true) {
  console.log("2");
}

if (a) {
  console.log("3");
}

if (!!a) {
  console.log("4");
}

if (Boolean(a)) {
  console.log("5");
}
```

#### null -> undefined
null과 undefined 간 느슨한 동등비교 시 암시적 강제변환 발생  
- x가 null이고 y가 undefined면 true 반환
- x가 undefined이고 y가 null이면 true 반환  

null과 undefined의 느슨한 동등비교 시 서로에게 타입을 맞춤. 상호 암시적인 강제변환 발생
```javascript
var a = null;
var b;

console.log(a == b);
console.log(a == null);
console.log(b == null);

console.log(a == false);
console.log(b == false);
console.log(a == "");
console.log(b == "");
console.log(a == 0);
console.log(b == 0);
```
null <-> undefined 강제변환은 안전하고 예측 가능하며, 다른 값과의 비교 결과 긍정 오류 가능성이 없음  
동일한 값으로 취급하는 강제변환은 권장  
가독성이 좋으며, 안전하게 작동하는 암시적 강제변환의 예시  

#### 객체 -> 비객체
- Type(x)가 String 또는 Number고 Type(y)가 객체라면, x == ToPrimitive(y)의 비교 결과를 반환
- Type(x)가 Object이고 Type(y)가 String 또는 Number라면, ToPrimitive(x) == y의 비교 결과를 반환
```javascript
var a = 42;
var b = [42];

console.log(a == b);
```
[42]는 ToPrimitive 연산 시 "42"  
"42" == 42 -> 42 == 42

==알고리즘의 ToPrimitive 강제변환: 언박싱과 관련
```javascript
var a = "abc";
var b = Object(a); // new String(a)와 같음

console.log(a === b);
console.log(a == b);
```
b는 ToPrimitive 연산으로 "abc" 값(단순 스칼라 원시값)으로 강제변환(언박싱), a와 동일하므로 a == b는 true  

== 알고리즘에서 더 우선하는 규칙 존재
```javascript
var a = null;
var b = Object(a);
console.log(a == b);

var c = undefined;
var d = Object(c);
console.log(c == d);

var e = NaN;
var f = Object(e); // new Number(e)와 같음
condole.log(e == f);
```
null과 undefined는 객체 래퍼가 따로 없으므로 박싱이 불가능  
-> Object(null)은 Object()로 해석되어 일반 객체가 생성됨  
NaN은 Number로 박싱되지만 ==를 만나 언박싱되면 조건식이 NaN == NaN이 되어 결과는 false

### 4.5.3 희귀 사례
강제변환 시 발생할 수 있는 버그  

내장 네이티브 프로토타입의 변경 시도
```javascript
Number.prototype.valueOf = function() {
  return 3;
}
var yn = new Number(2) == 3;
console.log(yn);
```
2 == 3의 예시와는 무관 - 2, 3 둘 다 원시 숫자 값이기 때문에 곧바로 비교 가능 -> Number.prototype.valueOf()는 호출되지 않음  
그러나 new Number(2)는 무조건 ToPrimitive 강제변환 후 valueOf()를 호출

```javascript
var a = 1;
if (a == 2 && a == 3) {
  console.log('No');
}
```
a가 2이면서 동시에 3이라는 논리가 아님  
두 표현식 중 a == 2가 a == 3보다 먼저 평가됨

valueOf()의 부수 효과 부여
```javascript
var i = 2;
Number.prototype.valueOf = function() {
  return i++;
}

var a = new Number(42);

if (a == 2 && a == 3) {
  console.log('Yes');
}
```
위의 코드는 적절한 사용의 예시가 아님. 강제변환 반대의 근거로 사용되기에 부적절

#### Falsy 비교
긍정 오류
```javascript
console.log("0" == null);
console.log("0" == undefined);
console.log("0" == false);  // true
console.log("0" == NaN);
console.log("0" == 0);
console.log("0" == "");

console.log(false == null);
console.log(false == undefined);
console.log(false == NaN);
console.log(false == 0);  // true
console.log(false == ""); // true
console.log(false == []); // true
console.log(false == {});

console.log("" == null);
console.log("" == undefined);
console.log("" == NaN);
console.log("" == 0);   // true
console.log("" == []);  // true
console.log("" == {});

console.log(0 == null);
console.log(0 == undefined);
console.log(0 == NaN);
console.log(0 == []);   // true
console.log(0 == {});
```

```javascript
console.log([] == ![]);
```
!단항 연산자: 불리언 값으로 명시적 강제변환  
-> [] == ![] => [] == false

```javascript
console.log(2 == [2]);
console.log("" == [null]);
```
[2]와 [null]은 ToPrimitive에 의해 좌변과 비교 가능한 원시값으로 강제변환됨  
배열의 valueOf()는 배열 자신을 반환, 강제변환 시 배열을 문자열화  
-> [2] => "2", [null] => ""

```javascript
console.log(0 == "\n");
```
"", "\n"은 ToNumber를 경유하여 0으로 강제변환

24가지 사례 중 true가 나오는 7가지  
의미 없는 비교 - 사용하지 말 것
```javascript
console.log("0" == false);
console.log(false == 0);
console.log(false == "");
console.log(false == []);
console.log("" == 0);
console.log("" == []);
console.log(0 == []);
```

#### 암시적 강제변환의 안전한 사용법
동등 비교 원칙
- 피연산자 중 하나가 true/false일 가능성이 있으면 절대 == 연산자를 사용해서는 안됨
- 피연산자 중 하나가 [], "", 0이 될 가능성이 있으면 가급적 == 연산자를 사용해서는 안됨  
위의 상황이라면 == 대신 ===를 사용하여 의도하지 않은 강제변환을 차단  
결론: 강제변환의 허용 여부가 == 또는 ===의 사용을 결정함

## 4.6 추상 관계 비교
관계 비교 시 ToPrimitive 강제변환 실시로 시작  
어느 한쪽이라도 문자열이 아닐 경우 양쪽 모두 ToNumber로 강제변환하여 숫자값으로 만들어 비교
```javascript
var a = [42];
var b = ["43"];

console.log(a < b);
console.log(b < a);
```

비교 대상이 모두 문자열 값이면, 각 문자를 단순 어휘 비교(알파벳 순)
```javascript
var a = ["42"];
var b = ["043"];
console.log(a < b);
```
배열
```javascript
var a = [4, 2];
var b = [0, 4, 3];
console.log(a < b);
```
Object
```javascript
var a = { b: 42 };
var b = { b: 43 };
console.log(a < b);
```
a와 b 모두 [object Object]로 변환되어 어휘적인 비교 불가

```javascript
var a = { b: 42 };
var b = { b: 42 };

console.log(a < b);
console.log(a == b); // 별개의 객체 비교이기 때문에 false
console.log(a > b);

console.log(a <= b);
console.log(a >= b);
```
a <= b의 경우 b < a의 결과를 부정하도록 기술됨  
b < a가 false이므로 a <= b는 true  

자바스크립트의 엔진은 <=를 '더 크지 않은'의 의미로 해석  
동등 비교에는 '엄격한 관계 비교'는 없음 - 비교 전 a와 b 모두 명시적으로 동일한 타입임을 확실히 하는 것만이 관계 비교 시 암시적 강제변환을 원천봉쇄할 수 있음  
조심해서 관계비교를 해야하는 상황에서는 비교 대상 값들을 명시적으로 강제변환해두는 것이 안전
```javascript
var a = [42];
var b = "043";

console.log(a < b);
console.log(Number(a) < Number(b));
```