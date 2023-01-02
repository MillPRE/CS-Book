03-2 | 명시적으로 this를 바인딩하는 방법
---

앞 절에서 상황별로 this에 어떤 값이 바인딩되는지를 살펴봤지만 이러한 규칙을 깨고 this에 별도의 대상을 바인딩하는 방법도 있습니다. 앞 절의 규칙에 부합하지 않는다면 다음 방법 중 하나를 사용했을 것이라고 추측할 수 있습니다.

---
### 03-2-1 | call 메서드

```
Function.prototype.call(thisArg[, arg1[, arg2[, ...]]])
```
call 메서드는 메서드의 호출 주체인 함수를 즉시 실행하도록 하는 명령입니다. 이때 call 메서드의 첫 번째 인자를 this로 바인딩하고, 이후의 인자들을 호출할 함수의 매개변수로 합니다. 
함수를 그냥 실행하면 this는 전역객체를 참조하지만 call 메서드를 이용하면 임의의 객체를 this로 지정할 수 있습니다.

```javascript
// 예제 3-14 | call 메서드 (1)
var func = function (a,b,c){
    console.log(this,a,b,c);
};

func(1,2,3);            // Window { ... } 1 2 3
func.call({ x : 1 }, 4, 5, 6); // { x : 1 } 4 5 6
```

메서드에 대해서도 마찬가지로 객체의 메서드를 그냥 호출하면 this는 객체를 참조하지만 call 메서드를 이용하면 임의의 객체를 this로 지정할 수 있습니다.

```javascript
// 예제 3-15 | call 메서드(2)
var obj = {
    a : 1,
    method : function(x,y){
        console.log(this.a, x,y);
    }
};

obj.method(2,3);                // 1 2 3
obj.method.call( { a : 4 }, 5, 6 ); // 4 5 6
```

---
### 03-2-2 | apply 메서드

```
Function.prototype.apply(thisArg[, argsArray])
```

apply 메서드는 call 메서드와 기능적으로는 완전히 동일합니다. 
call 메서드는 첫 번째 인자를 제외한 나머지 모든 인자들을 호출할 함수의 매개변수로 지정하는 반면, apply 메서드는 두 번째 인자를 배열로 받아 그 배열의 요소들을 호출할 함수의 매개변수로 지정한다는 점에서만 차이가 있습니다.

```javascript
// 예제 3-16 | apply 메서드
var func = function(a,b,c){
    console.log(this, a, b, c);
};

func.apply({ x : 1 }, [4,5,6]);     // { x : 1 } 4 5 6

var obj = {
    a : 1,
    method : function(x,y){
        console.log(this.a, x, y);
    }
};
obj.method.apply( { a : 4 }, [5,6]);    // 4 5 6
```

---
### 03-2-3 | call / apply 메서드의 활용

call 이나 apply 메서드를 잘 활용하면 자바스크립트를 더욱 다채롭게 사용할 수 있습니다. 몇 가지 활용 사례를 소개합니다.

```javascript
// 예제 3-17 | call/apply 메서드의 활용 1-1) 유사배열객체에 배열 메서드를 적용
var obj = {
    0 : 'a',
    1 : 'b',
    2 : 'c',
    length : 3
};

Array.prototype.push(call(obj,'d'));
console.log(obj);                       // { 0 : 'a', 1 : 'b', 2 : 'c', 3 : 'd', length :4 }

var arr = Array.prototype.slice.call(obj);
console.log(arr);                       // [ 'a','b','c','d' ]
```

객체에는 배열 메서드를 직접 적용할 수 없습니다. 그러나 **키가 0또는 양의 정수인 프로퍼티가 존재**하고 length **프로퍼티의 값이 0 또는 양의 정수**인 객체, 즉 배열의 구조와 유사한 객체의 경우(유사배열객체) call 또는 apply 메서드를 이용해 배열 메서드를 차용할 수 있습니다.
7번째 줄에서는 배열 메서드인 push를 객체 obj에 적용해 프로퍼티 3에 'd'를 추가했습니다. 9번째 줄에서는 slice메서드를 적용해 객체를 배열로 전환했습니다.
slice 메서드는 원래 시작 인덱스값과 마지막 인덱스값을 받아 시작값부터 마지막값의 앞부분까지의 배열 요소를 추출하는 메서드인데, 매개변수를 아무것도 넘기지 않을 경우에는 그냥 원본 배열의 얕은 본사본을 반환합니다. 그러니까 call 메서드를 이용해 원본인 유사배열객체의 얕은 복사를 수행한 것인데, slice메서드가 배열 메서드이기 때문에 복사본은 배열로 반환하게 된 것이죠.

