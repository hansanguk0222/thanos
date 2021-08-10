# 타입스크립트 알아보기

## 타입스크립트란?

타입스크립트는 자바스크립트의 상위 집합입니다.

따라서 모든 자바스크립트 프로그램은 이미 타입스크립트 프로그램입니다.

반대로 타입스크립트는 별도의 문법을 가지고 있기 때문에 일반적으로 유효한 자바스크립트 프로그램이 아닙니다.

타입스크립트는 자바스크립트의 런타임 동작을 모델링하는 타입 시스템을 가집니다.

따라서 자바스크립트의 런타임 에러를 발생시키는 코드를 찾아냅니다.

또한 타입스크립트의 타입 시스템은 자바스크립트의 동작을 모델링하지만 

자바스크립트에서는 허용되고 타입스크립트에는 허용되지 않는 문제가 발생하는 경우도 있습니다.

## 코드 생성과 타입이 관계없음을 이해하기

타입스크립트 컴파일러는 다음 두 가지 일을 합니다.

- 최신 타입스크립트 및 자바스크립트를 구버전의 자바스크립트로 `트랜스파일` 합니다.
- 코드의 타입 오류를 검사합니다.

여기서 주의할 점은 이 두 작업이 완전히 독립적이라는 것입니다.

따라서 자바스크립트의 런타임에 타입은 영향을 주지 않습니다.

그렇기에 타입 오류가 있는 코드도 컴파일이 가능합니다.

### 런타임에는 타입 체크가 불가능하다.

이번에 책을 읽으면서 가장 공감이 갔던 부분입니다.

실제로 타입스크립트를 작성할 때 다음과 같은 실수를 했었습니다.

```tsx
interface Square {
	width: number;
}

interface Rectangle extends Square {
	height: number;
}

type Shape = Square | Rectangle;

function calculate(shape: Shape) {
	if (shape instancneof Rectangle) {
		return shape.width * shape.heigth; // ... 타입 에러! (Shape 에는 height 없음)
	}
}
```

위와 같은 상황이 발생하는 이유는 `instanceof` 가 런타임에 체크되기 때문입니다.

`Rectangle` 은 타입이기 때문에 런타임에 아무 영향도 줄 수 없습니다.

이 문제를 해결하는 세 가지 방법이 있습니다.

**속성  체크하기**

```tsx
function calculate(shape: Shape) {
	if ('height' in shape) {
		return shape.width * shape.heigth; 
	}
}
```

**태그된 유니온 사용하기**

`태그된 유니온` 기법을 사용해서 런타임에 타입 정보를 유지할 수 있습니다.

```tsx
interface Square {
	kind: 'square';
	width: number;
}

interface Rectangle {
	kind: 'rectangle';
	height: number;
	widht: number;
}

type Shape = Square | Rectangle;

function calculate(shape: Shape) {
	if (shape.kind === 'rectangle') {
		// ...
	}
}
```

**타입과 값을 함께 사용하기**

`클래스` 로 구현할 경우 타입과 값으로 모두 사용 가능하기 때문에 오류가 발생하지 않습니다.

```tsx
class Square {}

class Rectangle extends Square {}

type Shape = Square | Rectangle;

function calculate(shape: Shape) {
	if (shape instanceof Rectangle) {
		// ...	
	}
}
```

## 구조적 타이핑(덕 타이핑)

자바스크립트는 본질적으로 덕 타이핑 기반입니다.

따라서 어떤 함수의 매개변수 값이 모두 제대로 주어지면 문제없이 실행됩니다.

타입스크립트는 이러한 자바스크립ㅌ의 동작 방식을 그대로 모델링합니다.

다음은 덕 타이핑의 한 예시입니다.

```tsx
interface Vector {
	x: number;
	y: number;
}

function calculate(v: Vector) {
	// ...
}

interface Vector2 {
	name: string;
	x: number;
	y: number;
}

cosnt v2: Vector2 = { x: 1, y: 2, name: 'v2' }
calculate(v2);  // 문제 없음
```

이러한 특징 때문에 타입 체커가 제대로 문제를 잡아내지 못할 때도 있지만

테스트를 작성할 때는 모킹을 위해 유리할 수도 있습니다.

## any 타입 지양하기

`any` 타입은 타입 체커를 무시할 수 있는 강력한 도구입니다.

하지만 이 타입을 사용할 경우 타입스크립트의 많은 장점을 이룰 수 없기 때문에 주의하여 사용해야 합니다.

다음은 `any` 타입 사용 시 주의해야할 사항들입니다.

- `any` 타입은 안정성이 없습니다.
- `any` 타입은 함수 시그니처를 무시합니다.
- `any` 타입을 사용할 경우 타입스크립트가 제공하는 편의 기능을 사용할 수 없습니다.
- `any` 타입은 리팩토링시 버그를 감추고 타입 설계를 감출 수 있습니다.
- `any` 타입은 타입 신뢰도를 떨어뜨립니다.
