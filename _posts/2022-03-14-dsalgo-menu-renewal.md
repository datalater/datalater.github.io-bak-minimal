---
title: 프로그래머스 - 메뉴 리뉴얼 (JavaScript)
date: 2022-03-14
categories: [TIL, Algorithm]
tags: [알고리즘] # TAG names should always be lowercase
image:
  src: https://noticon-static.tammolo.com/dgggcrkxq/image/upload/v1577524878/noticon/gzl7ru4i4vv3phyv34y3.png
  width: 100 # in pixels
  height: 100 # in pixels
  alt: Algorithm
mermaid: true
published: true
excerpt_separator: <!--end-of-description-->
---

## 💁 설명

[프로그래머스 메뉴 리뉴얼](https://programmers.co.kr/learn/courses/30/lessons/72411) 풀이를 설명합니다.

<!--end-of-description-->

## 문제 전략 세우기

- 문자열("ABC")로 특정 글자 길이(2)만큼 만들 수 있는 모든 조합("AB", AC", BC")을 만드는 함수를 만든다.
- 문자열 배열로 특정 글자 길이(2)만큼 만들 수 있는 모든 조합을 만드는 함수를 만든다.
- 배열에서 요소의 빈도를 계산해서 객체(`{AB: 2}`)로 만드는 함수를 만든다.
- 배열에서 빈도 수가 가장 많은 요소를 구하는 함수를 만든다. 단, 빈도 수가 동일할 경우 모두 선택해야 하므로 최빈값을 담은 배열로 리턴한다.

위 내용을 간단한 할일 목록으로 정리합니다.

<!-- prettier-ignore-start -->
```diff
  문자열조합("ABC", 2) = ["AB", "AC", "BC"]
  문자열 배열 조합
  배열에서 요소의 빈도 수를 계산해서 객체로 만들기
  배열에서 최빈값을 가지는 요소 구하기 (최빈값이 같을 경우 모두 포함)
```
{: file="할일목록" }
<!-- prettier-ignore-end -->

## TDD

> 테스트를 먼저 작성하고 테스트를 통과하도록 코드를 작성합니다.

### 배열에서 요소의 빈도 수를 계산해서 객체로 만들기

할일 목록에서 제일 간단해보이는 "배열에서 요소의 빈도 수를 계산해서 객체로 만들기"를 선택해서 테스트를 작성합니다. 실패하는 테스트 먼저 작성하겠다는 의미로 빨강색으로 표시합니다.

<!-- prettier-ignore-start -->
```diff
  문자열조합("ABC", 2) = ["AB", "AC", "BC"]
  문자열 배열 조합
- 배열에서 요소의 빈도 수를 계산해서 객체로 만들기
  배열에서 최빈값을 가지는 요소 구하기 (최빈값이 같을 경우 모두 포함)
```
{: file="할일목록" }
<!-- prettier-ignore-end -->

실패하는 테스트 코드를 작성합니다.

```js
test("배열에서 요소의 빈도 수를 계산해서 객체로 만든다", () => {
  expect(count(["AB", "AC", "BC", "AB"])).toEqual({ AB: 2, AC: 1, BC: 1 });
});
```

테스트 코드를 통과하도록 코드를 작성합니다.

```js
function count(array) {
  return array.reduce((acc, cur) => {
    acc[cur] = acc[cur] ? acc[cur] + 1 : 1;
    return acc;
  }, {});
}
```

배열의 요소가 객체에 이미 있으면 현재 값에 1을 더하고, 없으면 처음 등장한 것이므로 1을 넣습니다.

<details markdown="1">
<summary><strong>동일한 코드</strong></summary>

```js
function count(arr) {
  return arr.reduce((acc, cur) => {
    acc[cur] = (acc[cur] || 0) + 1;
    return acc;
  }, {});
}
```

</details>

완료되었으므로 테스트를 통과한다는 의미로 녹색으로 표시합니다.

<!-- prettier-ignore-start -->
```diff
  문자열조합("ABC", 2) = ["AB", "AC", "BC"]
  문자열 배열 조합
+ 배열에서 요소의 빈도 수를 계산해서 객체로 만들기
  배열에서 최빈값을 가지는 요소 구하기 (최빈값이 같을 경우 모두 포함)
```
{: file="할일목록" }
<!-- prettier-ignore-end -->

### 배열에서 최빈값 요소 구하기

실패하는 테스트 먼저 작성하겠다는 의미로 빨강색으로 표시합니다.

<!-- prettier-ignore-start -->
```diff
  문자열조합("ABC", 2) = ["AB", "AC", "BC"]
  문자열 배열 조합
+ 배열에서 요소의 빈도 수를 계산해서 객체로 만들기
- 배열에서 최빈값을 가지는 요소 구하기 (최빈값이 같을 경우 모두 포함)
```
{: file="할일목록" }
<!-- prettier-ignore-end -->

실패하는 테스트 코드를 작성합니다.

주의할 점:

- 최빈값 요소가 여러 개 있을 수 있으므로 배열로 리턴한다.
- 최빈값이 1이라면 빈 배열을 리턴한다. (문제의 조건: `최소 2명 이상의 손님으로부터 주문된 단품메뉴 조합에 대해서만 코스요리 메뉴 후보에 포함하기로 했습니다`)

```js
test("배열에서 최빈값 요소를 구한다", () => {
  expect(mostFrequent(["A", "B", "C", "A"])).toEqual(["A"]);
  expect(mostFrequent(["A", "B", "C"])).toEqual([]);
  expect(mostFrequent(["A", "B", "C", "A", "C"])).toEqual(["A", "C"]);
});
```

테스트 코드를 통과하도록 코드를 작성합니다.

```js
function mostFrequent(array) {
  const countMap = count(array);

  const max = Math.max(...Object.values(countMap));

  return max > 1
    ? Object.keys(countMap).filter((key) => countMap[key] === max)
    : [];
}
```

### 문자열 조합

실패하는 테스트 먼저 작성하겠다는 의미로 빨강색으로 표시합니다.

<!-- prettier-ignore-start -->
```diff
- 문자열조합("ABC", 2) = ["AB", "AC", "BC"]
  문자열 배열 조합
+ 배열에서 요소의 빈도 수를 계산해서 객체로 만들기
+ 배열에서 최빈값을 가지는 요소 구하기 (최빈값이 같을 경우 모두 포함)
```
{: file="할일목록" }
<!-- prettier-ignore-end -->

그런데 생각해보니 문자열로 조합하는 것보다 배열의 요소를 조합하는 것이 유사하지만 더 간단한 문제로 보입니다. 그래서 배열의 요소를 조합하는 문제를 할일 목록에 추가하고 현재 작업으로 시작하겠습니다.

<!-- prettier-ignore-start -->
```diff
  문자열조합("ABC", 2) = ["AB", "AC", "BC"]
  문자열 배열 조합
+ 배열에서 요소의 빈도 수를 계산해서 객체로 만들기
+ 배열에서 최빈값을 가지는 요소 구하기 (최빈값이 같을 경우 모두 포함)
- 주어진 배열에서 특정 길이만큼 조합하기
```
{: file="할일목록" }
<!-- prettier-ignore-end -->

실패하는 테스트를 먼저 작성합니다.

```js
test("배열에서 특정 길이만큼 조합할 수 있는 모든 경우의 수를 구한다", () => {
  expect(combinate(["A", "B", "C"], 2)).toEqual([
    ["A", "B"],
    ["A", "C"],
    ["B", "C"],
  ]);
});
```

테스트 코드를 통과하도록 코드를 작성합니다.

```js
function combinate(array, n) {
  const result = [];

  if (n === 1) {
    return array.map((it) => [it]);
  }

  array.forEach((fixed, index, origin) => {
    const rest = origin.slice(index + 1);

    const combinations = combinate(rest, n - 1);

    const concated = combinations.map((combination) => [fixed, ...combination]);

    result.push(...concated);
  });

  return result;
}
```

> 코드 참조: [냥인의 블로그](https://nyang-in.tistory.com/212)

이제 다시 문자열 조합 문제로 돌아가겠습니다.

<!-- prettier-ignore-start -->
```diff
- 문자열조합("ABC", 2) = ["AB", "AC", "BC"]
  문자열 배열 조합
+ 배열에서 요소의 빈도 수를 계산해서 객체로 만들기
+ 배열에서 최빈값을 가지는 요소 구하기 (최빈값이 같을 경우 모두 포함)
+ 주어진 배열에서 특정 길이만큼 조합하기
```
{: file="할일목록" }
<!-- prettier-ignore-end -->

실패하는 테스트 코드를 작성합니다.

```js
test("주어진 문자열에서 특정한 길이만큼 조합한다", () => {
  expect(combinateString("ABC", 2)).toEqual(["AB", "AC", "BC"]);
  expect(combinateString("ABCFG", 2)).toEqual([
    "AB",
    "AC",
    "AF",
    "AG",
    "BC",
    "BF",
    "BG",
    "CF",
    "CG",
    "FG",
  ]);
});
```

테스트 코드를 통과하도록 코드를 작성합니다.

```js
function combinateString(s, n) {
  const _s = s.split();

  const result = combinate(_s, n)
    .map((it) => it.sort().join(""))
    .sort();

  return result;
}
```

문자열을 베열로 만든 후에 조합을 구하고 다시 문자열로 합치기(`join("")`) 전에 항상 오름차순으로 정렬해야 합니다. 왜냐하면 주문 "AX"와 "XA"는 동일한 주문의 조합으로 처리해야 하기 때문입니다. 이 부분을 놓치면 조합을 카운트를 할 때 문제가 발생합니다. 그리고 최종 결과에서도 오름차순으로 정렬합니다. 왜냐하면 최종 결과에서 오름차순으로 정렬하여 리턴해야 하기 때문입니다.

### 문자열 배열 조합

<!-- prettier-ignore-start -->
```diff
+ 문자열조합("ABC", 2) = ["AB", "AC", "BC"]
- 문자열 배열 조합
+ 배열에서 요소의 빈도 수를 계산해서 객체로 만들기
+ 배열에서 최빈값을 가지는 요소 구하기 (최빈값이 같을 경우 모두 포함)
+ 주어진 배열에서 특정 길이만큼 조합하기
```
{: file="할일목록" }
<!-- prettier-ignore-end -->

이 문제는 문자열이 하나가 아니라 여러 개가 담긴 배열이 주어졌을 때 전체 문자열에서 특정 길이만큼 조합하는 문제입니다. "ABC", "BCD"가 있을 때 2의 길이로 모든 조합을 구하면 "BC"는 두 번 중복되어야 합니다.

실패하는 테스트를 작성합니다.

```js
test("문자열 배열이 주어졌을 때 주어진 길이만큼 조합한다", () => {
  expect(combinateStrings(["XYZ", "XWY", "WXA"], 2)).toEqual([
    "AW",
    "AX",
    "WX",
    "WX",
    "WY",
    "XY",
    "XY",
    "XZ",
    "YZ",
  ]);
});
```

테스트 코드를 통과하도록 코드를 작성합니다.

```js
function combinateStrings(array, n) {
  return array
    .map((s) => combinateString(s, n))
    .reduce((acc, cur) => [...acc, ...cur], [])
    .sort();
}
```

<!-- prettier-ignore-start -->
```diff
+ 문자열조합("ABC", 2) = ["AB", "AC", "BC"]
+ 문자열 배열 조합
+ 배열에서 요소의 빈도 수를 계산해서 객체로 만들기
+ 배열에서 최빈값을 가지는 요소 구하기 (최빈값이 같을 경우 모두 포함)
+ 주어진 배열에서 특정 길이만큼 조합하기
```
{: file="할일목록" }
<!-- prettier-ignore-end -->

### 솔루션 함수

이제 모든 할일이 끝났으니 최종 솔루션 함수의 테스트 코드를 작성합니다.

```js
test("메뉴 리뉴얼", () => {
  expect(
    solution(["ABCFG", "AC", "CDE", "ACDE", "BCFG", "ACDEH"], [2, 3, 4])
  ).toEqual(["AC", "ACDE", "BCFG", "CDE"]);
  expect(solution(["XYZ", "XWY", "WXA"], [2, 3, 4])).toEqual(["WX", "XY"]);
});
```

테스트 코드를 통과하도록 코드를 작성합니다.

```js
function solution(orders, course) {
  return course
    .map((n) => combinateStrings(orders, n))
    .flatMap(mostFrequent)
    .sort();
}
```

## 📚 함께 보기
