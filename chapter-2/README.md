# [함수형 프로그래밍]과 타입 시스템 그리고 [LISP]

## 타입 추론과 함수 타입 그리고 제네릭

### 타입 추론

- 타입스크립트에서 코드 작성 시 타입을 명시적으로 선언하지 않아도 타입스크립트 컴파일러가 자동으로 타입을 추론해주는 기능
- 덕분에 코드의 간결성을 유지하면서도 타입 안정성을 확보할 수 있음

#### 타입 추론의 기본적인 콘셉트

```ts
let a = 10;
```

- 위와 같은 상황에서는 타입을 명시적으로 선언하지 않아도 number로 추론한다

#### 변수와 상수의 타입 추론

```ts
let message = "Hello, TypeScript!";
```

- 명시적 타입 선언 없이도 `string` 타입

```ts
const selected = true;
// [const selected: true]

let checked = true;
// [let checked: boolean]
```

- const로 선언된 selected는 값을 재할당할 수 없기에 타입이 `true`로 추론
- let으로 선언된 checked는 값을 재할당하여 변경할 수 있기에 `boolean`으로 추론

#### 함수의 반환 타입 추론

```ts
function add(a: number, b: number) {
  return a + b;
}
```

- add 함수는 `number`타입의 인자를 받아 `number` 타입의 값을 반환
- 명시적으로 지정하지 않았지만 a, b를 통해 `number`로 추론

```ts
function add(a: string, b: string) {
  return a + b;
}
```

- 이전 예시와 같은 원리로 `string`으로 반환됨

```ts
function add(a: string, b: string) {
  return parseInt(a) + parseInt(b);
}
```

- 타입스크립트는 `parseInt(a)`, `parseInt(b)`가 `number` 타입의 값을 반환한다는 사실을 알고 있기에 각 결과의 자리를 `number`로 추론하여 add 함수의 반환 타입을 `number`로 추론
- 따라서 명시적으로 반환타입을 지정할 수도 있으며 이는 타입스크립트의 타입 추론과 일치

#### 객체의 속성 타입 추론

```ts
let user = {
  name: "Marty",
  age: 30,
};
```

- 위 예제에서 user 객체의 name 속성은 `string`, age의 속성은 `number` 타입으로 추론

#### 함수 인자의 타입 추론

```ts
let strs = ["a", "b", "c"];
strs.forEach((str) => console.log(str.toUpperCase())); // [str: string]
```

- 타입스크립트는 strs를 `string[]`으로 추론
- forEach 메서드는 strs 배열의 요소 타입을 기반으로 화살표 함수의 str 타입도 `string`으로 추론
- 따라서 `toUpperCase()` 메서드를 사용할 수 있어서 자동 완성이 동작, 정상적으로 컴파일 됨

- 타입스크립트의 타입 추론 사례 중 이 부분이 특히 매력적
- 고차 함수에서도 타입 추론이 동작하므로 인자로 던달받은 함수의 인자 타입을 추론할 수 있어 화살표 함수를 간결하게 작성할 수 있음
- 즉 화살표 함수의 간결한 표현을 유지하면서도 타입 안정성을 확보할 수 있음

#### 제네릭을 통한 타입 추론

- **제네릭 함수**를 사용하면 하나의 함수가 다양한 타입을 지원하여 다형성이 높은 함수가 된다.
- `identity`함수는 인자로 받은 값의 타입을 그대로 반환하는 제네릭 함수의 좋은 예

```ts
function identity<T>(arg: T): T {
  return arg;
}

const a = identity("hi"); // [const a: "hi"]
const b = identity(1); // [const b: 1]
const c = identity<string>("a"); // [const c: string]
const d = identity<number>(1); // [const d: number]

class User {}
const e = identity(new User()); // const e: User]
const f = identity((n: number) => n % 2 === 1); // [const f: (n: number) => boolean]
```

1. `identity` 함수에 문자열 "hi"를 전달하면 제네릭 타입 매개변수 `T`가 "hi"가 되고 a의 타입 역시 "hi"로 추론
2. `identity` 함수에 1을 전달하면 `T`가 1이 되고 b의 타입 역시 1이 됨
3. `identity<string>("a")`와 같이 `T`를 명시적으로 `string`으로 지정하고 인자는 "a"를 전달, 이때 c의 타입은 `string`으로 결정됨
4. `identity`함수에 `T`를 명시적으로 `number`로 지정하고 인자는 숫자 1을 전달하면 d의 타입은 `number`로 결정됨
5. `User` 인스턴스를 전달하면 `T`가 `User`로 결정되고 e의 타입 역시 `User`가 됨
6. `(n: number) => n % 2 === 1`이라는 함수를 전달하면 타입스크립는 이 함수의 타입을 `(n: number) => boolean`으로 추론하고 f의 타입을 동일하게 지정함

- 타입스크립트의 타입 추론은 코드의 가독성과 안정을을 높이는 데 중요한 역할을 함
- 이러한 방식 덕분에 개발자는 타입 시스템을 도입하고 높은 생산성을 유지할 수 있음

### 함수 타입과 제네릭

- 타입스크립트는 함수형 프로그래밍을 지원하기 위해 고차 함수, 함수 타입, 제네릭 등의 다양한 기능을 제공
- 함수 타입을 명시적으로 정의하면 함수의 입력과 출력 타입을 명확하게 표현할 수 있음
- 제네릭을 활용하면 폭넓은 타입을 지원하는 범용 함수를 만들 수 있음
- 특히 고차 함수는 인자로 전달받은 함수의 매겨변수 타입을 추론하고 함께 전달된 다른 인자들의 타입과도 연계해 타입을 유연하게 추론하도록 도움

#### 함수의 타입을 정의하는 여려 가지 방법

```ts
function double(a: number): number;
function double(a: string): string;
function double(a: number | string): number | string {
  if (typeof a === "number") {
    return a * 2;
  } else {
    return a + a;
  }
}

const num: number = double(10); // 20
const str: string = double("HI"); // ""HiHi"
```

- 타입스크립트는 **함수 오버로드**를 지원하여 동일한 함수명으로 다양한 시그니처를 정의할 수 있다.
- 이를 통해 함수의 유연성을 높이고 다양한 입력 타입을 처리할 수 있다.
- 함수 구현부에서 `typeof`연산자를 사용한 타입 가드는 런타임에서 a의 타입을 검사하여 타입에 따라 다른 로직을 실힝하도록 한다.
- 컴파일 타임에서도 타입스크립트가 if 블록 내에서 타입을 정확하게 구분하여 추론한다.
- 이를 타입스크립트에서는 `타입 가드(type guard)`에 의한 `타입 좁히기(type narrowing)`이라고 부른다.
- 이러한 타입 추론 덕분에 컴파일 타임에서 타입 안정성을 확보할 수 있다.

```ts
const multiply = (a: number, b: number): number => a * b;
const num: number = multiply(4, 5); // 20

const multiply = (a: number, b: number) => a * b;
const num: number = multiplay(4, 5); // 20
```

