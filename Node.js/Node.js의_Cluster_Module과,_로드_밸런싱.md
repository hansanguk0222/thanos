
# 이 글에서는 무얼 말하는가?
---

너무 장황하게 설명하는 대신에 이걸 왜 해야 하고, 어떻게 해야 하는지, 그리고 하면 뭐가 좋은지만 설명하고자 합니다. 대신에 이와 관련된 내용들은 링크를 걸어놓겠습니다.

> **🚧 이 글은 Node.js 16.9.1 version을 기준으로 작성되었습니다.**

> _"이 글은 Node.js의 클러스터와 PM2의 로드 밸런싱에 대해서 설명합니다. 추가적으로 이를 활용하면서 발생할 수 있는 장애들을 서술합니다. 이 글이 다른 사람들에게, 삽질을 줄이는 데 도움이 되면 좋겠습니다."_

---
# Node.js Cluster Module
---

> _Node.js는 확장성 있는 네트워크 애플리케이션(특히 서버 사이드) 개발에 사용되는 소프트웨어 플랫폼이다. 작성 언어로 자바스크립트를 활용하며 논블로킹(Non-blocking) I/O와 단일 스레드 이벤트 루프를 통한 높은 처리 성능을 가지고 있다. - 위키백과_

Node.js는 기본적으로 싱글 스레드입니다. Node.js 어플리케이션은 하나의 코어에서 실행되기 때문에 CPU가 멀티 코어인 경우에는 하나를 제외한 나머지 코어를 활용하지 못하는 꼴이 되는데, 이는 컴퓨터가 가진 성능을 온전히 발휘하지 못하는 일이 되기 때문에, Node.js에서는 클러스터라는 기능을 제공하고 있습니다. 클러스터는 Node에서 제공되는 코어 모듈의 일부로, CPU의 코어 수보다 한 개 적은 수 <span style ="color:#aaaaaa">_-하나는 마스터 프로세스가 되어야 하기 때문-_</span> 의 워커 프로세스를 생성하여, 각기 다른 프로세스에서 코드가 실행되도록 도와줍니다. 지금부터는 아주 재밌어질 건데, 코드를 보면서 논리 흐름을 따라와주길 바랍니다.

```javascript
// 코드 설명 1번 : cluster Module을 사용하여 cpu 숫자만큼의 process를 생성한다.
import cluster from "cluster";
import { cpus } from "os";
import process from "process";

const numCPUs = cpus().length;

if (cluster.isPrimary) {
    console.log(`Primary ${process.pid} is running`);

    // Fork workers.
    for (let i = 0; i < numCPUs - 1; i++) {
        cluster.fork();
    }
} else {
    console.log(`Worker ${process.pid} started`);
}

```

위 코드는 os module로부터 CPU의 코어 수를 가져온다. cluster module에서는 isPrimary <span style ="color:#aaaaaa">_-isPrimary는 node.js 16 version에 등장하므로 LTS에서는 확인할 수 없습니다.-_</span> 는 현재 코드가 동작할 때의 코드가 master process인지를 알려주는 프로퍼티입니다. 이 코드가 처음 실행될 때는 무조건 master이기 때문에 if문 내부를 타게 됩니다. if문을 타게 되면 내부에는 cluster.fork() 라는 메서드가 있습니다.

cluster.fork()는 하위 프로세스들을 생성하는 로직인데, 이 로직이 실행되면 다른 프로세스를 통해 해당 코드 전체를 다시 실행하게 됩니다. 이 때, **다시 실행된 코드에서는 else문을 타게 됩니다.** 따라서 실행 결과는 아래처럼 될 것입니다.

>
Primary 9180 is running
Worker 9704 started
Worker 16840 started
Worker 2340 started
Worker 11572 started
Worker 14804 started
Worker 14264 started
Worker 152 started
Worker 19188 started

master process가 실행된 다음, Worker들이 실행됩니다. 또한 각 프로세스 아이디는 실행될 때마다 바뀔 겁니다. 그래서 총 9개의 프로세스가 실행되고 있다고 보면 되겠습니다. 사실 CPU의 코어 수만큼 프로세스를 여는 게 맞지만, 실질적인 동작은 Worker들이 처리하기 때문에 마스터 프로세스를 포함해, 코어보다 1개 많은 프로세스가 열렸습니다. 당연히, 이렇게만 열어놨으면 아무런 의미가 없습니다. 이제 이 Worker들에게 일을 시켜야할 때입니다.

