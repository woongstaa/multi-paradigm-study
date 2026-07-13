# 멀티패러다임 프로그래밍
## HTML 템플릿 엔진 만들기
### Tagged Templates
- `Tagged Templates`는 템플릿 리터럴을 보다 유연하게 활용할 수 있게 하는 강력한 도구
- 사용자 정의 함수를 통해 템플릿 문자열과 삽입된 값을 처리할 수 있음
- 이를 통해 문자열을 조작하거나 특수한 출력을 생성하는 등의 다양한 작업을 수행할 수 있음
```ts
function upper(strs: TemplateStringArray, ...vals: string[]) {
	return strs[0] 
		+ vals[0].toUpperCase()
		+ strs[1]
		+ vals[1].toUpperCase()
		+ strs[2];
}
```
- `Tagged Templates` 기법을 활용하면 템플릿 리터럴을 분리하여 문자열을 유연하게 처리할 수 있다
- 이 기법은 문자열 조작과 다국어 지원, 보안검사(SQL 인젝션 방지, XSS 방지를 위한 이스케이프 처리)와 같은 다양한 작업에 활용할 수 있다
### 리스트 프로세싱으로 구현하기
```ts
import { pipe, zip, toArray } from '@fxts/core';

function html(strs: TemplateStringsArray, ...vals: string[]) {
	vals.push('')
	
	return pipe(
		zip(strs, vals),
		toArray
	)
}

const a = 'A',
	b = 'B',
	c = 'C';
	
const result = html`<b>${a}</b><i>${b}</i><em>${c}</em>`;

console.log(result)
// [["<b>","A"], ["</b><i>","B"], ["</i><em>","C"], ["</em>",""]]
```
- 이번에는 결합-누적 패턴에 `flat`을 추가하여 튜플을 반환하는 이터레이터를 평탄화한 뒤 `reduce`를 사용해 하나의 문자열로 누적하는 방식을 구현해보자
```ts
import { pipe, zip, flat, reduce } from '@fxts/core';

function html(strs: TemplateStringArray, ...vals: string[]) {
	vals.push('');
	
	return pipe(
		vals,
		zip(strs),
		flat,
		reduce((a, b) => a + b),
	)
}

const a = 'A',
	b = 'B',
	c = 'C';
	
const result = html`<b>${a}</b><i>${b}</i><em>${c}</em>`;
// <b>A</b><i>B</i><em>C</em>
```
### `push`를 `concat`으로
- `push`는 기존 배열을 변경하지만 `concat`은 기존 배열을 변경하지 않고 지연 평가되는 이터레이터를 반환하므로 부수 효과 없이 동일한 결과를 얻을 수 있다
- 또한 필요한 시점에 값을 꺼내면서 추가된 하나의 값에 대해서만 한 번더 순회하기 때문에 시간 복잡도 면에서도 `push`와 사실상 차이가 없다
- 전체 배열을 새로 만들거나 모든 값을 재할당하지 않으므로 메모리 사용 측면에서도 별다른 추가 부하가 없다
```ts
import { pipe, zip, flat, reduce, concat } from '@fxts/core';

const html = (strs: TemplateStringArray, ...vals: string[]) =>
	 pipe(
		concat(vals, ['']),
		zip(strs),
		flat,
		reduce((a, b) => a + b),
	)

const a = 'A',
	b = 'B',
	c = 'C';
	
const result = html`<b>${a}</b><i>${b}</i><em>${c}</em>`;
// <b>A</b><i>B</i><em>C</em>
```
- 코드가 복잡해지고 문장이 많아질수록 `concat` 같은 아이디어를 활용하면 유용하다
- 이번 변경에서 주목해야할 이점은 부수 효과의 감소보다는 표현식만으로 코드를 조합할 수 있게 되었다는 점이다
- `html` 함수를 화살표 함수로 간결하게 화살표 함수로 간결하게 작성할 수 있으며 결과적으로 코드가 점차 함수형 스타일을 띠게 됨
- 이러한 변화는 추후 코드의 재사용성과 확장성을 높이는 데도 도움이 됨
- 표현식만으로 코드를 구성하면 이후 문장에 의한 값 변형이나 참조 가능성이 사라짐
- 그 결과로 예측 가능성과 안정성이 향상되고 특정 표현식의 결과를 고립해 테스트하고 검증하기 쉬워짐
- 이러한 제약 조건은 더욱 신뢰할 수 있는 코드를 만드는데 기여
```ts
import { pipe, zip, flat, reduce, append } from '@fxts/core';

const html = (strs: TemplateStringArray, ...vals: string[]) =>
	 pipe(
		vals,
		append(''),
		zip(strs),
		flat,
		reduce((a, b) => a + b),
	)
```
- `append` 함수는 `concat`과 유사한 기능을 하며 마찬가지로 지연 평가를 지원하므로 필요한 시점에만 요소를 생성
- 또한 `append`라는 직관적인 함수명과 커링을 활용하면 코드가 더 선언적이고 직관적인 형태로 표현되어 함수형 프로그래밍의 특성과 장점을 더욱 살릴 수 있다
### XSS 공격 방지
- `XSS`*Cross Site Scripting*는 웹 페이지에 악성 스크립트를 삽입하여 해당 페이지를 보는 다른 사용자에게 피해를 주는 공격 기법
	- 사용자 입력을 그대로 HTML에 삽입하면 `<script>` 태그 등을 통해 공격자가 임의의 자바시크립트 코드를 실행 할 수 있음
	- 방지하려면 입력된 값 중 HTML 문법으로 해석될 수 있는 문자를 안전한 형태로 변환하는 작업이 필요
