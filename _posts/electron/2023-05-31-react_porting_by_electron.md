---
layout: post
current: post
cover: assets/built/images/electron/electron_logo.jpg
navigation: True
class: post-template
author: sian
title: "[ElectronJS] React 앱을 데스크톱 응용프로그램으로 변환(포팅)하기"
comments: true
categories:
  - Electron
tags:
  - Electron, ElectronJS, React
date: "2023-05-31 17:00:00"
---

## 서론

React 기반의 웹 사이트를 데스크톱 응용 프로그램으로 포팅하는 방법이 있을까 싶어서 찾아보던 중 알게 된 ElectronJS.
ElectronJS를 이용하면 굉장히 간단한 설정 몇번으로 React 기반(정확하게는 JS, HTML, CSS 기반)의 데스크톱 응용프로그램을 만들거나 포팅을 할 수 있다.
이 포스트는 React -> Windowns 응용 프로그램으로 포팅하는 아주 기본적인 설명을 작성한 포스트이다.


### 환경

- Windows 11
- Visual Studio Code 1.63.2
- npx 9.5.1
- yarn 1.22.19
- React 18.2.0, react-scripts 5.0.1 (create-react-app)
- electron 25.0.0
- electron-builder 23.6.0


## 1.	React 앱 준비

본 포스팅을 위해 create-react-app을 통해 기본 react 앱을 생성하기로 하였다.
윈도우 커맨드 창을 열어서 앱을 생성하기 원하는 위치로 이동한 뒤 아래 명령어를 입력한다.

```terminal
npx create-react-app my-electron-app
```
![img](\assets\built\images\electron\react_porting_by_electron_1.jpg)

프로젝트가 생성되면 해당 프로젝트 경로에서 아래 명령어를 통해 react 앱을 실행시킨다.

```terminal
yarn start
```
![img](\assets\built\images\electron\react_porting_by_electron_2.jpg)
![img](\assets\built\images\electron\react_porting_by_electron_3.jpg)
![img](\assets\built\images\electron\react_porting_by_electron_4.jpg)

create-react-app 기본포트인 3000번 포트로 접속하여 react 앱이 잘 올라갔는 지 확인한다.


## 2.	ElectronJS 준비 및 개발환경에서 구동해보기

react 앱이 잘 구동되는 모습을 확인했다면 이제 electron 관련된 모듈들을 프로젝트에 추가한다.

아래 명령어를 통해 프로젝트에 필요한 electron 관련 모듈을 추가하자.

(이미지에는 -d 로 되어 있는데 -D 로 대문자로 입력해주자.)

```terminal
yarn add -D electron electron-builder
```
![img](\assets\built\images\electron\react_porting_by_electron_5.jpg)

프로젝트 폴더로 이동하여 **package.json** 파일을 열어서 두 모듈이 잘 추가가 되었나 확인한다.

![img](\assets\built\images\electron\react_porting_by_electron_6.jpg)

모듈이 추가된 모습을 확인했다면 아래 코드를 **package.json** 파일에 입력한다.

```json
"scripts": {
  ...,  // 기존 스크립트들
  "electron-start": "set ELECTRON_START_URL=http://localhost:3000 && electron .",
},
"main": "public/Main.js",
"homepage": "./",
```
![img](\assets\built\images\electron\react_porting_by_electron_7.jpg)

위에 작성한 main 속성에 해당하는 js 파일을 참조하여 electron을 실행하기 때문에 프로젝트의 **public** 경로 안에 **Main.js** 파일을 생성한 뒤 아래 코드를 붙여넣는다.

```js
const {app, BrowserWindow} = require('electron');
const path = require('path');
const url = require('url');

function createWindow() {
    /*
    * 넓이 1920에 높이 1080의 FHD 풀스크린 앱을 실행시킵니다.
    * */
    const win = new BrowserWindow({
        width:1920,
        height:1080
    });

    /*
    * ELECTRON_START_URL을 직접 제공할경우 해당 URL을 로드합니다.
    * 만일 URL을 따로 지정하지 않을경우 (프로덕션빌드) React 앱이
    * 빌드되는 build 폴더의 index.html 파일을 로드합니다.
    * */
    const startUrl = process.env.ELECTRON_START_URL || url.format({
        pathname: path.join(__dirname, '/../build/index.html'),
        protocol: 'file:',
        slashes: true
    });

    /*
    * startUrl에 배정되는 url을 맨 위에서 생성한 BrowserWindow에서 실행시킵니다.
    * */
    win.loadURL(startUrl);
}

app.on('ready', createWindow);
```
![img](\assets\built\images\electron\react_porting_by_electron_8.jpg)