```javascript
import cluster from "cluster";
import { cpus } from "os";
import process from "process";
import http from "http";

const numCPUs = cpus().length;

if (cluster.isPrimary) {
    console.log(`Primary ${process.pid} is running`);

    // Fork workers.
    for (let i = 0; i < numCPUs; i++) {
        cluster.fork();
    }
} else {
    console.log(`Worker ${process.pid} started`);

    http.createServer((req, res) => {
        res.writeHead(200);
        // res.end("hello world\n");
      	res.end(process.pid.toString());
    }).listen(8000);
}
```

이는 Node.js document에 있는 코드입니다. http.createServer()를 사용해서 서버를 열었네요. 이 코드에서 process.pid를 확인하기 위해 res.end()의 파라미터로 넣어주었습니다. 이제 localhost의 8000번 포트에서 결과를 볼 수 있습니다. 다만 이렇게 사용을 해도 하나의 process가 일을 하는 걸 볼 수 있습니다. 이는 우리가 원한 결과는 아닐 것 같습니다.

> _Worker가 늘어나는 걸 보고 싶다면, for문에 아래 코드를 삽입해주세요._

```javascript
const workers = Array.from({ ...cluster.workers, length: i });
console.log(workers.length);
```

---
# Process끼리 메시지 주고 받기
---

일을 분배하지 않을 거라면 Worker를 만드는 의미가 없겠습니다. 그래서 이번에는 각 process 끼리 통신을 하고, 이를 통해 일을 나누는 걸 해보겠습니다. PM2를 배우고 싶은 분이라면, 뒤로 가셔도 좋습니다. 사실 몰라도 가능하거든요. 다만, 각 프로세스끼리 통신한다는 걸 배운다면 PM2를 더 우아하게 다룰 수 있을 겁니다. 일단은 아래처럼 코드를 수정해보겠습니다.

```javascript
import cluster from "cluster";
import { cpus } from "os";
import process from "process";
import http from "http";

const numCPUs = cpus().length;

if (cluster.isPrimary) {
    console.log(`Primary ${process.pid} is running`);

    // Fork workers.
    for (let i = 0; i < numCPUs; i++) {
        cluster.fork();
    }

    for (const id in cluster.workers) { // 달라진 부분 1 : 각 Worker들에게 이벤트를 추가했습니다.
        cluster.workers[id].on("message", (message) => {
            console.log(`process id ${id} said : ${message}`);
        });
    }
} else {
    console.log(`Worker ${process.pid} started`);

    http.createServer((req, res) => {
        process.send("제가 처리하겠습니다."); // 달라진 부분 2 : 이벤트 발생 시 메시지를 전달합니다.

        res.writeHead(200);
        res.end("hello world\n");
    }).listen(8000);
}

```

주석에도 써놨습니다만, 두 부분에서 차이가 발생합니다. 하나는 마스터 프로세스가 생성될 때, 각 Worker들에게 이벤트를 걸어준 겁니다. message라는 이벤트를 걸고, 핸들러 함수로는 message를 어느 프로세스가 보냈는지와 그 내용을 출력하게 하는 함수를 걸어줬습니다. 그리고 실제 하위 프로세스들에게는 처리 전에 임의의 메시지를 전달하게 했습니다. else문에는 하나의 worker밖에 없지만, 결과적으로 이 worker가 if문 안의 for문에서 각기 다른 하나의 worker들과 매칭됨을 알 수 있습니다.

```bash
curl http://localhost:8000
```


다른 터미널을 켜서 위 커맨드를 입력하면 마스터 프로세스가 반응하고 있는 걸 확인할 수 있습니다. 이렇듯 프로세스는 마스터와 통신을 할 수 있습니다. 마지막으로 위 코드의 for문 두 개를 하나로 합쳐서, 더 깔끔하게 정리합니다. fork()로 리턴된 결과가 각 worker들입니다.

```javascript
import cluster from "cluster";
import { cpus } from "os";
import process from "process";
import http from "http";

const numCPUs = cpus().length;

if (cluster.isPrimary) {
    console.log(`Primary ${process.pid} is running`);

    // Fork workers.
    for (let i = 1; i <= numCPUs; i++) {
        const worker = cluster.fork();

        worker.on("message", (message) => {
            console.log(`process id ${i} said : ${message}`);
        });
    }
} else {
    console.log(`Worker ${process.pid} started`);

    http.createServer((req, res) => {
        process.send("제가 처리하겠습니다.");

        res.writeHead(200);
        res.end("hello world\n");
    }).listen(8000);
}

```

