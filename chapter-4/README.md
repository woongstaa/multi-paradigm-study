## 타입으로 다루는 비동기

### `AsyncIterator`, `AsyncIterable`, `AsyncGenerator` 프로토콜

- 자바스크립트는 `AsyncIterator`, `AsyncIterable`, `AsyncGenerator와` 같은 프로토콜을 제공하여 비동기 작업의 순차적 처리를 지원
- 이를 활용하면 비동기 작업을 더욱 효율적이고 유연하게 처리할 수 있으며 각 요소를 비동기적으로 생성하고 소비할 수 있다

#### `AsyncIterator`, `AsyncIterable` 인터페이스

```ts
Interface IteratorYieldResult<T> {
	done?: false;
	value: T;
}

Interface InteratorReturnResult {
	done: true;
	value: undefined;
}

Interface AsyncIterator<T> {
	next(): Promise<IteratorYieldResult<T> | IteratorRetrunResult>;
}

Interface AysncIterable<T> {
	[Symbol.asyncIterator](): AsyncIterator<T>;
}

Interface AsyncIterableIterator<T> extends AsyncIterator<T> {
	[Symbol.asyncIterator](): AsyncIterableIterator<T>;
}
```

#### `AsyncGenerator` 기본 문법

- `AsyncGenerator는` 비동기적으로 값을 생성하고 순차적으로 처리하는 기능을 제공

```ts
async function* stringAsyncTest(): AsyncIterableIterator<string> {
  yield delay(1000, "a");

  const b = (await delay(500, "b")) + "c";

  yield b;
}

async function test() {
  const asyncIterator: AsyncIterableIterator<string> = stringAsyncTest();
  const result1 = await asyncIterator.next();
  console.log(result1.value); // 약 1000ms 뒤 return a

  const result2 = await asyncIterator.next();
  console.log(result2.value); // 다시 500ms 뒤 return bc

  const { done } = await asyncIterator.next();
  console.log(done); // true
}
```

#### `toAsync` 함수

- `toAysnc` 함수는 동기적인 `Iterable` 또는 `Promise`가 포함된 `Iterable`을 받아 비동기적으로 처리할 수 있는 `AsyncIterable`로 변환

```ts
// AsyncIterator를 직접 구현하는 방식
function toAsync<T>(iterable: Iterable<T | Promise<T>>): AsyncIterable<Awaited<T>> {
  return {
    [Symbol.asyncIterator](): AsyncIterator<Awaited<T>> {
      const iterator = iterable[Symbol.iterator]();

      return {
        async next() {
          const { done, value } = iterator.next();
          return done ? { done, value } : { done, value: await value };
        },
      };
    },
  };
}

// AsyncGenerator를 사용하는 방식
async function* toAsync<T>(iterable: Iterable<T | Promise<T>>) {
  for await (const value of iterable) {
    yield value;
  }
}
```

- `toAsync`의 경우 `AsyncGenerator`를 사용하는 방식이 더 간편하고 코드도 짧으며 직관적
- 이 문제에서는 명령형이 가장 적합한 방법이라고 할 수 있다
- `toAsync` 함수의 결과는 `for await ... of` 구문과 함께 사용할 수 있다

```ts
async function test() {
  for await (const a of toAsync([1, 2])) {
    console.log(a);
  }

  for await (const a of toAsync([Promise.resolve(1), Promise.resolve(2)])) {
    console.log(a);
  }

  for await (const a of [1, 2]) {
    console.log(a);
  }

  for await (const a of [Promise.resolve(1), Promise.resolve(2)]) {
    console.log(a);
  }
}
```

- `toAsync` 함수의 필요성이 지금은 와닿지 않을 수 있다.
- 특히 위 예제의 3, 4 케이스가 잘 동작하는 것을 보면 더욱 그렇다
- 그러나 `toAsync` 함수는 점점 **타입으로 다루는 비동기 코드**에서 중요한 역할을 한다
- `toAsync` 함수는 일반 `Iterable`을 `AsyncIterable`로 변환하여 실제 런타임에서 값을 처리할 뿐만 아니라 컴파일 타임에 타입이 변경되는 것을 선언한다
- `toAsync` 함수를 실행함으로써 앞으로 비동기적으로 값을 다룰 것임을 컴파일 타임에 선언하는 효과가 있으며 다양한 코드에서 이에 대해 추론이 가능해진다

