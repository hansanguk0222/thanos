# ESM : ECMAScript 모듈

사실 CommonJS는 표준이 아니에요.

조금 역사를 이야기할 필요가 있는데, 아시다시피 JavaScript는 프론트 언어였습니다. 그런데 프론트에서는 HTML 문서 상에 script 태그를 이용해서 JavaScript 코드를 전역 상태로 사용하곤 했죠. 그러다 보니 당연히 모듈이 필요없었어요. 모든 게 하나의 전역 코드니깐요.

그렇지만 백엔드에서도 JavaScript가 쓰이기 시작하면서 모듈 시스템에 대한 고민이 생겨났고, 그 결과 사용자들이 고안해낸 게 CommonJS였죠. 그렇지만 이 CommonJS는 표준이 되지는 못했습니다.

아마도 그 이유는, ECMAScript가 프론트, 백에 따라 같은 언어, 다른 표기로 나뉘는 걸 꺼렸기 때문인 거 같아요. 그래서 ECMAScript는 ESM이라는 표준 모듈 시스템을 만들었습니다.

**이제부터 ESM이 CommonJS와 다른 점들을 이야기해보려고 해요.**

경험적으로 뭐가 다른지는 이미 다들 아는 게 많겠지만요.

# ESM는 비동기 임포트가 불가능해

```jsx
if (condition) {
	import module1 from 'module1';
} else {
	import module2 from 'module2';
}
```

첫째로 ESM은 비동기 임포트가 불가능하다는 점이 차이점입니다.

위처럼 작성하는 것은 불가능해요.

이는 불필요한 제약처럼 보일 수 있지만, static import ( 정적 임포트 ) 를 통해 사용하지 않는 코드 제거 ( tree shaking )와 같이 코드 최적화를 해줄 수 있는 종속성 정적 분석을 가능하게 해준다고 해요.

말이 조금 어렵죠?

쉽게 말하면 코드 최적화란, 사용하지 않는 부분을 없애주는, 경량화라고 생각하면 좋을 거 같아요. 그렇지만 코드가 비동기적으로 실행되는 CommonJS에서는 실행 단계에서 전체 코드 형태를 알  수 없으니 필요 없는 부분을 걸러낼 수가 없죠.

이 부분에 대해서 더 얘기하려면 일단 모듈 종속성이라는 걸 더 깊게 이야기해야 할 거 같아요.

# 모듈 종속성이 뭐고, 그게 어떻게 생기는 건데?

> ESM이 어떻게 동작하고 순환 종속성을 다루는지 이해하기 위해서는 ES 모듈을 사용할 때 JavaScript가 어떻게 파싱되고 평가되는지 더 알아봐야 합니다.

### 로딩 단계

> 인터프리터는 모듈이 실행되어야 할 코드의 순서와 함께 모듈 간에 어떠한 종속성을 갖는지 이해하기 위해서 기본적으로 종속성 그래프를 필요로 합니다. Node 인터프리터가 실행되면, 일반적으로 JavaScript 파일 형식으로 실행할 코드가 전달됩니다. 파일은 종속성 확인을 위한 진입점 (entry point)입니다. `인터프리터는 진입점에서부터 필요한 모든 코드가 탐색되고 평가될 때까지 import 구문을 재귀적인 깊이 우선 탐색으로 찾습니다.`

쉽게 말하자면,

1. 생성 ( 또는 파싱 ) : import를 재귀적으로 찾아가며 모든 파일의 내용을 적재한다.
2. 인스턴스화 : 각 파일에서 export된 참조를 메모리에 유지하고, 각 종속성 관계를 추적한다.
3. 평가 : 인스턴스가 끝났다면 이제 모든 준비가 된 것이므로 코드를 평가한다.

ESM은 모듈 종속성 관계를 추적하기 위해서 이런 세 단계를 따르고 있다고 해요.

CommonJS와 유사해보여도 여기에는 근본적인 차이가 있는데,

CommonJS는 동적이기 때문에 종속성을 모두 파악하기 전에 파일이 읽힘과 동시에 실행된다는거죠.

```jsx
if (a === true) {
	const { aModule } = require('a.js');
}
```

- 예를 들어 위와 같이 그때 그때 파일을 읽을 수 있는 것도 동적인 특성 덕분이에요.
- 하지만 이는, 종속성 그래프를 그리지 않고 그 순간에 필요한 모듈을 불러오는 식이에요.

동적인 건 뭐고 정적인 게 뭔지 많이 헷갈릴 거에요.

코드가 동적이다, 정적이다 라고 하면 당연히 헷갈릴 법 해요. 

그러니깐 이 특성을 보면 좋을 거 같아요.

CommonJS는 코드 중간에 require()을 사용해도 되지만, ESM에서는 코드 중간에 import가 불가능해요.

마치 var에서 호이스팅이 일어나는 것처럼, import가 무엇보다도 우선시되는 거죠.

그래서 언제든 다른 모듈을 부를 수 있는 CommonJS는 동적이고, ESM은 반대로 정적이라는 거에요.

