- 부제 : Node의 모듈 시스템과 require()

# **require() : 해결 (resolve) 알고리즘**

```jsx
const myJsCode = require("./myJsCode"); // .js는 생략 가능하다.
const path = require("path");
const express = require("express");
```

해결 알고리즘은 크게 다음 세 가지로 나눌 수 있어요.

1. 파일 모듈 : 상대 경로로 작성되었는가?
2. 코어 모듈 : 파일 명만이 언급되었는가?
3. 패키지 모듈 : 코어 모듈에서 해당 파일을 찾을 수 없는가? ( => node_modules)

우리는 require로 이런 세 가지 형태를 불러올 수 있어요.

안타깝게도 이 모든 걸 하나로 정리해서 설명하기는 `많이 많이 많이` 힘들 거 같아요.

그래서 하나씩 순차적으로 설명해볼까 해요.

# **require() : 코어 모듈을 불러오는 방법**

실제 Node.js에서의 require는 이렇게 생겼어요.

```jsx
NativeModule.require = function (id) {
    if (id == "native_module") { // (1)
        return NativeModule;
    }
    var cached = NativeModule.getCached(id); // (2)
    if (cached) { // (3)
        return cached.exports;
    }
    if (!NativeModule.exists(id)) { // (4)
        throw new Error("No such native module " + id);
    }

    process.moduleLoadList.push("NativeModule " + id); // (5)
    var nativeModule = new NativeModule(id); // (6)
    nativeModule.cache();
    nativeModule.compile();
    return nativeModule.exports;
};
```

이건 아마도, native ( core )를 require() 하는 것으로 보여요.

각 라인에 대해서 설명할 필요가 있을 거 같아요.

그러니깐 주석의 넘버링을 봐주시면 될 거 같아요.

- (1) : 일단 native_module을 부르는 건지 봐야 해요. 그런거면 전체를 주면 돼요.
- (1) : 하지만 일반적으로는 필요 없을 거 같으니깐 이 코드는 설명할 때는 지워도 될 거 같아요.

```jsx
NativeModule.require = function (id) {
    var cached = NativeModule.getCached(id); // (2)
    if (cached) { // (3)
        return cached.exports;
    }
    if (!NativeModule.exists(id)) { // (4)
        throw new Error("No such native module " + id);
    }

    process.moduleLoadList.push("NativeModule " + id); // (5)
    var nativeModule = new NativeModule(id); // (6)
    nativeModule.cache();
    nativeModule.compile();
    return nativeModule.exports;
};
```

- (2) : 다음은 cache가 되었는지 찾아보는 거에요. 그냥 객체에 해당 key로 저장된 걸 찾는 거죠.
- (3) : 있으면 그냥 바로 꺼내주면 돼요.

간단해 보이지만, require()는 이런 식으로 cache를 쓰기 때문에,

여러 파일에서 require()를 사용하고, 동일한 걸 호출한다고 해도 1개의 모듈을 사용하게 강제해요.

```jsx
if (!NativeModule.exists(id)) { // (4)
	throw new Error("No such native module " + id);
}
```

- (4) : 이제 core 부분에 해당 id와 일치하는 게 있는지 체크하는 로직이에요.
- (4) : 이 부분은 직접 exist() 메서드를 확인해보는 게 좋을 거 같네요.

```jsx
const source = process.binding("natives");
const exists = (id) => {
    return NativeModule.source.hasOwnProperty(id);
};
```

- source는 process.binding에 'native'를 parameter로 전달한 결과물을 가지고 있어요.
- 여기에는 core에 해당하는 모든 코드가 텍스트 형태로 저장되어 있습니다.
- 따라서 exist는 id를 통해 그런 core 모듈이 있는지 찾는 거에요.
    - core는 readline이나 fs처럼 node.js 자체에 이미 있는 모듈을 말하는 거에요.

```jsx
NativeModule.require = function (id) {
    var cached = NativeModule.getCached(id); // (2)
    if (cached) { // (3)
        return cached.exports;
    }
    if (!NativeModule.exists(id)) { // (4)
        throw new Error("No such native module " + id);
    }

    process.moduleLoadList.push("NativeModule " + id); // (5)
    var nativeModule = new NativeModule(id); // (6)
    nativeModule.cache();
    nativeModule.compile();
    return nativeModule.exports;
};
```