함수 내부에서 접근할 수 있는 arguments 객체도 유사배열객체이므로 위의 방법으로 배열로 전환해서 활용할 수 있습니다. 
querySelectorAll, getElementsByClassName 등의 Node선택자로 선택한 결과인 NodeList도 마찬가지입니다.

```javascript
// 예제 3-18 | call/apply 메서드의 활용 1-2) arguments, NodeList에 배열 메서드를 적용
function a () {
    var argv = Array.prototype.slice.call(arguments);
    argv.forEach(function(arg){
        console.log(arg);
    });
}

a(1,2,3);

document.body.innerHTML = '<div>a</div><div>b</div><div>c</div>';
var nodeList = document.querySelectorAll('div');
var nodeArr = Array.prototype.slice.call(nodeList);
nodeArr.forEach(funciton(node){
    console.log(node);
});
```
그 밖에도 유사배열객체에는 call/apply 메서드를 이용해 모든 배열 메서드를 적용할 수 있습니다.
배열처럼 인덱스와 length 프로퍼티를 지니는 문자열에 대해서도 마찬가지입니다. 단, 문자열의 경우 length 프로퍼티가 읽기 전용이기 때문에 원본 문자열에 변경을 가하는 메서드(push, pop, shift, unshift, splice 등)는 에러를 던지며, concat 처럼 대상이 반드시 배열이어야 하는 경우에는 에러는 나지 않지만 제대로 된 결과를 얻을 수 없습니다.

```javascript
// 예제 3-19 | call/apply 메서드의 활용 1-3) 문자열에 배열 메서드 적용 예시
var str = 'abc def';

Array.prototype.push.call(str, ', pushed string');
// Error : Cannot assign to read only property 'length' of object [object String]

Array.prototype.concat.call(str, 'string'); // [String {"abc def"}, "string ]

Array.prototype.every.call(str, function(char){ return char !== ' ';}); // false

Array.prototype.some.call(str, function(char){ return char === ' ';}); // true

var newArr = Array.prototype.map.call(str, function(char){ return char +'!';});

console.log(newArr);        // [ 'a!', 'b!', 'c!', ' !', 'd!', 'e!', 'f!' ]

var newStr = Array.prototype.reduce.apply(str, [
    function(string, char, i) { return string + char + i; },
    ''
]);
console.log(newStr);        // "a0b1c2 3d4e5f6"
```

사실 call/apply를 이용해 형변환하는 것은 "this를 원하는 값으로 지정해서 호출한다"라는 본래의 메서드의 의도와는 다소 동떨어진 활용법이라 할 수 있습니다.
slice 메서드는 오직 배열 형태로 '복사'하기 위해 차용됐을 뿐이니, 경험을 통해 숨은 뜻을 알고 있는 사람이 아닌 한 코드만 봐서는 어떤 의도인지 파악하기 쉽지 않습니다. 
이에 ES6에서는 유사배열객체 또는 순회 가능한 모든 종류의 데이터 타입을 배열로 전환하는 Array.from 메서드를 새로 도입했습니다.

```javascript
// 예제 3-20 call/apply 메서드의 활용 1-4) ES5의 Array.from 메서드
var obj = {
    0: 'a',
    1: 'b',
    2: 'c',
    length: 3,
};

var arr = Array.from(obj);
console.log(arr); // ['a', 'b', 'c']
```

### 생성자 내부에서 다른 생성자를 호출
생성자 내부에 다른 생성자와 공통된 내용이 있을 경우 call 또는 apply를 이용해 다른 생성자를 호출하면 간단하게 반복을 줄일 수 있습니다.
다음 예제에서는 Student, Employee 생성자 함수 내부에서 Person 생성자 함수를 호출해서 인스턴스의 속성을 정의하도록 구현했습니다.

```javascript
// 에제 3-21 call/apply 메서드의 활용 2) 생성자 내부에서 다른 생성자를 호출
function Person(name, gender) {
    this.name = name;
    this.gender = gender;
}

function Studnet(name, gender, school) {
    Person.call(this, name, gender);
    this.school = school;
}

function Employee(name, gender, company) {
    Person.apply(this, [name, gender]);
    this.company = company;
}

var by = new Studnet('보영', 'female', '단국대');
var jn = new Employe('재난', 'male','구골');
```

