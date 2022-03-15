---
title: 타입스크립트 연습하기
date: 2022-02-09
categories: [TIL]
tags: [타입스크립트] # TAG names should always be lowercase
image:
  src: https://noticon-static.tammolo.com/dgggcrkxq/image/upload/v1566913457/noticon/eh4d0dnic4n1neth3fui.png
  width: 100 # in pixels
  height: 100 # in pixels
  alt: TypeScript
mermaid: true
published: false
excerpt_separator: <!--end-of-description-->
---

## status

```js
{
  status: "pending",
  message: "실제 애플리케이션 레벨에서 사용하는 예시가 추가되면 퍼블리시합니다."
}
```

## 목표

런타임에 가기 전에 컴파일 타임 때 에러를 잡아내도록 타입스크립트를 작성하는 방법을 연습합니다.

## tsconfig

```json
{
  "noImplicitAny": true,
  "strictNullChecks": true,
  "noImplicitReturns": true,
  "strictFunctionTypes": true,
  "strictPropertyInitialization": true
}
```

## 조건에 따라 달라지는 타입

```ts
interface StringContainer {
  value: string;
  format(): string;
  split(): string[];
}

interface NumberContainer {
  value: number;
  nearestPrime: number;
  round(): number;
}

// Q1. T에 따라 container가 StringContainer 또는 NumberContainer를 가지게 하려면?
type itemQ1<T> = {
  id: T;
  container: ???;
};

// A1.
type itemA1<T> = {
  id: T;
  container: T extends string ? StringContainer : NumberContainer;
};

// Q2. T가 string 또는 number만 가능하고 나머지(ex. boolean)는 불가능하도록 하려면?
// A2.
type itemA2<T> = {
  id: T extends string | number ? T : never;
  container: T extends string
    ? StringContainer
    : T extends number
    ? NumberContainer
    : never;
};
```

## Function인 프로퍼티 찾기

```ts
type FunctionPropertyNames<T> = {
  [K in keyof T]: T[K] extends Function ? K : never;
}[keyof T];
type FunctionProperties<T> = Pick<T, FunctionPropertyNames<T>>;

interface Person {
  id: number;
  name: string;
  hello(message: string): void;
}

type T1 = FunctionProperties<Person>;
```

## 서버 데이터에 대한 타입 만들기

```ts
type Result<T> =
  | { loading: true }
  | { data: T; loading: false }
  | { error: Error; loading: false };

declare function getResult(): Result<string>;

const res = getResult();

res.data; // error!
res.error; // error!
res.loading; // boolean

if ("data" in res) {
  res.error; // error!
  res.loading; // false
}
```

```ts
type Result<T> =
  | { type: "pending"; loading: true }
  | { type: "success"; data: T; loading: false }
  | { type: "fail"; error: Error; loading: false };

declare function getResult(): Result<string>;

const res = getResult();

if (res.type === "pending") {
  res; // { type: "pending"; loading: true }
}

if (res.type === "success") {
  res; // { type: "success"; data: string; loading: false }
}

if (res.type === "fail") {
  res; // { type: "fail"; error: Error; loading: false }
}
```

## See also

- [우아한 타입스크립트](https://youtu.be/ViS8DLd6o-E)