- 화살표 함수는 간결한 문법을 제공하며 함수의 타입을 정의할 때도 유용
- 반환 타입을 명시적으로 선언하여 타입 안전성을 확보할 수 있지만, 타입스크립트는 타입 추론이 강력하기 때문에 매개변수의 타입만 명시해도 충분

```ts
Type Add = (a: number, b: number) => number;

const add: Add = (a, b) => a + b;
```

- `Add`라는 함수 별칭을 정의하여 함수를 나타내어 `add` 함수 변수를 `Add` 타입으로 선언하면 타입에 맞게 가이드를 받으며 구현할 수 있음
- 이렇게 함수 타입을 별칭으로 정의하면 여러 곳에서 동일한 함수 타입을 재사용할 수 있다.

#### constant와 제네릭

- `constant`는 인자로 입력받은 값을 항상 그대로 돌려주는 함수를 반환, 이런 함수를 제네릭으로 구현하면 다양한 타입의 값을 처리할 수 있음

```ts
function constant<T>(a: T): () => T {
  return () => a;
}

const getFive = constant(5);
const ten: number = getFive() + getFive();
console.log(ten); // 10

const getHi = constant("Hi");
const hi2: string = getHi() + getHi();
console.log(hi2); // HiHi
```

- 여기서 `identity` 함수와 다르게 5나 'Hi'를 받고도 T가 각각 `number`와 `string`으로 추론된 이유는 타입스크립트에서는 고차 함수가 다루는 일급 함수의 인자나 반환값을 추론할 때 넓은 타입으로 추론하는 경향이 있기 때문.

## 멀티패러다임 언어에서 함수형 타입 시스템

### 이터레이션 프로토콜과 타입 다시 보기

```ts
interface IteratorYieldResult<T> {
  done?: false;
  value: T;
}

interface IteratorRetrunResult {
  done: true;
  value: undefined;
}

interface Iterator<T> {
  // 타입스크립트의 'Iterator' 인터페이스 중 필요한 부분만 남겼음
  next(): IteratorYieldResult<T> | IteratorRetrunResult;
}

interface Iterable<T> {
  [Symbol.iterator](): Iterator<T>;
}

interface IterableIterator<T> extends Iterator<T> {
  [Symbol.iterator](): IterableIterator<T>;
}
```

- 우리는 다음 개념들을 바탕으로 타입 시스템을 적용하는 방법을 배울 것
  1.  이터레이션 프로토콜과 관련된 세 가지 값인 `Iterator`, `Iterable`, `IterableIterator`에 대해 알고 있다
  2.  `for...of`문으로 순회할 수 있는 값은 오직 이터러블
  3.  전개 연산자로 배열을 만들 수 있는 값도 오직 이터러블
  4.  `IterableIterator`를 만드는 함수ㅡㄹ 구현할 때 반환 값을 `{ next() { ... }, [Symbol.iterator]() { ... } }`형식으로 구현할 수 있으며, 이 객체는 이터레이터이자 동시에 이터러블
  5.  제너레이터로도 이터레이터를 만들 수 있으며 제너레이터의 실행 결과는 `IterableIterator`
  6.  제너레이터의 `yield`와 이터레이터의 `next()`의 관계를 이해
  7.  이터레이터에 고차 함수를 조합하여 `forEach`, `map`, `filter`를 만들 수 있으며 이터레이션 프로토콜을 지원하여 언어의 기능과 상호작용하도록 만들 수 있다

### 함수형 고차 함수와 타입 시스템

- 반복자 패턴을 활용한 하무형 고차 함수들은 이터러블 자료구조를 중심으로 구성되므로 이를 이터러블 헬퍼 함수라고 부를 수 있다.

#### `forEach`와 타입

```ts
function forEach<A>(f: (a: A) => void, iterable: Iterable<A>): void {
  for (const a of iterable) {
    f(a);
  }
}

const array = [1, 2, 3];
forEach((a) => console.log(a.toFixed(2)), array); // [a: number]
// 1.00
// 2.00
// 3.00
```

1. `forEach` 옆에 제네릭 `<A>`를 작성하여 함수에서 `A` 타입을 사용하겠다고 선언
2. `A`를 활용하여 `f` 함수의 타입을 인자로 `a: A`를 받아 `void`를 반환하는 타입으로 정의
3. `f` 함수의 인자 `a`의 타입을 앞에서 선언한 제네릭 타입 `A`로 선언
4. 그리고 `iterable`의 타입을 A를 요소로 같은 `Iterable<A>`라고 정의
5. 설명을 덧붙이자면 `<A>를 선언하고 a: A와 Iterable<A>를 작성하여 iterable의 요소 타입이 A이며 그 A가 f 함수의 인자 a가 되도록 연결해주었다`라고 할 수 있다
6. `iterable`의 타입은 `Iterable<A>`이기에 `for(const a of iterable)`에서의 `a`의 타입은 `A`
7. `forEach`가 받은 `array: number[]`로 인해 `Iterable<A>`로 `Iterable<number>`가 되고 `f`의 `a`도 `number`
8. 제네릭을 잘 활용했기 때문에 `a`는 `number` 타입으로 정확히 추론되며 `toFixed(2)` 메서드를 안전하게 하출할 수 있다.

#### `map`과 타입

```ts
function* map<A, B>(f: (a: A) => B, iterable: Iterable<A>): IterableIterator<B> {
  for (const a of iterable) {
    yield f(a);
  }
}

const array = ["1", "2", "3"];
const mapped = map((a) => parseInt(a), array); // [a: string], [const mapped: IterableIterator<number>]

const array2: number[] = [...mapped];
console.log(array2);
// [1, 2, 3];

const [head] = map((a) => a.toUpperCase(), ["a", "b", "c"]);
console.log(head); // [head: string]
// A
```

1. `map<A, B>`를 작성하여 제네릭 타입 `A`와 `B`를 만듦
2. `map` 함수는 `A` 타입을 입력받아 `B` 타입을 출력하는 함수 `f`와 `Iterable<A>`를 인자로 받아 `IterableIteraor<B>`를 반환하도록 정의됨
3. 첫 번째 경우 `map` 함수를 실행할 때 전달한 `array`가 `Iterable<string>`으로 해석되어 `A`는 `string`
4. `a => parseInt(a)`의 반환 타입에 의해 `B`가 `number`가 되어 `map(a => parseInt(a), array)`의 반환 타입이 `IterableIterator<number>`가 됨
5. 따라서 `mapped`역시 `IterableIterator<number>`로 추론되고 `mapped`를 전개 연산자로 배열로 반환한 값인 `array2`가 `number[]` 타입으로 잘 처리됨
6. 두 번째 경우 `[head]`로 구조 분해 할당을 했고 역시 `string`으로 타입이 제대로 추론

#### `filter`와 타입

