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
- `XSS`*Cross Site Scripting*는 웹 페이지에 악성 스크립트를 삽입하여 해당 페이지를 보는 다른 사용자에게 피해를 주는 공격 기법