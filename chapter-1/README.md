# 멀티패러다임이 현대 언어를 확장하는 방법

## 객체지향 디자인 패턴의 반복자 패턴과 일급 함수

- 객체지향 기반 언어들은 반복자 패턴을 통해 지연성 있는 이터레이션 프로토콜은 구현
- 일급 함수가 추가되면서 이를 바탕으로 `map`, `filter`, `reduce`, `take` 같은 이터레이터 헬퍼 함수들이 구현될 수 있었음
- 객체지향 디자인 패턴인 반복자 패턴과 함수형 패러다임의 일급 함수가 만나 함수형 패러다임의 지연 평가와 리스트 프로세싱을 구현해나감

### GoF의 반복자 패턴

- 반복자 패턴은 객체 지향 디자인 패턴 중 하나로, 컬랙션의 요소를 순차적으로 접근하는 규약을 제시함
- 아래는 반복자 구조를 타입스크립트 인터페이스 정의를 사용해 표현한 코드

```ts
interface IteratorYieldResult<T> {
  done?: false;
  value: T;
}
interface IteratorReturnResult {
  done: true;
  value: undefined;
}

interface Iterator<T> {
  next(): IteratorYieldResult<T> | IteratorReturnResult;
}
```

- 반복자 패턴은 컬랙션의 내부 구조를 노출하는 대신 next() 같은 public 메서드로 내부 요소에 접근할 수 있도록 설계되었음

### ArrayLike로부터 Iterator 생성하기

```ts
class ArrayLikeIterator<T> implements Iterator<T> {
  private index = 0;
  constructor(private arrayLike: ArrayLike<T>) {}

  next(): IteratorResult<T> {
    if (this.index < this.arrayLike.length) {
      return {
        value: this.arrayLike[this.index++],
        done: false,
      };
    } else {
      return {
        value: undefined,
        done: true,
      };
    }
  }
}

const arrayLike: ArrayLike<number> = {
  0: 10,
  1: 20,
  2: 30,
  length: 3,
};

const iterator = new ArrayLikeIterator(arrayLike);

console.log(iterator.next()); // { value: 10, done: false }
console.log(iterator.next()); // { value: 20, done: false }
console.log(iterator.next()); // { value: 30, done: false }
console.log(iterator.next()); // { value: undefined, done: true }
```

- ArrayLikeIterator는 GoF의 반복자 패턴을 따르고 있음
- 이 클래스가 지원하는 컬랙션 타입은 ArrayLike

```ts
const array: Array<string> = ["a", "b", "c"];
const iterator2: Iterator<string> = new ArrayLikeIterator(array);

console.log(iterator2.next()); // { value: 'a', done: false }
console.log(iterator2.next()); // { value: 'b', done: false }
console.log(iterator2.next()); // { value: 'c', done: false }
console.log(iterator2.next()); // { value: undefined, done: trues }
```

- 이터레이터의 이런 특성을 활용하여 지연평가를 구현할 수 있음

### ArrayLike를 역순으로 순회하는 이터레이터 만들기

#### Array의 reverse() 메서드

```ts
const array = ["a", "b"];
array.reverse();

console.log(array[0], array[1]); // b a
```

#### 이터레이터의 지연성을 이용한 reverse 함수 만들기

- 이터레이터는 필요할 때마다 값을 하나씩 꺼내는 '지연 평가'를 지원하므로 모든 요소를 미리 뒤집어둘 필요가 없습니다.
- 이로써 불필요한 연산과 메모리 사용량을 줄이며 필요한 시점에만 연산이 이루어지도록 개선할 수 있습니다.

```ts
function reverse<T>(arrayLike: ArrayLike<T>): Iterator<T> {
  let idx = arrayLike.length;

  return {
    next() {
      if (idx === 0) {
        return { value: undefined, done: true };
      } else {
        return { value: arrayLike[--idx], done: false };
      }
    },
  };
}

const array = ["A", "B"];
const reversed = reverse(array);
console.log(array); // ['A' , 'B']

console.log(reversed.next().value, reversed.next().value); // B A
```

- 중요한 점은 reverse 함수를 호출하는 순간에는 아무 일도 일어나지 않고 reversed.next().value를 실행할 때마다 배열을 역순으로 하나씩 효율적으로 꺼낸다는 사실

#### 지연 평가의 효율성

```ts
const array = ["A", "B", "C", "D", "E", "F"];
const reversed = [...array].reverse();
console.log(reversed[0], reversed[1], array[0], array[1]); // F, E, A, B

const array2 = ["A", "B", "C", "D", "E", "F"];
const reversed2 = reverse(array2);
console.log(reversed2.next().value, reversed.next().value, array2[0], array2[1]); // F E A B
```

- 전자는 원본을 지키기 위해 동일한 크기의 배열을 복사한 다음 전체를 반전
- 후자는 원래도 원본을 변경하지 않기 때문에 복사가 필요하지 않음!

### 지연 평가되는 map 함수

- 일급 함수와 고차 함수는 함수형 프로그래밍 패러다임의 핵심적인 구성 요소

```ts
function map<A, B>(transform: (value: A) => B, iterator: Iterator<A>): Iterator<B> {
  return {
    next(): IteratorResult<B> {
      const { value, done } = iterator.next();

      return done //
        ? { value, done }
        : { value: transform(value), done };
    },
  };
}
```