### `AsyncIterable`을 다루는 고차 함수

- `AsyncIterable`을 다루는 고차 함수는 비동기 작업을 효율적으로 처리하는 데 유용
- `mapAsync`와 `AsyncGenerator`를 활용하는 `filterAsync`를 살펴보면서 비동기 작업을 다루는 고차 함수를 만드는 패턴을 탐구

#### `AsyncIterator`를 직접 구현한 `mapAsync` 함수

```ts
function mapSync<A, B>(f: (a: A) => B, iterable: Iterable<A>): IterableIterator<B> {
  const iterator = iterable[Symbol.Iterator]();

  return {
    next() {
      const { done, value } = iterator.next();
      return done ? { done, value } : { done, value: f(value) };
    },
    [Symbol.iterator]() {
      return this;
    },
  };
}

function mapAsync<A, B>(
  f: (a: A) => B,
  asyncIterable: AsyncIterable<A>,
): AsyncIterableIterator<Awaited<B>> {
  const asyncIterator = asyncIterable[Symbol.asyncIterator]();

  return {
    async next() {
      const { done, value } = await asyncIterator.next();

      return done ? { done, value } : { done, value: await f(value) };
    },
    [Symbol.Iterator]() {
      return this;
    },
  };
}

async function* strings(): AsyncIterableIterator<string> {
  yield delay(500, "a");
  yield delay(200, "b");
}

const mapped = mapAsync((a) => a.toUpperCase(), strings()); // [a: string]

for await (const a of mapped) {
  console.log(a); // [const a: string]
}
// 500ms 뒤 : A
// 다시 200ms 뒤: B
```

- 사실상 `mapSync`와 `mapAsync` 함수는 코드와 값이 흐르는 방식이 완전히 동일
- 다만 `mapAsync` 함수는 `mapSync`와 유사한 방식으로 작동하면서 비동기 이터러블을 다룰 수 있도록 설게되었음

#### `mapAsync`를 `AsyncGenerator`로 구현하기

- 제너레이터를 사용하면 다음과 같이 간결하게 구현할 수 있다

```ts
async function* mapAsync<A, B>(
  f: (a: A) => B,
  asyncIterable: AsyncIterable<A>,
): AsyncIterableIterator<Awaited<B>> {
  for await (const value of asyncIterable) {
    yield f(value);
  }
}
```

#### `toAsync` 함수와 함께 사용하기

- 구현한 `mapAsync` 함수를 실행시키기 위해서는 `AsyncIterable`을 전달해야 하므로 `AsyncGenerator`를 사용해야 한다

```ts
async function* numbers(): AsyncIterableIterator<number> {
  yield 1;
  yield 2;
}

for await (const a of mapAsync((a) => a * 2, numbers())) {
  console.log(a);
}
// 2
// 4

for await (const a of mapAsync((a) => a * 2, toAsync([1, 2]))) {
  console.log(a);
}
// 2
// 4

for await (const a of mapAsync((a) => delay(100, a * 2), toAsync([1, 2]))) {
  console.log(a);
}
// 2
// 4
```

#### `AsyncGenerator`로 만든 `filterAsync` 함수

```ts
function* filterSync<A>(f: (a: A) => boolean, iterable: Iterable<A>): IterableIterator<A> {
  for (const value of iterable) {
    if (f(value)) {
      yield value;
    }
  }
}

async function* filterAsync<A>(
  f: (a: A) => boolean | Promise<boolean>,
  asyncIterable: AsyncIterable<A>,
): AsyncIterableIterator<A> {
  for await (const value of asyncIterable) {
    if (await f(value)) {
      yield value;
    }
  }
}

for await (const a of filterAsync((a) => a % 2 === 1, toAsync[(1, 2, 3)])) {
  console.log(a);
}
// 1
// 3

for await (const a of filterAsync((a) => delay(100, a % 2 === 1), toAsync([1, 2, 3]))) {
  console.log(a);
}
// 100ms return 1
// and 200ms after return 3
```

- `filterAsync` 함수는 비동기적으로 필터링을 수행하는 함수로 `AsyncIterable` 객체를 받아 조건 함수 `f`를 만족하는 값만 `yield`로 반환
- `for await...of` 루프를 통해 `asyncIterable`의 값을 순회하고 각 값에 대해 조건 함수 `f`를 호출
- `await`를 통해 비동기적으로 `true`인지 확인하고 조건을 만족할 때만 `yield`로 값을 반환

