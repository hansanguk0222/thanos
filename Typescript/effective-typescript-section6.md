# 타입 선언과 @types

## 타입 선언과 관련된 세 가지 버전 이해하기

타입스크립트를 사용할 때는 다음 세가지 사항을 고려해야합니다.

이들 중 하나라도 맞지 않으면 호환성 문제가 발생할 수 있습니다.

- 라이브러리 버전
- 타입스크립트 버전
- 타입 선언(@types) 버전

### 타입스크립트 사용 시 일반적으로 의존성을 관리하는 방법

대게 typescript 사용 시 필요한 라이브러리는 `dependencies` 에 설치하고

그에 대한 타입 정보는 `devDependencies` 에 설치하는 것이 일반적입니다.

이렇게 별도의 타입 정보를 담은 모듈들은 `DefinitelyTyped` 에서 공개되고 관리되고 있습니다.

이러한 방식은 다음과 같은 문제점이 생길 수 있습니다.

- 라이브러리는 업데이트 했지만 타입 선언은 업데이트 하지 않는 경우
    - 이 경우 `보강` 기법을 사용하거나 사용하려는 메서드의 타입 정보를
        
        프로젝트에 추가하는 방식으로 해결 가능
        
- 라이브러리보다 타입 선언의 버전이 최신인 경우
    - 라이브러리 버전을 올리거나 타입 선언의 버전을 내리는 것으로 해결
- 프로젝트에서 사용하는 TS 버전보다 라이브러리에서 필요로 하는 TS 버전이 최신인 경우
    - 프로젝트의 타입 버전을 올리거나 타입 선언의 버전을 내리는 것으로 해결
    - 혹은 `declare module` 로 라이브러리의 타입 정보를 제거
- 서로 다른 타입 선언에서 사용하는 @types 의존성이 중복되는 경우
    - 서로 버전이 호환되도록 타입 선언의 버전을 업데이트

### 라이브러리 자체적으로 타입 선언을 가지고 있는 경우

일부 라이브러리의 경우 자체적으로 타입 선언을 가지고 있습니다.

이를 `번들링 방식` 이라고 하며 이는 다음과 같은 문제점을 가지고 있습니다.

- 번들된 타입에 보강 기법으로 해결 불가능한 오류가 있는 경우 TS 버전이 올라가면서 문제가 발생
- 프로젝트내의 타입이 다른 라이브러리의 타입에 의존성을 가지고 있다면 문제 발생 가능
    - 보통의 의존성은 `devDependencies` 에 들어가기 때문인데,
        
        JS 사용자는 타입 선언 모듈이 필요없기 때문에 `dependencies` 에 넣고싶어 하지 않음
        
        이를 해결하기 위해 `미러 타입` 을 사용할 수도 있음
        
- 프로젝트의 과거 버전에 있는 타입 선언에 문제가 있다면 과거 버전으로 돌아가서 패치해야함
- 타입 선언의 패치를 자주하기 힘듬

### 타입 선언 버전을 관리하는 공식적인 권장사항

1. 라이브러리가 Typescript 로 작성된 경우에만 타입 선언을 라이브러리에 함께 번들링하기
2. Javascript 로 작성된 라이브러리라면 `DefinitelyTyped` 에 공개하여 커뮤니티가 관리하도록 하기

## 공개 API에 등장하는 모든 타입 export 하기

가끔씩 사용성을 위해 불필요한 외부 `export` 를 막는 경우도 존재합니다.

```tsx
interface SecretName {
	first: string;
	last: string;
}

interface SecretSanta {
	name: string;
	gift: string;
}

export function getName(name: SecretName): SecretSanta { /** */ }
```

하지만 `Parameters` 나 `ReturnType` 과 같은 유틸리티 타입을 사용하면 

함수 시그니처를 추출할 수 있기 때문에 의미가 없어집니다.

따라서 굳이 숨기지 말고 라이브러리 사용자를 위해 명시적으로 `export` 하는 것이 좋습니다.

```tsx
type Santa = ReturnType<typeof getName>   // SecretSanta
type Name = Parameters<typeof getName>[0] // SecretName
```

## 콜백에서 this 에 대한 타입 제공하기

### this 사용 시 주의할 점

Javascript 에서 `this` 는 호출된 방식에 따라서 다른 값을 가지게 됩니다.

특히 `콜백 함수` 내에서 이를 사용할 때는 각별한 주의가 필요한데요, 그 중 `class` 의 메서드를

특정 이벤트의 콜백 함수로 사용하는 경우 다음과 같이 해결이 가능합니다.

```tsx
class ResetButton {
	constructor() {
		this.onClick = this.onClick.bind(this);
	}

	onClick() {
		console.log('reset');
	}
}
```

`bind` 를 호출하면 `this` 객체가 바인딩된 새로운 함수가 반환되어 

`ResetButton` 의 프로토타입 객체가 아닌 인스턴스에 생성되게 됩니다.

혹은 다음과 같이 화살표 함수를 사용해도 됩니다.

```tsx
class ResetButton {
	onClick = () => {
		console.log('reset');
	}
}

/**
위는 다음과 같은 코드로 변환됩니다.

class ResetButton {
	constructor() {
		var _this = this;
		this.onClick = function () { console.log('reset'); }
	}
}
*/
```

### Typescript 에서 콜백 함수에 this 타입 부여하기