> _"브라우저에서 테스트해도 좋습니다만, GET 요청이 2번씩 갈지도 모릅니다. 제 컴퓨터 상에서는 그렇게 동작하고 있는데요, 아마 favicon 때문에 그런 것 같습니다."_

---
# Process 간의 불평등한 분배
---

사실 이미 요청을 여러 번 보내보신 분이라면 하나의 Worker만 일을 하고 있는 것을 알아챘을 겁니다. <span style ="color:#aaaaaa">_-대개는 마지막으로 fork()된 프로세스가 일을 처리하곤 합니다.-_</span> 하지만 실제로는 나름대로 일을 분배하기는 합니다. 이를 알 수 없던 건 프로세스가 처리할 일이 적었기 때문으로 보입니다. 한 번 저렇게 만들어진 서버를 과부하 시켜보도록 하겠습니다.

```bash
npm i -g artillery

artillery -r 4000 http://localhost:8000
```

일단 해당 라이브러리를 설치하고 실행시키겠습니다. 이 라이브러리는 부하를 테스트하기 위한 라이브러리인데, -r 옵션은 1sec 당 도착하는 요청 수를 의미합니다. <span style ="color:#aaaaaa">_-실제로는 저것보단 적게 들어가게 됩니다만, 일단 많이 보낸다고 이해하시면 될 것 같습니다.-_</span> 이렇게 요청을 보내면 적어도 프로세스 2개 이상이 일하는 것을 볼 수 있습니다. 거의 2개의 프로세스가 70% 이상을 처리한다고 합니다. 원래대로라면 8개의 프로세스가 균등하게 일해야 할 것 같습니다만, 또 그게 사실은 맞겠지만, 실질적인 동작은 이게 최선입니다.

> _The second approach is where the primary process creates the listen socket and sends it to interested workers. The workers then accept incoming connections directly.
The second approach should, in theory, give the best performance. In practice however, distribution tends to be very unbalanced due to operating system scheduler vagaries. Loads have been observed where over 70% of all connections ended up in just two processes, out of a total of eight._ - node.js document cluster

공식문서에 나온 바에 따르면 이는 round-robin 방식은 맞지만, 운영체제 스케쥴러의 영향을 받기 때문이라고 합니다. 사실 일만 잘 처리한다면야 한 프로세스가 일을 다 처리해도 상관이 없습니다. 다만 엄청나게 많은 트래픽이 몰릴 경우는 어떨까요?

사실, 운영체제만큼 견고한 시스템도 없습니다. 운영체제가 판단하기에 하나의 프로세스로도 충분하다고 생각하기에 일을 분배하지 않은 거라면 존중할 만 하다고 생각합니다. 하지만, 나중에 이야기할 PM2에서는 active handles를 확인할 수 있습니다. 이것들은 커넥션의 수라고 이해하면 되는데, 다르게 말하면 서버에 연결된 클라이언트의 수입니다. 일반적으로는 처리 속도가 빨라서 커넥션의 수가 늘어나더라도 큰 장애는 생기지 않지만, 메모리 측면으로 볼 때는 그리 좋은 일은 아닙니다. 처리가 완료되면 그 만큼 커넥션을 닫아서 메모리를 회수해야 하는데, 한 프로세스에 대기열이 많아질수록 점유하고 있는 메모리 크기도 늘어나게 됩니다. 이는 메모리 누수의 가능성을 높입니다.

개발자 입장에서는 이게 불만족스러울 수도 있습니다. 하지만 이 문제를 해결하기 위해서 자식 프로세스끼리 통신을 구축하고, 또 실제로 분배하는 건 오히려 비용을 늘리는 꼴이 될 것 같습니다. 다시 처음으로 돌아가서, 이 문제를 해결하기 위해서는 별도의 프락시를 두는 게 차라리 저렴해보입니다. 하나의 프락시 서버를 두고, 그 서버가 요청 받아서 직접 분배하는 겁니다. 여기서 가장 좋은 방법은 각 요청에 따른 평균적인 응답 속도를 내서 가장 빠른 측에 보내도록 하는 것이고, 그 다음으로 좋은 방법은 현재까지 받은 요청 수가 가장 적은 쪽으로 요청을 보내는 겁니다. 혹시 이미 뭔지 아셨을까요?

> _"별 다른 설명없이 여기까지 왔습니다만, 이렇게 부하를 나누는 걸 로드 밸런싱이라고 합니다."_

