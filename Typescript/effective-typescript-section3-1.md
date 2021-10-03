# 타입 추론 Part.1

## 추론 가능한 타입을 사용해 장황한 코드 방지하기

### 불필요한 타입 선언

타입스크립트를 처음 사용할 때 가장 많이 하는 실수가 모든 선언문에서

타입을 함께 선언해주는 것입니다.

사실 타입스크립트는 `타입 추론` 을 해주기 때문에 되도록 명시적 타입 선언은 

꼭 필요한 경우에만 사용하는 것이 좋습니다.

### 타입 선언이 필요한 경우

```jsx
function logProduct(product: Product) {
	const { id, name, price } = product

	// ...
}
```

위와 같은 경우 함수 매개변수에 대해서 타입스크립트가 스스로 타입을 판단하기 어렵습니다.

때문에 `Product` 라는 타입을 정의해주었고 구조분해할당을 통해 타입추론이 일어납니다.

이상적인 타입스크립트 코드는 함수 내에서 생성된 지역 변수에 대해서는 타입 선언을 지양하고

함수 시그니처(함수의 타입)에 대해서는 타입을 정의하는 것을 권장합니다.

간혹 매개변수에 타입 정의를 하지 않을 경우도 있는데, 이는 기본값을 지정한 경우입니다.

```jsx
function parseNumber(str: string, base=10) {
	// ...
}
```

이 경우 `base` 는 `number` 타입으로 추론됩니다.

### 함수의 반환 타입을 정의하면 좋은 점

함수의 반환 타입을 명시하면 구현상의 오류를 사용자 코드의 오류로 표시하지 않습니다.

다음과 같이 `Promise` 를 사용하는 코드가 있다고 가정하겠습니다.

`getQuote` 는 시세정보를 비동기로 가져오고 캐싱하여 중복된 API 호출을 방지한 상황입니다.

```jsx
const cache: { [ticker: string]: number } = {};

function getQuote(ticker: string) {
	if (ticker in cache) {
		return cache[ticker];
	}

	return fetch(`https://quotes.example.com/?q=${ticker}`)
					.then(response => response.json())
					.then(quote => { cache[ticker] = quote; return quote; });
}
```

위 코드에는 사실 한가지 문제가 있습니다.

이는 위 함수를 호출하는 구문을 작성하면 알 수 있는데요

```jsx
getQuote('MSFT').then(considerBuying); // ERROR: number | Promise<any> 형식에 'then' 속성이 없습니다.
```

`getQuote` 함수는 항상 Promise 를 반환해야하기 때문에

조건문의 반환값은 `Promise.resolve(cache[ticker])` 로 해야합니다.

만약 위 함수에 반환 타입으로 `Promise<number>` 를 명시했다면

사용하는 코드에서 타입 에러가 발생하지 않고 구현 과정에서 타입 에러가 발생하였을 것입니다.

이를 통해 반환 타입을 명시하면 구현상의 오류가 사용자 코드의 오류로 표시하지 않습니다.

이외에도 반환 타입을 명시하면 다음과 같은 장점이 있습니다.

- 함수에 대해서 보다 명확히 파악할 수 있습니다.
- 명명된 타입을 통해 직관적인 표현을 할 수 있습니다.

## 다른 타입에는 다른 변수 사용하기

자바스크립트는 동적 타입 언어이기 때문에 다음과 같은 코드가 허용됩니다.

```jsx
let id = '123456';
console.log(id);

id = 123456;
console.log(id);
```

반면 타입스크립트는 이미 추론된 타입에 대해서 타입 변경이 불가능합니다.

타입스크립트에서 타입을 변환하는 경우는 `타입 좁히기` 와 `any` 타입을 사용하는 것입니다.

위 문제를 해결하기 위해 다음과 같이 `유니온 타입` 을 사용할 수도 있습니다.

```jsx
let id: number | string = '123456';
```

하지만 이 경우 `id` 를 사용하기 전에 어떤 타입인지 확인하는 과정이 필요하기 때문에

다른 타입에는 별도의 변수를 할당하는 것이 보다 유리합니다.

이는 다음과 같은 5가지 이유가 존재합니다.

- 서로 관련이 없는 값을 분리
- 구체적인 변수명 할당 가능
- 타입 추론 향상 및 간결해진 타입
- let 대신 const 를 사용하여 타입 체커가 타입을 추론하기 유리하도록 함

## 타입 넓히기

### 타입 넓히기란?

타입스크립트를 사용할 때 타입을 명시하지 않으면 타입 체커는 타입을 유추해야 합니다.

주어진 값을 가지고 가능한 모든 타입 경우를 유추해야하는데, 이를 `타입 넓히기` 라고 합니다.

때문에 타입 정보가 충분하지 않다면 의도된 것과 다른 타입으로 추론될 수 있습니다.

### 타입 넓히기 제어하기

타입 넓히기 과정을 제어하는 방법은 다음과 같이 두 가지가 존재합니다.

- `const` 사용하기

```jsx
let x = 'x' // x 의 타입은 string 으로 추론됨