### 동기와 비동기를 동시에 지원하는 함수로 만드는 규약 - `toAsync`

- `toAsync` 함수가 '**타입으로 다루는 비동기**에서 매우 중요한 역할을 할 것'
- '런타임에서 일반 `Iterable`을 `AsyncIterable`로 변환'
- `toAsync` 함수를 실행하는 것을 통해 '앞으로 비동기적으로 값을 다룰 것임을 컴파일 타임에 선언하는 효과'과 있다
- 결과적으로 `AsyncIterable<T>`를 만드는 함수

#### 동기와 비동기를 모두 지원하는 `map` 함수

```ts
type MapSync = <A, B>(
	f: (a: A) => B.
	iterable: Iterable<A>
) => IterableIterator<B>;

type MapAsync = <A, B>(
	f: (a: A) => B,
	asyncIterable: AsyncIterable<A>
) => AsyncIterableIterator<Awaited<B>>;
```

- 타입스크립트에서는 인자 타입에 따라 함수 오버로드를 통해 하나의 함수로 두 가지 이상의 역할을 수행할 수 있다
- 이를 통해 컴파일 타임에 코드에서 어떤 함수를 선택해 실행하는지를 타입 추론을 통해 명확히 처리할 수 있다
- 이렇게 하면 더 높은 다형성을 가지며 범용적이면서도 간결하고 안전한 코드를 작성할 수 있다

```ts
function isIterable<T = unknown>(a: Iterable<T> | unknown): a is Iterable<T> {
	return typeof a?.[Symbol.iterator] === 'function';
}

function map<A, B>(
	f: (a: A) => B.
	iterable: Iterable<A>
): IterableIterator<B>;

function map<A, B>(
	f: (a: A) => B,
	asyncIterable: AsyncIterable<A>
): AsyncIterableIterator<Awaited<B>>;

function map<A, B>(
	f: (a: A) => B,
	iterable: Iterable<A> | AsyncIterable<A>
): IterableIterator<B> | AsyncIterableIterator<Awaited<B>> {
	return isIterable(iterable)
		? mapSync(f, iterable) // [iterable: Iterable<A>]
		: mapAsync(f, iterable) // [iterable: AsyncIterable<A>]
}
```

- `isIterable` 함수는 주어진 값이 이터러블인지 검사하며 코드의 타입 안정성을 높인다
- `map` 함수는 `mapSync`와 `mapAsync`의 시그니처를 함수 오버로드로 적용하고 하나의 함수로 통합하여 구현
- `map` 함수는 실제 구현으로 두 가지 시그너처를 통합

```ts
async function test() {
  // 1. 동기적 배열 처리: mapSync
  console.log([...map((a) => a * 10, [1, 2])]);

  // 2. 비동기 이터러블 처리: mapAsync
  for await (const a of map((a) => delay(100, a * 10), toAsync([1, 2]))) {
    console.log(a);
  }

  // 3. 비동기 이터러블을 배열로 변환: mapAsync + fromAsync
  console.log(await fromAsync(map((a) => delay(100, a * 10), toAsync([1, 2]))));

  // 4. 동기 배열을 비동기적으로 처리: mapSync + Promise.all
  console.log(await Promise.all(map((a) => delay(100, a * 10), [1, 2])));
}
```

1. 동기 배열 `[1, 2]`를 `mapSync`를 통해 각 요소에 `a * 10`연산을 적용하고 결과를 출력, `mapSync`는 동기 이터레이터를 반환하므로 `[10, 20]`이라는 결과를 즉시 얻을 수 있다.
2. `toAsync`를 사용해 비동기 이터러블을 생성하고 각 요소에 `delay(100, a * 10)`을 적용하여 `mapAsync`로 처리, `for await ...of`루프는 각 요소를 순회하면서 100ms 마다 10과 20을 순차적으로 출력
3. `mapAsync`로 변환된 비동기 이터러블을 `fromAsync`를 사용하여 배열로 변환, 이 과정에서 모든 요소가 처리된 후 `[10, 20]`이 200ms 뒤에 출력
4. 동기 배열 `[1, 2]`를 `mapSync`로 처리하여 각 요소에 `delay(100, a * 10)`을 적용하고 `Promise.all`을 사용하여 모든 비동기 작업이 완료될 때까지 기다린다. 이로 인해 100ms 뒤에 `[10, 20]`이 출력

