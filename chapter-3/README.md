# 코드 : 객체 : 함수 = Generator : Iterator: LISP = IP : OOP : FP
#### 제너레이터는 명령형 코드로 이터레이터 생성
- 제너레이터 함수는 명령형 코드로 이터레이터를 만들 수 있는 수단
- 리스트 단위로 지연평가 하는 효과
- '코드가 리스트이고 리스트가 코드'
#### 이터레이터는 반복자 패턴의 구현체
- 컬렉션 형태의 데이터를 일반화된 패턴으로 순회하는 객체
- 지연성을 가짐
- 유한한 컬렉션뿐 아니라 무한 시퀀스도 처리
#### 이터러블 이터레이터는 명령형, 객체지향적, 함수형으로 다룰 수 있음
- 명령형
	- `next()` 메서드를 실행하며 `while`문으로 순회하거나 `for...of`문 이나 전개연산자(`...`)로 다룰 수 있다
- 객체지향적
	- 이터러블 이터레이터를 다루는 클래스를 만들거나 이터레이터 내부에서 다른 이터레이터와 통신하는 이터레이터를 만들 수 있다
- 함수형
	- 고차 함수를 통해 이터레이터와 각 요소를 처리할 함수를 전달하는 방식
	- 이터레이션 로직을 함수조합 형태로 구현하고 지연 평가와 리스트 프로세싱을 극대화
#### 이터레이터 생성 방식의 다양화
- 이터레이터를 만드는 방법
	- 직접 이터레이터 객체를 구현
	- 제너레이터를 통해 명령형 스타일로 생성
	- 리스트 프로세싱 기반 함수들을 조합하여 함수형 스타일로도 이터레이터를 생성
- 궁극적으로 이터레이터는 다음 세 가지 방식으로 만들 수 있으며 서로 1:1:1로 대체할 수 있다
	- IP (명령형)
		- 제너레이터를 통한 이터레이터 생성
	- OOP (객체지향)
		- 이터레이터 객체 직접 구현
	- FP (함수형)
		- 리스트 프로세싱 함수 조합으로 이터레이터 생성
		- 
