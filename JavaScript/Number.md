## 숫자

자바스크립트의 수는 실수에서 영감을 받았지만, 진짜 실수는 아닙니다.

### 자바스크립트의 숫자형

`Number` 하나로 모든 숫자를 나타냅니다.

- 생산성 증가 (고민필요 x)
- 타입 변화 오류 x
- int 사용으로 인한 오버플로우 발생 x

    **Integer Overflow in JAVA**

    허용된 정수 범위를 초과한 경우 발생하는 문제

    int: 2147483648 ~ 2147483647

    ⇒ 2147483647 + 1 입력 시: -2147483648 

    최상위 비트가 부호 비트로 사용되기에 -가 붙어 버린다.

    ```java
    System.out.println(Integer.toBinaryString(2147483647)); // int 가장 큰 값
    // 01111111 11111111 11111111 11111111

    System.out.println(Long.toBinaryString(2147483648L));
    // 10000000 00000000 00000000 00000000
    ```

### 부동소수점(Floating Point)

**부동소수점 표현하는데 가장 많이 쓰이는 표준**

[IEEE 754](https://ko.wikipedia.org/wiki/IEEE_754)

- 구조

    부호, 지수, 가수로 나뉩니다.

![image](https://user-images.githubusercontent.com/43411599/128874894-aa0bc05e-1d76-4cce-bbd6-0991f2c18d78.png)
자바스크립트의 number는 IEEE 754를 따르지만, 전체 표준을 따르지는 않습니다.

![image](https://user-images.githubusercontent.com/43411599/128874952-c9cde3aa-82c0-4264-a79a-acb066e05998.png)
자바스크립트의 number는 자바의 `double`과 비슷합니다. (64비트 2진 부동소수점 타입)

0.001111 → 1이 나올때 까지 자릿 수 옮겨서 `1.xxx`로 변경 시키게 되는데, 시작접이 항상 1이 되므로, 하나의 비트를 벌어 65bit의 수를 64bit로 표현할 수 있습니다.

**지수**

`부호 * 유효 숫자 * (2 ** 지수)`

- 참고하면 좋을 글: [https://medium.com/@syalot005006/자바스크립트의-실수-계산-오류-a72ec3326b50](https://medium.com/@syalot005006/%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8%EC%9D%98-%EC%8B%A4%EC%88%98-%EA%B3%84%EC%82%B0-%EC%98%A4%EB%A5%98-a72ec3326b50)

    ```jsx
    ((0.1 + 0.2) + 0.3) > (0.1 + (0.2 + 0.3)) // true..!!
    ```

### 특이한 넘버 타입들

- NaN (Not a Number)
- -0
- Infinity, -Infinity

```jsx
1 / Infinity // 0
1 / -Infinity // -0
+'hello' // NaN
typeof(NaN) // number
```

### Number

> ***Not number But Number***

숫자를 만드는 함수

- MDN Number: [https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Number](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Number)

**Number의 속성**

- EPSILON: 1과 1보다 큰 값 중에서 가장 작은 값의 차
- MAX_SAFE_INTEGER: 자바스크립트에서 안전한 최대 정수값(2^53 - 1)
- MAX_VALUE: 자바스크립트에서 표현 가능한 가장 큰 수
- MIN_VALUE: 자바스크립트에서 표현 가능한 가장 작은 수(0보다 큰 수 중에서 가장 작은 수)

**Number의 메서드**

- isSafeInteger(number): 전달된 값이 안전한 정숫값(2^53 - 1)인지 확인

```jsx
Number.isSafeInteger(3);                    // true
Number.isSafeInteger(Math.pow(2, 53));      // false
Number.isSafeInteger(Math.pow(2, 53) - 1);  // true
Number.isSafeInteger(NaN);                  // false
Number.isSafeInteger(Infinity);             // false
Number.isSafeInteger('3');                  // false
Number.isSafeInteger(3.1);                  // false
Number.isSafeInteger(3.0);                  // true
```

- isInteger(number): 정수인지만 확인,  Number.MAX_SAFE_INTEGER보다 큰 수는 정수로 간주하여 대부분은 틀립니다.

### 연산자

**전위 연산자**

- +: 숫자로 변환
- -: 부호 변환
- typeof: 타입 확인

**중위 연산자**

- +, -, *, /, %, **

### 비트 단위 연산자

비트 단위 연산자는 자바스크립트 수들을 부호가 있는 32비트 정수형으로 바꾼 다음 비트 연산을 수행하고 다시 자바스크립트 수로 변환합니다.

**단항 비트 단위 연산자**

- ^: not

**이항 비트 단위 연산자**

- &: and
- |: or
- ^: exclusive or
- <<: 왼쪽 shift (오른쪽에 0 삽입)
- >>: 오른쪽 shift 가장 왼쪽 비트의 복사본을 왼쪽에 밀어 오른쪽으로 이동하고, 맨 오른쪽 비트가 잘린다.
- >>>: 왼쪽에 0을 밀어 넣어 오른쪽으로 이동하고, 오른쪽 비트를 잘라낸다.

### Math

- floor: 인수보다 작거나 같은 수 중에서 가장 큰 정수
- trunc: 정수 부분 반환 → 0에 가까운 수

```jsx
Math.floor(-2.5) // -3
Math.trunc(-2.5) // -2
```

### 숫자 속의 괴물

숫자를 분리하여 부호와 정수 계수, 지수와 같은 구성 요소로 나누는 deconstruct 함수를 만들어봅니다.

```jsx
function deconstruct(number) {
	// number = sign * coefficeint * (2 ** exponent)
	
	let sign = 1;
	let coefficient = number;
	let exponent = 0;

	// 계수에서 부호 분리
	if (coefficient < 0) {
		coefficient = -coefficient;
		sign = -1;
	}

	// -1128: MIN_VALUE의 지수 값에서 유효 숫자 비트 - 보너스비트(1)
	// 계수가 0이 될 때까지 2로 나누고, 나눈 횟수를 -1128에 더해 지수를 구합니다.
	if (Number.isFinite(number) && number !== 0) {
		exponent = -1128;
		let reduction = coefficient;
		while (reduction !== 0) {
			exponent += 1;
			reduction /= 2;
		}
	reduction = exponent;

// 지수가 0이 아닐 때 계수 바로 잡기
	while (reduction > 0) {
		coefficient *= 2;
		reduction -= 1;
	}
	while (reduction < 0) {
		coefficient *= 2;
		reduction += 1;
	}
	
	}
	// 부호, 계수, 지수, 원래 숫자 반환
	return {
		sign,
		coefficient,
		exponent,
		number
	};
}
```

```jsx
deconstruct(1);
// {sign: 1, coefficient: 9007199254740992, exponent: -53, number: 1}}

deconstruct(Number.MAX_SAFE_INTEGER);
// {sign: 1, coefficient: 9007199254740991, exponent: 0, number: 9007199254740991}

deconstruct(0.1);
// {sign: 1, coefficient: 7205759403792794, exponent: -56, number: 0.1}
```

0.1 ≠ 1 * 7205759403792794 * 2 ** -56

(유사하긴 하지만, 정확히 0.1은 아니다.)

최대한 안전한 정수 범위 내에서 작업을 하는 것이 좋습니다.

## 큰 정수

자바스크립트는 64비트 정수를 만들 수 없습니다.

자바스크립트의 숫자형은 단순성에 목적이 있기 때문에 int64형 같이 새로운 타입을 추가하는 데는 어려움이 있습니다. 나아가서는 더 큰 숫자를 원하게 될 수도 있습니다.

### 배열 이용하기

큰 정수는 배열 형태로 저장됩니다.

배열의 형태가 자유롭기 때문에 큰 수를 저장하기에 적합하며, 연산을 위해 24bit 단위로 저장합니다.

```jsx
// 9000000000000000000
// 8650752 + 7098594 * 2^24 ** 1 + 31974 * 2^24 ** 2
["+", 8650752, 7098594, 31974]
```

큰 정수 시스템을 구현해봅시다.

```jsx
const radix = 16777216; // 2^24
const radix_squared = radix * radix;
const log2_radix = 24;
const plus = "+";
const minus = "-";
const sign = 0;
const least = 1;

const last = (array) => {
	return array[array.length - 1];
}

const next_to_last = (array) => {
	return array[array.length - 2];
}
```

```jsx
// to simplify code
const zero = Object.freeze([plus]);
const one = Object.freeze([[plus, 1]);
const two = Object.freeze([[plus, 2]);
const ten = Object.freeze([plus, 10]);
const negative_one = Object.freeze([minus, -1]);

// is big integer?
const is_big_integer = (big) => {
	return Array.isArray(big) && (big[sign] === plus || big[sign] === minus); 
}

const is_negative = (big) => {
	return Array.isArray(big) && big[sign] === minus;
}

const is_positive = (big) => {
	return Array.isArray(big) && big[sign] === plus;
}

const is_zero = (big) => {
	return !Array.isArray(big) || big.length < 2;
}
```

**mint 함수 만들기**

배열의 마지막 요소가 0인 경우 제거하는 함수 mint를 만들어봅시다.

배열의 뒤쪽에 있는 요소들은 실제 값에서는 높은 자릿수에 있기 때문에, 윗 자리의 0은 필요 없는 숫자가 됩니다.

상수 중에 일치하는 값이 있다면, 상수로 바꾸고, 더 이상 바꿀 값이 없다면 배열을 동결하는 코드를 작성해봅니다.

```jsx
// aply mint: get big integer
// find first zero and replace it with available constant
const mint = (proto_big_integer) => {
	while (last(proto_big_integer) === 0) {
		proto_big_integer.length -= 1;
	}
	if (proto_big_integer.length <= 1) {
		return zero;
	}

	if (proto_big_integer[sign] === plus) {
		if (proto_big_integer[least] === 1) {
			return one;
		}
		if (proto_big_integer[least] === 2) {
			return two;
		}
		if (proto_big_integer[least] === 10) {
			return ten;
		}
	} else if (proto_big_integer.length === 2) {
		if (proto_big_integer[least] === 1) {
			return negative_one;
		}
	}
	return Object.freeze(proto_big_integer);
}
```

```jsx
// negative <-> positive
const neg = (big) => {
	if (is_zero(big)) {
		return zero;
	}
	let negation = big.slice();
	negation[sign] = (
		is_negation(big) ? plus : minus;
	);
	return mint(negation);
}
```

```jsx
// get abs
const abs = (big) => {
	return (is_zero(big) ? zero : (is_negative(big) ? neg(big) : big));
}
```

```jsx
// get signum
const signum = (big) => {
	return (is_zero(big) ? zero : (is_negative(big) ? negative_one : one);
}
```

```jsx
// is equal
const eq = (comparahend, comparator) => {
	return comparahend === comparator || (
		comparahend.length === comparator.length
		&& comparahend.every(function (element, element_nr) {
			return element === comparator[element_nr];
		})
	);
}
```

```jsx
// is absolute less than another absolute
// 길이가 같을 경우에는 계산이 복잡하기에, 반대로 탐색해서 빨리 끝낼 수 있도록 함
const abs_lt = (comparahend, comparator) => {
	return (
		comparahend.length === comparator.length
		? comparahend.reduce(
			function (reduction, element, element_nr) {
				if (element_nr !== sign) {
					const other = comparator[element_nr];
					if (element !== other) {
						return element < other;
					}
				}
				return reduction;
			},
			false
		)
		: comparahend.length < comparator.length
	);
}
```

```jsx
const lt = (comparahend, comparator) => {
	return (
		comparahend[sign] !== comparator[sign]
		? is_negative(comparahend)
		: (
				is_negative(comparahend)
				? abs_lt(comparator, comparahend)
				: abs_lt(comparahend, comparator);
		)
	);
}
```

```jsx
// lt 함수를 이용해 보수를 취하거나 인자를 바꿔 비교 함수를 만들 수 있습니다.
const ge = (a, b) => !lt(a, b);
const gt = (a, b) => lt(b, a);
const le = (a, b) => !lt(b, a);
```

다음 시간에 비트 연산으로 계속..
