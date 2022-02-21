---
title: 리액트 useEffect 최적화 하기 (feat. 타이머 컴포넌트)
date: 2022-02-22
categories: [TIL]
tags: [react] # TAG names should always be lowercase
image:
  src: https://noticon-static.tammolo.com/dgggcrkxq/image/upload/v1566557331/noticon/d5hqar2idkoefh6fjtpu.png
  width: 100 # in pixels
  height: 100 # in pixels
  alt: React
mermaid: true
published: true
---

## 💁 설명

리액트의 `useEffect` 훅을 사용할 때 의존성 배열에 추가한 종속성이 너무 자주 변경되면 그때마다 이펙트가 실행됩니다. 이때 최적화한 예시를 정리합니다.

## 기존 코드

`Timer` 컴포넌트는 사용자가 타이머 시작 버튼을 누르면 타이머 시간(`totalTime`)을 1초씩 더해서 보여주고 사용자가 타이머 일시정지 버튼을 누르면 타이머 시간을 중지시킵니다.

<!-- prettier-ignore-start -->
```tsx
const Timer = ({ ...props }: any) => {
  // 로컬스토리지에 등록된 타이머값 가져오기
  const [userTimer, setUserTimer] = useLocalStorage("userTimer", {});
  // 타이머를 시작했는지 여부
  const [timerOn, setTimerOn] = useState<boolean>(false);
  // 타이머값
  const [totalTime, setTotalTime] = useState<number>(
    userTimer.totalTime ? userTimer.totalTime : 0
  );

  console.log('수정 전 리렌더링') // 컴포넌트가 렌더링될 때마다 확인할 수 있게 로깅합니다.

  // 타이머를 시작하면 setInterval로 타이머값을 1초씩 증가시키는 함수를 실행합니다.
  useEffect(() => {
    console.log('effect') // effect가 실행되었는지 확인할 수 있게 로깅을 합니다.

    let timer: ReturnType<typeof setInterval>;

    if (timerOn) {
      timer = setInterval(() => {
        setTotalTime(totalTime + 1);
        setUserTimer({ totalTime });
      }, 1000);
    }

    return () => clearInterval(timer);
  }, [timerOn, setUserTimer, totalTime]);
};
```
{: file="Timer.tsx" }
<!-- prettier-ignore-end -->

`useEffect` 코드를 보면 종속성으로 `totalTime`이 들어가 있습니다. 그래서 `totalTime` 값이 변할 때마다 이펙트가 실행됩니다. 그런데 `totalTime`은 1초마다 변하는 값이므로 위 컴포넌트는 1초마다 추가적으로 이펙트가 실행됩니다.

![useEffect-before](https://user-images.githubusercontent.com/8105528/155001504-d31e16ea-bcb9-41ab-9e7f-ee3f7d09c66b.gif){: .shadow }
_매초마다 effect가 실행된다_

그런데 [리액트 공식문서의 Hooks FAQ](https://ko.reactjs.org/docs/hooks-faq.html#what-can-i-do-if-my-effect-dependencies-change-too-often)를 보면 너무 자주 변하는 종속성에 대해 최적화를 하는 예시를 찾을 수 있습니다.

## 수정한 코드

<!-- prettier-ignore-start -->
```tsx
const Timer = ({ ...props }: any) => {
  console.log('수정 후 리렌더링') // 컴포넌트가 렌더링될 때마다 확인할 수 있게 로깅을 합니다.

  useEffect(() => {
    console.log('effect') // effect가 실행되었는지 확인할 수 있게 로깅을 합니다.

    if (!timerOn) {
      return;
    }

    let currentTotalTime: number;

    const timer = setInterval(() => {
      setTotalTime((prevTotalTime) => {
        currentTotalTime = prevTotalTime + 1;
        return currentTotalTime;
      });
      setUserTimer({ totalTime: currentTotalTime });
    }, 1000);

    return () => clearInterval(timer);
  }, [timerOn]);
};
```
{: file="Timer.tsx" }
<!-- prettier-ignore-end -->

이제 이펙트는 `timerOn` 값이 변할 때만 실행되고 `totalTime`이 변하더라도 실행되지 않습니다.

![useEffect-after](https://user-images.githubusercontent.com/8105528/155001458-69c1b271-7f2a-4e34-85e0-44d647444448.gif){: .shadow }
_타이머 시작하기 또는 일시정지 버튼을 누를 때만 effect가 실행된다_

## 📚 함께 읽기

- [리액트 공식문서 - Hooks FAQ: What can I do if my effect dependencies change too often](https://ko.reactjs.org/docs/hooks-faq.html#what-can-i-do-if-my-effect-dependencies-change-too-often)