## 코드가 곧 데이터 - 로직이 담긴 리스트
### `[for, i++, if, break]` - 코드를 리스트로 생각하기
- 코드를 리스트로 바라보는 사고방식은 프로그래밍 패러다임을 확장하는 강력한 도구
- 함수형 프로그래밍에서는 코드가 곧 데이터이고 데이터가 곧 코드인 특성을 이용해 더 읽기 쉽고 유지보수하기 좋은 코드를 작성할 수 있다
#### 명령형으로 작성한 n개의 홀수를 제곱하여 모두 더하는 함수
```ts
function sumOfSquaresOfOddNumbers(limit: number, list: number[]): number {
	let acc = 0;
	
	for (const a of list) {
		if (a % 2 === 1) {
			const b = a * a;
			acc += b;
			if (--limit === 0) break;
		}
	}
	
	return acc ;
}
```
#### `if`를 `filter`로 대체
```ts
function sumOfSquaresOfOddNumbers(limit: number, list: number[]): number {
	let acc = 0;
	
	for (const a of filter(a => a % 2 === 1, list)) {
		const b = a * a;
		acc += b;
		if (--limit === 0) break;
	}
	
	return acc ;
}
```
- 곳곳에 있던 코드 문장이 리스트 프로세싱 함수 실행으로 대체
- `filter(a => a % 2 === 1, list)`는 필터 로직을 수행하는 코드인 동시에 리스트
- 내부 로직에서 조건문이 제거되어 코드가 더욱 명확하고 단순해짐
#### 값 변화 후 변수 할당을 `map`으로 대체
```ts
function sumOfSquaresOfOddNumbers(limit: number, list: number[]): number {
	let acc = 0;
	
	for (const a of map( a => a * a, filter(a => a % 2 === 1, list))) {
		acc += a;
		if (--limit === 0) break;
	}
	
	return acc ;
}
```
- 코드 문장들이 리스트와 함수들의 조합으로 바뀌고 있음
- 이러한 개념을 LISP 문법으로 표현하여 설명하면 더욱 명확해질 것
- LISP 문법이 코드를 리스트로 다루는 개념을 더 잘 나타내기 때문
```scheme
; Scheme
(define list '(1 2 3 4 5))
(define (squre x) (* x x))
(map square (filter odd? list))
; (1 9 25)
; JavaScript
; map(square, filter(isOdd, list))
```
- LISP에서는 이러한 리스트이자 코드이자 데이터를 평가하는식으로 프로그램이 실행됨
- 즉 리스트는 코드이고 코드는 리스트이며 중첩된 리스트는 알고리즘이자 로직
- LISP의 문법은 이러한 철학을 잘 반영하며 더욱 우아하게 표현
#### `break`를 `take`로 대체
```ts
function* take<A>(limit: number, iterable: Iterable<A>): IterableIterator<A> {
	const iterator = iterable[Symbol.iterator]();
	
	while (true) {
		const { value, done } = iterator.next();
		if (done) break;
		yield value;
		if (--limit === 0) break;
	}
}

function sumOfSquaresOfOddNumbers(limit: number, list: number[]): number {
	let acc = 0;
	
	for (const a of take(limit, map( a => a * a, filter(a => a % 2 === 1, list)))) {
		acc += a;
	}
	
	return acc ;
}
```
- `take` 함수는 주어진 이터러블에서 지정된 `limit`만큼의 요소를 반환하는 지연된 리스트인 이터러블 이터이터를 반환
- 위 코드에서 `take(limit, map( a => a * a, filter(a => a % 2 === 1, list)))`는 모든 연산이 지연되어 아무런 연산도 이루어지지 않는다
- `for...of`문에서 `a`를 처음 뽑을 때 `1`이 들어오고 두번째 뽑을 때 `9`, 마지막으로 `25`가 들어오면서 반복문이 종료된다
- 이때 코드에서 반복문을 빠져나가는 `break`문을 제거했음에도 **시간복잡도가 동일**하다는 사실에 주목해야한다
- `break`는 필요한 만큼만 코드가 반복되도록 제어하여 로직의 효율성을 높이는 키워드
- `take`를 통해 `break`와 같은 제어문마저도 **리스트로 사고**할 수 있음을 확인
- 가능하게 하는 핵심은 **지연 평가**
#### 합산을 `reduce`로 대체
```ts
const sumOfSquaresOfOddNumbers = (limit: number, list: number[]): number => (
	reduce((a, b) => a + b, 0, 
		take(limit, 
			map( a => a * a, 
				filter(a => a % 2 === 1, list)
			)
		)
	)
)
```
- `reduce`는 지연된 리스트를 평가하면서 요소를 뽑아낸다
#### 체이닝으로 변경
```ts
class FxIterable<A> {
	constructor (private iterable: Iterable<A>) {}
	
	// ... 메서드 생략 ...
	
	take(limit: number): FxIterable<A> {
		return fx(take(limit, this)); // new FxIterable(take(limit, this));
	}
}

const sumOfSquaresOfOddNumbers = (limit: number, list: number[]): number =>
	fx(list) // [1, 2, 3, 4, 5, 6, 7, 8, 9]
		.filter(a => a % 2 ===1) // [(1), (3), (5), (7), (9)]
		.map(a => a * a) // [(1), (9), (25) (49), (81)]
		.take(limit) // [(1), (9), (25)]
		.reduce((a, b) => a + b, 0) // add(add(1, 9), 25)
		
console.log(sumOfSquaresOfOddNumbers(3, [1, 2, 3, 4, 5, 6, 7, 8, 9])) // 35
```
- 선언적인 함수명을 통해 각 코드 부분의 목적을 쉽게 파악할 수 있고 위에서부터 아래로 읽으면서 어떤 일이 이루어지는지 파악하기 좋아졌음
#### `sumOfSquaresOfOddNumbers`가 하는 일 목록
- 명령형 `sumOfSquaresOfOddNumbers`
	- 순회
		- `for (const a of list)`를 통해 list 배열의 각 요소를 순회, `a`는 배열의 현재 요소
	- 홀수 검사
		- `if (a % 2 === 1)`조건문을 사용하여 `a`가 홀수인지 검사
		- 홀수인 경우에만 다음 단계를 실행
	- 제곱 계산
		- `const b = a * a`를 통해 홀수 `a`의 제곱을 계산하여 `b`에 저장
	- 누적 합계 갱신
		- `acc += b`를 통해 누적 합계에 `b`를 더함
	- 길이 검사 및 종료
		- `if (--limit === 0) break;` 조건문을 사용하여 `limit`을 감소시키고 `limit`가 0이 되면 반복문을 종료
	- 결과 반환
		- `return acc;`를 통해 최종 누적 합계를 반환