- 위 예제를 통해 `map` 함수가 동기 및 비동기 이터러블을 모두 효과적으로 처리할 수 있음을 확인
- 특히 4번에서 `Promise.all`을 사용해 비동기 작업을 병렬로 처리할 수 있음을 보여줌
- 이처럼 의도적으로 비동기 작업을 순차적으로 제어하지 않고 병렬 헬퍼 함수인 `Promise.all`에게 작업을 위임하는 선택을 할 수도 있음

#### 동기와 비동기를 모두 지원하는 `filter` 함수

```ts
function filter<A>(f: (a: A) => boolean, iterable: Iterable<A>): IterableIterator<A>;

function filter<A>(
  f: (a: A) => boolean | Promise<boolean>,
  asyncIterable: AsyncIterable<A>,
): AsyncIterableIterator<A>;

function filter<A>(
  f: (a: A) => boolean | Promise<boolean>,
  iterable: Iterable<A> | AsyncIterable<A>,
): IterableIterator<A> | AsyncIterableIterator<A> {
  return isIterable(iterable)
    ? filterSync(f as (a: A) => boolean, iterable)
    : filterAsync(f, iterable);
}
```

- 함수 `f`는 별도의 타입 검증을 하지 않았으므로 `f`가 `(a: A) => boolean` 타입임을 명시하기 위해 `as` 키워드를 사용

```ts
const isOdd = (a: number) => a % 2 === 1;

async function test() {
  // 1. filterSync -> mapSync로 동작
  console.log([...map((a) => a * 10, filter(isOdd, naturals(4)))]);
  // [10, 30]

  // 2. toAsync -> filterAsync -> mapAsync로 동작
  const iter2: AsyncIterableIterator<string> = map(
    (a) => a.toFixed(2),
    filter((a) => delay(100, isOdd(a)), toAsync(naturals(4))),
  );
  for await (const a of iter2) {
    console.log(a);
  }
  // 100ms after: 1.00
  // and 200ms after: 3.00
  // and 100ms after: end

  // 3. filter -> toAsync -> mapAsync로 동작
  console.log(await fromAsync(map((a) => delay(100, a * 10), toAsync(filter(isOdd, naturals(4))))));
  // 200ms after: [10, 30]
}
```

1. `filter` 함수가 동기 이터러블 `naturals(4)`와 동기 조건 함수 `isOdd`를 인자로 받는다, 이 경우 `filter`함수는 `filterSync`로 동작. 이어서 `map` 함수가 각 요소에 대해 `a * 10`을 적용하여 `[10, 30]`이 출력
2. `filter` 함수가 비동기 이터러블 `toAsync(naturals(4))`와 비동기 조건 함수 `a => delay(100, isOdd(a))`를 인자로 받음, 이 경우 `filterAsync`로 동작하며 각 요소를 100ms 지연 후 `isOdd` 함수로 평가.
   필터링된 1과 3에 대해 `map` 함수가 각 요소에 `a.toFixed(2)`를 적용함으로써 _1.00_, _3.00_ 형태의 결과가 출력
3. `filter` 함수가 동기 이터러블 `naturals(4)` 동기 조건 함수 `isOdd`를 인자로 받음. 필터링된 1과 3을 `toAsync`를 활용해 `AsyncIterable`로 변환하여 `mapAsync`로 선택되도록 한 후 각 요소에 `delay(100, a * 10)`을 적용
   `fromAsync` 함수를 통해 `AsyncIterableIterator<number>`의 요소를 꺼내 `Promise<number[]>` 타입의 배열로 변환하여 `[10, 30]`이 출력

### 타입 시스템 + 비동기 함수형 함수 + 클래스

- 타입 시스템과 비동기 함수형 함수, 클래스를 결합하면 비동기 작업을 더욱 구조화하여 일관성 있게 관리할 수 있다

#### `FxIterable`과 `FxAsyncIterable`

- 클래스 활용을 더한 방법에서는 `toAsync`메서드를 통해 `FxIterable` 타입을 `FxAsyncIterable`로 변환하며 이어주는데 같은 함수로 같은 문제를 해결하지만 약간 다르게 표현