### 여러 인수를 묶어 하나의 배열로 전달하고 싶을 때 - apply 활용
여러 개의 인수를 받는 메서드에게 하나의 배열로 인수들을 전달하고 싶을 때 apply 메서드를 사용하면 좋습ㄴ디ㅏ.
예를 들어, 배열에서 최대/최솟값을 구해야 할 경우 apply를 사용하지 않는다면 부득이하게 다음과 같은 방식으로 직접 구현할 수밖에 없을 것입니다.

```javascript
// 예제 3-22 call/apply 메서드의 활용 3-1) 최대/최솟값을 구하는 코드를 직접 구현
var numbers = [10,20,3,16,45];
var max = min = numbers[0];
numbers.forEach(function(number) {
    if (number > max) {
        max = number;
    }
    if (number < min) {
        min = number;
    }
});
console.log(max, min); // 45 3
```

코드가 불필요하게 길고 가독성도 떨어집니다. 이보다는 Math.max/Math.min 메서드에 apply를 적용하면 훨씬 간단해집니다.

```javascript
// 예제 3-23 call/apply 메서드의 활용 3-2) 여러 인수를 받는 메서드(Math.max/Math.min)에 apply를 활용
var numbers = [10,20,3,16,45];
var max = Math.max.apply(null, numbers);
var min = Math.min.apply(null, numbers);
console.log(max, min); // 45 3
```

참고로 ES6에서는 펼치기 연산자(spread operator)를 이용하면 apply를 적용하는 것보다 더욱 간편하게 작성할 수 있습니다.
```javascript
// 예제 3-24 call/apply 메서드의 활용 3-3) ES6의 펼치기 연산자 활용
var numbers = [10,20,3,16,45];
var max = Math.max(...numbers);
var min = Math.min(...numbers);
console.log(max, min); // 45 3
```
call/apply 메서드는 명시적으로 별도의 this를 바인딩하면서 함수 또는 메서드를 실행하는 훌륭한 방법이지만 오히려 이로 인해 this를 예측하기 어렵게 만들어 코드 해석을 방해한다는 단점이 있습니다.
그럼에도 불구하고 ES5 이하의 환경에서는 마땅한 대안이 없기 때문에 실무에서 매우 광범위하게 활용되고 있습니다.

### 03-2-4 | bind 메서드
```javascript
Function.prototype.bind(thisArg[, arg1[, arg2[, ...]]])
```
bind 메서드는 ES5에서 추가된 기능으로, call과 비슷하지만 즉시 호출하지는 않고 넘겨 받은 this 및 인수들을 바탕으로 새로운 함수를 반환하기만 하는 메서드이다.
다시 새로운 함수를 호출할 때 인수를 넘기면 그 인수들은 기존 bind 메서드를 호출할 때 전달했던 인수들의 뒤에 이어서 등록됩니다. 
즉 bind 메서드는 함수에 this를 미리 적용하는 것과 부분 적용 함수를 구현하는 두 가지 목적을 모두 지닙니다. 예제를 통해 확인해봅시다.

```javascript
// 예제 3-25 bind 메서드 - this 지정과 부분 적용 함수 구현
var func = function(a, b, c, d) {
    console.log(this, a, b, c, d);
}

func(1,2,3,4);      // Window{ ... } 1 2 3 4

var bindFunc1 = func.bind({x: 1});
bindFunc1(5,6,7,8);     // {x:1} 5 6 7 8

var bindFunc2 = func.bind({x:1}, 4, 5);
bindFunc2(6,7);     // {x:1} 4 5 6 7
bindFunc2(8,9);     // {x:1} 4 5 8 9
```

6번째 줄에서 bindFunc1 변수에는 func에 this를 {x:1} 로 지정한 새로운 함수가 담깁니다.
이제 7번째 줄에서 bindFunc1을 호출하면 원하는 결과를 얻을 수 있게 됩니다. 한편 9번째 줄의 bindFunc2 변수에는 func에 this를 {x:1}로 지정하고, 앞에서부터 두 개의 인수를 각각 4,5 로 지정한 새로운 함수를 담았습니다.
이후 10번째 줄에서 매개변수로 6,7을 넘기면 this 값이 바뀐 것을 제외하고는 최초 func 함수에 4,5,6,7 을 넘긴 것과 같은 동작을 합니다. 11번째 줄에서도 마찬가지입니다. 6번째 줄의 bind는 this만을 지정한 것이고, 9번째 줄의 bind는 this지정과 함께 부분 적용 함수를 구현한 것입니다.


