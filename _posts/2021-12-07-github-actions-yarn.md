---
title: Github Actions로 CI 구축하기 (feat. yarn)
date: 2021-12-07
categories: [TIL]
tags: [github actions] # TAG names should always be lowercase
image:
  src: https://noticon-static.tammolo.com/dgggcrkxq/image/upload/v1567128822/noticon/osiivsvhnu4nt8doquo0.png
  width: 100 # in pixels
  height: 100 # in pixels
  alt: Github Actions
excerpt_separator: <!--end-of-description-->
---

## 목표

CI/CD 플랫폼인 [Github Actions](https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions)를 사용하여 린트와 테스트를 자동화할 수 있는 파이프라인을 구축한다.

> 브랜치에 푸시가 되면 자동으로 배포가 되는 정적 사이트(ex. netlify)를 이용할 예정이므로 빌드와 배포는 다루지 않는다.

<!--end-of-description-->

## 워크플로우 시작하기

Github에서 제공하는 워크플로우 템플릿을 이용하면 우리가 원하는 자동화 작업의 워크플로우를 쉽게 만들 수 있다.

![github actions](https://user-images.githubusercontent.com/8105528/144965808-d0ddf84d-350f-48aa-a8dd-449c3b85319f.png){: .shadow }
_Github 저장소에서 Actions 탭을 누르면 워크플로우 템플릿을 이용할 수 있다_

## 워크플로우 문법 파악하기

[Github 공식문서](https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions#create-an-example-workflow)에 보면 다음과 같이 예제 코드가 나와 있다. 각 코드에 대한 설명은 다음과 같다:

```yml
# 워크플로우 이름
name: learn-github-actions

# 워크플로우 트리거
on: [push] # push를 하면 해당 워크플로우가 자동 실행된다

# 워크플로우에서 실행할 작업
jobs:
  # 작업 이름
  check-bats-version:
    # 작업이 실행될 환경
    runs-on: ubuntu-latest
    # 현재 작업에서 실행할 단계별 작업
    steps:
      - uses: actions/checkout@v2 # 사용할 액션
      - uses: actions/setup-node@v2
        with:
          node-version: "14"
      - run: npm install -g bats # 실행할 커맨드
      - run: bats -v
```

{: file=".github/workflows/learn-github-actions.yml" }

## 워크플로우 만들기

요구사항:

- 패키지 매니저로서 `yarn`을 사용한다
- 워크플로우 작업 시간을 줄이기 위해 설치된 패키지 의존성을 캐시한다
- 린트를 실행한다
- 테스트를 실행한다

{% raw %}

<!-- prettier-ignore-start -->
```yml
name: Lint and test Devnity FE

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [16.x] # 사용할 노드 버전을 명시한다

    steps:
      # 현재 저장소로 체크아웃 한다
      - name: Check out repository
        uses: actions/checkout@v2

      # 노드를 설치한다
      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v2
        with:
          node-version: ${{ matrix.node-version }}

      # yarn 캐시를 한다
      - name: Get yarn cache directory path
        id: yarn-cache-dir-path
        run: echo "::set-output name=dir::$(yarn cache dir)"

      - name: Cache yarn
        uses: actions/cache@v2
        id: yarn-cache
        with:
          path: ${{ steps.yarn-cache-dir-path.outputs.dir }}
          key: ${{ runner.os }}-yarn-${{ hashFiles('**/yarn.lock') }}
          restore-keys: |
            ${{ runner.os }}-yarn-

      # 프로젝트 의존성을 설치한다
      - name: Install a project
        run: yarn install

      # 린트를 실행한다
      - name: Lint
        run: yarn lint

      # 테스트를 실행한다
      - name: Test
        run: yarn test
```
{: file=".github/workflows/ci.yml" }
<!-- prettier-ignore-end -->

{% endraw %}

이때 린트에서 에러가 발생하거나 테스트가 실패할 경우 해당 브랜치는 머지할 수 없게 막을 수 있다. 관련 설정은 [이 글]({% post_url 2021-12-12-github-required-status-check %})을 참조한다.

![required statuses must pass](https://user-images.githubusercontent.com/8105528/145705684-fb6d7b18-d179-4322-ba25-84157bbc46ba.png){: .shadow }
_Github Actions 작업이 실패할 경우 PR에서 머지를 할 수 없다는 경고 메시지가 뜬다. "안 돼. 머지시킬 생각 없어. 돌아가."_

따라서 작업이 완료된 코드가 머지해도 괜찮은지에 대해 린트와 테스트 측면에서 자동으로 피드백을 받을 수 있게 된다.

## See also

- [Github Actions - yarn cache example](https://github.com/actions/cache/blob/master/examples.md#node---yarn)