> _"node.js 초기에는 CPU 코어당 여러 프로세스를 생성하기 위해 클러스터 모듈을 사용했다고 합니다. 하지만 이 방식은 코드를 관리하기 더 어렵게 만들었다고 하네요. 확장 및 로드 밸런싱은 Devops 문제이니, node.js 코드는 오로지 Application Logic에 중점을 두어야 한다는 게 현재의 결론인 것 같습니다."_


---
# Process 간의 평등한 분배 (Nginx, PM2)
---

> _"여기서는 nginx와 pm2를 사용하겠습니다."_

미리 말씀드렸다시피 운영체제에게 일임하는 것도 나쁜 방법은 아니지만, 메모리 측면에서 더 개선할 수 있도록 nginx를 사용합니다. <span style="color:#aaaaaa">-_물론 nginx를 사용하는 이유는 정적 자원에 대한 캐시를 포함해 여러 가지 이유가 있습니다. 하지만 여기서는 로드 밸런싱을 이유로 설명합니다._-</span> nginx를 사용하면 위에서 말한 것과 동일한 방식으로 부하를 분배할 수 있게 됩니다. 이 부분은 코드에 대한 설명이 아니므로 구축하는 데에 중점을 둡니다. ngnix에 앞서, 일단은 환경을 세팅하겠습니다.

> _"여기서는 Linux 환경이라 가정하겠습니다. aws ec2 프리티어가 있으신 분들은, 인바운드에 80번 포트가 열려 있는지 확인해주시면 되겠습니다. nginx 구축에 대한 설명은 간단하게만 하고 넘어가겠습니다."_

```bash
apt-get update -y # linux를 처음 실행했다면 업데이트부터 진행한다.
apt-get install nginx -y # 업데이트가 완료되었다면 nginx를 설치한다.
sudo service nginx start #  nginx가 설치되었다면 퍼블릭 호스트로 접속해 nginx default 페이지를 확인한다.

curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.34.0/install.sh | bash # node.js를 설치한다.
. ~/.nvm/nvm.sh # node version manager를 설치한다.
nvm install node # node를 설치한다.
node -e "console.log('Running Node.js ' + process.version)" # node가 동작하는지 확인한다.
```

여기까지가 nginx 및 node.js를 세팅하는 부분입니다. 이제 아래처럼 서버를 작성합니다.

```javascript
// app.js : express server
const express = require("express");
const logger = require("morgan");
const port = 3000;

const app = express();

app.use(logger("dev"));

app.get("/status", async (req, res, next) => {
    const pid = process.pid;
    const instance_number = process.env.NODE_APP_INSTANCE;
    console.log(`${pid} is pm2 instance_number : ${instance_number}`);

    res.send(instance_number);
});

app.listen(port, () => {
    process.send("ready");
    console.log(`application is listening on port ${port}`);
});

```

위처럼 서버를 만들었다면 node 명령어로 실행시키는 대신에 pm2를 사용하겠습니다.

```bash
npm install -g pm2
pm2 start ./app.js -i max
```

이 구절은  pm2를 이용하여 app.js를 실행하며, -i 옵션에 max를 준 것은 사용 중인 기기에서 활용할 수 있는 코어 수를 최대한으로 하라는 뜻이 됩니다.

```bash
pm2 list
```

pm2 list를 입력하면 백그라운드에서 실행 중인 node 서버를 모두 볼 수 있습니다.

```bash
pm2 restart
pm2 reload
pm2 delete pid|app name
```

또한 이 명령어들을 통해 다시 시작하거나, 서버를 재실행하거나, 원하는 프로세스를 삭제할 수 있습니다. pm2를 사용한 것 역시 cluster와 동일하게 동작합니다. 내부에서는 똑같이 cluster를 사용하고 있습니다. 위 입력에 대한 옵션들은 매번 입력하는 대신에 ecosystem.config.js 환경 파일을 만들어서 대체할 수 있습니다. 아래는 제 파일입니다.

```javascript
// ecosystem.config.js
module.exports = {
    apps: [
        {
            name: "app",
            script: "./app.js",
            instances: 0,
            exec_mode: "cluster",
            wait_ready: true,
            listen_timeout: 50000,
        },
    ],
};

```

실행되는 프로세스들은 app이라는 이름이며, 스크립트는 app.js를 사용한다. isntances의 수에 0을 넣은 것은 max와 동일한 의미를 가집니다. 실행 모드는 클러스터를 뜻합니다. wait_ready와 이하 옵션은 나중에 설명하겠습니다. 다음으로 nginx 설정입니다.