- map 함수 역시 next()를 실행하기 전까지는 아무런 작업을 하지 않음
- 외부에서 next() 메서드를 호출하면 그때 원본 이터레이터의 next() 메서드를 호출하여 값을 가져옵니다.
- 고차 함수는 이나로 받은 함수를 원하는 시점에 실행시킬 수 있는 구조를 가짐

```ts
const array = ["A", "B", "C", "D", "E", "F"];
const iterator = map((str) => str.toLowerCase(str), reverse(array));
console.log(iterator.next().value, iterator.next().value); // f, e
```

- **반복자 패턴**의 지연성은 지연 평가가 가능한 객체를 생성할 수 있게 해주고 일급 함수는 고차함수를 정의할 수 있게함
- 결과적으로 이 두 가지를 조합하면 map, filter, take, reduce 등의 지연 평가를 활용하거나 지연 평가된 리스트를 다루는 고도화된 리스트 프로세싱 함수를 구현할 수 있다

## 명령형 프로그래밍으로 이터레이터를 만드는 제너레이터 함수

- 정통적인 객체지향 디자인 패턴인 반복자 패턴이 함수형 패러다임의 일급 함수와 만나며 서로의 가치를 더욱 높이고 있다
- 명령형 패러다임으로 작성되는 제너레이터 역시 이 조합화 호환되며, 이 세가지 패러다임이 하나의 언어 안에서 협력하며여 객체지향, 함수형, 명령형 패러다임을 함께 고도화하고 언어를 멀티패러다임적으로 발전시키고 있다
- 제너레이터는 객체지향, 함수형 패러다임과 명령형 스타일이 서로 협력할 수 있게하는 중요한 기반을 제공

### 제너레이터 기본 문법

- **제너레이터**는 명령형 스타일로 **이터레이터**를 작성할 수 있게 해주는 문법
- 호출 시 곧바로 실행되지 않고 이터레이터 객체를 반환

#### yield와 next()

- next() 메서드를 호출하면 제너레이터 함수의 본문이 yield 키워드를 만날때까지 실행
- yield 키워드를 통해 외부로 값을 반환하고 이후 next()를 다시 호출하면 이전 실행 지점부터 이어서 함수가 재개

```ts
function* generator() {
  yield 1;
  yield 2;
  yield 3;
}

const iter = generator();

console.log(iter.next()); // { value: 1, done: false }
console.log(iter.next()); // { value: 2, done: false }
console.log(iter.next()); // { value: 3, done: false }
console.log(iter.next()); // { value: undefined, done: true }
```

- 만약 yield 1; yield 2; 사이에 console.log('hi');가 있다면 yield2;까지 동작해야하기 때문에 console.log + yield 2까지 리턴이 된다

#### 제너레이터와 제어문

```ts
function* generator(condition: boolean) {
  yield 1;

  if (condition) {
    yield 2;
  }

  yield 3;
}

const iter1 = generator(false);

console.log(iter1.next()); // { value: 1, done: false }
console.log(iter1.next()); // { value: 3, done: false }
console.log(iter1.next()); // { value: undefined, done: true }

const iter2 = generator(true);

console.log(iter1.next()); // { value: 1, done: false }
console.log(iter1.next()); // { value: 2, done: false }
console.log(iter1.next()); // { value: 3, done: false }
console.log(iter1.next()); // { value: undefined, done: true }
```

- 제너레이터 안에서 if문을 사용하여 이터레이터가 리스트를 만드는 로직을 제어할 수 있다

#### yield\* 키워드

- yield\* 키워드는 제너레이터 함수 안에서 이터러블을 순회하며 해당 이터러블이 제공하는 요소들을 순차적으로 반환하도록 해준다

```ts
function* generator() {
  yield 1;
  yield* [2, 3];
  yield 4;
}

const iter = generator();

console.log(iter.next()); // { value: 1, done: false }
console.log(iter.next()); // { value: 2, done: false }
console.log(iter.next()); // { value: 3, done: false }
console.log(iter.next()); // { value: 4, done: false }
console.log(iter.next()); // { value: undefined, done: true }
```

#### naturals 제너레이터 함수

```ts
function* naturals() {
  let n = 1;
  while (true) {
    yield n++;
  }
}

const iter = naturals();

console.log(iter.next()); // { value:1, done: false }
console.log(iter.next()); // { value:2, done: false }
console.log(iter.next()); // { value:3, done: false }
```

- naturals 제너레이터 함수는 무한 루프를 사용하여 자연수를 생성하지만 next 메서드를 호출할 때만 n을 반환하고 다시 일시 중지하기 때문에 프로세스나 브라우저가 멈추지 않는다

#### 제너레이터로 작성한 reverse 함수

```ts
function* reverse<T>(arrayLike: ArrayLike<T>): IterableIterator<T> {
  let idx = arrayLike.length;

  while (idx) {
    yield arrayLike[--idx];
  }
}
```

- 이전에 작성한 리버스 함수랑 구현은 다르지만 동작은 동일함
- 이전 함수는 idx라는 상태를 바라보는 next() 메서드를 가진 객체로 구현한 반면, 여기서는 제어문을 활용한 명령형 코드로 작성함
- 이를 통해 우리는 현대 프로그래밍 언어에서 동일한 문제를 객체지향 박식이나 명령형 등 여러패러다임 중 하나를 선택하여 해결할 수 있다는 사실을 생각해볼 수 있다.

## 자바스크립트에서 반복자 패턴 사례: 이터레이터션 프로토콜

