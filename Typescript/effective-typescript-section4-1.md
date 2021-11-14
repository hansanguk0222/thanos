# 타입 설계 PART. 1

## 유효한 상태만 표현하는 타입 지향하기

효과적으로 타입을 설계하려면 유효한 상태만 표현할 수 있는 타입을 만들어 내는 것이 중요합니다.

만약 다음과 같이 상태를 설계한다면,

```tsx
interface State {
	pageText: string;
	isLoading: boolean;
	error?: string;
}
```

`isLoading` 과 `error` 가 모두 존재하는 `무효한` 상태가 생길 수 있습니다.

이를 개선한 타입 선언 방법은 다음과 같이 네트워크 요청 상태를 각각 분리하는 것입니다.

```tsx
interface RequestPending {
	state: 'pending';
}

interface RequestError {
	state: 'error';
	error: string;
}

interface RequestSuccess {
	state: 'ok';
	pageText: string;
}

type RequestState = RequestPending | RequestError | RequestSuccess;
```

이렇게 유효한 상태를 표현하는 값만 허용한다면 코드를 작성하기 쉬워지고 타입 체크가 용이해집니다.

## 문서에 타입 정보를 쓰지 않기

### 불필요한 주석 피하기

코드에 불필요한 정보를 주석으로 담는 것은 좋지 못한 습관입니다.

누군가 꾸준히 주석 내용까지 업데이트 해주지 않는 이상, 주석과 코드의 내용은 동기화 되지 않습니다.

```tsx
/**
 * 매개변수 idx 를 받아 멋진 작업을 수행하며 최종적으로 string 타입의 결과를 반환합니다.
 */
function someAwesomeFunction(idx: number): string {
	// ...
}
```

타입스크립트에는 타입 체커가 있기 때문에 이를 통해 타입 정보 동기화가 가능합니다.

또한 함수 시그니쳐를 통해 매개변수와 리턴 타입 또한 추론 가능하기 때문에 

주석에 해당 내용을 담는 것은 적절하지 않습니다.

만약 주석에 매개변수에 대한 추가적인 설명이 필요하다면 `JSDoc` 의 `@param` 을 사용하면 됩니다.

```tsx
/**
 * @param idx: 멋진 작업을 수행하기 위한 인덱스
 */
function someAwesomeFunction(idx: number): string {
	// ...
}
```

### 위 규칙은 변수 이름을 지을때도 적용됩니다.

변수명을 정의할 때 타입 정보를 포함하는 것은 좋지 못한 방법입니다.

```tsx
const ageNum = 10;
```

타입 추론으로 해당 변수가 `number` 타입이라는 것을 알 수 있기 때문에, 

굳이 `Num` 이라는 수식어를 붙여줄 필요가 없습니다.

대신 구체적인 단위를 필요로 하는 경우에는 함께 표시해주는 것이 좋습니다.

```tsx
const distanceKM = 10;
```

## 타입 주변에 null 값 배치하기

타입이 확실히 `null` 이거나 `null 이 아닌 경우`로 나뉜다면 섞여 있을 때보다 훨씬 다루기 쉽습니다.

타입에 `null` 을 추가하는 것으로 모델링 가능합니다.

### 숫자들의 최대 및 최소 값을 계산하는 함수 예시

매개변수로 숫자 배열이 주어질 때, 해당 배열에서 최대 및 최소값을 반환하는 함수가 

다음과 같이 정의되어 있다고 한다면,

```tsx
function extent(nums: number[]) {
	let min, max;

	for (const num of nums) {
		if (!min) {
			min = num;
			max = num;
		} else {
			min = Math.min(min, num);
			max = Math.max(max, num); // ERROR: Argument of type 'number | undefined' is not assignable to parameter of type 'number'.
		}
	}
}
```

`nums` 에 빈 배열이 전달되거나 `nums` 안에 0이 있을 경우 반환값에 문제가 생깁니다.

또한 `max` 에 대한 `null` 체크가 없기 때문에 타입 에러도 함께 발생합니다.

더 나은 구조는 반환값을 `[number, number]` 혹은 `null` 반환하도록 하는 것입니다.

```tsx
function extent(nums: number[]) {
	let result: [number, number] | null = null;

	for (const num of nums) {
		if (!result) {
			result = [num, num];
		} else {
			result = [Math.min(num, result[0]), Math.max(num, result[1])];
		}
	}

	return result;
}
```

이렇게 하면 반환 타입이 `[number, number] | null` 이기 때문에 다음과 같이 반환값을 얻을 수 있습니다.

```tsx
// 1. !(단언) 을 사용하기
const [min, max] = extent([0, 1, 2])!;

// 2. if 조건문 사용하기
const range = extent([0, 1, 2]);
if (range) {
	// ...
}

```

### 클래스에서 null 속성 지우기

클래스를 정의할 때 속성값은 `null` 의 가능성을 없애는 것이 버그를 줄이는 좋은 방법입니다.

만약 다음과 같이 `user` 와 `posts` 를 `null` 이 할당 가능하도록 정의한다면,

해당 속성을 사용하는 메서드는 모두 `null 타입 체크` 로직을 포함해야 할 것입니다.

```tsx
class UserPosts {
	user: UserInfo | null
	posts: Post[] | null

	constructor() {
		this.user = null;
		this.posts = null;
	}

	async init(userId: string) {
		return Promise.all([
			async () => this.user = await fetchUser(userId),
			async () => this.posts = await fetchPostsForUser(userId),
		])
	}

	getUserName() {
		return this.user.name // ERROR: Object is possibly 'null'.
	}
}
```

대신 다음과 같이 `init` 함수에서 비동기로 받아온 데이터를 생성자에게 넘겨주는 방식으로 정의한다면

클래스의 속성 `user` 와 `posts` 에 `null` 타입 가능성이 사라지기 때문에 해당 속성을 사용하는 메서드에서

`null 타입 체크` 가 필요하지 않게 됩니다.

```tsx
class UserPosts {
	user: UserInfo
	posts: Post[]

	constructor(user: UserInfo, posts: Post[]) {
		this.user = user;
		this.posts = posts;
	}

	async init(userId: string) {
		return Promise.all([
			async () => this.user = await fetchUser(userId),
			async () => this.posts = await fetchPostsForUser(userId),
		])
	}

	getUserName() {
		return this.user.name
	}
}
```
