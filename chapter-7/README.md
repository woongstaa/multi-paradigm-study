# 객체지향 프런트엔드 개발 그리고 멀티 패러다임적 접근과 응용
- 여전히 풍부한 기능과 고품질 사용자 경험을 제공해야 하는 실시간 편집 툴을 만드려면 객체지향 프로그래밍 능력이 필요
- 이러한 애플리케이션을 개발할 때는 객체지향 기반의 SDK를 통해 UI 요소를 직접 핸들링하고 시스템 자원이나 플랫폼 기능을 유기적으로 연동하는 구조를 이해하며 코딩하는 능력이 요구됨
- 방대한 기능을 모듈화하고 관리하는 데도 객체자향적인 접근이 매우 유용
- 객체지향 프로그래밍 기술은 캡슐화, 추상화, 상속, 다형성 등 명확한 개념 체계를 갖추고 있고 디자인 패턴이나 수많은 SDK 사례들을 포함해 현업에서 오랜 기간 검증된 유산을 가지고 있음
- 객체지향 프로그래밍 패턴과 설계는 특정 라이브러리나 프레임워크에 의존하지 않고도 더 근본적인 방식으로 코드를 작성할 수 있으며 프로그래밍 업무 영역을 넓히는 데도 용이함
## Settings 앱 만들기
### SwitchView
```ts
import { html, View } from 'rune-ts';

class SwitchView extends View<{ on: boolean }> {
	override template() {
		return html`
			<button class="${this.data.on ? 'on' : ''}">
				<span class="toggle"></span>
			</button>
		`;
	}
}

export function main() {
	console.log(
		new SwitchView({ on: true }).toHtml()
	)
	
	document.querySelector('#body')!.append(
		new SwitchView({ on: false }).render()
	);
}
```
```ts
class SwitchView extends View<{ on: boolean }> {
	override template() {
		return html`
			<button class="${this.data.on ? 'on' : ''}">
				<span class="toggle"></span>
			</button>
		`;
	}
	
	protected override onRender() {
		this.element().addEventListner('click', () => 
			this.setOn(!this.data.on)
		);
	}
	
	setOn(bool: boolean) {
		this.data.on = bool;
		this.element().classList.toggle('on', bool);
	}
}
```
#### `public`과 `protected`
- `public` 접근 제어자는 클래스 외부에서도 해당 메서드를 자유롭게 호출할 수 있음을 의미
- 이를 통해 프로그램 로직이 필요할 때 언제든지 `SwitchView`의 상태를 직접 변경할 수 있음

- `protected` 접근 제어자를 사용하면 클래스 내부나 이를 상속받는 하위 클래스에서는 접근할 수 있지만 클래스 인스턴스 외부에서는 접근할 수 없다
- 이를 통해 다음과 같은 장점을 얻을 수 있음
	- 클래스 내부 확장용 메서드
		- `onRender()`는 `View` 클래스 내부나 이를 상속받는 하위 클래스에서만 UI 초기화나 이벤트 바인딩 등의 작업을 수행하도록 의도된 메서드
		- 상속 관계를 통해 오버라이드하거나 확장할 수 있게 하면서도 외부 코드에서 직접 접근하지 못하게 함으로써 해당 메서드가 뷰 렌더링 과정에 속하는 내부용 로직임을 명확히 함
	- 외부 호출 방지
		- `onRender()`는 뷰 렌더링 과정에서 자동으로 호출되는 라이프사이클 훅 메서드로 `View` 클래스 내부 로직에 의해 제어됨
		- 만약 이 메서드를 `public`으로 두면 외부 코드에서 임의로 호출할 수 있어 예상치 못한 UI 변화나 상태 문제를 일으킬 수 있음
		- `protected`로 선언하면 외부에서 직접 호출할 수 없어 렌더링 프로세스의 일관성과 안정성을 보장할 수 있음