```ts
function fx<A>(iterable: Iterable<A>): FxIterable<A>;
function fx<A>(asyncIterable: AsyncIterable<A>): FxAsyncIterable<A>;
function fx<A>(
	iterable: Iterable<A> | AsyncIterable<A>
): FxIterable<A> | FxAsyncIterable<A> {
	return isIterable(iterable)
		? new FxIterable(iterable)
		: new FxAsyncIterable(iterable)/
}

class FxIterable<A> implements Iterable<A> {
	constructor(private iterable: Iterable<A>) {}

	[Symbol.iterator]() {
		return this.iterable[Symbol.iterator]();
	}

	map<B>(f: (a: A) => B): FxIterable<B> {
		return fx(map(f, this));
	}

	filter(f: (a: A) => boolean): FxIterable<A> {
		return fx(filter(f, this));
	}

	toArray(): A[] {
		return [...this];
	}

	toAsync(): FxAsyncIterable<Awaited<A>> {
		return fx(toAsync(this));
	}
}

class FxAsyncIterable<A> implements AsyncIterable<A> {
	constructor(private asyncIterable: AsyncIterable<A>) {}

	[Symbol.asyncIterator]() {
		return this.asyncIterable[Symbol.asyncIterator]();
	}

	map<B>(f: (a: A) => B): FxAsyncIterable<Awaited<B>> {
		return fx(map(f, this));
	}

	filter(f: (a: A) => boolean | Promise<boolean>) : FxAsyncIterable<A> {
		return fx(filter(f, this));
	}

	toArray(): Promise<A[]> {
		return fromAsync(this);
	}
}
```

- `implements`를 이용하면 타입스크립트로부터 인터페이스를 만족시키는 데 필요한 구현이 완료되었는지 컴파일 타임에 가이드를 받을 수 있다
- 타입스크립트에게 타입 추론을 모두 위임하는 방식으로 더욱 간결하게 작성할 수도 있다

```ts
class FxIterable<A> {
  constructor(private iterable: Iterable<A>) {}

  [Symbol.iterator]() {
    return this.iterable[Symbol.Iterator]();
  }

  map<B>(f: (a: A) => B) {
    return fx(map(f, this));
  }

  filter(f: (a: A) => boolean) {
    return fx(filter(f, this));
  }

  toArray() {
    return [...this];
  }

  toAsync() {
    return fx(toAsync(this));
  }
}

class FxAsyncIterable<A> {
  constructor(private asyncIterable: AsyncIterable<A>) {}

  [Symbol.asyncIterator]() {
    return this.asyncIterable[Symbol.asyncIterator]();
  }

  map<B>(f: (a: A) => B) {
    return fx(map(f, this));
  }

  filter(f: (a: A) => boolean | Promise<boolean>) {
    return fx(filter(f, this));
  }

  toArray() {
    return fromAsync(this);
  }
}
```

```ts
async function test() {
	console.log(
		fx(naturals(4))
			.filter(isOdd)
			.map(a => a * 10)
			.toArray*()
	);
	// [10, 30]

	const iter2 = fx(natruals(4))
		.toAsync()
		.filter(a => delay(100, isOdd(a)))
		.map(a => a.toFixed(2));

	for await (const a of iter2) {
		console.log(a);
	}
	console.log('end');
	// 100ms after: 1.00
	// and 200ms after: 3.00
	// and 100ms after: 'end'

	console.log(
		await fx(naturals(4))
			.filter(isOdd)
			.toAsync()
			.map(a => delay(100, a * 10))
			.toArray()
	);
	// 200ms after: [10, 30]
}
```

- 위 예제는 동기 및 비동기 작업을 효과적으로 관리할 수 있는 타입 스크립트의 타입 시스템과 함수형 프로그래밍 기법을 결합한 클래스를 사용하여 간결하고 일관된 방식으로 비동기 작업을 처리하는 방법을 보여준다
- `toAsync` 메서드는 `FxIterable`의 메서드 체인을 `FxAsyncIterable`의 메서드 체인으로 연결하여 동기와 비동기 작업을 자연스럽게 결합할 수 있게 한다

#### 타입 시스템을 활용한 비동기 로직 검증

- 클래스 조합 사례에서도 타입 시스템을 통해 비동기 작업 로직을 컴파일 타임에 미리 검증할 수 있다
- 이를 통해 코드의 안정성을 높이고 런타임 오류를 줄일 수 있다

```ts
async function test() {
  const iter2 = fx(naturals(4))
    .filter((a) => delay(100, isOdd(a))) // 타입 오류 발생 (TS2322)
    .map((a) => a.toFixed());

  // TS2322: Type Promise<boolean> is not assignable to type boolean
}
```

