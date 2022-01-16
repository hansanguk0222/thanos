# Typescript 에서 Any 타입 다루기

## any 타입은 가능한 좁은 범위에서만 사용하기

`someFunction` 이 반환하는 타입이 `process` 함수의 인자와 호환되지 않는다고 하겠습니다.

다음 두 코드 중 더 나은 방법은 어떤 것일까요?

```tsx
// 1.
function f1() {
	const x: any = someFunction();
	process(x);
}

// 2.
function f2() {
	const x = someFunction();
	process(x as any);
}
```

위 코드 중 `2번째` 코드가 더 나은 방법인데, 이는 `process` 함수 호출 이후

`x` 의 타입이 변하지 않기 때문입니다.

여기서 함수 `f1` 이 `x` 를 반환한다면 문제가 더 커지며 프로젝트 전반적으로 영향을 미치게 됩니다.

이는 객체의 속성 타입을 정의할 때도 마찬가지입니다.

`key` 속성이 타입 오류를 가지는 상황을 가정했을 때, 아래와 같이 객체 전체를 `any` 로 선언하는 것이 아니라

```tsx
const config: Config = {
	a: 1,
	b: 2,
	c: {
		key: value,
	}
} as any
```

다음과 같이 해당 속성에만 `any` 타입으로 단언해주면 다른 속성들에는 영향을 미치지 않습니다.

```tsx
const config: Config = {
	a: 1,
	b: 2,
	c: {
		key: value as any,
	}
}
```

## any 를 구체적으로 변형해서 사용하기

`any` 는 Javascript 에서 표현할 수 있는 모든 값을 아우르는 매우 큰 범위의 타입입니다.

때문에 더 구체적으로 `any` 타입을 사용하는 것이 중요합니다.

```tsx
function getLength(array: any[]) {
	return array.length
}
```

위 코드에서 `array` 의 타입으로 `any` 라고 할 수도 있지만 배열이라는 것을 명확히 하기 위해서

`any[]` 로 정의했습니다.

이 경우 다음과 같은 세가지 장점이 있습니다.

- 함수 내의 array.length 타입이 체크됩니다.
- 함수의 반환 타입이 any 대신 number 로 추론됩니다.
- 함수 호출될 때 매개변수가 배열인지 체크 가능합니다.

함수의 매개변수가 객체이지만 값을 알 수 없다면 다음과 같이 타입을 정의할 수도 있습니다.

```tsx
declare function someFunction(param: {[key: string]: any}): boolean;
```

또는 모든 비원시타입을 나타내는 `object` 타입을 사용할 수도 있지만 이 경우 인덱스 접근이 안됩니다.

```tsx
declare function someFunction(param: object): boolean;
```

## 함수 안으로 타입 단언문 감추기

`타입 단언문` 은 일반적으로 타입을 위험하게 만들 가능성을 가지고 있지만 

때에 따라서는 유용하고 현실적인 해결 방법으로 사용될 수 있습니다.

함수를 구현하다보면 모든 타입을 안전한 타입으로 정의하는 것이 불가능할 수 있습니다.

이 경우 함수 외부로 반환되는 값(반환값) 의 타입을 명시하고 함수 내부에서 타입 단언문으로

세부 구현을 숨기는 것이 현명한 판단이 될 수도 있습니다.

## any의 진화를 이해하기

`Typescript` 에서 일반적으로 변수의 타입은 선언시 결정되며 이후 확장되는 것이 불가능합니다.

다만 `any` 타입의 경우 특정 경우에 대해서 확장이 가능합니다.

이를 `any 타입의 진화` 라고 하며 `암시적 any 타입` 에 어떤 값을 할당할 때 발생합니다.

```tsx
let val = null; // any 타입
try {
	somethingDangerous();
	val = 12; // number 타입
} catch (e) {
	console.warn('alas!');
}
val // number | null 타입
```

다음처럼 타입을 명시적으로 `any` 타입으로 선언하면 타입이 유지됩니다.

```tsx
let val: any = null; // any 타입
try {
	somethingDangerous();
	val = 12; // any 타입
} catch (e) {
	console.warn('alas!');
}
val // any 타입
```

## 모르는 타입의 값에는 any 보다는 unknown 사용하기

`unknown` 타입과 `any` 타입의 차이점에 대해서 먼저 살펴보는 것으로 시작하겠습니다.

또한 추가적으로 `never` 타입에 대해서도 알아보겠습니다.

### `any` 타입

- 어떠한 타입이든 `any` 타입에 할당 가능합니다.
- `any` 타입은 어떠한 타입으로도 할당이 가능합니다.

### `unknown` 타입

- 어떠한 타입이든 `unknown` 타입에 할당 가능합니다.
- `unknown` 타입은 오직 `unknown` 타입과 `any` 타입에만 할당 가능합니다.

### `never` 타입

- 어떠한 타입도 `never` 에 할당할 수 없습니다.
- `never` 타입은 어떠한 타입으로도 할당 가능합니다.

