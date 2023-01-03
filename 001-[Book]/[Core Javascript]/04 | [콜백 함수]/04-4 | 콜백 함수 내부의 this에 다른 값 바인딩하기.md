04-4 | 콜백 함수 내부의 this에 다른 값 바인딩하기
---

객체의 메서드를 콜백 함수로 전달하면 해당 객체를 this로 바라볼 수 없게 된다. 그럼에도 콜백 함수 내부에서 this가 객체를 바라보게 하고 싶다면 어떻게 해 할까? 별도의 인자로 this를 받는 함수의 경우에는 여기에 원하는 값을 넘겨주면 되지만 그렇지 않은 경우에는 this의 제어권도 넘겨주게 되므로 사용자가 임의로 값을 바꿀 수 없다. 
그래서 전통적으로는 this를 다른 변수에 담아 콜백 함수로 활용할 함수에서는 this 대신 그 변수를 사용하게 되고, 이를 클로저로 만드는 방식이 많이 쓰였습니다.

```javascript
// 예제 4-8 | 콜백 함수 내부의 this에 다른 값을 바인딩하는 방법(1) - 전통적인 방식
var obj1 = {
    name: 'obj1',
    func: funcion() {
        var self = this;
        return function() {
            console.log(self.name);
        };
    }
};
var callback = obj1.func();
setTimeout(callback, 1000);
```
obj1.func 메서드 내부에서 self 변수에 this를 담고, 익명 함수를 선언과 동시에 반환했습니다.
이제 10번째 줄에서 obj1.func()를 호출하면 앞서 선언한 내부 함수가 반환되어 callback 변수에 담깁니다.
11번째 줄에서 이 callback을 setTimeout 함수에 인자로 전달하면 1초(1000ms) 뒤 callback이 실행되면서 'obj1'을 출력할 것입니다.

이 방식은 실제로 this를 사용하지 않을뿐더러 번거롭기 그지 없습니다. 차라리 this를 아예 안 쓰는 편이 더 낫겠다는 생각이 듭니다.

```javascript
// 예제 4-9 | 콜백 함수 내부에서 this를 사용하지 않은 경우
var obj1 = {
    name: 'obj1',
    func: function() {
        console.log(obj1.name);
    }
};
setTimeout(obj1.func, 1000);
```
예제 4-9는 예제 4-8에서 this를 사용하지 않았을 때의 결과입니다. 훨씬 간결하고 직관적이지만 아쉬운 부분도 있습니다. 이제는 작성한 함수를 this를 이용해 다양한 상황에 재활용할 수 없게 되어버렸습니다.
예제 4-8에서 만들었던 함수를 다른 객체에 재활용하는 상황을 살펴보죠.

```javascript
// 예제 4-10 | 예제 4-8의 func 홤수 재활용
var obj1 = {
    name: 'obj1',
    func: funcion() {
    var self = this;
    return function() {
        console.log(self.name);
    };
}
};
var callback = obj1.func();
setTimeout(callback, 1000);

var obj2 = {
    name: 'obj2',
    func: obj1.func,
};
var callback2 = obj2.func();
setTimeout(callback2, 1500);

var obj3 = {
    name: 'obj3',
};
var callback = obj1.func(obj3);
setTimeout(callback3, 2000);
```

예제 4-10의 callback2에는 (obj1의 func를 복사한) obj2의 func를 실행한 결과를 담아 이를 콜백으로 사용했습니다.
callback3의 경우 obj1의 func를 실행하면서 this를 obj3가 되도록 지정해 이를 콜백으로 사용하였습니다. 
예제를 실행해보면 실행 시점으로부터 1.5초 후에는 'obj2'가, 실행 시점으로부터 2초 후에는 'obj3'가 출력됩니다.

이처러 4-8의 방법은 번거롭긴 하지만 this를 우회적으로나마 활용함으로써 다양한 상황에서 원하는 객체를 바라보는 콜백 함수를 만들 수 있는 방법입니다.

반면 예제 4-9의 경우는 처음부터 바라볼 객체를 명시적으로 obj1으로 지정했기 때문에 어떤 방법으로도 다른 객체를 바라보게끔 할 수가 없습니다.
이런 문제점 때문에 불편할뿐 아니라 메모리를 낭비하는 측면이 있음에도 예제 4-8과 같은 전통적인 방식이 널리 통용될 수밖에 없었던 측면도 있습니다.
물론 다양한 객체에 재활용할 필요성이 없는 경우라면 예제 4-9와 같이 해도 아무런 문제가 없습니다. 상황에 따라 적절한 방식을 취사선택하면 될 일입니다.

다행히 이제는 전통적인 방식의 아쉬움을 보완하는 훌륭한 방법이 있습니다. 바로 ES5에서 등장한 bind 메서드를 이용하는 방법입니다. 3-2-4 절에서 이미 소개한 내용으로 여기서는 간단히 예제 코드만 소개하고 넘어가겠습니다.

```javascript
// 예제 4-11 | 콜백 함수 내부의 this에 다른 값을 바인딩하는 방법(2) - bind 메서드 활용
var obj1 = {
    name: 'obj1',
    func: function() {
        console.log(this.name);
    }
};
setTimeout(obj1.func.bind(obj1), 1000);

var obj2 = { name: 'obj2' };
setTimeout(obj1.func.bind(obj2), 1500);
```

