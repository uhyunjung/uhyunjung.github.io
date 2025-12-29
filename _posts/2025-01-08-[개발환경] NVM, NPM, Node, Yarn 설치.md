---
title: '[개발환경] NVM, NPM, Node, Yarn 설치'
date: 2025-01-08 15:10:00 +0900
categories: [정리, 개발환경]
tags: [개발환경]
math: true
mermaid: true
---

## NVM 설치(NPM 자동 설치)
<https://github.com/coreybutler/nvm-windows/releases>

## Node 설치 및 삭제
```bash
nvm install v20.18.0
nvm use 20.18.0
nvm on

nvm uninstall v20.18.0
```

## PowerShell에서 실행 안 되는 경우
```bash
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
```

## 버전 확인 및 캐시 삭제
```bash
nvm -version
nvm ls
npm -version
npm install -g npm@11.2.0
node -v

npm cache clean --force
yarn cache clean --force
```

## SSL 인증서 검증 해제
```bash
npm config set strict-ssl false
yarn config set "strict-ssl" false

set NODE_TLS_REJECT_UNAUTHORIZED=0
set NODE_EXTRA_CA_CERTS=pem 키 경로
```

## Vite 설치
```bash
npm install vite --save-dev
npm install @vitejs/plugin-vue
```

## Vue 설치
```bash
npm i vue
```

## Yarn 설치
```bash
npm install -g yarn
npm list -g
```
```bash
# PowerShell
Set-ExecutionPolicy RemoteSigned
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
Get-ExecutionPolicy -List
```
- 보안에 따라 CMD에서는 가능하나, PowerShell에서는 안 될 수도 있다.
- PowerShell 실행 원하는 경우, PowerShell 정책 변경 시도(안 될 수 있음)

## Yarn 프로젝트 실행 시
```bash
yarn install

yarn add node-sass
yarn remove node-sass
```

## Axios 설치
```bash
npm install axios
```

## 서버 실행
```bash
npm start
set PORT=3000 && npm start # proxy port 8080 8081
```

## 코드 포맷팅
- EsLint, Prettier Plugin 설치
- Prettier 규칙 우선
```bash
npm i -D eslint-plugin-prettier
```
- Eslint
```bash
npm install eslint --save-dev
npm init @eslint/config

npm uninstall -g eslint
```

## 참고
- npm.psi 삭제
- 시스템 환경 변수 편집 후 VSCode 재실행

```bash
JAVA_HOME C:\Program Files\Eclipse Adoptium\jdk-21.0.4.7-hotspot\bin
mvn C:\Program Files\Maven\apache-maven-3.9.9-bin\bin
NVM_HOME C:\Users\AppData\Local\nvm\20.18.1
NVM_SYMLINK C:\Program Files\nodejs
```

- 버전 변경 필요

```bash
gyp ERR! configure error
gyp ERR! stack FetchError: request to https://nodejs.org/download/release/v20.18.1/node-v20.18.1-headers.tar.gz failed, reason: self-signed certificate in certificate chain

"node-sass": "^7.0.1"
gyp ERR! node -v v22.13.0
gyp ERR! node-gyp -v v8.4.1
gyp ERR! not ok
```

- Window 환경

```bash
npm install --global --production windows-build-toolsyarn
window.location.origin
```