### name 프로퍼티
bind 메서드를 적용해서 새로 만든 함수는 한 가지 독특한 성질이 있습니다. 바로 name 프로퍼티에 동사 bind의 수동태인 'bound'라는 접두어가 붙는다는 점입니다.
어떤 함수의 name 프로퍼티가 'bound xxx'라면 이는 곧 함수명이 xxx인 원본 함수에 bind 메서드를 적용한 새로운 함수라는 의미가 되므로 기존의 call이나 apply보다 코드를 추적하기에 더 수월해진 면이 있습니다.

```javascript
// 예제 3-26 bind 메서드 - name 프로퍼티
var func = function(a,b,c,d) {
    console.log(this, a,b,c,d);
}

var bindFunc = func.bind({x:1}, 4, 5);
console.log(func.name); // func
console.lof(bindFunc.name); // bound func
```

### 상위 컨텍스트의 this를 내부함수나 콜백 함수에 전달하기
3-1-3 절에서 메서드의 내부함수에서 메서드의 this를 그대로 바라보게 하기 위한 방법으로 self 등의 변수를 활용한 우회법을 소개햇는데, call, apply 또는 bind 메서드를 이용하면 더 깔끔하게 처리할 수 있습니다.

```javascript
// 예제 3-27 | 내부함수에 this 전달 - call vs bind
var obj = {
    outer: function() {
        console.log(this);
        var innerFunc = function() {
            console.log(this);
        };
        innerFunc.call(this);
    }
};
obj.outer();

-----------------------------------------

var obj = {
    outer: function() {
        console.log(this);
        var innerFunc = function() {
            console.log(this);
        }.bind(this);
        innerFunc();
    }
};
obj.outer();
```

또한 콜백 함수를 인자로 받는 함수나 메서드 중에서 기본적으로 콜백 함수 내에서의 this에 관여하는 함수 또는 메서드에 대해서도 bind메서드를 이용하면 this 값을 사용자의 입맛에 맞게 바꿀 수 있습니다.

```javascript
// 예제 3-28 | bind 메서드 - 내부함수에 this 전달
var obj = {
    logThis: function() {
        console.log(this);
    },
    logThisLater1: function() {
        setTimeout(this.logThis, 500);
    },
    logThisLater2: function() {
        setTImeout(this.logThis.bind(this), 1000);
    }
};

obj.logThisLater1(); // Window { ... }
obj.logThisLater2(); // obj { logThis: f, ... }
```

### 03-2-5 | 화갈표 함수의 예외사항
ES6에 새롭게 도입된 화살표 함수는 실행 컨텍스트 생성 시 this를 바인딩하는 과정이 제외됐습니다.
즉 이 함수 내부에는 this가 아예 없으며, 접근하고자 하면 스코프체인상 가장 가까운 this에 접근하게 됩니다.

```javascript
// 예제 3-29 | 화살표 함수 내부에서의 this
var obj = {
    outer: function() {
        console.log(this);
        var innerFunc = () => {
            console.log(this);
        };
        innerFunc();
    }
}
obj.outer();
```

예제 3-29는 예제 3-27의 내부함수를 화살표 함수로 바꾼 것입니다. 이렇게 하면 별도의 변수로 this를 우회하거나 call/apply/bind를 적용할 필요가 없어 더욱 간결하고 편리합니다.

### 03-2-6 | 별도의 인자로 this를 받는 경우(콜백 함수 내에서의 this)
콜백 함수는 다음 장에서 자세히 다루겠습니다.
여기서는 this와 관련 있는 부분만 간략히 살펴봅시다. 콜백 함수를 인자로 받는 메서드 중 일부는 추가로 this로 지정할 객체(thisArg)를 인자로 지정할 수 있는 경우가 있습니다. 이러한 메서드의 thisArg 값을 지정하면 콜백 함수 내부에서 this 값을 원하는 대로 변경할 수 있습니다.
이런 형태는 여러 내부 요소에 대해 같은 동작을 반복 수행해야 하는 **배열 메서드**에 많이 포진돼 있으며, 같은 이유로 ES6에서 새로 등장한 Set, Map 등의 메서드에도 일부 존재합니다. 그중 대표적인 배열 메서드인 forEach의 예를 살펴보겠습니다.

