---
title: 리액트 보일러플레이트 만들기 (feat. Webpack, TypeScript, ESLint, Prettier, Storybook)
date: 2022-01-24
categories: [TIL]
tags: [리액트] # TAG names should always be lowercase
image:
  src: https://noticon-static.tammolo.com/dgggcrkxq/image/upload/v1566557331/noticon/d5hqar2idkoefh6fjtpu.png
  width: 100 # in pixels
  height: 100 # in pixels
  alt: React
mermaid: true
---

## 목표

Webpack을 이용해서 React + Typescript + ESLint + Prettier + Storybook 세팅하는 방법을 정리합니다.

완성된 코드:

- [datalater/react-boilerplate](https://github.com/datalater/react-boilerplate)

## React 및 TypeScript 설정

### 디렉토리 생성 후 진입

```bash
md react-boilerplate
cd react-boilerplate
```

### 프로젝트 초기화

```bash
yarn init
```

### 리액트 설치

```bash
yarn add react react-dom
```

### 타입스크립트 설치

```bash
yarn add -D typescript @types/react @types/react-dom
```

### tsconfig 작성

```bash
npx tsc --init
```

`tsconfig.json` 파일 수정

```json
{
  "compilerOptions": {
    "target": "es5",
    "jsx": "react-jsx",
    "module": "esnext",
    "moduleResolution": "node",
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "strict": true,
    "skipLibCheck": true
  },
  "include": ["src"]
}
```

관련 옵션 설명:

- [react-jsx](https://www.typescriptlang.org/ko/tsconfig#jsx): 리액트 17에서 도입된 새로운 JSX 변환
  - `import React` 없이 JSX를 사용할 수 있다

## Webpack 설치

```bash
yarn add -D webpack webpack-cli webpack-dev-server
```

webpack 플러그인 설치

```bash
yarn add -D html-webpack-plugin copy-webpack-plugin webpack-bundle-analyzer mini-css-extract-plugin
```

webpack 로더 설치

```bash
yarn add -D babel-loader ts-loader style-loader css-loader postcss-loader sass-loader sass
```

## Webpack config 작성

웹팩 설정을 개발 환경 모드와 운영 환경 모드로 분리해서 작성하고, 두 환경에서 공통적으로 쓰이는 코드는 공통 설정 파일에 작성합니다.

> [Webpack - production](https://webpack.js.org/guides/production/)

웹팩 설정을 병합하는 패키지 설치

```bash
yarn add -D webpack-merge
```

CSS 후처리 패키지 설치

```bash
yarn add autoprefixer
```

`webpack.common.js`:

```js
const path = require("path");
const HtmlWebpackPlugin = require("html-webpack-plugin");
const CopyWebpackPlugin = require("copy-webpack-plugin");
const MiniCSSExtractPlugin = require("mini-css-extract-plugin");

const isProduction = process.env.NODE_ENV === "production";

module.exports = {
  resolve: {
    extensions: [".ts", ".tsx", ".js", ".jsx"],
    /* alias를 사용할 경우 주석을 해제하고 사용한다. */
    // alias: {
    //   "@components": path.resolve(__dirname, "src/components"),
    // },
  },
  entry: {
    app: "./src/index.tsx",
  },
  output: {
    filename: "[name].bundle.js",
    path: path.resolve(__dirname, "dist"),
    clean: true,
  },
  plugins: [
    new HtmlWebpackPlugin({
      filename: "./index.html",
      template: path.resolve(__dirname, "./index.html"),
      favicon: "./static/logo.png",
    }),
    new CopyWebpackPlugin({
      patterns: [{ from: "static" }],
    }),
    isProduction &&
      new MiniCSSExtractPlugin({
        filename: "[name].css",
      }),
  ].filter(Boolean),
  module: {
    rules: [
      // TSX
      {
        test: /\.tsx?$/i,
        exclude: /node_modules/,
        use: [
          // JS to older JS with polyfills
          {
            loader: "babel-loader",
          },
          // TS to JS
          {
            loader: "ts-loader",
          },
        ],
      },
      // SCSS
      {
        test: /\.s?css$/i,
        use: [
          isProduction ? MiniCSSExtractPlugin.loader : "style-loader",
          "css-loader",
          "postcss-loader",
          "sass-loader",
        ],
      },
      // ASSETS
      {
        test: /\.(png|jpe?g|gif|mp3)$/i,
        type: "asset/resource",
      },
    ],
  },
};
```

참조:

- [ts-loader와 babel-loader를 함께 쓰는 이유](https://stackoverflow.com/questions/49624202/why-use-babel-loader-with-ts-loader)
- [ts-loader와 babel-loader를 함께 쓰는 예제 코드를 언급한 문서](https://github.com/TypeStrong/ts-loader#babel)
- [Microsoft/TypeScript samples - ts-loader와 babel-loader를 함께 쓰는 예제 코드](https://github.com/microsoft/TypeScriptSamples/blob/master/react-flux-babel-karma/webpack.config.js)

`postcss.config.js`:

```js
const autoprefixer = require("autoprefixer");

module.exports = {
  plugins: [autoprefixer],
};
```

`webpack.dev.js`:

```js
const { merge } = require("webpack-merge");
const common = require("./webpack.common.js");

module.exports = merge(common, {
  mode: "development",
  devtool: "inline-source-map",
  devServer: {
    historyApiFallback: true,
    hot: true,
    compress: true,
    port: 8000,
  },
});
```

`webpack.prod.js`:

```js
const { merge } = require("webpack-merge");
const common = require("./webpack.common.js");

const { BundleAnalyzerPlugin } = require("webpack-bundle-analyzer");

module.exports = merge(common, {
  mode: "production",
  devServer: {
    static: "./dist",
    historyApiFallback: true,
    hot: true,
    compress: true,
    port: 9000,
  },
  plugins: [
    new BundleAnalyzerPlugin({
      analyzerMode: "static",
      openAnalyzer: false,
      generateStatsFile: true,
      statsFilename: "bundleStats.json",
    }),
  ],
});
```

> [sourcemap 설정 종류](https://webpack.js.org/configuration/devtool/)

`package.json`:

```json
"scripts": {
  "dev": "NODE_ENV=development webpack-dev-server --config webpack.dev.js",
  "start": "NODE_ENV=production webpack-dev-server --config webpack.prod.js",
  "build": "NODE_ENV=production webpack --config webpack.prod.js"
},
```

`jsconfig.json`:

```json
{
  "compilerOptions": {
    "checkJs": false,
    "resolveJsonModule": true,
    "moduleResolution": "node",
    "target": "es2020",
    "module": "es2015",
    "baseUrl": ".",
    "paths": {
      /* 경로 별칭을 사용하는 경우 아래 주석을 해제하여 적절히 사용한다 */
      // "@components/*": ["./src/components/*"]
    }
  },
  "exclude": [
    "dist",
    "node_modules",
    "build",
    ".vscode",
    ".nuxt",
    "coverage",
    "jspm_packages",
    "tmp",
    "temp",
    "bower_components",
    ".npm",
    ".yarn"
  ],
  "typeAcquisition": {
    "enable": true
  }
}
```

## ESLint 및 Prettier 설정

> [React TypeScript Cheatsheet](https://react-typescript-cheatsheet.netlify.app/docs/basic/linting/)

ESLint 및 Prettier 설치

```bash
yarn add -D @typescript-eslint/eslint-plugin @typescript-eslint/parser eslint

yarn add -D prettier
```

ESLint 플러그인 설치:

```bash
yarn add -D eslint-config-prettier eslint-plugin-import eslint-plugin-jsx-a11y eslint-plugin-react eslint-plugin-react-hooks eslint-config-airbnb-typescript @typescript-eslint/eslint-plugin @typescript-eslint/parser
```

`.eslintrc.js`:

```js
module.exports = {
  env: {
    browser: true,
    es2021: true,
    node: true,
    jest: true,
  },
  ignorePatterns: ["*.js"],
  extends: [
    "eslint:recommended",
    "plugin:react/recommended",
    "plugin:react-hooks/recommended",
    "plugin:import/recommended",
    "plugin:jsx-a11y/recommended",
    "plugin:@typescript-eslint/recommended",
    "plugin:@typescript-eslint/recommended-requiring-type-checking",
    "airbnb-typescript",
    "airbnb/hooks",
    "prettier",
  ],
  parser: "@typescript-eslint/parser",
  parserOptions: {
    ecmaFeatures: {
      jsx: true,
    },
    ecmaVersion: 12,
    sourceType: "module",
    project: "./tsconfig.json",
  },
  plugins: ["@typescript-eslint"],
  rules: {
    indent: ["error", 2],
    "no-empty": "warn",
    "no-console": "warn",
    "no-unused-vars": "off",
    "linebreak-style": ["error", "unix"],
    "react/jsx-uses-react": "off",
    "react/react-in-jsx-scope": "off",
    "@typescript-eslint/no-unused-vars": [
      "error",
      { vars: "all", args: "after-used", ignoreRestSiblings: false },
    ],
    "@typescript-eslint/explicit-function-return-type": "warn", // Consider using explicit annotations for object literals and function return types even when they can be inferred.
    "@typescript-eslint/naming-convention": [
      "error",
      // Allow camelCase variables (23.2), PascalCase variables (23.8), and UPPER_CASE variables (23.10)
      {
        selector: "variable",
        format: ["camelCase", "PascalCase", "UPPER_CASE"],
      },
      // Allow camelCase functions (23.2), and PascalCase functions (23.8)
      {
        selector: "function",
        format: ["camelCase", "PascalCase"],
      },
      // Airbnb recommends PascalCase for classes (23.3), and although Airbnb does not make TypeScript recommendations, we are assuming this rule would similarly apply to anything "type like", including interfaces, type aliases, and enums
      {
        selector: "typeLike",
        format: ["PascalCase"],
      },
    ],
  },
};
```

참조:

- [React TypeScript Cheatsheet](https://react-typescript-cheatsheet.netlify.app/docs/basic/linting/)
- [Webpack으로 React + TypeScript + Styled Component + Storybook 세팅하기](https://yujo11.github.io/React/React-TS-Webpack-%EC%84%B8%ED%8C%85/)

## 바벨 설치

```bash
yarn add -D @babel/core @babel/preset-env @babel/preset-react @babel/plugin-transform-runtime
```

`.babelrc`:

```json
{
  "presets": ["@babel/preset-env", "@babel/preset-react"],
  "plugins": [
    [
      "@babel/plugin-transform-runtime",
      {
        "corejs": 3
      }
    ]
  ]
}
```

## Emotion 설치

```bash
yarn add @emoition/react @emotion/styled @emotion/babel-plugin
```

`.babelrc`:

개발 환경에서만 [Emotion 클래스 네임을 컴포넌트 경로로 자동 레이블링](https://emotion.sh/docs/@emotion/babel-plugin#usage) 하도록 세팅한다.

```json
{
  "presets": ["@babel/preset-env", "@babel/preset-react"],
  "plugins": [
    [
      "@babel/plugin-transform-runtime",
      {
        "corejs": 3
      }
    ],
    [
      "@emotion",
      {
        // sourceMap is on by default but source maps are dead code eliminated in production
        "sourceMap": true,
        "autoLabel": "dev-only",
        "labelFormat": "[dirname]-[filename]-[local]",
        "cssPropOptimization": true
      }
    ]
  ]
}
```

## 파일 작성하기

`index.html`:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>React Boilerplate</title>
  </head>
  <body>
    <div id="app"></div>
  </body>
</html>
```

`src/index.tsx`:

```tsx
import ReactDOM from "react-dom";
import App from "./App";

const container = document.getElementById("app");

ReactDOM.render(<App />, container);
```

`src/App.tsx`:

```tsx
import Button from "./components/Button/Button";

const App = () => {
  return <Button>버튼이닷!</Button>;
};

export default App;
```

`src/components/Button/Button.tsx`:

> [cobaltinc/caple-design-system - Button](https://github.com/cobaltinc/caple-design-system/tree/master/packages/react/src/components/Button)

```tsx
import React from "react";

export type ButtonType = "basic" | "core" | "special" | "danger" | "warning";
export type ButtonHtmlType = "button" | "submit" | "reset";
export type ButtonSizeType = "tiny" | "small" | "normal" | "large" | "xlarge";

export interface ButtonProps {
  children?: React.ReactNode;
  htmlType?: ButtonHtmlType;
  type?: ButtonType;
  size?: ButtonSizeType;
  block?: boolean;
  ghost?: boolean;
  disabled?: boolean;
  loading?: boolean;
  onClick?: React.MouseEventHandler<HTMLButtonElement>;
  className?: string;
  style?: React.CSSProperties;
}

export default React.forwardRef<HTMLButtonElement, ButtonProps>(
  (
    {
      children,
      htmlType = "button",
      type = "basic",
      size = "normal",
      block,
      ghost,
      disabled,
      loading,
      onClick,
      className = "",
      style,
      ...props
    },
    ref
  ) => {
    const handleClick = (e: React.MouseEvent<HTMLButtonElement>) => {
      if (disabled || loading) {
        e.preventDefault();
        return;
      }

      onClick?.(e);
    };

    return (
      <button
        ref={ref}
        type={htmlType}
        style={style}
        disabled={disabled}
        onClick={handleClick}
        {...props}
      >
        버튼
      </button>
    );
  }
);
```

`src/components/Button/Button.stories.tsx`:

```tsx
import Button from "./Button";

export default {
  title: "Component/Button",
  component: Button,
};

export const Default = () => {
  const task = {
    content: "출근하기",
    complete: false,
  };
  return <Button>버튼이닷</Button>;
};
```

`src/components/Button/IconButton.tsx`:

```tsx
import { TransparentButton } from "./Button.styles";

interface IProps {
  children?: React.ReactNode;
}

const IconButton = ({ children }: IProps) => {
  return <TransparentButton>{children}</TransparentButton>;
};

export default IconButton;
```

`src/components/Button/Button.Styles.ts`:

```ts
import styled from "@emotion/styled";

export const TransparentButton = styled.button`
  background-color: transparent;
  border: none;
  outline: none;
  box-shadow: 1px 1px 1px 1px rgba(0, 0, 0, 0.2);
`;
```

`src/components/Button/index.ts`:

```ts
export { default as Button } from "./Button";
export { default as IconButton } from "./IconButton";
```

## Storybook 설정

```bash
npx sb init
```

`.storybook/main.js`:

스토리북에서 Emotion 패키지 경로와 경로 별칭을 이해하도록 설정한다.

```js
const path = require("path");

const resolvePath = (_path) => path.join(process.cwd(), _path);

module.exports = {
  stories: ["../src/**/*.stories.mdx", "../src/**/*.stories.@(js|jsx|ts|tsx)"],
  addons: ["@storybook/addon-links", "@storybook/addon-essentials"],
  webpackFinal: async (config) => ({
    ...config,
    resolve: {
      ...config.resolve,
      alias: {
        ...config.resolve.alias,
        "@emotion/styled": resolvePath("node_modules/@emotion/styled"),
        /* 경로 별칭을 사용하는 경우 아래 주석을 해제하여 적절하게 사용한다 */
        // "@components": resolvePath("src/components"),
      },
    },
  }),
};
```

## See also

- TSConfig
  - [react-jsx](https://www.typescriptlang.org/ko/tsconfig#jsx): 리액트 17에서 도입된 새로운 JSX 변환
  - [React TypeScript Cheatsheet - Troubleshooting Handbook: tsconfig.json](https://react-typescript-cheatsheet.netlify.app/docs/basic/troubleshooting/tsconfig)
- ESLint
  - [TypeScript ESLint](https://typescript-eslint.io/docs/linting/)
  - [React TypeScript Cheatsheet](https://react-typescript-cheatsheet.netlify.app/docs/basic/linting/)
- [cobaltinc/caple-design-system - Button](https://github.com/cobaltinc/caple-design-system/tree/master/packages/react/src/components/Button)
- Emotion
  - [Github PR - 이모션 클래스 네임을 컴포넌트 경로로 자동 레이블링 하도록 세팅](https://github.com/prgrms-web-devcourse/Team_BbungCles_Devnity_FE/pull/164)
