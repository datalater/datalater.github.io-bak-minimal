---
title: 비동기 우아하게 처리하기 (feat. react)
date: 2022-02-06
categories: [TIL]
tags: [리액트] # TAG names should always be lowercase
image:
  src: https://noticon-static.tammolo.com/dgggcrkxq/image/upload/v1566557331/noticon/d5hqar2idkoefh6fjtpu.png
  width: 100 # in pixels
  height: 100 # in pixels
  alt: React
mermaid: true
published: false
---

## 목표

리액트에서 비동기 코드를 단계별로 나누어서 우아하게 처리합니다.

## 단계 1

> 비동기와 관련된 처리를 컴포넌트 내부에서 한 번에 처리하는 경우

```tsx
const LoginPage = () => {
  // ...

  const handleLogin = async (loginFields: LoginFields) => {
    try {
      const newUserInfo = await postLogin(loginFields);
      setCookie("userInfo", newUserInfo.token);

      if (newUserInfo.token) {
        const { fullName, username } = newUserInfo;
        const newMeta = getInitializedMeta(newUserInfo.meta);
        await updateUser({ fullName, username, meta: newMeta });
        fillUserInfo({ ...newUserInfo, meta: newMeta });
      }
    } catch (error) {
      if (axios.isAxiosError(error)) {
        // TODO: alert를 토스트로 교체한다
        // eslint-disable-next-line no-alert
        alert(error?.response?.data);
      }
    }
  };

  // ...

  return <LoginForm onSubmit={handleLogin} />;
};

export default LoginPage;
```

로그인 페이지(`LoginPage`) 컴포넌트의 핵심 역할은 로그인 폼을 렌더링하는 것입니다.. 그러나 위 코드에서는 로그인을 실제로 어떻게 진행하고, 실패하면 에러 처리를 어떻게 해야 하는지 모두 다루고 있습니다. 로그인 페이지 컴포넌트의 핵심 역할과 거리가 먼 코드가 상세하게 작성되어 있어서 로그인 페이지 컴포넌트의 핵심역할을 흐리게 만듭니다.

## 단계 2

> 에러 처리를 분리한 경우

`단계 1`에서 하던 에러 처리는 axios 에러에 대한 처리이므로 에러 처리 코드를 axios 코드 단으로 분리합니다.

```ts
import axios from "axios";

const { API_ENDPOINT } = process.env;

const request = axios.create({ baseURL: API_ENDPOINT });

// ...

request.interceptors.response.use(
  (res) => res,
  (error) => {
    // eslint-disable-next-line no-alert
    alert(error?.response?.data);
  }
);

export default request;
```

```tsx
const LoginPage = () => {
  // ...

  const handleLogin = async (loginFields: LoginFields) => {
    const newUserInfo = await postLogin(loginFields);
    setCookie("userInfo", newUserInfo.token);

    if (newUserInfo.token) {
      const { fullName, username } = newUserInfo;
      const newMeta = getInitializedMeta(newUserInfo.meta);
      await updateUser({ fullName, username, meta: newMeta });
      fillUserInfo({ ...newUserInfo, meta: newMeta });
    }
  };

  // ...

  return <LoginForm onSubmit={handleLogin} />;
};
```

`handleLogin` 함수에는 성공하는 경우만 적게 되어 읽기도 쉽고 함수의 책임도 더 명확히 드러나게 되었습니다.

## See also

- [토스ㅣSLASH 21 - 프론트엔드 웹 서비스에서 우아하게 비동기 처리하기](https://youtu.be/FvRtoViujGg)
