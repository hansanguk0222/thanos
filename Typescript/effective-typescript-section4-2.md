# 타입 설계 PART.2

## 유니온의 인터페이스보다는 인터페이스의 유니온 사용하기

유니온 타입을 속성으로 가지는 인터페이스를 사용할 경우

인터페이스의 유니온을 사용하는 것이 더 알맞은지 검토해야할 경우도 존재합니다.

기하학적인 타입을 가지는 계층의 인터페이스를 정의한다고 하겠습니다.

```tsx
interface Layer {
	layout: FillLayout | LineLayout | PointLayout;
	paint: FillPaint | LinePaint | PointPaint;
}
```

여기서 각각의 `Layout` 과 `Paint` 는 동일하게 매칭이 되어야 합니다.

(가령 `FillLayout` 과 `LinePaint` 가 함께 사용되는 것은 오류입니다.)

이러한 경우 `layout` 과 `paint` 를 유니온 타입으로 정의하는 것보다는

유효한 타입을 만들기 위해 각각을 인터페이스로 분리하여 정의하는 것이 좋습니다.

이렇게 하면 잘못된 속성이 섞이는 것을 방지할 수 있습니다.

```tsx
interface FillLayer {
	layout: FillLayout;
	paint: FillPaint;
}

interface LineLayer {
	layout: LineLayout;
	paint: LinePaint;
}

interface PointLayer {
	layout: PointLayout;
	paint: PointPaint;
}

type Layer = FillLayer | LineLayer | Point
```

여기에 `태그된 유니온` 을 사용하면 보다 타입을 명확히 할 수 있습니다.

```tsx
interface FillLayer {
	type: 'fill';
	layout: FillLayout;
	paint: FillPaint;
}

interface LineLayer {
	type: 'line';
	layout: LineLayout;
	paint: LinePaint;
}

interface PointLayer {
	type: 'point';
	layout: PointLayout;
	paint: PointPaint;
}
```

또한 다음과 같이 여러 개의 선택적 필드가 존재하고 서로 연관이 있는 상황이라면,

```tsx
interface Person {
	name: string;
	// 다음 두 필드는 동시에 있거나 동시에 없습니다.
	placeOfBirth?: string;
	dateOfBirth?: Date;
}
```

주석을 사용해서 표현하는 것보다는 다음과 같이 하나의 속성으로 묶는 것이 더 좋습니다.

```tsx
interface Person {
	name: string;
	birth?: {
		placeOfBirth: string;
		dateOfBirth: Date;
	}
}
```

## string 타입 보다는 더 구체적인 타입 사용하기

### 무분별한 string 타입은 어쩌면 any 타입과 같이 위험할 수 있습니다.

`string` 타입의 범위는 매우 넓기 때문에 `string` 타입을 남발하는 것은

`any` 타입을 남발하는 것과 동일합니다.

만약 다음과 같은 `Album` 인터페이스를 정의한다면,

```tsx
interface Album {
	artist: string;
	title: string;
	releaseDate: string;  // YYYY-MM-DD
	recordingType: string;  // live 혹은 studio
}
```

`releaseDate` 와 `recordingType` 에 어떠한 문자열이 들어와도 문제가 안생깁니다.

이는 곧 버그로 이어지기 때문에 느슨한 타이핑보다는 

`string` 을 대체할 수 있는 구체적인 타입이 더 좋습니다.

```tsx
type RecordingType = 'studio' | 'live';

interface Album {
	artist: string;
  title: string;
	releaseDate: Date;
	recordingType: RecordingType;
}
```

### 함수 매개변수를 string 타입으로 정의하는 잘못된 예시

어떤 배열에서 한 필드의 값만 추출하는 함수 `pluck` 를 다음과 같이 정의하겠습니다.

```tsx
function pluck(records: any[], key: string) {
	return records.map(r => r[key]);
}
```

위와 같은 함수 시그니쳐보다는 다음과 같이 `제네릭 타입` 과 `keyof` 연산자를 사용하여

타입을 정의하는 것이 보다 정확한 타입을 얻을 수 있는 방법입니다.

```tsx
function pluck<T, K extends keyof T>(records: T[], key: K) {
  return records.map(r => r[key]);
}
```

여기서 `extends` 를 사용한 이유는 여려개의 필드를 아우르는 타입 보다 특정 키에 해당하는

타입을 명시하기 위해 `keyof T` 의 부분집합에 해당하는 타입 `K` 를 정의한 것입니다.

## 부정확한 타입보다는 미완성 타입 사용하기

정확하게 타입을 모델링 할 수 없다면 부정확하게는 모델링 하지 말아야 합니다.

또한 타입을 정의할 때 너무 과하게 정밀하게 정의할 경우에도 

오히려 코드가 더 부정확해지는 경우도 있습니다.

## 해당 분야의 용어로 타입 이름 짓기

### 모호한 이름 사용하지 않기

타입의 형태를 정의하는 것 만큼 중요한 것이 명확한 이름을 정의하는 것입니다.

잘못된 타입 이름은 코드의 의도를 왜곡되게 해석할 여지를 부여합니다.

```tsx
interface Animal {
	name: string;
	endangered: boolean;
	habitat: string;
}
```

위 타입은 다음과 같은 세 가지 문제점이 있을 수 있습니다.

- `name` 은 동물의 학명을 나타내기에는 너무 일반적인 명칭입니다.
- `endangered` 는 이미 멸종된 동물을 `true` 로 표현하기 애매합니다.
- `habitat` 은 `string` 타입으로 너무 광범위한 타입입니다.

이러한 문제를 해결하기 위해서는 실제 학계에서 사용하는 용어를 사용하는 것이 좋습니다.

또한 `data, info, entity` 와 같이 모호한 용어는 피하고 데이터 자체가 의미하는

용어를 부여하는 것에 초점을 맞춰야 합니다.

[An Introduction to Nominal TypeScript](https://betterprogramming.pub/nominal-typescript-eee36e9432d2)

[Nominal Typing](https://basarat.gitbook.io/typescript/main-1/nominaltyping)
