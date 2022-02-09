---
title: Jest 환경 세팅하기 (feat. TypeScript, ESModule)
date: 2022-02-09
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

ESModule 방식과 타입스크립트로 테스트 코드를 작성할 수 있는 Jest 환경을 세팅하는 방법을 정리합니다.

<details>
<summary><strong>ESModule</strong></summary>

ES6에 도입된 모듈 시스템을 말하며 <code class="language-plaintext highlighter-rouge">import</code>, <code class="language-plaintext highlighter-rouge">export</code>를 사용합니다.

</details>

## ⚙️ 세팅

### 프로젝트 초기화

```bash
npm init -y
```

### Jest, TypeScript, ts-jest 설치

```bash
npm i -D jest typescript ts-jest @types/jest
```

### 설정 파일 생성

```bash
npx ts-jest config:init
```

## 📜 설정 파일

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

<!-- prettier-ignore-start -->
```json
{
  "scripts": {
    "test": "jest"
  }
}
```
{: file="package.json" }
<!-- prettier-ignore-end -->

## 📚 See also

- [Jest - Using TypeScript via ts-jest](https://jestjs.io/docs/getting-started#using-typescript-via-ts-jest)
- [Github - kulshekhar/ts-jest](https://github.com/kulshekhar/ts-jest)
- [Jest - Configuring Jest: testEnvironment](https://jestjs.io/docs/configuration#testenvironment-string)