- **이터레이션 프로토콜**은 자바스크립트의 규약
- ES6에서 도입된 이터레이션 프로토콜은 어떤 객체가 이터러블인지 여부를 나타내는 규칙과 해당 규칙을 따르는 문법들을 제공하는 언어 전반의 규약

### 이터레이터와 이터러블

- 어떤 객체가 이터레이터를 반환하는 `[Symbol.iterator]() { return { next() { ... } }; }` 메서드를 가지고 있다면 **이터러블**
- 이터러블 객체는 for...of문, 전개 연산자, 구조 분해 등 다양한 기능과 함께 사용할 수 있다.
- 대표적으로 Array, Map, Set 등이 있고 Web API도 컬렉션 유형의 값들을 이터러블로 만들어 이터레이션 프로토콜을 따르고 있다.

#### 이터레이터

```ts
function naturals(end = Infinity): Iterator<number> {
  let n = 1;

  return {
    next(): IteratorResult<number> {
      return n <= end //
        ? { value: n++, done: false }
        : { value: undefined, done: true };
    },
  };
}
```

- 자연수를 생성하는 이터레이터를 반환하는 함수를 제너레이터가 아닌 일반 함수로 생성
- 이전에 제너레이터로 작성한 naturals와 동일, 동작과 결과도 모두 동일

#### for...of문으로 순회하려면

```ts
const iterator = naturals(3);

// TS2488: Type Iterator<number, any, undefined>
// must have a '[Symbol.iterator]()' method that returns an iterator
for (const num of iterator) {
  console.log(num); // 1, 2, 3
}
```

- 위와 같이 사용하려면 타입에러가 발생
- 제대로 동작하게 하려면 아래와 같이 수정

```ts
function naturals(end = Infinity): IterableIterator<number> {
  let n = 1;

  return {
    next(): IteratorResult<number> {
      return n <= end //
        ? { value: n++, done: false }
        : { value: undefined, done: true };
    },
    [Symbol.iterator]() {
      return this;
    },
  };
}

const iterator = naturals(3);

for (const num of iterator) {
  console.log(num);
}

// 1
// 2
// 3
```

```ts
interface IteratorYieldResult<T> {
  done?: false;
  value: T;
}

interface IteratorReturnResult {
  done: true;
  value: undefined;
}

interface Iterator<T> {
  next(): IteratorYieldResult<T> | IteratorReturnResult;
}

interface Iterable<T> {
  [Symbol.iterator](): Iterator<T>;
}

interface IterableIterator<T> extends Iterator<T> {
  [Symbol.iterator](): IterableIterator<T>;
}
```

- Iterator
  - { value, done } 객체를 반환하는 next() 메서드를 가진 값
- Iterable
  - 이터레이터를 반환하는 [Symbol.iterator]() 메서드를 가진 값
- IterableIterator
  - 이터레이터이면서 이터러블인 값
- 이터레이션 프로토콜
  - 이터러블을 for...of문, 전개 연산자, 구조 분해 등과 함께 사용할 수 있게 해주는 규약

#### 내장 이터러블

```ts
const array = [1, 2, 3];
const arrayIterator = array[Symbol.iterator]();

console.log(arrayIterator.next()); // { value: 1, done: false }
console.log(arrayIterator.next()); // { value: 2, done: false }
console.log(arrayIterator.next()); // { value: 3, done: false }
console.log(arrayIterator.next()); // { value: undefined, done: true }

for (const value of array) {
  console.log(value); // 1, 2, 3
}

// 1
// 2
// 3
```

- 배열 array는 기본적으로 이터러블
- Symbol.iterator 메서드를 통해 이터레이터를 생성하고 next() 메서드로 요소들을 하나씩 순회할 수 있다.
- for...of문을 사용하여 모든 요소를 다시 순회할 수 있다.

```ts
const set = new Set([1, 2, 3]);
const setIterator = set[Symbol.iterator]();

console.log(setIterator.next()); // { value: 1, done: false }
console.log(setIterator.next()); // { value: 2, done: false }
console.log(setIterator.next()); // { value: 3, done: false }
console.log(setIterator.next()); // { value: undefined, done: true }

for (const value of set) {
  console.log(value); // 1, 2, 3
}
```

- Set 객체 역시 이터러블

```ts
const map = new Map([
  ["a", 1],
  ["b", 2],
  ["c", 3],
]);
const mapIterator = map[Symbol.iterator]();

console.log(mapIterator.next()); // { value: ['a', 1], done: false }
console.log(mapIterator.next()); // { value: ['b', 2], done: false }
console.log(mapIterator.next()); // { value: ['c', 3], done: false }
console.log(mapIterator.next()); // { value: undefined, done: true }

for (const [key, value] of map) {
  console.log(key, value); // a 1, b 2, c 3
}
```

- Map 객체도 이터러블

```ts
const mapEntries = map.entries();

console.log(mapEntries.next()); // { value: ['a', 1], done: false }
console.log(mapEntries.next()); // { value: ['b', 2], done: false }
console.log(mapEntries.next()); // { value: ['c', 3], done: false }
console.log(mapEntries.next()); // { value: undefined, done: true }

for (const [key, value] of map.entries()) {
  console.log(key, value); // a 1, b 2, c 3
}
```

- map.entries() 메서드는 Map 객체의 엔트리를 IterableIterator로 반환한다.

