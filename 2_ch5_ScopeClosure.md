# 5 스코프 클로저

## 5.1 깨달음
클로저는 자바스크립트의 모든 곳에 존재  
렉시컬 스코프에 의존해 코드를 작성한 결과로 그냥 발생  
의도적으로 생성할 필요 없이, 모든 코드에서 클로저는 생성되고 사용됨

## 5.2 핵심
클로저: 함수가 속한 렉시컬 스코프를 기억하여 함수가 렉시컬 스코프 밖에서 실행될 때에도 해당 스코프에 접근할 수 있게 해주는 기능
```javascript
function foo() {
  var a = 2;
  function bar() {
    console.log(a);
  }
  bar();
}
foo();
```
함수 bar()는 렉시컬 스코프 검색 규칙을 통해 바깥 스코프의 변수 a에 접근 가능(RHS 참조 검색)  
-> 렉시컬 스코프 검색 규칙에 따른 설명. 클로저의 일부  
함수 bar()는 foo() 스코프에 대한 클로저를 지님 -> bar()는 foo() 스코프에서 닫힘  
=> bar()는 중첩되어 foo() 안에 존재하기 때문  

```javascript
function foo() {
  var a = 2;
  function bar() {
    console.log(a);
  }
  return bar;
}

var baz = foo();
baz();
```
함수 bar()는 foo()의 렉시컬 스코프에 접근할 수 있고, bar() 함수 자체를 값으로 넘김  
foo()를 실행하여 반환한 값(bar())를 baz 변수에 대입한 후 호출 -> bar() 호출  
bar는 함수가 선언된 렉시컬 스코프 밖에서 실행된 결과  

함수는 실행 후 더 이상 사용되지 않는 상황이라면 가비지 콜렉터에 의해 메모리가 해제됨  
그러나 bar()가 여전히 foo의 내부 스코프를 사용중이므로 해제되지 않음  
bar()는 foo() 스코프에 대한 렉시컬 스코프 클로저를 갖고, foo()는 bar가 나중에 참조할 수 있도록 스코프를 살려둠  
즉, bar()는 여전히 해당 스코프에 대한 참조를 지님 -> 이 참조를 '클로저'라고 함  
baz 호출 시, 원래 코드의 렉시컬 스코프에 접근할 수 있고, 이는 함수가 변수 a에 접근할 수 있다는 의미  

클로저는 호출된 함수가 원래 선언된 렉시컬 스코프에 계속해서 접근할 수 있도록 허용함
```javascript
function foo() {
  var a = 2;
  function baz() {
    console.log(a);
  }
  bar(baz);
}

function bar(fn) {
  fn();
}
```
함수 baz를 bar에 넘기고, fn이라고 명명된 함수를 호출  
foo()의 내부 스코프에 대한 fn의 클로저는 변수 a에 접근할 때 확인 가능  

```javascript
var fn;

function foo() {
  var a = 2;
  function baz() {
    console.log(a);
  }
  fn = baz;
}

function bar() {
  fn();
}

foo();

bar();
```
어떤 방식으로 내부 함수를 자신이 속한 렉시컬 스코프 밖으로 수송하든 함수는 처음 선언된 곳의 스코프에 대한 참조를 유지함. 즉, 어디에서 함수를 실행하든 클로저가 작용

## 5.3 이제 나는 볼 수 있다
```javascript
function wait(message) {
  setTimeout(function timer() {
    console.log(message);
  }, 1000);
}

wait("Hello, closuer!");
```
내부 함수 timer가 setTimeout()의 인자로 투입  
timer 함수는 wait() 함수의 스코프에 대한 스코프 클로저를 가지고 있으므로 변수 messagege에 대한 참조를 유지하고 사용 가능  
wait() 실행 1초 후, wait 내부 스코프는 사라져야 하지만 익명 함수가 여전히 해당 스코프에 대한 클로저 유지  