```ts
const escapeMap = {
	'&': '&amp;',
	'<': '&alt;',
	'>': '&gt;',
	'"': '&quot;'
	"'": '&#x27';
	"`": '&#x60;',
};

const source = '(?:' + Object.keys(escapeMap).joing('|') + ')';
const testRegexp = RegExp(source);
const replaceRegexp = RegExp(source, 'g');

function excpaeHtml(val: unknown): string {
	const string = `${val}`;
	return testRegexp.test(string)
		? string.replace(replaceRegexp, (match) => escapeMap[match])
		: string;
}

export { escapeHtml };

console.log(escapeHtml('<script>alert("XSS")</script>'))
```
- `vals`의 타입을 `string[]`에서 `unknown[]`으로 변경한 이유는 `escapeHtml`이 `unknown` 타입의 인자를 받아 문자열로 변환하는 기능을 갖췄기 때문

## 멀티패러다임을 활용한 동시성 핸들링
### `executeWithLimit` 다시보기
```ts
async function executeWithLimit<T>(fs: (() => Promise<T>)[], limit: number): Promise<T[]> {
	const results: T[] = [];
	
	for (let i = 0; i < fs.length; i += limit) {
		const batchPromises = [];
		for (let j =0; j < limit && (i + j) < fs.length; j++) {
			batchPromises.push(fs[i + j]());
		}	
		
		const batchResults = await Promise.all(batchPromises);
		results.push(...batchResults);
	}
	
	return results;
}
```
- 위 코드는 전통적인 명령형 방식
- 4.2절에서는 함수형 패러다임을 활용해 재구현하는 과정을 소개하였음
```ts
const executeWithLimit = <T>(fs: (() => Promise<T>)[], limit: number): Promise<T[]> =>
	fx(fs)
		.map(f => f())
		.chunk(limit)
		.map(ps => Promise.all(ps))
		.to(fromAsync)
		.then(arr => arr.flat());
```
- 위 코드에서 사용한 `fx`는 이 책에서 함께 만든 `FxIterable`을 반환
- 그리고 아래 에서 사용한 `fx`는 `FxTs` 라이브러리의 함수이며 이를 이용하면 동일한 로직을 더욱 간결하고 선언적으로 표현할 수 있다
```ts
import { fx } from '@fxts/core';

const executeWithLimit = <T>(fs: (() => Promise<T>)[], limit: number): Promise<T[]> =>
	fx(fs)
		.toAsync()
		.map(f => f())
		.concurrent(limit)
		.toArray();
