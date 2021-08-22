# Boolean

Boolean은 놀랍게도 로직 시스템을 개발한 George Boole의 이름에서 따온 이름입니다. 

### 이상한 bool들

```jsx
undefined < null // false
undefined > null // false
undefined === null // false

NaN === NaN // false
NaN !== NaN // true

"11" < "2" // true
"2" < 5 // true
5 < "11" // true
```

- 반복문의 종료 조건에서 === 대신 `<=` 를 쓰는 게 안전합니다.
- 비교 연산 시, 양 쪽 둘 다 같은 자료형 쓰면 좋습니다.
- ==, ≠ 쓰지 맙시다.
- NaN을 확인하기 위해서는 Number.isNan(x)를 씁시다. 다르게 활용하는 방법은 되도록 이용하지 않는 편이 좋습니다.

### bool인척 하기

**제대로 동작하는 bool들**

- if, while, for, do, !, &&, ||, ? :, Array filter, find...

**짭 bool들**

짭 bool중 falsy한 값들

- false
- null
- undefined
- ""
- 0
- NaN

⇒ 실제로 false처럼 동작하지만 false가 아닌 값들 입니다.

자바스크립트의 조건문은 bool 값을 쓰지 않아도 동작하지만, 되도록 더 나은 프로그램을 만들기 위해서는 지양하는 것이 좋습니다.

### 논리 연산자

bool인척 하기의 또 다른 피해자입니다.

- !: not
- &&: and
- ||: or

### 하지 말라면 하지 마세요!

- bool인척 하는 값들 대신 진짜 bool을 씁시다.
- 풀어서 단순화하는 것이 좋습니다.

```jsx
!(a === b) === (a !== b)
!(a <= b) === (a > b)
!(a > b) === (a <= b)
```

```jsx
!(p && q) === !p || !q
!(p || q) === !p & !q
```