```bash
cd /etc/nginx/sites-enabled # 여기는 nginx의 환경 파일이 있는 곳입니다.
sudo rm default # 여기서 default 설정을 삭제합니다.
vi default # 새로 파일을 만듭니다, 내용은 아래처럼 작성해주시면ㄷ ㅚㅂ니다.
```

```bash
# default
server {
        listen 80;
        listen [::]:80;

        access_log /var/log/nginx/reverse-access.log;
        error_log /var/log/nginx/reverse-error.log;

        location / {
                    proxy_pass http://127.0.0.1:3000;
  }
}
```

default를 위처럼 변경시켰다면 nginx를 재실행시켜주면 됩니다.

```bash
sudo nginx -t # 수정한 설정으로 nginx가 정상 동작할 수 있는지를 테스트합니다.
sudo service nginx restart # 문제가 없다면 nginx를 재실행합니다.
```

이제 브라우저에 aws 퍼블릭 도메인을 입력하면 포트 80번 없이도 3000번 포트로 자동 이동하게 되며, 여기서는 pm2로 클러스터링된 서버들로 이동하게 됩니다. 또한 기존처럼 하나의 프로세스가 모든 일을 처리하는 게 아니라 각 서버들이 번갈아가며 로직을 수행합니다!


# pm2와 무중단 배포
마무리입니다. 앞서 pm2 명령어 중에 restart와 reload가 있었습니다. restart는 모든 프로세스를 재실행시키는 반면, reload는 프로세스가 다운되는 일이 없도록 합니다. 무슨 뜻이냐면, 첫번째 프로세스부터 마지막까지 순차적으로 재실행함으로써 프로세스 한 개 이상은 항상 실행 상태가 되도록 하는 것입니다.

그런데 이 역시 완벽하지는 않습니다. 몇몇 프로젝트는 실행 후에 모든 모듈이 load되고 정상 작동하기 위해 실행 후 몇 초 간의 시간이 더 필요할 수 있습니다. 그런데 reload가 다음 프로세스를 호출하는 데에는 이런 경우가 고려되지 않습니다. 따라서 무거운 프로젝트를 reload하게 된다면 그 미세한 시간 사이에 사용자 요청이 유입되도 처리하지 못하는 경우가 발생할 수 있습니다. 따라서 reload 만 가지고는 완벽한 처리가 불가능합니다.

```javascript
// ecosystem.config.js
module.exports = {
    apps: [
        {
            name: "app",
            script: "./app.js",
            instances: 0,
            exec_mode: "cluster",
            wait_ready: true,
            listen_timeout: 50000,
          	kill_timeout: 5000,
        },
    ],
};
```

앞서 설명을 보류했던 wait_ready와 listen_timeout은 이런 까닭에 등장합니다. wait_ready는 실행이 완료된 경우 process로부터 ready message를 보낼 것이라는 약속을 의미합니다. 이 설정을 true로 했다면 우리의 http 서버, 또는 express file에 process.send('ready')가 추가되어야 합니다. listen_timout은 pm2의 마스터 프로세스가 ready 응답을 받을 때까지 대기하는 시간을 의미합니다. 여기서는 적어도 50,000ms를 대기합니다.

```javascript
app.listen(3000, () => {
  process.send('ready');
})
```

각각의 Wokrker들은 서버를 열 때, listen 메서드에 대하여 다음 콜백을 지닙니다. 서버가 무사히 대기 상태가 되었다면 이 시점에서 ready를 반환하는 코드입니다. **새로 생성된 프로세스가 ready를 반환하면, 마스터 프로세스는 해당 프로세스의 이전 version 프로세스에게 application을 종료할 것을 지시합니다.** 여기서 SIGINT라는 값을 보내는데, 이를 보내고도 일정 시간이 지나도록 프로세스가 살아있다면 다음에는 프로세스를 강제 종료시킵니다. 이렇게 해서 이전 application이 종료되고, 프로세스는 최신 상태로 모두 바뀌게 됩니다. 이로써 정말로 무중단 배포가 가능해졌습니다. 하지만 마지막 문제가 남았습니다. 다음의 경우를 생각해야 합니다.

논리 흐름을 따라가보세요.