```javascript
// 예제 3-30 | thisArg를 받는 경우 예시 - forEach 메서드
var report = {
    sum: 0,
    count: 0,
    add: function () {
        var args = Array.prototype.slice.call(arguments);
        arg.forEach(function(entry) {
            this.sum += entry;
            ++this.count;
        }, this);
    },
    average: function() {
        return this.sum / this.count;
    }
};

report.add(60, 85, 95);
console.log(report.sum, report.count, report.average()); // 240 3 80
```

report 객체에는 sum, count 프로퍼티가 있고, add, average 메서드가 있습니다. 5번째 줄에서 add 메서드는 arguments를 배열로 변환해서 args 변수에 담고, 6번째 줄에서는 이 배열을 순회하면서 콜백 함수를 실행하는데, 이때 콜백 함수 내부에서의 this는 forEach 함수의 두 번째 인자로 전달해준 this(9번째 줄)가 바인딩됩니다. 11번째 줄의 average는 sum 프로퍼티를 count프로퍼티로 나눈 결과를 반환하는 메서드입니다.

15번재 줄에서 60, 85, 95를 인자로 삼아 add 메서드를 호출하면 이 세 인자를 배열로 만들어 forEach 메서드가 실행됩니다. 콜백 함수 내부에서의 this는 add 메서드에서의 this가 전달된 상태이므로 add 메서드의 this(report)를 그대로 가리키고 있습니다.
따라서 배열의 세 요소를 순회하면서 report.sum 값 밒 report.count 값이 차례로 바뀌고, 순회를 마친 결과 report.sum에는 340이, report.count에는 3이 담기게 됩니다.

배열의 forEach를 예로 들었지만, 이 박에도 thisArg를 인자로 받는 메서드는 꽤 많이 있습니다. 이들에 대한 문법을 나열하면 다음과 같습니다.

```javascript
// 예제 3-31 | 콜백 함수와 함께 thisArg를 인자로 받는 메서드
Array.prototype.forEach(callback[, thisArg])
Array.prototype.map(callback[, thisArg])
Array.prototype.filter(callback[, thisArg])
Array.prototype.some(callback[, thisArg])
Array.prototype.every(callback[, thisArg])
Array.prototype.find(callback[, thisArg])
Array.prototype.findIndex(callback[, thisArg])
Array.prototype.flatMap(callback[, thisArg])
Array.prototype.from(callback[, thisArg])
Set.prototype.forEach(callback[, thisArg])
Map.prototype.forEach(callback[, thisArg])
```

03 정리
---
다음 규칙은 명시적 this 바인딩이 없는 한 늘 성립합니다.
원리를 바탕으로 꾸준히 다양한 상황에서 this가 무엇일지 예측해보는 연습을 해보시기 바랍니다.

- 전연공간에서의 this는 전역객체(브라우저에서는 window, Node.js에서는 global)를 참조합니다.
- 어떤 함수를 메서드로서 호출한 경우 this는 메서드 호출 주체(메서드명 앞의 객체)를 참조합니다.
- 어떤 함수를 함수로서 호출한 경우 this는 전역객체를 참조합니다. 메서드의 내부함수에서도 같습니다.
- 콜백 함수 내부에서의 this는 해당 콜백 함수의 제어권을 넘겨받은 함수가 정의한 바에 따르며, 정의하지 않은 경우에는 전역객체를 참조합니다.
- 생성자 함수에서의 this는 생성될 인스턴스를 참조합니다.


다음은 명시적 this 바인딩입니다. 위 규칙에 부합하지 않는 경우에는 다음 내용을 바탕으로 this를 예측할 수 있습니다.

- call, apply 메서드는 this를 명시적으로 지정하면서 함수 또는 메서드를 호출합니다.
- bind 메서드는 this 및 함수에 넘길 인수를 일부 지정해서 새로운 함수를 만듭니다.
- 요소를 순회하면서 콜백 함수를 반복 호출하는 내용의 일부 메서드는 별도의 인자로 this룰 받기도 합니다.