### 클로저
제이쿼리 예시
```javascript
function setupBot(name, selector) {
  $(selector).click(function activator() {
    console.log("Activating: " + name);
  });
}
setupBot("Closure Bot 1", "#bot_1");
setupBot("Closure Bot 2", "#bot_2");
```
자체의 렉시컬 스코프에 접근할 수 있는 함수를 인자로 넘길 때 그 함수가 클로저를 사용함  
타이머, 이벤트 처리기, Ajax 요청, 윈도 간 통신, 웹 워커와 같은 비동기(또는 동기)적 작업 시 콜백 함수를 넘기면 클로저를 사용할 준비가 된 것  

IIFE와 클로저
```javascript
var a = 2;
(function IIFE() {
  console.log(a);
})();
```
작동은 하나 엄격히 말해 클로저가 사용된 것은 아님  
IIFE 함수가 자신의 렉시컬 스코프 밖에서 실행된 것은 아니기 때문 - 선언된 바로 그 스코프 안에서 호출됨  
변수 a는 클로저가 아니라 일반적인 렉시컬 스코프 검색을 통해서 가져옴  

클로저는 선언 시 발생하지만, 바로 관찰할 수 있는 것은 아님  
IIFE 자체는 클로저 사례가 아니지만 클로저를 사용할 수 있는 스코프를 만드는 도구 중 하나  

## 5.4 반복문과 클로저
클로저를 설명하는 가장 흔하고 표준적인 사례
```javascript
for (var i = 1; i <= 5; i++) {
  setTimeout(function timer() {
    console.log(i);
  }, i * 1000);
}
```
반복문이 끝나는 조건은 i가 6이 되었을 경우  
출력된 값은 반복문이 끝났을 때의 i값을 반영한 것  
timeout 함수 콜백은 반복문이 끝나고 나서 작동  
기대대로 실행되기 위해서는 반복마다 각각의 i 복제본을 활용해야 함  
그러나 반복문 안 총 5개의 함수들은 반복마다 따로 정의됐음에도 모두 같이 글로벌 스코프 클로저를 공유 -> 하나의 i만 존재, 같은 i에 대한 참조를 공유  

반복마다 새로운 닫힌 스코프가 필요  
IIFE는 함수를 정의하고 바로 실행시키면서 스코프를 생성
```javascript
for (var i = 1; i <= 5; i++) {
  (function() {
    setTimeout(function timer() {
      console.log(i);
    }, i * 1000);
  })();
}
```
스코프가 비어있기 때문에 닫힌 스코프만으로는 부족  
각 스코프는 자체 변수가 필요 - 반복마다 i의 값을 저장할 변수가 필요
```javascript
for (var i = 1; i <= 5; i++) {
  (function() {
    var j = i;
    setTimeout(function timer() {
      console.log(j);
    }, j * 1000);
  })();
}
```
다른 버전
```javascript
for (var i = 1; i <= 5; i++) {
  (function(j) {
    setTimeout(function timer() {
      console.log(j);
    }, j * 1000);
  })(i);
}
```
IIFE를 사용하여 반복마다 새로운 스코프를 생성하는 방식 - timeout 함수 콜백은 원하는 값이 제대로 저장된 변수를 가진 새로운 '닫힌 스코프'를 반복마다 생성하여 사용  

### 5.4.1 다시 보는 블록 스코프
반복마다 IIFE를 사용하여 새로운 스코프를 생성하는 방식  
-> 실제로 필요한 것은 반복 별 블록 스코프  
let 키워드 - 본질적으로 하나의 블록을 닫을 수 있는 스코프로 바꿈
```javascript
for (var i = 1; i <= 5; i++) {
  let j = i;
  setTimeout(function timer() {
    console.log(j);
  }, j * 1000);
}
```
for 반복문 내 let 선언문의 작동 방식  
반복문 시작 부분에서 let으로 선언된 변수는 한 번만 선언되는 것이 아니라 반복마다 선언됨  
해당 변수는 반복마다 이전 반복이 끝난 이후의 값으로 초기화됨
```javascript
for (let i = 1; i <= 5; i++) {
  setTimeout(function timer() {
    console.log(i);
  }, i * 1000);
}
```

