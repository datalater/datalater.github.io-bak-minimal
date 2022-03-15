---
title: LeetCode - Letter Combinations of a Phone Number (JavaScript)
date: 2022-03-15
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

[LeetCode - 17. Letter Combinations of a Phone Number](https://leetcode.com/problems/letter-combinations-of-a-phone-number/)를 풀고 나서 겪었던 시행착오를 기록하고 머릿속에 있던 생각을 글로 정리하면서 복기해봅니다.

<!--end-of-description-->

## DFS로 생각하기

DFS를 연습하려고 풀었던 문제였습니다. 그래서 이 문제를 어떻게 하면 DFS로 풀이할 수 있는지 중점적으로 살펴보겠습니다.

먼저 `map`을 만듭니다.

```js
var letterCombinations = function (digits) {
  const map = {
    2: "abc",
    3: "def",
    4: "ghi",
    5: "jkl",
    6: "mno",
    7: "pqrs",
    8: "tuv",
    9: "wxyz",
  };
};
```

만약 `digits`가 `"23"`으로 입력되었다면 기대되는 정답과 우리가 원하는 연산은 다음과 같습니다:

<!-- prettier-ignore-start -->
```js
Input: digits = "23";
Output: ["ad", "ae", "af", "bd", "be", "bf", "cd", "ce", "cf"];

// digits 문자열에서 i번째에 해당하는 숫자를 꺼내서 해당 숫자에 맵핑되는 문자열을 반환합니다.
digits[0] = 2 | "abc"
digits[1] = 3 | "def"

// 각 문자열을 모두 조합하는 형태로 탐색한 후 백트래킹하면서 결과를 조합합니다.
["ad", "ae", "af", "bd", "be", "bf", "cd", "ce", "cf"]
```
<!-- prettier-ignore-end -->

여기서 "각 문자열을 모두 조합하는 형태로 탐색한 후 백트래킹하면서 결과를 조합"하는 것을 `for`문으로 먼저 생각해보고 DFS 방식으로 바꿔보겠습니다.

<!-- prettier-ignore-start -->
```js
for (let i = 0; i < digits.length; i += 1) { // 0, 1
  for (let j of map[digits[i]]) { // a, b, c | d, e, f
    /**
     * 첫번째 경우
     * 
     * i = 0, j = a 
     * 
     * a와 조합해야 하는 대상으로서 우리가 원하는 건 다음에 d가 오는 것이다.
     * 그런데 d, e, f는 i = 1일 때 등장하므로 다음 반복문으로 넘어가야 만날 수 있다.
     * 
     * i에 대한 반복문으로 넘어가려면 지금까지 작성한 내용을 함수로 만들고 재귀호출하면 된다.
     * 그리고 i값을 1씩 증가시켜서 재귀호출하면 된다.
     * 
     * 그러므로 dfs(i + 1)과 같은 형태로 호출하면 다음 반복문인 i = 1로 넘어간다.
     * 다만, i = 1이 되었을 때는 a값을 그대로 활용해야 하므로 이 값을 재귀호출할 때 인자로 넘겨줘야 한다.
     * 이때 인자로 통째로 넘겨주는 게 아니라 어차피 문자열을 조합하면 되니 a를 문자열에 추가한 형태로 넘겨주면 된다.
     * 
     * dfs(i + 1, "" + j)
     * 
     * 이 형식을 일반화한다.
     * 
     * dfs(i + 1, path + j)
     */
    }
}
```
<!-- prettier-ignore-end -->

```js
var letterCombinations = function (digits) {
  const map = {
    2: "abc",
    3: "def",
    4: "ghi",
    5: "jkl",
    6: "mno",
    7: "pqrs",
    8: "tuv",
    9: "wxyz",
  };

  function dfs(index, path) {
    for (let i = index; i < digits.length; i += 1) {
      for (let j of map[digits[i]]) {
        dfs(i + 1, path + j);
      }
    }
  }

  dfs(0, "");
};
```

그런데 이렇게 하면 dfs가 종료되지 않고 끊임없이 호출되어 무한루프에 빠집니다. 재귀호출할 때는 항상 종료 조건을 떠올려야 합니다. 탐색을 모두 마치는 조건은 조합한 문자열의 길이가 `digits`의 길이와 같아지는 것입니다. 그리고 길이가 같아지면 조합한 문자열이 곧 우리가 원하는 결과물이므로 `results` 배열을 만들고 배열에 추가합니다.

```js
var letterCombinations = function (digits) {
  const map = {
    2: "abc",
    3: "def",
    4: "ghi",
    5: "jkl",
    6: "mno",
    7: "pqrs",
    8: "tuv",
    9: "wxyz",
  };

  const result = [];

  function dfs(index, path) {
    if (path.length === digits.length) {
      result.push(path);
      return;
    }

    for (let i = index; i < digits.length; i += 1) {
      for (let j of map[digits[i]]) {
        dfs(i + 1, path + j);
      }
    }
  }

  dfs(0, "");

  return result;
};
```

## 재귀함수의 프로세스 그려보기

재귀 함수가 어떻게 동작하는지 머릿속으로 그려내려면 먼저 "언제 백트래킹을 하느냐"를 "종료 조건"과 함께 생각해보는 게 도움이 되는 것 같습니다. 종료 조건은 비교적 명확하고 간단하기 때문입니다. 위 코드의 경우 dfs 함수가 종료되면(즉, `path`의 길이가 완성되면), 백트래킹하여 중첩된 `for`문의 `j`값이 기존 값(ex. `d`)에서 다음 값(ex. `e`)으로 변경됩니다. 즉, 일반적인 그래프 탐색과 마찬가지로 모든 노드를 순회하다가 끝에 다다르면 가장 마지막 노드를 빼고 다른 노드를 탐색하는 것을 반복하는 프로세스입니다.

종료 조건을 봤으니 시작 지점과 전체 흐름을 보겠습니다. 위 코드에서 루프에 대입되는 값을 적어보면 다음과 같습니다:

<!-- prettier-ignore-start -->
```js
// 시작
dfs(0, "")

// 참고 값
digits[0] = 2 | "abc"
digits[1] = 3 | "def"

// 전체 흐름
| dfs(0, "")  | dfs(1, "a")  | dfs(1, "a")  | dfs(1, "a")  | dfs(0, "")  |     |
| depth1      | depth2       | depth2       | depth2       | depth1      |     |
| i = 0       | i = 1        | i = 1        | i = 1        | i = 0       | ... |
| j = a       | j = d        | j = e        | j = e        | j = b       |     |
| dfs(1, "a") | dfs(2, "ad") | dfs(2, "ae") | dfs(2, "af") | dfs(1, "b") |     |
|             | 종료 후 백트래킹 | 종료 후 백트래킹 | 종료 후 백트래킹 |             |     |

// 노드 탐색 순서
["a", "d", "e", "f", "b", "d", "e", "f", "c", "d", "e", "f"]
```
<!-- prettier-ignore-end -->

## 📚 함께 보기

- [LeetCode - 17. Letter Combinations of a Phone Number](https://leetcode.com/problems/letter-combinations-of-a-phone-number/)