```ts
function* filter<A>(f: (a: A) => boolean, iterable: Iterable<A>): IterableIterator<A> {
  for (const a of iterable) {
    if (f(a)) {
      yield a;
    }
  }
}

const array = [1, 2, 3, 4];
const filtered = filter((a) => a % 2 === 0, array); // [a: number]

const array2: number[] = [...filtered]; // [const filtered: IterableIterator<number>]
console.log(array2);
// [2, 4]
```

#### `reduce`와 타입

```ts
function reduce<A, Acc>(f: (acc: Acc, a: A) => Acc, acc: Acc, iterable: Iterable<A>): Acc {
  for (const a of iterable) {
    acc = f(acc, a);
  }
  return acc;
}

const array = [1, 2, 3];
const sum = reduce((acc, a) => acc + a, 0, array);
console.log(sum); // [const sum: number]
// 6

const strings = ["a", "b", "c"];
const abc = reduce((acc, a) => `${acc}${a}`, strings);
console.log(abc); // [const abc: string]
// abc
```

1. `reduce<A, Acc>`를 작성하여 제네릭 타입 A와 Acc를 만듦
2. `reduce` 함수는 `Acc` 타입의 초깃값 `acc`와 `A` 타입의 요소를 가진 `Iterable<A>`를 인자로 받음
3. 그리고 `Acc` 타입의 초깃값과 `A` 타입의 현잿값을 받아 `Acc` 타입의 새로운 누적값을 반환하는 함수 `f`를 인자로 받는다
4. 함수는 이터러블의 각 요소를 순화하면서 `f(acc, a)`를 실행하여 누적값을 갱신함
5. 마지막으로 `reduce` 함수는 최종 누적값 `Acc`를 반환

#### `reduce` 함수 오버로드

- 자바스크립트의 `Array.prototype.reduce`는 초깃값을 생략할 수 있다.
- 여기서 함께 만든 이터러블을 다루는 `reduce`도 동일한 스펙을 지원하도록 구현
  - 초깃값이 있을 때는 세 개의 인자를 받음
  - 초깃값을 생략하고자 할 때는 f와 iterable만을 받음. 이때 이터러블의 첫 번째 요소가 초깃값이 됨
  - 초깃값 없이 빈 배열이 전달된 경우에는 누적할 수 없고 타입이 올바르지 않으므로 에러를 발생시킴

```ts
function baseReduce<A, Acc>(f: (acc: Acc, a: A) => Acc, acc: Acc, iterator: Iterator<A>): Acc {
  while (true) {
    const { done, value } = iterator.next();
    if (done) {
      break;
    }
    acc = f(acc, value);
  }
  return acc;
}

// 1.
function reduce<A, Acc>(f: (acc: Acc, a: A) => Acc, acc: Acc, iterable: Iterable<A>): Acc;

// 2.
function reduce<A, Acc>(f: (a: A, b: A) => Acc, iterable: Iterable<A>): Acc;

function reduce<A, Acc>(
  f: (a: Acc | A, b: A) => Acc,
  accOrIterable: Acc | Iterable<A>,
  iterable?: Iterable<A>,
): Acc {
  if (iterable === undefined) {
    // 3.
    const iterator = (accOrIterable as Iterable<A>)[Symbol.iterator]();
    const { done, value: acc } = iterator.next();

    if (done) {
      throw new TypeError("'reduce' of empty iterable with no initial value");
    }
    return baseReduce(f, acc, iterator) as Acc;
  } else {
    // 4.
    return baseReduce(f, accOrIterable as Acc, iterable[Symbol.iterator]());
  }
}
```

1. `reduce<A, Acc>(f: (acc: Acc, a: A) => Acc, acc: Acc, iterable: Iterable<A>): Acc`
   - 제네릭 타입 `A`와 `Acc`를 선언하고 초깃값 `acc: Acc`와 `Iterable<A>`를 인자로 받음. `(acc: Acc, a: A) => Acc`타입의 함수를 인자로 받아 누적값을 계산한 다음 최종적으로 `Acc` 타입의 값을 반환
2. `reduce<A, Acc>(f: (a: A, b: A) => Acc, iterable: Iterable<A>): Acc`
   - 제네릭 타입 `A`와 `Acc`를 선언하고 초깃값 없이 `Iterable<A>`와 `(a: A, b: A) => Acc` 타입의 함수를 인자로 받음. 이터러블의 첫 번째 요소를 초깃값으로 사용, 누적값을 계산한 다음 최종적으로 `Acc` 타입의 값을 반환
3. 마지막 인자인 `iterable`이 없는 경우 `(iterable === undefined)`
   - 두 번째 인자인 `accOrIterable`이 이터러블. `[Symbol.iterator]()` 메서드를 실행해 이터레이터로 변환, `next()` 메서드로 첫 번째 요소를 꺼냄. 빈 이터러블이면 에러를 발생, 요소가 있으면 `baseReduce`실행
4. 세 개의 인자를 모두 받은 경우 `(else)`
   - 두 번째 인자인 `accOrIterable`이 초깃값이고 `iterable`은 이터러블. `baseReduce`를 실행하여 이터레이터의 각 요소를 순회하면서 `f(acc, value)`를 실행하여 누적값을 갱신. 최종적으로 누적된 값을 반환

#### `reduce`의 에러 관리

- 자바스크립트의 `reduce`는 초깃값을 생략한 상태에서 호출되면 배열의 첫 요소를 초깃값으로 삼아 순회
- 만약 빈 배열이라면 초깃값으로 삼을 요소조차 없고 값을 산출할 수 없는 상황이므로 `TypeError`를 던져 처리를 중단함
- `Array.prototype.reduce`나 `Iterable Helpers`의 `reduce`. 그리고 이 절에서 구현한 `reduce` 모두 초깃값을 생략한 상황에서 빈 배열이나 빈 이터러블을 만났을 때 에러를 전파하도록 구현되어있음
- 이런 에러 처리를 어떻게 바라보고, 어떻게 관리해야 좋을까?
  1.  초깃값을 명시적으로 넣는 방법
  2.  빈 배열인 경우를 미리 체크해 기본값으로 반환하는 방법
  3.  `try/catch`로 에러를 처리하는 방법
  4.  지연 이터레이터인 경우에는 지연 이터레이터를 `reduce`에 전달하기 전에 배열로 변환해 길이를 체크하거나, 끝까지 평가를 미루면서 `reduce`에 넘기고 에러를 처리하는 두가지 접근이 가능
      - 전자는 빈 배열인지 미리 파악하여 기본값을 반환하는 로직이 가능해짐
      - 후자는 빈 이터레이터가 `reduce`에 들어가 발생한 에러를 던지거나 `try/catch`로 처리하는 방향을 선택해야함
- 결국 초깃값 없이 `reduce`를 사용하며 지연 이터레이터를 전달하려면, 에러 처리를 할 것인지 아니면 이 상황이 발생하지 않는다고 가정해도 되는지를 판단해 적절한 방식으로 결정해야함

### 함수 시그니처와 중첩된 함수들의 타입 추론