## 5.5 모듈
모듈 - 콜백과 상관 없지만 클로저를 활용하는 패턴 유형
```javascript
function foo() {
  var something = "cool";
  var another = [1, 2, 3];

  function doSomething() {
    console.log(something);
  }

  function doAnother() {
    console.log(another.join("!"));
  }
}
```
변수 something과 another, 함수 doSomething과 doAnother는 foo() 내부 스코프를 렉시컬 스코프로 지님 - 클로저도 따라옴

```javascript
function CoolModule() {
  var something = "cool";
  var another = [1, 2, 3];

  function doSomething() {
    console.log(something);
  }

  function doAnother() {
    console.log(another.join("!"));
  }

  return {
    doSomething: doSomething,
    doAnother: doAnother
  };
}

var foo = CoolModule();

foo.doSomething();
foo.doAnother();
```
첫째, CoolModule()은 하나의 함수일 뿐이지만, 모듈 인스턴스 생성을 위해 반드시 호출해야 함  
최외곽 함수가 실행되지 않으면 내부 스코프와 클로저는 생성되지 않음  

둘째, CoolModule() 함수는 객체를 반환. 반환된 객체는 객체-릴터럴 문법에 따라 표기됨  
해당 객체는 내장 함수들에 대한 참조를 갖지만, 내장 데이터 변수에 대한 참조는 갖지 않음 - 내장 데이터 변수는 비공개로 숨겨져 있음  
객체의 반환 값이 모듈의 공개 API  

객체의 반환 값은 최종적으로 외부 변수 foo에 대입되고, foo.doSomething()과 같은 방식으로 API 속성 메서드에 접근 가능  

함수 doSomething()과 doAnother()는 모듈 인스턴스의 내부 스코프에 포함하는 클로저를 보유  
반환된 객체에 대한 속성 참조 방식으로 해당 렉시컬 스코프 밖으로 옮길 때 클로저를 확인하고 이용 가능  

정리 - CoolModule() 모듈 패턴 사용 조건  
1. 하나의 최외곽 함수가 존재하고, 이 함수가 최소 한 번은 호출되어야 함
2. 최외곽 함수는 최소 한 번은 하나의 내부 함수를 반환해야 함 -> 해당 내부 함수가 비공개 스코프에 대한 클로저를 가져 비공개 상태에 접근하고 수정 가능  

하나의 함수 속성만 갖는 객체는 모듈이라고 할 수 없으며, 함수 실행 결과로 반환된 객체에 데이터 속성들은 있지만 닫힌 함수가 없다면, 그 객체 또한 모듈이 아님  

하나의 인스턴스(singleton)만 생성하는 모듈 - 모듈 함수를 IIFE로 바꾸고 즉시 실행, 반환 값을 확인자 foo에 직접 대입
```javascript
var foo = (function CoolModule() {
  var something = "cool";
  var another = [1, 2, 3];

  function doSomething() {
    console.log(something);
  }
  function doAnother() {
    console.log(another.join("!"));
  }

  return {
    doSomething: doSomething,
    doAnother: doAnother
  };
})();

foo.doSomething();
foo.doAnother();
```

인자를 받는 함수 형태로 모듈 작성
```javascript
function CoolModule(id) {
  function identify() {
    console.log(id);
  }
  return {
    identify: identify
  };
}

var foo1 = CoolModule("foo 1");
var foo2 = CoolModule("foo 2");

foo1.identyfy();
foo2.identyfy();
```

공개 API로 반환하는 객체에 이름을 정하는 방식
```javascript
var foo = (function CoolModule(id) {
  function change() {
    publicAPI.identify = identify2;
  }

  function identify1() {
    console.log(id);
  }
  
  function identify2() {
    console.log(id.toUpperCase());
  }

  var publicAPI = {
    change: change,
    identify: identify1
  };

  return publicAPI;
})("foo module");

foo.identify();
foo.change();
foo.identify();
```
공개 API 객체에 대한 내부 참조를 모듈 인스턴스 내부에 유지 -> 모듈 인스턴스를 내부에서부터 메서드와 속성을 추가 또는 삭제하거나 값을 변경하는 식으로 수정 가능  