const x = 'x' // x 의 타입은 문자열 리터럴인 'x' 로 추론됨 (x 가 재할당 불가능하기 때문에)
```

- 타입스크립트의 기본 동작을 재정의

타입스크립트의 기본 동작을 재정의하는 방법은 세가지가 존재합니다.

첫째로 `명시적인 타입 구문` 을 제공하는 것입니다.

```jsx
const v: { x: 1 | 2 | 3 } = { x: 1 }
```

두번째는 타입 체커에 추가적인 문맥을 제공하는 것입니다.

세번째는 `const 단언문` 을 사용하는 것입니다. (`const` 변수 선언과 다름!)

```jsx
const v1 = {
	x: 1,
	y: 2
}; // 타입은 { x: number; y: number; }

const v2 = {
	x: 1 as const,
	y: 2,
}; // 타입은 { x: 1; y: number; }

const v3 = {
	x: 1,
	y: 2
} as const; // 타입은 { readonly x: 1; readonly y: 2; }

const a1 = [1, 2, 3] as const; // 타입은 readonly [1, 2, 3]
```

## 타입 좁히기

### 타입 좁히기란?

반대로 타입 좁히기란 타입스크립트가 넓은 타입에서 좁은 타입으로 

진행하는 과정을 말합니다.

일반적으로 다음과 같이 `null` 체크가 대표적인 예시라고 할 수 있습니다.

```jsx
const el = document.getElementById('foo'); // HTMLElement | null
```

### 타입을 좁히는 방법들

이러한 타입 좁히기를 수행하기 위한 몇가지 방법이 있습니다.

첫 번째는 분기문에서 예외를 발생시키거나 함수를 반환시키는 방법입니다.

```jsx
const el = document.getElementById('foo') // HTMLElement | null

if (!el) { throw new Error('unable to find #foo') }

el.innerHTML = 'typescript study'; // 여기서부터 el 의 타입은 HTMLElement
```

이외에도 다양한 방법이 있는데, 

대표적으로 `instanceof 체크, 속성 체크, 내장 함수` 를 사용하는 방법이 있습니다.

```jsx
// instanceof 사용
function test(param) {
	if (param instanceof RegExp) {
		// ...
	}
}

// 필요한 속성이 있는지 체크
interface A { a: number; }
interface B { b: number; }

function pickAB(ab: A | B) {
	if ('a' in ab) { ab // 타입은 A }
	else { ab // 타입은 B }
	ab // 타입은 A | B
}

// 배열인지 확인하기 위해 Array.isArray 사용
function contains(terms: string | string[]) {
	const termList = Array.isArray(terms) ? terms: [terms];
	// ...
}
```

또 다른 방법으로 `태그된 유니온` 을 사용할 수 있습니다.

```jsx
interface UploadEvent {
	type: 'upload';
	filename: string;
}

interface DownloadEvent {
	type: 'download';
	filename: string;
}

type AppEvent = UploadEvent | DownloadEvent;

function handleEvent(e: AppEvent) {
	switch(e.type) {
		case 'download':
			// ...
			break;
		case 'upload':
			// ...
			break;
	}
}
```

타입스크립트가 타입을 식별하지 못한다면 

`사용자 정의 타입 가드` 라는 커스텀 함수를 사용할 수 있습니다.

```jsx
function isInputElement(el: HTMLElement): el is HTMLInputElement {
	return 'value' in el;
}

function getElementContent(el: HTMLElement) {
	if (isInputElement(el)) {
		el // 타입은 HTMLInputElement
		return;
	}
	el // 타입은 HTMLElement
}
```