```
- 앞선 예시들은 모두 의도한 대로 동작하며 동시성 부하를 제어하는 다양한 접근 방식을 잘 보여줌
### 챗GPT가 명령형으로 구현한 동시성 핸들링 함수
- `executeWithLimit` 함수는 비동기 작업을 일정한 크기단위로 나누어 순차적으로 실행하는 단순한 부하 조절 로직을 갖추고 있음
- `runTasksWithPool`은 특정 `poolSize` 값만큼 동시에 작업을 실행하고 하나의 작업이 완료될 때마다 대기 중인 새로운 작업을 실행하는 형태로 동작하도록 설계
```ts title='Chat GPT가 구현한 함수'
async function runTasksWithPool<T>(
	fs: (() => Promise<T>)[],
	poolSize: number
): Promise<T[]> {
	const results: T[] = [];
	const activePromises: Promise<void>[] =[];
	
	for (let i = 0; i < fs.length; i++) {
		const taskFactory = fs[i];
		
		const p = taskFactory()
			.then((fetchedValue) => {
				results[i] = fetchedValue;
			})
			.then(() => {
				const removeIndex = activePromises.indexOf(p);
				if (removeIndex > -1) {
					activePromises.splice(removeIndex, 1);
				}
			})
			
		activePromises.push(p);
		
		if (activePromises.length >= poolSize) {
			await Promise.race(activePromises);
		}
	}
	
	await Promise.all(activePromises);
	
	return results;
}
```
- 이 코드는 다음과 같이 작동
	1. results 배열
		- 각 `fs`가 반환하는 `Promise`의 결과를 인덱스에 맞춰 저장
		- 실행 순서와 상관 없이 원래의 함수 순서대로 결과를 관리할 수 있음
	2. activePromises 배열
		- 현재 실행 중인 `Promise`들을 추적하는 배열
		- 작업 하나가 완료되면 이 배열에서 해당 `Promise`를 제거
	3. 루프 내 작업 실행
		- `for`루프를 사용해 각 작업을 순회
		- 각 작업을 호출하고 결과를 `then` 체인에서 `results[i]`에 할당
		- 이어서 `then`을 추가로 호출하여 작업 완료 시 `activePromises`에서 해당 `Promise`를 제거
	4. 동시 실행 개수 제어
		- 새로운 작업을 `activePromises` 배열에 추가한 뒤 배열 길이가 `poolSize`에 도달하면 `Promise.race(activePromises)`를 통해 가장 먼저 완료되는 작업을 대기
		- 이로써 한 번에 최대 `poolSize` 개수 만큼만 동시 실행
	5. 모든 작업 완료 대기
		- 루프가 끝난 뒤에도 아직 완료되지 않은 작업들이 `activePromises`에 남아 있을 수 있음
		- `Promise.all(activePromise)`를 호출하여 모든 남은 작업이 완료될 때까지 기다린 뒤 최종적으로 `results` 배열을 반환
### 멀티패러다임으로 구현한 동시성 핸들링 함수
```ts
class TaskRunner<T> {
	private _promise: Promise<T> | null = null;
	private _isDone = false;
	get promise() {
		return this._promise ?? this.run();
	} 
	get isDone() {
		return this._isDone;
	}
	
	constructor(private f: () => Promise<T>) {}
	
	async run() {
		if (this._promise) {
			return this._promise;
		} else {
			return this._promise = this.f().then(res => {
				this._isDone = true;
				return res;
			})
		}
	}
}

