# 타입 스크립트의 타입 시스템 Part.3

## number 인덱스 시그니처보다는 Array, 튜플, ArrayLike 사용하기

자바스크립트에서는 Array 의 인덱스로 number 타입을 사용할 수 있습니다.

```tsx
const arr = [0, 1, 2];

arr[0] === 0
```

하지만 실제 런타임에서는 인덱스 시그니처로 사용된 `0` 은 문자열로 변경되어 사용됩니다.

일반적으로 `string` 대신 `number` 타입을 인덱스 시그니처로 사용할 일이 많지 않습니다.

이 때문에 불필요한 오해를 피하기 위해 숫자를 사용하여 인덱스를 지정해야 한다면

`Array` 또는 튜플 타입을 사용하는 것이 좋습니다.

또한 길이를 가지는 유사 배열과 같은 타입을 지정하고 싶은 경우 `ArrayLike` 를 사용할 수도 있습니다.

```tsx
function get<T>(array: ArrayLike<T>, k: number): T {
    if (k < array.length) {
        return array[k]
    }

    throw new Error('배열의 범위를 벗어나는 접근')
}
```

## 변경 관련된 오류 방지를 위해 readonly 사용하기

### readonly 를 활용한 간단한 예시

```tsx
function arraySum(arr: number[]) {
	let sum = 0, num;
	
	while ((num = arr.pop()) !== undefined) {
		sum += num;
	}

	return sum;
}

function printTriangles(n: number) {
	const nums = [];

	for (let i = 0; i < n; i++) {
		nums.push(i);
		console.log(arraySum(nums));
	}
}
```

위 함수는 삼각수를 출력하는 코드입니다.

하지만 위와 같이 코드를 작성하면 `printTriangles` 호출 시 

우리가 예상된 결과값을 출력하지 못하는 것을 알 수 있습니다.

이는 `arraySum` 의 인자로 받은 `arr` 가 객체에 대한 참조값을 전달 받아서 해당 객체에 대해

`pop` 메서드를 호출하기 때문입니다.

이를 사전에 방지하기 위해서는 `arr` 가 읽기 전용으로 선언되어 있으면 되는데

타입스크립트에서는 이를 `readonly` 키워드로 선언합니다.

```tsx
function arraySum(arr: readonly number[]) {
	let sum = 0;
	
	for (const num of arr) {
		sum += num;
	}

	return sum;
}
```

변경된 `arraySum` 은 위와 같으며 `arr` 가 `readonly` 로 선언되었기 때문에 

배열을 변경하는 `pop` 메서드를 사용하지 못합니다.

암묵적인 방법 대신에, 위와 같이 명시적으로 타입을 선언해준다면 

런타임 이전에 예상치 못한 에러를 방지할 수 있습니다.

### readonly는 얕게 동작한다

만약 객체의 `readonly` 배열이 있다면 그 객체 자체는 `readonly` 가 아닙니다.

```tsx
const dates: readonly Date[] = [new Date()];

dates.push(new Date()); // ERROR: dates 는 읽기 전용 속성이기 때문에

dates[0].setFullYear(2037);
```

또한 이는 유틸리티 타입인 `Readonly` 에 동일하게 적용됩니다.

```tsx
interface Outer {
	inner: {
		x: number;
	}
}

const o: Readonly<Outer> = { inner: { x: 0 } };

o.inner = { x: 1 }; // ERROR: o.inner 는 읽기 전용 속성이기 때문에

o.innner.x = 1;
```

현재 `깊은 readonly 타입` 은 기본적으로 지원되지 않고 있기 때문에 `ts-essential` 라이브러리에서

제공하고 있는 `DeepReadonly` 타입을 사용하거나 다음 코드를 활용합니다.

```tsx
type DeepReadonly<T> =
    T extends (infer R)[] ? DeepReadonlyArray<R> :
    T extends Function ? T :
    T extends object ? DeepReadonlyObject<T> :
    T;

interface DeepReadonlyArray<T> extends ReadonlyArray<DeepReadonly<T>> {}

type DeepReadonlyObject<T> = {
    readonly [P in keyof T]: DeepReadonly<T[P]>;
};
```

[DeepReadonly Object Typescript](https://stackoverflow.com/questions/41879327/deepreadonly-object-typescript/49670389#49670389)

## 매핑된 타입을 사용하여 값을 동기화하기

매핑된 타입을 잘 활용했을 경우의 장점을 알아보기 위해서 산점도를 그리는 컴포넌트의

`update` 함수를 작성한다고 가정하겠습니다.

우선 산점도는 다음과 같은 속성을 가집니다.

```tsx
interface ScatterProps {
	xs: number[];
	ys: number[];

	xRange: [number, number];
	yRange: [number, number];
	color: string;
	
	onClick: (x: number, y: number, index: number) => void;
}
```

매핑된 타입을 활용하면 한 객체가 또 다른 객체와 정확히 같은 속성을 가지도록 강제할 수 있습니다.

```tsx
const REQUIRES_UPDATE: { [k in keyof ScatterProps]: boolean } = {
	xs: true,
	xy: true,
	xRange: true,
	yRange: true,
	color: true,
	onClick: false,
};

function shouldUpdate(
	oldProps: ScatterProps,
	newProps: ScatterProps,
) {
	let k: keyof ScatterProps;

	for (k in oldProps) {
		if (oldProps[k] !== newProps[k] && REQUIRES_UPDATE[k]) {
			return true;
		}
	}

	return false;
}
```

위 코드에서 `REQUIRES_UPDATE` 의 타입으로 사용된 `[k in keyof ScatterProps]` 는

`REQUIRES_UPDATE` 가 ScatterProps 의 모든 속성을 가져야 한다는 것을 명시합니다.

따라서 이후 ScatterProps 에 새로운 속성이 추가되더라도 타입 체크 덕분에 

REQUIRES_UPDATE에서 새로운 속성에 대한 업데이트 유무를 정의하는 것을 잊지 않을 수 있습니다.