```ts
function* naturals(end = Infinity): IterableIterator<number> {}

function forEach<A>(f: (a: A) => void, iterable: Iterable<A>): void {}

function* map<A, B>(f: (a: A) => B, iterable: Iterable<A>): IterableIterator<B> {}

function* filter<A>(f: (a: A) => boolean, iterable: Iterable<A>): IterableIterator<A> {}

function printNumber(n: number) {

return console.log(n);

}

forEach(printNumber,

map(n => n * 10, // [n: number]

filter(n => n % 2 === 1, naturals(5))));

forEach(printNumber,

filter(n => n % 2 === 1, map(text => parseInt(text), map(el => el.textContent!, // [n: number]

// [n: number]

// [text: string]

// [el: HTMLDivElement] [Node.textContent: string | null]document.querySelectorAll('div')))));
```

- 타입을 잘 정의한 함수는 함수들을 중첩하여 평가할 때도 코드 전반에서 타입 추론을 원활하게 처리
- 고차 함수에 인자로 전달된 모든 함수의 인자 타입이 추론되므로 개발자는 직접 타입을 정의하지 않고도 안전한 코드를 작성할 수 있음

## 멀티패러다임 언어와 메타프로그래밍 - LISP로부터

- 이번 절의 예제들에서는 제네릭과 일급 함수, 클래스, 이터러블 프로토콜 등 다양한 언어 기능을 조합해 유연하고 확장성 높은 추상화를 구축하는 과정을 살펴본다
- 이를 통해 메타프로그래밍에서 얻을 법한 코드 표현력의 향상, 런타임에서의 기능 변형을 구현하고 마치 언어 자체를 확장한 듯한 경험을 얻을 수 있다
- **메타프로그래밍**이란 프로그램이 자기 자신이나 다른 프로그램을 데이터처럼 바라보며 분석/변형/생성하거나 실행하는 프로그래밍 기법을 의미
- 프로그램이 코드를 데이터로 다루면서 동적으로 조작하고 확장하는 방식은 전통적인 LISP계열 언어에서 극대화되었음
- 이를 활용하면 코드구조나 평가 과정을 직접 재정의하거나 매크로를 통해 언어 구문을 자유롭게 다룰 수 있다

### Pipe Operator

- 예제 코드처럼 오른쪽 아래에서 왼쪽 위 방향으로 읽어야하는 코드는 익숙하지 않아 가독성이 떨어질 수 있다
- LISP는 지연 평가와 메타프로그래밍 측면에서 탁월한 강점이 있으므로 개발자가 직접 pipe 함수를 만들어 이러한 문제를 해결할 수 있다
- 몇몇 언어에서는 이미 Pipe Operator를 지원하여 가독성 문제를 효과적으로 완화 중

```ts
forEach(printNumber,
	map(n => n * 10,
	filter(n => n % 2 === 1,
	naturals(5))));
// 10
// 30
// 50

// Pipe Operator (Stage 2)
naturals(5)
	|> filter(n => n % 2 === 1, %)
	|> map(n => n * 10, %)
	|> forEach(printNumber, %)
// 10
// 30
// 50

forEach(map(filter(naturals(5), n => n % 2 === 1), n => n * 10), printNumber);

naturals(5)
	|> filter(%, n => n % 2 === 1)
	|> map(%, n => n * 10)
	|> forEach(%, printNumber)
```

- 확실히 Pipe Operator의 코드가 더 가독성이 좋음
- Pipe Operator 코드는 괜찮은 편이지만 고차 함수의 첫번째 인자 자리에 %가 있어서 시선을 방해함

### 클래스와 고차함수, 반복자, 타입 시스템을 조합하기

- 객체지향 패러다임의 클래스와 이터러블, 함수형 함수, 타입 시스템을 적절히 결합하여 가독성 문제를 해결해보자

#### 제네릭 클래스로 `Iterable` 확장하기

```ts
class FxIterable<A> {
  private iterable: Iterable<A>;

  constructor(iterable: Iterable<A>) {
    this.iterable = iterable;
  }
}
```

- `private iterable: Iterable<A>`와 같은 형태로 접근 제어자를 생성자 매개변수에 직접 명시함
- 이를 통해 필드를 정의하는 코드와 값을 할당하는 코드를 생략하고도 `iterable` 필드가 클래스 내부에 자동으로 생성, 이 방법으로 클래스 정의를 간결하게 만들 수 있다

```ts
class FxIterable<A> {
  constructor(private iterable: Iterable<A>) {}
}
```

- `FxIterable<A>`의 타입 파라미터 `A`는 `FxIterable` 클래스의 인스턴스를 생성하는 시점에 전달하는 `iterable` 인자의 타입에 따라 결정
- 이는 제네릭 함수의 타입 파라미터가 함수 호출 시점에 인자 타입에 따라 정해지는 방식과 유사
- 이제 이 제네릭 클래스에 다양한 고차 함수들을 메서드로 추가할 수 있다.

#### `FxIterable<A>`에 `map` 메서드 추가하기

```ts
class FxIterable<A> {
  constructor(private iterable: Iterable<A>) {}

  map<B>(f: (a: A) => B): FxIterable<B> {
    return new FxIterable(map((a) => f(a), this.iterable));
  }
}

const mapped = new FxIterable(["a", "b"]).map((a) => a.toUpperCase()).map((b) => b + b);

// [const mapped: FxIterable<string>]
// [a: string]
// [b: string]
```

- `map` 메서드는 `this.iterable`에 `map(f)`를 적용한 이터러블 이터레이터를 만든 후 `FxIterable<B>`를 생성하여 반환
- `FxIterable` 클래스의 인스턴스는 체이닝 방식으로 `map`을 연속적으로 실행할 수 있다.
- 이를 통해 코드를 위에서 아래로 읽을 수 있으며 제네릭을 잘 활용하여 타입 추론도 원활하게 됨
- `mapped`는 `FxIterable<string>`의 인스턴스, `a`는 `string`

#### `fx(iterable: Iterable<A>): FxIterable<A>`로 간결하게 표현하기

```ts
function fx<A>(iterable: Iterable<A>): FxIterable<A> {
  return new FxIterable(iterable);
}

const mapped2 = fx(["a", "b"])
  .map((a) => a.toUpperCase())
  .map((b) => b + b);

// [const mapped2: FxIterable<string>]
```

#### `filter`, `forEach` 메서드 만들기

```ts
class FxIterable<A> {
  constructor(private iterable: Iterable<A>) {}

  map<B>(f: (a: A) => B): FxIterable<B> {
    return new FxIterable(map(f, this.iterable));
  }

  filter(f: (a: A) => boolean): FxIterable<A> {
    return new FxIterable(filter(f, this.iterable));
  }

  forEach(f: (a: A) => void): void {
    return forEach(f, this.iterable);
  }
}
```

- `fx` 함수를 활용하면 내부 코드를 더 간결하게 표현할 수 있다