async function runTasksWithPool<T>(
	fs: (() => Promise<T>)[],
	poolSize: number
): Promise<T[]> {
	const tasks = fs.map(f => new TaskRunner(f));
	
	let pool: TaskRunner<T>[] = [];
	
	for (const nextTask of tasks) {
		pool.push(nextTask);
		
		if (pool.length < poolSize) continue;
		
		await Promise.race(pool.map(task => task.run()));
		pool.splice(pool.findIndex(task => task.isDone), 1);
	}
	
	return Promise.all(task.map(task => task.promise));
}
```
- 여기에는 
	- 객체지향: TaskRunner 클래스
	- 배열 메서드: map, findeIndex, splice
	- 명령형 요소 for...of 루프, await
- 등이 자연스럽게 결합 되어 있음

1. TaskRunner 클래스 도입
	- 각 비동기 작업을 TaskRunner라는 클래스로 감싸서 Promise의 상태와 완료 여부를 명확하기 관리
	- TaskRunner는 promise와 isDone 상태를 가지고 있어 외부에서 작업 진행 상황을 쉽게 파악할 수 있다
	- run() 메서드를 통해 작업 시작 로직을 명확히 분리하였으며 이를 통해 Promise 생성과 완료 상태 갱신을 일관되게 처리할 수 있음
	- 이러한 캡슐화를 통해 상태 관리가 직관적으로 이루어지고 코드 이해도가 높아짐
2. 배열 매서드 활용
	- map(task => task.run())을 통해 객체지향적으로 정의된 TaskRunner를 함수형 스타일로 제어할 수 있게됨
	- findIndex와 splice를 통해 완료된 작업을 배열에서손쉽게 제거하는 명령형 로직을 구현
	- 이처럼 배열 메서드를 적절히 활용하여 최소한의 코드로 풀 관리 로직을 단순하고 명료하게 표현 할 수 있음
3. 명령형 흐름 제어
	- for...of 르프를 사용하여 작업을 순차적으로 pool에 추가하고 await Promise.race(...)로 비동기 이벤트를 명확히 기다리는 명령형 제어 흐름을 갖췄음
	- 이러한 명령형 흐름은 *'언제 다음 작업을 시작하고'* , *'언제 완료된 작업을 제거하는지'* 를 직관적으로 보여줌
- 함수형 스타일과 객체지향 패턴을 적절히 섞되 흐름 제어는 명령형으로 단순화함으로써 가독성과 유지보수성을 높였음
- 때로는 명령형 문법을 적절히 섞는 것이 적합하고 간결한 선택이 될 수 있음
- 특정 부분은 함수형 패러다임이 적합하고 어떤 부분은 명령형 코드가 더 단순할 때, 두 패러다임을 과감히 섞거나 빼는 태도가 유지보수성과 확장성에 도움을 줄 수 있음
### 동시성(부하) 크기를 동적으로 변경할 수 있도록 확장하기
- 동시성 부하 크기를 동적으로 조절할 필요가 있다면 단순히 함수 형태로 구현하기보다는 클래스 기반 구조를 채택하는 편이 훨씬 유리
- 클래스는 상태와 로직을 한 곳에 모아 관리할 수 있어 동적 동시성 조절이나 자원 재할당을 훨씬 명확하고 직관적으로 구현할 수 있음
```ts
class TaskPool<T> {
	private readonly tasks: TaskRunner<T>[];
	private readonly pool: TaskRunner<T>[];
	public poolSize: number;
	
	constructor(fs: (() => Promise<T>[]), poolSize: number) {
		this.tasks = fs.map(f => new TaskRunner(f));
		this.poolSize = poolSize;
	}
	
	setPoolSize(poolSize: number) {
		this.poolSize = poolSize;
	}
	
	private canExpandPool() {
		return this.pool.length < this.poolSize;
	}
	
	async runAll() {
		cosnt { pool, tasks } = this;
		
		let i = 0;
		const { length } = tasks;
		
		while (i < length) {
			const nextTask = task[i];
			pool.push(nextTask);
			const isNotLast = ++i < length;
			
			if (isNotLast && this.canExpandPool()) {
				continue;
			}
			
			await Promise.race(pool.map(task => task.run()));
			pool.splice(pool.findIndex(task => task.isDone), 1);
		} 
		
		return Promise.all(tasks.map(task => task.promise));
	}
}
```
- TaskPool 클래스의 특징
	- 클래스 기반 상태 관리
	- 상태 변화에 따른 유연한 로직 적용
		- `isNotLast`를 사용해 마지막이 아닐 때만 `continue`를 하도록 제어
		- 마지막 항목이라면 `continue`를 수행하지 않아 중간에 `poolSize`가 늘어나더라도 더 이상 `pool`을 확장하지 않도록 함
		- `canExpandPool()` 메서드는 현재 풀 상태를 점검하여 추가 작업을 투입할 수 있는지 판별하며 메서드명으로 그 의도를 직관적으로 드러냄
		- 클래스 내부에서 상태를 일관성 있게 관리하므로 동적 동시성 조절이나 자원 재할당과 같은 로직을 한곳에서 명확하게 구현할 수 있음
		- `setPoolSize()` 메서드를 통해 외부에서 동시성 한도를 동적으로 변경할 수 있음
	- 명령형, 객체지향, 함수형 패러다임을 통한 직관적 해결
- 처음에는 간단한 함수 형태로 문제를 해결한 뒤 기능적 요구사항이나 복잡성이 늘어날 때 점진적으로 클래스를 도입하고 추상화를 높이는 방식이 더 실용적
- 이런 방식으로 접근하면 불필요한 선행 설계로 인한 코드 부다을 줄이고 필요한 시점에 필요한 만큼만 추상화를 적용함으로써 팀 생산성과 코드 품질 모두를 향상시킬 수 있음
### 무한 반복되는 작업의 부하 조절하기
- 무한 반복되는 작업의 부하를 `TaskPool`로 조절하고자 한다면
- 작업을 이터레이터로 받아들일 수 있도록 구조를 변경하고 이터레이터의 지연성을 활용해 무한 반복이 가능하도록 대응하면 됨
```ts
function* map<A, B>(
	f: (value: A) => B,
	iterable: Iterable<A>
): IterableIterator<B> {
	for (const value of iterable) {
		yield f(value)
	}
}

