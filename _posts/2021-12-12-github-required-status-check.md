---
title: CI가 통과하지 못할 경우 PR 머지 불가능하도록 설정하기 (feat. Github Actions)
date: 2021-12-12
categories: [TIL]
tags: [github] # TAG names should always be lowercase
image:
  src: https://noticon-static.tammolo.com/dgggcrkxq/image/upload/v1567128822/noticon/osiivsvhnu4nt8doquo0.png
  width: 100 # in pixels
  height: 100 # in pixels
  alt: Github Actions
---

## 목표

Github 저장소에 브랜치 보호 규칙을 추가하여 CI를 통과하지 못한 PR은 머지할 수 없도록 설정한다.

> 예제 CI는 Github Actions를 사용한다 - [참고](/_posts/2021-12-07-github-actions-yarn.md)


## 브랜치 보호 규칙

브랜치 보호 규칙이란 특정 브랜치에 코드 변경을 반영하는 데 있어서 방어 시스템을 갖추는 것을 말한다. 예를 들어, PR 없이는 브랜치를 머지할 수 없게 만들 수 있다. 가령, git이나 Github에 익숙하지 않은 개발자가 프로덕션 코드의 베이스가 되는 `main` 브랜치를 잘못 덮어씌우는 일을 시스템을 구축하여 미연에 방지할 수 있다.

Github 저장소마다 브랜치 보호 규칙을 추가할 수 있는데 자세한 설명은 [Github 공식문서](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/defining-the-mergeability-of-pull-requests/managing-a-branch-protection-rule)에 세세하게 잘 나와 있다.

## CI를 통과하지 못한 PR은 머지할 수 없도록 설정하기

현재 Github 저장소 > Settings > Branches > Branch Protection Rule로서 develop 브랜치를 추가한 상태이다.

![current branch rules](https://user-images.githubusercontent.com/8105528/145705054-5b7c8e3a-a229-479e-8e32-212338d93a50.png){: .shadow }
_여기서 develop 브랜치의 규칙을 편집(`Edit`)한다_

![required status check](https://user-images.githubusercontent.com/8105528/145705014-b57c9572-baed-4dac-8fb4-9dce9a9de008.png){: .shadow }
_staus check는 Circle CI나 Travis CI 같은 외부 CI를 사용할 수도 있는데, 여기서는 기존에 만들어 둔 [Github Actions](/_posts/2021-12-07-github-actions-yarn.md)를 사용한다_

현재 Github Actions CI 파일의 내용은 다음과 같다:

```yml
name: On pull request CI

on: pull_request

jobs:
  build:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [16.x]

    steps:
    - name: Check out repository
      uses: actions/checkout@v2
      # ...
```
{: file=".github/workflows/on-pull-request.yml" }

여기서 `jobs`의 이름에 해당하는 `build`를 검색창에 입력해야 한다.

![required status check with search](https://user-images.githubusercontent.com/8105528/145705190-46c2f55e-86bc-4104-ae4f-9c472cd8e5cf.png){: .shadow }

![required status check with github actions](https://user-images.githubusercontent.com/8105528/145705003-210914b1-1b89-4dda-ae70-111c2ec2ad88.png){: .shadow }
_추가가 완료되었다_

설정이 완료된 상태에서 PR을 업로드하면 status check로 등록한 CI가 `Required`로 표시되는 것을 확인할 수 있고, CI를 통과하지 못할 경우 머지 버튼이 비활성화된다:

![required statuses must pass](https://user-images.githubusercontent.com/8105528/145705684-fb6d7b18-d179-4322-ba25-84157bbc46ba.png){: .shadow }


## See also

- [Github - About protected branches](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/defining-the-mergeability-of-pull-requests/about-protected-branches)
- [Github - Branch protection rule](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/defining-the-mergeability-of-pull-requests/managing-a-branch-protection-rule)
- [Github - Required status checks](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/defining-the-mergeability-of-pull-requests/troubleshooting-required-status-checks)