- 함수형 `sumOfSquaresOfOddNumbers`
	- 순회
		- `fx(list`로 순회할 지연된 리스트를 생성
	- 홀수 검사
		- `filter(a => a % 2 === 1)`로 홀수만 남길 지연된 리스트를 생성
	- 제곱 계산
		- `map(a => a * a)`로 필터가 적용된 이터레이터 요소에 제곱을 추가로 적용한 지연된 리스트를 생성
	- 길이 검사 및 종료
		- `take(limit)`로 `limit`만큼만 순회할 지연된 리스트를 생성
	- 누적 합계 갱신
		- `reduce((a, b) => a + b, 0)`으로 모든 요소를 더함
	- 결과 반환
		- `=> ((()))`로 중첩된 리스트를 평가하여 누적 합계를 반환
- **리스트 프로세싱**은 명령형 코드 라인들을 리스트로 변환한다
- 코드를 값으로 다루고 함수를 값으로 다루어 작은 코드의 목록으로 복잡한 문제를 해결해나간다
- 이것이 함수형 프로그래밍과 리스트 프로세싱의 방법
- 이 접근 방식은 코드의 각 부분을 독립적인 리스트 요소로 취급함으로써 복잡한 로직을 세분화하여 정복하는 방법
- 결과적으로 리스트 프로세싱으로 구현된 코드는 더 읽기 쉽고 유지보수하기 쉬우며 각 부분의 역할이 명확해짐
### 현대 언어에서 리스트 프로세싱 - 클로저, 코틀린, 스위프트, 스칼라, C#, 자바

#### `sumOfSquaresOfOddNumbers`를 다른 언어로 구현하기
```clojure
(defn square [x]
	(* x x))

(defn sumOfSquaresOfOddNumbers [limit list]
	(->> list
		(filter odd?)
		(map square)
		(take limit)
		(reduce +)))

(println (sumOfSquaresOfOddNumbers 3 [1 2 3 4 5 6 7 8 9]))
;35
```
- **클로저**
	- 함수형 프로그래밍 패러다임에 중점을 둔 언어
	- `->>`는 파이브라인으로 코드를 표현할 수 있게하는 매크로
	- `->>`는 나중에 리스트를 받을 준비가 된 시퀀스인 `filter(odd?)`, `(map square)`등의 코드를 받아 앞에서 부터 `list`에 적용하고 그 결과를 연속적으로 함수들에 적용
	- LISP답게 `+`기호를 `reduce`의 누적 함수로 사용하도록 인자로 전달할 수 있다
```java
fun sumOfSquaresOfOddNumbers(limit: Int, list: List<Int>): Int {
	return list.asSequence()
		.filter { it % 2 == 1 }
		.map { it * it }
		.take(limit)
		.fold(0) { a, b -> a + b }
}

fun main() {
	val result = sumOfSquaresOfOddNumbers(3, listOf(1, 2, 3, 4, 5, 6, 7, 8, 9))
	println(result) // 35
}
```
- **코틀린**
	- `Iterable` 인터페이스를 통해 이터레이션을 지원
	- `asSequence()`를 사용해 지연 연산을 시작
	- 코틀린의 표준 라이브러리는 높은 수준의 함수형 프로그래밍을 지원
	- `filter`, `map`, `take`, `reduce`, `fold`와 같은 고차 함수를 제공
	- 간결하고 독특한 람다식을 지원
	- `it` 키워드를 사용해 현재 요소를 나타낼 수 있음
	- 강력한 타입 시스템과 클래스를 함꼐 지원하는 멀티패러다임 언어
	- 함수형으로 가독성이 높고 간결한 문법을 제공