**이런 차이 때문에 두 방식에서는 종속성의 차이가 발생해요.**

종속성은, 쉽게 말하면 어떤 파일이 어떤 파일을 부르고 있는지, 그 관계를 말해요.

# 두 방식에 대한 순환 종속성 분석

ESM에서는 서로에 대한 완전한 참조를 가지게 돼요. 이 부분이 CommonJS와는 다르다네요.

아니, 그러면 CommonJS에서는 완전하지 못한 참조라는 걸까요?

네, 맞습니다. CommonJS에서는 시간 차가 발생해요.

가령 a가 b를 부르고, b가 a를 부르는 구조로 되어 있을 때,

a는 b를 가지고 있지만, a가 가진 b에는 a가 없다는 거죠.

왜냐하면, a가 먼저 b를 불렸기 때문에 b에는 아직 a가 없죠.

```jsx
a = { b : { } } // a가 가지고 있는 종속성
b = { a : { b } } // b가 가지고 있는 종속성
```

ESM에서는 이런 문제가 발생하지 않아요.

서로의 관계를 모두 파악한 다음에 종속성을 표현하기 때문에 이런 문제가 발생하지 않죠.

실행은 이런 종속성 관계가 모두 파악된 다음에 일어납니다.

각 단계 별로 더 깊게 살펴보면,

### 1단계 : 파싱

```jsx
main.js -> a.js -> b.js // 차례대로 모두 방문한다.
main.js -> b.js // b는 이미 방문한 지점이므로 무시된다.
b.js -> a.js // a는 이미 방문한 지점이므로 무시된다.
```

- 최종적으로 main, a, b의 리스트가 되지만, 더 많은 모듈이 있으면 트리 구조에 가까워진다.

### 2단계 : 인스턴스화

> 인스턴스화 단계에서는 인터프리터가 이전 단계에서 얻어진 트리 구조를 따라 아래에서 위로 움직입니다. 인터프리터는 모든 모듈에서 익스포트된 속성을 먼저 찾고 나서 메모리에 익스포트된 이름의 맵을 만듭니다.

```jsx
b = { loaded : <uninitalized> a : <uninitalized> } // a file을 참조하고 있다.
a = { loaded : <uninitalized> b : <uninitalized> }
main
```

### 3단계 : 평가

모든 인터턴스화 단계를 거쳤다고 해도 이름을 추적하여 링크를 만든 것일 뿐, 실질적인 코드가 있는 것은 아니다. 따라서 평가를 할 필요가 있는데, 이 단계에서는 DFS를 후위 깊이 우선 탐색으로 하여 아래에서 위로 올라가는 방식으로 평가한다. 이런 경우 마지막 파일은 entryPoint가 된다. 이는 비즈니스 로직을 수행하기 전에 익스포트된 모든 값이 초기화되는 것을 보장한다.

이로 인해, 또한, 상호 참조하고 있는 모듈 간의 문제도 해결할 수 있어요.

이는 CommonJS에서 서로 코드를 가져와서 읽고 있는 것과 달리,

ESM은 참조를 사용하고 있기 때문이죠.

ES6는 이미 대중적이므로, 특이할 사항만 더 적어보도록 할게요.

- `export를 사용하는 것은, 자동 임포트, 자동 완성, 리팩토링 툴을 지원할 수 있게 해준다.` 하지만 export default 구문은 그럴 수 없다.
- `export default는 단일 책임 원칙을 권장하고, 깔끔한 하나의 인터페이스만을 제공하기에 적합`하다. 사용자의 관점에서 봤을 때 `바인딩을 위한 이름을 정확히 알 필요 없다는 것도 장점`이다.
- export default는 사용하지 않는 코드를 제거하기가 어렵다. export default는 보통 객체나 클래스 등을 통해 내보내지기 때문에, 객체의 일부분만 사용하더라도 전체 코드는 남게 된다.

이 외에도 몇 가지 특징들이 더 있긴 하지만, 실제로 접할 일은 아마 없을 거라고 생각해요.

# 결론적으로 ESM은 CommonJS보다...

1. 비동기적인 동작을 제한한 대신에 전체 파일 구조를 파악할 수 있게 되었다.
2. 따라서 코드의 최적화가 가능해졌고,
3. 순환 종속성 문제를 해결할 수 있게 되었다. ⇒ 실질적으로 CommonJS보다 안정적이다.

즉, ESM과 CommonJS는 코드 취향의 문제가 아니라는 걸로 결론내립니다.

잘못된 내용이 있으면 언제든지 경청하겠습니다. :)