```ts
class FxIterable<A> {
  constructor(private iterable: Iterable<A>) {}

  map<B>(f: (a: A) => B): FxIterable<B> {
    return fx(map(f, this.iterable));
  }

  filter(f: (a: A) => boolean): FxIterable<A> {
    return fx(filter(f, this.iterable));
  }

  forEach(f: (a: A) => void): void {
    return forEach(f, this.iterable);
  }
}
```

- 이제 `forEach`로 순회하여 출력 효과를 만들 수 있다

```ts
fx(["a", "b"])
  .map((a) => a.toUpperCase())
  .map((a) => a + a)
  .forEach((a) => console.log(a));
// AA
// BB
```

- 앞서 작성했던 코드들도 `fx`를 활용하여 작성

```ts
// 함수 중첩
forEach(printNumber,
	map(n => n * 10,
	filter(n => n % 2 === 1,
	naturals(5))));

// 파이프 오퍼레이터
naturals(5)
	|> filter(n => n % 2 === 1, %)
	|> map(n => n * 10, %)
	|> forEach(printNumber, %)

// 체이닝
fx(naturals(5))
	.filter(n => n % 2 === 1)
	.map(n => n * 10)
	.forEach(printNumber);
// 10
// 30
// 50
```

- 체이닝 방식은 현대 언어의 접근 방식과 매우 유사하여 익숙하기도 할뿐더러 가독성이 뛰어남
- 체이닝 방식은 연속적인 메서드 호출을 통해 데이터 변환 방식을 직관적으로 표현할 수 있으며 각 단계가 명확하게 드러나기 때문에 코드의 흐름을 쉽게 파악할 수 있다
- 또한 체이닝 방식은 사용할 수 있는 메서드를 IDE에서 힌트로 제공받을 수 있다

#### `reduce` 메서드 만들기

```ts
class FxIterable<A> {
  constructor(private iterable: Iterable<A>) {}

  // ... 메서드 생략 ...

  reduce<Acc>(f: (acc: Acc, a: A) => Acc, acc: Acc): Acc; // ❶
  reduce<Acc>(f: (a: A, b: A) => Acc): Acc; // ❷
  reduce<Acc>(f: (a: Acc | A, b: A) => Acc, acc?: Acc): Acc {
    return acc === undefined
      ? reduce(f, this.iterable) // ❸
      : reduce(f, acc, this.iterable); // ❹
  }
}
```

- `reduce` 메서드는 앞서 구현한 것 처럼 메서드 오버로드를 통해 두가지 호출 방식을 지원하도록
- `reduce` 메서드 사용 예제

```ts
// 초깃값이 없을 때
const num = fx(naturals(5)) // FxIterable<number> (1, 2, 3, 4, 5)
  .filter((n) => n % 2 === 1) // FxIterable<number> (1, 3, 5)
  .map((n) => n * 10) // FxIterable<number> (10, 30, 50)
  .reduce((a, b) => a + b); // [a: number] [b: number]

console.log(num); // [num: number]
// 90

// 초깃값이 있을 때
const num2 = fx(naturals(5)) // FxIterable<number> (1, 2, 3, 4, 5)
  .filter((n) => n % 2 === 1) // FxIterable<number> (1, 3, 5)
  .map((n) => n * 10) // FxIterable<number> (10, 30, 50)
  .reduce((a, b) => a + b, 10); // [a: number] [b: number]

console.log(num2); // [num2: number]
// 100
```

### LISP(클로저)에서 배우기 - 코드가 데이터, 데이터가 코드

- LISP의 가장 큰 특징은 '코드가 데이터이고 데이터가 코드'라는 개념
- 이를 통해 프로그래밍 언어의 구문을 데이터 구조로 표현하고 조작할 수 있다
- 결과적으로 프로그램이 동적으로 새로운 코드를 생성하고 실행할 수 있어 메타프로그래밍을 비롯한 다양한 고급 기법을 손쉽게 구현할 수 있다
- 이러한 특성은 코드의 유연성과 확장성을 극대화할 수 있는 기반

#### 클로저

- LISP 계열에 속함
- JVM 위에서 실행, 현대적인 LISP 언어의 특성과 함께 자바의 강력한 라이브러리 생태계를 활용
- 불변성과 일급 함수를 강조하며 동시성 프로그래밍과 관련된 강력한 기능들을 지원하는 언더
- 코드와 데이터를 동일하게 취급함
- 이러한 특성은 메타프로그래밍을 가능하게 하며 코드의 유연성과 확장성을 높임

#### 클로저 시작하기 - S-표현식

- LISP의 S-표현식은 리스트 형태의 구문 표현을 의미
- 이를 통해 코드와 데이터를 동일한 구조(리스트)로 다룰 수 있다
- 이는 곧 코드 자체를 데이터로 조작할 수 있다는 이야기

```clojure
(+ 1 2)
```

- 위 예제는 두 수를 더하는 표현식인 동시에 다음과 같은 리스트 구조로 해석할 수 있다
  - 첫 번째 요소: 연산자(함수) +
  - 나머지 요소: 피연산자 1과 2
- LISP 계열 언어에서는 함수 호추이 리스트 구조로 이루어지며 리스트의 첫 번째 요소가 함수, 그 뒤 요소들이 함수에 전달할 인자
- 타입스크립트로 단순화해 표현한다면 아래와 같음

```ts
[add, 1, 2];
```

- 하나의 배열에 두 수를 더하는 `add` 함수와 나머지 요소인 1과 2가 담겨 있다
- `[add, 1, 2]` 자체는 배열이고 데이터
- 이때 이 데이터를 평가하는 함수가 있다면 데이터를 코드로 만들어 평가할 수 있을 것

```ts
type Evaluatable<A, B> = [(...args: A[]) => B, ...A[]]; // ❶

function evaluation<A, B>(expr: Evaluatable<A, B>) {
  // ❷
  const [fn, ...args] = expr;
  return fn(...args);
}

const add = (a: number, b: number) => a + b;
const result: number = evaluation([add, 1, 2]);
console.log(result); // 3
```

- 이로써 데이터(배열)로 표현된 코드(함수 호출)를 `evaluation` 함수를 통해 실제로 실행해볼 수 있다

#### 클로저에서 map이 실행될 때

```clojure
(map #(+ % 10) [1 2 3 4])
```

- 이 코드는 다음과 같이 동작
  - 첫 번째 요소: 함수 `map`
  - 두 번째 요소: 익명 함수 `#(+ % 10)`
    - 현재 요소에 10을 더하는 함수
  - 세 번째 요소: 백터 `[1 2 3 4]`
    - 클로저에서 `[]`는 백터 `()`는 리스트를 의미
- `map` 함수는 주어진 함수 `#(+ % 10)`를 백터의 각 요소에 적용한 결과를 반환
- 이를 평가하면 결과는 `(11 12 13 14)`라는 리스트 형태의 지연 시퀀스가 됨
- 다만 아직 어디에도 소비되지 않았으므로 실제로 값이 필요할 때 평가가 완료
- `#(+ % 10)`은 리더 매크로에 의해 `(fn [x] (+ x 10))` 형태의 익명 함수로 확장
- **리더 매크로**란 클로저와 같은 언어가 소스 코드를 읽는 단계에서 특정 기호나 패턴을 미리 정해진 형태의 다른 코드로 치환하는 기능

