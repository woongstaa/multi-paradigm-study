## 리스트 프로세싱 패턴화
### 변형-누적 패턴
- 리스트 프로세싱에서 가장 널리 사용되는 패턴
- 초기 이터러블을 `map`으로 변형한 뒤 `reduce`로 누적하여 최종 결과를 도출하는 방식
- 프로그램의 결과물이 단일 값일 때. ㅜ로 사용
- 배열이 아닌 객체, 숫자, 문자열 등으로 데이터를 변환하는데 적합
- 데이터 집계나 변환, 누적 작업에 매우 유용하며 다양한 문제를 단순하고 선언적으로 해결할 수 있다
#### 상품의 총수량
```ts
const totalQuantity = (products) => 
	products
		.map((product) => product.quantity)
		.reduce((a, b) => a + b, 0)
```
- `reduce`의 보조 함수는 데이터를 어떻게 누적할지를 정의
- 이 함수는 단순히 숫자를 더하는 데 그치지 않고 문자열, 객체, 배열, 심지어는 커스텀 데이터 타입을 포함한 모든 구조를 결합하거나 변형할 수 있는 강력한 도구
#### Query String을 객체로 변환하기
```ts
const queryString = "name=Marty%20Yoo&age=41&city=Seoul";

const queryObject = queryString
	.split("&")
	.map((param) => param.split("="))
	.map(entry => entry.map(decodeURIComponent))
	.map(([key, value]) => ({ [key]: value }))
	.reduce((a, b) => Object.assign(a, b), {})
```
#### 객체를 Query String으로 변환하기
```ts
const params = { name: "Marty Yoo", age: "41", city: "Seoul" };

const queryString =
	Object.entries(params)
		.map(entry => entry.map(encodeURIComponent))
		.map(([key, value]) => `${key}=${value}`)
		.reduce((a, b) => `${a}&${b}`)
		
const queryString2 =
	Object.entries(params)
		.map(entry => entry.map(encodeURIComponent))
		.map((entry) => entry.join("="))
		.join('&');
		
const queryString3 = pipe(
	Object.entries(params),
	map(map(encodeURIComponent)),
	map(join('=')),
	join('&'),
)
```
- 여러 번의 API 요청 후 `Promise`가 담긴 이터러블을 배열로 변환했던 코드 또한 변형-누적 패턴
- `Object.fromEntries`나 `Array.fromAsync`와 같은 헬퍼 함수 역시 본질적으로 `reduce`로 구현할 수. ㅣㅆ는 동작을 추상화한 함수이기 때문
- 변형-누적 패턴은 데이터를 변형한 후 누적하여 최종결과를 생성하는 가장 기본적이면서도 강력한 리스트 프로세싱 패턴
- 단순한 숫자 합산에서부터 문자열 직렬화, 객체 변환 등 다양한 문제를 효과적으로 해결할 수 있는 유연한 접근 방식을 제공
### 중첩-변형 패턴
- 리스트 프로세싱에서 중첩된 데이터 구조를 처리하거나 데이터의 계층을 따라 각 수준에서 변형을 수행할 때 사용하는 패턴
- 이 패턴은 `map(map(f))`처럼 `map` 안에서 다시 `map`을 호출하여 외부 컬렉션의 각 요소를 처리하면서 내부적으로 또 다른 변형을 수행하는 경우에 적합
- 특히 트리 구조나 2차원 배열 등 계층적인 데이터를 변형하거나 처리할 때 매우 유용
#### 트리 구조 변형
- 중첩-변형 패턴은 트리 구조의 데이터를 순회하며 각 노드와 하위 노드를 변형할 때도 유용
```ts
const tree = [
	{ id: 1, children: [{ id: 2}, { id: 3 }] },
	{ id: 4, children: [{ id: 5 }] },
];

const transformedTree = tree.map(({ id, children }) =>
	({
		name: `parent-${id}`,
		children: children.map((child) => ({ name: `child-${child.id}` }))
	})
)
```
#### 달력 그리기(2차원 배열 join)
```ts
import { pipe, flat, range, chunk, toArray, map, join } from '@fxts/core';

const getMonthEndDates = (monthEnd: Date) =>
	monthEnd.getDay() === 6
		? []
		: range(
			monthEnd.getDate() - monthEnd.getDay(),
			monthEnd.getDate() + 1,
		)
		
const generateCalendar = (prevMonthEnd: Date, currentMonthEnd: Date) =>
	pipe(
		flat([
			getMonthEndDates(prevMonthEnd), // range(29, 31) 9월 29일 - 30일
			range(1, currentMonthEnd.getDate() + 1), // range(1, 32) 10월 1 - 31일
			range(1, 6 - currentMonthEnd.getDaty() + 1) // range(1, 3) 11월 1- 2일
		]),
		chunk(7),
		toArray,
	)

const formatCalendar = (calendarWeeks: number[][]) => 
	pipe(
		calendarWeeks,
		map(map(day) => (day < 10 ? ` ${day}`: `${day}`)),
		map(join(" ")),
		join('\n"),
	)
	
const renderCalendar = (year: number, month: number) =>
	pipe(
		generateCalendar(
			new Date(year, month - 1, 0), // 지난달 마지막 날
			new Date(year, month, 0) // 이번 달 마지막 날
		),
		formatCalendar,
		console.log
	)
```
1. `renderCalendar` 함수
	- **자바스크립트의 new Date 특성**
		- 자바스크립트에서 `new Date(year, month, day)`의 month는 0(January)부터 11(December)까지의 값을 갖음
		- month 값에 1을 더한 후 날짜를 0으로 설정하면 해당 월의 마지막 날을 반환
	- **달력 생성과 변환**
		- `generateCalendar`는 지난달, 이번 달, 다음 달의 필요한 날짜 데이터를 생성하여 주 단위로 나눈다
		- `formatCalendar`는 숫자로 이루어진 2차원 배열을 문자열로 변환하여 가독성을 높임
	- **합성과 재사용성**
		- `renderCalendar`는 `generateCalendar`와 `formatCalendar`를 합성하여 최종적으로 달력을 생성하고 출력
		- `formatCalendar`만 다른 함수로 변경하면 HTML이나 CSV 등 다양한 출력 형태로 확장할 수 있다
