---
title: "Express 서버와 React: Proxy 활용과 빌드 및 헤로쿠(Heroku) 배포"
date: "2018-12-21"
categories:
- React
excerpt: |
  Dev 환경에서 Proxy를 이용해 Express 서버와 Create-React-App 서버를 함께 돌리는 방법을 살펴본다. 이후 빌드(Build)를 하고 Heroku에 해당 앱을 배포하는 과정에 대해 알아본다.
feature_text: |
  ## Express 서버와 React: Proxy 활용과 빌드 및 헤로쿠(Heroku) 배포
  로컬에서 Proxy를 활용해 Express와 CRA의 서버를 동시에 구동하고, 클라우드 배포시엔 빌드를 통해 Express가 서버로 작동하게 한다.
feature_image: "https://images.unsplash.com/photo-1485627941502-d2e6429a8af0?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1350&q=80"
image: "https://images.unsplash.com/photo-1485627941502-d2e6429a8af0?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1350&q=80"
---
# Express 서버와 React 함께 사용하기
Create-React-App을 이용해 React 프로젝트를 생성하면 자동으로 서버가 함께 생성된다.
개발 과정에서 서버를 따로 구축하지 않고도 바로바로 프로젝트의 보여지는 모습을 확인할 수 있어 편리하다. 그러나, Express로 구축된 서버에 React 어플리케이션을 물려야 할 경우, 두 개의 서버가 존재하게 된다.

그러나, 결론부터 말하자면 React 어플리케이션을 Build하면, Create-React-App(CRA)d에서 제공하는 서버는 자동으로 포함되지 않는다. /build/ 디렉토리에는 Webpack과 Babel 등을 통해 빌드된 번들 등이 담기지만, 서버는 제외되어 있다.

Heroku나 AWS에 배포할 경우, 기존에 존재하는 서버에서 라우팅 설정만 수정해주면 React를 기존 서버에서 바로 사용할 수 있게 된다.

## Dev모드와 Prod 모드
dev는 development 모드를, prod 모드는 production 모드를 의미한다.

상업용 프로젝트를 할 때엔 종종 dev와 prod로 모드를 나눠서 작업하곤 한다. 이를 통해 보안 상에 생길 수 있는 이슈를 미연에 방지하고, 상용 프로덕트의 경우 DB와 data가 오염되지 않은 클린 한 상태를 유지하게 한다. 그 밖에도 여러 장점이 있지만 여기서는 개발의 편리성을 위해 이 방식을 채택한다.

CRA로 React 프로젝트를 생성하면 CRA에서 제공하는 서버가 기본으로 딸려온다. 이로 인해 프록시(Proxy) 설정을 해줘야 하는 불편함이 있지만, React 프로젝트를 바닥부터 작성하지 않아도 되어 개발 속도가 빠르고 과정이 매우 편해진다. 만약 바닥부터 짜야 한다면 각종 보일러플레이트를 직접 작성해야 하여 개발 속도가 더뎌진다. 또, CRA 서버가 없다면 매번 빌드해서 아웃풋을 확인해봐야 하니 매우 번거로울 것이다. 

따라서 React 프로젝트를 개발할 때는, CRA(create-react-app)을 활용해 생성하고, 이를 통한 장점을 활용해 빠른 개발을 진행하는 것이 바람직하다.

한편, React 프로젝트를 Build하면 CRA가 생성한 서버는 자동으로 제외되어 build 디렉토리가 생성된다. 이를 배포하게 되면 express 단일 서버에서 build 디렉토리 내의 React 앱을 배포하는 방식으로 작동하게 된다.

이처럼 dev 상황과 prod 상황은 서로 다른 환경을 가진다. dev에서는 Express서버와 CRA 서버가 동시에 존재하고, prod에서는 Express서버만 존재하게 된다.

Express와 React를 함께 사용하는 방법은 아래 도표와 같다. dev모드에서는 proxy를 활용해 두 개의 서버를 연결해준다. 한편, prod모드에서는 CRA 서버가 없이 build 디렉토리만 존재하므로 하나의 서버에서 React 프로젝트를 서비스 하게 되는 것이다.