```swift
func sumOfSquaresOfOddNumbers(limit: Int, list: [Int]) -> Int {
	return list.lazy
		.filter { $0 % 2 == 1 }
		.map { $0 * $0 }
		.prefix(limit) // take와 동일한 함수
		.reduce(0,+) // 스위프트의 reduce는 첫 번째 인자가 초깃값이며 생략할 수 없음
}

print(sumOfSquaresOfOddNumbers(limit: 3, list: [1, 2, 3, 4, 5, 6, 7, 8, 9]))
// 35
```
- **스위프트**
	- `lazy` 키워드를 사용하여 지연 연산을 시작
	- `시퀀스 프로토콜`과 결합하면 고성능의 지연 평가를 구현할 수 있음
	- 스위프트 표준 라이브러리는 `filter`, `map`, `prefix`, `reduce`와 같은 고차 함수를 제공하며 타입 추론이 강력하여 코드를 간결하게 작성할 수 있다
	- 컴파일 타임에 많은 최적화를 수행하여 지연 평가와 같은 고차 함수를 사용할 때도 높은 성능을 유지
	- `reduce`함수의 누적 함수로 `+`연산자를 사용할 수 있는 점도 매력적
	- 스위프트 역시 함수형 프로그래밍 패러다임을 강력히 지원하면서도 명령형 및 객체지향 프로그래밍 패러다임과도 결합할 수 있다
```scala
object Main extends App {
	def sumOfSquaresOfOddNumbers(limit: Int, list: List[Int]): Int = {
		list.to(LazyList)
			.filter(_ % 2 == 1)
			.map(a => a * a)
			.take(limit)
			.foldLeft(0)(_ + _)
	}

	println(sumOfSquaresOfOddNumbers(3, List(1, 2, 3, 4, 5, 6, 7, 8, 9))) // 35
}
```
- **스칼라**
	- 함수형 프로그래밍과 객체지향 프로그래밍을 통합한 멀티패러다임 언어로 높은 수준의 함수형 프로그래밍 기능을 제공
	- `LazyList`를 통해 지연 평가 방식을 지원
	- 람다식에서는 `(_)`를 사용해 현재 요소를 간단히 나타낼 수 있으며 `a => a * a`와 같은 명시적인 람다식을 사용할 수도 있다
	- `(_ + _)`와 같은 간결한 문법도 지원
		- 컴파일러가 함수의 인자 개수를 명확히 알 때 해당 개수를 추론하여 익명 함수를 생성하기 때문
```c#
using System;
using System.Collections.Generic;
using System.Linq;

public class LispTest
{
	public static void Main()
	{
		List<int> list = new List<int> { 1, 2, 3, 4, 5, 6, 7, 8, 9 };
		int result = SumOfSquaresOfOddNumbers(3, list);
		Console.WriteLine(result); // 35
	}



	static int SumOfSquaresOfOddNumbers(int limit, List<int> list)
	{
		return list.Where(a => a % 2 == 1)
			.Select(a => a * a)
			.Take(limit)
			.Aggregate(0, (a, b) => a + b);
	}
}
```
- **C#**
	- `LINQ(language intergrated query)`기능을 통해 높은 수준의 함수형 프로그래밍을 지원
	- `Where`, `Select`, `Take`, `Aggregate`와 같은 고차 함수들을 사용하여 간결하고 가독성이 높은 코드를 작성할 수 있다
		```c#
		static int SumOfSquaresOfOddNumbers(int limit, List<int> list)
		{
			var query = from num in list
						where num % 2 == 1
						select num * num;
				
			return query.Take(limit).Aggregate(0, (acc, a) => acc + a);
		}
		```
	- `LINQ`는 일부 기능을 `SQL`과 유사한 구문으로 작성할 수 있도록 지원
	- `SQL`과 유사한 `from`, `where`, `select` 등의 키워드를 사용하여 프로그램에서 괄호나 기호 없이도 `SQL`과 유사한 표현식을 작성할 수 있는 독특한 매력을 선사
```java
import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;

public class Main {
	public static void main(String[] args) {
		List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9);
		int result = sumOfSquaresOfOddNumbers(3, list);
		System.out.println(result); // 35
	}
	
	public static int sumOfSquaresOfOddNumbers(int limit, List<Integer> list) {
		return list.stream()
			.filter(a -> a % 2 == 1)
			.map(a -> a * a).limit(limit) // take와 동일
			.reduce(0, Integer::sum);
	}
}
```
- **Java**
	- `Stream API`를 통해 함수형 프로그래밍을 지원
	- `filter`, `map`, `limit`, `reduce`와 같은 스트림 메서드를 사용하여 컬렉션을 변환하고 처리할 수 있다
	- 자바의 람다식은 간결하고 표현력이 뛰어나 `Stream API`와 결합하여 복잡한 데이터를 손쉽게 조작할 수 있다
	- `Stream.reduce` 메서드는 초깃값을 요구하는 형태와 초깃값 없이 `Optional`을 반환하는 형태, 두 가지로 제공
	- 초깃값을 생략하면 `Optional`을 반환하는 형태이므로 실제 값을 얻으려면 `Optional`의 값을 언래핑하는 추가 작업이 필요