1. reload 명령으로 새로운 프로세스가 생성되는 중이다.
2. 생성되는 도중이기 때문에 기존의 프로세스를 닫지 않고, 계속 대기한다.
3. 만약 생성이 완료되면 process는 'ready' 메시지로 완료됐음을 알릴 것이고,
4. 마스터 프로세스는 해당 프로세스의 이전 version을 종료하라는 뜻으로 인지할 것이다.
5. 따라서 마스터는 이러한 명령을 이전 프로세스에게 보낼 텐데,
6. 생성이 완료되기 전, 이 짧은 사이에도 사용자 요청이 들어올 수도 있다.
7. 이 경우 이전 프로세스가 여전히 그 작업을 수행 중에 있을 것이고,
8. 이 로직을 처리하고 있어서 제 시간에 프로세스를 종료할 수가 없었다.
9. 어쨌거나 사용자 요청을 처리하는 것이 더 중요할 텐데,
10. 마스터 프로세스는 이전 프로세스가 응답하지 않는 거로 간주하여 강제 종료시킨다.
11. 유저의 요청은 묵살된다.

이런 경우를 해결하기 위해서 ecosystem.config.js에는 새로운 값을 추가합니다.

```javascript
module.exports = {
    apps: [
        {
            name: "app",
            script: "./app.js",
            instances: 0,
            exec_mode: "cluster",
            wait_ready: true,
            listen_timeout: 50000,
          	kill_timeout: 5000, // 새로 추가된 부분
        },
    ],
};
```

이 부분은 프로세스를 강제 종료하기 전까지 대기하는 시간을 의미합니다. 만약 keep-alive와 같이 헤더 설정이 있으면 헤더에 Connection을 close 값으로 설정해주면 됩니다.

# 결론

아무 생각 없이 그저 PM2, nginx만 쓰다가, 실질적으로 부하를 줄이는 데 도움이 되는지 알고 싶어 찾아보게 되었습니다. 생각보다도 PM2가 매우 깊이가 있었고, 다른 것들 역시도 마찬가지였습니다. 이상의 내용은 Node를 다루는 개발자라면 누구나 흥미로울 법 해서 정리하게 되었습니다. 읽어주신 분들께 감사합니다.

여담으로, 많은 사람들이 Cluster Module과 PM2의 round-robin 알고리즘에 대해 의문을 가진 듯 했습니다. 다들 저와 같이, 각 프로세스가 동등하게 일하는 것을 원했지만, 운영체제가 수행하는 일이라는 답변만 있었습니다. nginx를 활용한 방법이 Node 레벨에서 해결한 것이 아니라서 많이 아쉽습니다. 실제로는 nginx에서 해결하는 게, Node 에서 해결하는 것보다 각 역할이 분리되어 더 좋다는 것을 알지만, 연습 삼아서라도 Node 레벨에서 해결하고 싶었던 게 아쉬움이 큽니다. 이런  방법이나 키워드를 아시는 분은 댓글을 남겨주시면 감사하겠습니다.


# 참고 자료
1. [PM2를 활용한 Node.js 무중단 서비스하기 - Line Engineering](https://engineering.linecorp.com/ko/blog/pm2-nodejs/)
	- 이 글을 쓰기에 가장 많은 영감을 받았습니다.
2. [Node.js Cluster Document](https://nodejs.org/api/cluster.html)
	- cluster에 대한 자세한 설명은 공식 문서를 참고하면 좋을 것 같습니다.
3. [PM2 github source code](https://github.com/Unitech/pm2/blob/1d81757d1c94d46c015a3a26d626c1e13da6a15d/lib/God/ClusterMode.js)
4. [[네이버 클라우드 플랫폼] Load Balancer(4) - PM2 및 부하 테스트](https://mingoogle.tistory.com/7)
	- 이 글에서는 artillery 라이브러리에 대해서 알게 되었습니다.
5. [What is the difference between pm2 restart and pm2 reload - stackoverflow](https://stackoverflow.com/questions/44883269/what-is-the-difference-between-pm2-restart-and-pm2-reload)
6. [Is it better to use the cluster module with Node.js servers or to place several Node.js servers (running on same machine) behind a load balancer? - Quora](https://www.quora.com/Is-it-better-to-use-the-cluster-module-with-Node-js-servers-or-to-place-several-Node-js-servers-running-on-same-machine-behind-a-load-balancer)
7. 그 외 많은 stackoverflow 글의 도움을 받았습니다.

위 순서는 제가 도움 받은 글들을 나열한 것입니다. 특히나 첫번째 글은 제가 pm2에 흥미를 갖고 직접 찾아보게 한 동기이니 만큼 한 번 보셨으면 합니다. 제 글보다도 정리가 잘 되어 있어 많은 도움을 받을 수 있을 것 같습니다.