<img src="https://github.com/ChaeWonKong/chaewonkong.github.io/blob/master/assets/post_img/dev-prod.png?raw=true"
/>
## 프로젝트 구조
<img src="https://github.com/ChaeWonKong/chaewonkong.github.io/blob/master/assets/post_img/structure.png?raw=true" height="500px"/>

프로젝트 구조는 위와 같다. Pilot이라는 디렉터리 안에 서버를 구현하고, 하위 디렉토리로 client라는 디렉토리 안에 React 앱을 구현한다.

package.json이 메인 디렉토리와 client 디렉토리 내에 모두 존재하는 것을 확인할 수 있다. node_modules/ 디렉토리 역시 메인 디렉토리와 client 디렉토리 내에 모두 존재한다.

프로젝트 구조가 복잡해 보이지만, 실제로 배포할 때는 client/ 디렉토리 내에 build라는 디렉토리가 생성되고 거기에 압축된 파일들이 저장되며 그 디렉토리만 활용할 것이다.

## 프로젝트 만들기
```
$ mkdir pilot
$ cd pilot
$ yarn init
```
yarn init 후 연속으로 엔터를 친다. 이후 ls 로 확인해보면 package.json이 생성된 것을 알 수 있다.

이후 index.js파일을 생성한다. 이 파일에 Express 기반 서버를 생성할 것이다.
## 서버 구현하기
pilot/index.js에 express를 이용한 서버를 구현할 것이다.
먼저 express를 설치해보자.
```
$ yarn add express
$ yarn add nodemon
```

nodemon은 수정사항이 있을 경우 자동으로 서버를 새로고침 해준다.

이후 pilot/index.js에 다음 코드를 추가한다.

```javascript
const express = require('express');

const app = express()

app.get("/api/greeting", (req,res) => {
  res.send("Hello World!")
})

const PORT = process.env.PORT || 5000;

app.listen(PORT);
```
PORT의 경우 이후 Heroku에 배포하는 경우를 고려해 설정한 것이다. Heroku의 경우 내부에서 어떤 포트를 사용하는지 우리가 알 수 없다. 따라서 Heroku에서 부여하는 포트를 사용하도록 설정해준다. 로컬에서 실행할 경우 'http://localhost:5000'에서 확인 가능하다. 

다음으로 pilot/package.json을 열어 다음과 같이 수정한다.
```javascript
{
  ...,
  "scripts": {
    "server": "nodemon index.js"
  },
  "dependencies": {
    "express": "^4.16.4",
    "nodemon": "^1.18.9"
  }
}
```
scripts에 {"server" : "nodemon index.js"}를 추가했다.

작성을 완료했으면, 저장하고 실행해 확인해보자.
```
$ yarn run server
```

'localhost:5000/api/greeting'에 가서 "Hello World!"가 제대로 나타나는지 확인한다.

## CRA로 React 프로젝트 만들기
다음으로 CRA를 이용해 pilot/ 디렉토리 안에 React 프로젝트를 생성할 것이다.
pilot 디렉토리 안에서 다음 명령을 실행한다.

```
$ yarn create react-app client
```
실행이 완료되면 client 디렉토리가 생성된 것을 확인할 수 있다.

src/App.js를 열고, 간단하게 proxy가 제대로 설정되었는지 확인할 수 있도록 링크를 삽입하자.
```javascript
import logo from "./logo.svg";
import "./App.css";

class App extends Component {
  render() {
    return (
      <div className="App">
        <header className="App-header">
          <img src={logo} className="App-logo" alt="logo" />
          <a className="App-link" href="/api/greeting">
            Greeting
          </a>
        </header>
      </div>
    );
  }
}

export default App;
```
App.js의 내용을 위와 같이 변경한다.
이제 localhost:3000에 가게 되면 Greeting이라는 링크를 만나게 될 것이다. 

## 프록시(Proxy) 설정하기
react-scripts의 버전이 2 이상인 경우 http-proxy-middleware를 설치해 setupProxy.js라는 파일을 통해 proxy 설정을 해줘야 한다.

### react-scripts 버전이 2 이상인 경우
```
yarn add http-proxy-middleware
```