```ts
const mapValues = map.values();

console.log(mapValues.next()); // { value:1, done: false }

for (const value of mapValues) {
  console.log(value);
}

// 2
// 3
```

- map.values() 메서드는 Map 객체의 값들을 IterableIterator로 반환한다.
- for...of문으로 나머지 값을 순회할 수 있다.
- values() 메서드로 반환된 이터레이터는 이미 첫 번째 요소를 소비했기 때문에 2부터 시작한다.

```ts
const mapKeys = map.keys();

console.log(mapKeys.next()); // { value: 'a', done: false }

for (const key of mapKeys) {
  console.log(key);
}

// b
// c
```

- map.keys() 메서드는 Map 객체의 키들을 IterableIterator로 반환한다.
- for...of문으로 나머지 키들을 순회할 수 있다.
- keys() 메서드로 반환된 이터레이터는 이미 첫 번째 요소를 소비했기 때문에 'b'부터 시작한다.

#### 정리

- 배열
  - Symbol.iterator 메서드를 통해 이터레이터를 생성할 수 있다.
  - for...of문을 사용하여 모든 요소를 순회할 수 있다.
- Set 객체
  - Symbol.iterator 메서드를 통해 이터레이터를 생성할 수 있다.
  - for...of문을 사용하여 모든 요소를 순회할 수 있다.
- Map 객체
  - Symbol.iterator 메서드를 통해 이터레이터를 생성할 수 있다.
  - for...of문을 사용하여 모든 요소를 순회할 수 있다.
- map.entries()
  - Map 객체의 엔트리를 IterableIterator로 반환한다.
  - for...of문을 사용하여 모든 엔트리를 순회할 수 있다.
- map.values(), map.keys()
  - Map 객체의 값들을 IterableIterator로 반환한다.
  - for...of문을 사용하여 모든 값을 순회할 수 있으나 이미 소비한 요소는 순회할 수 없다.

### 언어와 이터러블의 상호작용

#### 전개 연산자와 이터러블

- 전개 연산자는 이터러블 객체의 모든 요소를 개별 요소로 확장하는 데 사용

```ts
const array = [1, 2, 3];
const array2 = [...array, 4, 5, 6];
```

- 이터러블 객체는 전개 연산자를 사용하여 배열로 변환할 수 있다

```ts
const set = new Set([1, 2, 3]);
const array = [...set];

console.log(array); // [1, 2, 3]
```

- Set 객체의 요소들이 전개 연산자를 통해 배열로 변환
- Array.from(set)과 동일한 결과
- 전개 연산자는 함수 호출 시 이터러블 객체의 요소들을 개별 인자로 전달할 때도 유용

```ts
const numbers = [1, 2, 3];

function sum(...nums: number[]): number {
  return nums.reduce((a, b) => a + b, 0);
}

console.log(sum(...numbers)); // 6
```

- 함수의 인자를 전개 연산자를 활용하여 배열이든, 하나의 인자든, 여러개의 인자든 모두 처리할 수 있는 함수를 만들 수 있다

#### 구조 분해 할당과 이터러블

```ts
const array = [1, 2, 3];
const [first, second] = array;

console.log(first); // 1
console.log(second); // 2

const array2 = [1, 2, 3, 4];
const [head, ...tail] = array2;
console.log(head); // 1
console.log(tail); // [2, 3, 4]
```

```ts
const map = new Map();

map.set("a", 1);
map.set("b", 2);
map.set("c", 3);

for (const [key, value] of map.entries()) {
  console.log(`${key}: ${value}`);
}

// a: 1
// b: 2
// c: 3
```

#### 사용자 정의 이터러블과 전개 연산자

```ts
const array = [0, ...naturals(3)];

console.log(array); // [0, 1, 2, 3]
```

- 전개 연산자와 구조 분해 할당은 이터러블 프로토콜을 활용하여 자바스크립트와 타입스크립트에서 데이터와 코드를 더욱 효과적으로 다루는 방법을 제공
- 언어를 사용하는 개발자도 이터레이션 프로토콜을 통해 언어의 다양한 기능과 협업할 수 있다.
- 이러한점은 개발자에게 많은 가능성을 열어준다.
- 반복자 패턴은 컬렉션 내부 구조를 노출하는 대신 next() 같은 public 메서드를 통해 내부 요소에 접근할 수 있도록 설계
- 이는 컬렉션의 실제 구조와 상관없이 다양한 컬렉션 스타일 데이터의 요소를 일관된 방식으로 순회할 수 있도록 함
- 이터러블은 [Symbol.iterator]() 메서드를 통해 이터레이터를 반환하는 값
- 이를 통해 해당 값이 이터러블인지 검사할 수 있으며 이터레이터로 변환하거나 순회할 수 있다.
- 이 과정은 어떤 자료구조인지 상관없이 실제 구조와 무관하게 일관된 방식으로 이루어진다.

### 제너레이터로 만든 이터레이터도 이터러블

#### 제너레이터로 만든 map 함수

```ts
function* map<A, B>(f: (value: A) => B, iterable: Iterable<A>): IterableIterator<B> {
  for (const value of iterable) {
    yield f(value);
  }
}

const array = [1, 2, 3, 4];
// array의 각 요소에 2배 연산을 적용한 IterableIterator를 반환
const mapped: IterableIterator<number> = map((x) => x * 2, array);
const iterator = mapped[Symbol.iterator]();

// mapped.next()와 iterator.next()는 동일한 이터레이터를 참조하므로 이미 소비된 요소는 다시 나오지 않음
console.log(mapped.next().value); // 2
console.log(iterator.next().value); // 4

// 남은 요소만을 배열로 만들어 반환
console.log([...iterator]); // [6, 8]
```