#### 앞에서부터 두 개의 값 꺼내기

- 다음 코드는 `let`과 구조분해를 통해 `map`의 결과에서 앞의 두 값을 추출하여 출력하는 예

```clojure
(let [[first second] (map #(+ % 10) [1 2 3 4])]
	(println first second))
;; 11 12
```

- `(map #(+ % 10) [1 2 3 4]`를 통해 `(11 12 13 14)` 형태의 지연 시퀀스가 생성
- `let`에서 `[first second]`로 구조를 분해하면 처음 두 요소를 추출하면서 필요한 부분만 평가한다
- `map`은 지연 평가되므로 실제로 필요할 때만 요소를 계산
- `printIn`을 통해 `first`와 `second` 값을 출력하면 결과는 `11 12`
- LISP 계열 언어에서는 코드가 리스트 형태로 표현되며 리스트는 평가되기 전까지는 단순한 데이터 구조에 불과함
- 평가 과정이 시작되면 실제 함수 호출이나 로직으로 해석되어 실행
- 이러한 값이 `(map f list)`와 같은 또 다른 리스트 구조의 코드와 결합되는 과정에서 클로저는 필요한 시점까지의 평가를 지연
- 마침내 평가가 필요한 순간이 되면 중첩된 리스트들의 조합을 실제 로직으로 완성하고 실행
- 이처럼 코드와 데이터를 동일한 형태로 다루며 필요할 때 점진적으로 평가하는 것이 LISP 계열 언어의 특징이자 핵심적인 강점 중 하나

### 멀티패러다임 언어에서 사용자가 만든 코드이자 클래스를 리스트로 만들기

- 클로저로 만든 것과 동일한 시간 복잡도를 가지면서 동일한 표현력을 가지도록 `FxIterable` 클래스를 확장해보자

```ts
class FxIterable<A> {
  constructor(private iterable: Iterable<A>) {}

  // ... 메서드 생략 ...

  toArray(): A[] {
    // ❶
    return [...this.iterable];
  }
}

const [first, second] = fx([1, 2, 3, 4])
  .map((a) => a + 10)
  .toArray(); // ❷

console.log(first, second); // 11 12 ❸
```

- 이 코드는 다음과 같이 동작함
  1.  추가한 `toArray()` 메서드는 내부 이터러블을 배열로 변환하여 반환, `return [...this.iterable];` 문장을 통해 이터러블 객체를 전개 연산자를 사용해 배열로 변환
  2.  `fx` 함수는 `FxIterable` 인스턴스를 생성, `map()` 메서드를 사용하여 각 요소에 10을 더한 후 `toArray()` 메서드를 통해 변환된 배열을 반환
  3.  구조 분해 할당을 통해 배열의 첫 번째와 두 번째 값을 `first`와 `second` 변수에 바인딩하면 출력 결과는 `11`과 `12`가 된다
- 원하는 결과는 얻었지만 `map(...)`뒤에 `.toArray()`라는 코드가 추가된 점, `toArray()`를 할 때 모든 요소를 평가하여 길이가 4인 배열이 만들어지는 점이 클로저 코드와 비교하여 아쉬운 점
- `FxIterable`을 지금까지 계속 다뤘던 이터레이션 프로토콜을 따르는 값으로 만들면 이를 해결할 수 있다

```ts
class FxIterable<A> {
  constructor(private iterable: Iterable<A>) {}

  [Symbol.iterator]() {
    return this.iterable[Symbol.iterator]();
  }

  // ... 메서드 생략 ...
}

const [first, second] = fx([1, 2, 3, 4]).map((a) => a + 10);
console.log(first, second); // 11 12
```

- `FxIterable` 클래스에 `[Symbol.iterator]()` 메서드를 구현하여 `this.iterable`을 이터레이터로 변환하여 반환
- 그러면 `toArray()` 메서드 없이도 `first`와 `second` 값을 추출할 수 있다
- 따라서 두 값을 추출하기 위해 10을 더하는 연산이 단 두 번만 실행됨

### LISP의 확장성 - 매크로와 메타프로그래밍

- 아직 평가되지 않은 상태에서 10을 더할 리스트가 있다면, 여기에 동적으로 다른 코드를 추가할 수 있다
  - 홀수만 제거, 특정 요소를 제외하는 등의 새 기능
- 개발자가 특정 로직으로 요소를 제거하는 함수를 만들고 이를 리스트의 첫 번째 요소로 둔다면, 그 리스트는 마치 새로운 연산자와 함수로 구성된 코드처럼 동작
- 이러한 과정을 통해 개발자가 스스로 언어의 기능을 확정하고 `let`과 같은 기존언어 기능과 자연스럽계 연계할 수 있음
- 아래는 `reject`라는 함수를 직접 정의하여 기존 언어에 없던 연산을 클로저에 추가하는 예시

```clojure
(defn reject [pred coll] ;; ❶
	(filter (complement pred) coll))

(let [[first second] (reject odd? (map #(+ % 10) [1 2 3 4 5 6]))] ;; ❷
	(println first second)) ;; ❸
;; 12 14 // ❹
```

1. `reject`함수는 `pred` 조건을 만족하지 않는 요소들만 남기기 위해 `filter`와 `complement`를 사용
2. 3행의 코드를 오른쪽 끝부터 살펴보면 `(map #(+ % 10) [1 2 3 4 5 6])`은 각각에 10을 더해 `(11 12 13 14 15 16)`을 생성, `reject odd?`는 홀수인 요소를 제거하므로 `(12 14 16)`을 반환. `let`에서 `[first second]` 구조분해를 통해 `(12 14 16)` 중 처음 두 요소를 `first`와 `second`에 바인딩
3. `printIn`을 통해 `first`와 `second` 값을 출력하면 결과로 확인할 수 있다

#### 매크로

- LISP 계열 언어에서 매크로는 **코드(리스트 형태)를 입력받아 코드(리스트형태)를 반환하는 하나의 함수**라 할수 있다.
- 매크로는 **컴파일 타임**에 작동하여 코드가 아직 실행되지 않은 '구문'상태일 때 원하는 형태로 재구성
- 이를 통해 최종적으로 실행될 코드를 유연하게 동적으로 만들어내고 원하는 새로운 문법이나 기능을 언어에 손쉽게 추가할 수도 있음

```clojure
(defmacro unless [test body]
	`(if (not ~test) ~body nil))
```

- `test`와 `body`는 매크로에 전달되는 '코드 형태의 인자'
- 함수 호출에서는 인자들이 먼저 평가된 뒤 함수에 전달되지만 매크로에서는 인자들이 평가되지 않은 '원본 코드 형태'로 주어짐
- 이 말은 `unless` 매크로가 `test`와 `body`를 마치 함수의 인자처럼 받되, 그 값을 실행하지 않고 코드 구조(리스트) 자체로 취급한다는 의미

```clojure
(unless false
	(println "조건이 거짓이므로 이 문장은 실행됩니다."))