이후 client/src에 setupProxy.js라는 파일을 생성, 다음 코드를 입력하고 저장한다.
```javascript
const proxy = require("http-proxy-middleware");

module.exports = function(app) {
  app.use(proxy("/api/greeting", { target: "http://localhost:5000" }));
};
```

http-proxy-middleware는 앱이 실행될 때 src/ 디렉토리 내에서 setupProxy.js 파일을 찾고, 있을 경우 이 파일의 설정을 참고해 proxy를 설정해준다. 우리는 "/api/greeting"이라는 상대 경로로 요청이 들어올 경우, localhost:5000의 서버를 이용하도록 설정했다.

만약 "/api"로 시작하는 모든 경로에 대해 리디렉션을 실행하고 싶다면 아래처럼 수정하면 된다.
```javascript
app.use(proxy("/api", { target: "http://localhost:5000" }));
```
http-proxy-middleware를 사용하는 경우 client/package.json에는 proxy 설정이 있으면 안 된다. 혹시라도 "proxy":"http://localhost:5000"을 추가했다면 해당 열을 삭제하도록 하자.

### react-scripts 버전이 2 이하인 경우
client/package.json의 scripts 밑에, "proxy": "http://localhost:5000"을 추가해주면 된다.
```javascript
{
  "name": "client",
  "version": "0.1.0",
  "private": true,
  ...,
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject"
  },
  "proxy": "http://localhost:5000",
  ...
}
```

위 코드블록과 같이 proxy를 추가해주자.
## 로컬에서 작동 확인하기
여기서 문제가 있다. 서버가 2개이고, CRA 서버에는 프록시 설정도 마쳤다. 그러나 어떻게 두 개의 서버를 동시에 실행할 것인가?

다시 루트 디렉토리 pilot/으로 돌아가 루트 디렉토리 내에 concurrently를 설치하고, package.json을 수정해보자.
pilot/
```
$ yarn add concurrently
```
concurrently는 두개의 서버를 동시에 구동시켜줄 것이다.

pilot/package.json을 열고 scripts 부분을 다음과 같이 수정한다.
```javascript
"scripts": {
    "server": "nodemon index.js",
    "client": "npm run start --prefix client",
    "dev": "concurrently \"npm run server\" \"npm run client\""
  },
```
백슬래시를 이용해 큰따옴표 안에서 끈따옴표 부분이 escpae할 수 있도록 해준다.

이제 실행해볼 준비가 다 되었다.

```
$ yarn run dev
```

서버가 구동되길 기다리면 자동으로 브라우저가 열리고 localhost:3000으로 이동할 것이다.

여기서 Greeting이라는 링크를 클릭해보자.
localhost:3000/api/greeting으로 이동하고 "Hello World!"가 정상적으로 표시되는 것을 확인할 수 있다.
## 빌드(Build)하기
자 이제 빌드할 시간이다. pilot/client 디렉토리로 이동하고 다음 명령을 실행한다.
```
$ yarn build
```
자동으로 빌드가 시작되며, 빌드가 완료되면 client/ 디렉토리 내에 build/ 디렉토리가 생성된다.

앞서 설명하였듯이, 빌드 과정을 거치게 되면 CRA 서버가 제외된 상태로 배포용 파일들이 build에 생성된다.

로컬에서 실행할 때는 자동으로 서버가 2개 구동되었다. 따라서 프록시 설정으로도 충분하였지만, 이제는 Express 서버만 구동된다. 따라서 React 앱에 접속할 수 있는 Route 설정이 필요하다. 

pilot/index.js 파일을 열고 다음과 같이 수정한다.
```javascript
if (process.env.NODE_ENV === "production") {
  app.use(express.static(path.join(__dirname, "client/build")));
}

app.get("/api/greeting", (req, res) => {
  res.send("Hello World!");
});

app.get("/", (req, res) => {
  res.sendFile(path.join(__dirname, "client/build", "index.html"));
});
```

process.env.NODE_ENV === "production"이라는 조건을 주고, 만약 그렇다면 client/build에서 static 파일을 사용하도록 설정한다.