#### 제너레이터로 만든 이터레이터와 for...of문

```ts
let acc = 0;

for (const num of map((x) => x * 2, naturals(4))) {
  acc += num;
}

console.log(acc); // 20
```

- naturals()의 반환값도 이터레이터인 동시에 이터러블이기 때문에 map함수와 조합할 수 있다.
- map 함수에 배열이 아닌 지연 평가되는 이터레이터를 전달함
- 실행 과정에서 배열을 만들지 않고도 acc에 모든 값을 더함

##### 보충설명

- for...of가 map 함수가 반환하는 이터레이터를 순회하면서 각 요소에 2배 연산을 적용하여 acc에 더함
- 실용적인 이점은 naturals(10000)이어도 메모리에 10000개를 미리 만들어두지 않고, for...of가 요청할 때마다 하나씩 만들어서 처리하고 버린다는 점

## 이터러블을 다루는 함수형 프로그래밍

### `forEach` 함수

- `forEach` 함수는 함수와 이터러블을 받아 이터러블을 순회하면서 각 요소에 인자로 받은 함수를 적용하는 고차함수

```ts
function forEach(f, iterable) {
  for (const value of iterable) {
    f(value);
  }
}

const array = [1, 2, 3];
forEach(console.log, array);

// 1
// 2
// 3
```

```ts
function forEach(f, iterable) {
  const iterator = iterable[Symbol.iterator]();
  let result = iterator.next();
  while (!result.done) {
    f(result.value);
    result = iterator.next();
  }
}

const set = new Set([4, 5, 6]);
forEach(console.log, set);
// 4
// 5
// 6
```

- `forEach` 함수는 `while` 루프를 사용하여 이터레이터를 직접 조작
- 이터레이터의 `next()` 메서드를 사용하여 각 요소를 순회
- 인자로 받은 함수 `f`를 실행하며 `value`를 전달
- `result.done`이 `true`일 때 루프를 멈춤
- `Set`도 이터러블이기 때문에 `forEach` 함수를 사용할 수 있다

### `map` 함수

- 아래 `map` 함수는 제너레이터를 사용하여 구현
- `for...of`문을 사용하여 이터러블의 각 요소인 `value`에 대한 인자로 받은 함수 `f`를 적용한 결과를 `yield `키워드로 반환

```ts
function* map(f, iterable) {
  for (const value of iterable) {
    yield f(value);
  }
}

const array = [1, 2, 3];
const mapped = map((x) => x * 2, array);
console.log([...mapped]); // [2, 4, 6]

const mapped2 = map((x) => x * 3, naturals(3));
forEach(console.log, mapped2);
// 3
// 6
// 9
```

- `map` 함수는 이터러블을 인자로 받아 이터레이터를 결과로 반환
- 반환 결과는 동시에 이터러블이기 때문에 전개 연산자를 사용하거나 `for...of`문으로 순회할 수 있다
- 그렇기에 `map` 함수는 `IterableIterator`를 반환하는 `naturals()`나 이터러블을 인자로 받는` forEach`와도 함께 사용할 수 있다

```ts
function* map(f, iterable) {
  const iterator = iterable[Symbol.iterator]();

  while (true) {
    const { value, done } = iterator.next();
    if (done) break;
    yield f(value);
  }
}

const mapped = map(
  ([k, v]) => `${k}:${v}`,
  new Map([
    ["a", 1],
    ["b", 2],
  ]),
);

forEach(console.log, mapped);
// a:1
// b:2
```

- `제너레이터`를 사용하여 구현했지만 앞선 `forEach`의 구현과는 약간 다른 패턴
  1. 무한 루프를 만듦
  2. `next()`의 결과를 구조 분해
  3. `done`이 `true`인 경우 `break;`를 수행
  4. `value`에 인자로 받은 함수 `f`를 적용한 결과를 `yield` 키워드로 반환
- `Map`의 요소인 엔트리 역시 `이터러블`이기에 `구조 분해`를 이용할 수 있으며 `forEach`와 조합하여 동작하게 함

```ts
function map(f, iterable) {
  const iterator = iterable[Symbol.iterator]();

  return {
    next() {
      const { value, done } = iterator.next();

      return done //
        ? { value, done }
        : { value: f(value), done };
    },
    [Symbol.iterator]() {
      return this;
    },
  };
}

const iterator = function* () {
  yield 1;
  yield 2;
  yield 3;
};

const mapped = map((x) => x * 10, iterator);

console.log([...mapped]); // [10, 20, 30]
```

1. 이 `map` 함수는 `IterableIterator` 객체를 직접 만들어 반환
2. `next` 메서드를 직접 구현하여 각 요소인 `value`에 대한 함수 `f`를 적용한 결과를 반환
3. 이터러블 프로토콜을 따르도록 `[Symbol.iterator]` 메서드를 추가
4. 익명 제너레이터 함수를 사용하여 1, 2, 3을 순차적으로 생성하는 이터레이터를 만든 후, `map` 함수에 전달
5. `map` 제너레이터 함수는 각 요소에 대해` x \* 10`을 적용할 준비를 해둔 이터레이터를 만듬