### 5.5.1 현재의 모듈
많은 모듈 의존성 로더 및 관리자는 이러한 패턴의 모듈 정의를 친숙한 API 현태로 감싸고 있음
```javascript
var MyModules = (function Manager() {
  var modules = {};

  function define(name, deps, impl) {
    for (var i = 0; i < deps.length; i++) {
      deps[i] = modules[deps[i]];
    }
    modules[name] = impl.apply(impl, deps);
  }
  function get(name) {
    return modules[name];
  }
  return {
    define: define,
    get: get
  };
})();
```
코드의 핵심부 - modules[name] = impl.apply(imple, deps);  
의존성을 인자로 넘겨 모듈에 대한 정의 래퍼 함수를 호출하여 반환 값인 모듈 API를 이름으로 정리된 내부 모듈 리스트에 저장함  

```javascript
var MyModules = (function Manager() {
  var modules = {};

  function define(name, deps, impl) {
    for (var i = 0; i < deps.length; i++) {
      deps[i] = modules[deps[i]];
    }
    modules[name] = impl.apply(impl, deps);
  }
  function get(name) {
    return modules[name];
  }
  return {
    define: define,
    get: get
  };
})();

MyModules.define("bar", [], function() {
  function hello(who) {
    return "Let me introduce: " + who;
  }
  return {
    hello: hello
  };
});

MyModules.define("foo", ["bar"], function(bar) {
  var hungry = "hippo";
  function awesome() {
    console.log(bar.hello(hungry).toUpperCase());
  }
  return {
    awesome: awesome
  };
});
var bar = MyModules.get("bar");
var foo = MyModules.get("foo");

console.log(
  bar.hello("hippo")
);
foo.awesome();
```
foo와 bar 모듈 모두 공개 API를 반환하는 함수로 정의  
foo는 bar의 인스턴스를 의존성 인자로 받아서 사용할 수 있음  
함수 정의 래퍼를 호출하여 해당 모듈의 API인 반환 값을 저장함

### 5.5.2 미래의 모듈
ES6 - 모듈 개념을 지원하는 최신 문법 추가  
모듈 시스템을 불러올 때 파일을 개별 모듈로 처리  
각 모듈은 다른 모듈 또는 특정 API 멤버를 불러오거나 자신의 공개 API 멤버를 내보낼 수 있음

함수 기반 모듈 - 정적 패턴이 아님 -> 런타임 전까지 API의 의미는 해석되지 않음. 즉, 실제 모듈의 API를 런타임에 수정할 수 있음  
ES6 모듈 API - 정적 모듈(API가 런타임에 변하지 않음) -> 컴파일러는 컴파일레이션 중에 불러온 모듈의 API 멤버 참조가 실제로 존재하는지 확인 가능. API 참조가 존재하지 않으면 컴파일러는 컴파일 시 초기 오류를 발생. 변수 참조를 위해 동적 런타임까지 기다리지 않음  

ES6 모듈은 inline 형식을 지원하지 않고, 개별 파일에 정의되어야 함  
브라우저와 엔진은 기본 모듈 로더 보유  
모듈 로더는 모듈을 불러울 때 동기적으로 모듈 파일을 불러옴
```javascript
// bar.js
function hello(who) {
  return "Let me introduce: " + who;
}
export hello;

// foo.js
import hello from "bar";
var hungry = "hippo";
function awesome() {
  console.log(hello(hungry).toUpperCase());
}
export awesome;

// baz.js
module foo from "foo";
module bar from "bar";
console.log(bar.hello("rhino"));
foo.awesome();
```
import - 모듈 API에서 하나 이상의 멤버를 불러와 특정 변수에 묶어 현재 스코프에 저장  
module - 모듈 API 전체를 불러와 특정 변수에 묶음  
export = 확인자를 현재 모듈의 공개 API로 내보냄