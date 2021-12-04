---
title: emotion theme 타입 지정하는 방법
date: 2021-12-04
categories: [TIL]
tags: [typescript] # TAG names should always be lowercase
image:
  src: https://noticon-static.tammolo.com/dgggcrkxq/image/upload/v1566913457/noticon/eh4d0dnic4n1neth3fui.png
  width: 100 # in pixels
  height: 100 # in pixels
  alt: TypeScript
---

## 문제 상황

프로젝트 내에서 공통으로 사용하는 스타일 값을 `theme` 모듈에 모아두고 사용하려고 한다. 이때 새 파일에서 `theme`에 있는 값을 사용하려면 `theme`을 `import` 해야 한다. 하지만  emotion의 `ThemeProvider`를 이용하면 `theme` 모듈 파일을 `import` 하지 않고도 해당 스타일 값을 사용할 수 있다.

```ts
export default {
  colors: {
    primary: "#ffb266",
    fontColor: "#4d5256",
    disabled: "#c0c0c0",
    black: "#000",
    white: "#fff",
  },
};
```
{: file="theme.ts" }

```tsx
import { ThemeProvider } from "@emotion/react";
import styled from "@emotion/styled";
import theme from "./assets/theme";

const Wrapper = styled.div`
  background-color: ${(props) => props.theme.colors.primary};
`;

const App = () => {
  return (
    <ThemeProvider theme={theme}>
      <Wrapper>
        <h1>Hello, Theme</h1>
      </Wrapper>
    </ThemeProvider>
  );
};

export default App;
```
{: file="App.tsx" }

그런데 `emotion`이 제공하는 `ThemeProvider`의 `theme` props에는 내가 정의한 property `colors`가 존재하지 않는다. 따라서 타입 오류가 발생한다:

![error](https://user-images.githubusercontent.com/8105528/144692954-51399743-dc54-4f3f-b79f-5032cf451a3f.png){: .shadow }
_theme에서 타입 오류가 발생한다_

## 해결 방법

`theme`의 인터페이스 `ITheme`을 정의하고, `@emotion/react` 모듈에 존재하는 `Theme` 인터페이스가 `ITheme`을 상속 받게 만든다.

```ts
interface Colors {
  [key: string]: string;
}

export interface ITheme {
  colors: Colors;
}
```
{: file="theme.ts" }

```ts
import "@emotion/react";

import { ITheme } from "../assets/theme";

declare module "@emotion/react" {
  export interface Theme extends ITheme {}
}
```
{: file="types/emotion.d.ts" }

더 이상 타입 에러가 발생하지 않는다:

![solved](https://user-images.githubusercontent.com/8105528/144692997-3bc3b850-3868-41d4-82f7-3842c08a99c9.png){: .shadow }
_타입 오류가 발생하지 않는다_

## See also

- [emotion - Define a theme with TypeScript](https://emotion.sh/docs/typescript#define-a-theme)