- (5) : 다시 돌아와서, (5)는 현재 사용 중인 모듈을 기록해두는 것이고요,
- (6) : 이 부분은 그 부분에 대한 NativeModule을 생성해주는 거에요.

이후 캐시를 하고, 그 모듈을 컴파일하여 nativeModule.exports를 return 해주면 끝납니다.

(5), (6)보다는 cache와 compile 메서드들이 더 중요할 수 있는데요,

cache는 다음에도 동일한 모듈을 반환할 수 있도록 캐시해두는 역할을 하고요,

**compile은 단순 string 타입으로 저장되어 있는 모듈을, 실제 동작 가능한 형태로 변환하는 거에요.**

자세한 건 아래에서 이야기할게요.

# module, module.exports?

놀랍게도 사실 얘네는 그저 구조분해할당에 불과합니다.

```jsx
// a.js

module.exports = { a : 3 };
```

```jsx
// b.js

// const { a } = require('./a.js');
const { a } = module.exports; // a.js에서의 module.exports
```

- 두번째 code box를 보면, 주석처리된 부분과 아래 코드는 사실 상 동일합니다.
- 결국 module.exports에서 구조분해할당해서 a를 받는 거에요.
- 왜 exports 할 때의 이름과 동일해야만 a를 받을 수 있느냐, 이것도 그럼 간단한 문제죠.
    - A : 구조분해할당이니깐요.

그러면 이번에는 이 코드들이 어떻게 만들어져 있는지를 보도록 할까요?

```jsx
const loadModule = (filename, module, require) => {
    const wrappedSrc = `(function (module, exports, require) {
            ${fs.readFileSync(filename, "utf8")}
            })(module, module.exports, kakasooRequire)`;
    eval(wrappedSrc);
};
```

- loadModule은 filename, module, require를 받고,
- 실행시킨 결과를 exports에 담도록 해주는 함수에요.
- exports에 담아놨으면, 결과적으로 return 해주지 않더라도 객체의 값은 유지가 되게 되므로,
- 실행된 함수의 결과물이 담기게 되는 거죠.

```jsx
const fs = require("fs");

function kakasooRequire(moduleName) {
    const loadModule = (filename, module, require) => {
        const wrappedSrc = `(function (module, exports, require) {
                ${fs.readFileSync(filename, "utf8")}
                })(module, module.exports, kakasooRequire)`;
        eval(wrappedSrc);
    };

    const id = moduleName;

    const module = {
        exports: {},
        id,
    };

    loadModule(id, module, kakasooRequire);
    return module.exports;
}
```

- 캐시도 없고, resolve()도 없다고 가정한 상태의 코드에요.
- 캐시가 없지만, 어차피 1번만 호출할 테니 저 상태로도 문제가 없을 거고요,
- resolve가 없지만, moduleName을 "./a.js"처럼 resolve된 형태로 넣어주면 문제가 없죠.
    - 원래는 const id = resolve(moduleName); 형태거든요.

그 다음에는 적당히 module 객체를 만든 다음에, loadModule을 호출하면 돼요.

그러면 loadModule에서 id에 맞는 모듈을 가져다가 랩핑된 함수를 실행시키게 돼요.

이 부분을 다시 한 번 보죠.

```jsx
const fs = require("fs");

const loadModule = (filename, module, require) => {
    const wrappedSrc = `(function (module, exports, require) {
            ${fs.readFileSync(filename, "utf8")}
            })(module, module.exports, kakasooRequire)`;
    eval(wrappedSrc);
};
```

- 랩핑이 되어 있는 부분만 자세히 보도록 해요.

```jsx
// a.js

module.exports = { a : 3 };
```

```jsx
(function(module, exports, require) {
	fs.readFileSync(filename, 'utf8')
})(module, module.exports, kakasoRequire);
```

- 만약 filename이 './a.js'라고 한다면, 위의 코드를 텍스트 상태로 읽게 되겠지만,
- eval 내에서는 이 텍스트가 JavaScript 코드로 평가되어서 실제 코드처럼 동작하게 되죠.
- 그러면 그 코드 에는 module.exports가 있으니까 담기게 돼요.
- 위의 두 코드 박스를 합치면,

```jsx
(function(module, exports, require) {
	module.exports = { a : 3 };
})(module, module.exports, kakasoRequire);
```

- 결국 이런 형태거든요.

그러면 이 코드 바깥에서도 module.exports의 값이 바뀌게 되고,

구조분해할당으로 값을 받는 게 가능해지죠, 결국 처음에 설명한 대로 되는 거에요.