### `SettingItemView`
```ts
class SwitchView extends View<{ on: boolean }> {
	// ...
}

Interface Setting {
	title: string;
	on: boolean;
}

class SettingItemView extends View<Setting> {
	override template() {
		return html`
			<div>
				<span class="title">${this.data.title}</span>
				${new SwitchView(this.data)}
			</div>
		`;
	}
}

export function main() {
	const setting = { title: 'Wi-Fi', on: false };
	
	document.querySelector('#body')!.append(
		new SettingItemView(setting).render()
	);
}
```
### `SettingListView`
```ts
class SwitchView extends View<{ on: boolean }> {
	// ...
}

Interface Setting {
	title: string;
	on: boolean;
}

class SettingItemView extends View<Setting> {
	override template() {
		return html`
			<div>
				<span class="title">${this.data.title}</span>
				${new SwitchView(this.data)}
			</div>
		`;
	}
}

class SettingListView extends View<Setting[]> {
	override template() {
		return html`
			<div>
				${this.data.map(setting => new SettingItemView(setting))}
			</div>
		`;
	}
}

export function main() {
	const settings: Setting[] = [
		{ title: 'Wi-Fi', on: false },
		{ title: 'Bluetooth', on: true },
		{ title: 'Sound', on: false },
	];
	
	document.querySelector('#body')!.append(
		new SettingListView(settings).render()
	);
}
```
### `SettingPage`
```ts
class SettingPage extends View<Setting[]> {
	override template() {
		return html`
			<div>
				<div class="header">
					<h2>Setting</h2>
					${new SwitchView({ on: false })} 
				</div>
				<div class="body">
					${new SettingListView(this.data)} 
				</div>
			</div>
		`;
	}
}

export function main() {
	const settings: Setting[] = [
		{ title: 'Wi-Fi', on: false },
		{ title: 'Bluetooth', on: true },
		{ title: 'Sound', on: false },
	];
	
	document.querySelector('#body')!.append(
		new SettingPage(settings).render()
	);
}
```
### 전체 토글 기능 추가하기
#### `toggleAll` 메서드 구현하기
```ts
class SettingItemView extends View<Setting> {
	switchView = new SwitchView(this.data);

	override template() {
		return html`
			<div>
				<span class="title">${this.data.title}</span>
				${this.switchView}
			</div>
		`;
	}
}

class SettingListView extends View<Setting[]> {
	itemViews = this.data.map(setting => new SettingItemView(setting))
	
	override template() {
		return html`
			<div>
				${this.itemViews}
			</div>
		`;
	}
}

class SettingPage extends View<Setting[]> {
	listView = new SettingListView(this.data)

	override template() {
		return html`
			<div>
				<div class="header">
					<h2>Setting</h2>
					${new SwitchView({ on: false })} 
				</div>
				<div class="body">
					${this.listView} 
				</div>
			</div>
		`;
	}
	
	toggleAll(bool: boolean) {
		this.listView.itemViews.forEach((itemView) => itemView.switchView.setOn(bool));
	}
}
```
1. `SettingPage`에서 `listView` 인스턴스 보관
2. `SettingListView`에서 `itemViews` 배열 유지
3. `SettingItemView`에서 `switchView` 프로퍼티 유지

