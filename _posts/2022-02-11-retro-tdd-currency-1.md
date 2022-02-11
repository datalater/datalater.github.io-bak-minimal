---
title: 회고 - 화폐 예제 실습으로 TDD의 리듬 느껴보기
date: 2022-02-11-15:00:00+09:00
categories: [Retrospective]
tags: [tdd, tdd-by-example] # TAG names should always be lowercase
image:
  src: https://noticon-static.tammolo.com/dgggcrkxq/image/upload/v1574740301/noticon/up950fjgyekwqizq6xa6.png
  width: 100 # in pixels
  height: 100 # in pixels
  alt: TDD
mermaid: true
published: true
---

## 💁 설명

[화폐 예제 실습으로 TDD의 리듬 느껴보기 (feat. TypeScript)]({% post_url 2022-02-11-tdd-currency-1 %})를 하면서 느낀 점을 회고합니다.

## 📝 회고

> 무엇을 했고 무엇을 느꼈는지 적고 배운 점을 액션 플랜으로 연결해봅니다.

### TDD 리듬이 깨지지 않도록 할일 목록을 지속적으로 관리해보자

다중 통화를 지원하는 Money 객체를 만들기 위해 할일 목록을 작성하고 테스트 코드를 작성하여 작업을 했다.

이전에 TDD로 개발을 했을 때는 할일 목록은 처음에만 간단하게 주석으로 작성하고 주로 테스트 코드를 어떻게 작성해야 할지에 집중했다. 그런데 책을 읽고 실습하면서 느낀 점은 할일 목록을 한 곳에 두고 지속적으로 업데이트하는 점이었다. 그리고 그렇게 할일 목록을 한 곳에 두고 관리하는 게 앞으로 해야 할 일을 명확히 하는 데에 큰 도움을 주는 것 같다.

특히 테스트 코드를 빠르게 통과시키기 위해 작업을 하다 보면 지금 작성한 코드가 예기치 못한 사이드 이펙트를 커버하지 못하는 것을 깨닫고 코드를 수정하거나 추가하는 등 작업의 범위가 예상보다 커져버려서 TDD의 리듬이 깨지는 경우도 종종 있었다. 그런데 책에서 제안하는 해결책은 매우 간단했다. 문제점이 눈에 보이면 할일 목록에 적어두고 그냥 넘어가고, 기존에 하던 작업을 계속 진행한다.

이제부터는 TDD를 할 때 할일 목록을 `app.todo.md` 같은 이름의 파일로 분리해서 작업하는 동안 지속적으로 관리해봐야겠다. 그러면 기존에 TDD 리듬이 깨지던 문제를 해결할 수 있을 것 같다.

## 📚 See also

- [화폐 예제 실습으로 TDD의 리듬 느껴보기 (feat. TypeScript)]({% post_url 2022-02-11-tdd-currency-1 %})
