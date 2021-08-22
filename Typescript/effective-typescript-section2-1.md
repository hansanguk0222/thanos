# 타입스크립트의 타입 시스템

## 타입 단언보다는 타입 선언 사용하기

일반적으로 타입 단언이 꼭 필요한 경우가 아니라면 `타입 선언` 을 사용하는 것이 좋습니다.

`타입 단언` 의 경우 강제로 타입을 지정하는 것이기 때문에

다음과 같은 상황에서 문제가 생길 수 있습니다.

```tsx
interface Person {
	name: string;
}

const alice: Person = {}; // name 속성이 없다고 오류 발생
const bob = {} as Person; // 오류 없음
```

이는 잉여 속성 체크에 대해서도 동일합니다.

```tsx
const alice: Person = {
	name: 'Alice',
	occupation: 'Typescript developer'
}; // 오류 발생

const bob = {
	name: 'bob',
	occupation: 'Typescript developer',
} as Person; // 오류 없음
```

### 타입 단언이 필요한 경우

하지만 경우에 따라서 `타입 단언` 이 필요한 상황이 생길 수 있습니다.

`타입 단언` 은 타입 체커가 추론한 타입보다 개발자가 판단한 타입이 더 정확할 때 의미가 있습니다.

대표적으로 DOM 요소에 접근할 때가 그 예시입니다.

타입스크립트는 DOM 에 접근이 불가능하기 때문에 타입 단언문 사용이 타당합니다.

```tsx
document.querySelector('#myButton').addEventListener('click', e => {
	const button = e.currentTarget as HTMLButtonElement;
	// ...
})
```

또한 `타입 단언` 을 통해 null 이 아님을 명시할 수도 있습니다.

```tsx
const el = document.getElementById('foo')!;
```

## 객체 래퍼타입 피하기

JS 에는 기본형 값들을 좀 더 편하기 사용할 수 있도록 해주는 `객체 래퍼` 가 존재합니다.

기본형(primitive) 타입들은 불변하며 메서드를 가지지 않지만

JS 에서는 다음과 같이 사용이 가능한 것을 알 수 있습니다.

```tsx
'apple'.charAt(3)
```

이는 해당 메서드가 호출될 때 `String` 생성자를 통해서 기본형 타입을 `String` 객체로 래핑하고

메서드를 호출한 뒤, 마지막에 래핑한 객체를 버리기 때문입니다.

타입스크립트에서는 기본형과 객체 래퍼 타입을 별도로 모델링합니다.

따라서 타입을 명시할 때 객체 래퍼 타입을 사용하는 것에 유의합니다. (굳이 사용할 필요가 없음)