- 이 코드는 `toAsync`를 사용하지 않아 타입스크립트가 `FxIterable`에서 비동기함수 `filter`를 사용하려고 할 때 타입 에러가 발생한다
- *ECMAScript*는 `Iterable`과 `AsyncIterable`을 용도에 적합히게 구분하여 설계하였고 타입스크립트는 이 구조를 활용하여 강력한 타입 검사를 제공

#### 동기와 비동기를 모두 지원하는 `reduce` 함수

- `map`과 `filter`는 모두 이터러블을 반환하는 함수
- 반면에 `reduce`는 최종 결과를 만들어내는 함수
- 프로그램의 결론은 결국 배열 상태로 끝나지는 않음
- `map`과 `filter`가 메모리 내에서 이터러블을 유지하면서 값을 변형해 나간다면
- `reduce`는 이터러블을 순회하며 누적된 값을 생성하는 과정에서 이터러블을 해체하고 최종적인 값을 만든다
- 즉 `reduce`는 이터러블을 통해 내부의 값을 결합하고 최종적으로 메모리에서 이터러블과 그 내부의 값을 제거하는 역할을 한다
- 정리하면 `reduce`는 `Iterable<A>`를 받아 순회하면서 누적된 값을 합산해 `Acc`를 만들거나 `AsyncIterable<A>`를 받아 `Promise<Acc>`를 반환하는 함수

```ts
function reduce<A, Acc>(f: (acc: Acc, a: A) => Acc, acc: Acc, iterable: Iterable<A>): Acc;
function reduce<A, Acc>(
  f: (acc: Acc, a: A) => Acc | Promise<Acc>,
  acc: Acc,
  asyncIterable: AsyncIterable<A>,
): Promise<Acc>;
function reduce<A, Acc>(
  f: any,
  acc: Acc,
  iterable: Iterable<A> | AsyncIterable<A>,
): Acc | Promise<Acc> {
  return isIterable(iterable) ? reduceSync(f, acc, iterable) : reduceAsync(f, acc, iterable);
}

function reduceSync<A, Acc>(f: (acc: Acc, a: A) => Acc, acc: Acc, iterable: Iterable<A>): Acc {
  for (const a of iterable) {
    acc = f(acc, a);
  }

  return acc;
}

async function reduceAsync<A, Acc>(
  f: (acc: Acc, a: A) => Acc | Promise<Acc>,
  acc: Acc,
  asyncIterable: AsyncIterable<A>,
): Promise<Acc> {
  for await (const a of asyncIterable) {
    acc = await f(acc, a);
  }

  return acc;
}
```

- 동기와 비동기 이터러블 모두에 대응하기 위해 함수 오버로드를 사용
- `reduce` 함수는 받은 `Iterable` 혹은 `AsyncIterable`의 내부 요소를 하나씩 평가하며 순회
- 비동기 상황에서도 요소를 순차적으로 하나씩 추출하기 때문에 이터레이터를 통한 지연평가의 이점을 살린 로직을 작성할 수 있도록 한다
- 비동기 상황에서도 이터레이터를 연속적으로 가공함으로써 선언적으로 로직을 작성할 수 있게 되며 `reduce`는 합을 맞추어 하나씩 평가하고 하나의 `Promise<Acc>`의 값으로 누적
- 지연 평가는 지연 평가될 이터레이터를 잘 만드는 것도 중요하지만 최종적으로 평가하는 곳에서 알맞게 평가하는 것을 통해 완성됨

```ts
const result: number = fx(naturals(4))
  .filter(isOdd)
  .map((a) => a * 10)
  .reduce((acc, a) => acc + a, 0);

const resultPromise: Promise<number> = fx(naturals(4))
  .filter(isOdd)
  .map((a) => delay(100, a * 10))
  .toAsync()
  .reduce((acc, a) => acc + a, 0);

console.log(result, await resultPromise);
// 40 40
```

## 비동기 에러 핸들링

- 비동기 프로그래밍에서 에러를 효과적으로 처리하는 것은 필수
- 비동기 로직의 특성상 에러가 발생했을 때 코드가 어디서 실행되고 있는지 명확하게 파악하기 어려울 수 있다
- 에러 처리가 적절하지 않으면 성능 문제와 부수 효과, 디버깅의 어려움이 발생
- 특히 네트워크 요청, 파일 읽기/쓰기, 데이터베이스 연동과 같이 외부 시스템과 상효작용하는 작업은 에러 가능성이 높으므로 이를 효율적으로 핸들링하는 방법이 중요