- 결과적으로 mapped는 전개 연산자로 모든 값을 평가할 경우 `[10, 20, 30]`을 만들 준비가 된 이터레이터
- `console.log([...mapped])`에서 전개 연산자를 사용하여 이터레이터의 모든 값을 배열로 변환하여 출력

### `filter` 함수

- `filter` 함수는 주어진 이터러블의 각 요소에 대해 조건을 확인하여 해당 조건을 만족하는 요소들만 반환하는 고차 함수

```ts
function* filter(f, iterable) {
  for (const value of iterable) {
    if (f(value)) {
      yield value;
    }
  }
}

const array = [1, 2, 3, 4, 5];
const filtered = filter((x) => x % 2 === 0, array);
console.log([...filtered]); // [2, 4]
```

- 위 `filter` 함수는 제너레이터를 사용하여 구현
- `for...of`문을 사용하여 이터러블의 각 요소에 대해 조건 함수 `f`의 식을 만족하는 요소만 `yield` 키워드로 반환

```ts
function* filter(f, iterable) {
  const iterator = iterable[Symbol.iterator]();

  while (true) {
    const { value, done } = iterator.next();
    if (done) break;
    if (f(value)) {
      yield value;
    }
  }
}

const array = [1, 2, 3, 4, 5];
const filtered = filter((x) => x % 2 === 0, array);
console.log([...filtered]); // [2, 4]
```

- `map`과 `filter` 함수의 구현에서 `for...of `방식과 `while` 방식을 비교
- `for...of`의 역할을 하는` while 루프`의 바깥 부분과 `done`으로 종료하는 곳까지는 동일
- 내부 구현 부분만 다른 것을 볼 수 있다

```ts
function filter(f, iterable) {
  const iterator = iterable[Symbol.iterator]();

  return {
    next() {
      while (true) {
        const { value, done } = iterator.next();
        if (done) return { value, done };
        if (f(value)) return { value, done };
        return this.next();
      }
    },
    [Symbol.iterator]() {
      return this;
    },
  };
}

console.log([...filter((x) => x % 2 === 1, [1, 2, 3, 4, 5])]); // [1, 3, 5]
```

1. `next()` 메서드를 구현하여 각 요소에 대해 주어진 조건 함수 `f`를 만족할 때 `{ done, value }`를 그대로 반환, 이때 `done`은` false`고 `value`에는 값이 있음
2. 주어진 조건 함수 `f`를 만족하지 않는다면 재귀적으로` this.next()`를 다시 실행하여 순회를 계속함
3. 인자로 받은 `iterable`로 만든 `iterator`의 모든 순회를 마쳐 `done`이 `true`가 되면 `{ done, value }`를 그대로 반환하여 이터레이터를 종료, 이때 `done`은 `true`고 `value`는 `undefined`임

- 위 코드의 `next()` 메서드는 `while문`이나 `for문` 같은 반복문 없이 자신의 매서드를 재귀 호출하는 방식으로 객체지향적인 코드로만 순회를 구현하여 매우 간결
- 또한 이 코드는 `꼬리 호출 최적화(tail call optimization)`가 가능
  - `꼬리 호출 최적화`를 적용할 수 있는 조건은 함수가 반환될 때 마지막으로 호출되는 함수가 재귀 호출이어야 함
  - 위 코드에서는 `this.next()` 호출이 함수의 마지막 동작이며 이 결과가 직접 반환되기 때문에 꼬리 호출 최적화가 적용될 수 있는 구조
- 아쉽게도 ES6 스펙에 포함된 꼬리 호출 최적화를 V8 엔진에서는 지원하지 않아 스택 오버플로우의 위험이 존재
- 이를 해결하기 위해 아래와 같이 변경할 수 있음
- 꼬리 호출 최적화를 대신하여 최대한 비슷한 구조로 코드를 표현하여, 간결함을 유지하면서도 매우 큰 크기의 컬렉션 까지 안전하게 처리할 수 있음

```ts
function filter(f, iterable) {
  const iterator = iterable[Symbol.iterator]();

  return {
    next() {
      do {
        const { value, done } = iterator.next();
        if (done) return { value, done };
        if (f(value)) return { value, done };
      } while (true);
    },
    [Symbol.iterator]() {
      return this;
    },
  };
}

function filter(f, iterable) {
  const iterator = iterable[Symbol.iterator]();

  return {
    next() {
      while (true) {
        const { value, done } = iterator.next();
        if (done) return { value, done };
        if (f(value)) return { value }; // done이 false인 경우 done 프로퍼티는 생략 가능
      }
    },
    [Symbol.iterator]() {
      return this;
    },
}
```

- 위 두가지 방식 모두 함수 전체를 무한 루프로 감싸므로 재귀 호출과 거의 유사한 구조를 유지하면서도 안전하게 최적화 되었음
- 현대 언어 중 스칼라와 코틀린은 모두 꼬리 재귀 함수를 반복문 형태로 변환해 스택 오버플로우를 방지하는 꼬리 재귀 최적화를 지원

### 고차 함수 조합하기

```ts
forEach(
  console.log,
  map(
    (x) => x * 10,
    filter(
      (x) => x & (2 === 1), 
      naturals(5),
    ),
  ),
);

// 10
// 30
// 50
```