2. `generateCalendar` 함수
	- 특정 연도와 월의 달력을 생성하여 주 단위(2차원 배열)로 반환
	- `getDate`는 `Date` 객체의 날짜(일)를 반환하고 `getDay`는 `Date` 객체의 요일을 반환
	- `getDay`의 반환값은 0 - 6까지의 숫자
	- 지난달 날짜는 `getMonthEndDates`로 계산
	- `range`를 사용해 지난달의 마지막 며칠을 포함
	- `monthEnd.getDate()`는 30(9월의 마지막 날)이고 `monthEnd.getDay()`는 1(9월 30일은 월요일)이므로 시작값은 30 - 1 = 29, 종룟값은 30 + 1 = 31
	- 따라서 `range(29, 31)`의 결과로 `[29, 30]`이 출력
	- 만약 `monthEnd.getDay()`가 토요일(6) 이면 빈 배열을 전달하여 달력에 출력되지 않도록
	- 이번 달 날짜를 계산할 때는 `range`를 사용해 이번 달의 1일부터 마지막 날까지 생성
	- `currentMonthEnd.getDate()`는 31이므로 시작값은 1 종룟값은 31 + 1 = 32  `range(1, 32)`
	- 다음 달 날짜는 `range`를 사용해 이번 달 마지막 요일 이후 남은 날짜를 채움
	- `currentMonthEnd.getDay()`는 4(10월 31일은 목요일)이고 필요한 날짜 수는 6 - 4 = 2개 이므로 `range(1, 2 + 1)`
	- 만약 이번 달 마지막 날이 토요일이면 6 - 6 + 1 = 1이 되고 `range(1, 1)`은 빈 배열이므로 출력되지 않음
	- 앞서 구한 `range` 3개의 결과
		- `[29, 30]`
		- `[1, 2 ... 31]`
		- `[1, 2]`
	- 이 데이터는 주 단위로 나뉘어 있지 않으므로 `flat()`을 사용해 1차원 이터레이터로 평탄화한 후 `chunk(7)`을 사용해 데이터를 다시 7일 단위로 나누어 주 단위 배열을 생성