Typescript 에서는 콜백 함수의 매개변수에 `this` 를 추가하면 `this 바인딩` 이 체크됩니다.

이를 통해 사전에 `this 바인딩 이슈` 를 방지할 수 있습니다.

```tsx
function addKeyListener(
	el: HTMLElement, 
	fn: (this: HTMLElement, e: KeyboardEvent
) => void) {
	el.addEventListener('keydown', e => { fn.call(el, e); })
} 
```

## 오버로딩 타입보다는 조건부 타입을 사용하기

### Typescript 에서의 함수 오버로딩

Typescript 에서는 `오버로딩` 이라는 기능을 제공합니다.

먼저 다음과 같이 값을 2배로 만들어주는 `double` 이라는 함수를 정의하겠습니다.

```tsx
function double(x) {
	return x + x;
}
```

만약 인자로 `number` 타입과 `string` 타입을 받고 싶다면 어떻게 해야할까요?

다음과 같이 유니온 타입을 함께 정의할 수 있을 것입니다.

```tsx
function double(x: number | string): number | string;
function double(x: any) { return x + x; }
```

하지만 이렇게 유니온 타입으로 묶어버리면 다음과 같이 타입이 애매한 상황이 생기게 됩니다.

`number` 타입을 인자로 주었는데 반환된 타입이 `number | string` 이 되는 것이죠

```tsx
const num = double(12); // string | number
```

또다른 방법으로 `제네릭` 을 사용하는 것이 있을 수 있습니다.

하지만 이 경우에는 타입이 너무 구체적인 탓에 리터럴 타입으로 추론되고 있습니다.

```tsx
function double<T extends string | number>(x: T): T 
function double(x: any) { return x + x; }

const num = double(12);  // 12
const str = double('x'); // 'x'
```

그러면 이를 위해 명확한 두 타입으로 아얘 함수를 오버로딩 하면 어떨까요?

다음과 같이 `string` 과 `number` 타입으로 함수 시그니처를 각각 정의합니다.

하지만 이 경우에는 `string | number` 라는 유니온 타입을 할당하지 못하는 문제가 있습니다.

```tsx
function double(x: number): number;
function double(x: string): string;
function double(x: any) { return x + x; }

const num = double(12);  // number
const str = double('x'); // string

function f(x: number | string) {
    return double(x) // ERROR
} 
```

`string | number` 를 추가하여 세번째 오버로딩을 활용하는 방식으로 해결할 수도 있지만,

더 좋은 방법은 `조건부 타입` 을 사용하는 것입니다.

### 조건부 타입

조건부 타입은 JS 의 삼항 연산자와 같은 방식으로 사용합니다.

위 예제에서는 `x` 가 number 일때는 number, string 일때는 string 을 반환하도록 하면 됩니다.

```tsx
function double<T extends number | string>(x: T): T extends string ? string : number;
function double(x: any) { return x + x; } 
```

유니온 타입에 조건부 타입을 적용하면 조건부 타입의 유니온으로 분리되기 때문에 올바르게 동작합니다.

위 예제에서는 다음과 같은 과정으로 타입을 해석합니다.

```tsx
(number | string) extends string ? string : number

-> (number extends string ? string : number) | (string extends string ? string : number)
-> number | string
```

따라서 오버로딩과 달리 조건부 타입은 유니온 문제를 해결할 수 있기 때문에

오버로딩을 고려하고 있다면 조건부 타입을 통해 타입을 정의할 수는 없는지 고민해볼 필요가 있습니다.

## 의존성 분리를 위해 미러 타입 사용하기

만약 작성중인 라이브러리가 의존성을 가지고 있는 라이브러리의 구현체와 무관하게

단순히 타입 정보만을 필요로 하는 경우에는 필요한 선언부만 추출하는 `미러링` 을 고려해볼 수 있습니다.

`미러링` 은 `구조적 타이핑` 을 적용하는 것을 의미하며 다음과 같은 경우에 사용합니다.

먼저 CSV 파일을 파싱하는 함수를 구현했다고 하겠습니다.

```tsx
function parseCSV(data: string | Buffer) { /** */ }
```

이 경우 `Buffer` 타입을 허용하기 위해 라이브러리 사용자는

해당 타입 정의를 포함한 `@types/node` 모듈을 `devDependencies` 에 포함해야 합니다. 

하지만 이 경우 다음 두 사용자들에게 문제가 됩니다.

1. @types 와 무관한 JS 개발자
2. Node 와 무관한 TS 개발자

오직 `@types/node` 는 TS 를 사용하는 NodeJS 개발자에게만 의미가 있는 모듈인 것입니다.

이러한 불필요한 모듈 선언을 피하기 위해서 `구조적 타이핑` 을 사용할 수 있습니다.

바로 필요한 선언부만 직접 구현해서 프로젝트에 추가하는 것입니다.

```tsx
function CSVBuffer {
	toString(encodeing: string): string;
}
function parseCSV(contents: string | CSVBuffer) { /** */ }
```

이렇게 구현해도 실제 `Buffer` 타입과 호환되기 때문에 문제가 없고 훨씬 간결하게 해결 가능합니다.

그러나 다른 라이브러리의 타입 선언 대부분을 추출해야하면 

그냥 별도의 `@types` 를 의존성으로 추가하는 것이 좋을 수도 있습니다.