- 함수가 많이 중첩되어 있기는 하지만 LISP 계열 언어들에서 흔희 사용하는 컨벤션
- 오른쪽 아래에서 왼쪽 위로 올라가며 읽는다고 생각하면 더 쉽게 이해할 수 있다
  1. `naturals(5) `결과를
  2. `x % 2 === 1` 조건으로 필터링하고
  3. `x * 10`으로 변환한 다음
  4. 모두 콘솔에 출력하라

### 재미난 `filter`

```ts
function* filter(f, iterable) {
  for (const value of iterable) {
    yield* [value].filter(f);
  }
}

const array = [1, 2, 3, 4, 5];
const filtered = filter((x) => x % 2 === 0, array);
console.log([...filtered]); // [2, 4]
```

- 약간의 재미 요소를 담은 이 코드는 각 요소를 단일 배열로 감싸고 `Array.prototype.filter`를 사용하여 if문을 대신하고 있음
- 그리고 제너레이터의 `yield*`를 사용해 단일 배열의 요소를 바로 `yield`하도록 처리
- 이 방식도 지연 평가를 지원하며 기존 방식과 시간 복잡도는 본질적으로 동일
- 이터러블의 각 요소를 한번씩 순회하므로 요소의 개수가 n일 때 시간 복잡도는 `O(n)`
- 물론 단일 요소 배열을 생성하고 `Array.prototype.filter`를 호출하는 과정에서 약간의 오버헤드가 존재하지만 이는 매우 작아 실제 실행 시간에 큰 영향을 미치지 않음

## 이터러블 프로토콜이 상속이 아닌 인터페이스로 설계된 이유

- 객체지향 프로그래밍에서 우리가 익숙하게 알고 있는 개념 중 하나는 상속
- 상속은 코드를 추상화 해 기능을 공유하는 좋은 도구이며 실제 개발 현장에서도 자주 사용
- 그런데 반복자 패턴과 이테레이터를 지원하는 헬퍼 함수들은 상속이 아닌 인터페이스로 설게되어 있음
- 이 절에서는 현대 언어가 언어 레벨 설계에서 왜 상속을 자제하고 인터페이스를 적극적으로 활용하는지 살펴봄
  - 이 절에서 상속과 인터페이스가 가리키는 것
    - 상속: 기존 클래스의 구성과 구현 모두 물려받는 클래스 상속을 의미
    - 인터페이스: 시그니처만을 정의하는 것을 의미

### `Web API`의 `NodeList`도 이터러블

```html
<ul>
  <li>1</li>
  <li>2</li>
  <li>3</li>
  <li>4</li>
  <li>5</li>
</ul>
<script>
  const nodeList = document.querySelectorAll("li");

  for (const node of nodeList) {
    console.log(node.textContent);
    // 1
    // 2
    // 3
    // 4
    // 5
  }
</script>
```

- 위 예제는 `document.querySelectorAll('li')`를 통해 문서 내의 모든 li 요소를 선택하고 이를 `NodeList`로 반환
- `NodeList`는 이터러블이므로 `for...of`문을 사용하여 각 노드를 순회
- 당연하게도 앞에서 우리가 만든 이터러블을 다루는 함수들과도 함께 사용할 수 있음

```ts
forEach(
  console.log,
  filter(
    (x) => x % 2 === 1,
    map((node) => parseInt(node.textContent), document.querySelectorAll("li")),
  ),
);
// 1
// 3
// 5

forEach(
  (element) => element.remove(),
  filter((node) => parseInt(node.textContent) % 2 === 0, document.querySelectorAll("li")),
);

// removed: <li>2</li>,
// removed: <li>4</li>
```

### 상속이 아닌 인터페이스로 해결해야 하는 이유

#### 이터러블을 사용하는 이유

- 왜 이터러블을 사용해야할까?
- `Array`의 `map`, `filter를` 쓰면 되지 않나?
- 자바스크립트에서 배열은 `map`, `filter`, `forEach`와 같은 고차 함수를 지원한다

```ts
const nodes: NodeList = document.querySelectorAll("li");

console.log(nodes[0], nodes[1], nodes.length);
// <li>1</li> <li>3</li> 3

nodes.map((node) => node.textContent);
// TypeError: nodes.map is not a function
```

- 위 코드를 실행하면 에러가 발생함
- `NodeList`는 `Array`가 아닐뿐더러 위와 같이 정의되어 있기 때문에 `Array`의 메서드를 사용할 수 없다

```ts
interface NodeList {
  readonly length: number;
  item(index: number): Node | null;
  forEach(callbackFn: (value: Node, key: number, parent: NodeList) => void, thisArg?: any): void;
  [index: number]: Node;
}
```

- 반면 위 예제는 이터레이션 프로토콜에 기반하여 `NodeList`에 `filter`, `map`, `forEach`를 바로 적용할 수 있음

#### 순회가 필요한 자료구조들인데 왜 `Array`를 상속받도록 만들지 않았을까?

