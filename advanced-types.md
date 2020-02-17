# 목차 (Table of contents)

[교차 타입 (Intersection Types)](#교차-타입-Intersection-Types)
[유니언 타입 (Union Types)](#유니언-타입-union-types)

# 교차 타입 (Intersection Types)

교차 타입은 여러 타입을 하나로 결합합니다.
기존 타입을 합쳐 필요한 모든 기능을 가진 하나의 타입을 얻을 수 있습니다.
예를 들어, `Person & Serializable & Loggable`은 `Person` *과* `Serializable` *그리고* `Loggable`입니다.
즉, 이 타입의 객체는 세 가지 타입의 모든 멤버를 갖게 됩니다.

기존 객체-지향 틀과는 맞지 않는 믹스인(mixin)이나 다른 컨셉들에서 교차 타입이 사용되는 것을 볼 수 있습니다.
(JavaScript에는 이런 것들이 많습니다!)
믹스인을 어떻게 만드는지 간단한 예제를 보겠습니다:

```ts
function extend<First, Second>(first: First, second: Second): First & Second {
    const result: Partial<First & Second> = {};
    for (const prop in first) {
        if (first.hasOwnProperty(prop)) {
            (result as First)[prop] = first[prop];
        }
    }
    for (const prop in second) {
        if (second.hasOwnProperty(prop)) {
            (result as Second)[prop] = second[prop];
        }
    }
    return result as First & Second;
}

class Person {
    constructor(public name: string) { }
}

interface Loggable {
    log(name: string): void;
}

class ConsoleLogger implements Loggable {
    log(name) {
        console.log(`Hello, I'm ${name}.`);
    }
}

const jim = extend(new Person('Jim'), ConsoleLogger.prototype);
jim.log(jim.name);
```

# 유니언 타입 (Union Types)

유니언 타입은 교차 타입과 밀접하게 관련되어 있지만, 매우 다르게 사용됩니다.
가끔, `숫자`나 `문자열`을 매개변수로 기대하는 라이브러리를 사용할 때가 있습니다.
예를 들어, 다음 함수를 사용할 때입니다:

```ts
/**
 * 문자열을 받고 왼쪽에 "padding"을 추가합니다.
 * 만약 'padding'이 문자열이라면, 'padding'은 왼쪽에 더해질 것입니다.
 * 만약 'padding'이 숫자라면, 그 숫자만큼의 공백이 왼쪽에 더해질 것입니다.
 */
function padLeft(value: string, padding: any) {
    if (typeof padding === "number") {
        return Array(padding + 1).join(" ") + value;
    }
    if (typeof padding === "string") {
        return padding + value;
    }
    throw new Error(`Expected string or number, got '${padding}'.`);
}

padLeft("Hello world", 4); // "    Hello world"를 반환합니다.
```

`padLeft`의 문제는 매개변수 `padding`이 `any` 타입으로 되어있다는 것입니다.
즉, `숫자`나 `문자열` 둘 다 아닌 인수로 호출할 수 있다는 것이고, TypeScript는 이를 괜찮다고 받아들일 것입니다.

```ts
let indentedString = padLeft("Hello world", true); // 컴파일 타임에 통과되고, 런타임에 오류.
```

전통적인 객체지향 코드에서, 타입의 계층을 생성하여 두 타입을 추상화할 수 있습니다.
이는 더 명시적일 수는 있지만, 좀 과하다고 할 수도 있습니다.
`padLeft`의 기존 버전에서 좋은 점은 그냥 원시값을 전달할 수 있다는 것입니다.
즉 사용법이 간단하고 간결합니다.
이 새로운 방법은 다른 곳에서 이미 존재하는 함수를 사용하려 할 때, 도움이 되지 않습니다.

`any` 대신에, *유니언 타입*을 매개변수 `padding`에 사용할 수 있습니다:

```ts
/**
 * 문자열을 받고 왼쪽에 "padding"을 추가합니다.
 * 만약 'padding'이 문자열이라면, 'padding'은 왼쪽에 더해질 것입니다.
 * 만약 'padding'이 숫자라면, 그 숫자만큼의 공백이 왼쪽에 더해질 것입니다.
 */
function padLeft(value: string, padding: string | number) {
    // ...
}

let indentedString = padLeft("Hello world", true); // 컴파일 중에 오류
```

유니언 타입은 값이 여러 타입 중 하나임을 설명합니다.
세로 막대 (`|`)로 각 타입을 분리하여 사용합니다. 그래서 `number | string | boolean`은 값의 타입이 `number`, `string` 혹은 `boolean`이 될 수 있음을 나타냅니다.

유니언 타입인 값을 가지고 있으면, 유니언에 있는 모든 타입에 공통인 멤버에만 접근할 수 있습니다.

```ts
interface Bird {
    fly();
    layEggs();
}

interface Fish {
    swim();
    layEggs();
}

function getSmallPet(): Fish | Bird {
    // ...
}

let pet = getSmallPet();
pet.layEggs(); // 성공
pet.swim();    // 오류
```

유니언 타입은 여기서 약간 까다로울 수 있으나, 익숙해지는데 약간의 직관만 있으면 됩니다.
만약 값이 `A | B` 타입을 가지고 있으면, `A` *와* `B` 둘 다 가지고 있는 멤버가 있다는 것만 *확실히* 알고 있습니다.
이 예제에서, `Bird`는 `fly`로 부르는 멤버를 가지고 있습니다.
`Bird | Fish`로 타입이 지정된 변수가 `fly` 메서드를 가지고 있는지 확신할 수 없습니다
만약 변수가 실제로 런타임에 `Fish`이면, `pet.fly()`를 호출하는 것은 오류입니다.