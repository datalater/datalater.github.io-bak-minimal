---
title: 리액트 앱을 위한 Jest 환경 세팅하기 (feat. TypeScript, React Testing Library)
date: 2022-02-10
categories: [TIL]
tags: [jest] # TAG names should always be lowercase
image:
  src: https://noticon-static.tammolo.com/dgggcrkxq/image/upload/v1566919905/noticon/c5qc9a9sytsnqmi4ixjz.png
  width: 100 # in pixels
  height: 100 # in pixels
  alt: Jest
mermaid: true
published: true
---

## 💁 설명

타입스크립트로 리액트 앱을 개발할 때 테스트 코드를 작성할 수 있도록 Jest 환경을 세팅하는 방법을 정리합니다.

<details>
<summary><strong>참고: <code class="language-plaintext highlighter-rouge">babel-jest</code>를 사용하는 이유</strong></summary>

타입스크립트가 호환되는 Jest 환경을 구축할 때 <code class="language-plaintext highlighter-rouge">ts-jest</code> 를 사용할 수도 있고 <code class="language-plaintext highlighter-rouge">babe-jest</code>를 사용할 수도 있다. 그런데 JS로 구성되었다가 TS로 마이그레이션 하는 프로젝트에서 <code class="language-plaintext highlighter-rouge">ts-jest</code>를 사용할 경우 JS로 작성된 파일의 테스트가 제대로 동작하지 않는 문제가 발견되어 JS와 TS 파일 모두 잘 동작하는 <code class="language-plaintext highlighter-rouge">babel-jest</code>를 사용합니다.

</details>

## ⚙️ 세팅

### 프로젝트 초기화

```bash
npm init -y
```

### Jest, babel-jest, TypeScript 설치

```bash
npm i -D jest babel-jest @babel/core @babel/preset-env @babel/preset-typescript typescript @types/jest
```

### Describe-Context-It 플러그인 설치

`describe`와 `it` 사이에 `context` 키워드를 사용하면, `Given-When-Then`과 같이 BDD 스타일로 테스트 코드를 작성할 수 있다.

```bash
npm i -D jest-plugin-context @types/jest-plugin-context
```

### React Testing Library 설치

```bash
npm i -D @testing-library/react @testing-library/user-event @testing-library/dom @testing-library/jest-dom
```

### 설정 파일 생성

```bash
jest --init
```

> 위 명령어를 실행하는 대신 아래 `jest.config.js` 파일을 그대로 복사해서 사용해도 된다.

## 📜 설정 파일

### babel.config.js

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

### jest.config.js

<!-- prettier-ignore-start -->
```js
const { jestPathAlias } = require('./pathAlias');

module.exports = {
  testEnvironment: 'jsdom',
  setupFilesAfterEnv: [
    '@testing-library/jest-dom/extend-expect',
    'jest-plugin-context/setup',
  ],
  testPathIgnorePatterns: ['<rootDir>/node_modules/', '<rootDir>/dist/'],
  moduleNameMapper: jestPathAlias,
};
```
{: file="jest.config.js" }
<!-- prettier-ignore-end -->

경로 별칭을 사용할 경우 아래와 같이 개별 파일로 분리하여 관리할 수 있다:

> webpack의 alias나 tsconfig의 alias 모두 아래와 같이 한 파일에서 관리하면 편하다.

### pathAlias.js

<!-- prettier-ignore-start -->
```js
const path = require('path');

const resolvePath = (relativePath) => path.resolve(__dirname, relativePath);

module.exports = {
  jestPathAlias: {
    '^@components/(.*)$': '<rootDir>/src/components/$1',
    '^@hooks/(.*)$': '<rootDir>/src/hooks/$1',
    '^@pages/(.*)$': '<rootDir>/src/pages/$1',
    '^@contexts/(.*)$': '<rootDir>/src/contexts/$1',
    '^@apis/(.*)$': '<rootDir>/src/apis/$1',
    '^@assets/(.*)$': '<rootDir>/src/assets/$1',
    '^@utils/(.*)$': '<rootDir>/src/utils/$1',
    '^@constants/(.*)$': '<rootDir>/src/constants/$1',
  },
};
```
{: file="pathAlias.js" }
<!-- prettier-ignore-end -->

### package.json

<!-- prettier-ignore-start -->
```json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch"
  }
}
```
{: file="package.json" }
<!-- prettier-ignore-end -->

### tsconfig.json

<!-- prettier-ignore-start -->
```json
{
  "compilerOptions": {
    "target": "es5",
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
  "include": ["src"],
  "exclude": [
    "node_modules",
    "dist",
    "fixtures",
    "src/**/*.test.tsx",
    "src/**/*.test.ts"
  ]
}
```
{: file="tsconfig.json" }
<!-- prettier-ignore-end -->

### .eslintrc.js

<!-- prettier-ignore-start -->
```js
module.exports = {
  env: {
    browser: true,
    es2021: true,
    jest: true,
  },
  globals: {
    context: 'readonly',
  },
  // ...
}
```
{: file=".eslintrc.js" }
<!-- prettier-ignore-end -->

## 📚 See also

- [Jest - Using Babel](https://jestjs.io/docs/getting-started#using-babel)
- [React Testing Library - intro](https://testing-library.com/docs/react-testing-library/intro)
- [React Testing Library - user-event](https://testing-library.com/docs/ecosystem-user-event/)
- [Github - bbc/simorgh - jest.config.js](https://github.com/bbc/simorgh/blob/latest/jest.config.js)