### 언어를 넘어 적용 가능한 개념, 패러다임 
- 현대 언어들은 함수형 패러다임을 적극적으로 적용하고 고도화하고 있다
- 일부 언어는 컴파일 단계에서 함수형 코드를 최적화
- 자바스크립트의 경우
	- `Iterable`, `Iterator`, `Generator`, `AsyncGenerator`, `AsyncIterator` 등 다양한 프로토콜을 활용하여 언어와 상호작용할 수 있는 고수준의 함수형 라이브러리를 구현할 수 있다
	- `ECMAScript Stage 3`에 있는 `Iterator Helpers` 사양은 이 책에서 구현한 형태와 거의 동일한 스펙을 지향하고 있으며, 이를 통해 지연 평가가 가능한 헬퍼 함수가 언어 내장 기능으로 제공하게 될 것으로 기대
	- 결국 객체지향, 명령형, 함수형 패러다임을 결합한 멀티패러다임적 사고와 문제 해결 능력은 특정 언어에 국한되지 않는다
	- 강력한 타입 시스템과 타입 추론을 지원하는 이들 언어에서는 클래스와 인터페이스, 이터레이션 프로토콜, 함수형 고차 함수를 모두 활용할 수 있다
## 하스켈로부터 배우기
- **하스켈**
	- 순수 함수형 프로그래밍 언어로 평가되며 함수형 패러다임을 잘 반영하도록 설계된 문법을 가지고 있다
	- 순수 함수와 함수 합성을 강조
	- 커링이 기본
	- 지연 평가를 지원
	- 부수 효과를 특별하게 관리
	- 강력한 타입 시스템
	- 타입 추론
	- 대수적 데이터 타입
	- 높은 다형성을 지원하는 타입 클래스
	- 함수형 프로그래밍에 특화되고 차별화된 많은 기능을 제공
### 하스켈의 함수와 함수 시그니처
```haskell
square :: Int -> Int
square x = x * x
```
- `::`는 타입 선언을 나타냄
- `square`는 함수명
- `x`는 매개변수
- `=`는 함수 정의와 오른쪽 식의 결과 반환
```ts
function square(x: number): number {
	return x * x;
}

type Square = (x: number) => number;
const square: Square = x => x * x;
```
- 타입스크립트로 표현하면 위와 같다
- 하스켈과 타입스크립트 모두 함수 시그니처를 명시함으로써 함수의 입출력 타입을 분명히 할 수 있다
- 함수형 프로그래밍을 더 효과적으로 활용하는데 큰 도움
- 다양한 언어에서 함수 시그니처를 익숙하게 다루는 것은 함수형 패러다임을 이해하고 응용하는 데 유용
### 언어 차원에서 지원하는 커링
- 하스켈은 언어 차원에서 커링을 지원
- 여러 인자를 받는 함수를 자연스럽케 커링된 형태로 다룰 수 있다
- **커링**이란 여러 인자를 받는 함수를 인자 하나씩 받는 함수들의 연쇄로 표현하는 기법을 의미
```haskell
add :: Int -> Int -> Int
add x y = x + y
```
- 시그니처 `add :: Int -> Int -> Int`는 `add`가 두 개의 `Int`를 받아 하나의 `Int`를 반환한다는 뜻
- 하지만 하스켈에서는 이 함수를 기본적으로 커링된 형태로 사용할 수 있다
```haskell
addFive :: Int -> Int
addFive = add 5
```
- `addFive`는 `add`함수에 5를 부분 적용한 결과
- 이로써 `addFive`는 `Int -> Int` 타입의 함수이며 새로운 인자 하나를 받으면 그에 따른 결과를 반환하는 함수가 됨
```haskell
main :: IO ()
main = do
	print (addFive 10) -- 출력: 15
	print (add 3 7) -- 출력: 10
	print (3 `add` 7) -- 출력: 10
```
- `addFive 10`은 `add` 함수에 5를 먼저 부분 적용하여 `(add 5)` 형태의 함수를 만든 뒤 여기에 10을 전달해 15를 결과로 얻는다
- 이어서 `add` 함수에 두 인자를 직접 전달한 `(add 3 7)`의 결과도 10을 출력
- 여기서 `(add 3)`은 `add`에 3을 부분 적용한 함수이며 이 함수에 7을 전달하면 최종적으로 10이 계산
- 하스켈에서는 함수 호출을 ***중위 연산자***로 표현할 수 있어 ``(3 `add` 7)`` 역시 `(add 3 7)`과 동일한 결과
- 이때 `(add 3 7)`은 ***전위***형태의 함수 호출
- 하스켈은 모든 함수 호출에 커링을 기본 적용한다
- 예를 들어 `add :: Int -> Int -> Int`는 사실상 `add :: Int -> (Int -> Int)`와 동일한 의미
- 즉 `add`는 `Int` 하나를 받아 `(Int -> Int)` 형태의 새로운 함수를 반환
- 일반적인 언어에서 이와 동일한 패턴을 표현하려면 함수 오버로드나 추가로 함수 타입 정의가 필요
### `main` 함수와 `IO`
- 하스켈에서는 모든 프로그램이 `main` 함수로 시작
- `main` 함수는 `IO` 타입을 반환해야한다
- `IO` 타입은 입출력 작업을 수행할 수 있도록 설계된 특별한 타입
- 위 예제를 살펴보면 아래와 같다
	- `do` 구문을 사용하면 여러 개의 `IO 액션`을 순차적으로 실행
	- `do` 블록 안의 각 줄은 하나의 `IO 액션`이며 위에서 아래로 순서대로 실행
