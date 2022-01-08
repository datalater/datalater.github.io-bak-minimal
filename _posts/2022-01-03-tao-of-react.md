---
title: Tao of React
date: 2022-01-03
categories: [wiki]
tags: [react] # TAG names should always be lowercase
image:
  src: https://noticon-static.tammolo.com/dgggcrkxq/image/upload/v1566557331/noticon/d5hqar2idkoefh6fjtpu.png
  width: 100 # in pixels
  height: 100 # in pixels
  alt: React
published: false
---

## 목표

베스트 프렉티스를 정리한 참조 문서 [Tao of React](https://alexkondov.com/tao-of-react/)를 읽고 개인 위키 형태로 정리한다.

> **[위키란?](https://ko.wikipedia.org/wiki/%EC%9C%84%ED%82%A4)** 최초의 위키 소프트웨어인 위키위키웹(WikiWikiWeb)을 만든 워드 커닝엄은 위키를 "동작하는 가장 단순한 온라인 데이터베이스"라고 설명했다.

## [헬퍼 함수 정리하기](https://alexkondov.com/tao-of-react/#organize-helper-functions)

<details>
<summary><strong>(컴포넌트 내부 환경을 클로저로 갖고 있을 필요가 없다면) 헬퍼 함수를 컴포넌트 외부로 분리하자.</strong></summary>

왜냐하면

- 컴포넌트 내부에는 필요한 코드만 두는 게 읽기 편하다.
- 헬퍼 함수를 (컴포넌트에 종속되지 않는) 순수 함수로 작성하면 테스트하기 쉽고 버그 추적에 유리하며 확장하기 쉽다.

</details>

## [조건부 렌더링](https://alexkondov.com/tao-of-react/#conditional-rendering)

<details>
<summary><strong>논리곱으로 short circuit operator을 사용하면 UI에 <code>0</code>이 나올 수 있으므로 삼항 연산자를 사용하자.</strong></summary>

```jsx
// 👎 Try to avoid short-circuit operators
function Component() {
  const count = 0;

  return <div>{count && <h1>Messages: {count}</h1>}</div>;
}
```

논리곱의 좌항값 0은 falsy한 값이므로 false로 판단되지만, 출력할 때는 불리언 값인 false가 아니라 숫자 0으로 출력된다.

```jsx
// 👍 Use a ternary instead
function Component() {
  const count = 0;

  return <div>{count ? <h1>Messages: {count}</h1> : null}</div>;
}
```

</details>

## [리스트 마크업을 개별 컴포넌트로 분리하자](https://alexkondov.com/tao-of-react/#move-lists-in-components)

<details>
<summary><strong>리스트를 생성하는 <code>map</code>문법은 가독성을 좋지 않으므로 개별 컴포넌트로 분리하자.</strong></summary>

```tsx
<div>
  <strong>
    {mapgakcoDetail?.mapgakco?.applicantCount} /{" "}
    {mapgakcoDetail?.mapgakco?.applicantLimit} 명
  </strong>

  <ul>
    <li key={mapgakcoDetail?.author?.userId}>
      <div>
        <RiVipCrownFill color={theme.colors.crownGold} />
        <UserProfileImage
          title={`주최자: ${mapgakcoDetail?.author?.name}`}
          imageUrl={mapgakcoDetail?.author?.profileImgUrl}
          size={24}
        />
      </div>
    </li>
    {mapgakcoDetail?.applicants?.map(({ userId, name, profileImgUrl }) => (
      <li key={userId}>
        <UserProfileImage title={name} imageUrl={profileImgUrl} size={24} />
      </li>
    ))}
  </ul>
</div>
```

```tsx
<div>
  <strong>
    {mapgakcoDetail?.mapgakco?.applicantCount} /{" "}
    {mapgakcoDetail?.mapgakco?.applicantLimit} 명
  </strong>
  <ParticipantList
    author={mapgakcoDetail?.author}
    applicants={mapgakcoDetail?.applicants}
  />
</div>
```

</details>