- 여기에 `togleAll` 메서드에서 `filter`를 활용하면 로직을 보다 효율적으로 만들 수 있음
```ts
class SettingPage extends View<Setting[]> {
	// ...
	toggleAll(bool: boolean) {
		this.listView.itemViews
			.filter(itemView => itemView.data.on !== bool)
			.forEach(itemView => itemView.switchView.setOn(bool));
	}
}
```
### 객체 간 통신과 커스텀 이벤트 디스패치
- 헤더의 `SwitchView` 상태가 변경될 때마다 `toggleAll`을 호출하기 위해 `SwitchView`에 클릭 이벤트를 등록하는 대신, `SwitchView` 자체가 상태 변화 시점을 알려주는 커스텀 이벤트를 발행하도록 설계할 수 있음
- 이렇게 하면 `SwitchView`가 스스로 상태 변경을 외부에 알리는 구조를 갖추며 `SwitchView`의 역할과 구조에 더욱 어울리는 명확한 이벤트 통신 구조를 설계할 수 있음
#### `SwitchView`와 이벤트로 통신하기
```ts
class SwitchView extends View<{ on: boolean }> {
	override template() {
		return html`
			<button class="${this.data.on ? 'on' : ''}">
				<span class="toggle"></span>
			</button>
		`;
	}
	
	protected override onRender() {
		this.element().addEventListner('click', () => 
			this.setOn(!this.data.on)
		);
	}
	
	setOn(bool: boolean) {
		this.data.on = bool;
		this.element().classList.toggle('on', bool);
		
		const event = new CustomEvent('toggled', { bubbles: true, detail: this.data});
		this.element().dispatchEvent(event);
	}
}

class SettingPage extends View<Setting[]> {
	listView = new SettingListView(this.data)
	toggleAllView = new SwitchView({ on: false })

	override template() {
		return html`
			<div>
				<div class="header">
					<h2>Setting</h2>
					${this.toggleAllView} 
				</div>
				<div class="body">
					${this.listView} 
				</div>
			</div>
		`;
	}
	
	protected override onRender() {
		this.toggleAllView.element().addEventListner('toggled', (e) => {
			const customEvent = e as CustomEvent<{ on: boolean }>
			const bool = customEvent.detail.on;
			console.log('header:', bool);
			this.toggleAll(bool);
		})
	}
	
	toggleAll(bool: boolean) {
		this.listView.itemViews
			.filter(itemView => itemView.data.on !== bool)
			.forEach(itemView => itemView.switchView.setOn(bool));
	}
}
```
#### 전체 토글 상태를 본문으로부터 동기화하기
```ts
class SettingPage extends View<Setting[]> {
	listView = new SettingListView(this.data)
	toggleAllView = new SwitchView({ on: false })

	override template() {
		return html`
			<div>
				<div class="header">
					<h2>Setting</h2>
					${this.toggleAllView} 
				</div>
				<div class="body">
					${this.listView} 
				</div>
			</div>
		`;
	}
	
	protected override onRender() {
		this.toggleAllView.element().addEventListner('toggled', (e) => {
			const customEvent = e as CustomEvent<{ on: boolean }>
			const bool = customEvent.detail.on;
			console.log('header:', bool);
			this.toggleAll(bool);
		})
		
		this.listView.element().addEventListner('toggled', () => {
			this.syncToggleAllView();
		})
	}
	
	toggleAll(bool: boolean) {
		this.listView.itemViews
			.filter(itemView => itemView.data.on !== bool)
			.forEach(itemView => itemView.switchView.setOn(bool));
	}
	
	syncToggleAllView() {
		const bool = this.listView.itemViews.every(itemView => itemView.data.on);
		console.log('body:', bool);
		this.toggleAllView.setOn(bool)
	}
}
```
- 이 코드는 의도한 대로 동작하지 않고 매우 복잡하고 예측하기 어려운 패턴이 나타남
- 어떤 `SwitchView`를 클릭하느냐에 따라 동작 패턴이 달라지고 전반적으로 코드가 어디서 어떻게 반복되어 돌아가는지 가늠하기 어려움
- 문제의 핵심은 본문의 `SwitchView`와 헤더의 `SwitchView` 모두 상태 변화 시 `toggled` 이벤트를 발생시키고 이 이벤트가 상호 영향을 주고 받으며 사실상 루프에 가까운 연쇄적인 호출이 발생한다
- 엄밀히 말하면 무한 루프에 빠지는 구조지만 `.filter(itemView => itemView.data.on !== bool)`코드 덕분에 일전 단계에서 반복이 멈추며 완벽한 무한 루프 대신 이상하고 불안정한 동작으로 마무리 됨
- 이로 인해 전체적인 로직이 의도와 다르게 작동하고 코드를 이해하거나 예측하기가 어려움
### 이벤트가 자꾸 루프에 빠지고 부수 효과가 발생하는 이유
- 우리의 이벤트가 반복적으로 루프에 빠지고 예상치 못한 부수효과를 일으키는 이유는 무엇일까?
- 이 문제를 단순히 이벤트 흐름을 단방향으로 제한하거나 스토어나 중앙 이벤트 버스와 같은 상태 관리 계층 또는 특정 라이브러리를 도입하는 것으로 해결할 수 있을까?
- 그렇지 않다. 더욱 근본적인 해결책이 존재함
#### `SwitchView` 이벤트 설계 변경하기
- 사용자의 직접적인 상호작용으로 상태가 바뀔 때만 이벤트를 발생 시키고 프로그램 로직을 통해 상태를 변경하는 경우에는 이벤트를 발생시키지 않도록 하는 방식
- 이렇게 하면 의도치 않은 이벤트 루프나 복잡한 상호작용 문제를 방지할 수 있음
```ts
class SwitchView extends View<{ on: boolean }> {
	override template() {
		return html`
			<button class="${this.data.on ? 'on' : ''}">
				<span class="toggle"></span>
			</button>
		`;
	}
	
	protected override onRender() {
		this.element().addEventListner('click', () => 
			this.toggle()
		);
	}
	
	private toggle() {
		this.setOn(!this.data.on);
		const event = new CustomEvent('toggled', { bubbles: true, detail: this.data});
		this.element().dispatchEvent(event);
	}
	
	setOn(bool: boolean) {
		this.data.on = bool;
		this.element().classList.toggle('on', bool);
	}
}
```
- 기존에는 `SwitchView`의 상태를 바꿀 때마다 무조건 `toggled` 이벤트를 발생시켰음
- 그 결과 프로그램 로직에서 `SwitchView`의 상태를 변경할 때도 이벤트가 울려 의도치 않은 이벤트 루프나 예측하기 어려운 상호작용이 발생할 수 있었음
- 이제 변경된 코드에서는 다음과 같은 구조를 갖음
	- 사용자 상호작용에 의한 이벤트 발생
	- 프로그램 로직으로 상태 변경 시 이벤트 없음