```

- `false`는 `unless` 매크로에서 `test` 인자로 `(println "조건이 거짓이므로 이 문장은 실행됩니다.")`는 `body`인자로 전달
- 이때 이들은 평가되지 않은 코드 조각(리스트) 형태 그대로 매크로에 넘어감
- 그리고 `unless` 매크로는 이 코드 조각들을 활용해 컴파일 타임에 다음과 같은 새로운 코드를 생성함

```clojure
(if (not false)
	(println "조건이 거짓이므로 이 문장은 실행됩니다.")
	nil)
```

- 결국 `unless` 매크로는 `test`와 `body`라는 코드를 입력받아 최종적으로 실행될 새로운 코드 조각을 반환하는 코드 변환기
- 컴파일러는 반환된 코드를 실제 실행 코드로 사용하게 되므로 매크로를 통해 언어가 제공하지 않는 새로운 구문이나 기능을 마음껏 만들어낼 수 있다
- 정리하자면 `test`와 `body`는 매크로에 전달되는 '코드조각'이며 `unless` 매크로는 코드 조각들을 재구성하여 컴파일 타임에 새로운 코드를 '뱉어내는' 역할을 한다
- 이로써 개발자는 자신의 언어 확장 도구를 손쉽게 확보할 수 있다
- 이는 LISP 계열 언어가 지닌 강력한 메타프로그래밍 능력 중 하나

#### ->> 매크로

- 클로저에서는 파이프라인 형태의 코드를 표현하기 위해 `- >>` 매크로를 사용

```clojure
(let [[first second] (->> [1 2 3 4 5 6] ;; ❶, ❹
		(map #(+ % 10)) ;; ❷
		(reject odd?))] ;; ❸
	(println first second)) ;; ❺
;; 12 14 // ❻
```

- 클로저에서는 매크로를 개발자가 직접 정의할 수 있으며 특수 문자나 기호를 활용한 표현도 가능
- 이를 통해 `- >>`같은 새로운 구문을 언어에 손쉽게 추가할 수 있다
- 쉼표 없이 괄호만 사용하는 S-표현식과 결합하여 더욱 간결한 코드를 만들어낼 수 있다
- 이러한 강력한 확장성과 유연성은 프로그램의 구문을 데이터 구조로 표현하고 이를 지연된 값처럼 다를 수 있는 LISP 계열 언어의 특성 덕분

#### `reject` 메서드를 `FxIterable`에 추가하기

```ts
class FxIterable<A> {
  constructor(private iterable: Iterable<A>) {}

  [Symbol.iterator]() {
    return this.iterable[Symbol.iterator]();
  }

  // ... 메서드 생략 ...

  reject(f: (a: A) => boolean): FxIterable<A> {
    return this.filter((a) => !f(a));
  }
}

const isOdd = (a: number) => a % 2 === 1;

const [first, second] = fx([1, 2, 3, 4, 5, 6])
  .map((a) => a + 10)
  .reject(isOdd);

console.log(first, second);
// 12 14
```

- 앞서 클로저에서 파이프라인과 구조 분해 할당을 사용한 위 코드와 비교해보면, 두 예제 모두 같은 프로그래밍 패러다임과 철학을 공유하며 이를 통해 본질적으로 동일한 의미와 가치를 구현하고 있음

#### 코드, 객체, 함수가 협력하여 구현한 언어의 확장

```ts
const [first, second] = fx([1, 2, 3, 4, 5, 6])
  .map((a) => a + 10)
  .reject(isOdd);
```

- 구조 분해 할당
  - `const [first, second]`
- 객체지향 메서드 체이닝 패턴
  - `fx().map().reject()`
- 함수형 고차 함수와 LISP
  - `map = (f: (a: A) => B, iterable: Iterable<A>) => Iterable<B>`
- 이 외에도 명령형 코드인 제너레이터, 객체지향 패턴인 이터레이터, 일급 함수, 클래스, 제네릭과 타입 추론 등의 개념과 기능들이 서로 상호 작용하여 많은 가치와 가능성을 담아내고 있다
- 결론적으로 이 코드는 멀티패러다임적으로 구현되었으며 동시에 멀티패러다임 언어가 지원할 모든 기능과 상호작용이 가능할 범용적인 코드

### 런타임에서 동적으로 기능 확장하기

#### `to`로 확장하고 객체지향적인 객체와 호흡하기

- `FxIterable`이 이터러블이기 때문에 전개 연산자를 써서 `Array`로 변환할 수 있어 `toArray()`가 반드시 필요한 건 아님
- 하지만 `toArray()`를 이용해 `Array`로 변환하면 체이닝을 이어갈 수 있다는 특징이 있음

```ts
const sorted = fx([5, 2, 3, 1, 4, 5, 3])
  .filter((n) => n % 2 === 1)
  .map((n) => n * 10)
  .toArray() // Array<number>로 변환
  .sort((a, b) => a - b); // Array.prototype.sort로 오름차순으로 정렬

console.log(sorted);
// [10, 30, 30, 50, 50]

const sorted2 = [
  ...fx([5, 2, 3, 1, 4, 5, 3])
    .filter((n) => n % 2 === 1)
    .map((n) => n * 10),
].sort((a, b) => a - b);

console.log(sorted2);
// [10, 30, 30, 50, 50]
```

- 체이닝 방식이 순차적으로 읽히고 동작하기 때문에 같은 기능이지만 가독성이 더 좋음

```ts
class FxIterable<A> {
  constructor(private iterable: Iterable<A>) {}

  [Symbol.iterator](): Iterator<A> {
    return this.iterable[Symbol.iterator]();
  }

  // ... 메서드 생략 ...

  to<R>(converter: (iterable: Iterable<A>) => R): R {
    return converter(this.iterable);
  }
}

const sorted = fx([5, 2, 3, 1, 4, 5, 3])
  .filter((n) => n % 2 === 1)
  .map((n) => n * 10)
  .to((iterable) => [...iterable]) // iterable을 받아 전개 연산자로 배열로 변환
  .sort((a, b) => a - b); // [Array<number>.sort(compareFn?: ...): number[]]

console.log(sorted); // const sorted: number[]
// [10, 30, 30, 50, 50]
```

- `Array`로 변환하고 타입도 `Array`로 잘 추론되어 메서드 체이닝을 안전하게 이어갔으며 배열을 정렬하는 고차 함수 `sort`의 `compareFn`의 인자 타입도 `number`로 잘 추론되고 있음
- 사실 `FxIterable`은 자체로 곧 `Iterable`이므로 다음과 같이 `this`만을 넘기도록 구현해도 동일하게 동작

```ts
class FxIterable<A> {
  constructor(private iterable: Iterable<A>) {}

  [Symbol.iterator](): Iterator<A> {
    return this.iterable[Symbol.iterator]();
  }

  // ... 메서드 생략 ...

  filter(f: (a: A) => boolean) {
    return fx(filter(f, this)); // <-- return fx(filter(f, this.iterable));
  }

  toArray() {
    return [...this]; // <-- return [...this.iterable];
  }

  to<R>(converter: (iterable: this) => R): R {
    return converter(this); // <-- return converter(this.iterable);
  }
}

const sorted = fx([5, 2, 3, 1, 4, 5, 3])
  .filter((n) => n % 2 === 1)
  .map((n) => n * 10)
  .to((iterable) => [...iterable]) // iterable인 this를 받아 전개 연산자로 배열로 변환
  .sort((a, b) => a - b); // [a: number] [b: number]

console.log(sorted); // const sorted: number[]
// [10, 30, 30, 50, 50]
```

- `converter` 함수의 인자 타입도 `this`로 표현하여 처리
- `converter`에 전달한 값 역시 자기 자신인 이터러블
- 이렇게 작성하면 코드가 간결해지는 동시에 타입 추론 역시 잘 동작하므로 메서드 체이닝을 안전하게 사용할 수 있다
- `to` 메서드를 이용하면 배열이 아닌 값으로도 변환하여 메서드 체이닝을 이어갈 수 있다

```ts
const set = fx([5, 2, 3, 1, 4, 5, 3])
  .filter((n) => n % 2 === 1)
  .map((n) => n * 10)
  .to((iterable) => new Set(iterable));

console.log(set);
// Set(3) {50, 30, 10}

const size = fx([5, 2, 3, 1, 4, 5, 3])
  .filter((n) => n % 2 === 1)
  .map((n) => n * 10)
  .to((iterable) => new Set(iterable))
  .add(10) // [Set<number>.add(value: number): Set<number>]
  .add(20).size; // set.size

console.log(size); // [size: number]
// 4
```

- 이처럼 `Set`으로 변환할 수 있으며 `Set`의 `add` 매서드와 `size`를 사용하여 4를 출력
- 이 과정에서ㅓ 타입 추론이 정확하게 이루어져 코드 힌트를 얻으며 안전하게 체이닝을 이어갈 수 있음

#### `Set`의 집합 메서드와 함께 사용하기

- 자바스크립트의 `Set`은 집합 메서드들을 지원하므로 다음과 같이 객체지향적인 객체와 이터레이션 프로토콜을 조화롭게 결합해 멀티패러다임적으로 활용 가능

```ts
const set = fx([5, 2, 3, 1, 4, 5, 3])
  .filter((n) => n % 2 === 1)
  .map((n) => n * 10)
  .to((iterable) => new Set(iterable)) // Set으로 변환, 중복 요소 제거: Set {50, 30, 10}
  .difference(new Set([10, 20])); // Set에서 [10, 20]과의 차집합: Set {50, 30}

console.log([...set]);
// [50, 30]
```

#### `chain`으로 확장하기

- 이번에는 `iterable`을 반환하는 함수를 인자로 받아 그 결과를 다시 `FxIterable`로 이어갈 수 있는 `chain` 메서드를 도입해본다
- 이렇게 하면 동적으로 생성된 이터러블을 바로 체이닝에 포함시켜 다양한 변환을 유연하게 적용할 수 있다

```ts
class FxIterable<A> {
  constructor(private iterable: Iterable<A>) {}

  [Symbol.iterator](): Iterator<A> {
    return this.iterable[Symbol.iterator]();
  }

  // ... 메서드 생략 ...

  chain<B>(f: (iterable: this) => Iterable<B>): FxIterable<B> {
    return fx(f(this)); // new FxIterable(f(this));
  }
}
```

- `chain`을 활용하면 이터러블을 받아 이터러블을 한환하는 어떤 함수든지 동적으로 만들어 `FxIterable`을 런타임에 확장할 수 있다

```ts
const result = fx([5, 2, 3, 1, 4, 5, 3])
  .filter((n) => n % 2 === 1) // [50, 30, 10, 50, 30]
  .map((n) => n * 10)
  .chain((iterable) => new Set(iterable)) // Set으로 중복 제거, Set도 이터러블
  .reduce((a, b) => a + b); // [FxIterable<number>.reduce<number>(f: ...): number]

console.log(result); // [result: number]
// 90

const result2 = fx([5, 2, 3, 1, 4, 5, 3])
  .filter((n) => n % 2 === 1)
  .map((n) => n * 10) // [50, 30, 10, 50, 30]
  .chain((iterable) => new Set(iterable)) // Set으로 중복 제거, Set도 이터러블
  .map((n) => n - 10) // [FxIterable<number>.map<number>(f: ...): FxIterable<number>]
  .reduce((a, b) => `${a}, ${b}`); // [FxIterable<number>.reduce<string>(f: ...): string]

console.log(result2); // [result2: string]
// 40, 20, 0
```

- `chain` 메서드를 추가함으로써 이터러블을 반환하는 함수를 동적으로 적용하거나 컬랙션을 다른 자료구조로 변환한 뒤 다시 체이닝을 이어갈 수 있게 되었습니다.
- 이를 통해 `FxIterable`은 언어와 자연스럽게 어우러져 타입 안전한 메서드 체이닝을 제공하고 구조 분해 할당 등의 언어 기능과도 매끄럽게 연동할 수 있음을 확인했습니다.
- 또한 타입스크립트의 타입 시스템을 효과적으로 활용하여 타입 추론이 원활하게 이뤄지도록 설게했기 때문에 별도의 타입 명시 없이도 안전하게 값 변환 과정과 체이닝을 이어갈 수 있다

### 언어를 확장하는 즐거움

- 객체지향 기반의 언어가 LISP 계열 언어의 메타프로그래밍 수준에 한 걸음 다가갈 수 있었던 중요한 전환점이 바로 일급 함수의 도입이었다고 생각
- 과거에도 인터페이스와 반복자 패턴을 활용할 수는 있었으나 반복자 내부에서 외부 함수를 직접 받아 실행할 수 있는 일급 함수가 없었다면 함수형 패러다임의 다양한 함수를 구현할 수 없었을 것
- 정리하자면 클래스 기반 반복자 패턴에 최근 일급 함수가 결합되면서 다양한 언어들이 멀티패러다임 언어로 진화하였으며 이터레이션 프로토콜의 도입으로 일관되고 표준화된 방식으로 언어 기능을 확장할 수 있게 됨
- 그 결과 개발자는 언어 스펙이나 컴파일러를 변경하지 않고도 클래스와 함수형 고차 함수, 객체지향 패턴, 제네릭, 커링, 이터러블 프로토콜 등을 유기적으로 결합하여 고도화된 추상화와 언어 확장 효과를 얻을 수 있다
- 결과적으로 현대 멀티패러다임 언어들이 제공하는 다양한 기능을 깊이 이해하고 전략적으로 활용하는 능력은 개발자에게 강력한 무기가 된다
- 탄탄한 기본기를 바탕으로 다양한 문제에 접근할 때 개발자는 더욱 창의적인 응용력을 발휘하여 문제를 효과적이고 확장성 있게 해결할 수 있을 것