또, "/" 루트를 추가해 client/build/index.html이 디폴트 페이지가 될 수 있도록 라우팅 설정을 추가한다.
여기서 "/" 루트는 반드시 "/api/greeting" 루트 밑에 와야 에러가 발생하지 않는다.

## 배포를 위해 수정하기
이제 Heroku에 배포하기 위한 간단한 수정만 하면 완성된다.

pilot/package.json을 열고 다음을 추가하자.
"heroku-postbuild": "cd client && npm install && npm run build"

```javascript
  "scripts": {
    "heroku-postbuild": "cd client && npm install && npm run build",
    "server": "nodemon index.js",
    "client": "npm run start --prefix client",
    "dev": "concurrently \"npm run server\" \"npm run client\""
  }
```
client의 node_modules를 설치하고 client를 빌드하도록 명령을 추가해둔다. 이렇게 해두면, Build를 깜빡하더라도, 혹은 React 프로젝트 내에서 새로운 라이브러리를 설치하더라도 문제 없이 작동하게 할 수 있다.

## 배포 및 확인하기
자 이제 Heroku에 배포할 시간이다.

먼저 Heroku를 사용하는 방법부터 간단히 살펴보자.
> [Heroku Node.js Documentation](https://devcenter.heroku.com/articles/getting-started-with-nodejs)

필요하다면 위의 도큐멘테이션을 참고하길 바란다.

Mac 사용자라면,
```
$ brew install heroku/brew/heroku
```

Ubuntu 사용자라면,
```
$ sudo snap install heroku --classic
```

다음으로 Heroku 사이트에서 가입을 한다.

만약 아직 Git 리포지토리를 생성하지 않았다면, pilot 디렉토리 내에서 git repository를 생성한다.
```
$ git init
```

이후 Heroku에 로그인 한다.
```
$ heroku login
```

자 이제 heroku 환경을 생성하자.
```
$ heroku create
```
이 명령을 실행한 후 다음 명령어를 통해 heroku가 remote로 성공적으로 추가되었는지 확인한다.

```
$ git remote -v
```
결과가 다음처럼 뜨면 성공이다.
```
$ git remote -v
heroku  https://git.heroku.com/react-express-pilot.git (fetch)
heroku  https://git.heroku.com/react-express-pilot.git (push)
```
URL은 같지 않아도 상관 없다.

자 이제 준비가 완료되었다. 
```
$ git add .
$ git commit -m "Complete express and react"
```
모든 변경내용들을 커밋한다.

heroku 서버에 배포하자.
```
$ git push heroku master
```
Heroku에 master 브랜치의 내용을 push하라는 명령이다.
시간이 좀 소요되니 느긋하게 기다려본다. 명령이 실행완료되면 사이트를 열어 작동을 확인한다.
```
$ heroku open
```
React 시작화면이 뜨고 Greeting 링크가 보이는지 확인한다.
Greeting 링크를 눌러보자. 하얀 화면에 "Hello World!"가 적혀 있다면 성공이다!

## 왜 이 구조인가?
이 구조 말고도 서버를 두개 두고 API 서버에서 data를 fetching 하는 방법을 사용할 수도 있다. 즉 서버를 독립적으로 두 개 두는 것이다. 이런 접근도 틀린 것은 아니며, 상황에 따라서는 더욱 바람직할 수도 있다.

그러나, 만약 React 앱 내에서 passport.js를 이용한 OAuth를 사용해야 하거나, 쿠기 기반의 사용자 인증 등 서버를 어쩔 수 없이 직접 구축해야 한다면, 지금 작성한 구조가 더 효율적일 수 있다.

지금 작성한 구조에서는 Dev와 Prod 환경이 구분된다. Dev 환경에서는 Express 서버와 CRA 서버가 동시에 구동되며, Prod 환경에서는 Express 서버만 존재하고 React앱은 build 폴더 내에 번들 형태로 존재한다.

이 구조를 선택하게 되면 대규모의 복잡한 앱 제작 과정에서 간단하게 테스트하며 작동을 확인해볼 수 있다. 또, 개발이 완료된 후 프로덕션 환경에서는 하나의 서버만 작동하고, 별도의 수정 작업이 거의 필요하지 않아 편리하다.