- 이로써 *상태 변경*과 *이벤트 발생*을 명확히 분리할 수 있음
#### `private` 접근 제어자
- `private`로 선언된 메서드나 속성은 클래스 내부에서만 사용 가능하며 상속받은 하위 클래스나 클래스 인스턴스 외부에서는 접근할 수 없음
- `toggle()` 메서드를 `private`로 선언한 이유는 `SwitchView` 내부 로직을 더 명확하게 하기 위함임
- 이 메서드는 사용자 클릭 이벤트 발생 시에만 실행되는 것이 좋으며 외부 코드에서 직접 호출할 수 없게 하는 것이 중요
	- 내부 로직 보호
	- 명확한 의도 표현
	- 역할 분리
#### 올바른 설계 기준 찾기
- 설계 기준은 어디에서 참고하고 어떤 자료를 살펴봐야할까
	- 표준 문서
	- 널리 알려진 디자인 패턴
	- 기존 플랫폼이나 프레임 워크에서 사용되는 검증된 이벤트 처리방식
- 들을 폭넓게 참고하는 것이 좋음
- 예를 들어 `Web API` 사양이나 `iOS SDK` 문서를 살펴보면 `WHATWG`나 `W3C`에서 정의한 이벤트 처리 방식과 `DOM` 이벤트 모델을 통해 이벤트 버블링, 캡처링, 전파 제어 같은 기본 개념을 깊이 이해할 수 있음
- 또한 이번 사례 에서 겪은 이벤트 무한 루프 문제를 근본적으로 피할 수 있는 구조적 해법을 참고할 수 있으며 더 나아가 안정적이고 유지보수하기 쉬운 이벤트 설계를 구현할 수 있게 됨
- 이처럼 오랜 시간 축적되고 검증된 기술로부터 배우는 것은 언제나 바람직한 접근
- 가까이에 있는 `Web API`의 체크박스는 이미 무한 루프 문제 없이 잘 설계되어 있으며 누구나 참고할 수 있게 잘 보여주고 있었음
- 어쩌면 우리의 이벤트가 자꾸만 루프에 빠지고 부수 효과가 발생하는 이유는 처음 작은 문제를 마주했을 때 근본적인 원인을 파악하기보다 구조가 어긋난 사애에서 단순히 if문으로 상황을 막으려하거나 이벤트 상태/흐름을 관리하는 라이브러리를 무분별하게 도입하는 등의 단편적 대응을 했기 때문일지도 모름
- 아니면 이벤트로 처리하기에 적합하지 않은 상황까지 `pub/sub` 구조나 `reactive programming`, `Observable` 같은 패러다임으로 전체 로직을 통합하려고 하면서 오히려 프로그램이 산발적인 이벤트 핸들링 상황에 노출되고 관리가 더욱 어렵게 되었을 수도 있음
- 무한 루프의 근본적인 원인을 이해하지 못한 채 계속해서 라이브러리를 도입하고 제거/변경하는 방식으로는 이러한 문제를 근본적으로 해결하기 어려움
- 반면 `Web` 기술 표준 문서나 `iOS/Android SDK` 같은 사례 그리고 전통정긴 설계 원칙을 참고하면 단발적인 문제 해결을 넘어 더욱 넓고 탄탄한 기술적 기반을 구축할 수 있음
- 이러한 접근은 앞으로 마주하게 될 다양한 상황에서도 한층 더 안정적이고 유연한 설계와 구현 역량을 갖추는데 큰 도움이 될 것
### 타입 안전한 커스텀 이벤트 통신 패턴
- `rune-ts` 라이브러리의 `CustomEventWithDetail`을 이용하면 커스텀 이벤트를 한층 더 타입 안전하게 처리할 수 있는 설계를 제공함
```ts
import { CustomEventWithDetail, html, View } from 'rune-ts';

type Toggle = { on: booelan };

class Toggled extends CustomEventWithDetail<Toggle> {}

class SwitchView extends View<{ on: boolean }> {
	override template() {
		return html`
			<button class="${this.data.on ? 'on' : ''}">
				<span class="toggle"></span>
			</button>
		`;
	}
	
	protected override onRender() {
		this.element().addEventListner('click', () => 
			this.toggle()
		);
	}
	
	private toggle() {
		this.setOn(!this.data.on);
		this.dispatchEvent(Toggled, { bubbles: true, detail: this.data })
	}
	
	setOn(bool: boolean) {
		this.data.on = bool;
		this.element().classList.toggle('on', bool);
	}
}

class SettingPage extends View<Setting[]> {
	listView = new SettingListView(this.data)
	toggleAllView = new SwitchView({ on: false })

	override template() {
		return html`
			<div>
				<div class="header">
					<h2>Setting</h2>
					${this.toggleAllView} 
				</div>
				<div class="body">
					${this.listView} 
				</div>
			</div>
		`;
	}
	
	protected override onRender() {
		this.toggleAllView.addEventListner(Toggled, e => this.toggleAll(e.detail.on));
		this.listView.element().addEventListner(Toggled, () => {
			this.syncToggleAllView();
		})
	}
	
	toggleAll(bool: boolean) {
		this.listView.itemViews
			.filter(itemView => itemView.data.on !== bool)
			.forEach(itemView => itemView.switchView.setOn(bool));
	}
	
	syncToggleAllView() {
		const bool = this.listView.itemViews.every(itemView => itemView.data.on);
		console.log('body:', bool);
		this.toggleAllView.setOn(bool)
	}
}
```
###  재사용 가능한 컴포넌트 `SwitchView`
```ts
class SwitchView extends View<{ on: boolean }> {
	override template() {
		return html`
			<button class="${this.data.on ? 'on' : ''}">
				<span class="toggle"></span>
			</button>
		`;
	}
	
	protected override onRender() {
		this.element().addEventListner('click', () => 
			this.toggle()
		);
	}
	
	private toggle() {
		this.setOn(!this.data.on);
		this.dispatchEvent(Toggled, { bubbles: true, detail: this.data })
	}
	
	setOn(bool: boolean) {
		this.data.on = bool;
		this.element().classList.toggle('on', bool);
	}
}
```
### 패러다임이 만드는 리액티브한 코드
```ts
type Setting = {
	title: string;
	on: boolean;
}

class SettingItemView extends View<Setting> {
	switchView = new SwitchView(this.data);

	override template() {
		return html`
			<div>
				<span class="title">${this.data.title}</span>
				${this.switchView}
			</div>
		`;
	}
}

class SettingListView extends View<Setting[]> {
	itemViews = this.data.map(setting => new SettingItemView(setting))
	
	override template() {
		return html`
			<div>
				${this.itemViews}
			</div>
		`;
	}
}

class SettingPage extends View<Setting[]> {
	listView = new SettingListView(this.data)
	toggleAllView = new SwitchView({ on: false })

	override template() {
		return html`
			<div>
				<div class="header">
					<h2>Setting</h2>
					${this.toggleAllView} 
				</div>
				<div class="body">
					${this.listView} 
				</div>
			</div>
		`;
	}
	
	protected override onRender() {
		this.toggleAllView.addEventListner(Toggled, e => this.toggleAll(e.detail.on));
		this.listView.element().addEventListner(Toggled, () => {
			this.syncToggleAllView();
		})
	}
	
	toggleAll(bool: boolean) {
		this.listView.itemViews
			.filter(itemView => itemView.data.on !== bool)
			.forEach(itemView => itemView.switchView.setOn(bool));
	}
	
	syncToggleAllView() {
		this.toggleAllView.setOn(this.isAllOn());
	}
	
	isAllOn() {
		return this.listView.itemViews.every(itemView => itemView.data.on);
	}
}
```
- `SettingItemView`, `SettingListView`, `SettiingPage` 안에서 직접 `DOM`을 조작하거나 화면 변경을 다루는 코드를 찾을 수 없음
- 오직 데이터를 다루고 데이터 모델의 메서드를 실행하며 상태를 변경하는 로직만 담고 있음
- 이에 따라 화면 갱신은 자연스럽게 뒤따르게 됨