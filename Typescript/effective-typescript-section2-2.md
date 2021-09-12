# 타입스크립트의 타입 시스템 Part.2

## 잉여 속성 체크의 한계

타입스크립트는 해당 타입의 속성이 있는지, 그 외 속성은 없는지 확인합니다.

```tsx
interface Room {
	numDoors: number;
	ceilingHeightFt: number;
}

const r: Room = {
	numDoors: 1,
	ceilingHeightFt: 10,
	elephant: 'present' // error
}

const obj = {
	numDoors: 1,
	ceilingHeightFt: 10,
	elephant: 'present'
}

const r: Room = obj;
```

구조적 타이핑 관점에서 보면 위와 같은 동작이 이상하게 느껴질 수 있는데

이는 `잉여 속성 체크` 와 `할당 가능 검사` 는 별도의 과정이기 때문에 이러한 식으로 동작합니다.

`잉여 속성 체크` 를 통해 의도와는 다르게 작성한 코드를 찾을 수 있습니다.

즉 객체 리터럴을 변수에 할당하거나 함수의 매개변수로 전달할 때 잉여 속성 체크가 발생합니다.

## 함수 표현식에 타입 적용하기

`JS` 에는 함수 표현식과 함수 선언문이 존재합니다.

`TS` 에서는 함수 표현식을 사용하는 것이 유리한데, 이는 해당 함수의 매개변수 및 반환값 들을

함수 타입으로 선언하여 재활용할 수 있기 때문입니다.

```tsx
type BinaryFn = (a: number, b: number) => number;

const add: BinaryFn = (a, b) => a + b;
const sub: BinaryFn = (a, b) => a - b;
const mul: BinaryFn = (a, b) => a * b;
const div: BinaryFn = (a, b) => a / b;
```

## 타입과 인터페이스의 차이점 알기

### 타입과 인터페이스의 공통점

1. 추가 속성 할당이 이루어진다면 오류가 발생한다.
2. 인덱스 시그니처를 사용할 수 있다.
3. 제네릭 사용이 가능하다.
4. 클래스를 구현할 때 타입과 인터페이스를 둘 다 사용할 수 있다.

### 타입과 인터페이스의 차이점

1. 인터페이스는 타입을 확장 할 수 있지만 유니온은 할 수 없습니다.

    ```tsx
    type Input = { /* ... */ }
    type Output = { /* ... */ }

    type NamedVariable = (Input | Output) & { name: string; }
    ```

1. 인터페이스로 튜플과 비슷하게 구현하면 튜플에서 사용할 수 있는 메서드들을 사용할 수 없습니다.

```tsx
interface Tuple {
	0: number;
	1: number;
	length: 2;
}

const t: Tuple = [10, 20];
```

1. 인터페이스에는 타입에는 없는 `보강` 기능이 있습니다.

```tsx
interface IState {
	name: string;
	capital: string;
}

interface IState {
	population: number;
}

const wyoming: IState = {
	name: 'aa',
	capital: 'bb',
	population: 22
}
```

위와 같이 속성을 확장하는 것을 `선언 병합` 이라고 하며 주로 타입 선언 파일에서 사용됩니다.

따라서 타입 선언 파일에서는 선언 병합을 지원하기 위해 반드시 인터페이스를 사용해야 합니다.

### 언제 어떤것을 사용해야 할까?

- 복잡한 타입 → 타입 별칭
- 두 가지 방법으로 모두 가능한 간단한 객체 → 일관성과 보강의 관점에서 고민하기
- API 에 대한 타이핑이라면 → 보강을 고려해 인터페이스 사용하기

## 타입 연산과 제네릭 사용으로 반복 줄이기

### 반복을 줄이는 유틸리티 타입

타입을 정의할 때 `DRY` 를 피하기 위해서 타입스크립트 표준 라이브러리에서 제공하는

여러가지 유틸리티 타입들을 활용할 수 있습니다.

**Pick**

`Pick` 은 매핑된 타입을 사용하도록 도와주는 유틸리티 타입입니다.

해당 유틸리티 타입은 다음과 같은 상황에 유용합니다.

```tsx
interface State {
	userId: string;
	pageTitle: string;
	recentFiles: string[];
	pageContents: string;
}

// 아래와 같이 하는 것 보다는
interface TopNavState {
	userId: string;
	pageTitle: string;
	recentFiles: string[];
}

// 이렇게 매핑된 타입을 사용할 수도 있고
type TopNavState = {
	[k in 'userId' | 'pageTitle' | 'recentFiles' ]: State[k]
}

// Pick 을 이용해서 나타낼 수도 있음
type TopNavState = Pick<State, 'userId' | 'pageTitle' | 'recentFiles'>;
```

**Partial**

만약 특정 타입의 일부 필드만 타입으로 재사용하고 싶다면 `Partial` 을 사용하면 됩니다.

```tsx
interface State {
	userId: string;
	pageTitle: string;
	recentFiles: string[];
	pageContents: string;
}

// 다음 코드는
type Optional = Partial<State>

// 아래 코드와 동일한 역할
type Optional = {
	[k in keyof State]?: State[k]
}
```

**ReturnType**

특정 함수의 반환 타입을 가져오고 싶다면 `ReturnType` 을 사용하면 됩니다.

```tsx
declare function f1(): { a: number, b: string }
type T4 = ReturnType<typeof f1>;  // { a: number, b: string }
```

### 그외 몇가지 팁들

**제네릭 타입에서 매개변수 제한하기**

제네릭에서 매개변수를 제한하기 위해서는 `extends` 를 사용하면 됩니다.

이를 통해 제네릭 매개변수가 특정 타입을 확장한다고 선언하게 됩니다.

```tsx
interface Name {
	first: string;
	last: string;
}

type DancingDuo<T extends Name> = [T, T];

const couple: DancingDuo<Name> = [
	{ first: 'fred', last: 'astaire' },
	{ first: 'ginger', last: 'rogers' } 
]
```

**이미 존재하는 타입 확장하기**

이미 존재하는 타입을 확장하는 첫 번째 방법은 `extends` 를 활용하는 것입니다.

```tsx
interface Person {
	name: string;
}

interface PersonWithBirthday {
	name: string;
	birth: Date;
}
```

두번째 방법은 `&` 연산자를 활용하는 것입니다.

이 방법은 유니온 타입(`extends` 를 사용할 수 없음) 을 확장할 때 유용합니다.

```tsx
type PersonWithBirthday = Person & { birth: Date }
```