- `IO`는 하스켈에서 입출력 작업을 나타내는 타입
- 순수 함수형 언어인 하스켈에서는 입출력과 같은 부수 효과를 관리하기 위해 `IO` 타입을 사용
- `IO` 타입을 사용하면 순수 함수형 프로그래밍의 이점을 유지하면서 입출력 작업을 수행할 수 있다
#### `IO`와 부수 효과 관리
- 하스켈은 순수 함수형 언어
- 모든 함수가 같은 인자에 대해 항상 동일한 결과를 내놓아야하는 순수성을 지향
- 현실 세계에서 실제 프로그램은 사용자 입력, 파일 IO, 네트워크 통신 등 부수 효과를 반드시 수행해야 한다
- 하스켈에서는 이 문제를 부수 효과가 있는 함수는 `IO` 타입을 통해 격리하는 식으로 해결
- 하스켈에서 어떤 함수가 `IO`를 반환한다면 이는 해당 함수가 내부적으로 입출력 등의 부수 효과를 일으킬 수 있음을 타입 차원에서 명시하는 것
- 이로써 *순수 함수(a ->b)* 와  *IO 함수(a -> IO b)* 를 명확히 구분할 수 있고 부수 효과로 인한 예측 불가능성을 최소화 할 수 있다
- 하스켈을 사용하는 개발자는 `main :: IO ()`라는 선언을 통해 *프로그램의 최종 결과는 `IO`가 될 것이다* 라고 언어에게 알려준다
- 결과적으로 `main` 내에서는 되도록 여러 순수 함수를 조합해 로직을 구성하되 마지막 결론으로는 입출력과 같은 효과를 수행할 수 있는 `IO 컨택스트`가 되도록 한다
- 이렇게 하스켈은 *부수 효과는 `IO` 안에서만 허용된다* 는 합의를 통해 순수성을 지킨다
- `IO`는 하스켈에서 *이 함수는 입출력, 상태 변경 등 순수하지 않은 일을 할 수 있다* 는 것을 선언하는 타입
- 이를 통해 순수 함수와 부수 효과 함수를 엄격히 구분하고 프로그램의 예측 가능성과 안전성을 높인다
#### `Unit` 타입 `()`와 타입스크립트의 `void`
- 하스켈에서 `()`는 유일한 값을 갖는 `Unit` 타입으로 *의미 없는 값* 을 나타내며 함수가 유의미한 결과를 반환하지 않을 때 사용
- 즉 `()`는 함수가 별다른 값을 제공하지 않고 단순히 부수 효과를 발생시키는 상황을 명확히 나타냄
- 타입스크립트에서는 함수의 반환 타입으로 `void`를 사용하여 비슷한 의도를 드러낼 수 있음
- `void`로 선언된 반환 타입은 함수가 특정 값을 반환하지 않음을 의미하며 대부분 부수 효과만을 발생시키는 함수를 구분하는 데 활용
- 비록 하스켈의 `()`와 타입스크립트의 `void`는 구현 방식이나 정적 분석 수준에서 차이가 있지만 둘 다 *의미 있는 결과를 반환하지 않는 함수* 를 표현할 때 사용된다는 점에서 개념적으로 유사한 역할
- 이를 통해 개발자는 함수의 반환 타입만 보고도 해당 함수가 순수하게 계산 결과를 반환하는지 아니면 외부 상태를 변환시키는 등의 부수 효과를 일으키는지 쉽게 파악할 수 있다
### `head`, `map`, `filter`, `foldl` 함수 시그니처
#### `head` 함수 시그니처
- `head` 함수는 리스트의 첫 번째 요소를 반환
- `a`는 제네릭 타입으로 어떤 타입이든 받아들일 수 있음을 의미
```haskell
head :: [a] -> a
```
#### `map` 함수 시그니처
- `map` 함수는 리스트 `[a]`의 각 요소에 주어진 함수를 적용하여 새로운 리스트 `[b]`를 반환
- `(a -> b)`는 `a` 타입의 값을 `b` 타입의 값으로 변환하는 함수의 타입
```haskell
map :: (a -> b) -> [a] -> [b]
```
#### `filter` 함수 시그니처
- 리스트의 각 요소에 대해 주어진 조건을 검사하여 조건을 만족하는 요소만을 포함하는 새로운 리스트를 반환
- `(a -> Bool)`은 `a` 타입의 값을 받아 `Bool` 값을 반환하는 함수의 타입
```haskell
filter :: (a -> Bool) -> [a] -> [b]
```
#### `foldl` 함수 시그니처
- 다른 언어의 `reduce` 함수와 유사하게 리스트의 요소를 왼쪽에서 오른쪽으로 순회하면서 하나의 값으로 누적
```haskell
foldl :: (b -> a-> b) -> b -> [a] -> b
```
- 여기서 `(b -> a-> b) `는 *누적 함수* 의 타입 시그니처를 의미
- 이 함수는 *현재 누적값(b)* 와 *리스트의 현재 요소(a)* 를 받아 *새로운 누적값(b)* 를 반환하는 형태
- 두번째 인자 `b`는 초기 누적값이며 세번째 인자 `[a]`는 처리할 리스트
- 하스켈에서는 제네릭 타입 변수로 `a`, `b`와 같은 단일 문자 이름을 주로 사용
- 이러한 단순하고 일관된 표기법 덕분에 `foldl`, `map`과 같은 고차 함수를 매우 간결하게 표현할 수 있다
### 함수 합성 - `.` 연산자와 `$` 연산자
- `.` 연산자는 함수를 합성하는 데 사용
- `$` 연산자는 함수 적용을 위한 연산자로서 우선순위를 조정하고 인자를 전달하여 함수를 즉시 평가
```haskell
f :: Int -> Int
f x = x + 1

g :: Int -> Int
g x = x * 2

h :: Int -> Int
h x = x - 3

main :: IO ()
main = do
	let result = f . g . h $ 5
	print result -- 5
```
- 이 예제의 실행을 자바스크립트로 표현하면 `f(g(h(5)))`와 같다
### `sumOfSquaresOfOddNumbers` 함수
```haskell
square :: Int -> Int
square x = x * x

sumOfSquaresOfOddNumbers :: Int -> [Int] -> Int
sumOfSquaresOfOddNumbers limit list =
	foldl (+) 0 . take limit . map square . filter odd $ list
	
main :: IO ()
main = print (sumOfSquaresOfOddNumbers 3 [1, 2, 3, 4, 5, 6, 7, 8, 9])
-- 35
```
- 이 코드는 오른쪽에서 왼쪽으로 읽으면 이해할 수 있다
### 파이프라인 스타일 - `&`
- 하스켈에서는 **파이프라인** 스타일로 함수 합성을 표현할 때 함수 합성 연산자 `.` 대신 정방향 함수 적용 연산자 `&`를 사용할 수 있다
- `&` 연산자는 `Data.Function` 모듈에서 불러올 수 있다
```haskell
import Data.Function (( & ))

square :: Int -> Int
square x = x * x

sumOfSquaresOfOddNumbers :: Int -> [Int] -> Int
sumOfSquaresOfOddNumbers limit list =
	list 
	& fiilter odd
	& map square
	& take limit
	& foldl (+) 0
		
main :: IO ()
main = print (sumOfSquaresOfOddNumbers 3 [1, 2, 3, 4, 5, 6, 7, 8, 9])
-- 35
```
### `Either`를 통한 에러 처리
- 하스켈은 순수 함수형 언어로서 예외를 전통적인 방식으로 처리하기보다는 타입을 통해 에러 상황을 명시적으로 표현하는 방식을 선호
- `Either` 타입은 성공(Right)과 실패(Left)를 구분하여 함수의 결과를 명확히 표현함으로써 컴파일 타임에 에러 처리가 필요함을 인지시킨다
- 이러한 접근 방식은 런타임 예외 발생을 줄이고 코드의 안정성과 가독성을 높이는 데 큰 도움이 된다
#### `(div 10 0)` - 예외 발생
- 기본 라이브러리 `div` 함수는 0으로 나누려 할 때 예외를 발생시킨다
```haskell
main :: IO ()
main do 
	print (div 10 2) -- 5
	print (div 10 0) -- divide by zero
```
#### 안전한 나눗셈
- `Either`는 성공적인 연산 결과를 Right에 에러 상황을 Left에 담아 반환
```haskell
safeDiv :: Int -> Int -> Either String Int
safeDiv _ 0 = Left "0으로 나눌 수 없습니다."
safeDiv x y = Right (div x y)
```
- 위와 같이 런타임 예외 발생 대신 명시적으로 에러 상황을 표현할 수 있다
### 패턴 매칭
- 위의 `safeDiv` 함수는 하스켈의 패턴 매칭 문법을 사용하여 인자 패턴에 따라 함수 실행을 분기한다
- `_`는 와일드카드 패턴으로 어떤 값이든 상관없음을 의미
- `0`은 두번째 인자가 0일때를 나타낸다
- 이 패턴 매칭은 두 번째 인자가 0일 경우 *Left "0으로 나눌 수 없습니다."* 를 반환
- 이 패턴 매칭에 해당하지 않는 경우 3행의 `safeDiv x y = Right (div x y)`가 실행
- 두 번째 인자가 0이 아닌 경우에 정상적인 나눗셈 결과를 Right에 감싸서 반환
- 이처럼 하스켈의 패턴 매칭 문법은 간결하고 직관적인 코드를 작성하도록 돕는다
- 타입스크립트와 비교하면 함수 오버로드, if문, 타입 가드, 타입 좁히기, 매개변수 구조 분해 등의 역할을 모두 패턴 매칭 한 번으로 해결할 수 있다.
- 하스켈은 문장이 아닌 표현식으로 프로그램을 구성하는 철학을 패턴 매칭을 통해 효과적으로 구현하고 있다
```haskell
main :: IO ()
main = do
	print (safeDiv 10 2) -- Right 5
	print (safeDive 10 0) -- Left "0으로 나눌 수 없습니다."
```
```haskell
processResult :: Either String Int -> String
processResult (Left errMsg) = "에러 : " ++ errMsg
processResult (Right value) = "결과 : " ++ show value

main :: IO ()
main = do
	let result1 = safeDiv 10 2
	let result2 = safeDiv 10 0
	putStrLn (processResult result1) -- 결과: 5
	putStrLn (processResult result2) -- 에러: 0으로 나눌 수 없습니다.
```
- `(Left errMsg)`와 `(Right value)`는 패턴 매칭을 통해 `Either` 타입의 내부 값을 추출한다
- 이를 타입스크립트에 비유하면 매개변수 구조 분해를 통해 객체의 내부 속성을 꺼내는 것과 유사한 개념이라 할 수 있다
- 이처럼 `Either` 타입을 사용하면 하스켈에서 함수의 성공 또는 실패 상태를 명시적으로 구분할 수 있어 에러를 런타임 예외 대신 타입을 통해 안전하게 처리할 수 있다
- 또한 하스켈에서는 `Maybe`라는 타입으로 값이 없을 수도 있는 상황을 안전하게 처리할 수 있다