3. `formatCalendar` 함수
	- 생성된 주 단위의 배열을 가독성 높은 문자열로 변환하여 출력
	- 날짜 포매팅은 ``map(map(day => day < 10 ? ` ${day}` : `${day}`))``를 사용해 한 자리 숫자 앞에 공백을 추가
	- `map(join(' '))`으로 각 주 단위를 공백으로 연결된 문자열로 변환하고 `join('\n')`으로 각 주 단위를 줄바꿈으로 연결해 최종 달력을 만듬

- 지금까지 2차원 배열을 처리하여 각 레벨에서 변형과 누적을 수행하는 패턴을 확인했다
- 여기서 사용된 핵심 패턴을 정리하면 아래와 같다
##### 평탄화 후 재분할
`chunk(7, flat([range(), range(), range()]))`는 날짜 범위들을 평탄화한 후 7일 단위로 재분할하여 주 단위 데이터를 생성
##### 2차원 데이터 중첩 변형
`map(map(...))`에서는 중첩된 데이터를 처리하며 각 단계를 변형
##### 레벨 간 데이터 누적
`map(join(' '))`으로 내부 배열을 공백으로 연결된 문자열로 변환, `join('\n')`으로 주 단위 문자열을 줄바꿈으로 연결하여 최종 달력 형식을 완성
### 반복자-효과 패턴
- 리스트 프로세싱에서 이터레이터를 만들어둔 후 지연 평가를 통해 데이터를 소비하여 부수적인 효과를 발생시키는 패턴
- 이 패턴은 주로 데이터를 변형하거나 가져온 후 하나씩 소비하면서 특정 작업(e.g. 로깅, 출력, 네티워크 요청 등)을 수행할 때 사용
- 이 패턴의 결과로 최종 데이터는 생성되지 않으며 작업 자체가 목적이 되는 경우에 적합
#### 콘솔 출력으로 로그 남기기
```ts
fx(range(5))
	.map(x => x * 2)
	.forEach(x => console.log(`Processed ${x}`));
```
#### 결제 동기화 스케줄러 코드
```ts
async function main() {
	await fx(range(Infinity))
		.toAsync()
		.forEach(() => Promise.all([
			syncPayments(),
			delay(10000) 
		]));
}

await fx(payments)
	.toAsync()
	.rejeect(p => ordersMapById.has(p.store_order_id))
	.forEach(async p => {
		const { message } = await PgApi.cancelPayment(p.pg_uid);
		console.log(message)
	})
```
#### 부수 효과를 격리하는 `forEach`
- `forEach`는 반환 값이 없는 매서드로 명시적으로 부수효과를 수반하는 동작을 수행하기 위해 설계되었음
- 이는 주어진 콜백 함수를 배열의 각 요소에 대해 호출하지만 호출 결과를 반환하지 않음
- 대신 코드의 의도를 명확히 전달하며 데이터의 변형과 부수 효과를 분리하는데 기여
- 이처럼 부수 효과를 격리하는 설계 방식은 코드의 유지 보수성을 높이는 데 중요한 역할을 함
- 데이터 변형과 부수 효과를 분리하면 특정 코드 블록에서 어떤 변화가 일어나는지 예측 가능해지므로 문제 발생 시 디버깅이 용이해짐
- e.g. 데이터의 순수한 변환 `map`, `filter`, `reduce`와 같은 메서드에서 처리되고 `DOM` 삭제, 파일 저장, 로그 작성, API 호출 등의 부수 효과는 `forEach` 내에서 처리
- 때로는 부수적인 효과를 일으키면서도 실행 결과를 반환해야 하는 경우가 있음
- 이때는 `mapEffect`와 같은 함수명을 사용하여 `map`과 유사하게 동작하지만 부수 효과를 포함한 동작임을 명확히 표현할 수 있다
```ts
await fx(payments)
	.toAsync()
	.reject(p => ordersMapById.has(p.stroe_order_id))
	.mapEffect(p => PgApi.cancelPayment(p.pg_uid))
	.forEach(res => console.log(res.message));
```
- 이렇게 구분하면 코드의 모듈성과 재사용성이 상상되며 함수형 프로그래밍의 중요한 철학인 순수 함수와 부수 효과의 격리를 실현하는 데 도움이 된다. `forEach`나 `mapEffect` 같은 함수는 부수 효과를 의도적으로 허용하면서도 부수 효과가 필요한 구간을 명확히 구분하도록 돕는 유용한 도구
#### 필터-중단 패턴
- 리스트 프로세싱에서 데이터를 조건에 따라 필터링한 후 일부 데이터만 선택하여 소비하는 패턴
- 대규모 데이터에서 특정 조건을 만족하는 일부 데이터만 빠르게 추출해야 할 때 유용
- 데이터를 필요한 만큼만 처리하고 조건을 만족한 시점 이후의 데이터는 처리하지 않아도 되기 때문에 성낭상 효율적
#### `find`, `some`, `every` 함수
```ts
const find = <A>(f: (a: A) => boolean, iterable: Iterable<A>) =>
	pipe(
		iterable,
		filter(f),
		take(1),
		([found]) => found as A | undefined
	);

const some = <A>(f: (a: A) => boolean, iterable: Iterable<A>) =>
	pipe(
		iterable,
		filter(f),
		take(1),
		([...arr] => arr.length === 1),
	);
	
const every = <A>(f: (a: A) => boolean, iterable: Iterable<A>) =>
	pipe(
		iterable,
		reject(f),
		take(1),
		([...arr] => arr.length === 0),
	)
```
### 무한-중단 패턴
- 끝이 없는 데이터 스트림에서 필요한 만큼만 데이터를 추출하기 위한 패턴
- `range`를 사용해 무한히 증가하는 숫자나 특정 규칙을 따르는 데이터를 생성하고 `take`를 이용해 원하는 개수만큼 데이터를 추출
- 이 패턴은 특히 데이터가 정해진 개수보다 더 많을 때 효율적
- 명령형 스타일로 비유하자만 `while-break` 구조와 유사한 역할
#### 콜라츠 추측 풀기
```ts
const nextCollatzValue = (num: nubmer) => 
	num % 2 === 0
		? num / 2
		: num * 3 + 1;

const collatzCount = (num: number) => 
	pipe(
		repeatAppy(nextCollatzValue, num),
		zip(range(1, Infinity)),
		find(([, value]) => value === 1),
		collatz => collatz!,
		head,
	)
```
#### 결제 내역이 있을 때까지 가져오기
```ts
const payments = 
	await fx(range(1, Infinity))
		.toAysnc()
		.map(page => PgApi.getPayments(page))
		.takeUntilInclusive(({ length }) => length < 3)
		.flat()
		.toArray();
```
### 분할-평탄 패턴
- 데이터를 일정 크기로 분할한 뒤 다시 평탄화하여 원하는 형태로 변환하는 리스트 프로세싱 기법
- 데이터의 구성을 조정하거나 대규모 데이터를 일정한 단위로 처리한 뒤 결과를 합칠 때 유용
- e.g. 요청 크기 제한이 있는 API 호출, 페이지 단위로 데이터를 처리할 때
#### API 요청 제한에 적용하기
```ts
const orders = await fx(payments)
	.map(p => p.store_order_id)
	.chunk(5)
	.toAsync()
	.map(StoreDB.getOrders)
	.flat()
	.toArray()
```
### 변형-평탄 패턴
- 데이터를 변형한 뒤 결과를 평탄화하여 하나의 연속된 데이터 흐름으로 만드는 리스트 프로세싱 기법
- 중첩된 데이터를 단일 수준으로 펼치거나 각 요소를 변형하여 새 데이터를 생성하고 이를 하나의 구조로 병합할 때 유용
#### 댓글과 답글 하나로 병합하기
```ts
const comments = [
	{
		id: 1, 
		text: "First comment", 
		replies: [
			{
				id: 11,
				text: "Reply 1-1" 
			}
		]
	},
	{
		id: 2,
		text: "Second Comment",
		replies: [],
	},
	{
		id: 3, 
		text: "Third comment", 
		replies: [
			{
				id: 31,
				text: "Reply 3-1" 
			},
			{
				id: 32,
				text: "Reply 3-2" 
			}
		]
	}
];

fx(comments)
	.map(({ id, text, replies }) => [{ id, text }, ...replies])
	.flat()
	.forEach(console.log);
```
#### 중첩된 데이터 구조에서 평탄화여 합산하기
```ts
const totalHighScorers = teams
	.flatMap(team => team.players)
	.map(player => player.score)
	.reduce((a, b) => a + b, 0);
	
const totalQuantity = products
	.flatMap(prd => prd.options)
	.map(opt => opt.quantity)
	.reduce((a, b) => a + b, 0)
```
### 결합-누적 패턴
- 여러 이터러블을 결합한 후 이를 순회하며 최종 결과를 누적하는 패턴
#### `keys`와 `values`로 객체 만들기
```ts
const keys = ['name', 'job', 'location'];
const values = ['Marty', 'Programmer', 'New York'];

const object =
	fx(zip(keys, values))
		.map(([key, val]) => ({ [key]: val }))
		.reduce((a, b) => Object.assign(a, b), {});
```
#### 리스트에 고유 ID 부여하기
```ts
const items = ['Apple', 'Banana', 'Cherry'];

const itemsWithIds = pipe(
	zip(range(Infinity), items),
	map(([id, item]) => ({ id, item })),
	toArray
);
```
### 해시-매치 패턴
- 리스트 프로세싱에서 데이터를 효율적으로 구성하거나 조회하기 위해 해시 구조를 생성하는 데 사용
- 데이터를 키로 매핑하거나 그룹화, 카운팅, 변환 등의 작업에 활용하며 `indexBy`, `groupBy`와 같은 작업이 이에 해당
- 참고로 `indexBy`, `groupBy` 역시 본질적으로 `reduce`를 활용해 구현
- 특정 데이터에 빠르게 접근하거나 데이터를 제구성해야하 할 때 특히 유용
- 해시 기반 구조를 생성하면 `O(n)` 복잡도로 특정 작업을 처리할 수 있어 성능적인 이점도 제공
#### `posts`와 `users`를 매칭하기
```ts
const users = [
	{ id: 1, name: 'Alice' },
	{ id: 2, name: 'Bob' }
];

const posts = [
	{ id: 1, title: 'FP', user_id: 1 },
	{ id: 2, title: 'OOP', user_id: 2 },
	{ id: 3, title: 'MPP', user_id: 2},
];

const usersById = indexBy(user => user.id, users);

const postsWithUsers = posts.map(post => ({
	...post,
	user: userById[post.user_id],
}))
```
- `indexBy` 함수는 특정 키를 기준으로 이터러블 데이터를 해시 구조로 변환하는데 사용됨
- 주어진 이터러블의 각 요소에 보조 함수를 적용하여 키 값을 추출하고 해당 키를 객체의 속성으로 설정
- 이 과정에서 보조 함수는 반드시 고유한 키 값을 반환하도록 구성해야 함
#### `posts`와 `comments`를 매칭하기
```ts
const comments = [
	{ id: 1, title: 'Great post!', post_id: 1 },
	{ id: 2, title: 'Very informative', post_id: 2 },
	{ id: 3, title: 'Thanks for sharing', post_id: 2},
];

const commentsByPostId = groupBy(comment => comment.post_id, comments);

const postsWithComments = posts.map(post => ({
	...post,
	comments: commentsByPost[post.id] || []
}))
```
### 리스트 프로세싱 함수 유형별 개념 정리
#### 지연 중간 연산
- 결과가 실제로 필요할 때까지 연산을 미루며 이 단계만으로는 최종 결과가 나오지 않음
- e.g. `map`, `filter`, `zip`
#### 단축 중간 연산
- 특정 조건이 충족되면 그 시점에서 더 이상 데이터를 읽지 않아 불필요한 연산을 건너뜀
- e.g. `take`, `takeWhile`, `takeUntilInclusive`
#### 터미널 연산
- 실제 이터러블을 전부 소비하여 최종결과를 만들어 냄
- 한번 연산을 호출하면 지연이 해제되고 실제 순회가 일어남
- e.g. `find`, `every`, `some`, `reduce`
#### 폴드 / 리듀스 연산
- 터미널 연산 중에서도 시퀀스 전체를 하나의 값으로 누적하여 반환하는 연산
- e.g. `reduce`, `groupBy`, `indexBy`, `Promise.all`, `Array.fromAsync`
#### 부수 효과
- 출력 / 로그 / 파일 쓰기 등 외부 상태를 변화시키는 연산, 보통 내부 콜백에서 '무언가를 실행'하고 끝내는 형태
- e.g. `forEach`