타입을 값의 집합으로 생각하면 한 집합은 다른 모든 집합의 부분 집합이면서 

동시에 상위집합이 될 수 없기 때문에 `any` 타입 시스템과 상충됩니다.

`unknown` 타입인 채 그대로 사용하면 오류가 발생하기 때문에 적절한 타입으로 변환하도록 강제할 수 있습니다.

`unknown` 타입에는 함수의 반환값, 변수 선언, 단언문과 관련된 형태가 있습니다.

### 함수 반환값과 관련된 unknown

```tsx
const book = safeParseYAML(`
	name: Villette
	author: Charlotte Bronte
`) as Book;
```

함수 반환 타입인 `unknown` 을 그대로 사용할 수 없기 때문에 `Book` 타입으로 단언해줘야 합니다.

애초에 `safeParseYAML` 함수가 `Book` 타입을 반환할 것이라고 알고 있기 때문에 가능한 방법입니다.

### 변수 선언과 관련된 unknown

```tsx
interface Feature {
	properties: unknown;
}
```

어떠한 값의 타입을 모르는 경우 `any` 보다는 `unknown` 타입으로 정의하는게 좋습니다.

타입 단언문말고도 `unknown` 타입에서 원하는 타입으로 변환할 수 있는 방법이 있습니다.

먼저 `instanceof` 를 사용하는 방법입니다.

```tsx
function checkType(val: unknown) {
	if (val instanceof Date) {
		val // Date 타입
	}
}
```

그리고 `사용자 정의 타입 가드` 를 이용해서 `unknown` 에서 원하는 타입으로 변환 가능합니다.

다만 이 경우 조건을 신경써서 체크해야하는데, 우선 val 이 `object` 인지 파악하고

`null` 의 타입도 `object` 이기 때문에 `null` 인지도 체크해야합니다.

```tsx
function isBook(val: unknown): val is Book {
	return (
		typeof(val) === 'object' && val !== null && 'name' in val
	)
}
```

가끔 `unknown` 대신에 `제네릭` 을 사용할 지 고민하는 경우도 생기는데

사실 다음과 같이 제네릭을 사용하는 것은 타입 단언을 하는 것과 차이가 없습니다.

이 경우 제네릭보다는 `unknown` 을 반환하고 사용자가 직접 단언문을 사용하거나 

원하는 타입으로 좁히도록 하는게 더 좋습니다.

```tsx
function safeParseYAML<T>(yaml: string): T {
	return parseYAML(yaml)
}
```

### 단언문과 관련된 unknown

```tsx
declare const foo: Foo;

let barAny = foo as any as Bar;
let barUnk = foo as unknown as Bar;
```

위 두가지 단언문은 기능적으로 동일하지만 

나중에 두 개의 단언문을 분리하는 것을 생각하면 후자가 더 좋은 방법입니다.

`any` 타입은 분리되는 순간 그 영향력을 예측하기가 힘듭니다.

## 몽키패치보다는 안전한 타입 사용하기

Javascript 의 유연함 중 하나는 객체와 클래스에 임의의 속성을 추가할 수 있다는 것입니다.

종종 `document` 객체에 다음과 같이 전역 변수를 추가해서 사용할 수 있습니다.

이러한 것을 흔히 `몽키 패치` 라고 합니다.

```tsx
document.fruit = 'apple'
```

<aside>
💡 몽키 패치란?

프로그램이 런타임 되는 동안 사용되는 모듈이나 클래스를 변경하는 행동을 일컫습니다.

</aside>

사실 이러한 설계는 프로그램 구조에서 서로 멀리 떨어진 부분들 사이에서 의존성을 생성하게 됩니다.

또한 타입스크립트를 사용하고 있을 경우 `document` 객체는 

`fruit` 에 대한 존재를 알고 있지 않기 때문에 타입 에러가 발생하게 됩니다.

이 오류를 해결하는 가장 간단한 방법은 `any` 단언문을 사용하는 것입니다.

```tsx
(document as any).fruit = 'apple'
```

더 나은 방법은 `interface` 의 `보강` 기능을 사용하는 것입니다.

이를 통해 타입스크립트에서 제공하는 다양한 기능(자동완성, 타입 체크 등등) 을 활용할 수 있습니다.

```tsx
declare global {
	interface Document {
		fruit: string;
	}
}
```

만약 기존의 `Document` 모듈의 타입을 수정하는 것이 걱정된다면 

기존의 타입을 확장하여 보다 명확한 타입을 정의하는 방법도 있습니다.

```tsx
interface FruitDocument extends Document {
	fruit: string;
}

(document as FruitDocument).fruit = 'apple';
```

이렇게하면 기존의 `Document` 는 그대로 유지하면서 확장된 타입을 사용하기 때문에

모듈 영역 문제를 해결할 수 있으며 몽키 패치된 속성을 참조하는 경우에만 타입 단언문을 사용하면 됩니다.