class TaskPool<T> {
	private readonly taskIteratro: IterableIterator<TaskRunner<T>>
	private readonly pool: TaskRunner<T>[] = [];
	public poolSize: number;
	
	constructor(fs: Iterable<() => Promise<T>>, pool: number) {
		this.taskIterator = map(f => new TaskRunner(f), fs);
		this.poolSize = poolSize;
	}
	
	setPoolSize(poolSize: number) {
		this.poolSize = poolSize;
	}
	
	private canExpandPool () {
		return this.pool.length < this.poolSize;
	}
	
	async runAll() {
		const { pool, taskIterator } = this;
		const tasks: TaskRunner<T>[] = [];
		
		while (true) {
			const { done, value: nextTask } = taskIterator.next();
			
			if (!done) {
				pool.push(nextTask);
				tasks.push(nextTask);
				if (this.canExpandPool()) {
					continue;
				}
			} 
			
			if (done && pool.length === 0) {
				break;
			}
			
			await Promise.race(pool.map(task => task.run()));
			pool.splice(pool.findIndex(task =< task.isDone), 1);
		}	
		
		return Promise.all(tasks.map(task => task.promise))
	}
}
```
- 제너레이터로 전환하면서 변경된 주요 사항과 이유
	1. `fs` 타입 변경
		-  `counstructor(fs: (() => Promise<T>[], ...)` => `constructor(fs: Iterable<() => Promise<T>> , ...)`
		- 무한 반복 작업을 지원하려면 `fs`가 배열이 아니라 이터러블 혹은 이터러블 이터레이터여야함
	2. `this.tasks` 초기화로 로직 변경
		- `this.tasks = fs.map(f => new TaskRunner(f))` => `this.taskIterator = map(f => new TaskRunner(f), fs)`
		- `map` 제너레이터 함수는 `fs` 이터러블을 입력받아 `TaskRunner`들을 반환할 이터러블 이터레이터를 생성함
	3. `runAll` 메서드에서 작업 반복 방식 변경
		- `taskIterator.next()`로 이터레이터에서 항목을 하나씩 꺼내고 `nextTask`를 `pool`과 `tasks`에 담음
		- 더 이상 꺼낼 항목이 없고, `pool`이 모두 소진되었을 때 반복을 종료
		- `taskIterator`가 무한 이터레이터라면 무한 반복을 지원
- 무한 이터레이터를 활용해 페이지를 계속해서 크롤링하는 작업을 `TaskPool`로 부하를 조절하며 처리하는 개념적인 예시
```ts
import { map, range, delay } from '@fxts/core';

async function crawling(page: number) {
	console.log(`${page} 페이지 분석 시작`);
	await delay(5_000);
	console.log(`${page} 페이지 저장 완료`);
	return page;
}

void new TaskPool(
	map(page => () => crawling(page), range(Infinity)),
	5
).runAll();
```
### `runAllSettled` 추가하기
- `runAllSettled()`는 `Promise.allSettled()`와 동일하게 모든 작업이 완료될 때까지 기다린 뒤 각 작업의 성공/실패 상태를 배열 형태로 반환
- 그러면서도 `TaskPool`은 `poolSize`를 활용하여 동시에 실행하는 작업 수를 제어하므로 부하를 적절히 분산하면서 모든 작업의 결과를 한번에 확인할 수 있음
```ts
class TaskPool<T> {
	private readonly taskIteratro: IterableIterator<TaskRunner<T>>
	private readonly pool: TaskRunner<T>[] = [];
	public poolSize: number;
	
	constructor(fs: Iterable<() => Promise<T>>, pool: number) {
		this.taskIterator = map(f => new TaskRunner(f), fs);
		this.poolSize = poolSize;
	}
	
	setPoolSize(poolSize: number) {
		this.poolSize = poolSize;
	}
	
	private canExpandPool() {
		return this.pool.length < this.poolSize;
	}
	
	 private async run(errorHandle: (err: unknown) => unknown) {
		 const { pool, taskIterator } = this;
		const tasks: TaskRunner<T>[] = [];
		
		while (true) {
			const { done, value: nextTask } = taskIterator.next();
			
			if (!done) {
				pool.push(nextTask);
				tasks.push(nextTask);
				if (this.canExpandPool()) {
					continue;
				}
			} 
			
			if (done && pool.length === 0) {
				break;
			}
			
			await Promise.race(pool.map(task => task.run())).catch(errorHandle);
			pool.splice(pool.findIndex(task =< task.isDone), 1);
		}	
		
		return tasks.map(task = task.promise)
	 }
	
	async runAll() {
		return Promise.all(await this.run(err => Promise.reject(err)));
	}
	
	async runAllSettled() {
		return Promise.allSettled(await this.run(() => undefined));
	}
}
```
- `runAllSettled`를 추가한 `TaskPool` 클래스 특징
	1. 공통 로직 추출
		- `run()` 메서드가 작업들을 순차적으로 풀에 추가하고 `poolSize`만큼 병렬로 수행되도록 관리하는 공통 로직을 담당
	2. `runAll()` 동작
		- `runAll()`은 `run()`에서 반환한 `Promise` 배열을 `Promise.all()`로 처리한다
		- 모든 작업이 성공적으로 완료될 때까지 대기하는 패턴을 구현
	3. `runAllSettled()` 동작
		- `runAllSettled()`는 `run()`에서 반환한 `Promise` 배열을 `Promise.allSettled()`로 처리
		- 모든 작업을 실패 여부와 상관없이 끝날 때까지 기다리고 결과를 한 번에 받아볼 수 있는 패턴을 적용
	4. `run()` 메서드의 보조 함수와 `catch` 추가
		- `run()` 메서드는 `errorHandle`과 같은 보조 함수를 인자로 받음
		- 이를 통해 외부에서 에러 처리 전략을 유연하게 바꿀 수 있음
		- `Promise.race(...)`에는 `.catch(errorHandle)`을 추가
	5. `runAll()`의 에러 처리
		- `runAll()`은 `errorHandle`로 `err => Promise.reject(err`를 전달
		- 이는 `await Promise.race(...)`가 실행될 때 만약 어떤 작업이 에러를 발생시키면 `Promise.reject`를 던져 그 즉시 실패하도록 만듦
		- 결과적으로 `runAll()`은 *하나라도 실패하면 전체가 실패*하는 `Promise.all()`의 원래 동작 방식에 충실
	6. `runAllSettled()`의 에러 처리
		- `runAllSettled()`는 `errorHandle`로 `() => undefined`를 전달
		- 이는 `task.run()`에서 발생하는 에러를 사실상 무시하는 방식으로 처리하여 `await Promise.race(...)` 호출 시 에러가 전파되지 않게함
		- 그 결과 모든 작업이 끝날 때까지 계속 진행할 수 있고 마지막에 `Promise.allSettled()`를 통해 성공/실패 결과를 모두 담은 배열을 얻을 수 있음
- 이렇게 설계한 `TaskPool` 클래스는 원하는 시나리오 맞춰 동작 모드를 쉽게 전환활 수 있음
- 메서드를 선택하는 것만으로 *하나라도 실패 시 즉시 중단*하거나 *실패한 작업이 있어도 끝까지 진행한 뒤 전체 결과를 수집*하는 식으로 로직과 결과물을 결정할 수 있음
```ts
const tasks = [
	createAsyncTask("A", 1000),
	() => createAsyncTask("B", 500)().then(() => Promise.reject('no!')),
	createAsyncTask("C", 800),
	createAsyncTask("D", 300),
	createAsyncTask("E", 1200)
];

async function runAllTest() {
	try {
		const result = await new TaskPool(task, 2).runAll();
		console.log(result);
	} catch (e) {
		console.log(e) // no!
	}
}

await runAllTest();

async function runAllSettledTest() {
	const result = await new TaskPool(task, 2).runAllSettled();
	console.log(result);
	// [
	// { status: 'fullfilled', value: 'A' },
	// { status: 'rejected', value: 'no!' },
	// { status: 'fullfilled', value: 'C' },
	// { status: 'fullfilled', value: 'D' },
	// { status: 'fullfilled', value: 'E' },
	// ]
} 

await runAllSettledTest();

async function runAllTest2() {
	try {
		const task = (page: number) => () => 
			page === 7
				? Promise.reject(page)
				: crawling(page)'
				
		await new TaskPool(map(task, range(Infinity)), 5).runAll();
	} catch (e) {
		console.log(`crawling 중간에 실패! (${e}페이지)`);
	}
}

await runAllTest2();

async function runAllSettledTest2() {
	const task = (page: number) => () =>
		page === 7
			? Promise.reject(page)
			: crawling(page);
			
	const taskPool = new TaskPool(map(task, range(Infinity)), 5);
	
	void taskPool.runAllSettled();
	
	setTimeout(() => {
		taskPool.setPoolSize(10);
	}, 10_000);
}
```
## 요약 정리
#### 구조 문제는 객체지향으로, 로직 문제는 함수형으로 해결하기
- 복잡하고 중첩된 데이터나 계층적인 구조를 다룰 때는 객체지향 패러다임을 활용해 명확한 구조를 잡을 수 있음
- 반면 데이터 변환이나 리스트 프로세싱과 같은 순수한 로직 문제는 함수형 패러다임을 통해 예측 가능하고 안정적으로 구현할 수 있음
- 이러한 역할 분담은 코드 이해도를 높이고 유지보수성을 향상시키는 데 큰 도움이 됨
#### 문제에 적합한 패러다임을 과감히 선택하기
- 단일 패러다임에 매여 복잡한 문제를 억지로 풀기보다는 상황에 따라 객체지향, 함수형, 명령형 패러다임을 조화롭게 섞어 쓰는 전략이 훨씬 효율적
- 문제의 본질에 맞는 패러다임을 과김히 선택하고 해당 패러다임이 제공하는 강점을 적극 활용하면 복잡한 요구사항도 깔끔하게 해결할 수 있음
#### 객체에 상태를 기록하고 값으로 다루기
- 객체지향 패러다임을 활용하면 관심사를 명확히 분리하고 문제 영역의 개념을 직관적으로 모델링할 수 있음
- 이로써 코드 구조는 더욱 논리적이고 예측 가능한 형태로 정돈됨
- 클래스와 객체는 데이터(상태)와 그 상태를 변화시키는 행위(메서드)를 하나의 추상화 단위로 묶어줌
- 이를 통해 변화하는 상황을 체계적으로 관리하고 다른 부분에서 반복적으로 고려해야할 세부사항을 숨길 수 있음
- 이러한 객체지향적 설계는 복잡한 상태 변화를 간결하게 표현하고 유지보수성을 높이며 코드 이해도와 재사용성을 강화하는 데 크게 기여
- 또한 이런 객체를 함수형 변환 로직과 결합하면 순수하고 예측 가능한 데이터를 처리하는 과정과 구조적이고 명확한 상태 곤리를 자연스럽게 통합하여 궁극적으로 안정적이고 읽기 쉬운 코드를 만들어 낼 수 있음
#### 변화를 알리고 통신하기
- 분리된 객체들은 자신의 상태 변화를 이벤트 형태로 외부에 알리도록 설계할 수 있음
- 이를 통해 다른 객체나 로직이 상황 변화를 감지하고 적절히 대응하는 구조를 손쉽게 구축할 수 있음
- 이러한 객체 간 통신은 일급 함수, 이터레이터, 제너레이터 등 멀티패러다임 언어에서 제공하는 다양한 기능을 적극 활용함으로써 더욱 직관적이고 우아하게 구현 가능함
- 결과적으로 각 컴포넌트의 복잡한 상호작용을 명확하고 이해하기 쉬운 방식으로 표현할 수 있게 됨