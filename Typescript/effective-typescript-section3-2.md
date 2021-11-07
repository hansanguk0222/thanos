# 타입 추론 Part.2

## 한꺼번에 객체 생성하기

### 한꺼번에 객체를 생성해야 하는 이유

변수의 값은 변경 가능성이 있지만 타입스크립트의 타입은 일반적으로 

한번 지정되면 변경되지 않습니다.

따라서 여러 속성을 한꺼번에 선언해야 타입추론에 유리합니다.

```jsx
const point = {};

pt.x = 3; // ERROR: {} 형식에 x 속성 없음
pt.y = 4; // ERROR: {} 형식에 y 속성 없음
```

위와 같은 결과가 나오는 이유는 `point` 의 타입이 `{}` 로 추론되기 때문입니다.

### 여러 단계에 걸쳐서 객체 생성하기

만약 큰 객체를 만들어야 하는 경우에도 다음과 같이 여러 단계에 걸쳐서

객체를 생성하는 것은 좋지 못한 방법입니다.

```tsx
const point = { x: 3, y: 4 };
const id = { name: 'jh' };
const namedPoint = {};

Object.assign(namedPoint, point, id);

namedPoint.name; // ERROR: {} 형식에 name 속성이 없습니다.
```

다만 다음과 같이 `객체 전개 연산자`  를 사용하면 문제를 피할 수 있습니다.

```tsx
const namedPoint = { ...point, ...id };
namedPoint.name; // 정상
```

하지만 객체 전개 연산자를 사용하면 다음과 같이 의도하지 않은 타입 추론이 일어날 수 있습니다.

```tsx
declare let hasDates: boolean;

const nameTitle = { name: 'jh', title: 'pharaoh' };
const pharaoh = {
	...nameTitle,
	...(hasDates ? { start: -2589, end: -2566 } : {})
};
```

이 경우 타입은 다음과 같이 추론됩니다.

```tsx
const pharaoh: {
	start: number;
	end: number;
	name: string;
	title: string;
} | {
	name: string;
	title: string;
}
```

만약 의도한 것이 `start` 와 `end` 를 선택적인 필드로 지정하려고 한 것이었다면

다음과 같은 헬퍼 함수를 사용하면 됩니다.

```tsx
function addOptional<T extends object, U extends object>
(a: T, b: U | null): T & Partial<U> {
	return { ...a, ...b };
}
```

## 비동기 코드에는 콜백 대신 async 함수 사용하기

`콜백 지옥` 은 자바스크립트에서 비동기 코드 작성 시 과거에 많은 개발자들을 괴롭혔던 녀석입니다.

```tsx
fetchURL(url1, function(response1) {
	fetchURL(url2, function(response2) {
		fetchURL(url3, function(response3) {
			// ...
		});
	});
});
```

이를 대체하기 위한 방법으로 `Promise` 가 나왔고, 더 나아가 `async await` 을 통해

비동기 코드를 보다 간편하게 작성할 수 있게 되었습니다.

타입스크립트에서는 콜백보다 `Promise` 혹은 `async await` 을 사용하는게 유리합니다.

또한 가능하다면 `Promise` 보다는 `async await` 을 사용하는게 좋습니다.

그 이유는 다음과 같습니다.

- 콜백보다는 Promise 가 코드 작성이 쉽다.
- 콜백보다는 Promise 가 타입 추론이 쉽다.
- 일반적으로 Promise 보다는 async await 구문이 이해하기 쉽다. (가독성이 좋다.)
- async 함수는 항상 Promise 를 반환하도록 강제된다.

### 함수 설계 시 주의할 점

함수를 설계할 때, 함수는 항상 동기적으로 동작하거나 항상 비동기로 실행되도록 해야합니다.

```tsx
const cache: {[url: string]: string} = {};

function fetchWithCache(url: string, callback: (text: string) => void) {
	if (url in cache) {
		callback(cache[url]);
	} else {
		fetchURL(url, text => { cache[url] = text; callback(text); })
	}	
}

function getUser(userId: string) {
	fetchWithCache(`user/${userId}`, profile => { requestStatus = 'success' });
	requestStatus = 'loading';
}
```