### 여러 이미지를 불러와서 높이 구하기

```ts
function loadImage(url: string): Promise<HTMLImageElement> {
  return new Promise((resolve, reject) => {
    const image = new Image();
    image.src = url;
    image.onload = function () {
      resolve(image);
    };
    image.onerror = function () {
      reject(new Error(`load error : ${url}`));
    };
  });
}

async function calcTotalHeight(urls: string[]) {
  try {
    const toalHeight = await urls
      .map(async (url) => {
        const img = await loadImage(url);
        return img.height;
      })
      .reduce(async (a, b) => (await a) + (await b), Promise.resolve(0));

    return totalHeight;
  } catch (e) {
    console.error("Error: ", e);
  }
}

console.log(await calcTotalHeight(urls));
// 585
console.log(await calcTotalHeight(urls2));
// Error: load error...
// undefined
```

- `calcTotalHeight` 함수는 `urls`의 각 URL에 대해 이미지를 비동기적으로 로드하고 높이를 계산한 후 총합을 반환
- 에러가 발생할 경우 `try/catch` 블록에서 처리하며 에러 내용을 로그로 남김
- 위 예제는 잘 동작하는 듯해 보이지만 사실은 다음과 같은 문제가 있음
  - 불필요한 부하
    - 에러가 발새해도 나머지 URL에 대해 이미지 다운로드를 모두 시도함
  - 부수 효과
    - 위 상황은 단순히 `GET` 요청이지만 위와 같은 방식으로 `POST`나 데이터베이스의 `INSERT`와 같은 API를 제어한다면 불필요한 요청으로 부수 효과가 발생하게 됨

### 개선된 비동기 로직

```ts
async function calcTotalHeight2(urls: string[]) {
  try {
    const totalHeight = await fx(urls)
      .toAsync()
      .map(loadImage)
      .map((img) => img.height)
      .reduce((a, b) => a + b, 0);
    return totalHeight;
  } catch (e) {
    console.error("Error: ", e);
  }
}

console.log(await calcTotalHeight(urls));
// 585
console.log(await calcTotalHeight(urls2));
// Error: load error...
// loadImage 두 번만 시도: undefined
```

- 첫 번째 URL에서 에러가 발생하면 나머지 URL 요청을 멈추고 즉시 에러를 처리
- Promise와 AsyncIterator를 안전하게 제어하고 의도대로 정확하게 동작하면서도 가독성이 높음
- 비동기 프로그래밍에서 효율적이고 견고한 코드를 작성하는 핵심은 `Promise`와 동기-비동기 상황에 대해 정확하게 이해하는 것
- 특히 비동기 작업이 점점 복잡해지는 현대 애플리케이션 환경에서는 반드시 코드 흐름과 에러 처리를 명확하게 설계해야함
- 이 과정에서 `AsyncIterator`와 같은 자바스크립트의 강력한 비동기 프로토콜을 활용하면 함수형 프로그래밍의 선언적 스타일과 지연 평가를 결합하여 더욱 유연하고 유지보수성이 높은 코드를 작성할 수 있다
- 이러한 방식은 단순히 코드를 동작하게 만드는 것을 넘어 명확하고 일관성이 있는 로직을 통해 개발자의 생산성과 사용자 경험 모두를 향상시키는 데 기여

### 에러가 제대로 발생되도록 하는 것이 핵심

- 비동기 프로그래밍에서 가장 중요한 것은 단순히 에러를 핸들링하는 것이 아니라 에러가 제대로 발생되도록 설계하는 것
- 에러가 발생해야 할 상황에서 이를 적절히 발생시키는 것은 코드의 신뢰성과 유지보수성을 높이는 핵심 원칙

```ts
const getTotalHeight = (urls: string[]) =>
  fx(toAsync(urls))
    .map(loadImage)
    .map((img) => img.height)
    .reduce((a, b) => a + b, 0);
```

- 위 함수에서는 내부에서 에러를 처리하지 않는다
- 대신 호출하는 곳에서 에러를 감지하고 처리하게 한다
- 따라서 에러 핸들링 코드를 의도적으로 제외한 것
- 왜 이게 더 좋은 방법일까?