그 다음 react 앱이 실행되어 있는 상태로 터미널을 한개 더 실행하여 **electron-start** 명령어를 입력한다.

```terminal
yarn electron-start   # 다른 터미널에서 yarn start로 react 앱이 구동되고 있어야 함!
```
![img](\assets\built\images\electron\react_porting_by_electron_9.jpg)

electron 구동이 완료되면 아래와 같이 데스크톱 응용 프로그램으로 실행되는 모습을 확인할 수 있다.

![img](\assets\built\images\electron\react_porting_by_electron_10.jpg)

이미 구동되고 있는 3000포트의 react 앱을 불러오는 것이기 때문에 코드 수정 등이 바로바로 반영되는 모습을 확인할 수 있다.
예시로, 아래와 같이 메인 화면에 문구를 추가하고 저장하면 응용 프로그램이 바로 수정되는 모습을 볼 수 있다.

![img](\assets\built\images\electron\react_porting_by_electron_11.jpg)


## 3.	응용 프로그램으로 빌드하기

2번에서 진행한 내용은 개발단계로써, 3000포트에 띄운 react 앱을 응용 프로그램이 호출하는 것 뿐이다.
하지만 ElectronJS를 통해 빌드를 하면 위와 같이 react 앱이 미리 구동되어 있을 필요가 없이 자체적으로 react 앱을 실행시켜서
정말 하나의 데스크톱 응용 프로그램처럼 동작하게 된다.

이제 빌드를 위해 다시 **package.json** 파일을 열어서 scripts 항목에 명령어 추가 및 build 항목을 추가하자.

```json
"scripts": {
  ...,  // 기존 스크립트들
  "electron-pack": "yarn build && electron-builder"
},
...,
"build": {
  "productName": "my-electron-app",
  "asar": true,
  "appId": "com.app.electron",
  "extends": null,
  "files": [
    "*.js",
    "public",
    "build",
    "node_modules"
  ]
},
```
![img](\assets\built\images\electron\react_porting_by_electron_12.jpg)

파일 저장 후 아래 명령어를 입력하여 빌드를 진행해보자.

```terminal
yarn electron-pack
```
![img](\assets\built\images\electron\react_porting_by_electron_13.jpg)

빌드가 정상적으로 진행됬다면 아래와 같은 문구가 나온다.

![img](\assets\built\images\electron\react_porting_by_electron_14.jpg)

이제 프로젝트 경로에 진입해서 **dist** 폴더를 열어보면 아래와 같이 exe 파일이 빌드되어 있는 모습을 볼 수 있다.
해당 exe 파일을 실행해주면 react 앱을 데스크톱에 설치해준다.

![img](\assets\built\images\electron\react_porting_by_electron_15.jpg)

설치가 완료되면 데스크톱 응용 프로그램으로 실행되는 모습을 확인할 수 있다.
기존 3000포트에 react 앱을 구동시켜놓았다면 react 앱을 종료한 뒤 프로그램을 실행해보자.
react 앱이 돌아가고 있지 않아도 프로그램이 정상 작동하는 모습을 확인할 수 있다.

![img](\assets\built\images\electron\react_porting_by_electron_16.jpg)

응용 프로그램을 설치한 것이기 때문에 **프로그램 추가/제거**에서도 해당 프로그램을 확인할 수 있다.

![img](\assets\built\images\electron\react_porting_by_electron_17.jpg)


## 진행 중 마주친 에러

```terminal
Application entry file "public\Main.js" in the "C:\develop\Nodejs workplace\my-electron-app\dist\win-unpacked\resources\app.asar" does not exist. Seems like a wrong configuration.
```
![img](\assets\built\images\electron\react_porting_by_electron_18.jpg)

포스트 작성 진행 중 위와 같은 에러를 마주친 적이 있는데 처음 진행시에는 **package.json** 파일 안의 **build** 항목에
**"extends": null** 속성이 없는 상태로 진행을 하였다. 그랬더니 위와 같은 에러가 발생했다.

**package.json** 파일 안에 아래와 같이 추가해주자.

```json
"build": {
  ...,
  "extends": null,
},
```


## 마치며

급하게 알아보느라 다른 분들의 포스트를 기반으로 진행하여 build 항목 안의 속성들이 무슨 의미를 가지고 있는지 등등을
알아보지 않은 채로 일단 포스트를 작성하게 되었다. 추후 시간이 남을 때 더 자세히 알아봐야겠다.


## 참조 사이트
[https://blog.codefactory.ai/electron/create-desktop-app-with-react-and-electron/1-project-setting/](https://blog.codefactory.ai/electron/create-desktop-app-with-react-and-electron/1-project-setting/)

[https://velog.io/@threejoon/React-Electron-%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0](https://velog.io/@threejoon/React-Electron-%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0)