만약 다음과 같이 캐싱되지 않은 경우에만 API 를 호출하는 코드가 있다고 하겠습니다.

이 경우 캐싱이 된 유저의 경우 `동기` 로 함수가 호출되기 때문에 해당 함수를 사용하기 까다로워집니다.

위 함수에 `async await` 를 사용할 경우 문제는 깔끔히 해결됩니다.

```tsx
function getUser(userId: string) {
	requestStatus = 'loading';
	const profile = await fetchWithCache(`/user/${userId}`);
	requestStatus = 'success';
}
```

또한 `async` 함수에서 Promise 반환값은 또 한번 Promise 로 감싸지지 않습니다.

다음의 함수에서 반환 타입은 `Promise<Promise<any>>` 가 아니라 `Promise<any>` 입니다.

```tsx
async function getJSON(url: string): Promise<any> {
	const response = await fetch(url);
	return response.json();
}
```

## 타입 추론에 문맥이 어떻게 사용되는지 이해하기

자바스크립트 및 타입스크립트는 다음과 같이 값을 분리하는 리팩토링이 가능합니다.

```tsx
callSomeFunction('param');

// 위 함수 호출문을 다음과 같이 값 분리

const param = 'param';
callSomeFunction(param);
```

다만, 위와 같은 함수 리팩토링 시 타입스크립트 입장에서는 의도와 다른 타입 추론이 발생할 수 있습니다.

다음과 같은 한 가지 예시를 보겠습니다.

```tsx
type Language = 'Javascript' | 'Typescript'

function setLanguage(lang: Language) {
	//
}

let language = 'Javascript';
setLanguage(language);
```

여기서 `language` 는 `string` 타입으로 추론되기 때문에 

`'Javascript' | 'Typescript'` 유니온 타입에 할당이 불가능합니다.

이를 해결하는 방법은 `language` 의 변수 선언을 `const` 로 바꿔주는 것입니다.

이렇게 하면 더 이상 `language` 에 다른 값을 할당할 수 없기 때문에 `language` 가 `'Javascript'` 라는

리터럴 문자열로 타입 추론됩니다.

### 튜플 사용시 주의할 점

다음과 같이 튜플 타입을 매개변수로 하는 함수가 있다고 하겠습니다.

```tsx
function someFunction(param: [number, number]) {
	// ...
}

someFunction([10, 10]); // 정상

const param = [10, 10];
someFunction(param); // ERROR: number[] 를 [number, number] 에 할당 불가능
```

이는 배열의 길이는 가변적이기 때문에 튜플 타입인 `[number, number]` 에 할당 불가능 한 것입니다.

`param` 에 다음과 같이 타입을 명시해주거나,

```tsx
const param: [number, number] = [10, 10];
```

`as const` 선언을 통해 값의 내부까지 상수라는 사실을 타입스크립트에게 알려줘서 해결할 수 있습니다.

다만 이렇게 하면 기존의 튜플타입과는 호환되지 않기 때문에 

`someFunction` 의 기존 매개변수 타입 선언에 `readonly` 를 추가해야 합니다.

```tsx
const param = [10, 10] as const; // readonly [number, number] 로 추론됨

function someFunction(param: readonly [number, number]) { } 
```

### 객체 사용 시 주의할 점

문맥에서 값을 분리할 때 생기는 문제는 객체의 경우에도 마찬가지입니다.

```tsx
type Language = 'Javascript' | 'Typescript';
interface GovernedLanguage {
	language: Language;
	organization: string;
}

function complain(language: GovernedLanguage) { /** */ }

const ts = {
	language: 'Typescript',
	organization: 'Microsoft'
}
complain(ts); // ERROR
```

이 경우에 `ts` 에 `GovernedLanguage` 라는 타입을 명시해주거나,

`as const` 선언을 통해 리터럴 문자열 타입으로 추론 가능하도록 해주면 됩니다.

```tsx
const ts: GovernedLanguage = {
	language: 'Typescript',
	organization: 'Microsoft'
}

// 혹은

const ts = { 
	language: 'Typescript',
	organization: 'Microsoft'
} as const // { language: 'Typescript', organization: 'Microsoft' } 로 추론됨
```