```jsx
// b.js

const { a } = module.exports; // a.js에서의 module.exports
```

# 전체 kakasooRequire 코드

```jsx
const fs = require("fs");

function kakasooRequire(moduleName) {
    const resolve = function (moduleName) {
        const moduleNameSplited = moduleName.split("");
        if (moduleNameSplited.slice(0, 2) === "./") {
            if (moduleNameSplited.slice(-3).join("") === ".js") {
                return moduleName;
            }
            return moduleName + ".js";
        } else {
					return moduleName;
				}
    };

    const loadModule = (filename, module, require) => {
        const wrappedSrc = `(function (module, exports, require) {
                ${fs.readFileSync(filename, "utf8")}
                })(module, module.exports, kakasooRequire)`;
        eval(wrappedSrc);
    };

    const id = resolve(moduleName);

    if (kakasooRequire.cache[id]) {
        return kakasooRequire.cache[id].exports;
    }

    const module = {
        exports: {},
        id,
    };

    kakasooRequire.cache[id] = module;
    loadModule(id, module, kakasooRequire);

    return module.exports;
}

kakasooRequire.cache = {};
```

- resolve를 추가한 모습이에요.
    - 실제로는 상대 경로, 절대 경로, 확장자 명, 코어 모듈, 없으면 node_modules까지 탐색하게끔 설계가 되어야 하지만, 여기서는 상대경로만 찾도록 되어 있어요.
    - 코어 모듈을 찾는 로직은 위에 있지만, 각각 별개로 만들어놔서 둘을 합쳐야 온전하겠죠?
- cache도 넣었어요, 그냥 빈 객체일 뿐입니다.

주목할 점은, loadModule에서의 id는 resolve된 결과물,

module은 id에 맞게 생성된 각각의 모듈, require은 require 자기 자신을 담은 파라미터라는 점이죠.

```jsx
module.exports = { a: 3 };
```

```jsx
const { a } = kakasooRequire("./test.js");

console.log(a); // 3
```

- 이 코드는 정상적으로 동작합니다!
- 코어 모듈을 먼저 만들어야 하는 이유가 사실, 그게 없으면 fs을 부를 수 없어서였어요.

# 더 알아보면 좋을 것들?

```jsx
eval(wrappedSrc);
```

- Q : 정말로 eval을 사용하고 있나요?

저도 아직 보는 중이긴 한데, NativeModule을 호출할 때에는 eval 대신에 runInThisContext 라고,

별도의 함수를 만들어서 사용하고 있어요.

```jsx
function runInThisContext(code, options) {
    const script = new ContextifyScript(code, options);
    return script.runInThisContext(); // 바깥의 runInThisContext()랑 다릅니다!
}
```

이런 형태를 하고 있는데요,

node.js의 vm처럼, 현재 코드가 실행되고 있는 context가 아닌 다른 context에서 코드를 실행시키고,

그 결과물을 module.exports에 담게 하는 거죠.

eval은 아무래도 문제 요소가 많아서 실제 코드에 사용하기는 좀 어렵지 않나 싶어요.

# 이걸 배워서 어디에 써요?

사실 ESM 이라는, Node.js를 계속 사용할 거라면, import/exports에 더 익숙해져야 할 거에요.

정식으로 포함된 것은 ESM ( ES module )이고, 앞으로는 많은 코드가 위처럼 작성될 거니깐요.

다만 import/require가 나오기 전에는 많은 개발자들이 서버 쪽 코드를 작성하기 위해 고민했대요.

프론트 코드는 script tag를 사용해 넣기만 하면,

다른 script의 코드가 전역으로 쓰이기 때문에 공유해서 사용할 수 있어서 모듈 시스템이 필요없는데,

서버 쪽에서는 아키텍처 전체 구조를 짜기 위해서라도 파일을 부르고 읽는 건 필수적이었으니깐요.

그 결과로 만들어진 게 require() 메서드였고, 사실 상 이게 commonJS의 핵심인 거 같아요.

그래서, 아직도 많은 프로젝트가 이렇게 이루어져 있을 거고, 설령 그게 다 사라진다고 하더라도,

서로 다른 파일 간에 코드를 교환하기 위한 모듈 시스템을 알아두는 건 큰 도움이 된다고 생각해요.

그 패턴을 알아두는 것만 해도 어딘가에 써먹을 수 있는 요지가 있지 않나... 생각해요.
