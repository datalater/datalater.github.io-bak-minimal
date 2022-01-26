---
title: emotion theme 타입 지정하는 방법 (feat. 자동완성)
date: 2021-12-04
categories: [TIL]
tags: [타입스크립트, emotion] # TAG names should always be lowercase
image:
  src: https://noticon-static.tammolo.com/dgggcrkxq/image/upload/v1566913457/noticon/eh4d0dnic4n1neth3fui.png
  width: 100 # in pixels
  height: 100 # in pixels
  alt: TypeScript
---

## 문제 상황

프로젝트에 일관적인 디자인을 적용하기 위해 공통으로 사용할 색상 값을 `assets/theme` 파일에 모아두고 사용하려고 합니다.

<!-- prettier-ignore-start -->
```ts
const theme = {
  colors: {
    primary: "#ffb266",
    fontColor: "#4d5256",
    disabled: "#c0c0c0",
    black: "#000",
    white: "#fff",
  },
};

export default theme;
```
{: file="assets/theme.ts" }
<!-- prettier-ignore-end -->

이때 새 컴포넌트에서 `theme`에 있는 값을 사용하려면 그때마다 `theme`을 `import` 해야 합니다.

```ts
// Component A
import theme from "./assets/theme";

// Component B
import theme from "./assets/theme";

// ...
```

하지만 emotion의 [`ThemeProvider`](https://emotion.sh/docs/theming)를 이용하면 `theme` 파일을 `import` 하지 않고도 스타일 값을 사용할 수 있습니다.

<!-- prettier-ignore-start -->

```tsx
import { ThemeProvider } from "@emotion/react";
import styled from "@emotion/styled";
import theme from "./assets/theme";

// 2. 스타일드 컴포넌트에서 props.theme으로 theme 접근 가능
const Wrapper = styled.div`
  background-color: ${(props) => props.theme.colors.primary};
`;

const App = () => {
  return (
    <ThemeProvider theme={theme}> {/* 1. ThemeProvider 설정 */}
      <Wrapper>
        <h1>Hello, Theme</h1>
      </Wrapper>
    </ThemeProvider>
  );
};

export default App;
```
{: file="App.tsx" }
<!-- prettier-ignore-end -->

그런데 타입 오류가 발생합니다.

![error](https://user-images.githubusercontent.com/8105528/144692954-51399743-dc54-4f3f-b79f-5032cf451a3f.png){: .shadow }
_theme에서 타입 오류가 발생한다_

`emotion`이 제공하는 `ThemeProvider` 컴포넌트의 `theme` prop은 패키지 내부적으로 타입이 정의되어 있으니 살펴보겠습니다.

<!-- prettier-ignore-start -->

> [emotion-js/emotion](https://github.com/emotion-js/emotion/blob/main/packages/react/types/theming.d.ts#L9)

```ts
export interface ThemeProviderProps {
  theme: Partial<Theme> | ((outerTheme: Theme) => Theme);
  children?: React.ReactNode;
}

export interface Theme {}
```
{: file="@emotion/react/types/theming.d.ts" }
<!-- prettier-ignore-end -->

`Theme` 인터페이스 내부가 비어 있네요. 즉, `props.theme`은 프로퍼티가 비어 있는 타입인데, (제가 임의로 정의한) `colors` 프로퍼티로 접근하려고 하니 타입 오류가 발생한 것입니다.

## 해결 방법

`emotion` 패키지에 정의된 인터페이스 `Theme`에 제가 사용할 `colors` 프로퍼티를 추가하면 됩니다. [Emotion 공식문서 - Define a Theme](https://emotion.sh/docs/typescript#define-a-theme)을 보면 자세히 나와 있습니다.

먼저 `assets/theme`에서 `theme` 객체의 인터페이스 `ITheme`을 정의하고 내보냅니다.

<!-- prettier-ignore-start -->
```ts
const theme = {
  colors: {
    primary: "#ffb266",
    fontColor: "#4d5256",
    disabled: "#c0c0c0",
    black: "#000",
    white: "#fff",
  },
} as const;

export default theme;

export type ITheme = typeof theme;
```
{: file="assets/theme.ts" }
<!-- prettier-ignore-end -->

그 다음 `types` 디렉토리에 `emotion.d.ts` 파일을 만들고 `@emotion/react` 모듈에 존재하는 `Theme` 인터페이스가 아까 내보냈던 `ITheme`을 상속 받게 합니다.

<!-- prettier-ignore-start -->
```ts
import "@emotion/react";

import { ITheme } from "../assets/theme";

declare module "@emotion/react" {
  export interface Theme extends ITheme {}
}
```
{: file="types/emotion.d.ts" }
<!-- prettier-ignore-end -->

참고로 객체 `theme`에 `as const`로 타입 단언(const assertion)을 해주면, 해당 객체 내부 값의 타입을 리터럴 타입으로 추론합니다. 타입 단언을 하든 안 하든 IDE에서 `props.colors`에 어떤 프로퍼티가 있는지 자동완성을 잘 해줍니다. 다만 타입 단언을 하지 않으면 객체 내부의 값은 얼마든지 바뀔 수 있기 때문에 `string`으로 추론하여 프로퍼티 이름만 자동완성을 지원해주지만, 타입 단언을 할 경우 리터럴 타입으로 변경되어 프로퍼티의 값(색상 값)까지 자동완성을 해줍니다.

## 결과

자동완성이 가능하고,

![solved](https://user-images.githubusercontent.com/8105528/144735578-f9fe3c47-4ebb-457d-8f14-5ef277a5c206.png){: .shadow }
_자동완성을 지원하며 프로퍼티 이름 뿐만 아니라 프로퍼티 값까지 알려준다_

더 이상 타입 에러가 발생하지 않습니다.

![solved](https://user-images.githubusercontent.com/8105528/144692997-3bc3b850-3868-41d4-82f7-3842c08a99c9.png){: .shadow }
_타입 오류가 발생하지 않는다_

## See also

- [emotion - Define a theme with TypeScript](https://emotion.sh/docs/typescript#define-a-theme)
- [toy-crane - Typescript에서 효과적으로 상수 관리하기](https://blog.toycrane.xyz/typescript%EC%97%90%EC%84%9C-%ED%9A%A8%EA%B3%BC%EC%A0%81%EC%9C%BC%EB%A1%9C-%EC%83%81%EC%88%98-%EA%B4%80%EB%A6%AC%ED%95%98%EA%B8%B0-e926db079f9)