```ts
try {
  const height = await getTotalHeight(urls);
  // ...
} catch (e) {
  console.error(e);
}

// or
async function myFunction(urls: string[]) {
  try {
    return await getTotalHeight(urls);
  } catch {
    return 0;
  }
}

console.log(await myFunction(urls));
console.log(await myFunction(urls2));
```

- 이러한 접근 방식은 순수 함수를 작성하는 데 유리하며 부수 효과를 관리하기도 용이함
- 에러 핸들링은 에러가 발생하는 맥락에 가깝게 작성해야 가장 효과적
  - e.g. 네트워크 요청, 파입 입출력 같은 부수 효과를 발생시키는 코드
- 에러를 호출하는 쪽에서 처리하도록 하면 각 호출자가 자신에게 필요한 방식으로 에러를 핸들링할 수 있는 유연성을 가질 수 있다
  - e.g. 에러 로깅, 에러 메세지 표시, 자동 복구 로직, 0 fallback
- 이러한 유연성은 코드의 재사용성과 유지보수성을 크게 향상시킨다
- 또한 이러한 방식은 에러를 감추지 않는다. 에러를 감추는 것은 문제의 원인을 파악하기 어렵게 하고 예기치 못한 동작을 유발할 가능성이 크다
- 반면 에러를 명확히 발생시키고 이를 호출자에게 위임하면 문제를 조기에 탐지하고 적절히 대응할 수 있다
- 결과적으로 에러를 내부에서 처리하지 않고 호출자에게 위임함으로써 코드의 책임을 명확히 분리하고 다양한 상황에 적합한 에러 처리방식을 지원할 수 있다. 이는 단순히 코드를 깨끗하게 유지하는 것 이상의 가치를 제공한다

#### 에러가 제대로 발생되도록 설계하기

##### Promise, async/await, try/catch를 정확히 이해하고 활용하기

- 비공기 작업을 수행할 때 `Promise`와 `aysnc/await`를 적절히 사용하여 에러가 명확히 드러나도록 작성
- `try/catch` 구문을 활용하면 호출자에게 에러를 명확히 전달할 수 있다

##### 에러를 숨기지 않고 명확히 드러내기

- 불필요하게 에러를 처리하려고 하거나 지나치게 복잡한 에러 핸들링 코드를 작성하면 오히려 에러가 숨겨질 가능성이 높다
- 에러를 감추기보다는 발생하도록 두고 이를 상위 레벨에서 처리하거나 로깅 도구를 통해 모니터링하는 것이 좋다

##### 순수 함수는 에러를 발생시키도록 설계

- 순수 함수는 부수 효과를 가지지 않으므로 에러를 발생시키고 이를 상위 호출자에게 위임하는 방식이 바림직
- 순수 함수 내부에서 에러를 처리하려고 시도하면 함수의 목적이 흐려질 수 있다

##### 제너레이터 / 이터레이터 / 이터러블을 활용한 선언적 프로그래밍

- 제너레이터와 이터러블을 활용하면 코드의 표현력을 높이는 동시에 비동기 작업에서 에러 핸들링을 보다 직관적이고 명확하게 설계할 수 있다
- 예를 들어 비동기 이터러블을 통해 에러 발생 시점을 제어하고 에러가 전파되는 흐름을 선언적으로 표현할 수 있다

##### 에러 핸들링 코드는 부수 효과 코드 근처에 작성

- 네트워크 요청, 파일 입출력, 데이터베이스 쿼리와 같은 부수 효과를 발생시키는 코드 근처에서 에러를 처리해야 에러의 원인과 해결 방안을 명확히 할 수 있다.
- 부수 효과와 무관한 영역에서 에러를 처리하려고 하면 디버깅과 유지보수가 어려워진다

##### Sentry.io와 같은 에러 로깅 서비스 활용

- 에러 핸들링의 한계를 보완히기 위해 `Sentry.io`와 같은 에러 로깅 서비스를 활용하면 에러가 발생하는 모든 상황을 실시간으로 모니터링할 수 있다
- 이를 활용하면 발생한 에러를 놓치지 않고 프로덕션 환경에서도 문제를 빠르게 파악하고 대응할 수 있다
- 이때도 에러가 숨겨지지 않고 제대로 발생되도록 코드가 설계되어 있어야한다. 그래야만 이러한 도구를 통해 모든 에러를 효과적으로 관리할 수 있다
