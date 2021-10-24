# 왜 Promise를 써야 하는가?

1. 콜백을 이용하는 방식은 콜백 지옥이라는 문제를 만든다.
2. 코드의 동작 순서를 이해하기가 어렵다.
    1. 이는, 오류를 다음 실행으로 전달하는 것을 잊게 만들 수도 있어서 위험하다.
3. 특정 경우에 따라 동기, 비동기가 나뉘는 함수가 있을 수 있다.

그래서 등장한 해법으로 **Promise**가 나왔다.

Promise는 상태를 전달하는 객체로, 비동기식 작업의 최종 결과를 나타낸다.

프라미스는 비동기 코드를 단순하게 만들고, 비동기를 동기처럼 보이도록 Async/Await를 쓴다.

> Promise는 비동기 작업의 결과를 담은 객체로, 비동기가 완료되지 않은 경우에는 대기중(pending), 성공적으로 끝난 경우에는 이행됨(fulfiled), 작업이 에러와 함께 종료되었을 때에는 거부됨(rejected)라고 한다. 프라미스가 이행되거나 거부된 것을 합쳐 결정된(settled) 것이라고 한다.
> 

아래는 사용법이다.

```jsx
asyncOperation(arg, (err, result) => {
	if(err) {
		// 에러 처리
	}
	// 결과 처리
})
```

```jsx
promise.then(onFulfiled, onRejected)
```

```jsx
asyncOperationPromise(arg).then(res => {}, err => {});
```

Promise가 이행되거나 거부된 경우, 위 Promise의 인스턴스의 then() 함수를 이용할 수 있다.

또한 err를 처리할 수도 있다.

결과적으로 첫 번째의 콜백 방식은 Promise를 통해 맨 아래의 코드처럼 다룰 수 있게 되었다.

# Promise/A+와 thenable

> Promise/A+ 표준을 채택한 결과, Native JavaScript 프라미스의 API를 포함한 많은 프라미스 구현들은 then() 함수가 있는 모든 객체를 thenable 이라는 Promise와 유사한 객체로 간주합니다. 이 동작을 통해 서로 다른 프라미스 구현들이 서로 원활하게 연결될 수 있습니다.
> 

## Promise API

아래는 Promise가 가진 Static method들이다.

```jsx
Promise.resolve(obj); // thenable을 생성한다.
Promise.reject(err); // err를 이유로 거부하는 Promise를 생성한다.
Promise.all(iterable);
Promise.allSettled(iterable);
```

다음은, Promise의 인스턴스가 지니는 메서드들이다.

```jsx
promise.then(onFulfilled, onRejected);
promise.catch(onRejected);
promise.finally(onFinally); // 입력으로 인자를 받지 않으며 이행 또는 거부를 최종 결정한다.
```

## 수동으로  Promise 만들기

```jsx
function delay(milliseconds) {
	return new Promise((resolve, reject) => {
		setTimeout(() => {
			resolve(new Date());
		}, milliseconds)
	})
}

delay(1000).then(res => console.log(res));
```

## Promise화 (Promisification)

```jsx
function promisify(callbackBasedApi) {
	return function promisified(...args) {
		return new Promise((resolve, reject) => {
			const newArgs = [...args, (err, result) => err ? reject(err) : resolve(result)];
			callbackBasedApi(...newArgs);
		})
	}
}
```

```jsx
import { randomBytes } from 'crypto';

const randomBytesP = promisify(randomBytes);
randomBytesP(32).then(buffer => console.log(buffer));
```

특정 함수의 구조를 안다면, 그 함수를 Promise로 변경하는 함수를 만들 수도 있다.

# Async/Await

> 프라미스는 콜백에 비해 획기적인 도약입니다. (중략) 그러나 프라미스는 순차적 비동기 코드를 작성할 때 여전히 차선책에 불과합니다. 프라미스 체인이 콜백 지옥을 갖는 것보다는 낫지만, 우리는 여전히 then()을 호출해야 하며 체인에서 각 작업에 대한 새로운 함수를 만들어야 합니다. (중략) JavaScript에, 그에 대한 대답으로 async 함수와 await 표현에 대한 ECMAScript 표준이 작성되었습니다.
> 

> 그러나 async/await는 우리가 지금까지 배워왔던 모든 비동기 제어 흐름 패턴을 대신하지 못합니다. 그와 반대로 async/await는 프라미스에 크게 의지합니다.
> 

1. async 함수는 비동기적으로 실행됩니다.
2. await 표현은 함수의 실행이 보류되고 상태가 저장되며 제어가 이벤트 루프로 반환합니다.
    1. 이벤트 루프의 그림을 기억하나요?
3. await 표현은 Promise가 아니어도 동작합니다.
    1. Promise가 아닌 값일 경우 Promise.resolve()에 전달된 값을 기다리는 것과 같아진다.
4. async 함수는 Promise를 반환한다.