- 더 알고 싶다면...

    ## 2.6.6. 비동기 임포트

    ESM의 단점은, 모듈 식별자를 실행 중에 생성할 수 없다는 점, 모든 파일의 최상위에 선언되어 제어 구문 내에 포함될 수 없다는 점이다. 이는 사용자에 따라 다른 모듈을 불러야 하는 경우에 지나친 제약이 될 수 있다. 따라서 이를 극복하기 위해 비동기 임포트 ( 동적 임포트 ) 를 제공한다.

    ```jsx
    if(true) {
    	require(a);
    } else {
    	require(b);
    }
    ```

    ```jsx
    // strings-ko.js
    export const HELLO = '안녕하세요.'

    // strings-jp.js
    export const HELLO = '오하이요';

    // strings-en.js
    export const HELLO = '하이'
    ```

    ```jsx
    const SUPPORTED_LANGUAGES = ['ko', 'en', 'jp'];
    const selectedLanguage = process.argv[2];

    if (!SUPPORTED_LANGUAGES.includes(selectedLanguage)) {
    	console.error('The specified language is not supported');
    	process.exit(1);
    }

    const translationModule = `./strings-${seletecLanguages}.js`;
    import(translationModule).then((string) => {
    	console.log(string.HELLO);
    })
    ```

    import() 연산자는 문법적으로 모듈 식별자를 인자로 취하고 모듈 객체를 Promise로 반환하는 함수와 동일하다.

    ```jsx
    node main.js ko // 실행 시 인자를 건네주면 된다.
    ```

    ## 2.6.8. 모듈의 수정

    ```jsx
    import fs from 'fs';

    const originalReadFile = fs.readFile;
    let mockResponse = null;

    function mockedReadFile (path, cb) {
    	setImmediate(() => {
    		cb(null, mockedResponse)
    	})
    }

    export function mockEnable(responseWith) {
    	mockedResponse = responseWith;
    	fs.readFile = mockedReadFile;
    }

    export function mockDisable() {
    	fs.readFile = originalReadFile;
    }
    ```

    ```jsx
    import fs from 'fs';
    import { mockEnable, mockDisable } from './mock-read-file.js';

    mockEnable(Buffer.from('Hello World'));

    fs.readFile('fake-path', (err, data) => {
    	if (err) {
    		console.error(err);
    		process.exit(1);
    	}
    	console.log(data.toString());
    })

    mockDisable();
    ```

    - 몽키 패치 방식
    - readonly live binding 방식이라고 해도 객체의 특성을 이용해서 속성을 재할당할 수 있다.
    - fs.readFile을 이 속성을 이용하여 재할당해 mock data만을 읽도록 수정하는 코드이다.
    - 만약 readFile을 읽기 전용 라이브 바인딩으로 가져온다면 위 방식은 동작하지 않는다.

    ESM 환경에서 몽키 패치는 복잡하고 신뢰하기 어렵기 때문에 jest 같은 프레임워크를 사용한다.

    ```jsx
    import fs, { readFileSync } from 'fs';
    import { syncBuiltinESMExports } from 'module';

    fs.readFileSync = () => Buffer.from('Hello, ESM');
    syncBuiltinESMExports();

    console.log(fs.readFileSync === readFileSync);
    ```

    module의 syncBuiltinESMExports를 이용하여 default exports 객체에 있는 속성들의 값이,

    named exports와 동일한 것으로 매핑되게 할 수 있다.

    # 2-7.ESM과 CommonJS의 차이점과 상호 운용

    1. strict mode에서 실행된다.
    2. 참조 유실 ( strict mode로 인한 )
        - require, export, module.exports, __filename, __dirname 등 몇 가지 참조가 정의되지 않는다.
        - 대신에 import.meta에서 데이터를 꺼낼 수 있다.
            - import.meta.url은 현재 모듈을 참조한다.
            - 문맥 상 require()도 허용할 수 있다.

                ```jsx
                import { createRequire } from 'module';
                const require = createRequire(import.meta.url);
                ```

    3. ES에서의 this는 undefined, CommonJS에서의 this는 exports이다.
    4. ESM은 default exports에 한하여 CommonJS를 import 할 수 있다.
    5. JSON을 CommonJS처럼 가져올 수는 없다.
        - module.createRequire을 사용해서 해결할 수 있다.

    ### 읽기 전용 라이브 바인딩

    > ES 모듈의 또 다른 기본적인 특성은 임포트된 모듈이 익스포트된 값에 대해 읽기 전용 라이브 바인딩된다는 개념입니다.

    ```jsx
    // count.js
    export let count = 0;
    export function increment() {
    	count++;
    }
    ```

    ```jsx
    // main.js
    import { count, increment } from './counter.js';
    console.log(count); // 0
    increment();
    console.log(count); // 1
    count++; // TypeError : Assignment to constant variavble!
    ```

    - 언제든지 count의 값을 읽을 수 있고, increment() 함수로 변경도 가능하지만, count 변수를 직접적으로 변경시키려 할 때에는 const 로 바인딩한 값을 변경하려 할 때와 같은 에러가 발생한다.
    - `이러한 특성을 읽기 전용 라이브 바인딩 ( readonly binding ) 이라고 한다.`
    - CommonJS는 구조분해할당, 또는 얕은 복사와 같이 동작하기 때문에 값을 바꿀 수도 있고, `문제는 모듈에서는 이러한 변화를 알지 못할 것이라는 점이다.`