- 객체지향 프로그래밍에서 코드를 추상화해 기능을 공유하는 좋은 도구로 `상속`이 있다
- 이런 상황을 해결하고자 할 때 `상속`을 떠올릴 수도 있음
- 하지만 자바스크립트나 타입스크립트의 표준 라이브러리에서 `Array`를 상속받은 내장 클래스는 없다
- `Map`, `Set`, `NodeList는` `Array`와 동일한 기능이 일부 필요하다고 해도 `Array`를 상속받지 않았음, 왜 그럴까?
- 핵심을 말하자면 이들은 모두 서로 다른 자료구조를 나타내며 각각 고유한 특성과 동작을 갖도록 설계되었기 때문
- 이들 모두 자바스크립트의 `Array`와는 내외부적으로 다른 방식으로 동작
- 만약 이들은 상속으로 연결하여 의존성을 생기게하면 불필요한 복잡성이 생기고 최적화된 동작을 보장할 수 없게 됨
- 각각의 자료구조를 발전시킬 때 서로에게 미칠 영향을 고려해야 하므로 어려움이 생길 수 있음
- 조금 더 자세히 들여다보면서 객체지향 패러다임에 대한 생각을 확장해보자
  - `Array`는 일반적인 배열의 특성과 동작 방식을 가지며 인덱스를 기반으로 요소에 접근하고 조작하는 데 최적화되어 있다
  - `Map`은 키-값 쌍을 저장하며 각 키는 유일, 순서가 없으며 키를 통해 값을 빠르게 검색할 수 있다.
  - `Set`은 유일한 값을 저장하며 중복을 허용하지 않는다. 순서가 없으며 값의 존재 여부를 빠르게 확인할 수 있다.
- 이와 같이 구조적 차이가 있는 자료구조들을 Array의 특성과 동작 방식에 맞추기 위해 상속하는 것은 부자연스럽고 비효율적

#### 그래도 `NodeList`는 `index`와 `length`를 가진, 말 그대로 `ArrayLike`인데 왜 상속받지 않을까?

- `NodeList`
  - `DOM `트리의 요소들을 순서대로 나타내는 특수한 데이터 구조를 띠며 주로 `DOM` 조작과 연관되어 있음
  - NodeList에는 라이브와 스태틱 모드가 있는데, 라이브 NodeList에는 `DOM`이 변경될 때 자동으로 업데이트됨
- `Array`
  - 생성된 후에는 정적이며 항상 수동으로 요소를 추가하거나 제거해야함
  - 자바스크립트 엔진이 메모리와 성능을 최적화된 방식으로 관리
- 상속을 받지 않는 이유는 각각 구조와 용도가 다른 객체들이 의존성을 갖게 되면 불필요한 복잡성이 발생하고 최적화가 어려워지기 때문

#### 공통 로직을 공유할 수 있는 방법

- 이터레이션 프로토콜을 활용하면 상속 없이도 다양한 자료구조를 일관성 있게 다룰 수 있다
- 각 자료구조의 특성을 유지하면서도 공통의 인터페이스를 통해 상호작용할 수 있다.
- 어떤 이터러블이더라도 이터레이션 프로토콜은 `외부 구조의 다형성`을 해결함
- 동시에 그 안에 담긴 `내부 요소의 다형성`은 주로 고차 함수에 전달되는 함수를 통해 처리하는 구조를 형성함
- 한 가지 더 흥미로운 부분은 `Array`, `Map`, `Set`은 자바스크립트의 표준 라이브러리지만 `NodeList`는 브라우저 구현 객체라는 점
- 이처럼 인터페이스에 기반한 규약은 언어나 환경에 따라 달라지는 다양한 구조를 포용할 수 있는 유연한 확장성을 제공
- 재차 강조하고 싶은 점은 이터레이션 프로토콜이 객체지향 디자인 패턴 중 하나인 반복자 패턴에 기반한다는 것
- 반복자 패턴처럼 공통의 인터페이스를 만들어 패턴화함으로써 다양한 자료구조에 사용할 공통로직을 분리할 수 있다
- 이러한 방법은 코드의 유지보수성을 높이고 다양한 자료구조에 동일한 패턴을 적용할 수 있는 더 나은 설계 방식을 제공

### 인터페이스와 클래스 상속

- **인터페이스**는 클래스나 객체가 따라야 할 규약을 정의하며 이를 통해 다양한 클래스가 동일한 방식으로 상호작용할 수 있도록 한다
- 이러한 규약을 통해 공통된 행동을 강제하고 서로 다른 클래스들이 동일한 메서드를 구현하게 함으로써 다형성을 지원하고 코드의 유연성을 높일 수 있다
- 여러 클래스가 동일한 인터페이스를 구현하면 동일한 메서드 호출 패턴으로 일관된 설계를 할 수 있다
---
- **상속**은 기존 클래스의 속성과 메서드를 물려받아 새로운 클래스를 만드는 과정
- 이를 통해 코드 재사용성을 높이고 확장성을 확보할 수 있다
- 상속을 활용하면 공통 로직을 직접 구현한 뒤 이를 필요에 따라 확장하거나 변경할 수 있다
- 그러나 상속을 남용할 경우 코드의 결합도가 높아져 유지보수가 어려워질 수 있으므로 주의가 필요하다
---
- 이 절에서 인터페이스의 장점을 강조하다 보니 자칫 인터페이스가 상속보다 우수한 방법으로 비칠 수 있으나 사실 두 기법은 목적과 용도가 다르다
- 인터페이스는 규약을 제시하여 다양한 클래스가 동일한 형식의 동작을 구현하도록 유도
- 상속은 공통 기능을 직접 구현한 뒤 이를 적절히 확정하는 데 초점을 둔다.
---
- 인터페이스는 언어나 표준 라이브러리 설계 단계에서 자주 사용되고 상속은 주로 SDK나 애플리케이션 레벨에서 사용
- 결국 목적과 상황에 맞게 인터페이스와 상속을 적절히 선택하는 것이 좋은 코드로 나아가는 첫걸음이라 할 수 있다

