---
title: Jest 환경 세팅하기 (feat. TypeScript, ESModule)
date: 2022-02-09
categories: [TIL]
tags: [jest, 환경설정] # TAG names should always be lowercase
image:
  src: https://noticon-static.tammolo.com/dgggcrkxq/image/upload/v1566919905/noticon/c5qc9a9sytsnqmi4ixjz.png
  width: 100 # in pixels
  height: 100 # in pixels
  alt: Jest
mermaid: true
published: true
excerpt_separator: <!--end-of-description-->
---

## 💁 설명

ESModule 방식과 타입스크립트로 테스트 코드를 작성할 수 있는 Jest 환경을 세팅하는 방법을 정리합니다.

타입스크립트로만 작성된 프로젝트일 경우 `ts-jest` 방식으로 설정하는 것이 빠르고, JS와 TS가 모두 호환되는 환경이 필요하다면 `babel-jest`를 이용한 방식을 추천합니다.

참고로 저는 JS로 알고리즘 문제를 풀 때 `babel-jest`로 구성한 테스트 환경을 사용하고 있습니다.

<!--end-of-description-->

<details>
<summary><strong>ESModule</strong></summary>

ES6에 도입된 모듈 시스템을 말하며 <code class="language-plaintext highlighter-rouge">import</code>, <code class="language-plaintext highlighter-rouge">export</code>를 사용합니다.

</details>

## ⚙️ 세팅

### 프로젝트 초기화

```bash
npm init -y
```

## 방법 1: babel-jest

### Jest, babel-jest, TypeScript 설치

```bash
npm i -D jest babel-jest @babel/core @babel/preset-env @babel/preset-typescript typescript @types/jest
```

### Describe-Context-It 플러그인 설치

`describe`와 `it` 사이에 `context` 키워드를 사용하면, `Given-When-Then`과 같이 BDD 스타일로 테스트 코드를 작성할 수 있다.

```bash
npm i -D jest-plugin-context @types/jest-plugin-context
```

### 📜 설정 파일

#### babel.config.js

<!-- prettier-ignore-start -->
```js
module.exports = {
  presets: [
    ["@babel/preset-env", { targets: { node: "current" } }],
    "@babel/preset-typescript",
  ],
};
```
{: file="babel.config.js" }
<!-- prettier-ignore-end -->

#### jest.config.js

<!-- prettier-ignore-start -->
```js
module.exports = {
  clearMocks: true,
  testEnvironment: "jsdom",
  setupFilesAfterEnv: ["jest-plugin-context/setup"],
  testPathIgnorePatterns: ["<rootDir>/node_modules/"],
};

```
{: file="jest.config.js" }
<!-- prettier-ignore-end -->

#### tsconfig.json

<!-- prettier-ignore-start -->
```json
{
  "compilerOptions": {
    "target": "es6",
    "jsx": "react-jsx",
    "module": "esnext",
    "moduleResolution": "node",
    "allowJs": true,
    "checkJs": false,
    "outDir": "dist",
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "strict": true,
    "skipLibCheck": true,
  },
  "exclude": [
    "node_modules",
  ]
}
```
{: file="tsconfig.json" }
<!-- prettier-ignore-end -->

#### package.json

<!-- prettier-ignore-start -->
```json
{
  "scripts": {
    "test": "jest --verbose",
    "test:changed": "jest --onlyChanged --verbose ",
    "test:watch": "jest --watch --verbose",
    "test:changed:watch": "jest --onlyChanged --watch --verbose ",
    "test:cw": "npm run test:changed:watch"
  }
}
```
{: file="package.json" }
<!-- prettier-ignore-end -->

## 방법 2: ts-jest

### Jest, TypeScript, ts-jest 설치

```bash
npm i -D jest typescript ts-jest @types/jest
```

### 설정 파일 생성

```bash
npx ts-jest config:init
```

### 📜 설정 파일

#### jest.config.js

<!-- prettier-ignore-start -->
```js
/** @type {import('ts-jest/dist/types').InitialOptionsTsJest} */
module.exports = {
  preset: "ts-jest",
  testEnvironment: "jsdom", // import 키워드를 사용하는 ESModule을 사용하기 위해 jsdom으로 설정한다.
};
```
{: file="jest.config.js" }
<!-- prettier-ignore-end -->

#### package.json

<!-- prettier-ignore-start -->
```json
{
  "scripts": {
    "test": "jest --verbose",
    "test:changed": "jest --onlyChanged --verbose ",
    "test:watch": "jest --watch --verbose",
    "test:changed:watch": "jest --onlyChanged --watch --verbose ",
    "test:cw": "npm run test:changed:watch"
  }
}
```
{: file="package.json" }
<!-- prettier-ignore-end -->

## 📚 See also

- [Jest - Using TypeScript via ts-jest](https://jestjs.io/docs/getting-started#using-typescript-via-ts-jest)
- [Github - kulshekhar/ts-jest](https://github.com/kulshekhar/ts-jest)
- [Jest - Configuring Jest: testEnvironment](https://jestjs.io/docs/configuration#testenvironment-string)
